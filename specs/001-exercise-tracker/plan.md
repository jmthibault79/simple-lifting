# Implementation Plan: Weight Training Exercise Tracker

**Branch**: `001-exercise-tracker` | **Date**: 2026-03-04 | **Spec**: [spec.md](spec.md)  
**Input**: Feature specification from `/specs/001-exercise-tracker/spec.md`  
**Research**: [research.md](research.md) - Android tech stack decisions

## Summary

Build an offline-first Android app for tracking weight training exercises. Core MVP allows users to:
1. Record exercises with weight, reps, sets (P1)
2. View exercise history with filtering (P1)
3. View progress graphs with statistics (P2)

**Technical Approach**: Native Android development using Jetpack Compose (modern UI), Room database (local persistence), MVVM architecture (clean code structure), LiveData (reactive state management). Targets Android 7.0+ with focus on learning best practices while maintaining production-quality code.

## Technical Context

**Language/Version**: Kotlin with Android Framework (target Android API 24+, minSdk 21, compileSdk latest)  
**Primary Dependencies**: 
- Jetpack Compose 1.5+ (UI framework)
- Room 2.5+ (local database)
- Jetpack ViewModel/LiveData (state management)
- Jetpack Navigation (multi-screen support)
- MPAndroidChart (progress graphs)

**Storage**: Room Persistence Library (SQLite wrapper); local device storage only, no cloud sync  
**Testing**: JUnit 4 (unit tests), Android Instrumented Tests (Compose/Room integration), manual feature testing  
**Target Platform**: Android mobile (phone & tablet); API level 24+ (Android 7.0 and above)  
**Project Type**: Mobile app (native Android)  
**Performance Goals**: 
- Session recording: <30 seconds from app launch
- History load: <2 seconds for last 12 months
- Graph rendering: <1 second
- Offline operation: 100% (no internet required)

**Constraints**: 
- Single-user (no multi-user support)
- Offline-only (no API calls)
- Original unit preservation (per Constitution Principle VIII: conversions display-only)
- Data immutability (historical data never retroactively modified)

**Scale/Scope**:
- 1-2 users per device
- Estimated <100MB storage for 10 years of exercise data
- ~10-15 screens (dashboard, record, history, stats, settings)
- Estimated 5,000-8,000 lines of code for MVP

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### Principle Alignment

| Principle | Requirement | Status |
|-----------|-------------|---------|
| **I. Library-First** | Features start as standalone libraries | 🟡 PARTIAL - This is a mobile app frontend, not a library. Per spec scope: "Single-user local app". Library extraction deferred to future work if core logic needs to be extracted. Mitigation: Implement domain logic (stat calculations, data conversions) as testable service layer that COULD be extracted later. |
| **II. CLI Interface** | Library exposes CLI | 🟢 N/A - Mobile app, not library. Text I/O principle applies to CSV export (data format preservation). ✓ Implemented in FR-020. |
| **III. Test-First (NON-NEGOTIABLE)** | TDD mandatory | 🟢 REQUIRED - Will implement with test-first approach for ViewModel and Repository layers. Compose UI tests require device/emulator (instrumented tests). Red-Green-Refactor enforced in phase 2 task planning. |
| **IV. Integration Testing** | Focus on contract/schema tests | 🟢 REQUIRED - Room DAO tests verify database contracts. Compose integration tests verify UI→ViewModel communication. CSV export tests verify data format contract. |
| **V. Observability** | Structured logging | 🟢 IMPLEMENT - Add Android logging (Timber library recommended for production). Store debug logs in app-accessible files for user inspection. |
| **VI. Versioning** | MAJOR.MINOR.BUILD | 🟢 N/A - App version follows Android convention: versionCode (numeric), versionName (user-visible). Convention: v1.0.0 for initial release. |
| **VII. Simplicity** | Start simple, YAGNI | 🟢 ENFORCE - MVP targets core 3 user stories (P1/P2); templates (US4) deferred; no cloud, no social, no AI. Complexity added incrementally with user feedback. |
| **VIII. Data Integrity & Format Preservation** | Original format preserved; conversions display-only | 🟢 REQUIRED - Core design principle: weight units stored immutably (lbs/kg recorded), displayed/exported in original format. Conversions happen in ViewModel only. ✓ Per FR-001b, FR-009, FR-013, FR-020. |

**Constitution Gate Result**: ✅ **PASS** - All principles either fully aligned or explicitly scoped out with mitigation.

**Principle VIII (Data Integrity) is CRITICAL to this app's design**. See assumptions in spec.md.

---

## Project Structure

### Documentation (this feature)

```
specs/001-exercise-tracker/
├── spec.md              ✓ Complete specification with 20 requirements
├── plan.md              ← This file
├── research.md          ✓ Tech stack decisions (Kotlin, Jetpack Compose, Room, MVVM)
├── data-model.md        → Phase 1 output (entities, DAOs, relationships)
├── contracts/           → Phase 1 output (Room contract tests, UI contract spec)
├── quickstart.md        → Phase 1 output (local dev setup guide)
├── checklists/          ✓ Quality checklist
└── tasks.md             → Phase 2 output (actionable implementation tasks)
```

### Source Code (Android project)

