# Jellyfin Roku Modernization Plan

## Contribution Workflow

Each item below becomes its own branch, issue comment, and PR. The Jellyfin Roku project requires:

1. **Comment on the existing issue** (or open a new one) saying you're working on it
2. **Branch from master** in your fork: `git checkout -b fix/697-login-crash`
3. **One focused change per PR** — don't bundle unrelated fixes
4. **Test on a physical Roku device** — no automated tests exist
5. **Run `npm run build`** to verify compilation and lint
6. **Submit PR** to `jellyfin/jellyfin-roku` with clear description
7. **Respond to review** from maintainers

Branch naming convention observed in the repo: `descriptive-name` (e.g., `tvShowLoadTime`, `defensiveCoding`)

---

## Phase 1: Bug Fixes (tie to existing issues)

### PR 1: Fix login crash (#697)
**Branch:** `fix-login-crash`
**Files:** `source/ShowScenes.bs`, `source/api/userauth.bs`
- Add null checks after `ServerInfo()` — `resp.GetResponseHeaders()` can return invalid, crashing on `headers.location`
- Add null check after `AboutMe()` before `session.user.Login(currentUser, true)`
- Guard server redirect URL validation

### PR 2: Fix playback loop at end (#656)
**Branch:** `fix-playback-loop`
**Files:** `components/video/VideoPlayerView.bs`
- Investigate end-of-playback state — ensure `finished` state stops rather than restarts

### PR 3: Fix forced subtitle override (#675)
**Branch:** `fix-subtitle-override`
**Files:** `components/ItemGrid/LoadVideoContentTask.bs`
- Ensure client-side "None" subtitle preference takes precedence over server defaults

### PR 4: Fix CC display regardless of settings (#626)
**Branch:** `fix-caption-mode`
**Files:** `components/video/VideoPlayerView.bs`
- Ensure app respects its own subtitle selection, not Roku system-level CC toggle

### PR 5: ASS subtitle server-side conversion (#687)
**Branch:** `fix-ass-subtitle`
**Files:** `source/utils/deviceCapabilities.bs`, `components/ItemGrid/LoadVideoContentTask.bs`
- ASS/SSA not in supported formats list — verify device profile triggers server-side conversion or burn-in

### PR 6: WebVTT display (#670)
**Branch:** `fix-webvtt`
**Files:** `components/captionTask.bs`
- Debug `parseVTT()` for edge cases causing subtitles not to render

---

## Phase 2: Performance (open new issues)

### PR 7: Home screen loading speed (#669)
**Branch:** `home-perf`
**Files:** `components/home/Home.bs`, `components/home/HomeRows.bs`
- Stagger section task loading: fire visible rows first (top 2-3), defer rest until scroll
- Add per-row loading indicator text

### PR 8: Reduce image fetch sizes
**Branch:** `image-optimization`
**Files:** `components/home/HomeItem.bs`, `source/api/Image.bs`
- Reduce default quality from 90 to 80
- Match requested image dimensions to actual display size
- Set `loadWidth`/`loadHeight` on poster components

---

## Phase 3: Visual Modernization (open new issues for each)

### PR 9: Scene fade transitions
**Branch:** `scene-transitions`
**Files:** `components/data/SceneManager.bs`
- Add ~0.3s opacity fade on `pushScene()`/`popScene()` using Roku `Animation` + `FloatFieldInterpolator`
- Pattern already exists in Home.xml button animations

### PR 10: Backdrop images on detail screens
**Branch:** `detail-backdrops`
**Files:** `components/movies/MovieDetails.xml`, `components/movies/MovieDetails.bs`, `components/tvshows/TVSeriesDetails.xml`, `components/tvshows/TVSeriesDetails.bs`
- Add full-width `Poster` using `ImageURL(id, "Backdrop")`
- Semi-transparent gradient overlay for text readability
- Fade-in on load

### PR 11: Focus scaling on home items
**Branch:** `home-focus-scale`
**Files:** `components/home/HomeItem.xml`, `components/home/HomeItem.bs`
- Scale focused item up slightly (e.g., 1.05x) for visual feedback
- Adjust row spacing to accommodate

### PR 12: Home row spacing and typography
**Branch:** `home-layout-refresh`
**Files:** `components/home/Home.xml`, `components/home/HomeRow.xml`
- Increase row title font size
- Add vertical spacing between rows
- Ties to #608 (shrink homescreen for scroll hinting)

---

## Phase 4: UX Improvements (open new issues)

### PR 13: Search debounce
**Branch:** `search-debounce`
**Files:** `components/search/SearchResults.bs`
- Add ~400ms debounce timer before firing search task
- Show "Searching..." text instead of full spinner

### PR 14: Richer search results
**Branch:** `search-results-polish`
**Files:** `components/search/SearchRow.bs`, `components/search/SearchResults.xml`
- Show type badges, year, rating
- Group results by type with section headers

### PR 15: Smoother profile switching
**Branch:** `profile-switch-polish`
**Files:** `source/ShowScenes.bs`, `source/MainEventHandlers.bs`
- Add fade transition during user switch
- Show user name during loading

---

## Phase 5: New Features (open new issues for each)

