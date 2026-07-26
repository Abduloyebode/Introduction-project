# Step 5: Organisations and permissions

## Instructions

Add organisations and role-based access control.

Your task is to:

1. Allow users to belong to an organisation.
2. Add two roles:

   * Admin
   * Member
3. Ensure workflows and documents belong to an organisation.
4. Ensure users can only access data from their own organisation.
5. Allow admins to:

   * Invite members
   * Remove members
   * Change member roles
6. Allow members to use the application without managing the organisation.
7. Add an audit log for important membership and permission changes.
8. Add tests for access-control boundaries.
9. Submit the work through a focused pull request and deploy it after approval.

Treat all access checks as server-side security requirements, not only UI restrictions.

## Success criteria

The task is complete when:

* Users can only access data belonging to their organisation.
* Admin and member permissions behave as documented.
* Users cannot bypass permissions through direct URLs or API requests.
* Admins can invite, remove and update members.
* Important permission changes appear in the audit log.
* Existing data is migrated safely.
* Tests cover cross-organisation access and role restrictions.
* Security decisions and limitations are documented.
* The application builds and deploys successfully.