```
android/                                          # Android app root
├── app/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/example/simplelifting/
│   │   │   │   ├── ui/
│   │   │   │   │   ├── screens/                  # Jetpack Compose screens
│   │   │   │   │   │   ├── RecordExerciseScreen.kt
│   │   │   │   │   │   ├── DashboardScreen.kt
│   │   │   │   │   │   ├── HistoryScreen.kt
│   │   │   │   │   │   ├── StatsScreen.kt
│   │   │   │   │   │   └── SettingsScreen.kt
│   │   │   │   │   ├── components/               # Reusable Composables
│   │   │   │   │   │   ├── ExerciseForm.kt
│   │   │   │   │   │   ├── ExerciseCard.kt
│   │   │   │   │   │   └── ProgressGraph.kt
│   │   │   │   │   └── theme/                    # Material Design 3
│   │   │   │   │       ├── Color.kt
│   │   │   │   │       └── Theme.kt
│   │   │   │   ├── viewmodel/                    # MVVM ViewModel layer
│   │   │   │   │   ├── ExerciseViewModel.kt
│   │   │   │   │   ├── StatsViewModel.kt
│   │   │   │   │   └── DashboardViewModel.kt
│   │   │   │   ├── model/                        # Data models (entities)
│   │   │   │   │   ├── Exercise.kt
│   │   │   │   │   ├── ExerciseSession.kt
│   │   │   │   │   ├── ExerciseSet.kt
│   │   │   │   │   └── UserPreference.kt
│   │   │   │   ├── data/                         # Data layer
│   │   │   │   │   ├── database/
│   │   │   │   │   │   ├── AppDatabase.kt        # Room database config
│   │   │   │   │   │   ├── ExerciseDao.kt        # Data access object
│   │   │   │   │   │   └── SessionDao.kt
│   │   │   │   │   └── repository/
│   │   │   │   │       ├── ExerciseRepository.kt # Repository pattern
│   │   │   │   │       └── StatsRepository.kt
│   │   │   │   ├── util/                         # Utilities
│   │   │   │   │   ├── DateUtils.kt
│   │   │   │   │   ├── UnitConverter.kt          # Unit conversion logic
│   │   │   │   │   │   └── (display-only, per Principle VIII)
│   │   │   │   │   └── CsvExporter.kt            # Export functionality
│   │   │   │   ├── MainActivity.kt               # Entry point
│   │   │   │   └── Constants.kt
│   │   │   └── AndroidManifest.xml
│   │   ├── test/                                 # Unit tests (JVM)
│   │   │   └── java/com/example/simplelifting/
│   │   │       ├── viewmodel/
│   │   │       │   ├── ExerciseViewModelTest.kt
│   │   │       │   └── StatsViewModelTest.kt
│   │   │       ├── data/
│   │   │       │   ├── ExerciseRepositoryTest.kt
│   │   │       │   └── UnitConverterTest.kt
│   │   │       └── util/
│   │   │           └── CsvExporterTest.kt
│   │   └── androidTest/                          # Instrumented tests (device/emulator)
│   │       └── java/com/example/simplelifting/
│   │           ├── database/
│   │           │   ├── RoomDatabaseTest.kt
│   │           │   └── ExerciseDaoTest.kt
│   │           └── ui/
│   │               └── RecordExerciseScreenTest.kt
│   └── build.gradle
├── build.gradle                                  # Root build config
├── gradle.properties
└── settings.gradle
```

**Structure Decision**: Single-module Android app (Option 3 variant). Organized by **feature/layer** (UI screens, ViewModel, Model, Data) for clarity and testability. No backend API (offline-first). Test structure mirrors source structure for easy test discovery.

---

## Development Environment Setup

**Prerequisites**:
- macOS (already confirmed for this project)
- 50GB free disk space (Android Studio + SDK + emulator)
- Kotlin 1.9+ (via Android Gradle plugin)

**Installation Steps**:
1. Download Android Studio from https://developer.android.com/studio
2. Install Android SDK (automatic in Android Studio)
3. Create/configure Android Virtual Device (emulator)
4. Create `/android` directory in project root
5. Initialize Gradle project and add dependencies per research.md

---

## Complexity Tracking

**Constitution Principle VIII (Data Integrity) Implementation Details**:

| Aspect | Why Needed | Simpler Alternative Rejected |
|--------|-----------|------------------------------|
| Store original unit with every weight record | Immutable data format (principle VIII); enables correct CSV export and audit trail | Store only single preferred unit → Loss of original data, breaks Constitutional principle |
| Display-only unit conversion in ViewModel | Separation of concerns; prevents accidental source data modification | Convert data on DB read → Violates data integrity principle; causes inconsistencies |
| Explicit `weightUnit` field in Room entity | Type safety for database; tracks original recording unit | Store as string comment → No schema enforcement, data corruption risk |

---

## Phase 0: Research - Status ✅ COMPLETE

Research phase output: [research.md](research.md)

**Technologies Researched & Decided**:
1. ✅ Language: Kotlin
2. ✅ UI Framework: Jetpack Compose
3. ✅ Database: Room Persistence Library
4. ✅ Architecture: MVVM + LiveData
5. ✅ Dependency Management: Manual (lightweight approach)
6. ✅ Testing: JUnit + Instrumented tests
7. ✅ API Level: 24+ (Android 7.0+)

**All NEEDS CLARIFICATION resolved**: No ambiguities remain.

**Next**: Proceed to Phase 1 (Design & Contracts)

---

## Next Phase: Phase 1 Design & Contracts (Will Generate)

Will create:
1. **data-model.md**: Entity definitions, DAO contracts, database schema
2. **contracts/**: Room DAO test contracts, UI Compose contracts, CSV export format spec
3. **quickstart.md**: Local development setup + first "hello world" guide

**Timeline**: Phase 1 expected to complete design within one session.

After Phase 1 → Phase 2 (Tasks) via `/speckit.tasks` command

---

## Next Steps

1. ✅ Phase 0 Research complete (this file + research.md)
2. → **Phase 1**: Generate data-model.md, contracts/, quickstart.md
3. → **Phase 1**: Update agent context (Copilot) with Android tech stack knowledge
4. → **Phase 2**: Generate tasks.md with actionable development tasks (TDD-first)
