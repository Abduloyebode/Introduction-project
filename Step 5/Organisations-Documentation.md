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

A user can only belong to **one** organisation at a time. If they're the sole member of their current org (the common case — every user gets a personal org on first dashboard visit, so almost everyone starts out this way), accepting an invite elsewhere safely retires that membership as part of accepting. If their current org has other members, the invite is blocked (both at invite-creation and at accept time) — there's no way to abandon a real team by accident.

## Data migration

Existing users each get a personal organisation and Admin membership. Their existing workflows and documents are attached to that organisation.

## Security decisions and limits

- Access is enforced with `requireOrganisationMembership()` plus `organisationId` on every workflow/document query and mutation. Direct IDs from another org return “not found”.
- Admin-only mutations return errors if a Member calls them (not only UI gating).
- The last Admin cannot be removed or demoted. This check locks every membership row in the org (stable order, so concurrent admin actions can't deadlock each other) and reads the current role/admin count from inside that lock — not from data read before the transaction opened. An earlier version of this check read the target's role once up front and used that stale value inside the lock, which let a role change racing a removal slip past the guard; a regression test now forces that exact interleaving.
- Invite accept requires the signed-in email to match the invite.
- Two concurrent identical invites (double-click, or two admins inviting the same email) can't both succeed — enforced by a partial unique DB index (one live PENDING invite per org+email), not just an application-level check, since a plain check-then-insert isn't race-safe against a row that doesn't exist yet.
- Double-submitted accepts (double-click, replayed request) degrade to a clean “may already be accepted” message instead of a raw error.
- Audit log writes are best-effort: a transient failure there never blocks or falsely fails an action that actually succeeded.
- No email sending yet; operators share the invite link manually.
- No multi-org switcher in this step.

## Where to look in the app

- `/dashboard/organisation` — members, invites (admin), audit log (admin)
- `/invite/[token]` — accept flow
- Audit actions: `ORGANISATION_CREATED`, `MEMBER_INVITED`, `INVITE_REVOKED`, `INVITE_ACCEPTED`, `MEMBER_REMOVED`, `MEMBER_ROLE_CHANGED`
