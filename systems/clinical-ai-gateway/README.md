# Build a Clinical-AI Gateway

> Inside the [Global Problem Systems Engineering](../../README.md) portfolio · *Population-scale systems built for civic and public-good outcomes.*

## Overview

In this build, I created a self-hosted, vendor-agnostic Clinical-AI gateway for healthcare programs that need reliable diagnostic support in offline or low-resource settings. The system was designed to keep clinical AI access available even when cloud connectivity, vendor access, or budget limits create pressure on the care path.

The gateway gave clinical applications one OpenAI-compatible endpoint instead of forcing them to bind directly to one vendor. That kept the app contract stable while the gateway handled cloud routing, local fallback, cost controls, and data-sovereignty rules behind it.

This mattered because healthcare AI pilots often fail before reaching patients. They depend on cloud-only paths, brittle vendor bindings, unclear spend controls, or data movement that does not respect local policy. The gateway was built to make the AI layer more portable, safer to govern, and more realistic for constrained environments.

The architecture is built across **7 phases**, anchored by **The Problem: Why AI Pilots Never Reach a Patient** on the input side and **The Population-Impact Briefing: Getting the Gateway Funded** at the end. Each phase is listed in the Implementation section below.

## Architecture

```mermaid
---
title: Clinical-AI Gateway, Offline-First and Data-Sovereign
---
%%{init: {"theme":"base","themeVariables": {"primaryColor":"#1B4332","primaryTextColor":"#F4D03F","primaryBorderColor":"#F4D03F","secondaryColor":"#264653","tertiaryColor":"#2F5233","lineColor":"#F4D03F","fontFamily":"ui-monospace, SFMono-Regular, Menlo, Consolas, monospace","fontSize":"13px"}}}%%
flowchart TD
    classDef datastore fill:#264653,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef service fill:#1B4332,stroke:#F4D03F,stroke-width:2px,color:#F4D03F
    classDef event fill:#7B42BC,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef io fill:#0d1117,stroke:#F4D03F,stroke-width:1.5px,color:#F4D03F,font-style:italic

    ClinicalApp[/Clinical App: One Stable Endpoint/]
    TriageRequest[/Clinical Triage Request/]
    PatientData[/Patient Data: PHI + Identifiers/]
    Funder[/Funder: Steering Meeting/]

    subgraph Delivery["Delivery Trail: Design Before Wiring"]
        GitHubRepo[(GitHub Repo)]
        LinearPlan[(Linear Build Plan + Tickets)]
        ADRs[(Five Architecture Decision Records)]
        ImplPlan[(Implementation Plan + README)]
    end

    subgraph Stack["Self-Hosted Stack: Eight Containerized Services"]
        DockerCompose(Docker Compose Orchestration)
        LiteLLM(LiteLLM: Vendor-Agnostic Router)
        OllamaLocal(Ollama: Local Offline Inference)
        PresidioSvc(Microsoft Presidio: Anonymization)
    end

    subgraph Contract["The App Contract Stays Stable"]
        OpenAIEndpoint(OpenAI-Compatible Endpoint)
        NoVendorBind{{App Never Binds to One Vendor}}
        ConfigSwap{{Config-Driven Model Swap: No Code Change}}
    end

    subgraph Sovereignty["Data Sovereignty: Enforced at the Gateway"]
        RedactPHI(redact_phi: Runs Before Egress)
        PatIdMask[(PAT-###### to REDACTED-PATIENT-ID)]
        PresidioMask[(Named Entities to PERSON Markers)]
        CrossBorder{{Cross-Border Check: Data Region vs Model Region}}
        RejectRequest{{Region Mismatch: Reject Outright}}
    end

    subgraph Failover["Cloud Route + Health-Aware Failover"]
        ClaudePrimary(claude-primary: Cloud Model)
        CloudCreds{{Route-Back Needs Valid Cloud Credentials}}
        HealthCheck(Health-Aware Route Check)
        Unreachable{{Vendor Outage: unreachable:9999}}
        FallbackHop{{One Automatic Fallback Hop}}
    end

    subgraph Evidence["Offline Floor: Proven by Response Headers"]
        HeaderBase[(x-litellm-model-api-base: ollama:11434)]
        HeaderHops[(x-litellm-attempted-fallbacks: 1)]
        Http200{{HTTP 200 Under a Forced Cut}}
    end

    subgraph Language["Multilingual Access"]
        LangRouting(Language-Aware Routing)
        FrenchPath(French Question, French Answer)
    end

    subgraph Budget["Per-Capita Cost Governance"]
        TeamProgram[(LiteLLM Team: moh-program-alpha)]
        MaxBudget[(max_budget: $0.50 per Program)]
        VirtualKey(Team-Scoped Virtual Key: Spend Logged)
        BudgetFallback{{Budget Exhausted: Local at Zero Marginal Cost}}
    end

    subgraph Proof["Proof Layer: Funder-Readable Evidence"]
        Prometheus[(Prometheus: Metrics)]
        Grafana(Grafana: Four Panels)
        PerCapitaGauge[(Per-Capita Gauge: Remaining Team Budget)]
        FiveBars{{Automated Five-Bar Validation Drills}}
    end

    subgraph Packaging["Path to Production"]
        Kubernetes(Kubernetes Manifests)
        TerraformPkg(Terraform Packaging)
    end

    subgraph Briefing["Population-Impact Briefing"]
        ImpactBrief(Two-Minute Brief: Problem, Proof, Cost, Caveats)
        AnnotatedDiagram[(Annotated Flow: Money, Connectivity, Patient Data)]
        TwoAudience(Two-Audience Walkthrough)
    end

    CareContinues[/Care Path Continues: Answer Served Either Way/]

    GitHubRepo -- "tracked from design" --> LinearPlan
    LinearPlan -- "ten tickets" --> ADRs
    ADRs -- "contract, fallback, sovereignty, bars" --> ImplPlan
    ImplPlan -- "architecture before wiring" --> DockerCompose

    DockerCompose -- "brings up the router" --> LiteLLM
    DockerCompose -- "brings up the local floor" --> OllamaLocal
    DockerCompose -- "brings up anonymization" --> PresidioSvc
    LiteLLM -- "exposes" --> OpenAIEndpoint
    ClinicalApp -- "calls one endpoint" --> OpenAIEndpoint
    TriageRequest -- "clinical prompt" --> OpenAIEndpoint
    OpenAIEndpoint -- "vendor stays behind the gateway" --> NoVendorBind
    NoVendorBind -- "routing changes, contract does not" --> ConfigSwap

    PatientData -- "identifiers in the prompt" --> RedactPHI
    OpenAIEndpoint -- "before any model sees it" --> RedactPHI
    PresidioSvc -- "detects named entities" --> PresidioMask
    RedactPHI -- "custom ID pattern" --> PatIdMask
    RedactPHI -- "anonymize entities" --> PresidioMask
    PatIdMask -- "redacted text" --> CrossBorder
    PresidioMask -- "redacted text" --> CrossBorder
    CrossBorder -- "regions disagree" --> RejectRequest
    CrossBorder -- "regions match" --> HealthCheck

    HealthCheck -- "cloud healthy" --> ClaudePrimary
    ClaudePrimary -. "route-back is credential-gated" .-> CloudCreds
    HealthCheck -- "connectivity cut" --> Unreachable
    Unreachable -- "primary failed" --> FallbackHop
    CloudCreds -- "no usable key" --> FallbackHop
    FallbackHop -- "serve locally" --> OllamaLocal

    OllamaLocal -- "proves the local route" --> HeaderBase
    FallbackHop -- "proves one hop" --> HeaderHops
    HeaderBase -- "evidence" --> Http200
    HeaderHops -- "evidence" --> Http200
    Http200 -. "contract intact, no app change" .-> ConfigSwap

    LangRouting -- "language-aware policy" --> LiteLLM
    LangRouting -- "French in, French out" --> FrenchPath

    TeamProgram -- "one team per health program" --> MaxBudget
    MaxBudget -- "scopes the key" --> VirtualKey
    VirtualKey -- "spend logged per program" --> LiteLLM
    MaxBudget -- "cap reached" --> BudgetFallback
    BudgetFallback -- "zero marginal cost" --> OllamaLocal

    LiteLLM -- "scrapes" --> Prometheus
    VirtualKey -- "remaining_team_budget_metric" --> Prometheus
    Prometheus -- "feeds four panels" --> Grafana
    Grafana -- "renders" --> PerCapitaGauge
    Http200 -- "reach bar" --> FiveBars
    RejectRequest -- "compliance bar" --> FiveBars
    BudgetFallback -- "cost bar" --> FiveBars
    FrenchPath -- "access bar" --> FiveBars

    FiveBars -- "packaged for deployment" --> Kubernetes
    Kubernetes -- "declared as code" --> TerraformPkg

    PerCapitaGauge -- "cost per program" --> ImpactBrief
    FiveBars -- "proof for non-technical readers" --> ImpactBrief
    ImpactBrief -- "how money and data move" --> AnnotatedDiagram
    ImpactBrief -- "what changed, what it cost" --> TwoAudience
    TwoAudience -- "funding decision" --> Funder
    AnnotatedDiagram -- "funding decision" --> Funder

    ClaudePrimary -- "cloud answer" --> CareContinues
    OllamaLocal -- "offline answer" --> CareContinues
    CareContinues -- "same endpoint, either way" --> ClinicalApp

    class GitHubRepo,LinearPlan,ADRs,ImplPlan,PatIdMask,PresidioMask,HeaderBase,HeaderHops,TeamProgram,MaxBudget,Prometheus,PerCapitaGauge,AnnotatedDiagram datastore
    class DockerCompose,LiteLLM,OllamaLocal,PresidioSvc,OpenAIEndpoint,RedactPHI,ClaudePrimary,HealthCheck,LangRouting,FrenchPath,VirtualKey,Grafana,Kubernetes,TerraformPkg,ImpactBrief,TwoAudience service
    class NoVendorBind,ConfigSwap,CrossBorder,RejectRequest,CloudCreds,Unreachable,FallbackHop,Http200,BudgetFallback,FiveBars event
    class ClinicalApp,TriageRequest,PatientData,Funder,CareContinues io
```

