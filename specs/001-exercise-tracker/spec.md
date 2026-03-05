# Feature Specification: Weight Training Exercise Tracker

**Feature Branch**: `001-exercise-tracker`  
**Created**: 2026-03-04  
**Status**: Draft  
**Input**: User description: "Build an Android app to help me track my weight training exercises"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Record Exercise Session (Priority: P1)

User opens the app after completing a weight training session and wants to record the exercises they performed. They enter the exercise name, weight lifted, number of reps, and number of sets, then save the session.

**Why this priority**: This is the core MVP. Without the ability to record exercises, the app becomes useless. Users must be able to quickly log their workout data immediately or shortly after exercising.

**Independent Test**: Can be fully tested by launching the app, entering exercise data (e.g., "Benchpress, 185 lbs, 8 reps, 4 sets"), saving it, and confirming the data is stored. This delivers immediate value even if no other features exist.

**Acceptance Scenarios**:

1. **Given** the app is open on the exercise entry screen, **When** user enters exercise name "Benchpress", weight "185", reps "8", sets "4" and taps Save, **Then** the exercise is recorded with current date/time and persisted to storage
2. **Given** an exercise has been recorded, **When** user closes and reopens the app, **Then** the recorded exercise data still exists
3. **Given** user is entering an exercise, **When** they leave the app and return, **Then** their data is preserved (not lost)
4. **Given** user enters invalid data (e.g., negative weight or zero reps), **When** they attempt to save, **Then** app displays clear error message and prevents save

---

### User Story 2 - View Exercise History (Priority: P1)

User wants to see all exercises they've recorded in the past to track their progress over time. They can view a list of exercises sorted by date, with details about each workout session.

**Why this priority**: Same core priority as US1. The ability to view history is essential for tracking progress and motivation. Users need to see what they've accomplished.

**Independent Test**: Can be fully tested by recording multiple exercises with different dates, then displaying them in a list view. User can scroll through and see all recorded sessions ordered chronologically. This is independently valuable.

**Acceptance Scenarios**:

1. **Given** multiple exercises have been recorded over different dates, **When** user opens the History screen, **Then** all exercises are displayed in chronological order (most recent first)
2. **Given** the history screen is displayed, **When** user taps on an exercise, **Then** full details are shown (exercise name, weight, reps, sets, date, time)
3. **Given** exercises span multiple weeks, **When** user views history, **Then** exercises are grouped by week or date for easy scanning
4. **Given** the app has been running for months with thousands of exercises, **When** user scrolls through history, **Then** the app remains responsive

---

### User Story 3 - View Progress Statistics with Graphs (Priority: P2)

User wants to understand their progress over time. They can see aggregated statistics with visual graphs showing progress for specific exercises: personal records (max weight per exercise), average reps/sets, trends, and workout frequency. Graphs display weight progression and other metrics over time.

**Why this priority**: P2 because it builds on the foundation of US1 and US2. While valuable for motivation, the app remains functional without detailed statistics. This is a value-add feature.

**Independent Test**: Can be fully tested by recording multiple sessions of the same exercise at different weights over several weeks, then displaying graphs that clearly show progression. Graphs accurately reflect recorded data.

**Acceptance Scenarios**:

1. **Given** user has recorded Benchpress exercises on multiple days with varying weights, **When** user opens the Statistics screen, **Then** app displays personal record (heaviest weight) for Benchpress AND shows a graph of weight progression over time
2. **Given** statistics are displayed, **When** user filters by exercise type, **Then** stats refresh to show only that exercise's data AND the graph updates to show only that exercise's progression
3. **Given** user has one week of exercise data, **When** user views trends with graphs, **Then** app shows count of sessions, average intensity, AND a visual graph of progress

---

### User Story 4 - Quick Entry with Exercise Templates (Priority: P2)

User has favorite exercises they do frequently (e.g., Benchpress, Squats, Deadlifts). They can save these as "templates" and quickly log them by tapping a button, with auto-populated defaults for weight and reps.

**Why this priority**: P2 because it optimizes the workflow for returning users but is not essential for the MVP. Users can always manually enter exercises.

**Acceptance Scenarios**:

1. **Given** user has recorded Benchpress multiple times with typically 185 lbs and 8 reps, **When** they save Benchpress as a template, **Then** a template is created with those defaults
2. **Given** Benchpress template exists with 185 lbs/8 reps defaults, **When** user taps the template button, **Then** exercise entry form pre-populates with 185 lbs and 8 reps (overrideable)

---

### Edge Cases

