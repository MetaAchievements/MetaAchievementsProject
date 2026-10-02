# The Meta Achievements Project

The Meta Achievements Project is a comprehensive ecosystem designed to restore and enhance achievement tracking for Meta Quest users. Founded by DenHBR and developed on top of by Not a Glitch Studios, these tools provide a high-fidelity alternative to the deprecated official Meta scoreboard.

---

## 🌐 Meta Achievements Ecosystem

This repository is the central public hub of the **[Meta Achievements Organization](https://github.com/MetaAchievements)**:

### Public Repositories
- 🌐 **Main Project Hub**: [MetaAchievementsTracker](https://github.com/MetaAchievements/MetaAchievementsTracker) *(This Repo)*
- 📄 **XML Test Viewer**: [MetaAchievementsXMLTest](https://github.com/MetaAchievements/MetaAchievementsXMLTest) ([xml.meta-achievements.org](https://xml.meta-achievements.org))

### Private Repositories
- 💻 **Web Portal & Server**: [MetaAchievementsHTML](https://github.com/MetaAchievements/MetaAchievementsHTML) ([meta-achievements.org](https://meta-achievements.org))
- 📱 **Android App**: [MetaAchievementsTrackerAndroid](https://github.com/MetaAchievements/MetaAchievementsTrackerAndroid)
- 📱 **iOS App**: [MetaAchievementsTrackerIOS](https://github.com/MetaAchievements/MetaAchievementsTrackerIOS)
- ⚙️ **Definitions Scraper**: [Achievement-Definitions-Grabber](https://github.com/MetaAchievements/Achievement-Definitions-Grabber)

---

| platform | build | publish                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| -------- | ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Android  | ✅     | [<img src="https://storage.googleapis.com/docs.itch.ovh/brand/rf/assets/badges/badge_bw.png" alt="Avaialable on Itch.io" height="80">][itchio-myapp] [<img src="https://raw.githubusercontent.com/Kunzisoft/Github-badge/main/get-it-on-github.png" alt="Get it on GitHub" height="80">][github-myapp]                                                                                                                                                                                                                                                                                                                                  |
| iOS      | ✅     | [<img src="https://raw.githubusercontent.com/FriesI23/altstore-repo/refs/heads/master/assets/get-it-on-altstore-org.png" alt="Get it on AltStore" height="80">][altstore-source] [<img src="https://raw.githubusercontent.com/FriesI23/altstore-repo/refs/heads/master/assets/get-it-on-sidestore-org.png" alt="Get it on SideStore" height="80">][sidestore-source] [<img src="https://storage.googleapis.com/docs.itch.ovh/brand/rf/assets/badges/badge_bw.png" alt="Avaialable on Itch.io" height="80">][itchio-myapp] [<img src="https://raw.githubusercontent.com/Kunzisoft/Github-badge/main/get-it-on-github.png" alt="Get it on GitHub" height="80">][github-apple] |
| Web    | ✅     | [<img src="https://meta-achievements.notaglitch.net/web.png" alt="Get it on the Web" height="80">][website] |
| Quest  | ✅     | [<img src="https://img.itch.zone/aW1nLzI2NTU1MjEwLnBuZw==/original/W69aa5.png" alt="Get it on SideQuest" height="80">][sidequest-myapp] [<img src="https://storage.googleapis.com/docs.itch.ovh/brand/rf/assets/badges/badge_bw.png" alt="Avaialable on Itch.io" height="80">][itchio-myapp][<img src=https://i.imgur.com/FpfNxjX.png>][meta-rc] |

[Status Page](https://meta-achievements.onlineornot.com/)

---

## The Web App

A robust webapp that provides all information you could possibly need about your played games and unlocked achievements on the Meta ecosystem

## The Achievement Tracker App

A standalone Android application (APK) designed for phones, tablets, and Meta Quest headsets (via sideloading).

Also available; An IOS IPA only available through sideloading.

Brings the full power of the Meta Achievements Project directly into the VR environment. It features a native UI optimized for both touchscreen and Quest controller input, allowing users to check their trophy progress without leaving their headset.

---

## Project Attribution

Host Organization: [Not a Glitch Studios](https://www.notaglitch.net/)

Lead Developer (GUI & App): [TheAndromedaCat](https://www.andromedacat.net/)

API and Original Website Development: [DenHBR](https://go.meta-achievements.org/render)

## Public DB Data API (Unauthenticated & CORS Allowed)
These endpoints allow third-party applications, developers, and browser users to query the local SQLite achievement database directly:

GET /api/public/all-definitions (or /api/public/all-achievement-definitions):

Purpose: High-performance single-request bulk definitions endpoint returning all indexed games along with their achievement definition arrays in one payload.
Behavior: Unconditionally returns all games (including hidden, delisted, and merged alias games) with their status metadata (is_hidden, is_delisted, merged_into, scrape_status, achievement_count, platform, platform_plaque, platform_type). Merged alias games automatically inherit achievement definitions from their master app ID. Merged games do not carry the is_delisted tag unless their master ID or the alias itself is delisted.
Platform Metadata: Reports platform classification resolved from platforms.json (platform [array of all available platforms], platform_plaque [earliest supported platform generation for public plaque], platform_type: "quest" | "rift" | "go" | "gearvr").
Caching: Supports HTTP ETag and conditional requests (If-None-Match -> 304 Not Modified). Responses include Cache-Control: public, max-age=60, stale-while-revalidate=120.
Format: JSON array of game objects (application/json).
CORS: Access-Control-Allow-Origin: *
GET /api/public/apps:

Purpose: Returns a JSON list of all games in the database, their titles (app_name), scrape status, hidden/delisted status (is_hidden, is_delisted), merge target (merged_into), total achievement counts (achievement_count), platform tags (platform, platform_plaque, platform_type), and last_scraped_at timestamps.
Behavior: Unconditionally includes all games and merged aliases so 3rd parties can query complete metadata. Merged games are only marked is_delisted: true if their master app or alias app is marked delisted.
Format: Pretty-printed JSON (application/json) with { total_apps: number, apps: [...] }.
CORS: Access-Control-Allow-Origin: *
GET /api/public/search-app?name=... (or /api/public/search-app/:query):

Purpose: Case-insensitive search of the /public/apps database by app_name, returning matching app_ids, names, scrape status, hidden/delisted status, merge targets, achievement counts, platform tags, and last_scraped_at timestamps.
Format: Pretty-printed JSON (application/json).
CORS: Access-Control-Allow-Origin: *
GET /api/public/achievements/:appId:

Purpose: Returns full achievement definitions for a single appId, automatically resolving merged alias apps to their master app definitions and attaching platform tags.
Format: JSON array of achievement objects (application/json).
CORS: Access-Control-Allow-Origin: *
Username-to-ID Proxies
GET /api/public-profile/:username/user-name (Returns plain text)
GET /api/public-profile/:username/get_active_achievements_count
GET /api/public-profile/:username/get_meta_achievements_count (Redirected to get_saved_achievements_count internally)
GET /api/public-profile/:username/get_saved_achievements
GET /api/public-profile/:username/save_new_achievements/:count
GET /api/public-profile/:username/deactivate_achievements

[itchio-myapp]: https://go.meta-achievements.org/itch
[sidestore-source]: https://meta-achievements.notaglitch.net/sidestore.html
[github-myapp]: https://github.com/TheAndromedaCat/MetaAchievementsTracker/releases/latest
[altstore-source]: https://meta-achievements.notaglitch.net/altstore.html
[website]: https://go.meta-achievements.org/
[sidequest-myapp]: https://go.meta-achievements.org/sidequest
[meta-rc]: https://go.meta-achievements.org/meta
[github-apple]: https://github.com/MetaAchievements/MetaAchievementsProject/releases/tag/A260901394-I260901503
