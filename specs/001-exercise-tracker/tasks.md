# Tasks: Exercise Tracker MVP Implementation

**Feature**: Weight Training Exercise Tracker (001-exercise-tracker)  
**Phase**: 2 - Implementation  
**Date**: 2026-03-11  
**Status**: Ready for Development  
**MVP Scope**: User Stories 1 & 2 (Record Exercise + View History) with P1 Priority

**Pre-Flight Decisions** (2026-03-12):
- ✅ **Dashboard**: Not implemented for MVP. App starts with RecordExerciseScreen (entry form) as home screen.
- ✅ **History Grouping**: Group by DAY (not week). Each calendar day shown as a header with exercises performed that day grouped below.
- ✅ **Notes**: Per-exercise per-session (not per-set). Users add notes for each exercise performed in a session (e.g., "Benchpress: felt strong" for all 4 sets of Benchpress on March 3).

---

## Overview: Thin-Slice Architecture

**Core Principle**: Every task results in a compilable, runnable state on Android device. No broken dependencies or incomplete code.

**Phase Progression**:
1. **Phase 1**: Setup (gradle, project structure) — COMPLETE ✅
2. **Phase 2**: Data Layer Foundation (entities, DAOs, database, converters) — FOUNDATIONAL (blocks all features)
3. **Phase 3**: Repositories & Use Cases (data access abstraction) — FOUNDATIONAL (blocks all features)
4. **Phase 4**: US1 - Record Exercise (P1) — PRIMARY FEATURE
5. **Phase 5**: US2 - View History (P1) — PRIMARY FEATURE
6. **Phase 6**: Polish & Integration (edge cases, validation, performance)

**Dependency Order**: Phases 2 and 3 must complete before Phases 4-5. Within phases, parallelizable tasks are marked [P].

---

## Phase 2: Data Layer Foundation (Database Entities & DAOs)

> **Goal**: Build Room database schema, entity classes, DAOs, and type converters. After Phase 2, the app compiles with fully functional data layer stubs.

### Data Models & Types

- [ ] **T001** Create `LocalDate` and `LocalDateTime` type converters for Room
  - File: `android/app/src/main/kotlin/com/simplelifting/data/db/converters/DateTimeConverters.kt`
  - Task: Implement @TypeConverter for LocalDate ↔ String (ISO 8601, local timezone)
  - Task: Implement @TypeConverter for LocalDateTime ↔ Long (Unix timestamp)
  - Stub: No logic needed, just type mappings
  - Compilable: Yes (new file, imported by entities in T002)

- [ ] **T002** [P] Create Exercise entity class
  - File: `android/app/src/main/kotlin/com/simplelifting/data/entity/ExerciseEntity.kt`
  - Task: Define @Entity class with id, name, description, personalRecord, personalRecordUnit, lastPerformed, createdAt
  - Stub: No behavior, just data class
  - Validation: Add @PrimaryKey, @ColumnInfo annotations per Room spec
  - Compilable: Yes (plain data class)

- [ ] **T003** [P] Create ExerciseSession entity class
  - File: `android/app/src/main/kotlin/com/simplelifting/data/entity/ExerciseSessionEntity.kt`
  - Task: Define @Entity class with id, sessionDate, notes, createdAt, updatedAt
  - Stub: No behavior, just data class
  - Validation: Add @PrimaryKey, @ColumnInfo, @Index(sessionDate DESC) annotations
  - Compilable: Yes (plain data class)

- [ ] **T004** [P] Create ExerciseSet entity class
  - File: `android/app/src/main/kotlin/com/simplelifting/data/entity/ExerciseSetEntity.kt`
  - Task: Define @Entity class with id, exerciseId, sessionId, weight, weightUnit, repsCompleted, setNumber, notes, createdAt
  - Stub: No behavior, just data class
  - Validation: Add @PrimaryKey, @ForeignKey, @Index annotations per Room spec
  - Compilable: Yes (plain data class)

- [ ] **T005** [P] Create UserPreference entity class
  - File: `android/app/src/main/kotlin/com/simplelifting/data/entity/UserPreferenceEntity.kt`
  - Task: Define @Entity class with id (fixed 1), preferredWeightUnit, theme, createdAt, updatedAt
  - Stub: No behavior, just data class
  - Validation: Add @PrimaryKey, @ColumnInfo annotations
  - Compilable: Yes (plain data class)

- [ ] **T005b** [P] Create ExerciseSessionEntry entity class
  - File: `android/app/src/main/kotlin/com/simplelifting/data/entity/ExerciseSessionEntry.kt`
  - Task: Define @Entity class with id, exerciseId (FK), sessionId (FK), notes (optional), createdAt
  - Task: Represents "this exercise performed in this session" - one record per unique (exercise, session) pair
  - Task: Notes field stores per-exercise notes (e.g., "felt strong", "form sloppy")
  - Validation: Add @PrimaryKey, @ForeignKey, @Index, @Unique(exerciseId, sessionId) annotations
  - Purpose: Enables per-exercise notes instead of per-set notes (cleaner UX)
  - Compilable: Yes (plain data class)

### Data Access Objects (DAOs)

- [ ] **T006** [P] Create ExerciseDao interface
  - File: `android/app/src/main/kotlin/com/simplelifting/data/db/dao/ExerciseDao.kt`
  - Task: Define DAO interface with methods: insertOrIgnore, getById, getAll, searchByName, updatePersonalRecord, deleteById
  - Stub: Add TODO comments for implementation; use @Insert, @Query, @Update Room annotations
  - Compilable: Yes (interface with annotations, not implemented)

- [ ] **T007** [P] Create ExerciseSessionDao interface
  - File: `android/app/src/main/kotlin/com/simplelifting/data/db/dao/ExerciseSessionDao.kt`
  - Task: Define DAO interface with methods: insert, getById, getByDate, getAll, getAllOrderedByDateDesc, update, delete
  - Stub: Add TODO comments; use @Insert, @Query, @Update, @Delete Room annotations
  - Compilable: Yes (interface with annotations)

- [ ] **T008** [P] Create ExerciseSetDao interface
  - File: `android/app/src/main/kotlin/com/simplelifting/data/db/dao/ExerciseSetDao.kt`
  - Task: Define DAO interface with methods: insert, getById, getBySessionId, getByExerciseId, getByExerciseAndDateRange, deleteById
  - Stub: Add TODO comments; use @Insert, @Query, @Delete Room annotations
  - Compilable: Yes (interface with annotations)

- [ ] **T009** [P] Create UserPreferenceDao interface
  - File: `android/app/src/main/kotlin/com/simplelifting/data/db/dao/UserPreferenceDao.kt`
  - Task: Define DAO interface with methods: insert, getPreference, updateWeightUnit, update
  - Stub: Add TODO comments; use @Insert, @Query, @Update Room annotations
  - Compilable: Yes (interface with annotations)

- [ ] **T009b** [P] Create ExerciseSessionEntryDao interface
  - File: `android/app/src/main/kotlin/com/simplelifting/data/db/dao/ExerciseSessionEntryDao.kt`
  - Task: Define DAO interface with methods: insertOrUpdate, getByExerciseAndSession, getBySessionId, deleteBySessionId, updateNotes
  - Stub: Add TODO comments; use @Insert, @Query, @Update Room annotations (OnConflictStrategy.REPLACE)
  - Compilable: Yes (interface with annotations)

