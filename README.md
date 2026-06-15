# Keepsake · מזכרת

A personal mood journal that runs entirely in the browser, available in English and Hebrew (RTL) with a single tap to switch between them. One entry per day — a score, a short memory, and a few tags. Entries accumulate into a weekly, monthly, and yearly picture of how time actually felt.

Both languages share the same entries — switching language only changes the interface, not your data.

---

## Features

**Bilingual interface**
- English and Hebrew, switchable at any time via the language toggle in the top corner
- Full RTL layout for Hebrew, including mirrored decorative elements
- Dates, month names, and weekday labels follow the selected language's locale
- Remembers your language choice (and otherwise follows your browser's language on first visit)

**Logging**
- Mood slider from 1 (Awful / נורא) to 7 (Bliss / אושר) with a named label and hint for each level
- Free-text memory field for anything worth keeping
- Six detail tags: Rest, Work, People, Outside, Food, Movement (מנוחה, עבודה, אנשים, בחוץ, אוכל, תנועה)

**History**
- Mood stats (average, best day, positive rate) scoped to the current week, month, or year
- Sparkline chart that reflects the selected range
- Monthly calendar with color-coded days
- Scrollable entry timeline with inline edit

**Cloud sync**
- Anonymous Firebase auth — no account required
- Entries sync automatically across devices via Realtime Database
- Copy your sync ID on one device, paste it on another via "Link ID"

**PWA**
- Installable on iOS and Android
- Works offline after first load

---

## Stack

Single HTML file. No build step, no dependencies, no framework.

- Vanilla JS and CSS
- Firebase Realtime Database (anonymous auth)
- GitHub Pages (or any static host) for hosting

---

## Setup

To run your own instance:

1. Clone the repo and open `index.html` in a browser — it works locally without Firebase (local storage only).

2. For cloud sync, create a Firebase project, enable Anonymous Authentication, and create a Realtime Database in your preferred region. Replace the `firebaseConfig` block in the Firebase `<script type="module">` section with your own config.

3. Set your Realtime Database rules:

```json
{
  "rules": {
    "users": {
      ".read": "auth != null",
      ".write": "auth != null"
    }
  }
}
```

4. Add your deployment's domain to Firebase Auth > Settings > Authorized domains. (The app reads `location.hostname` automatically, so the in-app error message will tell you the exact domain to add.)

5. Push `index.html` and the other files to your static host.

---

## Data

All entries are stored in `localStorage` under `keepsake-slider-entries` and mirrored to Firebase under `users/{uid}/entries`. Each entry is a JSON object:

```json
{
  "date": "2026-06-13",
  "score": 5,
  "memory": "Walked to the beach before sunset.",
  "chips": ["outside", "movement"]
}
```

Entry data (score, memory, chip ids) is language-independent — only the display labels change with the UI language. CSV export and the chip/mood labels shown in the timeline are translated based on whichever language is active at the time.

No analytics, no tracking, no third-party requests beyond Firebase and the Google Fonts CDN used for the icon SVG.

---

## i18n architecture

- `window.STRINGS.en` / `window.STRINGS.he` hold every UI string (including small functions for strings with variables, like "Delete all 3 entries?").
- `window.MOODS.en` / `window.MOODS.he` hold the seven mood levels (name, color, hint) per language.
- `window.WEEKDAYS.en` / `window.WEEKDAYS.he` hold the calendar weekday letters.
- Static markup uses `data-i18n`, `data-i18n-placeholder`, `data-i18n-aria-label`, and `data-i18n-title` attributes; `window.translateStaticDOM(lang)` applies them.
- `applyLanguage(lang)` (in the main script) translates the static DOM, re-renders all locale-dependent content (dates, calendar, sparkline, stats, entries, cloud status), and persists the choice to `localStorage["keepsake-lang"]`.
