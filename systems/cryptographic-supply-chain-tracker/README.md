# Cryptographic Supply Chain Tracker

> Inside the [Global Problem Systems Engineering](../../README.md) portfolio · *Population-scale systems built for civic and public-good outcomes.*

## Overview

This project builds a serverless supply chain tracking system that cryptographically signs every handoff between parties to create verifiable proof of custody across the full lifecycle of a product.

The system addresses a core weakness in modern supply chains: the inability to prove what actually happened between organizations during transit. Traditional tracking systems record events, but they do not guarantee integrity or authenticity. This creates gaps where timestamps can be altered, custody records skipped, or products substituted without reliable detection. The architecture is modeled around real-world failures such as the 2025 contaminated infant formula incident, where missing or unverifiable handoff records created uncertainty around where the breakdown occurred.

The architecture is built across **11 phases**, anchored by **The Crisis This System Was Built to Solve** on the input side and **💎 Secret Mission: SLSA L1 Compliance Dashboard** at the end. Each phase is listed in the Implementation section below.

## Architecture

```mermaid
---
title: Cryptographic Supply Chain Tracker
---
%%{init: {"theme":"base","themeVariables": {"primaryColor":"#1B4332","primaryTextColor":"#F4D03F","primaryBorderColor":"#F4D03F","secondaryColor":"#264653","tertiaryColor":"#2F5233","lineColor":"#F4D03F","fontFamily":"ui-monospace, SFMono-Regular, Menlo, Consolas, monospace","fontSize":"13px"}}}%%
flowchart TD
    classDef datastore fill:#264653,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef service fill:#1B4332,stroke:#F4D03F,stroke-width:2px,color:#F4D03F
    classDef event fill:#7B42BC,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef io fill:#0d1117,stroke:#F4D03F,stroke-width:1.5px,color:#F4D03F,font-style:italic

    Producer[/Producer Party: Origin Handoff/]
    Shipper[/Shipper Party: Transit Custody/]
    Retailer[/Retailer Party: Final Receipt/]
    Auditor[/External Auditor: Public Verification/]

    subgraph Governance["DevSecOps Governance Layer"]
        Cursorrules[(.cursorrules: Agent Ownership Contract)]
        SAM[(SAM Template: Versioned Infrastructure)]
        IAM[(Least-Privilege IAM Roles)]
        Rotation[(Key Rotation Runbook)]
        ADR1[(ADR-001: Cryptographic Chain of Custody)]
        ADR2[(ADR-002: Canonical JSON Serialization)]
        ADR3[(ADR-003: Independent Anomaly Validation)]
    end

    subgraph WriteFlow["Cryptographic Write Path"]
        Emitter(EmitterFunction: Sequence Enforcement)
        OrderGate{{HTTP 400 Gate: Party Ordering}}
        Canonicalize(Canonical JSON Serializer)
        Signer(SignerVerifierFunction: ECDSA over Hash)
        KMS[(AWS KMS: ECDSA Signing Key)]
        Ledger[(DynamoDB Ledger: chain_id + ledger_key)]
    end

    subgraph DetectFlow["Independent Detection Path"]
        Detector(AnomalyDetectorFunction)
        Rule1{{previous_event_id continuity}}
        Rule2{{timestamp tamper detection}}
        Rule3{{signature presence + validity}}
    end

    subgraph PublicFlow["Public Audit Surface"]
        AuditApi(AuditApiFunction)
        Gateway(API Gateway + OpenAPI Spec)
    end

    subgraph Compliance["Compliance Evidence Store"]
        S3Audit[(Encrypted S3: Tamper-Resistant Evidence)]
        CloudTrail[(CloudTrail: KMS Sign + Verify Operations)]
    end

    Producer -- "emit producer event" --> Emitter
    Shipper -- "emit shipper event" --> Emitter
    Retailer -- "emit retailer event" --> Emitter

    Cursorrules -. "scopes" .-> SAM
    SAM -. "provisions" .-> Emitter
    SAM -. "provisions" .-> Signer
    SAM -. "provisions" .-> Detector
    SAM -. "provisions" .-> AuditApi
    IAM -. "binds" .-> Signer
    Rotation -. "governs" .-> KMS
    ADR1 -. "ratifies" .-> Signer
    ADR2 -. "ratifies" .-> Canonicalize
    ADR3 -. "ratifies" .-> Detector

    Emitter -- "validate sequence" --> OrderGate
    OrderGate -- "if invalid: 400 + drop" --> Auditor
    OrderGate -- "if valid: serialize" --> Canonicalize
    Canonicalize -- "deterministic bytes" --> Signer
    Signer -- "sign hash" --> KMS
    KMS -- "ECDSA signature" --> Signer
    Signer -- "store signed event" --> Ledger

    Ledger -- "rebuild chain" --> Detector
    Detector -- "rule 1" --> Rule1
    Detector -- "rule 2" --> Rule2
    Detector -- "rule 3" --> Rule3
    Rule1 -- "anomaly flagged" --> S3Audit
    Rule2 -- "anomaly flagged" --> S3Audit
    Rule3 -- "anomaly flagged" --> S3Audit

    Auditor -- "GET /chains/:id/verify" --> Gateway
    Gateway -- "route + CORS + status" --> AuditApi
    AuditApi -- "read events" --> Ledger
    AuditApi -- "verify integrity" --> Detector
    AuditApi -- "signed JSON response" --> Gateway

    KMS -. "logs operations" .-> CloudTrail
    Signer -. "writes evidence" .-> S3Audit
    CloudTrail -. "preserves trail" .-> S3Audit
    class Cursorrules,SAM,IAM,Rotation,ADR1,ADR2,ADR3,KMS,Ledger,S3Audit,CloudTrail datastore
    class Emitter,Canonicalize,Signer,Detector,AuditApi,Gateway service
    class OrderGate,Rule1,Rule2,Rule3 event
    class Producer,Shipper,Retailer,Auditor io
```

