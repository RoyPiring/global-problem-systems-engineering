<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Build a Clinical-AI Gateway

**Project Link:** [View Project](https://nextwork.ai/projects/cbda214f-b779-409a-9f14-3ab27cf4d775)

**Author:** Roy Piring Jr  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/cbda214f-b779-409a-9f14-3ab27cf4d775_htd7jita)

## The Problem: Why AI Pilots Never Reach a Patient

### Mission context and engineering rationale

In this build, I created a self-hosted, vendor-agnostic Clinical-AI gateway for healthcare programs that need reliable diagnostic support in offline or low-resource settings. The system was designed to keep clinical AI access available even when cloud connectivity, vendor access, or budget limits create pressure on the care path.

The gateway gave clinical applications one OpenAI-compatible endpoint instead of forcing them to bind directly to one vendor. That kept the app contract stable while the gateway handled cloud routing, local fallback, cost controls, and data-sovereignty rules behind it.

This mattered because healthcare AI pilots often fail before reaching patients. They depend on cloud-only paths, brittle vendor bindings, unclear spend controls, or data movement that does not respect local policy. The gateway was built to make the AI layer more portable, safer to govern, and more realistic for constrained environments.

## Standing Up the Clinical-AI Gateway Stack

### What this step achieves

In this step, I completed the delivery infrastructure setup for the Clinical-AI gateway. I initialized the GitHub repository and created the Linear build plan with the required tickets so the work could be tracked from design through validation.

I authored the five Architecture Decision Records, the implementation plan, the system architecture diagram, and the README. Those artifacts defined the gateway contract, the offline fallback path, the sovereignty controls, and the validation bars before the service stack was wired together.

I committed and pushed the design artifacts to the repository, then prepared to write the gateway configuration for the eight containerized services. That gave the build a documented architecture before the OpenAI-compatible contract was tested.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/cbda214f-b779-409a-9f14-3ab27cf4d775_7czfhmov)

### Gateway value over direct API calls

The gateway provided one OpenAI-compatible endpoint so clinical apps did not hard-bind to Anthropic or any single cloud model provider. That kept the application interface stable even when routing changed behind the gateway.

It also provided automatic offline fallback to local Ollama when the cloud path failed. Budget caps and spend tracking were enforced per health program, which gave operators a way to preserve access without losing control of cost.

The gateway added sovereignty controls that direct API calls did not provide. PHI redaction, cross-border blocking, and multilingual routing happened at the gateway layer before the request reached a model.

## Proving the Contract and the Offline Floor

### Validating the reach bar

In this step, I tested whether the gateway could preserve the clinical application contract when the cloud path failed. The test used a clinical triage request to confirm the active route, then simulated a connectivity failure to see whether the local model took over.

The key validation was that the app continued to call the same endpoint. The gateway, not the app, made the routing decision.

This mattered because the offline floor was the core reliability claim. The system had to keep serving a response locally when the cloud model became unreachable.

### Connectivity-cut drill result

After the connectivity cut, the response header x-litellm-model-api-base showed http://ollama:11434. That proved the cloud endpoint was unreachable and LiteLLM served the request through local Ollama.

The header x-litellm-attempted-fallbacks: 1 confirmed that the gateway performed one automatic fallback hop. The route changed, but the application contract stayed intact.

That result proved the offline floor. The clinical app did not need a code change to keep working when the cloud path failed.

### Config-driven model swap across three requests

The first request asked for claude-primary. Because there was no usable Anthropic key, the gateway fell back to Ollama and returned api_base = http://ollama:11434.

During the connectivity cut, Claude was pointed at unreachable:9999. The request still returned HTTP 200, showed one fallback, and used the same Ollama floor. That proved the offline behavior under a forced failure.

After the unreachable endpoint was removed, the route still used Ollama because real cloud credentials were not available. The route-back behavior requires valid cloud credentials, so the honest result was that the same endpoint and app contract worked, while true cloud restoration still depended on a working cloud key.

## Implementing Data Sovereignty and Clinical Safety

### PHI redaction and cross-border policy

In this step, I implemented the clinical safety and sovereignty layer around the gateway. The controls focused on protecting patient identifiers before egress and blocking requests when region policy did not match the model route.

PHI redaction ran before data left the network. That ensured patient identifiers were removed or anonymized before a cloud model path could receive the prompt.

The cross-border policy added a second control. Even if text was redacted, the request could still be rejected when the data region and model region did not match the configured policy.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/cbda214f-b779-409a-9f14-3ab27cf4d775_02wkjho7)

### How patient identifiers are protected before egress

Before anything left the network, redact_phi ran on the message text. Custom patient IDs such as PAT-###### were replaced with [REDACTED-PATIENT-ID].

