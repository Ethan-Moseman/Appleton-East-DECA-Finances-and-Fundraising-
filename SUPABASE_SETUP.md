# One-time Supabase setup for shared partnerships

Companies that students log are saved to a `partnerships` table in Supabase so
they show up on everyone's screen (including the admin view). This table has to
be created once by hand.

## Steps

1. Go to https://supabase.com/dashboard and open the project
   (`cttkxjpiswzukqwhknne`).
2. In the left sidebar, click **SQL Editor** and open a new snippet.
3. Paste everything in the box below and click **Run**. It is safe to run this
   more than once — it only creates what's missing (so if you ran an earlier
   version of this script, run this one again to add the newer columns).
4. Reload the finance app. The red "Cross-device sync is not set up" banner on
   the Partnerships page should disappear.

```sql
create table if not exists public.partnerships (
  id text primary key,
  company text not null,
  created_at timestamptz not null default now()
);

alter table public.partnerships add column if not exists contact text not null default '';
alter table public.partnerships add column if not exists value numeric not null default 0;
alter table public.partnerships add column if not exists description text not null default '';
alter table public.partnerships add column if not exists assigned_to text not null default '';
alter table public.partnerships add column if not exists team_id text not null default '';
alter table public.partnerships add column if not exists next_date text not null default '';
alter table public.partnerships add column if not exists response text;
alter table public.partnerships add column if not exists comments text not null default '';
alter table public.partnerships add column if not exists visit_date text not null default '';
alter table public.partnerships add column if not exists callback_notes text not null default '';
alter table public.partnerships add column if not exists status text not null default 'pending';
alter table public.partnerships add column if not exists tier text;
alter table public.partnerships add column if not exists notes text not null default '';
alter table public.partnerships add column if not exists submitted_by text not null default '';

alter table public.partnerships enable row level security;

drop policy if exists "app can read partnerships" on public.partnerships;
drop policy if exists "app can add partnerships" on public.partnerships;
drop policy if exists "app can update partnerships" on public.partnerships;
drop policy if exists "app can delete partnerships" on public.partnerships;

create policy "app can read partnerships"
  on public.partnerships for select using (true);
create policy "app can add partnerships"
  on public.partnerships for insert with check (true);
create policy "app can update partnerships"
  on public.partnerships for update using (true) with check (true);
create policy "app can delete partnerships"
  on public.partnerships for delete using (true);
```

## How the app uses it

- Every device loads the shared list when the app opens and re-checks every
  30 seconds (there is also a Refresh button on the admin Partnerships page).
- When a student logs a business visit (or an admin adds/edits/deletes a
  company, or records a response/tier), the change is written to this table
  right away.
- Anything that was logged on a device before the table existed is uploaded
  automatically the first time that device connects after setup.
- Fundraising teams themselves are still stored per-device; a company's team
  assignment syncs, but team names only show on the device where the team was
  created.
