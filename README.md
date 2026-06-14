# Essential Grammar - 90-Day English Learning Program

## Overview

**Essential Grammar** is a complete, self-contained web application for learning English grammar over 90 days. Designed for Spanish speakers at levels A1–B1, it features 115 grammar units with interactive exercises, automatic correction, gamification, and full progress tracking.

- **Technology**: Pure HTML5, CSS3, JavaScript (no frameworks)
- **Size**: Single `index.html` file (~323KB)
- **Offline**: Works completely without internet
- **Data**: All content (115 units + 90-day schedule) embedded as JSON
- **Browser**: Chrome, Safari, Firefox (modern versions)

## Quick Start

1. Open `index.html` in your web browser
2. Select your start date
3. Complete one lesson per day
4. Unlock badges, level up, and track progress

**No installation, compilation, or backend server required.**

---

## Features

### 📚 Content

- **115 grammar units** covering:
  - Present tenses (9 units)
  - Past tenses (5 units)
  - Present perfect (6 units)
  - Modals & imperatives (8 units)
  - Articles & determiners (20 units)
  - Adjectives & adverbs (8 units)
  - Prepositions (13 units)
  - Phrasal verbs (2 units)
  - And 12 more categories

- **Per-unit content**:
  - Clear Spanish explanations
  - Grammar rules with examples
  - Contrast tables for confusing structures
  - Common mistakes (English ↔ Spanish)
  - Tips for Spanish speakers

### ✏️ Exercises

- **6 exercise types**:
  - Fill-gap (complete the blank)
  - Multiple choice
  - Transform sentences
  - Correct errors
  - Matching
  - Reorder words

- **933+ practice items** across all units
- **Smart validation**: Handles contractions, spacing, capitalization, and alternative correct answers
- **Immediate feedback**: Explanations shown after each answer
- **Adaptive difficulty**: Progresses from A1 → A2 → B1

### 🎮 Gamification

| Feature | Details |
|---------|---------|
| **XP System** | 50 XP per lesson, 25 XP for perfect score |
| **Levels** | 6 levels from Beginner to Master |
| **Daily Streak** | Track consecutive days completed |
| **Badges** | 8 achievements (first lesson, 3/7-day streaks, 10/45/90 lessons, etc.) |
| **Statistics** | Track average score, best streak, completion % |

### 📊 Progress Tracking

- **Calendar view**: See all 90 days with completion status
- **Daily warnings**: Get alerts if you fall behind schedule
- **Export/import**: Back up your progress as JSON
- **Responsive stats**: Overall %, avg score, XP, current level
- **Activity history**: Track when you completed each lesson

### 🎨 User Experience

- **Theme toggle**: Light/dark mode
- **Responsive design**: Mobile, tablet, desktop
- **Accessibility**: 
  - Semantic HTML
  - Keyboard navigation (Tab, Enter)
  - ARIA labels
  - Focus indicators
  - Reduced-motion support
- **Smooth animations**: Page transitions, progress bars
- **Settings**: Font size adjustment, data export

---

## Architecture

### File Structure

```
index.html (323 KB)
├── HTML Structure
│   ├── Welcome screen
│   ├── Dashboard (progress overview)
│   ├── Lesson view (content + exercises)
│   ├── Progress view (calendar, stats)
│   └── Settings panel
├── Embedded CSS (complete styling)
│   ├── CSS variables for theming
│   ├── Responsive grid layout
│   ├── Dark mode support
│   └── Print styles
└── Embedded JavaScript
    ├── Data storage (localStorage)
    ├── State management (AppState class)
    ├── Exercise rendering
    ├── Answer validation (Utils)
    ├── UI controller (app object)
    └── Validation diagnostics
```

### Data Structure

#### Course JSON
```javascript
{
  "course": { metadata, schema documentation },
  "sections_outline": [ 20 grammar topics ],
  "units": [
    {
      "id": 1-115,
      "section": "Present",
      "level": "A1|A2|B1",
      "explanation": { rules, examples, common mistakes },
      "exercises": [ { type, instructions, items } ]
    }
  ]
}
```