### Database Setup

- [ ] **T010** Create AppDatabase (Room database setup)
  - File: `android/app/src/main/kotlin/com/simplelifting/data/db/AppDatabase.kt`
  - Task: Define @Database abstract class with all 5 entities (Exercise, ExerciseSession, ExerciseSet, UserPreference, ExerciseSessionEntry) + 4 TypeConverters
  - Task: Expose abstract getter methods for all 5 DAOs (not implemented, just method signatures)
  - Task: Create companion object with in-memory database builder for testing
  - Stub: Singleton instance creation; use getInstance() static method returning stub DB
  - Validation: Verify entities are registered, version = 1
  - Compilable: Yes (database class with stub DAOs)

- [ ] **T011** Implement DAO query methods (SQLite compatibility)
  - File: All files from T006-T009 (ExerciseDao, ExerciseSessionDao, ExerciseSetDao, UserPreferenceDao)
  - Task: Replace @Query TODO comments with actual SQL queries (no implementation, queries are static)
  - Task: Verify queries match Room syntax and database schema (async/suspend functions)
  - Validation: Compile check; no runtime execution needed
  - Compilable: Yes (SQL queries defined, methods now callable but stub)

---

## Phase 3: Repositories & Use Cases (Data Access Abstraction)

> **Goal**: Build repository interfaces and implementations that wrap DAOs. Add business logic layer. After Phase 3, the data pipeline from UI → DB is complete (stubs ready for logic).

### Domain Models (Not Database Entities)

- [ ] **T012** [P] Create Exercise domain model
  - File: `android/app/src/main/kotlin/com/simplelifting/domain/model/Exercise.kt`
  - Task: Define data class with id, name, description, personalRecord, personalRecordUnit, lastPerformed, createdAt
  - Stub: No conversion logic, just data class
  - Compilable: Yes (plain data class, separate from entity)

- [ ] **T013** [P] Create ExerciseSession domain model
  - File: `android/app/src/main/kotlin/com/simplelifting/domain/model/ExerciseSession.kt`
  - Task: Define data class with id, sessionDate, notes, createdAt, updatedAt
  - Stub: No conversion logic, just data class
  - Compilable: Yes (plain data class)

- [ ] **T014** [P] Create ExerciseSet domain model
  - File: `android/app/src/main/kotlin/com/simplelifting/domain/model/ExerciseSet.kt`
  - Task: Define data class with id, exerciseId, sessionId, weight, weightUnit, repsCompleted, setNumber, notes, createdAt
  - Stub: No conversion logic, just data class
  - Compilable: Yes (plain data class)

- [ ] **T015** [P] Create UserPreference domain model
  - File: `android/app/src/main/kotlin/com/simplelifting/domain/model/UserPreference.kt`
  - Task: Define data class with id, preferredWeightUnit, theme, createdAt, updatedAt
  - Stub: No conversion logic, just data class
  - Compilable: Yes (plain data class)

### Mappers (Entity ↔ Domain Model)

- [ ] **T016** [P] Create EntityMapper for Exercise
  - File: `android/app/src/main/kotlin/com/simplelifting/data/mapper/ExerciseMapper.kt`
  - Task: Implement toDomainModel(entity) and toEntity(domain) functions
  - Stub: Simple field copying, no conversion logic
  - Compilable: Yes (utility functions)

- [ ] **T017** [P] Create EntityMapper for ExerciseSession
  - File: `android/app/src/main/kotlin/com/simplelifting/data/mapper/ExerciseSessionMapper.kt`
  - Task: Implement toDomainModel(entity) and toEntity(domain) functions
  - Compilable: Yes (utility functions)

- [ ] **T018** [P] Create EntityMapper for ExerciseSet
  - File: `android/app/src/main/kotlin/com/simplelifting/data/mapper/ExerciseSetMapper.kt`
  - Task: Implement toDomainModel(entity) and toEntity(domain) functions
  - Compilable: Yes (utility functions)

- [ ] **T019** [P] Create EntityMapper for UserPreference
  - File: `android/app/src/main/kotlin/com/simplelifting/data/mapper/UserPreferenceMapper.kt`
  - Task: Implement toDomainModel(entity) and toEntity(domain) functions
  - Compilable: Yes (utility functions)

### Repository Interfaces

- [ ] **T020** [P] Create ExerciseRepository interface
  - File: `android/app/src/main/kotlin/com/simplelifting/domain/repository/ExerciseRepository.kt`
  - Task: Define interface with suspend functions: createOrGetExercise, getExerciseById, getAllExercises, searchExercises, getPersonalRecord
  - Stub: No implementation, just interface contract
  - Compilable: Yes (interface)

- [ ] **T021** [P] Create SessionRepository interface
  - File: `android/app/src/main/kotlin/com/simplelifting/domain/repository/SessionRepository.kt`
  - Task: Define interface with suspend functions: createSession, getSessionByDate, getSessionsInDateRange, getAllSessions, updateSession, deleteSession
  - Stub: No implementation, just interface contract
  - Compilable: Yes (interface)

- [ ] **T022** [P] Create ExerciseSetRepository interface
  - File: `android/app/src/main/kotlin/com/simplelifting/domain/repository/ExerciseSetRepository.kt`
  - Task: Define interface with suspend functions: insertSet, getSetById, getSetsBySessionId, getSetsByExerciseId, getSetsByExerciseAndDateRange, deleteSet, updateSet
  - Stub: No implementation, just interface contract
  - Compilable: Yes (interface)

- [ ] **T023** [P] Create UserPreferenceRepository interface
  - File: `android/app/src/main/kotlin/com/simplelifting/domain/repository/UserPreferenceRepository.kt`
  - Task: Define interface with suspend functions: getPreference, getPreferenceOrDefault, setWeightUnit, updatePreference
  - Stub: No implementation, just interface contract
  - Compilable: Yes (interface)

### Repository Implementations

- [ ] **T024** Create ExerciseRepositoryImpl
  - File: `android/app/src/main/kotlin/com/simplelifting/data/repository/ExerciseRepositoryImpl.kt`
  - Task: Implement ExerciseRepository interface using ExerciseDao
  - Task: Implement createOrGetExercise with insert-or-get logic
  - Task: Implement other methods delegating to DAO calls
  - Stub: // TODO: Implement [method name] after DAO test
  - Compilable: Yes (class implemented, DAO calls stubbed with TODO)

- [ ] **T025** Create SessionRepositoryImpl
  - File: `android/app/src/main/kotlin/com/simplelifting/data/repository/SessionRepositoryImpl.kt`
  - Task: Implement SessionRepository interface using SessionDao
  - Task: Implement all methods delegating to DAO
  - Stub: // TODO: Implement after DAO test
  - Compilable: Yes (class implemented, DAO calls stubbed)

- [ ] **T026** Create ExerciseSetRepositoryImpl
  - File: `android/app/src/main/kotlin/com/simplelifting/data/repository/ExerciseSetRepositoryImpl.kt`
  - Task: Implement ExerciseSetRepository interface using ExerciseSetDao
  - Task: Implement all methods delegating to DAO
  - Stub: // TODO: Implement after DAO test
  - Compilable: Yes (class implemented, DAO calls stubbed)

