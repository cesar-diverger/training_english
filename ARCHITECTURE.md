# Essential Grammar - Architecture & Design Documentation

## System Overview

```
┌─────────────────────────────────────────────────────┐
│         Single HTML File (index.html)               │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │          HTML Structure (5 Views)            │  │
│  │  - Welcome  - Dashboard  - Lesson            │  │
│  │  - Progress - Settings                       │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │    Embedded CSS (2000+ lines)                │  │
│  │  - CSS Variables - Dark Mode                 │  │
│  │  - Responsive Grid - Animations             │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │    Embedded JavaScript (1000+ lines)        │  │
│  │  ┌────────────────────────────────────────┐ │  │
│  │  │  AppStorage (localStorage wrapper)     │ │  │
│  │  │  AppState (central state mgmt)         │ │  │
│  │  │  Utils (helpers)                       │ │  │
│  │  │  app (controller)                      │ │  │
│  │  └────────────────────────────────────────┘ │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │  Embedded JSON Data                          │  │
│  │  - courseData (115 units)                    │  │
│  │  - schedule (90 days)                        │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## Data Flow

### Initial Load
```
1. Browser opens index.html
2. HTML parser creates DOM
3. CSS variables loaded
4. JavaScript executed:
   - AppStorage.load() → localStorage
   - AppState constructor fills state
   - app.init() sets up listeners
5. Current view determined:
   - If startDate exists → show dashboard
   - Else → show welcome
```

### User Starts Course
```
User clicks "Comenzar Ahora"
    ↓
startCourse() called
    ↓
state.profile.startDate = selected date
state.save() → localStorage
    ↓
updateCurrentDay() calculates day offset
    ↓
showView('dashboard')
    ↓
updateDashboard() renders:
  - Day number
  - Progress bar
  - XP/Level/Streak
  - Today's lesson info
```

### User Completes Lesson
```
User views lesson content
    ↓
Fills exercise inputs
    ↓
Clicks "Completar Lección"
    ↓
completeLesson():
  1. Marks day as completed
  2. Adds 50 XP
  3. Increments streak
  4. Unlocks badges (if applicable)
  5. Advances currentDay
  6. Saves state
    ↓
Shows success message
    ↓
Returns to dashboard (next day selected)
```

### Answer Validation Flow
```
User enters answer
    ↓
Utils.compareAnswers(userInput, correctAnswer)
    ↓
Normalizes both strings:
  - Convert to lowercase
  - Trim whitespace
    ↓
Check multiple formats:
  - Direct match
  - Alternative options (/)
  - Empty answer (—)
  - Array of answers
    ↓
Return true/false
    ↓
If correct:  Show ✓ (green highlight)
If wrong:    Show ✗ (red highlight) + explanation
```

---

## JavaScript Architecture

### AppStorage Class

**Purpose**: Bridge between app state and browser localStorage

```javascript
class AppStorage {
  static load()          // Retrieve from localStorage
  static save(state)     // Save to localStorage
  static clear()         // Erase all data
}
```

**Error handling**:
- Try/catch around JSON parse
- Fallback to null if corrupted
- Validates Set conversions after load

### AppState Class

**Purpose**: Single source of truth for all user data

**Structure**:
```javascript
{
  profile: {
    startDate: "2026-06-14",
    currentDay: 15
  },
  progress: {
    completedDays: Set(1, 2, 3, 5, 7...),
    dayScores: { "1": 95, "2": 87... },
    dayAttempts: { "1": 1, "2": 2... },
    dayResponses: { "1": {...} }
  },
  gamification: {
    xp: 450,
    level: 2,
    currentStreak: 5,
    bestStreak: 12,
    lastActivityDate: "2026-06-14",
    badges: Set("first-lesson", "three-day-streak"...)
  },
  settings: {
    theme: "light",
    fontSize: "normal",
    sounds: false
  }
}
```

**Methods**:
- `save()`: Persist to localStorage
- `getLevelInfo(xp)`: Calculate level from XP

### Utils Object

**Purpose**: Reusable utility functions

```javascript
Utils = {
  normalizeAnswer(answer)           // Lowercase + trim
  compareAnswers(user, correct)     // Smart validation
  calculateDaysDifference(startDate) // Days since start
  escapeHTML(str)                   // XSS prevention
  formatDate(dateStr)               // Locale formatting
}
```

**Key algorithm - compareAnswers()**:
```
1. Normalize both inputs (lowercase, trim)
2. If correct is "/" separated: check each option
3. If correct is "—": expect empty string
4. If correct is array: check item-by-item
5. Otherwise: direct string comparison
```

### App Controller

**Purpose**: Central event handling and state mutations

**Structure**:
```javascript
const app = {
  state: AppState,
  
  // Lifecycle
  init()
  setupEventListeners()
  
  // Navigation
  showView(viewName)
  
  // Course Management
  startCourse()
  updateCurrentDay()
  loadLesson(dayNum)
  completeLesson()
  
  // Gamification
  unlockAchievements()
  
  // Rendering
  renderUnit(unit)
  renderExerciseItem(item, unitId, exIdx, itemIdx)
  
  // UI Updates
  updateDashboard()
  updateDashboard()
  updateSettings()
  updateCompleteButton()
  
  // Data Management
  exportProgress()
  importProgress(event)
  resetCourse()
  
  // Theme
  toggleTheme()
  applySettings()
}
```

---

## State Management Pattern

### Immutable Reads
```javascript
// Reading state doesn't trigger re-renders
const dayNum = this.state.profile.currentDay;
const xp = this.state.gamification.xp;
```

### Mutable Updates
```javascript
// Modifying state always:
// 1. Update object property
// 2. Call save()
// 3. Update UI (manual)

