> **AI-drafted technical reference.** This file covers the mechanics of Step 4's background-job setup (architecture, local/production run steps, reliability behavior, troubleshooting). Reviewed and verified — including a real end-to-end test against a live local Inngest dev server and the real OpenAI API — but drafted with AI assistance rather than hand-written. See [`Background-Jobs-Documentation.md`](./Background-Jobs-Documentation.md) in this folder for the human write-up.

Document AI extraction no longer runs inside the upload request. Upload validates the PDF, stores it, queues a job, and returns immediately. Inngest runs the extraction in the background and moves each document through `Pending → Processing → Completed` or `Failed`.

## Why Inngest

Required for this step per the task instructions. For this project's size, a separate Redis-backed worker would be more operational overhead than value — Inngest gives retries, a run history UI, and a free cloud control plane, while the actual job code still lives directly in the Next.js app at `/api/inngest`.

## Local run

1. Set `INNGEST_DEV=1` in `.env.local` (already done for local development).
2. Start Postgres, apply migrations, start the app: `docker compose up -d`, `npm run db:deploy`, `npm run dev`.
3. In a second terminal: `npx inngest-cli@latest dev -u http://localhost:3000/api/inngest`.
4. Open the Inngest Dev Server at `http://localhost:8288` — it should show `process-document` synced.
5. Upload a PDF on `/dashboard/documents`. The row shows Pending, then Processing, then Completed (or Failed) automatically — the page polls while anything is in flight.

**Verified directly** (not just by inspection): a real upload was pushed through this exact flow using the real Inngest dev server and the real OpenAI API, and the full `PENDING → PROCESSING → COMPLETED` transition was captured live in the database, with a correct extraction result saved and the PDF bytes cleared afterward.

## Production (Coolify + Inngest Cloud)

1. Create a free app at [app.inngest.com](https://app.inngest.com).
2. Point the app's sync URL at `https://<your-coolify-host>/api/inngest`.
3. Add Coolify env vars: `INNGEST_EVENT_KEY`, `INNGEST_SIGNING_KEY` (both from the Inngest Cloud dashboard). Leave `INNGEST_DEV` unset. Keep `OPENAI_API_KEY` and the existing auth/DB vars as they are.
4. Redeploy so the new migration (adding `PENDING` status and `fileData`) applies.
5. Confirm a real test upload moves through the statuses; inspect run history and any failures directly in the Inngest Cloud dashboard (full step output + error messages are visible there).

## Reliability notes

- **Retries**: `process-document` retries temporary failures (mostly AI/network issues) up to 3 times. Permanent errors (an unreadable PDF, a missing file, bad AI-response shape) throw `NonRetriableError` and mark the document Failed immediately instead of wasting retries on something that will never succeed.
- **Duplicate prevention**: this is enforced by the job's own atomic database claim (`claimDocumentForProcessing` in `lib/document-job.ts`), not by event-level deduplication — worth being precise about this, since testing showed the local Inngest dev server does *not* reliably dedupe identical event `id`s on its own (multiple duplicate sends all trigger a function attempt). The claim works by only allowing a document to move from `PENDING` to `PROCESSING` via a conditional update; if two attempts race to claim the same document, only one atomically succeeds and the other cleanly skips. **A real concurrency bug was found and fixed here during testing**: the claim's original condition allowed both `PENDING` and `PROCESSING` as valid source states, which meant an already-claimed (`PROCESSING`) document could be "claimed" again by a second concurrent attempt. Verified fixed with an integration test that races two claims against the same document with `Promise.all` and asserts exactly one succeeds.
- **Logs**: the function's logger calls record skip/claim/complete/fail events; both the local Dev Server and Inngest Cloud keep the full run timeline (including each step's input/output) for debugging a failed run.
- **Storage**: PDF bytes sit in `document.fileData` only until processing finishes (success or permanent failure), then they're cleared — never left sitting in the database longer than needed.

## Troubleshooting

| Symptom | Check |
|---|---|
| Stuck on Pending | Is the Dev Server running locally / is the Cloud sync healthy? Is `/api/inngest` reachable? Are the event/signing keys set in production? |
| Failed with an API-key-related message | Is `OPENAI_API_KEY` present in the environment actually running the app? |
| Upload itself errors out about queueing | Inngest keys or `INNGEST_DEV` may be misconfigured for the current environment. |
| Completed, but the UI still looks stale | Hard refresh — the auto-refresh polling only runs while a document is Pending/Processing, so it stops once nothing is in flight. |