- [ ] **T027** Create UserPreferenceRepositoryImpl
  - File: `android/app/src/main/kotlin/com/simplelifting/data/repository/UserPreferenceRepositoryImpl.kt`
  - Task: Implement UserPreferenceRepository interface using UserPreferenceDao
  - Task: Implement all methods delegating to DAO
  - Stub: // TODO: Implement after DAO test
  - Compilable: Yes (class implemented, DAO calls stubbed)

### Use Cases (Business Logic)

- [ ] **T028** [P] Create RecordExerciseUseCase
  - File: `android/app/src/main/kotlin/com/simplelifting/domain/usecase/RecordExerciseUseCase.kt`
  - Task: Define use case class with execute(exerciseName, weight, unit, reps, sets) suspend function
  - Task: Accepts: exercise name (string), weight (double), weight unit (lbs/kg), reps completed (int), number of sets (int)
  - Task: Returns: Result<ExerciseSet> or similar
  - Stub: // TODO: Implement recording logic after repository setup
  - Validation: Input validation (positive weight, reps, sets, non-empty name)
  - Compilable: Yes (class with stub execute function)

- [ ] **T029** [P] Create GetExerciseHistoryUseCase
  - File: `android/app/src/main/kotlin/com/simplelifting/domain/usecase/GetExerciseHistoryUseCase.kt`
  - Task: Define use case class with execute(dateRange?, exerciseName?) suspend function
  - Task: Returns: List<ExerciseWithSets> (exercise + all sets from that session)
  - Task: Supports filtering by exercise name, date range
  - Stub: // TODO: Implement query logic
  - Compilable: Yes (class with stub execute function)

- [ ] **T030** [P] Create GetUserPreferenceUseCase
  - File: `android/app/src/main/kotlin/com/simplelifting/domain/usecase/GetUserPreferenceUseCase.kt`
  - Task: Define use case class with execute() suspend function
  - Task: Returns: UserPreference (with default values if not set)
  - Stub: // TODO: Implement
  - Compilable: Yes (class with stub)

---

## Phase 4: User Story 1 - Record Exercise (P1 Priority)

> **Goal**: Build UI screens, ViewModels, and integration for users to record exercises. After Phase 4, users can launch app, enter exercise data (name, weight, reps, sets), and save to database.

### Domain Model for US1

- [ ] **T031** Create RecordExerciseArgs data class
  - File: `android/app/src/main/kotlin/com/simplelifting/domain/model/RecordExerciseArgs.kt`
  - Task: Define data class with exerciseName, weight, weightUnit, repsCompleted, numberOfSets, sessionDate, notes
  - Stub: No logic, just data class
  - Compilable: Yes

### ViewModel for US1

- [ ] **T032** Create RecordExerciseViewModel
  - File: `android/app/src/main/kotlin/com/simplelifting/viewmodel/RecordExerciseViewModel.kt`
  - Task: Define ViewModel with state: MutableLiveData<ExerciseFormState>
  - Task: Add methods: updateExerciseName, updateWeight, updateUnit, updateReps, updateSets, updateNotes, saveExercise, observeFormState
  - Task: Notes field for per-exercise notes (e.g., "felt strong", "form needs work")
  - Task: Use RecordExerciseUseCase and repositories for business logic
  - Stub: // TODO: Call use case on save
  - Validation: Form state includes loading, error, success flags
  - Compilable: Yes (class with stub use case calls)

### UI Components for US1

- [ ] **T033** [P] Create ExerciseNameInputComposable
  - File: `android/app/src/main/kotlin/com/simplelifting/ui/components/ExerciseNameInput.kt`
  - Task: Composable for exercise name input with autocomplete dropdown (from getAllExercises)
  - Task: Shows list of known exercises (from history + predefined common exercises)
  - Task: Supports typed filtering/search
  - Task: Callback: onNameSelected(name: String)
  - Stub: // TODO: Populate dropdown from repository
  - Compilable: Yes (Composable with text field + empty list stub)

- [ ] **T034** [P] Create WeightInputComposable
  - File: `android/app/src/main/kotlin/com/simplelifting/ui/components/WeightInput.kt`
  - Task: Composable for weight input (numeric field) + unit selector (lbs/kg dropdown)
  - Task: Callbacks: onWeightChanged, onUnitChanged
  - Stub: Unit selector starts with user preference (fetch from repo)
  - Compilable: Yes (text field + dropdown with placeholder)

- [ ] **T035** [P] Create RepsInputComposable
  - File: `android/app/src/main/kotlin/com/simplelifting/ui/components/RepsInput.kt`
  - Task: Composable for reps input (numeric field, allows 0 for failed sets)
  - Task: Callback: onRepsChanged(reps: Int)
  - Compilable: Yes (text field with validation)

- [ ] **T036** [P] Create SetsInputComposable
  - File: `android/app/src/main/kotlin/com/simplelifting/ui/components/SetsInput.kt`
  - Task: Composable for number of sets input (numeric field)
  - Task: Callback: onSetsChanged(sets: Int)
  - Compilable: Yes (text field with validation)

- [ ] **T037** [P] Create NotesInputComposable
  - File: `android/app/src/main/kotlin/com/simplelifting/ui/components/NotesInput.kt`
  - Task: Composable for per-exercise notes input (multiline text field, optional)
  - Task: Placeholder: "e.g., felt strong, form needs work"
  - Task: Callback: onNotesChanged(notes: String)
  - Compilable: Yes (text field)

- [ ] **T038** [P] Create SaveExerciseButtonComposable
  - File: `android/app/src/main/kotlin/com/simplelifting/ui/components/SaveExerciseButton.kt`
  - Task: Composable button for submitting exercise record
  - Task: Shows loading state while saving
  - Task: Callback: onSaveClicked()
  - Compilable: Yes (button with state)

### Screen for US1

- [ ] **T039** Create RecordExerciseScreen
  - File: `android/app/src/main/kotlin/com/simplelifting/ui/screens/RecordExerciseScreen.kt`
  - Task: Compose the full exercise entry form screen
  - Task: Assemble: ExerciseNameInput + WeightInput + RepsInput + SetsInput + NotesInput + SaveExerciseButton
  - Task: Connect to RecordExerciseViewModel (wire state & callbacks)
  - Task: Show error messages if validation fails
  - Task: Show success message + clear form after save
  - Task: Get current date/time for session creation
  - Stub: // TODO: On save button, call viewModel.saveExercise()
  - Validation: Form validates before enabling save button
  - Compilable: Yes (screen assembles all components, viewModel calls stubbed)

### MainActivity Navigation for US1

- [ ] **T040** Update MainActivity to show RecordExerciseScreen
  - File: `android/app/src/main/kotlin/com/simplelifting/MainActivity.kt`
  - Task: Add RecordExerciseScreen as the initial/home screen
  - Task: Wire ViewModels and repositories (manual dependency injection or factory)
  - Task: Create database instance on app launch
  - Task: Initialize UserPreference with default (if not exists) in RecordExerciseScreen onCompose
  - Stub: // TODO: Implement DAO methods
  - Compilable: Yes (screen loads, components render, buttons not functional)

