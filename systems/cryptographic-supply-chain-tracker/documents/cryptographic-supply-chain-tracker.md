<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Cryptographic Supply Chain Tracker

**Project Link:** [View Project](https://learn.nextwork.org/projects/efd78660-3d95-4d31-9398-cc12c15f2a36)

**Author:** Roy Piring Jr  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/efd78660-3d95-4d31-9398-cc12c15f2a36_agpk8v84)

## The Crisis This System Was Built to Solve

### Why cryptographic supply chain verification matters

This project builds a serverless supply chain tracking system that cryptographically signs every handoff between parties to create verifiable proof of custody across the full lifecycle of a product.

The system addresses a core weakness in modern supply chains: the inability to prove what actually happened between organizations during transit. Traditional tracking systems record events, but they do not guarantee integrity or authenticity. This creates gaps where timestamps can be altered, custody records skipped, or products substituted without reliable detection. The architecture is modeled around real-world failures such as the 2025 contaminated infant formula incident, where missing or unverifiable handoff records created uncertainty around where the breakdown occurred.

## Setting Up the DevSecOps Environment

### Environment and toolchain goals

The environment establishes the development, deployment, and governance layers for the platform.

AWS SAM CLI is installed to manage serverless packaging and deployment workflows, while Python and AWS CLI provide the runtime and cloud interaction layers. The project is scaffolded using a SAM template to standardize structure and deployment behavior from the beginning. A .cursorrules file is also introduced to define DevSecOps conventions, ownership boundaries, and operational rules across all agents working within the repository.

The setup mirrors a real engineering organization where multiple teams work in parallel while still following centralized governance and security standards.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/efd78660-3d95-4d31-9398-cc12c15f2a36_ka0u2ayb)

### Encoding team conventions with .cursorrules

The .cursorrules file acts as the operational contract for the repository.

It defines ownership boundaries for infrastructure, security, quality, platform services, and chain processing so that agents do not overwrite or interfere with each other’s work. Naming conventions, security constraints, and file responsibilities are enforced at the workflow level, creating consistency across all generated code and infrastructure.

Each agent operates within a dedicated domain. Infrastructure controls deployment templates and configuration files, Security manages cryptographic services and rotation procedures, Quality owns anomaly testing, Platform governs APIs and audit interfaces, and Chain manages event creation and simulation logic.

This separation reduces merge conflicts, improves traceability, and models how responsibilities are divided in production engineering organizations.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/efd78660-3d95-4d31-9398-cc12c15f2a36_p9thb4d5)

## Deploying the Full Serverless Stack

### Merging agent worktrees and deploying to AWS

This phase combines all parallel workstreams into a single deployable system.

The agent worktrees are reviewed, merged into the main branch, and deployed through AWS SAM and CloudFormation. The deployment provisions all required infrastructure automatically, allowing the stack to be recreated consistently from source-controlled templates.

Smoke tests are executed after deployment to validate that every Lambda function and API endpoint responds correctly. This ensures that the infrastructure layer, execution layer, and routing layer all operate together before deeper validation begins.

### AWS resources provisioned by CloudFormation

The deployment creates a complete serverless architecture using managed AWS services.

A DynamoDB table stores signed supply chain events using chain_id and ledger_key identifiers. Four Lambda functions handle event emission, signing and verification, anomaly detection, and public auditing. API Gateway exposes public audit routes that allow external verification of chain integrity.

IAM roles are scoped with least-privilege access policies, limiting each function to only the permissions required for its task. This reduces blast radius if a function is compromised and aligns with standard DevSecOps principles.

The result is a fully serverless system where compute, storage, and routing scale independently without managing persistent infrastructure.

## Building the Infrastructure with Agent 1

### Directing the infrastructure agent

Agent 1 is responsible for defining the full infrastructure layer inside the SAM template.

The implementation includes networking, execution roles, encryption settings, logging, and compliance-oriented controls. The infrastructure is treated as a first-class artifact, meaning deployment behavior is versioned, reviewable, and reproducible alongside application code.

The template is validated before deployment to ensure resource references, permissions, and dependency relationships resolve correctly.

### Security controls for supply chain compliance

The platform includes dedicated controls to preserve auditability and chain integrity.

