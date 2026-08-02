# Step 4 — Background document processing

I initially had an issue where, while uploading the PDF, it blocks the whole request until the AI finishes, and if it's slow or there's connection issues, the user is stuck looking at a spinner with no feedback.

So I used Inngest to move it to a background job, since it's a required piece of this step. Now the upload returns instantly, and the actual AI extraction happens separately while the UI shows pending status instantly.

The tricky part was making sure the same upload never gets processed twice, since jobs can retry and I didn't want a duplicate run to accidentally process a document twice. While testing this, I actually found a real concurrency bug: two attempts to claim the same document at the same time could both succeed, because the check allowed claiming a document that was already being processed. I fixed it so only a document that hasn't started processing yet can be claimed, and verified the fix with a test that races two claims against each other at once.

The detailed setup steps, environment variables, and troubleshooting notes are written up separately in [`Background-Jobs-Technical-Notes.md`](./Background-Jobs-Technical-Notes.md) in this same folder — that part was drafted with AI assistance and reviewed by me, so it's kept apart from this write-up.