#### Schedule JSON
```javascript
[
  {
    "day": 1-90,
    "units": [ 1, 2 ],  // Which units to study
    "type": "normal|review|evaluation",
    "unitDetails": [ { id, title, level } ]
  }
]
```

#### Local Storage
```javascript
localStorage["essentialGrammar90Days"] = {
  version: 1,
  profile: { startDate, currentDay },
  progress: { completedDays, dayScores, dayAttempts },
  gamification: { xp, level, streak, badges },
  settings: { theme, fontSize }
}
```

### Key Classes & Objects

| Component | Purpose |
|-----------|---------|
| `AppStorage` | Load/save state to localStorage with error handling |
| `AppState` | Central state object (profile, progress, gamification) |
| `Utils` | Helper functions (answer normalization, date math, HTML escape) |
| `app` | Main controller (event listeners, view switching, lesson logic) |

---

## 90-Day Curriculum Design

### Progression Strategy

1. **Days 1–20**: A1 Fundamentals (16 units)
   - Present simple & continuous
   - Past simple & continuous
   - Basic imperatives

2. **Days 21–68**: A2 Consolidation (52 units)
   - Perfect tenses
   - Modals & auxiliaries
   - Articles & determiners
   - Adjectives & adverbs

3. **Days 69–88**: B1 Advanced (46 units)
   - Complex sentence structures
   - Prepositions & phrasal verbs
   - Reported speech
   - Conditionals

4. **Days 89–90**: Final Review & Evaluation

### Pedagogical Decisions

✓ **Units ordered by difficulty** (A1 → A2 → B1)  
✓ **Thematic grouping** (same-section units studied together)  
✓ **Review every 10 days** (built into schedule)  
✓ **Balanced workload** (~1.3 units/day on average)  
✓ **No prerequisites blocked** (can revisit any day)  

---

## Answer Validation

The `Utils.compareAnswers()` function handles:

```javascript
// Simple answers
"am" == "am" ✓

// Case insensitive
"Am" == "am" ✓

// Contractions
"I'm" == "I am" ✓

// Alternative answers (with /)
Answer: "Do you have? / Have you got?" 
User: "do you have?" ✓

// Empty answers (marked as —)
Answer: "—"
User: "" ✓

// Multiple blanks (array)
Answer: ["the", "a", "is"]
User response array checked against each
```

---

## Development & Customization

### Modify Content

If you want to add or edit units:

1. Extract the embedded JSON from `index.html`
2. Edit the JSON (structure defined in `_schema_doc`)
3. Re-embed it in the HTML

**Note**: All content is embedded directly; no external files needed.

### Extend Functionality

Common extensions:

- **Add audio pronunciation**: Embed MP3 as base64 or data URLs
- **Add spaced repetition**: Modify `SCHEDULE` to insert review days
- **Add more exercise types**: Create new render function in `renderExerciseItem()`
- **Add user authentication**: Integrate with Firebase/Supabase
- **Deploy online**: Upload to GitHub Pages, Netlify, Vercel

### Debug

Open browser console and run:
```javascript
runCourseValidation()
// Returns: { errors, warnings, summary }
```

Outputs validation report:
- Unit count (should be 115)
- Schedule completeness (should be 90 days)
- Exercise coverage
- Any missing or duplicate data

---

## Testing Checklist

### Functionality
- [ ] Welcome screen loads with today's date
- [ ] Starting course loads dashboard
- [ ] Lesson loads units and exercises
- [ ] Exercise inputs accept text and options
- [ ] Completing lesson advances to next day
- [ ] XP/streak/badges update correctly
- [ ] Progress bar fills as days complete
- [ ] Lessons can be revisited without losing best score

### Data Persistence
- [ ] Page refresh preserves progress
- [ ] localStorage contains encrypted state
- [ ] Export downloads valid JSON
- [ ] Import loads data correctly
- [ ] Corrupt data doesn't crash app

