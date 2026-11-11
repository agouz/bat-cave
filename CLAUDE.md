# The Bat Cave

Personal apps for one user (owner GitHub login `agouz`). Static single-file PWAs on GitHub Pages, backed by one shared Supabase project (`kkdwsbbvgvklxqqumcdw`, ap-southeast-2). Live at https://agouz.github.io/bat-cave/

## Hard rules
- **Never put secrets in this repo.** It is public. No tokens, service_role keys, passwords or personal data in any file, commit message or issue. The Supabase anon key in the HTML is public by design.
- **Data lives in Supabase, not in the HTML.** Pages hold only layout, styling and images. Anything personal or editable goes in a table.
- **Security is Row Level Security.** Every table has one owner-only policy for `authenticated` (`auth.uid() = <owner uid>`). No anon policies. Never gate on `user_metadata`. Never disable RLS or touch existing tables without being asked.
- **Login is GitHub OAuth, owner only.** The session is stored in localStorage on `agouz.github.io`, so all pages under `/bat-cave/` share one sign-in. Pages must not hold data for signed-out visitors.
- Only `https://` links are allowed from the hub (`safeUrl()`).

## Stack and conventions
- Each app is one `index.html`: inline CSS and JS, no build step. supabase-js is loaded from a CDN, pinned to `@2.45.4` (UMD).
- Close every `<script>` tag. An unterminated script at end of file does not run on GitHub Pages.
- Defer init in the auth callback with `setTimeout`, or supabase-js deadlocks.
- OAuth `redirectTo` is `location.origin + location.pathname`. Supabase Redirect URLs must include `https://agouz.github.io/bat-cave/**`.
- Every app has a full-width **Back** button back to the hub, styled like the hub's.
- Mobile first (about 390px wide), safe-area insets, 44px minimum touch targets, dark theme in the hub.

## Apps

### Hub (`/index.html`)
Login page and launcher. Tengwar "one app to rule them all" (SVG outlines), original bat emblem, big buttons. The buttons come from the `cave_links` table (`parent_id`, `label`, `subtitle`, `icon`, `url`, `sort_order`). A row with no `url` opens its children as a sub-screen. Current structure: Trips (child: Americas Trip), Sticker Albums, Others (child: Training). Add an app by inserting a row, and add an icon to the `ICONS` map in the hub if it needs a new one. The hub is built from a template with the anon key and the Tengwar path injected, so edit the built file carefully.

### Trips (`/trips/`)
Americas trip dashboard: itinerary legs with a map, bookings, flight details, packing list, prep tasks, health, visas and other notes, and lessons. Tables: `trip_legs`, `trip_bookings`, `trip_flight_details`, `trip_packing_items`, `trip_prep_tasks`, `trip_info` (health, visas and other pages), `trip_lessons`. Country background images and offline map data are bundled in the folder.

### Stickers (`/stickers/`)
Panini sticker album tracker with several albums (FIFA World Cup 2026, Batman 80th Anniversary, Justice League, World of Batman, Batman: The Animated Series). An album picker, then Overview, Collection and Duplicates tabs. Tables: `stickers`, `countries`, `batman_stickers` and per-album tables, with a `duplicate_count` field. It calls the REST API directly with the signed-in token.

### Training (`/training/`)
Full body machine session guide: warm-up plus exercise groups with form cues, diagrams and video links. Tapping an exercise opens a sheet to log weight (kg) and reps. Each save inserts a new row, never overwriting. The sheet shows a weight chart, history grouped by day with sets, a PB badge, and two-tap delete. Table: `training_log` (`exercise_key` is a slug of the exercise name, `weight_kg`, `reps`, `logged_on` in Sydney time). Renaming an exercise detaches its history.

## Working here
- Test headless (Playwright) with a seeded session and stubbed `/auth/v1/user` and `/rest/v1/*`.
- Commit to `main`. Pages redeploys in about a minute. Before reporting a change as live, confirm it on the real site.
