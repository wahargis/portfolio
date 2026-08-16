# 09 — Evaluation, Testing, and Operations

Flip treats quality as a continuous property of the system, not a final checklist. Changes pass automated gates, deployments are verified against the live environment, and runtime behavior is observable through health, audit, and recovery paths.

## Quality gates

The server and client each have automated checks that run before changes reach production. These include unit and integration tests, compilation and formatting checks, dependency and security advisories, accessibility checks, and end-to-end coverage of the main web, desktop, and mobile surfaces. The intent is to catch regressions before deploy without publishing the internal test inventory.

## Deployment verification

Deployments are serialized and checked for drift against the live environment after release. The public property is that a deployed change is confirmed to be present and healthy on the intended host, not the specific script used to do it.

## Runtime operations

At runtime, Flip exposes health and version signals, records AI and administrative activity for audit, and stages generated media through recovery paths before final placement. This gives operators three verification points: before deploy, during deploy, and after deploy.
