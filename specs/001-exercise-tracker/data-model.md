# Data Model: Exercise Tracker

**Phase**: 1 - Design & Contracts  
**Date**: 2026-03-11  
**Status**: Complete (MVP scope: US1 + US2)

---

## Domain Entities

### 1. Exercise

Represents a unique exercise type (e.g., "Benchpress", "Squats").

| Field | Type | Constraints | Notes |
|-------|------|-------------|-------|
| `id` | Long | Primary Key, auto-incremented | |
| `name` | String | NOT NULL, unique per session | Exercise type (e.g., "Benchpress") |
| `description` | String | NULL | Optional notes (e.g., "Incline bench variant") |
| `personalRecord` | Double | NULL | All-time max weight lifted (updated on insert) |
| `personalRecordUnit` | String (lbs/kg) | NOT NULL if personalRecord set | Unit of PR (preserved from original entry) |
| `lastPerformed` | LocalDate | NULL | Most recent date this exercise recorded |
| `createdAt` | LocalDateTime | NOT NULL, default=now | When exercise first recorded |

**Relationships**:
- One-to-many with `ExerciseSet` (one exercise type has many set instances)

**Behavior**:
- `personalRecord` auto-calculated from all ExerciseSet entries (max weight across all sets for this exercise)
- `personalRecordUnit` = unit from the set with max weight
- `lastPerformed` = most recent sessionDate of any ExerciseSet for this exercise
- Never delete exercises; only deactivate if needed (preserves history)

**Validation**:
- `name`: Required, 1-100 characters, alphanumeric + spaces
- `description`: Optional, max 500 characters

---

### 2. ExerciseSession

Represents a complete workout session (all exercises performed on a single date).

| Field | Type | Constraints | Notes |
|-------|------|-------------|-------|
| `id` | Long | Primary Key, auto-incremented | |
| `sessionDate` | LocalDate | NOT NULL, indexed | Date only (local timezone, no time component) |
| `notes` | String | NULL | Session-level notes (e.g., "felt strong today") |
| `createdAt` | LocalDateTime | NOT NULL, default=now | When session record created |
| `updatedAt` | LocalDateTime | NOT NULL, default=now | Last modified timestamp |

**Relationships**:
- One-to-many with `ExerciseSet` (one session has multiple exercise sets)

**Behavior**:
- `sessionDate` = local date when exercises were performed (no time component)
- One session can contain multiple exercises and multiple sets per exercise
- Sessions immutable after creation (though individual sets can be deleted)

**Validation**:
- `sessionDate`: Cannot be future date, within reasonable past (e.g., not >10 years ago)

**Index**:
- Compound index on (sessionDate DESC) for fast chronological queries

---

### 3. ExerciseSet

Represents a single set of an exercise performed (atomic unit of work).

| Field | Type | Constraints | Notes |
|-------|------|-------------|-------|
| `id` | Long | Primary Key, auto-incremented | |
| `exerciseId` | Long | Foreign Key → Exercise, NOT NULL | Which exercise |
| `sessionId` | Long | Foreign Key → ExerciseSession, NOT NULL | Which session |
| `weight` | Double | NOT NULL, ≥ 0 | Weight lifted (numeric value) |
| `weightUnit` | String (lbs/kg) | NOT NULL | Original unit recorded (IMMUTABLE per Constitution VIII) |
| `repsCompleted` | Int | NOT NULL, ≥ 0 | Reps performed (can be 0 for failed sets) |
| `setNumber` | Int | NOT NULL, ≥ 1 | Set order (1st, 2nd, 3rd set, etc.) |
| `notes` | String | NULL | Per-exercise-per-day notes (e.g., "felt weak this set") |
| `createdAt` | LocalDateTime | NOT NULL, default=now | When recorded |

**Relationships**:
- Belongs to `Exercise` (via exerciseId)
- Belongs to `ExerciseSession` (via sessionId)

**Behavior**:
- `weightUnit` stored with EVERY entry per Constitution VIII (Data Integrity & Format Preservation)
- `repsCompleted` can be 0 (representing a failed/incomplete set)
- `setNumber` auto-assigned based on session + exercise (e.g., 3rd set of Benchpress on March 11)
- `weight` + `weightUnit` together form immutable historical record

