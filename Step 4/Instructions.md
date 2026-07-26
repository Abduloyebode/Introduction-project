# Step 4: Background processing

## Instructions

Move document processing into a background job.

Your task is to:

1. Add a suitable job queue or background-job service.
2. Change uploads so the request returns without waiting for AI processing.
3. Track each job using statuses such as:

   * Pending
   * Processing
   * Completed
   * Failed
4. Show processing status in the interface.
5. Add retry handling for temporary failures.
6. Prevent the same job from being processed more than once.
7. Add useful logs for debugging failed jobs.
8. Document how to run and monitor the worker.
9. Submit the work through a focused pull request and deploy it after approval.

Keep the architecture appropriate for the current project size.

## Success criteria

The task is complete when:

* Upload requests do not wait for AI processing to finish.
* Jobs move correctly through each status.
* Completed results appear without requiring a new upload.
* Temporary failures are retried safely.
* Duplicate processing is prevented.
* Permanent failures are visible and understandable.
* Worker setup and troubleshooting are documented.
* Relevant tests cover job states and failure handling.
* The application builds and deploys successfully.