### Use Case Implementation for US1

- [ ] **T041** Implement RecordExerciseUseCase.execute()
  - File: `android/app/src/main/kotlin/com/simplelifting/domain/usecase/RecordExerciseUseCase.kt`
  - Task: Replace stub with actual implementation
  - Task: Logic:
    - Get or create Exercise entity (using ExerciseRepository.createOrGetExercise)
    - Create ExerciseSession for today if not exists
    - Insert ExerciseSet record for each set (create N ExerciseSet entries, one per set number)
    - Update Exercise.personalRecord if weight exceeds current PR
    - Return list of inserted ExerciseSets or Result<List<ExerciseSet>>
  - Task: Add input validation (throw IllegalArgumentException for invalid inputs)
  - Compilable: Yes (business logic implemented, DB calls now functional)

### Repository Implementation for US1

- [ ] **T042** [P] Implement ExerciseRepositoryImpl.createOrGetExercise()
  - File: `android/app/src/main/kotlin/com/simplelifting/data/repository/ExerciseRepositoryImpl.kt`
  - Task: Replace stub with actual implementation
  - Task: Logic: Query ExerciseDao.getByName(name) → if exists return, else insert + create + return
  - Task: Handle Exercise not found gracefully (return null or throw)
  - Compilable: Yes (functional)

- [ ] **T043** [P] Implement ExerciseRepositoryImpl for US1 query methods
  - File: `android/app/src/main/kotlin/com/simplelifting/data/repository/ExerciseRepositoryImpl.kt`
  - Task: Implement: getExerciseById, getAllExercises, searchExercises, getPersonalRecord
  - Task: Add mappers to convert entities to domain models
  - Compilable: Yes (functional)

- [ ] **T044** [P] Implement SessionRepositoryImpl for US1 methods
  - File: `android/app/src/main/kotlin/com/simplelifting/data/repository/SessionRepositoryImpl.kt`
  - Task: Implement: createSession, getSessionByDate
  - Task: Add mappers
  - Compilable: Yes (functional)

- [ ] **T045** [P] Implement ExerciseSetRepositoryImpl for US1 methods
  - File: `android/app/src/main/kotlin/com/simplelifting/data/repository/ExerciseSetRepositoryImpl.kt`
  - Task: Implement: insertSet, updateSet (for PR calculation)
  - Task: Add mappers
  - Compilable: Yes (functional)

- [ ] **T046** [P] Implement UserPreferenceRepositoryImpl
  - File: `android/app/src/main/kotlin/com/simplelifting/data/repository/UserPreferenceRepositoryImpl.kt`
  - Task: Implement all methods
  - Task: Add logic: getPreferenceOrDefault returns default (lbs) if no record exists
  - Compilable: Yes (functional)

### DAO Implementation for US1

- [ ] **T047** Implement ExerciseDao query methods
  - File: `android/app/src/main/kotlin/com/simplelifting/data/db/dao/ExerciseDao.kt`
  - Task: Implement all query methods with actual SQL
  - Task: Methods needed for US1: insertOrIgnore, getById, getAll, searchByName, updatePersonalRecord
  - Task: Use @Query, @Insert, @Update Room annotations
  - Compilable: Yes (functional)

- [ ] **T048** Implement SessionDao methods for US1
  - File: `android/app/src/main/kotlin/com/simplelifting/data/db/dao/ExerciseSessionDao.kt`
  - Task: Implement: insert, getByDate, update
  - Compilable: Yes (functional)

- [ ] **T049** Implement ExerciseSetDao methods for US1
  - File: `android/app/src/main/kotlin/com/simplelifting/data/db/dao/ExerciseSetDao.kt`
  - Task: Implement: insert
  - Compilable: Yes (functional)

- [ ] **T050** Implement UserPreferenceDao
  - File: `android/app/src/main/kotlin/com/simplelifting/data/db/dao/UserPreferenceDao.kt`
  - Task: Implement all methods
  - Compilable: Yes (functional)

### Predefined Exercise List (Stub/Hardcoded)

- [ ] **T051** Create PredefinedExercises constant list
  - File: `android/app/src/main/kotlin/com/simplelifting/ui/data/PredefinedExercises.kt`
  - Task: Hardcoded list of common exercise names (Benchpress, Squats, Deadlifts, Pull-ups, etc.)
  - Task: Used in ExerciseNameInputComposable for initial dropdown (before user history available)
  - Compilable: Yes (const list)

### Integration Testing for US1

- [ ] **T052** Write integration test for RecordExerciseUseCase
  - File: `android/app/src/androidTest/kotlin/com/simplelifting/domain/usecase/RecordExerciseUseCaseTest.kt`
  - Task: Test: Record 1 exercise → verify ExerciseSet, Exercise, ExerciseSession created in DB
  - Task: Test: Record multiple sets of same exercise → verify all sets inserted
  - Task: Test: PR calculation → verify personalRecord updated correctly
  - Task: Test: Input validation → reject negative weight/reps/sets
  - Setup: Use in-memory Room database for tests
  - Compilable: Yes (test class)

### Manual Testing Checklist for US1

- [ ] **T053** Manual test: Record Exercise (US1 acceptance scenario 1)
  - Task: Launch app → Open RecordExerciseScreen
  - Task: Enter: Exercise name "Benchpress", weight "185", unit "lbs", reps "8", sets "4"
  - Task: Tap Save
  - Task: Verify: Database contains session + 4 ExerciseSet records (one per set)
  - Task: Verify: Exercise entity created with name="Benchpress", personalRecord=185
  - Prerequisite: T052 passes
  - Compilable: N/A (manual)

- [ ] **T054** Manual test: Persistence across restarts (US1 acceptance scenario 2)
  - Task: Complete T053 → Close app completely → Reopen app
  - Task: Verify (via logcat or DB inspection): Exercise data still present
  - Compilable: N/A (manual)

- [ ] **T055** Manual test: Form preservation on app pause (US1 acceptance scenario 3)
  - Task: Open RecordExerciseScreen → Enter "Benchpress" + "185" → Press home (pause app)
  - Task: Return to app
  - Task: Verify: Form data preserved (not lost)
  - Compilable: N/A (manual)

- [ ] **T056** Manual test: Input validation (US1 acceptance scenario 4)
  - Task: Enter negative weight (-100) → Tap Save → Verify error message shown, save blocked
  - Task: Enter zero reps → Tap Save → Verify allowed (accepted per spec)
  - Task: Leave exercise name empty → Tap Save → Verify error message
  - Compilable: N/A (manual)

---

## Phase 5: User Story 2 - View Exercise History (P1 Priority)

> **Goal**: Build history screen with chronological list, filtering, and detail view. After Phase 5, users can record exercises (P4) AND view all past exercises organized by date.

### Domain Model for US2

- [ ] **T057** Create ExerciseWithSets domain model
  - File: `android/app/src/main/kotlin/com/simplelifting/domain/model/ExerciseWithSets.kt`
  - Task: Define data class composing Exercise + List<ExerciseSet>
  - Task: Also include ExerciseSession details (sessionDate, notes)
  - Compilable: Yes (plain data class)

