---
nextwork_uuid: 20b03138-04b4-4943-a367-1034a8c6c5a1
original_filename: legendary-20b03138-04b4-4943-a367-1034a8c6c5a1.md
migrated_to: global-problem-systems-engineering/national-edtech-platform.md
migrated_at: 2026-05-04
schema: nextwork-generator
---

<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Build a National EdTech Platform

**Project Link:** [View Project](https://learn.nextwork.org/projects/20b03138-04b4-4943-a367-1034a8c6c5a1)

**Author:** Roy Piring Jr  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/20b03138-04b4-4943-a367-1034a8c6c5a1_jy5hhm4d)

## What I Built and Why It Matters

### Project vision and goals

This project builds a lab-scale national education platform modeled after India’s DIKSHA architecture.

The system orchestrates 15 Docker containers into a federated platform that includes an LMS, live video classrooms, video-on-demand streaming, single sign-on, edge caching, and real-time observability. All services are exposed through a single entry point, simulating how a national-scale education system would operate under unified access. The goal is not just feature parity, but demonstrating how distributed systems integrate into a cohesive platform with identity, traffic routing, and monitoring built in from the start.

## Setting Up the Environment and Scaffolding the Platform

### Step goals and approach

The environment setup establishes the execution layer for the entire platform.

All required tools are verified upfront, including Docker Desktop, Docker Compose, Git, and Node.js, ensuring compatibility before orchestration begins. The project structure is scaffolded to isolate each service, allowing independent configuration while maintaining a shared deployment context. Jitsi is downloaded and configured as a prebuilt subsystem, requiring secure credential initialization before it can participate in the broader stack.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/20b03138-04b4-4943-a367-1034a8c6c5a1_hrui80tg)

### Understanding the Jitsi configuration

The Jitsi configuration defines how internal communication is secured.

The .env file contains runtime parameters such as ports, public URLs, and feature flags, along with credentials required for internal service authentication. Components like XMPP, Jicofo, and JVB depend on these credentials to establish trust within the system. The stack refuses to start without valid values, enforcing security at initialization rather than runtime. This mirrors production systems where service-to-service authentication is mandatory before availability.

## Orchestrating 15 Containers with Docker Compose and NGINX

### Step goals and approach

The platform is composed using a single Docker Compose definition that describes all services.

Each container is configured with its image, environment variables, ports, volumes, and dependencies. NGINX is introduced as the edge layer, acting as the single entry point and reverse proxy for all traffic. The system is launched as a unified stack, where all services initialize together and establish interdependencies dynamically.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/20b03138-04b4-4943-a367-1034a8c6c5a1_7jvcfse1)

### How NGINX routes traffic across services

NGINX operates as the routing layer for the entire platform.

All external requests terminate at port 80, where NGINX inspects the host and path before forwarding traffic to the appropriate backend service. This abstraction allows multiple services to exist behind a single endpoint, simplifying access while maintaining separation internally. WebSocket support ensures real-time services such as Jitsi function correctly within this routing model.

## Configuring Keycloak Identity Federation

### Step goals and approach

Identity is centralized using Keycloak to enable single sign-on across services.

A dedicated realm is created to isolate application users from administrative control. Roles such as teacher and student define access boundaries, while Moodle is registered as an OpenID Connect client. The discovery endpoint validates that identity metadata is correctly exposed, ensuring compatibility between the identity provider and consuming services.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/20b03138-04b4-4943-a367-1034a8c6c5a1_j288jegz)

### Why a dedicated realm matters

A dedicated realm separates operational control from application usage.

The master realm is reserved for managing the identity system itself, while the application realm contains users, roles, and policies specific to the platform. This separation reduces risk and allows independent configuration of authentication rules, password policies, and access control without affecting system-level administration. It also enables scaling across environments such as staging and production.

## Wiring Moodle to Keycloak SSO and Embedding Live Classrooms

### Step goals and approach

This step integrates identity with application access and real-time communication.

Moodle is configured to use Keycloak through OAuth 2, establishing a single sign-on flow where authentication is handled centrally. The Jitsi plugin is installed to embed live classrooms directly inside course modules. The full login flow is validated by authenticating through Keycloak and landing inside Moodle with the correct role context applied.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/20b03138-04b4-4943-a367-1034a8c6c5a1_exg45mz2)

### Docker networking and internal hostnames

Service communication relies on Docker’s internal DNS.

Containers reference each other using service names such as keycloak:8080 rather than localhost. Localhost inside a container refers only to itself, so using it would break service-to-service communication. Public endpoints are still exposed through NGINX, but internal calls must use Docker network hostnames to function correctly.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/20b03138-04b4-4943-a367-1034a8c6c5a1_o5qrglyi)

## Building a Course with Live Video and On-Demand Content

### Step goals and approach

