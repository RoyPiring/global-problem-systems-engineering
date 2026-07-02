<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Build a Real-Time Scam Detection API

**Project Link:** [View Project](https://nextwork.ai/projects/0610f18e-ca3f-4d04-8690-ce648819f44d)

**Author:** Roy Piring Jr  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/0610f18e-ca3f-4d04-8690-ce648819f44d_gogd3aec)

## Stopping a $16 Billion Scam Machine

### The problem, the pattern, and the human cost

This project builds PhishGuard, a serverless real-time scam detection API deployed on Google Cloud Functions. The system fetched and scored suspect URLs against live threat intelligence feeds from PhishTank and URLhaus, then used a heuristic classifier to return structured verdicts in near real time.

The design mattered because phishing and malware distribution move fast. PhishGuard gave users a low-cost, sub-second defense path that could check a URL before a bad link caused damage.

The human cost was the reason for the build. Fraud losses reached $15.9 billion in 2025, so the system focused on fast detection, clear verdicts, and a practical way to reduce exposure before scams reached the user.

## Setting Up the Development Environment

### Tools and goals for this step

I installed Python 3.11+, Terraform, and the gcloud CLI to prepare the development environment. I also obtained the required API keys from PhishTank and URLhaus so the detection service could check suspect URLs against live threat intelligence sources.

I created the phishguard root directory and organized the build into infra, src, eval, and docs. That structure separated infrastructure, application code, evaluation work, and documentation so each part of the system had a clear place.

I stored the API keys and GCP project ID in a local .env file and added that file to .gitignore. This kept sensitive credentials and local environment artifacts out of version control.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/0610f18e-ca3f-4d04-8690-ce648819f44d_fe4hv49q)

### Project structure and secure API key storage

PhishGuard lives in this workspace with four top-level dirs: infra/ (Terraform/IaC), src/ (application code), eval/ (tests/metrics), and docs/ (documentation), plus config/ and scripts/ for GCP setup helpers. API keys and the GCP project ID are in a project-local .env file using GOOGLE_SAFEBROWSING_API_KEY, URLHAUS_AUTH_KEY, and GCP_PROJECT_ID. That file is listed in .gitignore so secrets stay out of git and off shared repos.

## Designing Before Building: Pre-Artifacts

### Why design-first matters for a production system

I completed the directory setup and secured the credentials before moving into system design. That gave PhishGuard a clean workspace, clear code boundaries, and a safer credential path before the service logic was written.

I created the phishguard root directory with subfolders for infra, src, eval, and docs. I also stored sensitive credentials in a .env file and explicitly ignored that file through .gitignore to reduce the risk of accidental exposure.

For the design work, I defined the need for an Architecture Decision Record covering GCP as the cloud provider, a heuristic-based detection pattern, and the path toward the $1/user pricing model. I also planned the C4 architecture diagrams for the Context, Container, and Component views so the system could be explained before it was deployed.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/0610f18e-ca3f-4d04-8690-ce648819f44d_da86f6w8)

### Architecture decision: GCP over AWS

The ADR selected GCP Cloud Functions 2nd gen mainly for cost at MVP scale. GCP provided about 2M free invocations per month compared with 1M on Lambda, and HTTP triggers were included without a separate API Gateway fee.

The AWS path would add roughly $1-3.50 per million requests through API Gateway. For a real-time /detect endpoint built around low-cost user protection, that extra request cost mattered.

The ADR also cited lower cold starts at about 200 ms for Python, up to 1000 concurrent requests per instance, and a 60-minute HTTP timeout. Those limits fit the real-time detection workload and supported sub-second detection with near-zero infrastructure cost at MVP scale.

## Building the Detection Service with Multi-Agent Parallelization

### Three parallel agents, one detection engine

I split the build across three parallel agents so infrastructure, service logic, and evaluation could move at the same time without mixing responsibilities. Each agent owned a specific part of the detection engine.

The infrastructure agent built the Terraform configuration in /infra for GCP Cloud Functions 2nd gen, a storage bucket for feed caching, and a Cloud Scheduler job for hourly feed refreshes. That gave the service a deployable cloud foundation and a way to keep threat feeds current.

The service agent built the HTTP entry point, fast-path feed lookup, heuristic scorer, model gateway interface, and verdict formatter in /src, while the evaluation agent built the labeled dataset, scoring script, and latency measurement script in /eval. Together, those paths produced a detection service that could be deployed, tested, and measured against the defined OpenAPI contract and SLIs.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/0610f18e-ca3f-4d04-8690-ce648819f44d_0xbo5qs2)

### The model gateway interface and heuristic scorer

The model gateway was built as a swappable scorer interface through DetectionModel.score(features). This kept ingestion, fast-path lookup, and verdict formatting separate from the specific detection backend.

The heuristic scorer sat behind that interface as the default implementation. It was fast, free at runtime, and did not depend on an LLM or external model API.

That matched the ADR’s fast-path plus model gateway pattern. Known-bad URLs resolved quickly from threat feeds, while unknown URLs went through a replaceable scorer that could later be upgraded to ML or an LLM without rewriting the full pipeline.

