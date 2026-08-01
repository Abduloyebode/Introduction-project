# Step 1 — Auth library choice and setup notes

For the auth library I had three options: Lucia, Auth.js, and Better Auth. Lucia isn't really used anymore — it's been that way since 2025, and it's not considered a proper library now, more of a way to learn how to build your own auth. Auth.js is still around, but even that team moved over to Better Auth about a year ago.

I picked Better Auth because it gives me what I'm looking for: it keeps everything in our own Postgres database instead of relying on an outside service, which matches how the rest of this project is set up (self-hosted VM, own database, no external accounts). It also has an organizations plugin built in, which will come in handy when I get to Step 5.

Setup took a bit of time since I had to build the sign-up/sign-in pages and wire up the database myself, rather than dropping in a pre-made template the way Clerk would let you.

The detailed setup steps, environment variables, and Coolify deployment notes are written up separately in [`Authentication-Technical-Notes.md`](./Authentication-Technical-Notes.md) in this same folder — that part was drafted with AI assistance and reviewed by me, so it's kept apart from this write-up.
