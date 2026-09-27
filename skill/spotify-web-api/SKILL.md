---
name: spotify-web-api
description: "Best practices and workflows for the Spotify Web API. Use when building apps that interact with Spotify to: (1) Handle rate limits and 429 errors, (2) Implement efficient polling strategies, (3) Optimize data fetching, or (4) Manage authentication. Includes specific patterns for 'Smart Polling' and 'Retry-After' handling."
---

# Spotify Web API Skill

This skill encodes best practices for building robust, rate-limit-aware applications using the Spotify Web API.

## 1. Rate Limiting Strategy (Critical)

Spotify uses a dynamic, rolling 30-second window for rate limiting. There is no fixed "requests per hour" limit.

### Handling 429 Response
You **must** handle HTTP 429 (Too Many Requests) errors gracefully.

**Server-Side (Python/Spotipy):**
1.  **Do not block:** Configure the client to fail fast rather than retrying indefinitely and blocking your server threads.
2.  **Extract Header:** Capture the `Retry-After` header from the response. This integer tells you exactly how many seconds to wait.
3.  **Pass to Client:** Return this value to your frontend so the UI can inform the user.

```python
# app.py example
sp = spotipy.Spotify(auth_manager=auth, requests_timeout=10, status_retries=0, retries=0)

try:
    sp.current_user_playing_track()
except spotipy.exceptions.SpotifyException as e:
    if e.http_status == 429:
        retry_after = int(e.headers.get('Retry-After', 5)) # Default to 5s if missing
        return jsonify({"error": "Rate limit", "retry_after": retry_after}), 429
```

**Client-Side (JavaScript):**
1.  **Backoff:** If you receive a 429, respect the `retry_after` value.
2.  **User Feedback:** Display a countdown (e.g., "Retrying in X seconds...") so the user knows the app isn't broken.

## 2. Polling Best Practices

### Safe Intervals
*   **Active Tab:** Poll no faster than every **5-10 seconds**.
*   **Background:** Significantly reduce polling frequency when the user is not actively viewing the page.

### 'Smart Polling' Pattern (Page Visibility API)
Use the Page Visibility API to dynamically adjust polling rates. This saves your "request budget" for when the user is actually looking at the dashboard.

```javascript
// script.js example
let pollInterval = 10000; // 10s default

document.addEventListener('visibilitychange', () => {
    if (document.hidden) {
        console.log("Tab hidden, slowing down to 60s");
        pollInterval = 60000;
    } else {
        console.log("Tab visible, restoring to 10s");
        pollInterval = 10000;
    }
});

function poll() {
    // ... fetch logic ...
    setTimeout(poll, pollInterval);
}
```

## 3. Data Fetching Optimization

### Maximize Page Size
Always request the maximum number of items allowed per call (usually `limit=50` or `limit=100`) to minimize the total number of HTTP requests.

**Before (Inefficient):**
```python
# Default limit is often 20
results = sp.playlist_items(playlist_id) 
```

**After (Optimized):**
```python
# Fetch 100 items at once
results = sp.playlist_items(playlist_id, limit=100) 
```

## 4. Authentication

Use `spotipy.oauth2.SpotifyOAuth` for robust OAuth 2.0 flow handling.

*   **Scopes:** Be specific. Only request scopes you actually need.
*   **Token Caching:** Spotipy handles caching automatically (`.cache` file), but ensure you handle token validation and refreshing logic if building a long-running server.

```python
def get_auth_manager():
    return SpotifyOAuth(
        client_id=os.getenv("SPOTIPY_CLIENT_ID"),
        client_secret=os.getenv("SPOTIPY_CLIENT_SECRET"),
        redirect_uri=os.getenv("SPOTIPY_REDIRECT_URI"),
        scope="user-read-playback-state user-library-read",
        open_browser=False # Important for server environments
    )
```

## 5. Web API vs. Web Playback SDK

