# Watching and finding replays — reference

Deep detail for the player and the find/filter surfaces. See SKILL.md for the overview.

## Table of contents
- [Player controls and shortcuts](#player-controls-and-shortcuts)
- [The Activity / inspector panel](#the-activity--inspector-panel)
- [Capturing moments](#capturing-moments)
- [Filtering a result list](#filtering-a-result-list)
- [Saved filters (dynamic)](#saved-filters-dynamic)
- [Collections (curated)](#collections-curated)
- [Built-in collections](#built-in-collections)
- [Console log search syntax](#console-log-search-syntax)

## Player controls and shortcuts

When watching a single replay you can:
- Watch one window or follow the user across windows.
- Share private/public links; add to a collection.
- Open the **Activity** panel (events, network waterfall, performance score, asset breakdown).
- **Inspect DOM** — interact with a snapshot of the page (approximate; use browser dev tools for precision).
- **View heatmap** — uses the recording's HTML as the heatmap background.
- Change playback speed; toggle **Skip inactivity** in the `...` menu.
- Toggle **Floating controls** (overlay on hover) vs **Pinned controls** (always visible); preference persists.
- Play/pause/seek; hold `alt` for 1ms-step seeking.
- AI summary segment markers on the timeline: green = success, red = failure; click to jump.
- Switch timestamps between relative seconds, UTC, and project timezone.

Keyboard shortcuts: `C` comment, `e` emoji reaction, `s` screenshot, `x` clip, `n` next recording, `t` cinema mode, `f` fullscreen.

## The Activity / inspector panel

A browser-DevTools-like suite synced to the timeline. PM uses:
- Jump to any event in the session timeline.
- See console logs, warnings, and errors exactly when they happened.
- Inspect the **network waterfall** to find slow or failed API calls — the usual culprit behind silent friction.

## Capturing moments

- **Screenshots** (`S` / camera icon): capture a UI state to share.
- **Short clips** (`X` / clip icon): 5, 10, or 15 seconds as GIF or MP4.
- Both sit in the player's bottom-right. Attach to bug reports (Jira, Linear), team discussions, or docs.
- Comments can be turned into a **task** via **Add as task** to track the fix.

## Filtering a result list

When viewing a filter result or collection you can:
- Open the filter panel to find more recordings.
- Ask PostHog AI to find recordings (see ai-and-mcp.md).
- Show all vs hide already-watched recordings.
- Switch time display (relative / UTC / project timezone).
- Sort by timestamp, activity, errors, and more.
- Set autoplay (off / next newest / next oldest).

Filter dimensions include: date, active duration, events, person and group properties, console logs, feature flags, rage clicks, and errors.

## Saved filters (dynamic)

A living list — new matching replays are added automatically.
1. **Show filters** and build the filter.
2. **Add to "Saved filters"**.
3. It appears under the **Shared filters** tab for everyone in the project, and as a searchable category in the filter dropdown.

Edit without recreating: load it, click the pencil to rename; click an event filter -> **Change event** to swap events.

## Collections (curated)

A permanent, hand-picked list for "watch later", focused analysis, or sharing.
- `+ Add to collection` next to **Share** on a replay; search or create a collection, then select it.
- Manage and share from the [collections page](https://app.posthog.com/replay/playlists) (link is accessible to anyone with project access).

Saved filter vs collection: saved filter = dynamic query that keeps filling; collection = static curated set you control.

## Built-in collections

Auto-maintained, available to everyone, update dynamically:

| Name | Contents |
| --- | --- |
| Watch history | Recordings you've watched |
| Recordings with comments | Recordings with team comments |
| Shared recordings | Recordings shared externally |
| Exported recordings | Recordings exported as clips/screenshots |
| Expiring soon | Recordings expiring in the next 10 days |
| Frustration signals | Sessions with multiple rage clicks or exceptions in the last 7 days |

## Console log search syntax

In the inspector's Console tab:

| Token | Match |
| --- | --- |
| `jscript` | fuzzy match |
| `=scheme` | exact match |
| `'python` | include match |
| `!ruby` | does not include |
| `^java` | starts with |
| `!^earlang` | does not start with |
| `.js$` | ends with |
| `!.go$` | does not end with |
