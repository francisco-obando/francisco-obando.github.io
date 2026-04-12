---
title: "AI-Driven E2E Testing: Mapping Blast Radius in Microservices"
date: 2026-04-11T12:00:00Z
tags: ["AI", "E2E Testing", "Microservices", "Quality Engineering", "Platform Health"]
categories: ["Projects"]
summary: "Building a centralized E2E testing source of truth using AI agents to map dependencies and prioritize tests for QA-gated production releases."
---

## The Microservices "Blast Radius" Problem

In a platform composed of dozens of web apps and microservices, service-level testing (unit and functional) is a prerequisite, but it isn't a guarantee of platform health. We found that while individual services passed their local CI/CD gates, production releases often caused unforeseen "blast radius" effects—degrading the health of seemingly unrelated parts of the platform.

To solve this, we initiated a project to create a centralized **Source of Truth** repository for end-to-end (E2E) testing, designed to validate the entire platform's integrity before any service reaches production.

---

## AI-Driven Dependency Mapping

The core innovation of this project lies in the use of **AI agents** to manage the complexity of our architecture. Traditional E2E suites often grow stale because they can't keep up with the pace of service changes.

We deployed agents to:
1.  **Scrape Code & Infrastructure**: Agents scan service repositories and Infrastructure-as-Code (IaC) definitions to identify how services interact.
2.  **Identify Hidden Dependencies**: By analyzing API contracts and event-bus topics, the agents map out which services are truly critical to specific business flows.
3.  **Automated Test Generation**: Using this dependency map, the agents help identify gaps in our E2E coverage that humans might overlook.

---

## A Tiered Priority Framework

With a platform of our size, running every E2E test for every small service update is inefficient. We used our AI-generated dependency map to categorize tests into four priority tiers:

| Priority | Category | Impact |
| :--- | :--- | :--- |
| **P0** | **Absolutely Breaking** | System-wide outages or critical failures of core infrastructure. |
| **P1** | **Business Logic** | Failures in primary user journeys (e.g., checkout, authentication). |
| **P2** | **Service Degradation** | Issues affecting latency, throughput, or partial feature unavailability. |
| **P3** | **Isolated Failures** | Non-critical UI bugs or background tasks with no immediate user impact. |

---

## Enabling QA-Gated Production Releases

The ultimate goal of this repository is to move from "testing as a checkmark" to **QA-gated production releases**. 

By integrating this centralized suite into our global deployment pipeline, we can ensure that:
- Every release candidate is validated against the **P0 and P1** tiers of the entire platform.
- Service owners have immediate visibility into the blast radius of their changes.
- We significantly reduce the number of high-severity incidents that impact our users.

This project represents a shift toward a **Platform-First** mentality, where the health of the entire ecosystem is the primary metric for deployment success.