An encrypted S3 bucket stores audit evidence in a tamper-resistant format, ensuring that sensitive signing activity is retained for investigation and compliance review. Bucket policies restrict access so that only authorized AWS logging services can write evidence into the bucket.

CloudTrail is configured to capture KMS signing and verification operations tied specifically to the signing key used by the system. This creates a cryptographically traceable audit history showing exactly when signing operations occurred and which resources initiated them.

These controls transform the system from a simple event tracker into a verifiable compliance platform capable of producing defensible evidence during investigations or audits.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/efd78660-3d95-4d31-9398-cc12c15f2a36_cc84xh22)

### Lambda functions defined in the SAM template

The architecture is divided into four focused execution services.

EmitterFunction creates and stores chain events, SignerVerifierFunction handles cryptographic operations, AnomalyDetectorFunction validates chain integrity against detection rules, and AuditApiFunction exposes verification results externally through HTTP endpoints.

Separating responsibilities across functions improves scalability, isolates failures, and keeps security boundaries narrow. Each function operates independently but contributes to a shared verification pipeline.

## Simulating the 3-Party Supply Chain with Agent 2

### Building the chain emitter functions

This phase builds the execution flow that simulates a real supply chain lifecycle across three parties.

Agent 2 defines the ChainEvent schema and develops the emitter function responsible for creating, signing, and storing events in DynamoDB. A simulation script exercises the complete producer-to-shipper-to-retailer workflow, validating that the system can model custody transitions from origin to delivery.

Each event contains structured metadata describing who performed the handoff, when it occurred, and which previous event it references. This creates a linked chain where every handoff depends on the integrity of the prior step.

The result is not just event logging, but an immutable sequence that can later be audited for continuity and authenticity.

### Enforcing party ordering for chain integrity

The emitter enforces strict sequencing rules before allowing any event to be signed or stored.

Producer events are only allowed when a chain is empty. Shipper events require an existing producer event, and retailer events require both producer and shipper events in the correct order. If this sequence is violated, the system immediately returns HTTP 400 and blocks the write operation.

This prevents impossible timelines from entering the ledger. A retailer cannot receive goods that were never shipped, and a shipper cannot move goods that were never produced.

Without ordering enforcement, isolated events could appear valid even when critical custody transitions are missing. That gap is exactly what allows counterfeit products, diverted shipments, or undocumented transfers to move through a supply chain undetected.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/efd78660-3d95-4d31-9398-cc12c15f2a36_c9moibxm)

### Injecting supply chain failures for testing

The platform intentionally simulates broken supply chain conditions to validate detection logic.

The skip-shipper scenario removes the transit handoff entirely and attempts to jump directly from producer to retailer. This models undocumented movement where custody records are missing between parties.

The tamper-timestamp scenario manipulates the retailer timestamp so it appears earlier than the shipper event. This represents forged timing data, bad synchronization, or deliberate backdating intended to hide when an event actually occurred.

These tests validate whether the system can detect integrity failures rather than simply process happy-path transactions.

## Cryptographic Signing with AWS KMS via Agent 3

### Building the KMS signing harness

This phase introduces cryptographic trust into the system.

Agent 3 develops the signing and verification layer using AWS KMS-backed ECDSA signatures. Every supply chain event is hashed, signed, and stored with a corresponding cryptographic signature. Verification functions later recompute the hash and validate that the stored signature matches the event payload exactly.

Retry logic and structured logging are added to improve operational reliability and auditability. A key rotation runbook is also created to standardize how signing keys are replaced without disrupting the integrity chain.

This transforms the platform from a basic event tracker into a cryptographically verifiable ledger.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/efd78660-3d95-4d31-9398-cc12c15f2a36_esv5uaic)

### Why canonical JSON serialization is essential

Canonical serialization guarantees that identical payloads always produce identical hashes.

JSON objects do not enforce key ordering, meaning two logically identical payloads could serialize differently at the byte level. Since cryptographic signatures operate on raw bytes rather than logical meaning, inconsistent serialization would produce different hashes and invalidate signatures.

Canonical serialization solves this by enforcing sorted keys, compact formatting, and deterministic output rules. This ensures that every participant computes the same digest for the same payload regardless of runtime or implementation differences.

Without canonical formatting, verification becomes unreliable because identical data could appear tampered simply due to serialization inconsistencies.