this.state.gamification.xp += 50;
this.state.save();
this.updateDashboard();  // Manual UI update
```

### Why Manual Updates?
- No reactive framework (Vue/React)
- Prevents cascading re-renders
- Full control over when UI updates
- Smaller file size

---

## View Architecture

### View System

```html
<div id="welcomeView" class="view active">
<div id="dashboardView" class="view">
<div id="lessonView" class="view">
<div id="progressView" class="view">
<div id="settingsView" class="view">
```

**CSS**:
```css
.view { display: none; }
.view.active { display: block; animation: fadeIn 0.3s; }
```

**Switching**:
```javascript
showView(name) {
  // Remove active from all
  // Add active to target
  // Scroll to top
}
```

### Lesson View Flow

```
loadLesson(dayNum)
    ↓
Get SCHEDULE[dayNum - 1]
    ↓
For each unit in day:
    ↓
renderUnit(unit)
    ↓
For each exercise in unit:
    - Render instructions
    - For each item:
        - Render prompt
        - Render input (text or options)
    ↓
Insert into #lessonContent
    ↓
Update progress bar
    ↓
Enable complete button
```

---

## 90-Day Schedule Algorithm

### Design Constraints
1. Exactly 90 days
2. All 115 units covered
3. Respects A1 → A2 → B1 progression
4. Groups units by section when possible
5. Includes review days every 10 days

### Distribution Strategy

**Goal**: 115 units ÷ 88 days = 1.31 units/day average

```javascript
for (day 1 to 88):
  unitsToday = max(1, min(3, unitsNeeded))
  
  if (day % 10 === 0):  // Review day
    unitsToday = 0
  
  schedule.push({ day, units: unitsToday })

// Days 89–90: Final review + evaluation
```

**Result**:
- Days 1–20: Mostly 1–2 units (A1 foundation)
- Days 21–68: 1–2 units (A2 consolidation)
- Days 69–88: 2–3 units (B1 acceleration)
- Days 89–90: Review + evaluation

---

## Answer Validation Examples

### Example 1: Simple Fill-Gap
```
Prompt:  "I ___ a student"
Answer:  "am"
User:    "AM"
Result:  ✓ (case-insensitive)
```

### Example 2: Contractions
```
Prompt:  "He ___ happy"
Answer:  "is / 's"
User:    "is"
Result:  ✓ (first option matches)

User:    "'s"
Result:  ✗ (incomplete)
```

### Example 3: Empty Answer
```
Prompt:  "I go ___ swimming"
Answer:  "—" (no word needed)
User:    ""
Result:  ✓ (empty is correct)

