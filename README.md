# Butcher's Kitchen — The After Dark Experience

A cinematic, scroll-driven website for **Butcher's Kitchen**, Budapest (Tűzoltó u. 22).
React + Vite + Tailwind 4 · three.js (custom GLSL) · GSAP ScrollTrigger · 4 languages (EN / HU / FA-RTL / DE).

## What is in the box
| Area | Implementation |
|---|---|
| Language gate | First screen, cards for English / Magyar / فارسی / Deutsch, saved in `localStorage` (`bk.lang`), switcher in the nav, full RTL for Persian |
| Opening shot | Timed dolly + letter-by-letter title on the night exterior, letterbox bars, **Skip intro** |
| Scroll film | 7 scenes (Entrance → Kitchen → Fire → Craft → Signature food → Table → Final CTA). A fixed WebGL canvas travels through the film while GSAP/ScrollTrigger drives an inertia-smoothed `store.scene` value |
| Shader work | Rack-focus depth of field on transitions, heat-haze + flame flicker on the grill, rain-on-glass droplets, wet-pavement ripples, sun shafts, grade per atmosphere; 3D-parallax dust, embers, smoke/mist, rain streaks |
| Atmosphere | Day / Evening / Night / Rain from **real Budapest time + Open-Meteo weather** (keyless). Falls back to time-only if the API is unreachable. The nav "Atmosphere" menu is the development simulation mode (`?atmosphere=rain` also works) |
| Sound | Off by default. Synthesised (Web Audio) room tone, fire crackle, rain — no audio files |
| Food | 11 popular dishes, cinematic wipe transitions, hover loupe zoom, keyboard arrows. Price / ingredients / allergens show **"to be confirmed"** until you fill them in |
| Reservations | Custom calendar + time grid, validation, loading/error/confirmation states, status lookup by reference |
| The Butcher's Guide | Chat concierge in 4 languages. Real AI through the backend proxy; built-in labelled **demo mode** otherwise |
| Resilience | Loading screen with progress, `prefers-reduced-motion`, mobile particle/DPR limits, WebGL-less CSS fallback, error boundary |

## Honest status / limitations
* **No Blender assets are shipped.** Blender wasn't available while building. The "photoreal" look comes from **AI-generated image plates** (`src/assets/img/*.jpg`) composited by real-time shaders. They are *illustrative* — not photographs of the restaurant or its dishes. Replace them with real photography/renders (same file names) before launch. See `blender/README.md` for the intended 3D pipeline and an untested export script.
* Day/Evening are **grades of the night plates** (exposure, tint, sun shafts), not separately shot daylight scenes.
* The static build (`dist/index.html`, single file) has **no backend**. Without `VITE_API_BASE_URL`, reservation requests are *not sent* — the UI says so, saves a draft in the browser and shows the phone number. Nothing is ever shown as "confirmed" unless the restaurant sets it via the admin API.
* Unknown facts are `null` in `src/content/` (opening hours, prices, ingredients, allergens). The coordinates in `restaurant.ts` are approximate and only used for the weather lookup. The booking time slots are placeholders, not opening hours.
* Persian/Hungarian/German copy was hand-written, but have a native speaker review it before launch (all strings are in `src/translations/*.ts`).
* Verified in headless Chromium with software WebGL (all screens, scroll film, forms). Not tested on physical mobile GPUs.

## Install & develop
```bash
npm install
npm run dev            # http://localhost:5173
npm run build          # → dist/index.html (single file, assets inlined)
npm run preview
```

## Backend (reservations + AI proxy)
Zero dependencies, Node ≥ 18.
```bash
cp .env.example .env   # fill in, then export the variables (or use your host's env settings)
cd backend && node server.js
```
| Endpoint | Purpose |
|---|---|
| `POST /api/reservations` | Create a request → `{ref, status: "pending_confirmation"}` |
| `GET /api/reservations/:ref` | Public status only (no personal data) |
| `GET /api/admin/reservations?status=&date=` | List (header `X-Admin-Token`) |
| `PATCH /api/admin/reservations/:ref` | `{status: "confirmed" \| "declined" \| "cancelled", note}` — the confirmation workflow |
| `POST /api/assistant` | AI chat; key stays server-side. No key → `{mode:"demo"}` |

Reservations live in `backend/data/reservations.json` behind a small store interface (`backend/reservations/store.js`); `schema.sql` is the SQLite/Postgres equivalent. Set `RESERVATION_WEBHOOK_URL` to get each request in Slack/Discord/email automation. Then build the frontend with `VITE_API_BASE_URL=https://api.your-domain npm run build`.

## Editing content
* Facts: `src/content/restaurant.ts` (and mirror in `backend/ai/knowledge.js`)
* Dishes (price, ingredients, allergens, images): `src/content/menu.ts`
* All copy, 4 languages: `src/translations/{en,hu,de,fa}.ts`
* Look & feel: `src/index.css` (theme tokens), grades in `src/state/experience.ts`, shaders in `src/scenes/shaders.ts`

## Structure
```
src/components/{Navigation,LanguageSelector,Intro,ScrollExperience,FoodMenu,Reservation,AIConcierge,Contact,common}
src/scenes/      CinematicRenderer (three.js), shaders, textures
src/services/    api, reservations, assistant (+demo), weather, audio
src/state/       mutable experience store (scroll/scene/grade)
src/animations/  GSAP + ScrollTrigger helpers
src/translations/  en hu de fa
backend/{api,reservations,ai}   blender/{models,materials,textures,scenes,exports,scripts}
```
