# 03 — Agent Runtime

Flip’s AI work runs in a bounded agent runtime rather than in unbounded model loops, with explicit ceilings on rounds, time, tokens, and tool execution.

<img src="../diagrams/agent-execution-sequence.svg" alt="Agent execution sequence" width="760" />

## Bounded tool loop

The runtime uses a two-phase loop. A gather phase lets the model call read and analysis tools; a terminal phase requires a plan or final response. The loop enforces a per-run round cap, a wall-clock deadline checked between rounds, and an input-token envelope so the model’s context remains predictable. When a cap is reached, the run forces a terminal submission and marks the finish-timeout as non-retryable, preventing side-effectful tools from being replayed by retries.

## Isolated dispatch

Each tool call runs in its own isolated unit with a per-tool timeout. A crash or timeout in one tool cannot bring down the worker or the rest of the run. The runtime also enforces an advertised-tool allowlist: only the tools the model was told about in that turn can actually be dispatched, independent of the larger tool catalog.

## Worker-level bounds

The async worker adds another layer of control: a deadline, round caps, backoff, and a rescue mechanism for runs that are stuck beyond the normal deadline. The runtime is supervised so queued AI work can always dispatch safely, and synthesis capabilities are feature-gated per deployment.

## Practical result

AI participants can use tools, but every run has a ceiling, and a single bad tool cannot take down the worker.