### UI/UX
- [ ] Responsive on mobile/tablet/desktop
- [ ] Dark mode toggle works
- [ ] Font size adjustment applies
- [ ] Buttons are clickable (min 44×44px)
- [ ] Keyboard navigation functional (Tab, Enter, Arrow keys)
- [ ] Focus indicators visible

### Accessibility
- [ ] Page structure semantic (h1, h2, labels, etc.)
- [ ] Color not only differentiator (+ text/icons)
- [ ] Contrast ratio ≥ 4.5:1 for text
- [ ] Links have visible :focus state
- [ ] Animations respect prefers-reduced-motion

### Browser Support
- [x] Chrome/Edge (latest)
- [x] Safari (latest)
- [x] Firefox (latest)
- [x] Mobile Chrome/Safari

---

## File Statistics

| Metric | Value |
|--------|-------|
| File size | ~323 KB |
| Lines of code | 1,544 |
| CSS variables | 30+ |
| JavaScript classes | 3 |
| Functions | 50+ |
| DOM elements rendered | 90+ |
| Exercise items | 933 |
| Grammar units | 115 |
| Days of curriculum | 90 |

---

## Known Limitations

1. **Single-file constraint**: All data & code in one HTML file
   - Pro: No dependencies, works offline
   - Con: Larger file, harder to modularize

2. **localStorage only**: ~5–10MB limit per domain
   - Current usage: <1MB
   - Future: Consider IndexedDB for larger datasets

3. **No server features**:
   - No cloud sync
   - No multi-device support
   - No user accounts

4. **Limited keyboard input parsing**:
   - Answer normalization is basic (case-insensitive, spacing)
   - Doesn't handle complex grammar variations
   - Users can suggest answers in feedback

---

## Browser Compatibility

| Feature | Chrome | Safari | Firefox | Edge |
|---------|--------|--------|---------|------|
| ES6 JavaScript | ✓ | ✓ | ✓ | ✓ |
| CSS Grid | ✓ | ✓ | ✓ | ✓ |
| CSS Variables | ✓ | ✓ | ✓ | ✓ |
| localStorage | ✓ | ✓ | ✓ | ✓ |
| File API | ✓ | ✓ | ✓ | ✓ |
| Date API | ✓ | ✓ | ✓ | ✓ |

---

## Future Ideas

- Spaced repetition algorithm for review days
- Leaderboard for streak records
- Customizable daily time commitment
- Text-to-speech for pronunciations
- AI-based answer evaluation
- Learning path branching (focus on weak areas)
- Community features (share scores, badges)
- Mobile app wrapper (Cordova, React Native)

---

## License & Usage

This application and all content are educational. Feel free to:
- ✓ Use for personal learning
- ✓ Modify for your needs
- ✓ Share with others
- ✓ Deploy online

---

## Troubleshooting

### "Progress not saving"
→ Check browser localStorage is enabled (Settings → Privacy)

### "App looks broken on dark mode"
→ Click theme toggle button (🌙) in header

### "Exercises not loading"
→ Check browser console for errors
→ Try refreshing the page
→ Clear localStorage: `localStorage.clear()`

### "Can't complete lesson"
→ Complete at least one exercise first
→ Check that form inputs are filled

### "Import not working"
→ Ensure file is valid JSON exported from app
→ Check file size < 5MB

---

## Contact & Support

For issues or suggestions:
1. Check the browser console for error messages
2. Run `runCourseValidation()` to diagnose data issues
3. Try exporting/importing data to reset
4. Document the exact steps to reproduce

---

## Changelog

### Version 1.0 (2026-06-14)
- Initial release
- 115 grammar units (A1–B1)
- 90-day structured curriculum
- Interactive exercises with validation
- Gamification system (XP, levels, badges, streaks)
- Progress tracking & calendar
- Export/import functionality
- Dark mode support
- Responsive design
- Full offline capability

---

**Made with ❤️ for English learners**

Start learning today. Pick up the habit. Master grammar in 90 days.