*   **Web API:** Use for **Dashboards** and **Controllers**. You want to see what is playing on *any* device or control playback remotely.
*   **Web Playback SDK:** Use for **Music Players**. You want the browser tab itself to emit sound and act as a Spotify Connect device. Do not use this just for metadata.

## 6. Playlist Ordering & End-to-End Verification (added 07-19-26)

### Add-order IS playlist order
*   `POST /playlists/{id}/tracks` (spotipy `playlist_add_items`) with `position` omitted **appends URIs in the exact order given**. Sequential chunked adds (max 100 URIs per call) preserve global order across chunk boundaries.
*   Therefore, to control a playlist's final order: **sort the source data before export**. No post-hoc reorder calls are needed.
*   In-app sort toggles ("Recently added", etc.) are **view-level only** — the API sets the canonical custom order, which is what clients display by default.

### Verify all the way (don't stop at code reading)
*   When a change claims to affect what Spotify receives (order, content, chunking), prove it by **executing the real export code path against a stubbed client**: inject a fake `spotipy` module via `sys.modules` **before importing** the exporter, record every `playlist_add_items` call, then assert the exact URI order and chunk sizes (e.g. 121 tracks → chunks of 100 + 21).
*   This exercises the real code end-to-end — CSV parsing, match loop, chunked add loop — without creating real playlists or touching OAuth. **Never create a live playlist just to check a behavior.**
*   In your summary, distinguish what was **proven by execution** from what rests on **documented API behavior** — and verify the latter against docs or production evidence, not memory.

## 7. Perceived-Latency Killers for Multi-Page Dashboards (added 07-21-26)

Proven in Your App Name: page switches went from seconds of "Loading..." to instantaneous. Three independent patterns, apply all of them:

### Short-TTL server-side cache for "currently playing"
Every full page load re-polling Spotify makes navigation feel like the track must be "re-recognized" — even though it's the same track. Cache the assembled current-track payload server-side for ~5s (`{"payload": ..., "ts": time.time()}`); page switches then answer in ~1–2ms instead of ~350ms+ and cost **zero** Spotify calls.
*   Cache the "nothing playing" answer (`None`) too — distinguish "cached None" from "no cache" via the timestamp, not the payload.
*   **Invalidate on self-initiated state changes** (repeat toggle, like/unlike via playlist add) or the user's own action shows stale for up to a TTL.
*   Keep TTL below the client poll interval so steady-state freshness is unchanged.

### Never serialize startup behind slow warm-up
Don't `await` a slow bulk fetch (all user playlists) before the first `current-track` poll — run them in parallel and let each UI region show its own loading placeholder. Related: a `/health` endpoint gating an app's loading screen must return 200 as soon as the server responds, not after warm-up completes; report warm-up separately (e.g. an `X-Loading-State` header).

### Optimistic instant render from localStorage
On every successful poll, persist `{ts, track}` to localStorage. On page load, if the saved track is <60s old, render it immediately (header, artwork, playlist checks) and let the first poll confirm or correct. The track playing 200ms ago is almost certainly still playing — perceived latency drops to zero.
*   Any "UI is ready" flag consumed by a native wrapper (e.g. `window.__trackReady`) must be set **unconditionally after each settled poll**, not inside a did-anything-change guard — an optimistic pre-render makes the guard skip and would starve the flag.
*   The same localStorage trick applies to **every** region, not just the track header: persist the playlist/tile list per page and the current track's membership ids, render them instantly on load, and let the live fetch correct. Guard the correction path: an **empty interim response must never clobber an already-rendered cached list** (assign fetched data to the live variable only when non-empty).

## 8. Membership Caches: Persist Across Restarts, Never Pre-Seed Empty (added 08-28-26)

Proven in Your App Name. There is no "which of my playlists contain track X" endpoint, so dashboards keep a server-side `{playlist_id → set(track URIs)}` cache warmed by fetching each playlist (~2s apart for rate limits — ~80 playlists ≈ 3 minutes). Rules learned the hard way:

*   **Persist the warm cache to disk and load it at startup.** Otherwise every launch spends the whole warm-up answering "this track is saved nowhere" — the user sees their saved track unrecognized. Save per-playlist during warm-up (a mid-warm-up quit still leaves the next launch mostly warm) and write-through on every add/remove the app itself performs. Atomic write via temp+`os.replace`, but `os.path.realpath()` FIRST when the file may be a symlink (shared runtime file across git worktrees) — a plain rename replaces the symlink with a local file and silently unshares it.
*   **Cache-dict membership must mean "real data".** Pre-seeding `cache[pid] = set()` at startup so lookups don't KeyError silently converts "not fetched yet" into "definitely not in this playlist" and disables any live-check fallback — the exact launch bug. Guard mutation sites with `if pid in cache` instead, and let absent keys route to a (bounded!) live check — cap it (~8 playlists/request); a first-ever run with nothing cached must not burst 80+ playlist fetches.
*   **Tell the client when answers are provisional.** Return an `X-Cache-State: warming|ready` header (`warming` while config/pages are still resolving, any warm-up thread is running, or live checks were skipped); the client re-checks every ~10s while warming so tiles self-correct as fresh data lands — external changes made while the app was closed converge without waiting for a track change.
*   **Answer from the persisted cache even before config resolves.** In the first seconds after launch the server may not have resolved which playlists each page shows; if the request arrives then, scan the persisted cache directly rather than returning `[]` — extra ids for no-longer-configured playlists are harmless to a client that matches ids against rendered tiles.

## 9. Playlist writes in 2026, borrowed tokens, and "the playlist playing now" (added 09-19-26 / 09-20-26)

Proven building the Keyboard Maestro "move selected tracks" script (`spotify_move_selected.py`, Keyboard Maestro repo) and migrating the Your App Name daemon.

*   **Playlist contents are `/playlists/{id}/items`; `/tracks` is deprecated.** *Trigger:* any code that reads, adds to or removes from a playlist — `grep -rn "playlists/.*tracks" <repo>`. *Discrimination:* `/me/tracks` (the Liked Songs library) is a different, current endpoint — leave it. *Action:* GET `/items` (each entry carries `item` AND `track` — read `item`, fall back to `track`); POST `/items` with `{"uris":[…]}` → 201; DELETE `/items` with `{"items":[{"uri":…}]}` → 200 (the legacy DELETE body key was `tracks`); still 100 per call; the playlist object's count is `items.total` (then `tracks.total`). Try `/items` first and retry `/tracks` only on a 404. Verified live 09-19-26.
*   **A second tool borrows the owner's token — it NEVER refreshes it.** *Trigger:* a script or macro needs Spotify access on a machine where a long-running daemon already holds the OAuth login (`~/Library/Application Support/Your App Name Daemon/token.json`). *Discrimination:* running your own refresh looks harmless but refresh tokens rotate — the daemon's copy dies and the whole dashboard signs out. *Action:* read `access_token`; when `expires_at` is under 60 s away (or on a 401), poke the daemon so IT refreshes (`GET 127.0.0.1:8877/playlists?refresh=1`), then re-read the file; if the daemon is down, `launchctl kickstart gui/$UID/com.example.spotify-dashboard.daemon`. The daemon's `/toggle-track` is not a clean write: it also Likes on add and un-Likes on remove.
*   **"The playlist playing now" needs three guards.** (1) `GET /me/player` returns **204** once playback has been paused a while — fall back to the contexts in `GET /me/player/recently-played?limit=50`, most recent first, capped (6). (2) **Skip playlists the user does not own** (`owner.id` vs `GET /me`): nothing can be removed from them, and reading one in full is ruinous. (3) **An owned playlist whose id starts `37i9dQZF1` is Liked Songs wearing a playlist URI** (13,809 tracks on 09-20-26): reading it cost 139 requests and 18 s — ask `GET /me/tracks/contains?ids=` (50 per call) instead and remove with `DELETE /me/tracks`. *Self-check:* trace one cold run's request list; more than ~25 requests for a small selection means a guard is missing.
*   **Fast membership reads without going stale.** Page 1 returns `total`; fetch the remaining pages in parallel (6 workers took a 1,153-track playlist from ~7 s to ~2 s). Keep a local URI set per playlist keyed by `snapshot_id` (`~/Library/Caches/<tool>/<playlist>.json`): an unchanged snapshot skips the read entirely, a changed or missing one only costs a re-read — it can never serve wrong contents. After your own add, store `existing ∪ added` under the `snapshot_id` the POST returned. Do not pass `market` when comparing URIs copied from the desktop app — relinking changes them.
*   **Move order is the OWNER's call — and either order needs its safety net (revised 09-20-26).** *Trigger:* a tool moves tracks between playlists. *Discrimination:* add-first can only ever leave a duplicate, but the visible feedback (the track vanishing from the list he is looking at) arrives LAST — the user asked for "the first step is deleting the track from the current playing playlist" (macro `FF98B146-…`). *Action:* remove-first is fine when: (1) what is about to be removed is appended to the move log BEFORE the DELETE (the undo list survives a crash); (2) a failed add PUTS THE TRACKS BACK (`POST …/items` to the source — they land at its end; `PUT /me/tracks` for Liked Songs, idempotent) and the message says so; (3) the destination's duplicate read runs in a background thread started before the source lookup, so the removal never waits for it. Never block the caller on a courtesy call afterwards (a cache-refresh ping to the daemon took 3 s: detach it).