User:    "to"
Result:  ✗ (should be empty)
```

### Example 4: Multiple Blanks
```
Prompt:  "The ___ girl and the ___ boy"
Answer:  ["old", "young"]
User responses: ["old", "young"]
Result:  ✓ (both correct)
```

---

## Gamification System

### XP Distribution

| Action | XP |
|--------|-----|
| Complete lesson | 50 |
| Perfect score (100%) | +25 |
| Recover from lag | +20 |
| First attempt correct | +10 |
| Hint used | +5 |

### Level System

| Level | Name | XP Range | Icon |
|-------|------|----------|------|
| 1 | Principiante | 0–99 | 🌱 |
| 2 | Explorador | 100–249 | 🔍 |
| 3 | Aprendiz | 250–499 | 📚 |
| 4 | Constructor | 500–999 | 🏗️ |
| 5 | Experto | 1000–1499 | ⭐ |
| 6 | Maestro | 1500+ | 👑 |

### Badges

```javascript
{
  'first-lesson': {
    name: 'First Step',
    description: 'Complete your first lesson'
  },
  'three-day-streak': {
    name: 'Getting Started',
    description: '3 days in a row'
  },
  'one-week-streak': {
    name: 'Weekly Warrior',
    description: '7 consecutive days'
  },
  'ten-lessons': {
    name: 'Dedicated Learner',
    description: 'Complete 10 lessons'
  },
  'halfway': {
    name: 'Halfway There',
    description: 'Complete 45 lessons (halfway)'
  },
  'complete-course': {
    name: 'Grammar Master',
    description: 'Complete all 90 days'
  }
}
```

---

## Data Persistence

### localStorage Structure

**Key**: `"essentialGrammar90Days"`

**Size**: Typically < 1MB (limit 5–10MB per domain)

**Format**: Single JSON object serialized

### Save Strategy

**When**:
- After every state mutation
- On page unload
- After import/export

**How**:
```javascript
// Convert Sets to Arrays for JSON
{
  completedDays: [1, 2, 5, 7...],
  badges: ["first-lesson", "three-day-streak"...]
}

// Store as single JSON string
JSON.stringify(state)
```

**Load Strategy**:
```javascript
// Parse JSON
const saved = JSON.parse(data)

// Reconstruct Sets
completedDays: new Set(saved.progress.completedDays)
badges: new Set(saved.gamification.badges)
```

### Error Recovery

**If localStorage corrupted**:
1. Parsing fails → return null
2. AppState constructor creates fresh state
3. User starts fresh (no data loss message)

**If import corrupted**:
1. FileReader reads file
2. JSON.parse fails → catch error
3. User sees "Error importing file" alert

---

## CSS Architecture

### CSS Variables (Dark Mode)

```css
:root {
  --primary: #2563eb;
  --success: #10b981;
  --error: #ef4444;
  --gray-100: #f3f4f6;
  --gray-800: #1f2937;
  /* ... */
}

[data-theme="dark"] {
  --gray-100: #1e293b;  /* Inverted */
  --gray-800: #e2e8f0;
  /* ... */
}
```

### Responsive Breakpoints

```css
@media (max-width: 768px) {
  /* Mobile adjustments */
  main { padding: 1rem; }
  .grid-2 { grid-template-columns: 1fr; }
}
```

### Accessibility Features

```css
/* Focus states */
:focus-visible {
  outline: 2px solid var(--primary);
  outline-offset: 2px;
}

/* Reduced motion */
@media (prefers-reduced-motion: reduce) {
  * { animation-duration: 0.01ms !important; }
}