### PR 16: Playback speed control
**Branch:** `playback-speed`
**Files:** `components/video/VideoPlayerView.bs`, `components/video/VideoPlayerView.xml`
- Roku's video node natively supports `playbackSpeed` field
- Add OSD menu option with speeds: 0.5x, 0.75x, 1x, 1.25x, 1.5x, 2x
- Show current speed indicator on OSD

### PR 17: Sleep timer
**Branch:** `sleep-timer`
**Files:** `components/video/VideoPlayerView.bs`, `components/video/VideoPlayerView.xml`
- Add sleep timer option to OSD: 15 / 30 / 60 / 90 minutes / Off
- Use Roku timer node to stop playback when timer expires
- Show countdown in OSD when active

### PR 18: Jellyseerr integration
**Branch:** `jellyseerr`
**Files:** New: `source/api/jellyseerr.bs`, `components/search/RequestMedia.xml`, `components/search/RequestMedia.bs`; Modified: `settings/settings.json`, `components/search/SearchResults.bs`
- Add Jellyseerr URL + API key to settings
- When search returns 0 results, show "Request this title" button
- Request screen: search Jellyseerr's TMDB catalog, pick result, submit via `POST /api/v1/request`
- Show request confirmation/status

### PR 19: Color theme presets
**Branch:** `theme-presets`
**Files:** `source/enums/ColorPalette.bs`, `settings/settings.json`, new: `source/utils/themes.bs`
- Bundle 4-5 preset themes: Dark (default), OLED Black, Jellyfin Purple, Cool Blue, Warm Red
- Each theme swaps ColorPalette values (background, highlight, accent, text colors)
- Add theme selector in settings
- Add accent color picker for custom highlight color

### PR 20: Active sessions view
**Branch:** `active-sessions`
**Files:** New: `components/settings/ActiveSessions.xml`, `components/settings/ActiveSessions.bs`, `components/tasks/GetSessionsTask.bs`
- Query `/Sessions` API endpoint
- Show active streams: user, device, content title, transcoding status, progress
- Accessible from settings/options menu
- Auto-refresh on a timer

### PR 21: Playback quality selector (in-stream)
**Branch:** `quality-selector`
**Files:** `components/video/VideoPlayerView.bs`, `components/video/VideoPlayerView.xml`
- Add bitrate/resolution picker to OSD during playback
- Options: Auto, 720p 2Mbps, 720p 4Mbps, 1080p 8Mbps, 1080p 20Mbps, Original
- Triggers re-request to server with new bitrate cap (saves position, reloads stream)

### PR 22: "Because You Watched" / smart recommendation rows
**Branch:** `smart-recommendations`
**Files:** `components/home/HomeRows.bs`, `components/home/Home.bs`
- Add new home section type for server-provided suggestion rows
- Works with the Home Screen Sections plugin (`/Suggestions` or similar API)
- Slots into existing configurable section architecture

### PR 23: Collection/box set hero navigation
**Branch:** `collection-hero`
**Files:** New: `components/collections/CollectionView.xml`, `components/collections/CollectionView.bs`; Modified: `source/MainEventHandlers.bs`
- When drilling into a collection, show hero backdrop image at top
- Ordered movie list below with poster, title, year, watched status
- Play All / Shuffle buttons

### PR 24: Quick Connect prominence on login
**Branch:** `quickconnect-prominent`
**Files:** `components/login/ServerSelection.xml`, `components/login/ServerSelection.bs`, `source/ShowScenes.bs`
- Show Quick Connect as equal option alongside username/password on login screen
- Large QR code or prominent code display
- Follows Swiftfin's approach of first-class Quick Connect

### PR 25: Parental PIN on profile switch
**Branch:** `profile-pin`
**Files:** `source/ShowScenes.bs`, `settings/settings.json`
- Optional PIN requirement when switching to specific user profiles
- Configurable per-profile in settings
- Uses Roku's built-in PIN dialog components

### PR 26: Audio normalization indicator
**Branch:** `audio-indicator`
**Files:** `components/video/VideoPlayerView.bs`, `components/video/VideoPlayerView.xml`
- Show icon/text when server is transcoding audio
- Display reason: codec unsupported, surround downmix, etc.
- Info available from playback session data

---

## Execution Order

**Start with bug fixes** — these have existing issues, are clearly scoped, and build goodwill with maintainers before proposing UI changes.

1. PR 1 (login crash) — highest priority, blocks new users
2. PRs 3-6 (subtitle fixes) — 4 open issues, most complained-about area
3. PR 2 (playback loop)
4. PR 7 (home perf) — existing issue #669
5. PRs 9-12 (visual modernization) — open issues first, discuss with maintainers before implementing
6. PRs 13-15 (UX polish)
7. PRs 16-17 (playback speed + sleep timer) — low effort, high value
8. PRs 19, 24-25 (themes, quick connect, parental PIN) — low-medium effort
9. PRs 18, 20-21 (Jellyseerr, sessions, quality selector) — medium effort
10. PRs 22-23 (recommendations, collections) — medium effort, needs server plugin support
11. PR 26 (audio indicator) — nice-to-have polish

**For visual changes (Phase 3-5):** Open issues with mockup descriptions first and wait for maintainer feedback before coding. The Jellyfin community prefers discussion before large UI changes.

---

## Verification (every PR)

1. `npm run build` — must compile clean
2. Sideload to Roku via VSCode F5
3. Test the specific fix/feature
4. Smoke test: login → home → play video → back navigation still works