The diagram shows the topology and data flow of the system as built. The full architectural narrative, with screenshots and prose, lives in [`documents/clinical-ai-gateway.md`](./documents/clinical-ai-gateway.md).

## Implementation

This system is built across **7 phases**:

1. **The Problem: Why AI Pilots Never Reach a Patient**
2. **Standing Up the Clinical-AI Gateway Stack**
3. **Proving the Contract and the Offline Floor**
4. **Implementing Data Sovereignty and Clinical Safety**
5. **Health-Aware Failover, Multilingual Access, and Per-Capita Cost Governance**
6. **The Funder Dashboard and Five-Bar Validation Drills**
7. **The Population-Impact Briefing: Getting the Gateway Funded**

For the full walkthrough with screenshots and step-by-step content, see [`documents/clinical-ai-gateway.md`](./documents/clinical-ai-gateway.md).

## Validation

Each build phase below is documented in [`documents/clinical-ai-gateway.md`](./documents/clinical-ai-gateway.md), with screenshots, configuration, and notes as captured during the build:

- ✅ The Problem: Why AI Pilots Never Reach a Patient
- ✅ Standing Up the Clinical-AI Gateway Stack
- ✅ Proving the Contract and the Offline Floor
- ✅ Implementing Data Sovereignty and Clinical Safety
- ✅ Health-Aware Failover, Multilingual Access, and Per-Capita Cost Governance
- ✅ The Funder Dashboard and Five-Bar Validation Drills
- ✅ The Population-Impact Briefing: Getting the Gateway Funded