- [ ] **T058** Create ExerciseHistoryItem domain model (for UI list display)
  - File: `android/app/src/main/kotlin/com/simplelifting/domain/model/ExerciseHistoryItem.kt`
  - Task: Flattened data class for list display: exerciseName, weight, weightUnit, reps, sets, setNumber, sessionDate, notes
  - Task: Used by history list to avoid nested loops in UI
  - Compilable: Yes

### ViewModel for US2

- [ ] **T059** Create HistoryViewModel
  - File: `android/app/src/main/kotlin/com/simplelifting/viewmodel/HistoryViewModel.kt`
  - Task: Define ViewModel with state: MutableLiveData<List<ExerciseHistoryItem>>
  - Task: Add methods: loadHistory, filterByExerciseName, filterByDateRange, refresh, subscribeToHistoryUpdates
  - Task: Use GetExerciseHistoryUseCase to fetch data
  - Task: Sort chronologically (most recent first)
  - Stub: // TODO: Call use case to fetch
  - Compilable: Yes (class with stub use case calls)

### UI Components for US2

- [ ] **T060** [P] Create ExerciseHistoryItemComposable
  - File: `android/app/src/main/kotlin/com/simplelifting/ui/components/ExerciseHistoryItem.kt`
  - Task: Composable card showing: exercise name, weight+unit, reps, session date
  - Task: Callback: onItemClicked (for detail view in future)
  - Compilable: Yes (card/row composable)

- [ ] **T061** [P] Create ExerciseFilterComposable
  - File: `android/app/src/main/kotlin/com/simplelifting/ui/components/ExerciseFilter.kt`
  - Task: Composable for filtering history by exercise name
  - Task: Dropdown/autocomplete populated with unique exercises from history
  - Task: Callback: onFilterChanged(exerciseName: String?)
  - Compilable: Yes (dropdown with stub data)

- [ ] **T062** [P] Create EmptyHistoryComposable
  - File: `android/app/src/main/kotlin/com/simplelifting/ui/components/EmptyHistory.kt`
  - Task: Composable shown when no exercises recorded (per spec)
  - Task: Display message: "No exercises recorded yet. Tap Record to start!"
  - Task: Show prominent Record button linking to RecordExerciseScreen
  - Compilable: Yes (text + button)

### Screen for US2

- [ ] **T063** Create HistoryScreen with day-based grouping
  - File: `android/app/src/main/kotlin/com/simplelifting/ui/screens/HistoryScreen.kt`
  - Task: Compose the history list screen with day-level grouping
  - Task: Assemble: ExerciseFilterComposable + LazyColumn(DayHeader + ExerciseGroupComposable + ExerciseHistoryItemComposable*)
  - Task: Grouping structure: Date Header (e.g., "March 3") → Exercise Name (e.g., "Benchpress") → List of sets
  - Task: Show per-exercise notes under exercise name
  - Task: Connect to HistoryViewModel (wire state & callbacks)
  - Task: Load history on screen creation (LaunchedEffect)
  - Task: Show loading state while fetching
  - Task: Show error message on failure
  - Stub: // TODO: Implement day/@exercise grouping queries
  - Validation: Empty state shown when no data
  - Compilable: Yes (screen assembles components, viewModel calls stubbed)

### Navigation for US2

- [ ] **T064** Add HistoryScreen to MainActivity navigation
  - File: `android/app/src/main/kotlin/com/simplelifting/MainActivity.kt`
  - Task: Add bottom navigation or tab bar to switch between RecordExerciseScreen and HistoryScreen
  - Task: Use Jetpack Compose NavController or simple state for screen selection
  - Task: Wire ViewModels for both screens
  - Compilable: Yes (navigation working, screens switchable)

### Use Case Implementation for US2

- [ ] **T065** Implement GetExerciseHistoryUseCase.execute()
  - File: `android/app/src/main/kotlin/com/simplelifting/domain/usecase/GetExerciseHistoryUseCase.kt`
  - Task: Replace stub with actual implementation
  - Task: Logic:
    - Query all ExerciseSets ordered by sessionDate DESC (most recent first)
    - Group by (exercise name, session date)
    - Return list of ExerciseHistoryItem (flattened for UI consumption)
    - Support optional filtering by exercise name
    - Support optional filtering by date range
  - Task: Add input validation
  - Compilable: Yes (business logic implemented)

### Repository Implementation for US2

- [ ] **T066** [P] Implement ExerciseSetRepositoryImpl.getSetsByExerciseAndDateRange()
  - File: `android/app/src/main/kotlin/com/simplelifting/data/repository/ExerciseSetRepositoryImpl.kt`
  - Task: Implement to query sets in date range
  - Task: Add mapper to domain model
  - Compilable: Yes (functional)

- [ ] **T067** [P] Implement SessionRepositoryImpl.getAllSessions()
  - File: `android/app/src/main/kotlin/com/simplelifting/data/repository/SessionRepositoryImpl.kt`
  - Task: Implement to retrieve all sessions ordered by date DESC
  - Task: Add mapper
  - Compilable: Yes (functional)

- [ ] **T068** [P] Implement ExerciseRepositoryImpl.getAllExercises() fully (aggregate data)
  - File: `android/app/src/main/kotlin/com/simplelifting/data/repository/ExerciseRepositoryImpl.kt`
  - Task: Ensure getAllExercises returns all exercises recorded (from history)
  - Compilable: Yes (functional)

### DAO Implementation for US2

- [ ] **T069** Implement ExerciseSetDao.getByExerciseAndDateRange()
  - File: `android/app/src/main/kotlin/com/simplelifting/data/db/dao/ExerciseSetDao.kt`
  - Task: Implement SQL query: SELECT ExerciseSet WHERE exerciseId = ? AND sessionDate BETWEEN ? AND ? ORDER BY sessionDate DESC
  - Task: Add @Query annotation with proper SQL
  - Compilable: Yes (functional)

- [ ] **T070** Implement SessionDao.getAllOrderedByDateDesc()
  - File: `android/app/src/main/kotlin/com/simplelifting/data/db/dao/ExerciseSessionDao.kt`
  - Task: Implement SQL query: SELECT * FROM exercise_sessions ORDER BY sessionDate DESC
  - Compilable: Yes (functional)

- [ ] **T071** Implement ExerciseSetDao.getBySessionId() fully
  - File: `android/app/src/main/kotlin/com/simplelifting/data/db/dao/ExerciseSetDao.kt`
  - Task: Ensure complete implementation for fetching all sets in a session
  - Compilable: Yes (functional)

### Integration Testing for US2

- [ ] **T072** Write integration test for GetExerciseHistoryUseCase
  - File: `android/app/src/androidTest/kotlin/com/simplelifting/domain/usecase/GetExerciseHistoryUseCaseTest.kt`
  - Task: Setup: Insert multiple exercises from different dates into in-memory DB
  - Task: Test: Query all → verify returned in correct chronological order
  - Task: Test: Filter by exercise name → verify only that exercise returned
  - Task: Test: Filter by date range → verify only records in range returned
  - Task: Test: Empty history → verify empty list returned
  - Compilable: Yes (test class)

### Manual Testing Checklist for US2

