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

---

### Edge Cases

- What happens when user enters extremely large weights (e.g., 9999 lbs)?
- How does the app handle timezone changes when user travels?
- Can app handle recordings of exercises with zero reps or sets recorded (incomplete set)?
- What happens if user has insufficient storage space on their device?
- How does app behave if user records exercise without completing all fields (partial entry)?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST allow users to create a new exercise record by selecting from a list of known exercise names (populated from previous exercises + predefined common exercises) or entering a new custom name; list supports search/filter (user types to narrow list)
- **FR-001b**: System MUST allow users to specify weight (with unit selection: lbs/kg), reps completed (including 0 for failed sets), sets, and optional notes; app stores both weight value AND original unit recorded
- **FR-002**: System MUST automatically capture and store the date when an exercise is recorded (local timezone, date-only, no time component needed)
- **FR-003**: System MUST persist all exercise data locally on the device using a reliable database
- **FR-004**: System MUST display a chronologically-ordered history of all recorded exercises with filtering by exercise name (show records for specific exercise only)
- **FR-005**: System MUST validate user input (e.g., weight/reps/sets must be positive numbers) and show clear error messages for invalid data
- **FR-006**: System MUST support editing previously recorded exercises (update weight, reps, sets, or notes)
- **FR-007**: System MUST support deleting individual exercise sets (one set at a time) with user confirmation; user selects specific set and confirms deletion
- **FR-008**: System MUST calculate and display all-time personal record (lifetime maximum weight lifted) for each exercise type
- **FR-009**: System MUST allow users to set weight unit preference (lbs vs kg); preference affects NEW entries and display conversions, but does NOT retroactively modify stored historical data units
- **FR-010**: System MUST display a dashboard/home screen showing today's session summary: all exercises recorded today with their weight, reps, sets, and exercise names
- **FR-011**: System MUST work offline with no internet connection required
- **FR-012**: System MUST display basic statistics: total workouts recorded, most frequently performed exercise, personal records per exercise
- **FR-013**: System MUST display line graphs showing weight progression for selected exercises with 4 time range options: last 30 days, last 3 months, last year, all-time; all historical data normalized to user's preferred weight unit for graph display
- **FR-014**: System MUST allow users to select/deselect multiple exercises (via checkboxes) to display on a single graph for comparison
- **FR-015**: System MUST aggregate daily maximum weight lifted (one data point per calendar day, showing highest weight across all sessions that day)
- **FR-016**: System MUST show message "Add more data for a fuller picture" for exercises with fewer than 3 historical records, but still display the minimal graph
- **FR-017**: System MUST allow exporting graph data as CSV format (selectable exercises, date range, weight values)
- **FR-018**: System MUST display an empty state on first launch: blank app with prominent "Record Exercise" button and empty history view
- **FR-019**: System MUST allow users to add notes at the exercise-per-day level (e.g., "Benchpress on March 3: first set clean, second set sloppy")
- **FR-020**: System MUST export CSV data at per-exercise-instance granularity: one row per set with columns (date, exercise name, weight, original weight unit recorded, reps, set number, notes)

### Key Entities *(include if feature involves data)*

- **ExerciseSession**: Represents a single completed exercise event
  - Attributes: id (unique), sessionDate (date in local timezone, no time component), notes (optional)
  - Relationships: Contains multiple ExerciseSet records

- **Exercise**: Represents a distinct type of exercise
  - Attributes: id, name (e.g., "Benchpress", "Squats"), description (optional), personalRecord (weight), lastPerformed (date)
  - Relationships: One-to-many with ExerciseSet

- **ExerciseSet**: Represents one set of an exercise performed
  - Attributes: id, exerciseId, sessionId, weight (numeric), weightUnit (lbs/kg), repsCompleted (can be 0 for failed sets), notes (optional), order (set number)
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
- **Quick-entry templates** (deferred to future work - focus on exercise name list first)

## Development Approach: Trust Domain Tooling

**CRITICAL PRINCIPLE**: Android Studio + AGP (Android Gradle Plugin) are domain-specific tools designed to handle gradle bootstrap and dependency management automatically. 

- **Open Android Studio first**, not as a last resort
- **Don't manually configure gradle internals** (downloading distributions, extracting wrapper jars, hardcoding versions)
- **When versions conflict, the IDE tells you.** Read the error message and make the suggested fix
- **Gradle bootstrap happens automatically.** You don't need to understand wrapper generation or maven central URLs
- **Domain-specific tooling has hard-earned best practices baked in.** Respect that design

Fighting this principle wastes hours on manual debugging that the IDE would have solved in seconds. See the Quickstart for the fastest path to a running app.

## Assumptions

