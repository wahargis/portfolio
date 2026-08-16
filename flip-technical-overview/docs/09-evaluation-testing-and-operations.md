# 09 — Evaluation, Testing, and Operations

Flip is exercised by automated test suites on both the server and the client before changes reach production.

The server suite spans unit and integration tests, deploy-integrity shell suites, dependency advisory checks, formatting checks, warnings-as-errors compilation, asset bundling, and the full test suite against a PostgreSQL 16 service container with logical WAL. The client suite spans unit and component tests, Rust tests in the Tauri layer, token-compliance scanning, accessibility audits, integration tests, and Playwright/WDIO/Appium end-to-end suites across web, desktop, and mobile targets.

Operations tooling includes `bin/check-deploy-drift` and `bin/deploy-history` for deploy verification, and deployments run drift checks against the live `/api/version` endpoint. Audit paths include `Flip.AuditLog` for admin and operational AI events, `Flip.LLM.CallAudit` for LLM activity, and recovery spools (`Flip.Media.RecoverySpool`, `Flip.Media.GeneratedRecovery`, `Flip.Synthesis.ImageRecovery`) that save generated media first so interrupted work can be recovered.

Together these practices make the system verifiable in three places: before deploy (automated gates), during deploy (drift checks), and at runtime (health endpoints, audit logs, and recovery spools).
