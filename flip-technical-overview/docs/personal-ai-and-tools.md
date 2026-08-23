# Personal AI replies and tool execution

Personal AI replies are separate records attached to individual source messages. This allows several messages to have independent queue, execution, completion, and failure state.

## Reply creation

When a message is eligible for a Personal AI response, the application creates a reply record with the source message, user, conversation, and current execution state. It then places the reply in the Personal AI queue.

The queue entry refers to the stored reply. The worker does not use an anonymous in-memory request as the only record of work.

## Request construction

The worker loads the current server state required for the reply:

- Source message and conversation history.
- User-owned memory or context that the user has enabled.
- Persona and model settings.
- Available provider routes.
- Tool definitions allowed for the user and request.
- Usage or credit policy required before provider-backed work.
- Existing reply state, attempt state, and cancellation state.

Context is selected for the current reply. It is not a copy of every record the application can access.

## Model and provider routing

The runtime selects a model route from configured local or hosted providers. Routes carry health and failure state so repeated provider errors can open a circuit breaker. Credentials remain server-side and are checked for readability before work is dispatched.

A provider response can contain normal text, structured tool calls, or a request to continue after tool results. The application records enough state to update the same reply across these steps.

## Tool execution

Tool calls cross a server boundary before they run. The handler verifies:

- The Personal AI reply that requested the call.
- The authenticated user and target resource.
- The tool name and argument schema.
- Ownership or membership requirements.
- Limits on media, search, storage, or external provider use.
- Cancellation and duplicate-execution state.

The result is returned to the model loop and can also create durable product records. A media tool, for example, creates a commission and pending library item before the external generation job starts.

## Progress and completion

The reply record moves through explicit states such as queued, working, completed, failed, or cancelled. Progress events can update the interface while the durable state remains in PostgreSQL.

Completion writes the final response and any linked artifacts to the reply. The source message remains unchanged.

## Recovery

A reconciliation process scans for replies that are queued or marked as working without active execution. It can return the reply to the queue using the same stored identifier. Stable reply and job identifiers prevent the recovery path from creating an unrelated second response.

Provider failures are classified before retry:

- A transient failure can defer or retry the same reply.
- A provider or credential outage can move the route into a cooldown state.
- A terminal failure writes a user-visible result and stops automatic execution.
- Cancellation prevents further tool or provider work for the reply.

This design allows the Personal AI queue to recover independently of the client connection that created the message.
