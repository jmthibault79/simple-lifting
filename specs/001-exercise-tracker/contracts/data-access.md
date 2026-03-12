# API Contracts: Exercise Tracker

**Phase**: 1 - Design & Contracts  
**Date**: 2026-03-11  
**Scope**: MVP (US1 + US2) - Local data access layer (no network APIs)

---

## Overview

For the MVP, there are no external network APIs (app is offline-first, local-only). This document defines the **internal data access contracts** that the UI layer interacts with:

1. **ExerciseRepository Contract** - CRUD operations for exercises
2. **SessionRepository Contract** - CRUD operations for workout sessions
3. **ExerciseSetRepository Contract** - CRUD operations for individual exercise sets
4. **ExerciseHistoryQueryContract** - Query patterns for history screen

---

## 1. ExerciseRepository Contract

**Purpose**: Manage Exercise entities (exercise types)

### Methods

#### `suspend fun createOrGetExercise(name: String, description: String? = null): Exercise`

**Input**:
- `name`: Exercise name (e.g., "Benchpress")
- `description`: Optional description

**Output**: Exercise entity (existing if found, new if created)

**Behavior**:
- If exercise with name exists, returns existing record
- If not, creates new Exercise and returns it
- Personal record updated elsewhere (when ExerciseSet recorded)

**Errors**:
- Throws if name is empty or null
- Throws if database error occurs

---

#### `suspend fun getExerciseById(id: Long): Exercise?`

**Input**: `id` - Exercise ID

**Output**: Exercise or null if not found

---

#### `suspend fun getAllExercises(): List<Exercise>`

**Input**: None

**Output**: Sorted list of all Exercise entities (alphabetical by name)

---

#### `suspend fun searchExercises(query: String): List<Exercise>`

**Input**: `query` - Partial exercise name (e.g., "bench" matches "Benchpress")

**Output**: Filtered list of Exercise entities

**Implementation**: Case-insensitive LIKE query

---

#### `suspend fun getPersonalRecord(exerciseId: Long): Double?`

**Input**: `exerciseId`

**Output**: Max weight ever lifted (numeric value only, without unit)

**Notes**: Unit is stored separately (ExerciseSet.weightUnit)

---

## 2. SessionRepository Contract

**Purpose**: Manage ExerciseSession entities (daily workout sessions)

### Methods

#### `suspend fun createSession(sessionDate: LocalDate, notes: String? = null): ExerciseSession`

**Input**:
- `sessionDate`: Date (local timezone)
- `notes`: Optional session-level notes

**Output**: Created ExerciseSession entity

**Errors**: Throws if date is in future

---

#### `suspend fun getSessionByDate(sessionDate: LocalDate): ExerciseSession?`

**Input**: `sessionDate`

**Output**: ExerciseSession or null if no session for that date

---

#### `suspend fun getAllSessions(): List<ExerciseSession>`

**Input**: None

**Output**: Chronologically ordered list (most recent first)

---

#### `suspend fun getSessionsInRange(startDate: LocalDate, endDate: LocalDate): List<ExerciseSession>`

**Input**: Date range

**Output**: Sessions within range, chronologically ordered

---

#### `suspend fun deleteSession(sessionId: Long)`

**Input**: `sessionId`

**Behavior**: Deletes session and cascades delete all associated ExerciseSets

---

## 3. ExerciseSetRepository Contract

**Purpose**: Manage ExerciseSet entities (individual sets)

### Methods

#### `suspend fun recordSet(exerciseId: Long, sessionId: Long, weight: Double, weightUnit: String, reps: Int, notes: String? = null): ExerciseSet`

**Input**:
- `exerciseId`: Which exercise
- `sessionId`: Which session
- `weight`: Numeric weight value
- `weightUnit`: "lbs" or "kg" (IMMUTABLE)
- `reps`: Number of reps (can be 0 for failed sets)
- `notes`: Optional per-set notes

**Output**: Created ExerciseSet entity with auto-assigned setNumber

**Validation**:
- weight ≥ 0, < 10,000
- reps ≥ 0, < 1,000
- weightUnit in ["lbs", "kg"]

**Errors**: Throws if validation fails

---

#### `suspend fun getSetsBySession(sessionId: Long): List<ExerciseSet>`

**Input**: `sessionId`

**Output**: All sets in session (ordered by setNumber)

---

#### `suspend fun getSetsByExercise(exerciseId: Long): List<ExerciseSet>`

**Input**: `exerciseId`

**Output**: All sets for exercise across all sessions (chronological)

---

#### `suspend fun deleteSet(setId: Long)`

**Input**: `setId`

**Behavior**: Deletes single set, preserves others and session record

---

#### `suspend fun updateSet(setId: Long, weight: Double? = null, reps: Int? = null, notes: String? = null): ExerciseSet`

**Input**: Set ID + fields to update

**Output**: Updated ExerciseSet

**Notes**: `weightUnit` cannot be changed (immutable per Constitution VIII)

---

## 4. ExerciseHistoryQueryContract

**Purpose**: Specialized queries for History Screen (US2)

### Methods