- [ ] **T073** Manual test: View Exercise History (US2 acceptance scenario 1)
  - Prerequisites: Record 3+ exercises from different dates (or use T053 results)
  - Task: Open HistoryScreen
  - Task: Verify: All exercises displayed in reverse chronological order (most recent first)
  - Compilable: N/A (manual)

- [ ] **T074** Manual test: View Exercise Details (US2 acceptance scenario 2)
  - Prerequisites: Exercises recorded
  - Task: Open HistoryScreen → Tap on exercise item
  - Task: Verify: Full details shown (name, weight, reps, sets, date, time)
  - Note: Actual detail screen implementation in future iteration
  - Compilable: N/A (manual)

- [ ] **T075** Manual test: History Grouping by Day (US2 acceptance scenario 3)
  - Prerequisites: Record multiple exercises of different types on 2-3 different dates
  - Task: Open HistoryScreen
  - Task: Verify: Exercises grouped by calendar date (e.g., "March 3" header, then all exercises from that day)
  - Task: Verify: Within each day, exercises grouped by type (e.g., all Benchpress sets together)
  - Task: Verify: Notes shown for each exercise group
  - Compilable: N/A (manual)

- [ ] **T076** Manual test: History Performance (US2 acceptance scenario 4)
  - Prerequisites: Record 365 exercises (or simulate with seed data)
  - Task: Open HistoryScreen
  - Task: Verify: List loads within 2 seconds (per SC-002)
  - Task: Scroll through entire list
  - Task: Verify: No lag or stuttering
  - Compilable: N/A (manual)

- [ ] **T077** Manual test: Empty State (on first launch)
  - Prerequisites: Fresh install (or clear app data)
  - Task: Open HistoryScreen on first launch
  - Task: Verify: Empty state displayed with "Record Exercise" button
  - Task: Tap button → Verify: Navigates to RecordExerciseScreen
  - Compilable: N/A (manual)

---

## Phase 6: Polish & Integration

> **Goal**: Edge cases, error handling, validation, performance optimization, and final integration testing. After Phase 6, MVP is production-ready.

### Input Validation & Error Handling

- [ ] **T078** [P] Add comprehensive input validation to RecordExerciseUseCase
  - File: `android/app/src/main/kotlin/com/simplelifting/domain/usecase/RecordExerciseUseCase.kt`
  - Task: Validate: weight ≥ 0 and < 10,000, reps ≥ 0 and < 1,000, sets ≥ 1 and < 100, name not empty
  - Task: Throw IllegalArgumentException with descriptive message for each validation failure
  - Task: Test: All validation errors caught and displayed to user in UI
  - Compilable: Yes (error handling)

- [ ] **T079** [P] Add comprehensive input validation to UI layer
  - File: All Composable input components (T033-T037)
  - Task: Add real-time validation feedback: disable Save button until form valid
  - Task: Show inline error messages for invalid fields
  - Task: Show password-style error: red border on invalid fields
  - Compilable: Yes (UI updates)

- [ ] **T080** [P] Add error handling to ViewModels
  - Files: RecordExerciseViewModel, HistoryViewModel
  - Task: Add error state to MutableLiveData (error message, error type)
  - Task: Catch exceptions from use cases and map to user-friendly messages
  - Task: Show error messages in UI (snackbar or alert)
  - Task: Implement retry mechanism for transient errors
  - Compilable: Yes (error handling)

- [ ] **T081** [P] Add database error handling
  - Files: All DAO and repository implementations
  - Task: Catch SQLiteException and other database errors
  - Task: Log error with stacktrace (for debugging)
  - Task: Re-throw or wrap in domain exceptions (for use case layer)
  - Compilable: Yes (error handling)

### Coroutines & Threading

- [ ] **T082** Update use cases to use Dispatchers.IO for database operations
  - Files: AllUseCaseImplementations
  - Task: Wrap database calls in withContext(Dispatchers.IO)
  - Task: Keep business logic on Dispatchers.Default or Default
  - Task: Ensure UI updates stay on Dispatchers.Main
  - Compilable: Yes (coroutines updated)

- [ ] **T083** Add timeout handling to use cases
  - Files: RecordExerciseUseCase, GetExerciseHistoryUseCase
  - Task: Wrap use case execution in withTimeoutOrNull (e.g., 5 second timeout for record, 2 second for history)
  - Task: Return failure result or throw TimeoutException if exceeds limit
  - Compilable: Yes (timeout handling)

### Edge Cases & Robustness

- [ ] **T084** [P] Handle app pause/resume state preservation
  - File: RecordExerciseViewModel, HistoryViewModel
  - Task: Save form state to savedInstanceState on pause
  - Task: Restore form state on resume
  - Task: Verify form data not lost when user leaves app
  - Compilable: Yes (state saving)

- [ ] **T085** [P] Handle db access for fresh app (first launch, empty DB)
  - Files: UserPreferenceRepositoryImpl, all query methods
  - Task: Initialize UserPreference with default (lbs) if not exists on first query
  - Task: Handle empty exercise list gracefully (return empty list, not null)
  - Task: Handle empty session list gracefully
  - Compilable: Yes (initialization logic)

- [ ] **T086** [P] Handle large dataset performance
  - Files: HistoryViewModel, GetExerciseHistoryUseCase, DAO queries
  - Task: Add pagination to history list (load 50 at a time, load more on scroll)
  - Task: Add viewport clipping to history list (only render visible items, use LazyColumn)
  - Task: Optimize DAO queries: add proper indexes on exerciseId, sessionDate
  - Task: Test with synthetic data (1000+ exercises)
  - Compilable: Yes (performance optimizations)

### Type Converters & Data Preservation

- [ ] **T087** Verify LocalDate converter preserves timezone correctly
  - File: `android/app/src/main/kotlin/com/simplelifting/data/db/converters/DateTimeConverters.kt`
  - Task: Test: Record exercise on date X → Verify sessionDate = X (exactly, no timezone shift)
  - Task: Test across timezones: Change device timezone → Reopen app → Verify dates unchanged
  - Compilable: Yes (tests)

- [ ] **T088** Verify weight unit preservation (Constitution VIII)
  - Files: ExerciseSetEntity, ExerciseEntity, mappers
  - Task: Test: Record 185 lbs → Change preference to kg → Verify stored as "185 lbs" (not converted)
  - Task: Verify display converts (shows ~84 kg) but storage immutable
  - Compilable: Yes (tests)

### UI Polish

- [ ] **T089** [P] Add Material Design 3 theming
  - File: `android/app/src/main/kotlin/com/simplelifting/ui/theme/Theme.kt`
  - Task: Define Material3 color scheme (primary, secondary, tertiary, error colors)
  - Task: Apply theme to all screens
  - Task: Verify light mode and dark mode both work
  - Compilable: Yes (UI updated)

- [ ] **T090** [P] Add loading placeholders to UI
  - Files: RecordExerciseScreen, HistoryScreen
  - Task: Show skeleton loaders while fetching data
  - Task: Show saving indicator while recording exercise
  - Task: Show shimmer effect on list while loading history
  - Compilable: Yes (UI updated)