The platform is validated through actual content delivery workflows.

A Moodle course is created and a Jitsi session is embedded as a live classroom activity. Video content is uploaded to MediaCMS and served through HLS streaming, confirming adaptive playback. Architecture diagrams are generated to document how services interact, ensuring the system is not only functional but explainable.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/20b03138-04b4-4943-a367-1034a8c6c5a1_ccw5hc94)

### Benefits of native Jitsi integration in Moodle

Embedding Jitsi inside Moodle maintains context and control.

Sessions are tied directly to course enrollment, ensuring only authorized users can join. Identity and roles are carried into the session, improving moderation and participant tracking. Users remain within the LMS interface, eliminating the need to switch tools or manage external links.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/20b03138-04b4-4943-a367-1034a8c6c5a1_op5nk7dd)

## Deploying Real-Time Observability with Prometheus, Grafana, and cAdvisor

### Step goals and approach

Observability is introduced to monitor system behavior in real time.

Prometheus is configured to scrape metrics from containers, Grafana is provisioned with dashboards, and cAdvisor exposes container-level metrics. The system is stress-tested to verify that load changes are reflected in dashboards, confirming that telemetry is active and accurate.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/20b03138-04b4-4943-a367-1034a8c6c5a1_jy5hhm4d)

### What the monitoring stack reveals

The monitoring stack provides visibility into system performance.

CPU metrics show processing load per container, memory metrics reveal working set usage, and network metrics track ingress and egress traffic. This data enables real-time analysis of how the system behaves under load and identifies bottlenecks across services.

## Adding Edge Caching and Benchmarking Platform Performance

### Step goals and approach

Edge caching is introduced to reduce backend load and improve response times.

NGINX is configured to cache static assets and video streams, allowing frequently requested content to be served directly from cache. Validation tests are executed to measure performance, accessibility, and system health, producing a benchmark for platform behavior.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/20b03138-04b4-4943-a367-1034a8c6c5a1_xe50igwd)

### Understanding cache behavior through X-Cache-Status

Cache performance is evaluated through response headers.

The X-Cache-Status header indicates whether content was served from cache or fetched from the origin. A MISS shows that the request reached the backend, while a HIT confirms that cached content was used. Observed results show repeated MISS responses, indicating that caching rules require further tuning to achieve expected efficiency.

## Documenting the Architecture and Evaluating Against DIKSHA

### Step goals and approach

The platform is documented as a production-style system.

Architecture diagrams, component mappings, and monitoring explanations are created to describe how the system operates. A deployment scenario is modeled for a large-scale environment, evaluating the system against real-world criteria such as scalability, observability, and reliability.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/20b03138-04b4-4943-a367-1034a8c6c5a1_p7kcfgpe)

### Industry evaluation findings

Observability is the strongest component of the system.

The platform includes active Prometheus scraping, Grafana dashboards, and container-level metrics that respond to load changes. This demonstrates that telemetry is functional rather than simulated. However, gaps remain in areas such as centralized logging, distributed tracing, and alerting, which are required for full production readiness.

## Secret Mission: Multi-State Identity Federation and Custom Monitoring

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/20b03138-04b4-4943-a367-1034a8c6c5a1_54j2ia50)

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/20b03138-04b4-4943-a367-1034a8c6c5a1_l2zl7vcf)

### Production readiness scorecard highlights

The system demonstrates partial production readiness.

Observability scores highest due to the presence of a working metrics pipeline. Evidence includes configured scrape intervals, active dashboards, and real-time metric updates. Other areas, such as resilience and operational automation, remain incomplete and limit the overall readiness score.

## Reflections and Key Takeaways

### Tools and concepts mastered

This project used Docker Compose, Keycloak, Prometheus, Grafana, and Pa11y CI to build and validate a multi-service platform.

The core concepts include microservices architecture, OpenID Connect SSO, real-time monitoring, accessibility testing, and container orchestration. These components worked together to simulate how a distributed system is deployed, secured, and observed as a single platform.

### Time and challenges

This project took approximately 12 hours and 30 minutes to complete.

The most challenging aspect was troubleshooting the full end-to-end system. Individual components worked in isolation, but aligning networking, identity, routing, and monitoring across all services required precise configuration and iterative debugging

### Final reflections

This project focused on orchestrating a 15-container Docker Compose stack, configuring OpenID Connect SSO with Keycloak, and deploying a monitoring stack using Prometheus, Grafana, and cAdvisor.

The system demonstrates how multiple services integrate into a single platform with shared identity and observability. The next step is building deeper capability in cloud infrastructure as code using tools like Terraform or Ansible to move from local orchestration to scalable cloud environments.

---

*Built with [NextWork](https://learn.nextwork.org) - [View this project](https://learn.nextwork.org/projects/20b03138-04b4-4943-a367-1034a8c6c5a1)*