#### `suspend fun getHistoryForDisplay(limit: Int = 100, offset: Int = 0): List<HistoryDisplayItem>`

**Input**:
- `limit`: Number of records to fetch (pagination)
- `offset`: Starting position

**Output**: Paginated list of display items

**Data Structure** (HistoryDisplayItem):
```kotlin
data class HistoryDisplayItem(
    val setId: Long,
    val exerciseName: String,
    val sessionDate: LocalDate,
    val weight: Double,
    val weightUnit: String,
    val reps: Int,
    val setNumber: Int,
    val notes: String?
)
```

**Behavior**:
- Joins Exercise, ExerciseSession, and ExerciseSet
- Ordered by sessionDate DESC (most recent first)
- Includes unit with weight for display

---

#### `suspend fun filterHistoryByExercise(exerciseId: Long): List<HistoryDisplayItem>`

**Input**: `exerciseId`

**Output**: History filtered to only that exercise (most recent first)

---

#### `suspend fun getDailySessionSummary(sessionDate: LocalDate): List<ExerciseDailySummary>`

**Input**: `sessionDate`

**Output**: Summary of all exercises for that day

**Data Structure** (ExerciseDailySummary):
```kotlin
data class ExerciseDailySummary(
    val exerciseName: String,
    val weight: Double,
    val weightUnit: String,
    val totalReps: Int,       // sum across all sets
    val setCount: Int,
    val notes: String?        // session-level notes
)
```

---

## 5. ViewerPreferenceContract

**Purpose**: User display preferences (weight unit conversion for viewing only)

### Methods

#### `suspend fun setPreferredWeightUnit(unit: String)`

**Input**: `unit` - "lbs" or "kg"

**Behavior**: Sets user preference; affects display layer only, not stored data

---

#### `suspend fun getPreferredWeightUnit(): String`

**Output**: "lbs" or "kg"

**Default**: "lbs"

---

#### `fun convertWeight(weightValue: Double, fromUnit: String, toUnit: String): Double`

**Input**:
- `weightValue`: Original value
- `fromUnit`: "lbs" or "kg"
- `toUnit`: "lbs" or "kg"

**Output**: Converted value

**Implementation**: `lbs → kg: value / 2.20462`, `kg → lbs: value * 2.20462`

**Notes**: Used ONLY in view layer for display; stored data never modified

---

## Data Flow Diagrams

### US1: Record Exercise Session

```
User enters form:
  Exercise name "Benchpress"
  Weight 185 lbs
  Reps 8
  Sets 4
        ↓
ViewModel calls ExerciseRepository.createOrGetExercise("Benchpress")
        ↓
Repository queries Exercise table, creates if not exist
        ↓
For each set (1-4):
  ViewModel calls ExerciseSetRepository.recordSet(...)
        ↓
Repository inserts row in ExerciseSet table with:
  exerciseId, sessionId, weight=185, weightUnit="lbs", reps=8, setNumber=N
        ↓
Repository updates Exercise.personalRecord = 185 if greater
        ↓
UI displays "Exercise recorded"
```

---

### US2: View Exercise History

```
User taps "History" tab
        ↓
ViewModel calls ExerciseHistoryQueryContract.getHistoryForDisplay(limit=100)
        ↓
Repository executes joined query:
  SELECT e.name, sess.sessionDate, es.weight, es.weightUnit, ...
  JOIN exercises, sessions, sets
  ORDER BY sessionDate DESC
        ↓
Repository applies ConvertWeight (if user preference != stored unit)
        ↓
UI receives HistoryDisplayItem list
        ↓
UI renders Compose list of exercises, grouped by date
```

---

## Contract Stability

All contracts above are **stable for MVP phase**. Changes expected:

| Contract | Stability | Notes |
|----------|-----------|-------|
| ExerciseRepository | 🟢 Stable | Core functionality unlikely to change |
| SessionRepository | 🟢 Stable | Simple CRUD, no changes expected |
| ExerciseSetRepository | 🟢 Stable | Core data recording action |
| ExerciseHistoryQueryContract | 🟡 May expand | P2 features (filtering, sorting) will add methods |
| ViewerPreferenceContract | 🟡 May expand | P2 theme support, other prefs |

---

## Error Handling Contract

All `suspend` functions follow these error patterns:

1. **Validation errors**: Throw `IllegalArgumentException` with message
2. **Database errors**: Throw `Exception` (Room wraps SQLite errors)
3. **Not found**: Return null (or empty list)

**ViewModel Error Handling**:
- Catch exceptions in ViewModel, convert to user-friendly UI state
- Example: Validation error "weight must be > 0" → display Toast to user

---

## CSV Export Contract (Future P2)

Reserved for future use (FR-017):

```kotlin
suspend fun exportHistoryAsCSV(startDate: LocalDate? = null, endDate: LocalDate? = null): String
```

**Output**: CSV formatted string with headers:
```
date,exercise_name,weight,original_unit,reps,set_number,notes
2026-03-11,Benchpress,185,lbs,8,1,felt strong
2026-03-11,Benchpress,185,lbs,8,2,
...
```