- [ ] **T091** [P] Add confirmation dialogs
  - Files: HistoryScreen (for delete operations in future), RecordExerciseScreen
  - Task: Show "Confirm Save?" dialog before recording exercise
  - Task: Show "Clear Form?" confirmation on deliberate clear
  - Compilable: Yes (dialogs added)

### End-to-End Testing

- [ ] **T092** Write e2e test: Record Exercise → View History flow
  - File: `android/app/src/androidTest/kotlin/com/simplelifting/e2e/RecordAndViewE2ETest.kt`
  - Task: Launch app → Record "Benchpress 185 lbs 8 reps 4 sets"
  - Task: Navigate to history → Verify exercise visible in list
  - Task: Verify data persisted (close app, reopen, data still there)
  - Task: Verify all 4 sets visible as separate items
  - Setup: Use in-memory or file-backed test database
  - Compilable: Yes (test class)

- [ ] **T093** Write e2e test: Multiple exercises, multiple dates
  - File: `android/app/src/androidTest/kotlin/com/simplelifting/e2e/MultiExerciseE2ETest.kt`
  - Task: Record 3 different exercises on different dates
  - Task: Navigate to history → Verify all 3 visible in correct order (most recent first)
  - Task: Filter by exercise name → Verify filter works
  - Task: Verify personal records calculated correctly
  - Compilable: Yes (test class)

### Documentation & Readability

- [ ] **T094** Add KDoc/Javadoc to all public classes and methods
  - Files: All domain models, ViewModels, use cases, repositories
  - Task: Add @param, @return, @throws documentation
  - Task: Clarify: Notes stored at per-exercise-per-session level (ExerciseSessionEntry), not per-set
  - Task: Example: RecordExerciseUseCase.execute("Benchpress", 185.0, "lbs", 8, 4, notes="felt strong") → creates 4 ExerciseSet records + 1 ExerciseSessionEntry with notes
  - Compilable: Yes (documentation added)

- [ ] **T095** Create README for development setup and running tests
  - File: `android/README.md` (if not exists) or update existing
  - Task: Document: How to open in Android Studio, run gradle builds, run tests, install on device
  - Task: List all gradle tasks available (./gradlew tasks)
  - Task: Troubleshooting section
  - Compilable: N/A (documentation)

### Final Integration Check

- [ ] **T096** Run full gradle build with all tests
  - File: Build command: `./gradlew build`
  - Task: Verify: All unit tests pass
  - Task: Verify: All instrumented tests pass
  - Task: Verify: APK generated successfully
  - Compilable: Yes (validated)

- [ ] **T097** Deploy to physical device and run manual smoke test
  - Task: Install APK on Android phone (API 24+)
  - Task: Launch app → Record exercise → View history → Verify both working
  - Task: Close app, reopen → Verify data persisted
  - Task: Verify no crashes or ANRs (application not responding)
  - Task: Verify performance acceptable (no lag)
  - Compilable: N/A (manual)

- [ ] **T098** Final Constitution Check (Before Release)
  - Task: Verify Library-First: Business logic testable without UI? ✅
  - Task: Verify Test-First: Unit + integration tests comprehensive? ✅
  - Task: Verify Data Integrity: Weight units stored immutably? ✅ (per T087-T088)
  - Task: Verify Data Integrity: Notes preserved per-exercise-per-session? ✅ (via ExerciseSessionEntry)
  - Task: Verify Learning: Documented Android patterns + Jetpack Compose decisions? ✅
  - Task: Verify Evolving: Latest versions used per refresh policy? ✅
  - Task: Verify MVP Scope: Dashboard deferred? ✅ (RecordExerciseScreen home only)
  - Compilable: N/A (design review)

---

## Phasing Summary & Parallelization Guide

### Parallelization Breakdown

**Phase 2 (Data Layer)**: 
- T002-T005 are fully parallelizable (entity definitions)
- T006-T009 are fully parallelizable (DAO interfaces)
- T016-T019 are fully parallelizable (mappers)

**Phase 3 (Repositories & Use Cases)**:
- T020-T023 are fully parallelizable (repository interfaces)
- T024-T027 are fully parallelizable (repository impls, no interdepencies until logic)
- T028-T030 are fully parallelizable (use case interfaces)

**Phase 4 (US1 - Record Exercise)**:
- T033-T038 are fully parallelizable (UI components, work on separate files)
- T042-T046 are fully parallelizable (implement different repos)
- T047-T050 are parallelizable in pairs (different DAOs)
- T078-T079 are parallelizable (input validation in different layers)
- T082-T083 are fully parallelizable (orthogonal coroutine updates)

**Phase 5 (US2 - View History)**:
- T060-T062 are fully parallelizable (UI components)
- T066-T068 are fully parallelizable (repo methods, different files)
- T069-T071 are fully parallelizable (DAO methods, different files)
- T084-T086 are fully parallelizable (edge case handling)

**Recommended Execution**:
1. Do Phases 2-3 sequentially (dependencies: entities → DAOs → mappers → repos → use cases)
2. Parallelizable groups within each phase can be assigned to multiple developers or done in parallel by one dev
3. Phase 4 mostly parallelizable except for T039 (depends on all components) and T041 (depends on repos)
4. Phase 5 similar to Phase 4

### Critical Path (Sequential Dependencies)

```
T001 (TypeConverters)
  ↓
T002-T005 (Entities) [P]
  ↓
T006-T009 (DAOs) [P]
  ↓
T010 (AppDatabase - depends on entities + DAOs)
  ↓
T011 (DAO Queries)
  ↓
T012-T015 (Domain Models) [P]
  ↓
T016-T019 (Mappers) [P]
  ↓
T020-T023 (Repo Interfaces) [P]
  ↓
T024-T027 (Repo Impls) [P]
  ↓
T028-T030 (Use Cases) [P]
  ↓
T031 (US1 Domain Model)
  ↓
T032 (US1 ViewModel) → [depends on T028 + repo impls]
  ↓
T033-T038 (US1 Components) [P] → [depends on nothing, can start earlier once ViewModel stubbed]
  ↓
T039 (US1 Screen) → [depends on T033-T038 + T032]
  ↓
T040 (MainActivity) → [depends on T039 + DB setup]
  ↓
T041 (Use Case Impl) → [depends on T024-T027 repo impls]
  ↓
T042-T046 (Repo Impl for US1) [P] → [sub-tasks of T024-T027]
  ↓
T047-T050 (DAO Impl for US1) [P] → [sub-tasks of T047]
  ↓
T052 (US1 Tests) → [depends on T041]
  ↓
T053-T056 (US1 Manual Tests)
  ↓
== US1 COMPLETE (Record Exercise) ==
  ↓
T057-T058 (US2 Domain Models)
  ↓
T059 (US2 ViewModel) → [depends on T065 use case]
  ↓
T060-T062 (US2 Components) [P]
  ↓
T063 (US2 Screen) → [depends on T060-T062, T059]
  ↓
T064 (Navigation) → [depends on T063, T040 MainActivity]
  ↓
T065 (Use Case Impl) → [depends on repo impls]
  ↓
T066-T068 (Repo Impl for US2) [P]
  ↓
T069-T071 (DAO Impl for US2) [P]
  ↓
T072 (US2 Tests)
  ↓
T073-T077 (US2 Manual Tests)
  ↓
== US2 COMPLETE (View History) ==
  ↓
T078-T091 (Polish & edge cases) [P]
  ↓
T092-T097 (Final testing & deployment)
```

