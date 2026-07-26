# Step 1: Application setup and authentication

## Instructions

Extend the existing Next.js app into a basic authenticated application.

Your task is to:

1. Add PostgreSQL and connect it to the application.
2. Add user authentication using a suitable, well-supported library.
3. Implement:

   * Sign-up
   * Sign-in
   * Sign-out
   * A protected dashboard
4. Ensure users who are not signed in cannot access the dashboard.
5. Add basic validation and clear error messages.
6. Add any required database migrations and environment-variable documentation.
7. Submit the work through a focused pull request explaining:

   * The approach taken
   * How to run and test it
   * Any important decisions or limitations
8. Deploy the changes through the existing Coolify setup.

Use AI tools where helpful, but understand and verify the code they produce. Do not add unnecessary features or dependencies.

9. Document your process and motivate your choice of library. Please make a PR to this repo (not the nextjs app repo) with this documentation.

## Success criteria

(For general evalution criteria please look at the success criteria for Step 1, these are the evaluation criteria for all future steps as well).

The task is complete when:

* A new user can create an account.
* An existing user can sign in and sign out.
* The dashboard is only accessible to authenticated users.
* Passwords and session data are handled securely by the chosen library.
* Invalid input and failed authentication show clear messages.
* Database migrations work on a clean database.
* The application builds and deploys successfully.
* Setup instructions and required environment variables are documented.
* The pull request is focused, understandable, and passes the available checks.
* No secrets or unrelated changes are committed.
