# WandrQuest

**A live, multiplayer "walk-around" quiz game for classrooms, training days, and events.**

Players join on their phones, explore a top-down map, walk up to characters (NPCs) to answer scenario questions, and climb a live leaderboard projected on the big screen — Kahoot meets a tiny Pokémon world.

---

### ▶️ Live demo (always on)

| | |
|---|---|
| **Play the demo** (mobile-friendly) | https://wandrquest.vercel.app/game?room=DEMO |
| **Projector view** | https://wandrquest.vercel.app/master?room=DEMO |
| **Host console** | https://wandrquest.vercel.app/host |

The always-on demo room (`DEMO`) is themed as a **Community Centre** — Reception, Library, Sports Hall, and Café zones with NPCs, obstacles, and a live leaderboard. No account needed to play.

---

## Screenshots

**Landing**

![Landing page](docs/screenshots/landing.png)

**Host sign-up** — password strength meter, confirm field, visually distinct from sign-in

![Host sign-up](docs/screenshots/host-signup.png)

**Host dashboard** — create games (each with a unique room code), edit, share player link, project, delete

![Host dashboard](docs/screenshots/host-dashboard.png)

**Projector / master view** — live top-down map with zones, obstacles, player positions, leaderboard and host controls

![Projector / master view](docs/screenshots/master.png)

**Player view** (mobile) — follow camera, virtual joystick, minimap, role badge

![Player game view](docs/screenshots/player-game.png)

**Player role selection** — pick a role at join time, each with a unique passive bonus

![Player role selection](docs/screenshots/player-role.png)

**Editor — NPCs tab** — two-pane layout: character list rail + full properties panel; drag NPCs on the map

![Editor NPCs tab](docs/screenshots/editor-npcs.png)

**Editor — Map tab** — drag/resize zones & obstacles, theme picker, background image upload, aspect ratio control

![Editor Map tab](docs/screenshots/editor-map.png)

**Editor — Roles tab** — define roles, emoji, description and passive bonus type

![Editor Roles tab](docs/screenshots/editor-roles.png)

**Starter templates** — 5 ready-made scenarios with real floor-plan backgrounds (School shown here)

![Template picker](docs/screenshots/editor-template-menu.png)

![Template loaded in editor](docs/screenshots/editor-template.png)

---

## Features

### Multi-room hosting
Hosts sign up, create rooms (each gets a short **room code**), and run isolated live sessions. Many hosts, many simultaneous games — each room's players, scores, content, and events are completely separate. Players join with **no account** — just a code, a name, and a role pick.

### Visual editor (no rebuild needed)
Everything is authored in the browser editor and published live to every player the moment you save. No redeploy.

- **NPCs** — add/remove characters, write dialogue, 4 answer options (mark the correct one), per-answer feedback, tier (Common / Rare / Elite-timed / Boss co-op), points value, zone assignment, and role bonus target.
- **Map** — drag characters to position them on a scaled map preview. A dashed viewport guide shows the camera window so you size things correctly.
- **Zones** — add, drag, resize, rename, and recolour the map regions that appear on the projector and minimap.
- **Obstacles** — furniture and walls with real AABB collision — players walk around them, they appear on the minimap and projector.
- **Themes** — Grid / Campus / Factory / Space colour palettes applied to floor and grid.
- **Background image** — paste a URL or upload an image (stored in Supabase Storage), with adjustable opacity.
- **Aspect ratio** — reshape the world (Tall / Portrait / Square / Landscape / Match background image) so backgrounds fill the frame without stretching or cropping.
- **Roles** — fully editable: emoji, label, description, and passive bonus type (role-match bonus, teammate bonus, or none).
- **Branding** — app name, organisation name, floor title, tagline — everything on every screen updates.

### Scenario templates
Five built-in templates to start from — each with a themed floor-plan background, pre-written questions, zones, obstacles, and roles:

| Template | Setting | Roles |
|---|---|---|
| 🏫 School | Classrooms, Science Lab, Library, Playground | Student · Teacher · Prefect |
| 🏢 Office | Reception, Meeting Rooms, Open Desks, Break Room | Staff · Manager · Intern |
| 🏥 Hospital | Reception, Wards, Pharmacy, First Aid | Nurse · Doctor · Volunteer |
| 🏛️ Museum | Entrance, Art Gallery, History Hall, Gift Shop | Visitor · Curator · Guide |
| 🛍️ Shopping Mall | Entrance, Fashion, Food Court, Cinema | Shopper · Staff · Security |

Load any template in the editor, customise the questions and layout, and save — it goes live instantly.

### Live session controls (projector)
- **Open / lock joining** — control when players can enter mid-game.
- **2× points boost** — 60-second multiplier event.
- **Speed round** — 30-second timed pressure event.
- **Boss spawn** — co-op challenge: 3+ players must gather at the centre to unlock a high-stakes question.
- **End game** — push the final leaderboard to all screens.
- **New game** — clear all players and scores to run again.

### Question mechanics
- **Common** — standard, always available.
- **Rare** — higher points, standard format.
- **Elite (timed)** — configurable countdown (e.g. 15 s), extra points for speed.
- **Boss (co-op)** — locked until 3+ players gather; unlocked by the host.
- **Role bonuses** — answering a question tagged to your role earns a percentage bonus (e.g. Teacher gets +25% on teaching questions). Teammate bonus rewards players whose teammate answered right just before them.

---

## How it works

```
index.html        Landing page
/host             Host console  — auth, create/manage rooms
/game             Player view   — Phaser 3 + nipple.js (joins by room code)
/master           Projector     — Canvas2D live map + leaderboard + controls
/editor           Editor        — drag-and-drop authoring (per room, host-gated)
```

- **Frontend** — plain HTML/JS, no build step, no framework. Served as static files on **Vercel**.
- **Backend** — **Supabase** (Postgres + Realtime + Auth + Storage). A `rooms` table holds each game's full config as JSONB. `players` and `game_events` are scoped by `room_id`. Realtime subscriptions are filtered per room so rooms never see each other's traffic.
- **Auth + RLS** — hosts authenticate via Supabase Auth (email + password). Row-Level Security enforces ownership: only a room's host can edit its config or fire events (boss / end / 2×). Players can only join rooms that are open. Players never create accounts.
- **Config flow** — the editor PATCHes `rooms.config`; the player view and projector read it on load and subscribe for changes. Edits go live in seconds without any redeploy.
- **Scaling** — Vercel is a CDN (zero server overhead for static files). Supabase Realtime handles concurrent player connections. 100 concurrent player inserts tested at ~0.3 s each.

---

## Tech stack

**Phaser 3** · **nipple.js** · **Supabase** (Postgres · Realtime · Auth · Storage · RLS) · **Vercel** · vanilla JS (no framework, no bundler, no build step)
