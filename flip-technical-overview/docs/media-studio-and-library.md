# Media commissions, Studio, and Library

Image and video generation are long-running product workflows. Flip stores a commission and a library projection before provider execution so the user can leave the page, open another client, or return after a worker restart.

## Commission submission

A media request can come from a direct creation interface or a Personal AI tool call. The server validates the requester, requested operation, source media ownership, provider policy, and usage rules. It then creates:

- A commission record.
- A generation job with a stable identifier.
- A pending library item linked to the commission.
- The relationship to the requesting message or Personal AI reply, when present.

The accepted response means the request has durable state. It does not claim that provider generation has completed.

## Provider execution

Image and video workers load the stored commission and choose a provider route. The worker records attempt state before it calls the provider. Provider-specific responses are normalized into application states and asset metadata.

The job can move through accepted, queued, running, completed, failed, or cancelled states. A retry continues the same commission. Provider polling and callback handling update the stored job instead of relying on one open HTTP request.

## Asset completion

On success, Flip:

1. Stores the generated file in object storage.
2. Writes media type, dimensions, duration, provider, model, prompt and source relationships required by the product.
3. Changes the pending library item to a ready asset.
4. Updates the requesting Personal AI reply or creation view.
5. Publishes durable and transient updates to connected clients.
6. Finalizes the usage records associated with the job.

A terminal failure changes the pending item and reply to a visible failed state. The system does not leave an indefinite loading placeholder as the only record.

## Library

The Library is the user's durable media workspace. It supports:

- Images, videos, uploaded media, and generated derivatives.
- Search over stored metadata.
- Folder assignment.
- Favorites.
- Keyset pagination for long collections.
- Pending, ready, failed, and cancelled states.
- Opening an asset in a detailed preview or Studio.
- Links back to generation metadata and source assets.

A pending item appears in the same collection where the completed asset will remain. Reconciliation can repair a projection after a worker or client interruption.

## Studio

Studio opens a selected image as the main visual surface. Editing controls use compact panels so the image remains central on desktop and mobile layouts. Current workflows include:

- Text and sticker insertion.
- Selection and transform operations.
- Layer ordering and visibility.
- Crop and canvas changes.
- Mask creation and mask-aware inpainting.
- Edit-in-place actions from a Personal AI or media reply.
- Saving an edited result as a new asset with a source relationship.

The original asset remains available. Derived assets carry enough metadata to show how they were created.

## Failure and recovery

Media reconciliation checks commissions, provider attempts, object-storage results, and library projections. It can repair a job that completed externally but did not finish local projection work.

Common recovery paths include:

- Retry a transient provider failure with the same commission identifier.
- Poll a provider job after a process restart.
- Mark a terminal provider result and update the pending library item.
- Stop work after cancellation.
- Rebuild a missing library projection from commission and asset records.
- Resume the requesting reply after the media artifact becomes available.