### Key rotation runbook and proactive security

Key rotation is treated as a normal operational process rather than an emergency response.

The runbook documents how keys are replaced, validated, and rolled back if issues occur. This ensures that rotation procedures are repeatable and do not depend on institutional memory or a single engineer.

Routine rotation is also an expected requirement in many compliance frameworks. Keys that remain active indefinitely increase long-term exposure risk and complicate incident response if compromise occurs.

By documenting the process in advance, the platform ensures that cryptographic hygiene remains operationally manageable even as the system scales.

## Detecting Anomalies and Chain Breaks with Agent 4

### Building the anomaly detection engine

The anomaly detection engine evaluates chain integrity after events are stored.

Agent 4 defines six detection rules focused on identifying broken custody chains, invalid signatures, missing links, and manipulated timestamps. The anomaly detector retrieves all chain events from DynamoDB and executes each rule against the reconstructed sequence.

This creates a second validation layer that operates independently of event creation. Even if malformed data somehow enters the ledger, the anomaly engine can still detect and flag integrity failures during audit operations.

### Real-world failures each detection rule catches

The detection rules model realistic operational failures.

The missing previous_event_id rule verifies that every event correctly references the prior event in sequence. If a retailer event references the wrong shipment or skips a handoff entirely, the chain becomes inconsistent and the anomaly is flagged.

This protects against scenarios where goods are diverted, substituted, or inserted into unauthorized distribution channels while still attempting to appear legitimate.

Other rules detect timestamp tampering, missing signatures, and malformed event relationships that would otherwise undermine trust in the chain-of-custody record.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/efd78660-3d95-4d31-9398-cc12c15f2a36_4ntf7bvn)

### Test coverage strategy and live verification

The validation strategy separates unit-level logic from cryptographic integration.

Most anomaly rules are validated through pytest using mocked DynamoDB interactions. Signature verification tests are partially excluded because production verification ultimately depends on AWS KMS behavior rather than local-only hashing logic.

The current implementation already includes targeted tests for tampered payloads and missing signatures, while future integration tests are planned to validate end-to-end KMS-backed verification against deployed infrastructure.

This layered testing approach balances speed during development with confidence in production behavior.

## Building the Public Audit API with Agent 5

### Exposing chain verification to the public

This phase exposes the verification layer through public-facing APIs.

Agent 5 develops the audit Lambda function and connects it to API Gateway routes that allow external consumers to inspect and validate supply chain records. The API supports querying chain history as well as running integrity verification checks against stored events.

Input validation, HTTP status handling, and CORS configuration are implemented to ensure that the API behaves predictably for external systems and browser-based consumers. The API is also documented through an OpenAPI specification, making the verification interface understandable and reusable for downstream partners.

This transforms the platform from an internal tracking system into a publicly verifiable audit service.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/efd78660-3d95-4d31-9398-cc12c15f2a36_hmsdj7mx)

### Semantic HTTP status codes for chain integrity

The API distinguishes between malformed requests and integrity failures.

HTTP 400 is reserved for invalid requests, such as malformed parameters or missing identifiers. HTTP 422 is used when the request itself is valid but the retrieved chain fails semantic integrity checks.

This distinction is important because a corrupted chain is not the same as an invalid API call. The request succeeds technically, but the underlying resource violates business and compliance rules.

Using semantic status codes improves operational clarity and makes integrations easier to debug because consumers can distinguish between client-side errors and integrity violations within the supply chain itself.

## Generating Leadership Effectiveness Artifacts

### Creating portfolio-ready documentation

The system produces supporting artifacts that translate technical implementation into operational and executive visibility.

An interactive HTML workflow dashboard visualizes the architecture, event flow, and parallel agent structure. A KPI dashboard presents benchmark data and operational metrics in a format designed for leadership review. Agent prompt documentation captures ownership boundaries, acceptance criteria, and execution responsibilities so the workflow can be repeated consistently.

These artifacts extend the project beyond infrastructure and into operational communication, making the system understandable across both technical and non-technical audiences.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/efd78660-3d95-4d31-9398-cc12c15f2a36_qoxawfkw)

### What each artifact communicates to stakeholders

Each artifact targets a different layer of organizational visibility.