**Validation**:
- `weight`: ≥ 0, reasonable upper bound (e.g., <10,000 lbs/kg to catch data entry errors)
- `repsCompleted`: ≥ 0, reasonable upper bound (e.g., <1000)
- `setNumber`: ≥ 1

**Foreign Key Cascade**:
- Delete ExerciseSession → cascades delete all its ExerciseSets (cleanup)
- Delete Exercise → prevents deletion if any ExerciseSet references it (data integrity)

**Indexes**:
- (exerciseId) - for query "all sets of exercise X"
- (sessionId) - for query "all sets in session Y"
- (exerciseId, sessionDate) - for graph queries (weight progression per exercise)

---

### 4. UserPreference

Represents user settings and preferences.

| Field | Type | Constraints | Notes |
|-------|------|-------------|-------|
| `id` | Long | Primary Key (only 1 record per app) | |
| `preferredWeightUnit` | String (lbs/kg) | NOT NULL, default='lbs' | Unit for display (not stored data) |
| `theme` | String (light/dark) | NULL | UI theme preference (future P2 feature) |
| `createdAt` | LocalDateTime | NOT NULL | When user first configured |
| `updatedAt` | LocalDateTime | NOT NULL | Last preference change |

**Behavior**:
- Only 1 record (enforced by application logic, not database constraint)
- Changes to `preferredWeightUnit` affect NEW entries + display views, NOT historical data
- Historical ExerciseSet records preserve original unit (Constitution VIII)
- Example: User records "185 lbs", changes preference to "kg", app displays "~84 kg" but stored value remains "185 lbs"

**Validation**:
- `preferredWeightUnit`: Must be 'lbs' or 'kg'

---

## Database Schema (Room DDL)

### Table: exercises
```sql
CREATE TABLE exercises (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL UNIQUE,
    description TEXT,
    personalRecord REAL,
    personalRecordUnit TEXT CHECK(personalRecordUnit IN ('lbs', 'kg')),
    lastPerformed TEXT,  -- ISO 8601 date
    createdAt INTEGER NOT NULL DEFAULT (strftime('%s', 'now')),
    UNIQUE(name)
);
```

### Table: exercise_sessions
```sql
CREATE TABLE exercise_sessions (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    sessionDate TEXT NOT NULL,  -- ISO 8601 format: YYYY-MM-DD
    notes TEXT,
    createdAt INTEGER NOT NULL DEFAULT (strftime('%s', 'now')),
    updatedAt INTEGER NOT NULL DEFAULT (strftime('%s', 'now'))
);

CREATE INDEX idx_exercise_sessions_date ON exercise_sessions(sessionDate DESC);
```

### Table: exercise_sets
```sql
CREATE TABLE exercise_sets (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    exerciseId INTEGER NOT NULL REFERENCES exercises(id) ON DELETE RESTRICT,
    sessionId INTEGER NOT NULL REFERENCES exercise_sessions(id) ON DELETE CASCADE,
    weight REAL NOT NULL CHECK(weight >= 0),
    weightUnit TEXT NOT NULL CHECK(weightUnit IN ('lbs', 'kg')),
    repsCompleted INTEGER NOT NULL CHECK(repsCompleted >= 0),
    setNumber INTEGER NOT NULL CHECK(setNumber >= 1),
    notes TEXT,
    createdAt INTEGER NOT NULL DEFAULT (strftime('%s', 'now'))
);

CREATE INDEX idx_exercise_sets_exerciseId ON exercise_sets(exerciseId);
CREATE INDEX idx_exercise_sets_sessionId ON exercise_sets(sessionId);
CREATE INDEX idx_exercise_sets_exercise_date ON exercise_sets(exerciseId, sessionDate);
```

### Table: user_preferences
```sql
CREATE TABLE user_preferences (
    id INTEGER PRIMARY KEY DEFAULT 1,
    preferredWeightUnit TEXT NOT NULL DEFAULT 'lbs' CHECK(preferredWeightUnit IN ('lbs', 'kg')),
    theme TEXT,
    createdAt INTEGER NOT NULL DEFAULT (strftime('%s', 'now')),
    updatedAt INTEGER NOT NULL DEFAULT (strftime('%s', 'now'))
);
```

---

## Data Flow Examples

