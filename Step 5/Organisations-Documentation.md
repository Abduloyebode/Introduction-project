# Step 5 — Organisations and permissions

Users belong to one organisation. Workflows and documents are scoped to that organisation. Admins manage membership; members use the product.

## Roles

| Role | Can do |
|------|--------|
| **Admin** | Everything a member can, plus invite / revoke invites / remove members / change roles, and view the audit log |
| **Member** | Create and manage workflows and documents inside the organisation |

All checks run in server actions and list helpers (`organisationId` filters). Hiding UI is not the security boundary.

## How people join

1. New sign-up without an invite gets a personal organisation and becomes its **Admin** (also happens for any existing user who somehow has no membership yet).
2. An admin creates an invite. With no email provider yet, the accept link is logged on the server and shown once in the UI (same pattern as password reset).
3. The invitee opens `/invite/<token>`, signs up or signs in with the invited email, and accepts. Invite tokens are stored hashed.

A user can only belong to **one** organisation. Someone who already has a membership cannot accept an invite into another org.

## Data migration

Existing users each get a personal organisation and Admin membership. Their existing workflows and documents are attached to that organisation.

## Security decisions and limits

- Access is enforced with `requireOrganisationMembership()` plus `organisationId` on every workflow/document query and mutation. Direct IDs from another org return “not found”.
- Admin-only mutations return errors if a Member calls them (not only UI gating).
- The last Admin cannot be removed or demoted.
- Invite accept requires the signed-in email to match the invite.
- No email sending yet; operators share the invite link manually.
- No multi-org switcher in this step.

## Where to look in the app

- `/dashboard/organisation` — members, invites (admin), audit log (admin)
- `/invite/[token]` — accept flow
- Audit actions: `ORGANISATION_CREATED`, `MEMBER_INVITED`, `INVITE_REVOKED`, `INVITE_ACCEPTED`, `MEMBER_REMOVED`, `MEMBER_ROLE_CHANGED`