- What happens when user enters extremely large weights (e.g., 9999 lbs)?
- How does the app handle timezone changes when user travels?
- Can app handle recordings of exercises with zero reps or sets recorded (incomplete set)?
- What happens if user has insufficient storage space on their device?
- How does app behave if user records exercise without completing all fields (partial entry)?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST allow users to create a new exercise record with exercise name, weight (with unit selection: lbs/kg), reps, sets, and optional notes
- **FR-002**: System MUST automatically capture and store the date and time when an exercise is recorded
- **FR-003**: System MUST persist all exercise data locally on the device using a reliable database
- **FR-004**: System MUST display a chronologically-ordered history of all recorded exercises with filtering/sorting options
- **FR-005**: System MUST validate user input (e.g., weight/reps/sets must be positive numbers) and show clear error messages for invalid data
- **FR-006**: System MUST support editing previously recorded exercises (update weight, reps, sets, or notes)
- **FR-007**: System MUST support deleting previously recorded exercises with user confirmation
- **FR-008**: System MUST calculate and display personal record (maximum weight lifted) for each exercise type
- **FR-009**: System MUST allow users to set weight and rep preferences (lbs vs kg as default unit)
- **FR-010**: System MUST display a dashboard/home screen with today's session summary and quick-access buttons for recording exercises
- **FR-011**: System MUST work offline with no internet connection required
- **FR-012**: System MUST display basic statistics: total workouts recorded, most frequently performed exercise, personal records per exercise
- **FR-013**: System MUST display line graphs showing weight progression for selected exercises with 4 time range options: last 30 days, last 3 months, last year, all-time
- **FR-014**: System MUST allow users to select/deselect multiple exercises (via checkboxes) to display on a single graph for comparison
- **FR-015**: System MUST aggregate daily maximum weight lifted (one data point per calendar day, showing highest weight across all sessions that day)
- **FR-016**: System MUST show message "Add more data for a fuller picture" for exercises with fewer than 3 historical records, but still display the minimal graph
- **FR-017**: System MUST allow exporting graph data as CSV format (selectable exercises, date range, weight values)

### Key Entities *(include if feature involves data)*

- **ExerciseSession**: Represents a single completed exercise event
  - Attributes: id (unique), sessionDate (date), sessionTime (time), totalDurationMinutes (optional), notes (optional)
  - Relationships: Contains multiple ExerciseSet records

- **Exercise**: Represents a distinct type of exercise
  - Attributes: id, name (e.g., "Benchpress", "Squats"), description (optional), personalRecord (weight), lastPerformed (date)
  - Relationships: One-to-many with ExerciseSet

- **ExerciseSet**: Represents one set of an exercise performed
  - Attributes: id, exerciseId, sessionId, weight (numeric), weightUnit (lbs/kg), repsRequired (planned), repsCompleted (actual), notes (optional), order (set number)
  - Relationships: Belongs to ExerciseSession and Exercise

- **UserPreference**: Stores user settings
  - Attributes: preferredWeightUnit (lbs/kg), theme (optional), other UI preferences (optional)

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can record a complete exercise session (name, weight, reps, sets) in under 30 seconds from app launch
- **SC-002**: App displays exercise history from the past 12 months without lag (loads within 2 seconds)
- **SC-003**: 100% of recorded exercise data is persisted across app restarts and device reboots
- **SC-004**: 99% of exercise recording actions complete successfully without app crashes
- **SC-005**: Personal record calculations are 100% accurate (correctly identifies max weight per exercise)
- **SC-006**: App functions offline with zero connectivity issues or error messages
- **SC-007**: User can view and search exercise records for any date up to one year in the past
- **SC-008**: Dashboard displays today's session summary within 1 second of app launch

## Scope Boundaries

### In Scope
- Local exercise tracking and history
- Basic statistics and personal records
- Data persistence on device
- Single-user local app

### Out of Scope
- Cloud sync or multi-device sync
- Social features (sharing, comparing with others)
- Video form analysis or AI coaching
- Integration with wearables or fitness trackers
- Detailed workout plan generation
- Advanced nutrition tracking

## Assumptions

- **Single User**: App assumes one user per device; no authentication or multi-user support required
- **Local Storage Only**: Exercise data stored locally on device; no cloud backup or sync
- **Weight Units**: Primary support for pounds (lbs) with optional kilogram (kg) support
- **Offline Operation**: App designed for offline-first usage; no API calls or internet connectivity required
- **Data Retention**: Exercise data retained indefinitely; no automatic deletion policies
- **Workout Structure**: Exercises tracked at set-level granularity (individual sets are atomic units)
- **No Undo**: Deletes are permanent (though user confirmation required)
- **Material Design**: UI follows Android Material Design guidelines
- **Minimum API Level**: Targets Android API 24+ (modern devices)
- **Storage**: Assumes sufficient device storage for typical user's lifetime exercise data (estimated <100MB for 10 years of data)

## Clarifications

### Session 2026-03-04

- Q: Must the statistics feature include graphs to show progress over time? → A: Yes, graphs must visualize weight progression over time for specific exercises
- Q: What graph types and time ranges should be supported? → A: Line graphs only with 4 fixed time ranges (last 30 days, last 3 months, last year, all-time)
- Q: How should the app handle exercises with insufficient data (< 3 records)? → A: Show graph even with 1-2 data points with message: "Add more data for a fuller picture"
- Q: Can users compare multiple exercises on one graph? → A: Yes, allow unlimited exercises on one graph with user-selectable checkboxes (user can enable/disable each exercise)
- Q: What does each graph data point represent? → A: One point per day showing the heaviest weight lifted that day for selected exercise(s), aggregated across all sessions
- Q: Should graphs be exportable? → A: Yes, allow exporting graph data as CSV format for use in external spreadsheets

## Acceptance Criteria Summary

Feature is complete when:
1. All user stories (P1/P2) have acceptance scenarios passing
2. All functional requirements (FR-001 through FR-017) are implemented
3. All success criteria metrics are met
4. Edge cases are handled gracefully with user-friendly error messages
5. App passes performance targets (30-second session recording, 2-second history load)
6. Graphs display correctly with 1+ data points and CSV export functions properly