- **Single User**: App assumes one user per device; no authentication or multi-user support required
- **Local Storage Only**: Exercise data stored locally on device; no cloud backup or sync
- **Weight Units**: Primary support for pounds (lbs) with optional kilogram (kg) support; original unit always stored with every weight value
- **Weight Unit Persistence**: User preference for display unit (lbs/kg) does NOT retroactively convert historical data; stored units are immutable *(per Constitution Principle VIII: Data Integrity & Format Preservation)*
- **Offline Operation**: App designed for offline-first usage; no API calls or internet connectivity required
- **Data Retention**: Exercise data retained indefinitely; no automatic deletion policies
- **Deletion Granularity**: Users delete individual sets (one at a time), not bulk exercise records
- **Exercise Search**: Exercise name selection supports search/filter functionality for quick access
- **Workout Structure**: Exercises tracked at set-level granularity (individual sets are atomic units)
- **Failed Sets**: App supports recording failed sets (0 reps) to track incomplete/unsuccessful attempts
- **Time Granularity**: Date only in local timezone; time component not tracked *(format preserved, per Constitution Principle VIII)*
- **Data Format Preservation**: All unit conversions and display transformations occur as viewing functions only; source data never modified retroactively *(per Constitution Principle VIII: Data Integrity & Format Preservation)*
- **Notes Scope**: Exercise notes are tracked at the exercise-per-day level (one note per exercise per calendar day), not per individual set
- **Personal Record**: All-time lifetime maximum weight for each exercise (never resets, not time-windowed)
- **No Undo**: Deletes are permanent (though user confirmation required)
- **Material Design**: UI follows Android Material Design guidelines
- **Minimum API Level**: Targets Android API 24+ (modern devices)
- **Storage**: Assumes sufficient device storage for typical user's lifetime exercise data (estimated <100MB for 10 years of data)

## Clarifications

### Session 2026-03-04 (First batch - Graph Requirements)

- Q: Must the statistics feature include graphs? → A: Yes, with line graphs and 4 fixed time ranges (30d, 3m, 1y, all-time)
- Q: How to handle insufficient data (< 3 records)? → A: Show minimal graphs with "Add more data for a fuller picture" message
- Q: Multi-exercise comparison? → A: Yes, unlimited exercises on one graph with checkbox selection
- Q: Graph data aggregation? → A: One point per day = highest weight lifted that day
- Q: Export capability? → A: CSV export for external use

### Session 2026-03-04 (Second batch - Additional Clarifications)

- Q: Should we implement quick-entry templates (US4)? → A: Postpone to future work. Focus on exercise name list instead.
- Q: What should dashboard summary display? → A: All exercises recorded today with their weight, reps, and sets (comprehensive view, not just count)
- Q: How granular should time tracking be? → A: Date only in local timezone; time component not needed
- Q: Should app support recording failed sets (0 reps)? → A: Yes, allow 0 reps to track incomplete/failed attempts
- Q: Exercise name entry method? → A: Provide list of known exercise names (from history + predefined list) with option to add custom name

### Session 2026-03-04 (Third batch - UX & Data Export Clarifications)

- Q: Should personal record be all-time max or time-windowed? → A: All-time maximum (global, never resets based on time filter)
- Q: What happens on first app launch with no data? → A: Display completely empty state: blank app with "Record Exercise" button and empty history
- Q: What granularity for CSV export? → A: Per-exercise-instance detail (one row per set with date, exercise, weight, unit, reps, set number, notes)
- Q: Where should exercise notes be tracked? → A: At exercise-per-day level (e.g., "Benchpress on March 3: set 1 clean, set 2 sloppy"), not per individual set and not session-wide
- Q: What filtering/sorting in History view? → A: Filter by exercise name (show only specific exercise); reverse chronological sort is default

### Session 2026-03-04 (Fourth batch - Data Integrity & UX Refinements)

- Q: How should weight unit preference changes affect historical data? → A: Store original unit always; preference only affects NEW entries and display conversions (read-only)
- Q: What can users delete - sets, exercises, or types? → A: Delete individual sets only (one at a time) with confirmation; no bulk deletion
- Q: How to handle graphs with mixed units (some lbs, some kg)? → A: Normalize all data to user's preferred unit for graph display
- Q: Should exercise name selection support search? → A: Yes, enable search/filter - user can type to narrow the list

## Acceptance Criteria Summary

Feature is complete when:
1. All user stories (P1/P2 - US4 removed) have acceptance scenarios passing
2. All functional requirements (FR-001 through FR-020) are implemented
3. All success criteria metrics are met
4. Edge cases are handled gracefully with user-friendly error messages
5. App passes performance targets (30-second session recording, 2-second history load)
6. Graphs display correctly with 1+ data points and CSV export functions properly
7. Exercise name selection works with predefined + history-based list + search/filter
8. Dashboard shows complete today's summary with all exercises, weights, reps, and sets
9. Failed sets (0 reps) can be recorded and tracked without errors
10. First launch displays empty state; history filtering by exercise name works
11. CSV export contains per-set granularity rows with all required columns + original unit
12. Per-exercise-per-day notes can be added and displayed correctly
13. Personal record (all-time max) displays accurately and never resets
14. Weight unit preference affects only NEW entries and display (read-only conversions)
15. Individual set deletion works with confirmation; no bulk deletion
16. Mixed-unit graphs normalize to user's preferred unit for display