---

## Implementation Strategy: MVP Thin-Slice Approach

### Why This Task Breakdown Works

1. **Compilable After Every Task**: Each task leaves the code in a compilable state. No half-finished types or import errors.
   - Example: After T002 (Exercise entity), even though Repository impl (T024) is stubbed, the entity exists as a usable type.

2. **Incremental Value Delivery**: 
   - After Phase 2+3 (29 tasks): Data layer ready, no UI yet, but testable.
   - After Phase 4 (52 tasks): Users can record exercises—first functional MVP.
   - After Phase 5 (77 tasks): Users can record AND view—full US1+US2 MVP complete.

3. **Parallelization**: Developers can work on multiple branches or the same branch in parallel on independent entities/components, then merge.

4. **Testing Throughout**: Tests added incrementally (T052, T072, T092-T093), not as a final phase. This ensures quality during development, not firefighting at the end.

5. **Validation Preserved**: Constitution principles checked throughout (Constitution VIII verified in T087-T088, Constitution III via tests in T052-T072, etc.).

### Why Not Cloud Sync / Statistics in MVP

- **US3 (Statistics + Graphs)** deferred to P2 because:
  - Requires P1 features (US1 + US2) as foundation
  - Adds data aggregation complexity (graph rendering, performance)
  - Not essential for MVP "record + view" capability
  - Can be added after MVP is stable

### Estimated Timeline (1 Developer, Part-Time)

| Phase | Tasks | Estimated Hours | Notes |
|-------|-------|-----------------|-------|
| Phase 2 | T001-T011 + T005b/T009b | 7-9 hrs | Data layer setup, entities (including ExerciseSessionEntry), DAOs, type converters |
| Phase 3 | T012-T030 | 8-10 hrs | Models, mappers, repositories, use cases |
| Phase 4 | T031-T056 | 12-15 hrs | RecordExerciseScreen, ViewModel, tests, manual testing |
| Phase 5 | T057-T077 | 10-12 hrs | HistoryScreen, filtering, tests, manual testing |
| Phase 6 | T078-T098 | 6-8 hrs | Polish, edge cases, final testing, deployment |
| **Total** | **100 tasks** | **43-54 hrs** | ~1 week full-time, 2-3 weeks part-time |

### Success Criteria (End of Phase 6)

✅ App compiles and builds with `./gradlew build`  
✅ App installs on Android device (API 24+)  
✅ User can record exercise (name, weight, reps, sets) and save to database  
✅ User can view chronological history of all recorded exercises  
✅ Data persists across app restarts  
✅ All 4 US1 acceptance scenarios pass  
✅ All 5 US2 acceptance scenarios pass  
✅ Input validation prevents invalid data (negative weight, etc.)  
✅ Performance acceptable: exercise record in <30s, history load <2s  
✅ No runtime crashes (0 ANRs, 0 null pointer exceptions)  
✅ Tests pass: unit + integration + e2e  

---

## Appendix: Tool & File Structure Reference

### File Naming Conventions

- **Entities**: `android/app/src/main/kotlin/com/simplelifting/data/entity/[EntityName]Entity.kt`
- **DAOs**: `android/app/src/main/kotlin/com/simplelifting/data/db/dao/[EntityName]Dao.kt`
- **Repositories (interface)**: `android/app/src/main/kotlin/com/simplelifting/domain/repository/[EntityName]Repository.kt`
- **Repositories (impl)**: `android/app/src/main/kotlin/com/simplelifting/data/repository/[EntityName]RepositoryImpl.kt`
- **Domain Models**: `android/app/src/main/kotlin/com/simplelifting/domain/model/[ModelName].kt`
- **Use Cases**: `android/app/src/main/kotlin/com/simplelifting/domain/usecase/[ActionName]UseCase.kt`
- **ViewModels**: `android/app/src/main/kotlin/com/simplelifting/viewmodel/[ScreenName]ViewModel.kt`
- **UI Screens**: `android/app/src/main/kotlin/com/simplelifting/ui/screens/[ScreenName]Screen.kt`
- **UI Components**: `android/app/src/main/kotlin/com/simplelifting/ui/components/[ComponentName].kt`
- **Type Converters**: `android/app/src/main/kotlin/com/simplelifting/data/db/converters/[ConverterName].kt`
- **Mappers**: `android/app/src/main/kotlin/com/simplelifting/data/mapper/[EntityName]Mapper.kt`
- **Tests (unit)**: `android/app/src/test/kotlin/com/simplelifting/[package]/[ClassName]Test.kt`
- **Tests (instrumented)**: `android/app/src/androidTest/kotlin/com/simplelifting/[package]/[ClassName]Test.kt`

### Gradle Build Commands

```bash
# Build the app
./gradlew build

# Run all tests
./gradlew test              # Unit tests (JVM)
./gradlew connectedAndroidTest  # Instrumented tests (device/emulator required)

# Run app on connected device
./gradlew installDebug        # Install debug APK
adb shell am start -n com.simplelifting/.MainActivity  # Launch app

# Clean build
./gradlew clean build

# View gradle tasks
./gradlew tasks
```

### Android Studio Keyboard Shortcuts

- **Format code**: Cmd+Option+L (Mac) or Ctrl+Alt+L (Windows/Linux)
- **Find usages**: Cmd+Option+F7 (Mac)
- **Rename**: Shift+F6
- **Go to class**: Cmd+O (Mac)
- **Go to implementation**: Cmd+B (Mac)
- **Run tests**: Ctrl+R (Mac) or Ctrl+Shift+F10 (Windows)

---

## Glossary

- **MVVM**: Model-View-ViewModel architecture pattern
- **DAO**: Data Access Object (Room pattern for database queries)
- **Room**: SQLite wrapper library by Google (ORM-like)
- **Compose**: Jetpack Compose—declarative UI framework for Android
- **Coroutines**: Kotlin async/await-style concurrency (used with suspend functions)
- **LiveData**: Observable data holder in Jetpack (UI automatically updates when data changes)
- **Entity**: Database table definition (mapped to Room entities)
- **ViewModel**: Holds UI state, survives configuration changes (screen rotations)
- **Repository**: Abstraction layer between data source (DB) and business logic (use cases)
- **Use Case**: Application business logic (implements one specific user interaction)
- **Type Converter**: Room annotation for custom types (LocalDate ↔ String conversion)
- **Mapper**: Function converting between entity (DB) and domain model (business logic)
- **Constitution**: Project principles (Library-First, CLI Interface, Test-First, etc.)

---

**Notes**:
- This tasks.md is **authoritative**. If specifications change, update tasks.md first, then regenerate code as needed.
- Tasks are ordered by dependency, not by complexity.
- Every task is designed to maintain **compilable state** after completion.
- Parallelizable tasks marked [P] can be worked on simultaneously without merge conflicts (different files).
- User stories (US1, US2) explicitly marked to show MVP prioritization.
- After Phase 6, the MVP is **production-ready** for deployment to Google Play Store (future step).
