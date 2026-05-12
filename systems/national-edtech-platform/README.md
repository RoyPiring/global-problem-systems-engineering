# Build a National EdTech Platform

> Inside the [Global Problem Systems Engineering](../../README.md) portfolio · *Population-scale systems built for civic and public-good outcomes.*

## Overview

This project builds a lab-scale national education platform modeled after India's DIKSHA architecture.

The system orchestrates 15 Docker containers into a federated platform that includes an LMS, live video classrooms, video-on-demand streaming, single sign-on, edge caching, and real-time observability. All services are exposed through a single entry point, simulating how a national-scale education system would operate under unified access. The goal is not just feature parity, but demonstrating how distributed systems integrate into a cohesive platform with identity, traffic routing, and monitoring built in from the start.

The architecture is built across **9 phases**, anchored by **Setting Up the Environment and Scaffolding the Platform** on the input side and **Multi-State Identity Federation and Custom Monitoring** at the end. Each phase is listed in the Implementation section below.

## Architecture

```mermaid
---
title: Build a National EdTech Platform
---
%%{init: {"theme":"base","themeVariables": {"primaryColor":"#1B4332","primaryTextColor":"#F4D03F","primaryBorderColor":"#F4D03F","secondaryColor":"#264653","tertiaryColor":"#2F5233","lineColor":"#F4D03F","fontFamily":"ui-monospace, SFMono-Regular, Menlo, Consolas, monospace","fontSize":"13px"}}}%%
flowchart LR
    classDef datastore fill:#264653,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef service fill:#1B4332,stroke:#F4D03F,stroke-width:2px,color:#F4D03F
    classDef event fill:#7B42BC,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef io fill:#0d1117,stroke:#F4D03F,stroke-width:1.5px,color:#F4D03F,font-style:italic







    subgraph Users["Population Users"]
        Students[/Students/]
        Teachers[/Teachers/]
        Admins[/Admins/]
    end

    subgraph Edge["Regional Edge"]
        NGINX(NGINX Reverse Proxy + Cache)
    end

    subgraph Identity["Identity Federation"]
        Keycloak(Keycloak Realm SSO)
    end

    subgraph Content["Content Delivery"]
        Moodle(Moodle LMS)
        MediaCMS(MediaCMS HLS Streaming)
        Jitsi(Jitsi Live Classrooms)
    end

    subgraph Assessment["Assessment & Sessions"]
        Jicofo(Jicofo + JVB + XMPP)
    end

    subgraph Analytics["Central Analytics"]
        Prometheus(Prometheus Scraper)
        cAdvisor(cAdvisor Container Metrics)
        Grafana(Grafana Dashboards)
        MetricsDB[(TSDB Metrics Store)]
    end

    Students -->|requests at port 80| NGINX
    Teachers -->|requests at port 80| NGINX
    Admins -->|admin access| NGINX

    NGINX -->|authenticates via OIDC| Keycloak
    NGINX -->|routes /lms| Moodle
    NGINX -->|delivers cached HLS| MediaCMS
    NGINX -->|proxies WebSocket| Jitsi

    Keycloak -->|federates identity| Moodle
    Moodle -->|embeds session| Jitsi
    Jitsi -->|signals via XMPP| Jicofo

    Moodle -.->|exposes metrics| Prometheus
    Jitsi -.->|exposes metrics| Prometheus
    MediaCMS -.->|exposes metrics| Prometheus
    cAdvisor -->|aggregates container stats| Prometheus
    Prometheus -->|writes samples| MetricsDB
    MetricsDB -->|queries dashboards| Grafana
    Admins -->|monitors| Grafana
class MetricsDB datastore
class Students,Teachers,Admins io

    class MetricsDB datastore
    class NGINX,Keycloak,Moodle,MediaCMS,Jitsi,Jicofo,Prometheus,cAdvisor,Grafana service
    class Students,Teachers,Admins io
```

The diagram shows the topology and data flow of the system as built. The full architectural narrative, with screenshots and prose, lives in [`documents/national-edtech-platform.md`](./documents/national-edtech-platform.md).

## Implementation

This system is built across **9 phases**:

1. **Setting Up the Environment and Scaffolding the Platform**
2. **Orchestrating 15 Containers with Docker Compose and NGINX**
3. **Configuring Keycloak Identity Federation**
4. **Wiring Moodle to Keycloak SSO and Embedding Live Classrooms**
5. **Building a Course with Live Video and On-Demand Content**
6. **Deploying Real-Time Observability with Prometheus, Grafana, and cAdvisor**
7. **Adding Edge Caching and Benchmarking Platform Performance**
8. **Documenting the Architecture and Evaluating Against DIKSHA**
9. **Multi-State Identity Federation and Custom Monitoring**

For the full walkthrough with screenshots and step-by-step content, see [`documents/national-edtech-platform.md`](./documents/national-edtech-platform.md).

## Validation

Build outcomes verified end-to-end. Each phase below is captured with screenshots, configuration, and observable behavior in [`documents/national-edtech-platform.md`](./documents/national-edtech-platform.md):

- ✅ Setting Up the Environment and Scaffolding the Platform
- ✅ Orchestrating 15 Containers with Docker Compose and NGINX
- ✅ Configuring Keycloak Identity Federation
- ✅ Wiring Moodle to Keycloak SSO and Embedding Live Classrooms
- ✅ Building a Course with Live Video and On-Demand Content
- ✅ Deploying Real-Time Observability with Prometheus, Grafana, and cAdvisor
- ✅ Adding Edge Caching and Benchmarking Platform Performance
- ✅ Documenting the Architecture and Evaluating Against DIKSHA
- ✅ Multi-State Identity Federation and Custom Monitoring