## 10. A WRITE is not visible to the next READ — hold what you just set (added 09-20-26, Your App Name)

*   **Trigger:** code that does `PUT /me/player/<something>` (repeat, shuffle, play / pause, volume, seek) and then polls `GET /me/player` "to confirm", or a user report shaped like "I press it, it changes, then it flips back" / "sometimes the button doesn't work". In a log, the signature is the SAME command fired twice 5 – 10 s apart (he pressed again).
*   **Discrimination:** it is not a failed request (the PUT answered 2xx) and not a paused-device problem — the player endpoint simply lags its own writes by up to several seconds, exactly as it lags the desktop client after a skip. An optimistic UI with a SHORT fixed lock (2.5 s) makes it worse: the lock expires, the stale poll wins, and the truth only returns with the next heartbeat. Where a push channel exists (the desktop app's `PlaybackStateChanged` notification covers play / pause on the same Mac) the stale read is corrected within milliseconds and the bug hides — check whether the field you changed is IN that payload before assuming it is covered (repeat is not).
*   **Action — two guards, no extra requests, never a tighter poll:** (1) the process that made the write HOLDS the written value for ~6 s: a poll inside the window that contradicts it keeps the held value (one small `(state, until)` slot, the same one used for hints from outside); (2) the UI lock is CONFIRMED, not timed — show the requested state for at least ~4 s, then until a poll agrees, at most ~14 s (one heartbeat + slack); a newer press replaces the lock, a failed request clears only its OWN lock, and an announced change that contradicts the lock (another client) clears it. Toggle from what the UI SHOWS, not from the last poll, or a second press sends the opposite command. Test with the transport stubbed so the fake API stays stale for 6 s — never against live playback.

## 11. 403 "Restriction violated" on next / previous means "there is none" — not a failure (added 09-19-26, Your App Name)

*   **Trigger:** `POST /me/player/previous` or `/next` answers `403` with `Player command failed: Restriction violated`; the user report is a red error after pressing Previous on the first track of an album, a single, or a fresh context.
*   **Discrimination:** the same 403 family covers Premium-only and no-active-device errors — read the message: "Restriction violated" is Spotify saying the CONTEXT has no track in that direction. It is not an auth problem, and retrying cannot succeed.
*   **Action:** treat it as a normal outcome. PREVIOUS → `PUT /me/player/seek?position_ms=0` (what Spotify's own button does there), silently — the play head jumping to 0:00 is the feedback, and the user had a "Restarted the track" toast removed as noise. NEXT → a neutral "No next track" message, never the red error style. Every other failure keeps the error style. Test by stubbing the client's POST to reject with that exact text — never against the live player.