## Deploying to GCP and Validating Against SLOs

### From local build to live cloud endpoint

I deployed the PhishGuard detection service to GCP using Terraform. This moved the API from the local build into a live cloud endpoint where the real /detect path could be tested.

After deployment, I validated the service against the predefined SLOs. The target was precision and recall at or above 0.8, with p95 latency under 500 ms.

I also created the formal operational runbook for the system. That gave the service an operating path after deployment instead of stopping at a working endpoint.

### Precision, recall, F1, and latency results

Against the live .run.app endpoint, precision was 1.0, recall was 0.6, and F1 was 0.75. The labeled sample produced 9 true positives, 0 false positives, and 6 false negatives across 20 samples.

The p95 latency was 137.62 ms over 10 sequential requests. That result met the real-time performance target because it stayed well under the 500 ms SLO.

Precision and latency met their SLOs. Recall did not meet the target because 0.6 was below the required 0.8, which made recall the main quality gap after live validation.

## Post-Artifacts and Stakeholder Presentation

### Compiling the evidence bundle

I compiled the post-artifact bundle after validating the live system. The bundle included the evaluation report, latency report, threat model, and model card for the scorer.

The evaluation report showed which SLOs passed and which one failed. The latency report confirmed that the service performed within the real-time target, while the threat model documented the main system risks.

I also documented user-scale projections from 100 to 1,000,000 users, covering infrastructure cost and revenue growth. Then I prepared a 5-slide stakeholder presentation focused on impact metrics, the business model, and system limitations.

### Threat model: four risks and residual honesty

The threat model identified four main risks: false positives blocking legitimate messages, feed outages forcing all traffic through the scorer, model drift as phishing tactics change, and adversarial evasion through homoglyphs, encoding, and redirect chains.

The system acknowledged residual risk because signature-based and heuristic detection cannot eliminate fraud. New attacker infrastructure that is not yet present in any feed can still evade detection until the feeds catch up.

PhishGuard shortened time-to-detection, but it did not replace user education, bank fraud controls, or law enforcement. That limitation mattered because the system had to be useful without pretending it could solve every fraud path.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/0610f18e-ca3f-4d04-8690-ce648819f44d_w6z3vja4)

### What the system honestly cannot do

The presentation stated that PhishGuard cannot detect phishing on novel infrastructure that is not yet in any threat feed. Signature and heuristic approaches always lag behind new attacker infrastructure.

It also stated that PhishGuard cannot eliminate fraud. The system shortens detection time, but it does not replace user education, bank fraud controls, or law enforcement.

The heuristic scorer also drifts without an adversarial feedback loop. The estimate was 30-90 days before significant accuracy loss if the weights are not updated.

## Secret Mission: Takedown Case Routing

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/0610f18e-ca3f-4d04-8690-ce648819f44d_wcsacvio)

### Auto-generating abuse reports with a human approval gate

Auto-case creation required three conditions to be true at the same time. The verdict had to be MALICIOUS, confidence had to be at least 0.8, and at least one threat feed hit had to confirm the indicator through a non-empty feed_hits value.

Human approval was required before dispatch because false positives can cause real harm by blocking legitimate communications or triggering incorrect abuse reports. The human-in-the-loop gate matched the ADR and kept takedown requests from being sent on unreviewed detections.

Cases stayed in pending_review until an operator approved them through POST /review/{case_id}/approve. That made automation useful for case preparation while keeping final dispatch under human control.

## Reflections and Lessons Learned

### Tools and concepts mastered

The key tools I used included Python for service logic, Terraform for Infrastructure as Code on GCP, the gcloud CLI for deployment, and threat intelligence APIs such as URLhaus and OpenPhish.

The main concepts I learned included fast-path detection for low latency, serverless architecture with Google Cloud Functions, design-before-build planning, and structured takedown-case routing with a human-in-the-loop review gate.

The larger lesson was that detection systems need more than a verdict. They need safe credential handling, live threat data, clear scoring logic, measurable SLOs, and an honest operating model for what the system can and cannot catch.

### Time taken and biggest challenges

This build took me approximately 90 minutes. That time covered the environment setup, credential handling, system design, service implementation, GCP deployment, SLO validation, evidence bundle creation, and takedown-case routing.

The most challenging part was implementing the takedown-case routing logic. The verdict, confidence score, and feed hits all had to align correctly before the JSON case file could be created.

That challenge mattered because takedown automation has a higher blast radius than detection alone. A bad detection result can be corrected, but a bad abuse report can create real harm if it is dispatched without review.

### What's next

I completed this build today to learn how to architect and deploy a real-time, serverless phishing detection API on GCP. The system used live threat intelligence and automated takedown workflows to move from URL detection to case preparation.

Moving forward, I want to learn how to integrate machine learning-based classification models into the existing pipeline. That would help PhishGuard detect novel threats that are not yet flagged by public threat feeds.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/0610f18e-ca3f-4d04-8690-ce648819f44d)*