The diagram shows the topology and data flow of the system as built. The full architectural narrative, with screenshots and prose, lives in [`documents/cryptographic-supply-chain-tracker.md`](./documents/cryptographic-supply-chain-tracker.md).

## Implementation

This system is built across **11 phases**:

1. **The Crisis This System Was Built to Solve**
2. **Setting Up the DevSecOps Environment**
3. **Deploying the Full Serverless Stack**
4. **Building the Infrastructure with Agent 1**
5. **Simulating the 3-Party Supply Chain with Agent 2**
6. **Cryptographic Signing with AWS KMS via Agent 3**
7. **Detecting Anomalies and Chain Breaks with Agent 4**
8. **Building the Public Audit API with Agent 5**
9. **Generating Leadership Effectiveness Artifacts**
10. **Running 4 Crisis-Inspired End-to-End Scenarios**
11. **💎 Secret Mission: SLSA L1 Compliance Dashboard**

For the full walkthrough with screenshots and step-by-step content, see [`documents/cryptographic-supply-chain-tracker.md`](./documents/cryptographic-supply-chain-tracker.md).

## Validation

Each build phase below is documented in [`documents/cryptographic-supply-chain-tracker.md`](./documents/cryptographic-supply-chain-tracker.md), with screenshots, configuration, and notes as captured during the build:

- ✅ The Crisis This System Was Built to Solve
- ✅ Setting Up the DevSecOps Environment
- ✅ Deploying the Full Serverless Stack
- ✅ Building the Infrastructure with Agent 1
- ✅ Simulating the 3-Party Supply Chain with Agent 2
- ✅ Cryptographic Signing with AWS KMS via Agent 3
- ✅ Detecting Anomalies and Chain Breaks with Agent 4
- ✅ Building the Public Audit API with Agent 5
- ✅ Generating Leadership Effectiveness Artifacts
- ✅ Running 4 Crisis-Inspired End-to-End Scenarios