Presidio also anonymized detected entities such as names, replacing them with markers like <PERSON>. This protected obvious identifiers before the request reached an external model path.

Cross-border checks could reject the request entirely when the data region did not match the model region. That made sovereignty a gateway-enforced rule instead of a best-effort reminder.

## Health-Aware Failover, Multilingual Access, and Per-Capita Cost Governance

### Resilience, language routing, and budget metering

In this step, I built the resilience and cost-governance layer for the Clinical-AI gateway. The system had to route around vendor failure, preserve language access, and track spend by health program.

The resilience test simulated a vendor outage by sending the cloud route to an unreachable endpoint. The gateway then triggered the fallback chain and served the request through the local model.

I also configured language-aware routing so a clinical question asked in French could be processed and answered in French. Budget metering was handled through team-based spending limits, with fallback to the local model when a program exhausted its cloud budget.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/cbda214f-b779-409a-9f14-3ab27cf4d775_wawc38s8)

### Per-capita budget enforcement for a health program

A health program was modeled as a LiteLLM team with a max_budget, such as $0.50 for moh-program-alpha. That gave each program its own budget boundary.

Clinical apps used a team-scoped virtual key, so each request’s spend was logged against that program. This made cost visible at the program level instead of hiding it inside shared gateway usage.

When the budget was exhausted, budget_fallbacks routed traffic to ollama-local at zero marginal cost instead of blocking care. That protected the clinical workflow while keeping the cloud spend cap intact.

## The Funder Dashboard and Five-Bar Validation Drills

### Building the proof layer

In this step, I configured the proof layer for the gateway. Grafana pulled data from Prometheus, and the dashboard was designed to show the signals funders needed to understand cost, reach, reliability, and fallback behavior.

The dashboard used four panels to translate gateway behavior into program-level evidence. It showed budget remaining, request volume, fallback behavior, and operational status in a format non-technical stakeholders could read.

I also automated the five-bar validation process and packaged the infrastructure with Kubernetes and Terraform. That gave the gateway a path from local proof to production-style deployment.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/cbda214f-b779-409a-9f14-3ab27cf4d775_8qtwqu63)

### What the per-capita gauge tells a funder

The per-capita gauge showed remaining budget for the health-program team, such as moh-program-alpha. The value came from LiteLLM’s litellm_remaining_team_budget_metric.

That value represented max_budget minus tracked spend for the team’s virtual key. When spend was near zero, the remaining budget stayed close to $0.50.

For a funder, this was the clearest program-cost signal. It showed how much money remained before budget_fallbacks shifted traffic to free local Ollama.

## The Population-Impact Briefing: Getting the Gateway Funded

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/cbda214f-b779-409a-9f14-3ab27cf4d775_kxnbe0gw)

### Three steering-meeting artifacts for non-technical funders

The population-impact briefing answered the funding question directly. It explained the problem, the solution, the proof, the cost, and the caveats in under two minutes.

The annotated system diagram showed how money, connectivity, and patient data moved through the gateway. That helped funders understand what happened during normal cloud routing, offline fallback, and budget fallback.

The two-audience walkthrough gave funders the non-technical half of the story. It explained what changed, what it cost per patient, what the drills proved, and what came next.

## Reflections and Lessons Learned

### Key tools and concepts

The key tools I used included Docker Compose for container orchestration, LiteLLM for the model-agnostic routing layer, Ollama for local offline inference, and Microsoft Presidio for data anonymization.

I also used Grafana and Prometheus for observability, plus GitHub and Linear for delivery tracking and release structure. These tools gave the build a working gateway, a proof layer, and a project trail.

The main concepts I learned included vendor-agnostic clinical AI routing, offline-first reliability, data sovereignty in high-stakes environments, per-capita cost governance for public health programs, and documentation that speaks to both DevOps teams and non-technical funders.

### Time and challenges

This build took me approximately 90 minutes. That time covered repository setup, design artifacts, gateway configuration, LiteLLM routing, Ollama fallback, PHI redaction, region policy, budget metering, dashboard work, and validation drills.

The hardest part was making the Ollama local model work cleanly behind the LiteLLM gateway during offline conditions. The failover logic had to trigger when the cloud path failed while keeping the same application-facing contract.

The route-back result also had an honest limit. The app contract stayed stable, but full cloud restoration required valid cloud credentials. Without those credentials, the gateway correctly stayed on the local Ollama path.

I completed this build to learn how to build a vendor-agnostic, offline-first Clinical-AI gateway for data-sovereign and cost-constrained healthcare environments. The next skill I want to build is automated monitoring and alerting for edge-deployed AI services so long-term reliability can be measured and maintained.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/cbda214f-b779-409a-9f14-3ab27cf4d775)*
