# Implementation Plan: Weight Training Exercise Tracker

**Branch**: `001-exercise-tracker` | **Date**: 2026-03-11 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/001-exercise-tracker/spec.md`
**MVP Scope**: Thinnest deployable slice for Android phone: US1 (Record Exercise) + US2 (View History)

## Summary

Android app enabling users to quickly record weight training exercises (exercise name, weight, reps, sets) and view their session history. MVP focuses on P1 user stories: fast exercise entry and chronological history with filtering. Offline-first, local storage only. Deployable to Android devices API 24+.

## Technical Context

**Language/Version**: Kotlin 1.9.x (modern Android standard)
**Primary Dependencies**: Android Framework (API 24+), Jetpack Compose (UI), Room (database), Coroutines (async)
**Storage**: Room database (SQLite wrapper; local device only)
**Testing**: JUnit 4, Espresso (UI automation)
**Target Platform**: Android API 24-35+ (phone, no tablets in MVP)
**Project Type**: mobile-app (Android native)
**Performance Goals**: Exercise entry in <30s (SC-001), history load <2s for 1 year of data (SC-002)
**Constraints**: Offline-capable, <100MB storage footprint, no network calls
**Scale/Scope**: Single user, local storage, estimated 365 exercises/year = <100MB over 10 years

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| **I. Library-First** | ⚠️ MODIFIED | Mobile app is primary deliverable, but internal architecture must decompose into testable library components (models, storage layer, business logic) that are independently testable |
| **II. CLI Interface** | ⚠️ MODIFIED | Primary interface is Android UI (Compose), but core exercise tracking logic must expose programmatic API for testing and future CLI if needed |
| **III. Test-First** | ✅ APPLY | TDD mandatory: unit tests for data models, storage layer; UI tests for critical paths (record exercise, view history) |
| **IV. Integration Testing** | ✅ APPLY | Test database operations, data persistence across app restarts, offline functionality |
| **VIII. Data Integrity** | ✅ APPLY | Weight units stored with every entry; display conversions only (readonly); no retroactive modifications |
| **IX. Learning & Knowledge Sharing** | ✅ APPLY | Document Android development patterns, Jetpack Compose decisions, database design rationale |
| **X. Evolving with Best Practices** | ✅ APPLY | Reference latest Jetpack guide, Compose best practices, Room database patterns |

**Gate Decision**: Justified Exceptions - Mobile app responds to user's stated goal ("deployable to my phone"). Constitution principles adapted as follows:
- Library-First intent preserved via internal architecture decomposition (business logic layer)
- CLI Interface deferred; primary interface is mobile UI with testable API layer beneath
- Library-like components (models, repos, use cases) are independently testable without UI

## Technology Refresh Policy

**CRITICAL**: Android/Jetpack ecosystem evolves rapidly. Versions in this spec are authoritative only on their verification date.

| Component | Last Verified | Cadence | Next Review | Truth Source |
|-----------|---------------|---------|----|---|
| Android Gradle Plugin (AGP) | 2026-03-11 | Quarterly | 2026-06-11 | [Google Maven](https://maven.google.com/) |
| Kotlin | 2026-03-11 | Quarterly | 2026-06-11 | [Kotlin Releases](https://github.com/JetBrains/kotlin/releases) |
| Jetpack Compose | 2026-03-11 | Quarterly | 2026-06-11 | [Jetpack Compose Release Notes](https://developer.android.com/jetpack/androidx/releases/compose) |
| Room Database | 2026-03-11 | Quarterly | 2026-06-11 | [Room Release Notes](https://developer.android.com/jetpack/androidx/releases/room) |
| Material Design | 2026-03-11 | Quarterly | 2026-06-11 | [Material 3 Docs](https://m3.material.io) |

**If this spec is > 3 months old**:
1. ✅ Follow [quickstart.md](quickstart.md) Step 2, Option B: Generate fresh gradle files from Android Studio's New Project Wizard
2. ✅ Compare generated versions to versions documented here
3. ✅ If major updates found, regenerate all code samples that reference versions
4. ✅ Update this table with new verification date + versions

**Constitution X Alignment**: This refresh policy ensures compliance with "Evolving with Best Practices." Stale versions are a violation; quarterly reviews keep the project current.

## Learning Discovery Gate

*PRINCIPLE IX: Capture learning outcomes, patterns, gotchas, and domain insights. Use this section during planning and research phases to document discoveries for the team and future-self.*

### Patterns & Insights Discovered
- Android Jetpack Compose is modern pattern for UI; previous Material 2 + XML layouts pattern largely obsolete
- Room database + Kotlin coroutines + suspend functions = modern async data layer pattern
- Offline-first architecture for fitness apps is standard practice (no cloud dependency)
- Single-user local-only data model simplifies significantly vs. cloud-synced patterns
- Exercise tracking requires immutable historical record (per Constitution VIII); unit preservation critical for data integrity

### Gotchas & Challenges Identified
- Android API compatibility: targeting API 24+ includes significant device range; careful testing on older devices needed
- Database migrations: as schema evolves, Room migrations can be complex; plan schema early
- Timezone handling: storing date-only (not datetime) simplifies most use cases but beware of DST edge cases
- Graph rendering: performance risk with 1000+ data points; may need viewport clipping or aggregation

### Tool & Technology Explorations
- **Jetpack Compose** vs XML: Chosen for modern, declarative UI; lower boilerplate; better testability
- **Room** vs SQLite directly: Chosen for built-in migrations, type safety, Kotlin integration
- **Coroutines** vs RxJava: Chosen for simpler learning curve, modern Kotlin standard, built-in Jetpack support
- **Material 3** vs Material 2: Chosen for Material 3 (current standard); Material 2 pattern being phased out

### Decision Rationale
- **Kotlin + Compose UI** (vs Java + XML): Kotlin is Android standard; Compose is current recommended pattern; aligns with Constitution X (evolving with best practices)
- **Room + Coroutines** (vs direct SQL): Reduces boilerplate, provides safety, aligns with modern Jetpack patterns
- **Local-only MVP** (vs cloud sync): Aligns with "thinnest deployable slice"; cloud sync deferred to later
- **Architecture: MVVM + Repository pattern** (vs monolithic): Enables testability (Constitution III); separates concerns

### Domain Learnings (Android / Spec Kit)
- Android Material Design 3 now emphasizes adaptive layouts (responsive to screen size); MVP is phone-only but design should consider tablet-future
- Jetpack Compose learning curve is manageable; declarative paradigm different from imperative XML but faster to iterate
- Spec Kit MVP approach aligns with Android agile development (ship working slice, iterate)
- Exercise tracking domain has simple data model; complexity comes from UI polish and graph performance, not data layer

## Project Structure

### Documentation (this feature)

```text
specs/001-exercise-tracker/
├── plan.md              # This file
├── research.md          # Phase 0: Detailed research findings (if needed)
├── data-model.md        # Phase 1: Entity definitions, schema design
├── quickstart.md        # Phase 1: Local setup, build & run, first exercise entry
├── contracts/           # Phase 1: API contracts (if applicable - may be minimal for mobile app)
└── tasks.md             # Phase 2: Implementation tasks (generated by /speckit.tasks)
```

### Source Code (Android Project Root)

```text
android/
├── app/                          # Main app module
│   ├── src/main/kotlin/
│   │   ├── ui/                   # Jetpack Compose screens & components
│   │   │   ├── screens/ExerciseRecorderScreen.kt
│   │   │   ├── screens/HistoryScreen.kt
│   │   │   └── components/
│   │   ├── data/
│   │   │   ├── db/               # Room database
│   │   │   │   ├── AppDatabase.kt
│   │   │   │   └── dao/
│   │   │   ├── entity/           # Database entities (Exercise, ExerciseSet, etc)
│   │   │   └── repository/       # Repository pattern (data access layer)
│   │   ├── domain/
│   │   │   ├── model/            # Domain models (business logic entities)
│   │   │   ├── usecase/          # Use cases (business logic)
│   │   │   └── repository/       # Repository interfaces
│   │   ├── viewmodel/            # MVVM ViewModels for Compose
│   │   └── MainActivity.kt
│   ├── src/test/kotlin/          # Unit tests (models, repository, use cases)
│   └── src/androidTest/kotlin/   # Instrumented tests (UI, database)

build.gradle.kts                  # Project-wide gradle config
gradle.properties                 # Gradle properties (API level, versions)
```

**Structure Rationale**: 
- MVVM + Repository pattern enables testability (Constitution III - Test-First)
- Separation of concerns: UI (Compose), domain logic, data layer
- room/ entities separate from domain models allows schema evolution without business logic coupling

## Complexity Tracking

> **Justified Exception to Constitution II (CLI Interface)**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| Primary Interface is Compose Mobile UI (not CLI) | User's explicit goal: "deployable to my phone" | CLI-only approach wouldn't meet user's immediate need for app on device; defeats MVP goal of "thinnest deployable slice to phone" |
| Library-First adapted for mobile | Mobile apps are single deliverable, not libraries | Core business logic still decomposed into testable library-like components (models, repository, use cases); UI layer sits atop testable API surface |
