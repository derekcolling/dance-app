# NYCDA KC Parent Tracker - System Overview & Build Prompt

> A complete specification for building a mobile-first dance competition schedule tracker with real-time updates.

---

## Product Overview

**Purpose**: A mobile web app for parents at the NYCDA Kansas City dance competition. It helps them track when their child performs across a 2-day, 500+ entry schedule.

**Target Users**: Non-technical parents ("moms") who need to:
1. Find their child's dance quickly
2. Know what's happening NOW on stage
3. Save favorites and get countdown estimates

---

## Technical Architecture

### Stack
- **Frontend**: Single `index.html` file with inline CSS/JS (no framework, no build step)
- **Data**: Static `schedule.json` fetched at runtime
- **Realtime**: Firebase Realtime Database for live "Now Playing" sync
- **Favorites**: localStorage
- **Hosting**: Vercel static deployment

### Why This Stack?
- **Zero dependencies**: Works offline after initial load
- **Instant deploy**: Push to GitHub → auto-deploy
- **Free**: Supabase free tier + Vercel free tier
- **Simple**: One file to edit, no build errors

---

## Data Model

### Schedule Entry
```json
{
  "entry_id": "110",
  "routine_title": "Mmmbop",
  "studio_code": "H2",
  "category": "Junior Tap Group",
  "approx_time": "2:07 PM"
}
```

### Derived Fields (computed on load)
- `day`: "1" or "2" (detected via PM→AM time transition)
- `ageGroup`: Parsed from category ("Junior", "Teen", etc.)
- `genre`: Parsed from category ("Tap", "Jazz", etc.)
- `type`: Parsed from category ("Solo", "Group", etc.)
- `index`: Array position (0-based)

### Firebase Structure: `competition`
| Node | Type | Description |
|------|------|-------------|
| current_entry_index | int | Array index of "Now Playing" |
| updated_at | string (ISO) | Last update time |

---

## UI Components

### 1. Header
- Brand name: "NYCDA KC"
- Subtitle: "Parent Tracker"
- Background: Dark muted green (`#3D4F3D`)

### 2. Quick Glance Bar (Sticky)
- Shows: "YOUR NEXT DANCE" with title and countdown
- Only visible if user has favorites
- Background: White, clean contrast

### 3. Tabs
- "All Dances" / "My Dances" toggle
- "My Dances" shows badge with count

### 4. Filter Bar (Sticky below tabs)
- **Day Chips**: "Day 1", "Day 2"
  - Green = Selected, Tan = Unselected
  - Click active chip to deselect (toggle-to-clear)
- **Dropdowns**: Studio, Age, Genre, Type
- **Search**: With clear button ("×")

### 5. Dance Cards
- Entry number (circle badge, green if "Now Playing")
- Title, Studio badge, Time, Category
- **Favorite Button**: Light gray circle, gold when active
- **Countdown Badge**: Shows "X away ~Ym" for upcoming favorites

### 6. FAB (Floating Action Button)
- Bottom-right corner
- Click = Reset all filters + scroll to "Now Playing"

---

## Key Interactions

### Favoriting
- Tap the star circle on any dance card
- Stored in localStorage as array of `entry_id` strings
- Persists across sessions
- "My Dances" tab filters to favorites only

### Reset to Live
Triggered by: FAB click, Search "×" click
1. Clear search query
2. Reset all dropdown filters
3. Set Day filter to the **current live day** (auto-select)
4. Scroll to "Now Playing" entry
5. Day button turns Green to show context

### Admin Mode
- URL: `?admin=true`
- Shows control bar with Prev/Next buttons
- "Jump to #" input to skip to any entry
- Updates Firebase `competition/current_entry_index`

---

## Design System

### Colors
```css
--bg: #FAF8F5          /* Page background (warm off-white) */
--card: #ffffff        /* Card backgrounds */
--text: #2D2A26        /* Primary text */
--header-bg: #3D4F3D   /* Dark muted green */
--sand: #EDE8E3        /* Tan/beige */
--brand-green: #7A9B7D /* Brand green */
--now-playing-bg: #96B599
--caramel-gold: #C4944E
```

### Typography
- Font: DM Sans (Google Fonts)
- Weights: 400, 500, 600, 700

### Interaction Rules
1. **Day Buttons**: Strictly Green/Tan. No dots, badges, or outlines.
2. **Favorite Button**: Always visible as a light gray circle.
3. **Touch Targets**: Minimum 44x44px.
4. **Transitions**: 200ms for micro-interactions.

---

## Firebase Setup

### 1. Database Structure
The app expects a Realtime Database with the following structure:
```json
{
  "competition": {
    "current_entry_index": 0,
    "updated_at": "2026-02-07T..."
  }
}
```

### 2. Security Rules
```json
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
```

### 3. Configuration
- Use the `firebaseConfig` object from `index.html`.
- Realtime listeners are attached to the `competition` node.

---

## Realtime Sync Logic

```javascript
import { getDatabase, ref, onValue } from 'firebase/database';

const entryRef = ref(db, 'competition/current_entry_index');
onValue(entryRef, (snap) => {
  state.currentEntryIndex = snap.val();
  renderCards();
});
```

---

## File Structure
```
/project-root/
├── index.html       ← Complete SPA (~1600 lines)
├── schedule.json    ← 512 entries from competition
├── vercel.json      ← Cache headers for JSON
├── MEMORY.md        ← Project knowledge base
└── .vercel/         ← Deployment config (gitignored)
```

---

## Build Prompt

> Use this prompt to recreate the system from scratch:

```
Build a mobile-first web app for parents at a dance competition.

REQUIREMENTS:
1. Single index.html file (all CSS/JS inline, no framework)
2. Fetch schedule.json at runtime (512 dance entries)
3. Parse each entry to extract: day, ageGroup, genre, type
4. Display as scrollable card list with filters (Day, Studio, Age, Genre, Type)
5. Search bar with clear button
6. Favorite system using localStorage
7. "My Dances" tab to show favorites only
8. Real-time "Now Playing" indicator via Firebase Realtime Database
9. Admin mode (?admin=true) to advance the current entry
10. FAB button to reset filters and scroll to now playing
11. Sticky Quick Glance bar showing next favorited dance

DESIGN:
- Warm, muted color palette (sage greens, warm tans, off-white)
- DM Sans font
- 44px minimum touch targets
- Cards with entry number, title, studio badge, time, category
- Favorite button: light gray circle, gold when active
- Day chips: Green = selected, Tan = unselected

FIREBASE:
- Node: competition/current_entry_index
- Realtime subscription for live updates

HOSTING:
- Deploy to Vercel as static site
- Auto-deploy from GitHub

The app should work perfectly on mobile Safari and Chrome.
```

---

## Lessons Learned

1. **Firebase Realtime**: Use `onValue` for permanent listeners.
2. **Day Detection**: Find PM→AM time transition to split days.
3. **Category Parsing**: Handle multi-word types like "Small Production".
4. **entry_id vs index**: `entry_id` is a string, `index` is array position.
5. **Touch UX**: Make buttons obvious (backgrounds, not just icons).
6. **Filter Logic**: "Reset to Live" should auto-select the current day.

---

## Future Enhancements (Not Implemented)
- Push notifications for upcoming favorites
- PWA with offline support
- Multi-competition support
- Dark mode toggle
- Share favorites via URL

---

*Last updated: February 2026*
