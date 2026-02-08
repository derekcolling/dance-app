# NYCDA KC Parent Tracker — Project Memory

## What This Is
A mobile-first web app for parents at the NYCDA Kansas City dance competition. Shows the full 512-entry schedule across 2 days, lets parents favorite dances, and syncs a real-time "Now Playing" indicator via Firebase.

## Architecture
- **Static site**: Single `index.html` with all HTML/CSS/JS inline (no framework, no build)
- **Data**: `schedule.json` (96KB, 512 entries) fetched at runtime
- **Realtime**: Firebase Realtime Database
- **Favorites**: localStorage (`nycda_favorites` key, stored as JSON array of entry_ids)
- **Hosting**: Vercel static deployment from GitHub

## Live URLs
- **Site**: https://nycda-kc-tracker.vercel.app
- **Admin**: https://nycda-kc-tracker.vercel.app?admin=true
- **GitHub**: https://github.com/derekcolling/nycda-kc-tracker

## Firebase
- **Project**: "dance-tracker-c3642"
- **Project ID**: `dance-tracker-c3642`
- **URL**: `https://dance-tracker-c3642-default-rtdb.firebaseio.com`
- **Node**: `competition/current_entry_index`
- **Security**: Public read/write (temporary for development)
- **Realtime**: Enabled via Firebase Realtime Database `onValue`
- Config object is embedded in index.html

## Schedule Data Structure
- Fields: `entry_id`, `routine_title`, `studio_code`, `category`, `approx_time`
- `entry_id` is a string (some have letters like "14A", "19A")
- `category` parses to: `{ageGroup} {genre} {type}` (e.g., "Junior Musical Theatre Solo")
- **Day 1**: entries up to index 238 (entry_id 1–234), 7:30 AM – 10:14 PM
- **Day 2**: entries from index 239 (entry_id 236–506), 7:00 AM – 11:23 PM
- Day boundary detected by PM→AM time transition (entry 234→236, id 235 skipped)
- 42 unique studio codes, 4 age groups, 10 genres, 8 performance types

## Key Implementation Details
- `currentEntryIndex` is the array index (0-based) into the sorted SCHEDULE array, not the entry_id
- Firebase stores `current_entry_index` as this array index
- Admin mode activated via `?admin=true` URL parameter
- The "Jump to #" input in admin bar expects an `entry_id` string, not an index
- Category parsing handles multi-word types like "Small Production" and "Duo/Trio"
- Filter dropdowns are dynamically populated from the data on load

## Deployment
- `vercel --prod --yes` from the project directory
- Project name had to be lowercase (`nycda-kc-tracker`) — Vercel rejects uppercase
- GitHub repo connected to Vercel for auto-deploy on push

## Files
```
/Users/derekcolling/Documents/NYCDA/
├── index.html       ← Complete SPA (~600 lines, all inline)
├── schedule.json    ← 512 dance entries (96KB)
├── schedule.pdf     ← Original PDF schedule
├── vercel.json      ← Rewrites + cache headers for schedule.json
└── .vercel/         ← Vercel project config (gitignored)
```

## Lessons Learned
- Supabase SQL Editor uses Monaco — use `window.monaco.editor.getEditors()[0].setValue()` to set content programmatically
- The `type` action in browser automation merges lines when text has newlines — use JS/Monaco API for multi-line code entry
- Firebase Realtime Database: Use `onValue` for live updates.
- The `databaseURL` in the config must be correct (currently `https://dance-tracker-c3642-default-rtdb.firebaseio.com`).

## Interaction Design Decisions (Feb 2026)
- **Day Buttons**: Strict "Green/Tan" system.
  - **Green**: Selected / Active Filter.
  - **Tan**: Unselected.
  - No separate "Live" indicator (badges, dots, outlines) on the buttons themselves. Status is communicated by auto-selecting the live day.
- **Auto-Selection Logic**: On initial load or "Reset" action, the app detects the current live entry and automatically sets the `day` filter to match. This ensures the corresponding Day Button turns Green, providing context.
- **Reset Behavior**: Clicking "X" in Search or the Play FAB triggers a full reset:
  - Clears search.
  - Resets all dropdowns.
  - Sets Day filter to the Live Day.
  - Scrolls to the live entry.
- **Header Cleanup**:
  - Removed "Up Next" alert strip (redundant).
  - Quick Glance bar is sticky, white background, standard text colors.
  - Search bar clears properly and resets state.