The workflow dashboard explains how the system is assembled and how events move through the architecture. The KPI dashboard reframes technical implementation into measurable outcomes and benchmark comparisons that leadership teams can evaluate quickly.

The agent prompts document operational workflow and governance, allowing future contributors to reproduce the same development structure without relying on undocumented tribal knowledge.

Together, these artifacts demonstrate that the platform is not only functional but operationally explainable.

## Running 4 Crisis-Inspired End-to-End Scenarios

### Full system verification through real-world scenarios

This phase validates the system under realistic failure conditions.

Four crisis-inspired scenarios are executed through the fully deployed stack. These scenarios test event ordering, timestamp integrity, audit verification, and anomaly detection while simultaneously validating logging and observability through CloudTrail and CloudWatch.

The verification process also includes a 14-item artifact checklist to confirm that infrastructure, APIs, logging, and anomaly rules behave as expected under each scenario.

This step proves that the system can move beyond isolated unit behavior and operate correctly under coordinated stress conditions.

### Ordering enforcement and HTTP response validation

One of the most important scenarios validates custody ordering enforcement.

The pharma skip-shipper scenario attempts to move directly from producer to retailer without a valid shipper handoff. The emitter rejects the operation and responds with HTTP 400, proving that the platform blocks impossible supply chain transitions before they are written to the ledger.

Other scenarios validate timestamp tampering and public audit operations, ensuring that failures are detected consistently regardless of where they occur in the pipeline.

These tests confirm that integrity validation exists both during event creation and during post-event auditing.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/efd78660-3d95-4d31-9398-cc12c15f2a36_j9jmhkfy)

## 💎 Secret Mission: SLSA L1 Compliance Dashboard

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/efd78660-3d95-4d31-9398-cc12c15f2a36_fb3a0j0i)

### How in-toto attestations prove supply chain compliance

In-toto attestations provide machine-verifiable evidence of system integrity.

The attestation binds chain events to SHA-256 digests so any external verifier can independently recompute hashes and confirm that the underlying data matches the reported evidence. The schema also includes structured metadata describing anomaly checks, integrity scores, and compliance mappings.

Because the attestation process is tied to KMS operations and CloudTrail logging, every signing action becomes traceable. This creates a reproducible audit trail rather than a static report or screenshot.

The result is portable compliance evidence that external partners can validate independently without needing direct trust in the originating system.

## Reflections and Key Takeaways

### Tools and concepts mastered

This project uses AWS SAM CLI, Python, AWS CLI, DynamoDB, Lambda, API Gateway, AWS KMS, and Cursor Agents to build a cryptographically verifiable supply chain platform.

The core concepts include Infrastructure as Code with SAM templates, serverless application architecture, cryptographic signing with AWS KMS using ECDSA, anomaly detection, DevSecOps governance, SLSA L1 provenance, and parallelized development using isolated Cursor agent worktrees.

The platform combines these components into a unified verification pipeline where infrastructure, signing, auditing, and anomaly detection operate together rather than as isolated features.

### Time investment and challenges

This project took approximately 3 hours.

The most challenging part was debugging the SAM template and ensuring that CloudFormation resource references, IAM permissions, and YAML indentation aligned correctly across the deployment stack. Because serverless resources are tightly interconnected, a small configuration error could cascade into failed deployments or broken runtime behavior.

Troubleshooting also required validating that Lambda permissions, API Gateway routes, and DynamoDB integrations resolved correctly after deployment.

This project demonstrates the ability to build and deploy a secure serverless system that validates supply chain integrity through cryptographic evidence rather than trust alone.

The system proves that every handoff can be signed, audited, and independently verified while still operating within a scalable cloud-native architecture. It also demonstrates how DevSecOps principles can be embedded directly into infrastructure, deployment workflows, and operational governance from the beginning of a project rather than added later.

The combination of anomaly detection, audit APIs, CloudTrail logging, and cryptographic signing creates a defensible chain-of-custody model capable of supporting compliance-oriented workflows.

The next step is expanding this architecture into containerized and orchestrated environments using Docker and Kubernetes.


---

*Built with [NextWork](https://learn.nextwork.org) - [View this project](https://learn.nextwork.org/projects/efd78660-3d95-4d31-9398-cc12c15f2a36)*
