# WandrQuest — a live, multiplayer quiz game you can re-theme for anything

A real-time, mobile-first “walk-around” quiz game for classrooms, training days, and events. Players join on their phones, explore a top-down map, walk up to characters (NPCs) to answer scenario questions, and climb a live leaderboard projected on the big screen — Kahoot meets a tiny Pokémon map.

It's **multi-tenant**: hosts sign in, create games (each gets a short **room code**), and run isolated live sessions. Players join a room by code with **no account**. Everything — questions, characters, roles, and the **map itself** — is edited in the browser and published live to every player. No rebuild, no redeploy.

### ▶️ Live demo (always on)

| | |
|---|---|
| **Try the demo** (player, phone) | https://wandrquest.vercel.app/game?room=DEMO |
| **Demo projector board** | https://wandrquest.vercel.app/master?room=DEMO |
| **Host a game** (sign in / sign up) | https://wandrquest.vercel.app/host |
| **Landing** | https://wandrquest.vercel.app |

The always-on demo room (**code `DEMO`**) is themed as a **Community Centre** (Reception, Library, Sports Hall, Café). To run your own game, create one at **/host** — you'll get a room code to share.

---

## Screenshots

**Landing**

![Landing page](docs/screenshots/landing.png)

**Host dashboard** — sign in, create games (each with a room code), edit/run/share

![Host dashboard](docs/screenshots/host-dashboard.png)

**Host / projector view** — live map with themed zones, obstacles, players, and leaderboard

![Projector / master view](docs/screenshots/master.png)

**Player view** (mobile) — follow-camera, joystick, minimap

![Player game view](docs/screenshots/player-game.png)

**Editor — NPCs tab** — character list decoupled from a full properties panel

![Editor NPCs tab](docs/screenshots/editor-npcs.png)

**Editor — Map tab** — drag/resize zones & obstacles, theme, background image, aspect ratio

![Editor Map tab](docs/screenshots/editor-map.png)

**Starter templates** — load a ready-made scenario with a real floor-plan background (Museum shown)

![Editor with a template loaded](docs/screenshots/editor-template.png)

**Editor — Roles tab** — define the roles players can pick and their passive bonuses

![Editor Roles tab](docs/screenshots/editor-roles.png)

---

## What you can do

- **Start from a template.** Five ready-made scenarios — **School, Office, Hospital, Museum, Shopping Mall** — each with themed questions, roles, zones, obstacles, and a real floor-plan background. Pick one in the editor's **Load template** menu, tweak, and publish.
- **Host your own games.** Sign in (email + password), create a game → get a shareable **room code**. Run many games at once; each room's players, scores, events, and content are fully isolated.
- **Players join with no account** — just the room code, a name, and a role.
- **Author questions visually.** Add/remove characters, write the dialogue, options, the correct answer, and per-answer feedback. Set tier (Common / Rare / Elite-timed / Boss co-op), points, and zone.
- **Drag & drop the map.** Position characters by dragging them on a scaled map. A dashed *“what a player sees”* guide shows the player camera window so you size things correctly.
- **Edit the map itself:**
  - **Zones** — add / move / resize / rename / recolour the regions (e.g. Library, Café).
  - **Obstacles** — walls & furniture with real **collision** (players walk around them).
  - **Themes** — Grid / Campus / Factory / Space palettes.
  - **Background image** — paste a URL or upload one (stored in Supabase), with opacity.
  - **Aspect ratio** — reshape the map (Tall / Portrait / Square / Landscape, or *match your background image*) so backgrounds never get cropped or stretched.
- **Custom roles.** Replace the default roles entirely (e.g. Volunteer / Staff / Visitor) and give each a passive scoring bonus.
- **Run the session live.** Open/lock joining, run events (2× points, speed round, co-op boss, end game), and watch the leaderboard update in real time.
- **Publish instantly.** Save in the editor → every player and the projector pick it up from Supabase. Export/Import the whole config as JSON too.

---

## How it works

Self-contained static pages share one realtime backend:

```
index.html        Landing page
host/index.html   Host console  — Supabase Auth, create/manage rooms
game/index.html   Player view   — Phaser 3 + nipple.js joystick (joins by room code)
master/index.html Projector     — Canvas2D live map + leaderboard + host controls
editor/index.html Editor        — drag-and-drop authoring (per room)
npc-data.js       Shared default config (branding, roles, zones, obstacles, map, NPCs)
supabase_schema.sql  Database + RLS + storage setup
```

- **Frontend:** plain HTML/JS, no build step. Served as static files on **Vercel** (it just ships the files; it scales effortlessly).
- **Backend:** **Supabase** (Postgres + Realtime + Auth + Storage). A `rooms` table holds each game's config + join-state; `players` and `game_events` are scoped by `room_id`. Realtime subscriptions are filtered per room, so rooms don't see each other's traffic.
- **Auth + RLS:** hosts authenticate; **Row-Level Security** enforces it — only a room's host can edit its config or trigger events (boss/end/2×), players can only join rooms that are *open*, and rooms are isolated. Players never log in.
- **Config flow:** the editor saves a room's config to its `rooms.config`; the player and projector read it (falling back to `npc-data.js` defaults), so edits go live without a redeploy.

---

## Run it yourself

1. **Supabase** — create a project, open the SQL editor, paste and run [`supabase_schema.sql`](supabase_schema.sql). This creates the `rooms` / `players` / `game_events` tables, **Row-Level Security policies**, realtime, and a public `map-assets` storage bucket. Enable **Email** auth (and turn on auto-confirm for instant host sign-up).
2. **Configure** — set `SUPABASE_URL` / `SUPABASE_ANON_KEY` near the top of `host/`, `game/`, `master/`, and `editor/` `index.html`.
3. **Deploy** — push the folder to Vercel (or any static host). `vercel.json` routes `/`, `/host`, `/game`, `/master`, `/editor`.
4. **Use it** — go to `/host`, create an account, create a game (you'll get a room code), open the **editor** to build your scenario, then **Project** it and **Open join** so players can enter the code at `/game`.

> Security note: players are intentionally anonymous (names + scores only). RLS protects host actions and isolates rooms. Within a single room, anonymous players aren't individually authenticated, so this suits friendly/event use; add per-player tokens + server-side score validation before high-stakes/graded use.

---

## Tech

Phaser 3 · nipple.js · Supabase (Postgres / Realtime / **Auth** / Storage, with **RLS**) · Vercel · vanilla JS (no framework, no bundler).

### Template background credits

Floor-plan backgrounds are free images from **Wikimedia Commons**, re-hosted in Supabase storage (fetched via [`tools/fetch-maps.mjs`](tools/fetch-maps.mjs)):

| Template | Source | License |
|---|---|---|
| School | Carl Hårleman — *Design for a School Building in Karlstad* | Public domain |
| Office | Henry Hardenbergh — *Hotel Martinique 1911 floor plan* | Public domain |
| Hospital | Wellcome Collection — *Addenbrooke's Hospital ground floor plan* | CC BY 4.0 |
| Museum | Rangan Datta — *Floor Plan, RBI Museum Kolkata* | CC BY-SA 4.0 |
| Shopping Mall | Wolfmann — *Bergen Storsenter floor plan* | CC BY-SA 4.0 |

*The demo room + host are provisioned by [`tools/setup-multiroom.mjs`](tools/setup-multiroom.mjs); the multi-room RLS is verified by [`tools/e2e.mjs`](tools/e2e.mjs); screenshots are captured with [`tools/shots.mjs`](tools/shots.mjs).*
