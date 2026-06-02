# Wellbeing Lab Meetings

A lightweight web app for organising a research lab's recurring meetings: tracking who is **attending** and who is **presenting**, so meetings stop falling apart when people quietly don't show up or arrive unprepared.

It runs as a single static page backed by a small database, so the whole lab opens one link and shares the same live schedule and attendance — no installs, no accounts.

---

## The problem it solves

Our lab meets every two weeks, alternating between two groups, with a few people presenting each time. Two things kept going wrong:

1. People didn't show up and didn't tell anyone, so meetings were half-empty by surprise.
2. People who were supposed to present sometimes hadn't prepared, and occasionally *no one* presented.

This app makes both visible: everyone logs their attendance, presenters are assigned in advance with clear time slots, and the organiser can see the state of the next meeting at a glance and send a reminder.

---

## Features

- **Shared schedule** — the next meeting's date, room, time, and assigned presenters, visible to everyone.
- **Per-presenter time slots** — auto-calculated from the start time, talk length, and individually adjustable breaks; warns if the schedule overruns the end time.
- **Attendance logging** — every member marks "attending" or "can't make it"; live tally of who's in, out, and yet to reply.
- **Absence reasons** — anyone who can't attend leaves a reason, visible only to the admin, plus a prompt to notify the professor.
- **Cross-group presenters** — supports the real-world case where someone swaps in from the other group.
- **Reminder emails** — generates a ready-to-send message (date, time, room, presenters) addressed to all members with the professor Cc'd; copy-to-clipboard or open-in-mail.
- **Admin tools** (PIN-protected) — manage the roster, edit names and emails inline, assign presenters, read absence reasons, and configure timings.
- **History** — past meetings with who presented and who attended.
- **Backup & restore** — export the entire setup as text and restore it in one step.

---

## How it works

The app is intentionally simple — two moving parts:

- **Front end** — a single self-contained HTML file (React, loaded in the browser, no build step). This is everything the user sees and interacts with.
- **Data** — a small [Supabase](https://supabase.com) (hosted Postgres) database stores all shared state in a single key/value table. Each person's chosen identity is kept locally in their own browser.

Hosting is just static file hosting (e.g. [Netlify](https://www.netlify.com)). The front end talks directly to Supabase from the browser using a public **publishable key**, with **Row Level Security** enabled on the table so the key alone can't be abused.

```
Browser (React single-file app)  ──►  Supabase (Postgres: shared roster, meetings, attendance)
        served by Netlify
```

---

## Tech stack

- **React** (via in-browser modules, no build tooling)
- **Supabase** — hosted Postgres + auto-generated REST API
- **Netlify** — static hosting
- Plain CSS (custom design, no framework)

---

## Run your own copy

1. **Create a Supabase project** at [supabase.com](https://supabase.com).
2. **Create the table.** In the project's SQL Editor, run:

   ```sql
   create table if not exists kv (
     key text primary key,
     value jsonb,
     updated_at timestamptz default now()
   );
   alter table kv enable row level security;
   drop policy if exists "kv read" on kv;
   create policy "kv read"   on kv for select using (true);
   drop policy if exists "kv insert" on kv;
   create policy "kv insert" on kv for insert with check (true);
   drop policy if exists "kv update" on kv;
   create policy "kv update" on kv for update using (true) with check (true);
   ```

3. **Add your keys.** In `index.html`, near the top, replace the two placeholder lines with your project's **Project URL** and **publishable (anon) key** from *Settings → API*:

   ```js
   const SUPABASE_URL      = "https://YOUR-PROJECT.supabase.co";
   const SUPABASE_ANON_KEY = "YOUR-ANON-PUBLIC-KEY";
   ```

4. **Deploy.** Drag `index.html` onto [Netlify Drop](https://app.netlify.com/drop), or host it anywhere that serves static files.
5. Open the live URL and complete the one-time setup (lab name, admin PIN, members).

---

## Notes & limitations

- **No individual logins.** Identity is by name-selection, and admin actions sit behind a shared PIN. This is appropriate for a small, trusting group, not a security boundary. Absence reasons are "admin-only" in the sense that they're behind the PIN.
- **The publishable key is meant to be public** — it ships in the page source of any Supabase site. Data is protected by Row Level Security, not by hiding the key.
- **Reminder emails** are generated client-side and handed to the user's mail app (or copied to paste into webmail); the app does not send mail by itself.

## Possible next steps

- Real per-member login via Supabase Auth (so attendance can only be logged for yourself).
- Automatic email reminders on a schedule (would require a backend function + an email service).
- Recurring-meeting generation instead of manual scheduling.
