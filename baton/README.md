# Baton

Baton is a control plane for **using full coding agents as powerful but untrusted workers**.

The problem it addresses is easy to underestimate. Modern coding agents can plan, edit, test, search, and operate for long stretches, but the moment they are allowed to work autonomously another systems problem appears: who owns their lifecycle, what exactly are they allowed to change, how do you know whether they are still making progress, how do you interrupt them safely, and why should you trust their claim that the result is correct?

Baton exists to answer those questions in ordinary deterministic software rather than by asking one model to supervise another through prose.

![Baton control loop](assets/control-loop.svg)

## The central design decision

Baton separates **reasoning authority** from **execution authority**.

The orchestrating model can decide how to decompose work, which worker should attempt it, whether a result looks semantically promising, and when more investigation is needed. The coordinator owns the things that should not be probabilistic: process identity, source scope, command ordering, worktree ownership, interruption, recovery, result capture, and the gate between a candidate patch and adopted source.

That distinction is the project.

Without it, an “agent orchestration system” often reduces to a collection of subprocesses whose own status messages are treated as ground truth. A worker can say it stopped when a child process is still running; say tests passed in a dirty worktree; answer a stale steering message after being replaced; or report completion without preserving enough evidence to reproduce the result. Baton treats those as control-plane failures, not prompt-engineering problems.

## A run as a controlled transaction

A Baton run begins with an explicit assignment and source scope. The worker receives an isolated worktree and a concrete route to a full-session harness. From that point onward, the coordinator keeps durable control state independently of the worker’s internal conversation.

The important transition occurs at completion: the worker’s output is treated as a **candidate**, not as accepted truth. Baton pins the result to immutable source state and verifies it again in a fresh environment before adoption. Review and integration remain separate decisions.

```text
intent
  -> controlled worker execution
  -> immutable candidate result
  -> independent verification
  -> review / comparison
  -> explicit adoption or integration
```

This gives the orchestrator freedom to use strong agents aggressively without allowing their confidence to become source-control authority.

## Multi-agent work without losing control

Baton can coordinate multiple workers as a wave or declarative workflow, but the purpose is not to maximize the number of simultaneous agents. The value is that parallel or staged work still has one coherent control model.

Each worker has an exact identity, route, scope, and result. Dependencies and steering rules can be declared centrally. If one member fails, its failure does not erase successful work from the rest of the wave. If a worker is replaced, stale commands cannot silently target the replacement. If the orchestrator disconnects, the resident coordinator retains enough durable state to reattach rather than pretending the session never existed.

That makes multi-agent execution closer to a distributed systems problem than a group-chat problem.

## Liveness and steering

A long model turn creates an awkward ambiguity: silence may mean useful reasoning, a hung process, a provider stall, or a blocked interaction. Baton therefore does not infer liveness from wall-clock delay alone.

The coordinator tracks explicit evidence of progress and distinguishes a running turn from a worker that has actually lost admissible liveness. Steering follows the same principle. A blocking question is different from a conversational message, and an interrupt is different from a stop. Those distinctions are represented in control state so that user or orchestrator actions have predictable semantics.

The goal is not to micromanage model cognition. It is to ensure that intervention has a well-defined target and effect.

## Verification as part of orchestration

Baton’s strongest architectural claim is that verification belongs inside the orchestration lifecycle rather than after it.

A coding agent can accidentally validate itself against state that only exists in its own worktree. It can modify tests, rely on untracked files, or simply misreport what it ran. By checking the pinned candidate in a fresh worktree, Baton asks a different question: *does this result still hold when the worker is no longer in control of the environment?*

Depending on the workflow, that verification can include ordinary build and test commands, targeted regression evidence, or an independent semantic review. The important point is not an ever-growing checklist. It is that adoption depends on externally reproducible evidence rather than worker self-report.

## Why this is not another coding agent

Baton intentionally does not try to become the best planner, coder, reviewer, or researcher itself. Those capabilities evolve rapidly in the underlying harnesses and models. Baton is the stable layer around them.

That makes adapter capability important. If one harness supports resumable sessions and another does not, the control plane should expose that difference rather than inventing a fake common denominator. The coordinator can emulate some behavior, but it must know when an invariant is native, approximated, or unavailable.

## Current maturity

The current implementation covers the core lifecycle, isolated worktrees, multi-worker execution, durable coordinator state, steering and interaction, recovery, verification, and the separation between candidate, reviewed, adopted, and integrated source state. Ongoing work is primarily about strengthening adapter parity, nested orchestration, and diagnostic quality—not expanding a list of agent “features.”

The implementation repository is private; this page documents the architecture and trust model rather than internal provider configuration or campaign history.
