# Meta Achievements Public API Reference

- **Production Base URL**: `https://meta-achievements.org`
- **XML Profile Viewer / Tester**: [`MetaAchievementsTester`](https://github.com/MetaAchievements/MetaAchievementsTester) ([xml.meta-achievements.org](https://xml.meta-achievements.org))
- **CORS Policy**: All endpoints send `Access-Control-Allow-Origin: *` (`Access-Control-Allow-Methods: GET, POST, OPTIONS, DELETE`, `Access-Control-Allow-Headers: Origin, X-Requested-With, Content-Type, Accept, X-API-Key`).
  > **Note for browser `fetch()` callers**: Only send simple GET requests without extra request headers (such as a manual `If-None-Match` header) when calling cross-origin from external websites so browsers use their native HTTP cache cleanly without triggering a rejected CORS preflight.

---

## 1. Public User XML API (Unauthenticated & CORS Allowed)

- **`GET /api/userXML/:filename`** (e.g., `GET /api/userXML/theandromedacat.xml` or `/api/userXML/TheAndromedaCat`)
  - **Purpose**: Serves a player's full exported achievement profile XML (profile stats, tracked/installed games, achievement metadata, unlock timestamps, rarity scores, and icon URLs) in a single HTTP request.
  - **Behavior**:
    - Opt-in per user (`enablePublicXmlApi`); hidden/private profiles (`profileHidden: true`) return `404`.
    - Stored canonically under lowercase filenames and resolved case-insensitively.
    - Generates XML server-side on demand if the user has Public XML API enabled and the static file is not yet cached on disk.
    - Returns `Content-Type: application/xml; charset=utf-8` with standard `ETag` and `Last-Modified` headers (`Cache-Control: public, max-age=0`).
  - **CORS**: `Access-Control-Allow-Origin: *`

---

## 2. Public DB Data API (Unauthenticated & CORS Allowed)

These endpoints allow third-party applications, developers, and browser tools to query the indexed achievement database directly without an API key:

- **`GET /api/public/all-definitions`** (aliases: `/api/public/all-achievement-definitions`, `/api/db/all-definitions`)
  - **Purpose**: High-performance catalog & achievement definitions endpoint supporting both **server-paginated slices** and **full bulk dumps** with gzip compression.
  - **Query Parameters**:
    - `page` & `limit` (or `perPage`): Enables paginated mode for fast UI streaming. When either is supplied, returns `{ total_apps, total_pages, page, limit, summary, apps }` directly from in-memory cache or indexed SQLite. When omitted, returns the full array of indexed games.
    - `sort`: `alpha` (default), `reverse-alpha`, `most-played`, `highest-score`, `lowest-score`, `most-achievements`, `least-achievements`, `recent-added` (or `recently-added`), `recent-updated` (or `recently-updated`), `recent-achievements` (or `latest`).
    - `search` (or `q`): Case-insensitive substring filter on `app_display_name` or `app_id`.
    - `includeHidden=true`: Include hidden/delisted/merged apps in paginated results (full unpaginated bulk mode includes status metadata unconditionally so callers can filter locally).
  - **Behavior & Metadata**: Merged alias games inherit achievement definitions from their master app ID (`merged_into`). Includes `is_hidden`, `is_delisted`, `merged_into`, `scrape_status`, `achievement_count`, versioned `app_icon_url`, and platform metadata (`platform`, `platform_plaque`, `platform_type`: `quest`, `quest1`, `quest2`, `questpro`, `quest3`, `quest3s`, `rift`, `go`, `gearvr`, or `mobile`).
  - **Caching**: Supports HTTP `ETag` / `304 Not Modified` in bulk mode (`Cache-Control: public, max-age=60, stale-while-revalidate=120`) and `Cache-Control: public, max-age=30, stale-while-revalidate=60` in paginated mode, plus proxy `gzip` compression when `Accept-Encoding: gzip` is present.
  - **CORS**: `Access-Control-Allow-Origin: *`

- **`GET /api/public/apps`**
  - **Purpose**: Returns the list of indexed games, titles (`app_name`), scrape status, `is_hidden`, `is_delisted`, `merged_into`, `achievement_count`, platform tags (`platform`, `platform_plaque`, `platform_type`), and `last_scraped_at` timestamps. Pass `?includeHidden=true` for full hidden/merged coverage.
  - **Format**: JSON object (`application/json`) `{ total_apps: number, apps: [...] }`.
  - **CORS**: `Access-Control-Allow-Origin: *`

- **`GET /api/public/search-app?name=...`** (or `/api/public/search-app/:query`, also accepts `?q=...`)
  - **Purpose**: Case-insensitive search across indexed apps by name, returning matching `app_id`s, names, status metadata, achievement counts, platform tags, and `last_scraped_at` timestamps. Pass `?includeHidden=true` to include hidden/delisted entries.
  - **Format**: JSON (`application/json`).
  - **CORS**: `Access-Control-Allow-Origin: *`

- **`GET /api/public/achievements/:appId`**
  - **Purpose**: Returns full achievement definitions for a single `appId`, resolving merged alias apps to their master app definitions and attaching platform metadata.
  - **Format**: JSON (`application/json`).
  - **CORS**: `Access-Control-Allow-Origin: *`

---

## 3. Public Users, Profiles, Avatars & Leaderboards (Unauthenticated & CORS Allowed)

- **`GET /api/public/user-count`**
  - **Purpose**: Returns `{ totalRegistered: number }` excluding internal error records.
  - **CORS**: `Access-Control-Allow-Origin: *`

- **`GET /api/public/user-count-badge.svg`**
  - **Purpose**: Returns an SVG badge displaying the total registered user count (`Content-Type: image/svg+xml`).

- **`GET /api/public/profiles`**
  - **Purpose**: Returns `{ total: number, profiles: string[] }` for all public profiles, excluding private (`profileHidden: true`) and error records.
  - **CORS**: `Access-Control-Allow-Origin: *`

- **`GET /api/public/users`** (alias: `/api/public/search-users`)
  - **Purpose**: Directory/search across public profiles with cached completion stats. Supports optional `?sort=unlocked|games|completions|score|alpha`.
  - **Format**: `{ total, totalPublic, totalRegistered, users: [...] }`.
  - **CORS**: `Access-Control-Allow-Origin: *`

- **`GET /api/public/staff-badges`**
  - **Purpose**: Returns configured staff role titles, badge colors, linked player IDs, and hashed identifiers (`application/json`).
  - **CORS**: `Access-Control-Allow-Origin: *`

- **`GET /api/profile-avatar/:username`**
  - **Purpose**: Serves a player's uploaded or cached avatar image (`GET`).

- **`GET /api/leaderboards`**
  - **Purpose**: Returns global leaderboard rankings. Supports `?board=` (or `?leaderboard=`): `ACHIEVEMENTS_GAINED` (default), `100PERCENT_CLUB`, `GAMES_PLAYED`, or `RARITY_SCORE`. Respects hidden profiles (returned anonymized).

---

## 4. Public Community Guides, Walkthroughs & Updates (`GET`, Unauthenticated)

- **`GET /api/games/recent-updates`** — Recently added games and recently updated achievement definitions feed.
- **`GET /api/games/all-guides-counts`** — Summary map of community guide counts across all games and achievements (`{ success, totalGuides, byApp, byAchievement, counts }`).
- **`GET /api/games/:appId/guides-summary`** — Guide counts grouped by achievement ID for a single game (automatically resolves merged aliases).
- **`GET /api/games/:appId/achievements/:achievementId/guides`** — Lists community guides and comments for a specific achievement sorted by vote score.
- **`GET /api/games/:appId/walkthrough`** — Overview and chapter list for a game's published walkthrough hub.
- **`GET /api/games/:appId/walkthrough/page/:pageNumber`** — Content and prev/next navigation metadata for a single published walkthrough chapter.

---

## 5. Public Username-to-ID Profile Proxies (Unauthenticated & CORS Allowed)

These routes resolve a public username (or user identifier) server-side so callers never need a player's internal authentication ID:

- `GET /api/public-profile/:username/profile-status` — Public display preferences, theme/language settings, bio, homepage, `enablePublicXmlApi`, and `isStaff` flags (returns `404` if username is not cached).
- `GET /api/public-profile/:username/tracked-games` — Resolved list of Master App IDs synced from the user's headset library (`{ success, username, trackedGames }`).
- `GET /api/public-profile/:username/beta-status` — Returns `{ betaEnabled: boolean }`.
- `GET /api/public-profile/:username/user-name` — Returns plain-text display username.
- `GET /api/public-profile/:username/get_active_achievements_count` — Returns JSON active achievement count (`{ ActiveAchievementsCount: number }`).
- `GET /api/public-profile/:username/get_saved_achievements_count` (alias: `get_meta_achievements_count`) — Returns saved achievements count (plain text/JSON depending on historical sync presence).
- `GET /api/public-profile/:username/get_saved_achievements` — Returns the player's full JSON array of unlocked/tracked achievement records (including merged historical syncs).
- `GET /api/public-profile/:username/save_new_achievements/:count`
- `GET /api/public-profile/:username/deactivate_achievements`

---

## 6. Official Web / API-Key Proxies (`X-API-Key` or Official Origin Required)

These internal/app endpoints require either an official `*.meta-achievements.org` browser `Origin`/`Referer` or a valid `X-API-Key` (external browser callers receive `403 Unauthorized` — third parties should use `/api/public/achievements/:appId`, `/api/public/all-definitions`, or `/api/userXML/:username.xml` instead):

- **`GET /api/getAchievementDefinitions/:appIds`**
  - Batch achievement definition lookup accepting up to ~100 comma-separated App IDs per call, gzip-compressed when supported (`Accept-Encoding: gzip`). Supports `?localOnly=true` (skip upstream Render fallback) and `?priority=urgent` (prioritize single-app scrape lookups).
- **`GET /api/getAllApps`** — Normalized app catalog array from local grabber / upstream Render.
- **`GET /api/getUser/:userId`** — Proxied user lookup.
- **`GET /api/merged-apps`** — Returns `{ success, count, mergedApps, map }` mapping alias App IDs to master App IDs.
