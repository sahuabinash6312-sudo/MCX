# MCA + GST Lead CRM

Responsive MCA + GST lead CRM with Supabase email/password authentication and shared per-user cloud storage.

## Authentication architecture

1. The browser loads the Supabase JavaScript client and creates a client with the project's public URL and publishable key.
2. Users sign in or create an account with email/password through `supabase.auth`.
3. The app reads the current session with `getSession()` and listens for authentication changes with `onAuthStateChange()`.
4. CRM screens are initialized only when an authenticated user is present; otherwise the sign-in screen is shown.
5. CRM state is stored in `public.crm_state`, keyed by the authenticated user's UUID.
6. Row Level Security policies restrict SELECT/INSERT/UPDATE/DELETE to rows where `auth.uid() = user_id`.

## Request/data flow

`Browser -> Supabase Auth -> authenticated session -> crm_state query/upsert -> CRM UI`

The app uses `select("state,updated_at")` to load the user's CRM state and an upsert keyed by `user_id` to save it. This keeps the same user's CRM data available across devices when they sign into the same account.

## Credentials and tokens

- The frontend contains only the Supabase project URL and public publishable/anon key. These are intended for browser use.
- No service-role key, database password, or other privileged secret is included in the frontend.
- Authentication is handled by Supabase Auth; the app does not implement its own password storage.
- Authorization is enforced by Supabase Row Level Security, not by hiding UI controls alone.
- Do not add a Supabase service-role key to `index.html` or commit any private credentials to GitHub.

## Database

Create the `public.crm_state` table and its RLS policies using the SQL supplied for this CRM. The table stores a JSONB `state` document per authenticated user and automatically updates `updated_at` on changes.

## Included files

- `index.html` — CRM application, responsive UI, import/export logic, Supabase Auth and cloud sync.
- `sample-data.json` — bundled MCA/GST sample dataset used by the CRM when available.
