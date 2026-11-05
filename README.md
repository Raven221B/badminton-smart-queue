# 🏸 Badminton Smart Queue & Player Management System

A private GitHub-ready Vite web application with Supabase-backed authentication, session management, player roster, role-restricted screens, court assignments, doubles matchmaking, scoring, game fees, shuttle requests, player/QM messages, notifications, tournament announcements and match history.

## Status

**Development source code — not yet deployed or production validated.** The included PostgreSQL migrations and Supabase Edge Function require a new Supabase project, and the UI requires browser-based testing with genuine accounts. Some advanced features of the original 33-section specification (complete tournament bracket engine, score corrections, automatic rating updates, full historical month-by-month reports, configurable scoring, automatic rest timers, and push notifications) are not yet implemented. Do not use for collection of real payments until thoroughly tested.

## Stack

- Vite + JavaScript, responsive HTML/CSS (no Base44 credit usage)
- Supabase Auth + PostgreSQL + row-level security
- PostgreSQL RPC functions for atomic game actions and fees
- Supabase Edge Function for manager-only invitations
- Supabase Realtime subscriptions for selected changes
- GitHub Actions continuous integration

## Setup

1. Create a **private** repository on GitHub and upload the contents of this folder, or run `git init`, `git add .`, `git commit -m "Initial badminton app"` and push to your own repository.
2. Create a new Supabase project at <https://supabase.com>.
3. Open Supabase **SQL Editor**. Apply `supabase/migrations/001_initial.sql` first, then `supabase/migrations/002_hardening.sql`. Do not reapply a migration twice. Check both scripts for errors before proceeding.
4. In Supabase **Auth > Providers > Email**, turn **off public email signup**. Keep email invitations/password recovery enabled. Add your app URLs under **Auth > URL Configuration** (Site URL and Redirect URLs). This restricts new accounts to the protected invite endpoint.
5. Create the **first manager account** using the Supabase dashboard/admin console (it is a one-time bootstrap, not the open public sign-up page). After that account is created, promote it in SQL Editor using its exact user UUID:
   ```sql
   update public.profiles set role='admin' where id='REPLACE_WITH_YOUR_AUTH_USER_UUID';
   ```
6. Deploy the invitation function with Supabase CLI:
   ```bash
   supabase login
   supabase link --project-ref YOUR_SUPABASE_PROJECT_REF
   supabase functions deploy invite-player
   ```
   Supabase provides `SUPABASE_URL`, `SUPABASE_ANON_KEY`, and `SUPABASE_SERVICE_ROLE_KEY` to Edge Functions as project secrets in supported deployments. Never expose the service role key to the browser or GitHub.
7. Copy `.env.example` to `.env` and fill `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` using your **public/publishable** Supabase key. Do **not** use the service role key here.
8. Run:
   ```bash
   npm install
   npm run dev
   ```
9. Test the manager login and invitation flow, then test with four approved players. Start a session, create a court, check in players, generate a match, start it, score the game, and check fee balances.
10. For Cloudflare Pages, connect your GitHub repository and set **Build command**: `npm run build`; **Build output directory**: `dist`. Add the two Vite environment variables in the Pages project settings. Configure Supabase Auth redirects to your published site.

## Typical usage

- Admin/QM: invite players, approve levels, create session and courts, monitor daily roster, auto-generate or manually assign games, record results, issue shuttles and record payments.
- Player: accept an invitation, set password, log in, check in, view upcoming game, view personal match history and fees, request a shuttle, send a chat or preferred-match request.

## Security notes

- Account roles are assigned by the server. New invited users have `player` role and `unrated` skill.
- Critical matchmaking, results, payments and shuttle issuance are handled in PostgreSQL with role checks, not browser-only code.
- The `public_players` view exposes only public nicknames, level, rating and active status; private `profiles` are not visible to other players.
- Keep RLS enabled and audit permissions after database changes.
- The invite route creates a provisional Supabase Auth account upon inviting; the user finishes *setting their password* only after opening the invitation. A pending invitation is tracked separately.
- Sensitive functions need a full Supabase integration and two-device concurrent testing before launch.

## Commands

```bash
npm test
npm run build
```

## Existing Base44 migration

This project is a **new independent implementation**, not a copy of Base44 files or production records. Export player and match data from Base44 separately and review it before importing into PostgreSQL; retain the original IDs via a migration mapping table. Do not copy private data into this repository.
