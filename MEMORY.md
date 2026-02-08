# NYCDA KC Parent Tracker — Project Memory

## Project Status (Feb 8, 2026)
- **Current Focus**: Refining the "My Dances" user experience and Live state visibility.
- **Milestone Reached**: The core competition tracker is fully functional with Firebase-synced "Now Playing" status, favorite management, and a robust "Find My Dancer" search flow.
- **Recent Polish**: Implemented a "LIVE" extended FAB, replaced star icons with "NOW" pulse indicators for the current dance, and refined all modal UI elements (centering, transparency, overlap fixes).

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
/Users/derekcolling/Documents/dance-app/
├── index.html       ← Complete SPA (~2000 lines, all inline)
├── schedule.json    ← 512 dance entries (96KB)
├── vercel.json      ← Rewrites + cache headers for schedule.json
└── MEMORY.md        ← This file
```

## Lessons Learned
- The `type` action in browser automation merges lines when text has newlines — use JS/Monaco API for multi-line code entry
- Firebase Realtime Database: Use `onValue` for live updates.
- The `databaseURL` in the config must be correct (currently `https://dance-tracker-c3642-default-rtdb.firebaseio.com`).

## Interaction Design Decisions (Feb 2026)
- **Day Buttons**: Strict "Green/Tan" system.
  - **Green**: Selected / Active Filter.
  - **Tan**: Unselected.
- **"My Dances" Management**: 
  - **Find My Dancer Modal**: A dedicated searching interface (`#findDancerDialog`) triggered by an "Add Routine" button in the filter row or the empty state.
  - **UI Refinements**: Search modal is opaque white, centered, with high-contrast backdrop and padded search icons to prevent overlap.
  - **Search Source**: Searches the entire `state.entries` list (ignoring active filters) by routine title or entry ID.
- **Live State Visuals**:
  - **The "NOW" Icon**: In the list, the star icon is replaced with a green "NOW" pulse icon for the currently active dance.
  - **The "LIVE" FAB**: Circular crosshair replaced with an extended pill button labelled "LIVE" + a pulsating red indicator dot. This serves as the primary "Return to Now" navigation.
- **Auto-Selection Logic**: On initial load or "Reset" action, the app detects the current live entry and automatically sets the `day` filter to match.
- **Reset Behavior**: Clicking "X" in Search or the "LIVE" FAB triggers a full reset (clears search, resets dropdowns, sets Day filter to Live Day, and scrolls to live entry).
- **Sticky Persistence**: Header and filter row are sticky to maintain context while scrolling deep lists.