/* High contrast for print */
@media print {
  body { color: black; background: white; }
}
```

---

## Exercise Rendering

### Exercise Types Supported

| Type | Renders As | Validation |
|------|-----------|-----------|
| fill_blank | Text input | String comparison |
| multiple_choice | Radio buttons | Option matching |
| transform | Text input | String comparison |
| correct_error | Text input | String comparison |
| matching | ? | ? |
| reorder | ? | ? |
| short_answer | Text input | String comparison |

### Item Rendering Logic

```javascript
renderExerciseItem(item, unitId, exIdx, itemIdx) {
  const itemKey = `u${unitId}ex${exIdx}it${itemIdx}`;
  
  // Escape prompt HTML
  let promptHTML = item.prompt.replace(/___/g, '<u>_____</u>');
  
  // Check exercise type
  if (item.options) {
    // Render radio buttons
  } else {
    // Render text input
  }
  
  // Add data attributes for identification
  // Return HTML string
}
```

---

## Testing & Validation

### runCourseValidation() Function

**Purpose**: Diagnose data integrity issues

**Checks**:
```javascript
1. Unit count (should be 115)
2. Schedule length (should be 90)
3. Unit coverage (all 115 should be scheduled)
4. Exercise presence (each unit should have exercises)
5. No duplicate IDs
6. Answer format validity
```

**Output**:
```javascript
{
  errors: [...],
  warnings: [...],
  summary: {
    units: 115,
    days: 90,
    exercises: 306
  }
}
```

**How to run**:
```javascript
// In browser console:
runCourseValidation()
```

---

## Performance Considerations

### Bundle Size
- HTML: ~1,500 lines
- CSS: ~500 lines
- JavaScript: ~1,000 lines
- Embedded JSON: ~100KB (gzips to ~20KB)
- **Total**: ~323KB

### Optimization Techniques
- Inline all assets (no external requests)
- CSS variables instead of media queries
- Minimal JavaScript (no lodash, etc.)
- Event delegation on exercise items
- Lazy rendering of lesson content

### Browser Performance
- DOM updates: <100ms per action
- localStorage saves: <50ms
- Page navigation: <200ms (fade animation)

---

## Security Considerations

### Input Validation
- User answers: No validation (safe to display)
- File imports: JSON schema validation
- localStorage: Set validation on load
- Exercise options: Escaped from JSON

### XSS Prevention
```javascript
// Don't use innerHTML with user data
// Use textContent instead:
div.textContent = userInput;

// Or escape HTML:
Utils.escapeHTML(userInput)

// Or use DOM methods:
const input = document.createElement('input');
input.value = userInput;  // Auto-escaped
```

### Data Protection
- No sensitive data stored (no passwords, etc.)
- localStorage is per-origin (no cross-site access)
- Export is manual (user controls what's shared)

---

## Browser API Usage

| API | Purpose | Usage |
|-----|---------|-------|
| localStorage | Persistent state | Save/load JSON |
| FileReader | Import progress | Read user's JSON file |
| Blob/URL | Export progress | Generate download link |
| Date | Date math | Calculate days elapsed |
| CSS Variables | Dynamic styling | Theme switching |
| Flexbox/Grid | Responsive layout | UI layout |

---

## Future Architecture Changes

### Potential Improvements

1. **Service Worker**
   - Cache index.html
   - Offline detection
   - Sync on reconnect

2. **IndexedDB**
   - Replace localStorage
   - Larger data capacity
   - Better performance

3. **Module Pattern**
   - Split into modules
   - Easier testing
   - Build step (Webpack)

4. **Reactive Framework**
   - Add Vue/React
   - Cleaner state updates
   - Automatic re-renders

5. **Microservices**
   - Backend API
   - User authentication
   - Cloud sync
   - Analytics

### Why Not Now?
- Adds complexity
- Violates "no dependencies" requirement
- Single HTML file becomes multiple files
- Build process required
- More maintenance burden

---

## Debugging Guide

### Check Current State
```javascript
// In console:
console.log(app.state)
```

### Check localStorage
```javascript
localStorage.getItem('essentialGrammar90Days')
// Returns JSON string
```

### Check Schedule
```javascript
SCHEDULE
// Shows all 90 days
```

### Check Course Data
```javascript
COURSE_DATA.units
// Shows all 115 units
```

### Simulate Day Completion
```javascript
app.state.progress.completedDays.add(5)
app.state.gamification.xp += 50
app.state.save()
app.updateDashboard()
```

### Clear All Data
```javascript
localStorage.clear()
location.reload()
```

---

## Summary

**Essential Grammar** is a self-contained, offline-first web application built with vanilla technologies. Its architecture prioritizes:

- **Simplicity**: No build process, single file deployment
- **Reliability**: localStorage persistence, error recovery
- **Accessibility**: Semantic HTML, keyboard navigation, theme support
- **Performance**: Minimal dependencies, fast rendering
- **Maintainability**: Clear separation of concerns (Storage, State, UI)

The 90-day curriculum is carefully structured to progress from A1 fundamentals through B1 advanced concepts, with built-in review and gamification to maintain learner motivation.

All data (115 units, exercises, schedule) is embedded for true offline capability—no internet required.
