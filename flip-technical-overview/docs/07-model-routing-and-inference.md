# 07 — Model Routing and Inference

Model access is routed, not hardcoded. `Flip.Synthesis.LlmClient` speaks an OpenAI-compatible endpoint and handles reasoning modes for DeepSeek, OpenRouter, and other providers. Provider keys are stored encrypted via `Flip.Settings.LlmProviderKey` for `openrouter`, `deepseek`, and `other` provider types.

`Flip.Settings.AppSettings` holds the selected `llm_provider`, `llm_provider_key_id`, and a reasoning parameter map. `Flip.Settings.ProviderMigration` supports migrating settings between providers. This allows per-deployment or per-room model choice without code changes; room-level `synthesis_config` can select a model within the configured provider boundary.

Video generation has its own deterministic router: `Flip.Videogen.Router` selects a video provider based on configured capabilities, and `Flip.Videogen.ProviderCapabilities` describes those capabilities. The router is deterministic, so the same request and configuration map to the same provider path.

Inference is auditable. `Flip.LLM.CallAudit`, `Flip.LLM.Activity`, and `Flip.LLM.Activities` record LLM activity, and telemetry bridges operational AI events into admin-visible audit rows. The LLM client installs a fuse/breaker pattern in the application, so repeated transport or server errors can open the circuit rather than accumulating retries.

Encrypted keys, provider migration, per-room model selection, deterministic video routing, and LLM call audit together form the inference boundary. Prompt assembly and persona rendering are product internals; this overview covers the routing and control mechanisms only.

<img src="../diagrams/model-routing-audit.svg" alt="Model routing and call audit" width="760" />