### Example 1: Recording an Exercise (US1 - P1)

**User Action**: "Record Benchpress 185 lbs, 8 reps, 4 sets"

**Database Operations**:
1. Get or create Exercise("Benchpress")
2. Get or create ExerciseSession(sessionDate=today)
3. Insert 4 ExerciseSet records:
   - Set 1: exerciseId=1, weight=185, weightUnit='lbs', reps=8, setNumber=1
   - Set 2: exerciseId=1, weight=185, weightUnit='lbs', reps=8, setNumber=2
   - Set 3: exerciseId=1, weight=185, weightUnit='lbs', reps=8, setNumber=3
   - Set 4: exerciseId=1, weight=185, weightUnit='lbs', reps=8, setNumber=4
4. Update Exercise.personalRecord = 185, personalRecordUnit='lbs', lastPerformed=today

**Result**: 4 rows in exercise_sets table, 1 session, exercise data persisted.

---

### Example 2: Viewing History (US2 - P1)

**User Action**: "Show all exercises, sorted by date"

**Database Query**:
```sql
SELECT es.*, e.name, sess.sessionDate
FROM exercise_sets es
JOIN exercises e ON es.exerciseId = e.id
JOIN exercise_sessions sess ON es.sessionId = sess.id
ORDER BY sess.sessionDate DESC, es.setNumber ASC;
```

**Result**: Chronological list of all recorded sets with exercise names, dates.

---

### Example 3: Calculating Personal Record

**Query**:
```sql
SELECT e.id, e.name, MAX(es.weight) as pr, es.weightUnit
FROM exercises e
LEFT JOIN exercise_sets es ON e.id = es.exerciseId
GROUP BY e.id
ORDER BY pr DESC;
```

**Result**: Maximum weight lifted per exercise, unit preserved from original entry.

---

## Constraints & Validation Rules

### Constitution VIII: Data Integrity & Format Preservation

✅ **Implementation**:
- `weightUnit` stored with EVERY ExerciseSet record (never retroactively converted)
- `sessionDate` stored as date-only (no time component, per spec)
- User preference for unit display is READ-ONLY transformation (display layer only)
- Example: User enters "100 kg", preferences changes to "lbs" → stored as "100 kg", displayed as "~220 lbs" (calculated in view layer, not DB)

### Constitution III: Test-First

✅ **Entities designed for testability**:
- All fields have clear validation rules (unit tests verify)
- Foreign key relationships enforced (integration tests verify)
- Cascade delete behavior explicit (integration tests verify)

---

## Migration Strategy

### Migration v1.0 (Initial)
Create all 4 tables as documented above.

### Future Migrations
- v1.1: Add logging/audit fields (timestamps already present)
- v1.2: Add tags/labels for exercises (extend Exercise table)
- v2.0: Add cloud sync fields (not in MVP scope)

**Pattern**: All migrations use Room's @Migration annotation; schema versioning incremental.

---

## Performance Considerations

### Query Performance (MVP Targets)

| Query | Complexity | Expected Time | Notes |
|-------|-----------|---|---|
| Fetch 365 sessions (1 year) | O(n) with index | <100ms | Index on sessionDate |
| Get all sets for exercise | O(log n) | <50ms | Index on exerciseId |
| Calculate personal records | O(n) aggregate | <200ms | One-time on app load, cached |
| Graph data (30-day progression) | O(n) filtered | <150ms | 30-day max weight per day |

### Optimization Notes
- No explicit query optimization needed for MVP
- Indexes on sessionDate, exerciseId sufficient
- Future P2 feature (graphs) may need pagination/aggregation queries

---

## Concurrency & Transactions

**Policy**: Single-user app, no multi-process access.

**Room Handling**:
- Room uses SQLite WAL (Write-Ahead Logging) for concurrency
- Android handles threading via Coroutines
- ViewModel layer ensures main-thread safety

**No explicit transaction management needed for MVP** (YAGNI).

---

## Backup & Export (Future Considerations)

- **CSV Export** (US3 requirement FR-017): Queries ExerciseSet table, writes rows as CSV
- **Format**: date, exercise_name, weight, original_unit, reps, set_number, notes
- **Backup Strategy**: Defer to P2 (cloud backup not in scope)
