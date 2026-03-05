# Research: Android Tech Stack for Exercise Tracker

**Feature**: Weight Training Exercise Tracker (001-exercise-tracker)  
**Date**: 2026-03-04  
**Goal**: Determine Android development stack for learning + best practices

## Technology Decisions

### 1. Programming Language: Kotlin

**Decision**: Use Kotlin for Android development

**Rationale**:
- Google's officially recommended language for Android development (since 2019)
- Modern, concise syntax—reduces boilerplate and verbosity 
- Excellent interoperability with Java ecosystem
- Perfect match for Jetpack Compose (Compose is optimized for Kotlin)
- Superior null safety and functional programming features
- Growing Android community and modern documentation
- Easier learning curve for modern language features compared to Java

**Alternatives Considered**:
- **Java**: Stable, mature ecosystem, extensive documentation. Trade-off: verbose syntax, not optimized for Compose, legacy patterns. Historical choice deferred in favor of modern best practices.
- **Flutter/Dart**: Cross-platform (iOS + Android). Trade-off: different ecosystem, not native Android development

**Learning Path**: Start with Kotlin fundamentals alongside Android basics; learn idiomatic Kotlin patterns (extension functions, coroutines, data classes) as part of core development

---

### 2. UI Framework: Jetpack Compose

**Decision**: Use Jetpack Compose for user interface

**Rationale**:
- Modern declarative UI approach (similar to React, Flutter)
- Google's current officially recommended UI framework
- Significantly reduces boilerplate compared to XML layouts
- Better learning curve conceptually (describe what UI should look like, let framework handle updates)
- Full Material Design 3 support (modern Android design system)
- Integrates seamlessly with Jetpack libraries
- Smaller file sizes and better performance than legacy XML

**Alternatives Considered**:
- **XML Layouts + View System**: Traditional Android approach. Trade-off: verbose, harder to maintain, legacy (though massive tutorial base available). Good fallback if Compose issues arise.
- **React Native**: Cross-platform. Trade-off: not native Android, different learning path

**Compose Version**: Jetpack Compose 1.5+ (stable, production-ready)

**Learning Resources**:
- Official Google: https://developer.android.com/compose
- Codelab (hands-on): https://developer.android.com/courses/compose/course
- Jetpack Compose state management is critical learning point

---

### 3. Local Data Storage: Room Persistence Library

**Decision**: Use Room (SQLite wrapper) for exercise data persistence

**Rationale**:
- Google's official recommended database library for Android
- Built on SQLite (proven, lightweight, reliable)
- Type-safe database access with compile-time SQL checking
- Excellent integration with LiveData for reactive updates
- Built into Jetpack ecosystem
- Handles migrations automatically
- Perfect for offline-first apps (no network needed)

**Alternatives Considered**:
- **SQLiteOpenHelper**: Raw SQLite API. Trade-off: much more verbose, manual cursor management, error-prone
- **Realm**: Easier query API than Room. Trade-off: separate ecosystem, less Jetpack integration, overkill for this app's complexity

**Room Architecture**:
- **Entity Classes**: Represent tables (Exercise, ExerciseSession, ExerciseSet, UserPreference)
- **DAO (Data Access Object)**: Define query methods (insert, update, delete, query)
- **Database Class**: Manages entity tables and database access

---

### 4. Application Architecture: MVVM + LiveData

**Decision**: Use Model-View-ViewModel (MVVM) architecture with LiveData for state management

**Rationale**:
- Google's officially recommended architecture pattern for Android
- Clear separation of concerns:
  - **View**: Composable UI functions (what user sees)
  - **ViewModel**: Holds UI state and business logic (survives configuration changes)
  - **Model**: Data layer (entities, repositories, database)
- LiveData provides reactive state updates (UI automatically reflects data changes)
- ViewModel survives device rotations (no data loss on screen rotate)
- Perfect for Jetpack Compose integration
- Testable: business logic separated from UI

**Pattern Explanation**:
```
UI Event (user clicks button)
    ↓
ViewModel processes event (calls repository)
    ↓
Repository queries Room database
    ↓
LiveData notifies Compose UI
    ↓
Compose re-renders with new state
```

---

### 5. Dependency Management: Manual (Lightweight Approach)

**Decision**: Use manual dependency injection for initial MVP phase

**Rationale**:
- Simpler learning path (understand dependencies before adding DI framework)
- Sufficient for app of this scope (single-user, one feature)
- Reduces initial framework overhead
- Easier debugging (direct dependency chain visible)

**Migration Path**: Once this feature is complete, evaluate Hilt (Google's DI framework) for future multi-feature apps

**Pattern**: Pass dependencies through constructors (e.g., passing Repository to ViewModel)

---

### 6. Testing Framework: JUnit + Instrumented Tests

**Decision**: Use JUnit for unit tests + Android Instrumented Tests for UI

**Rationale**:
- JUnit: Industry standard for Kotlin/Java unit testing
- Instrumented tests run on actual Android device/emulator (test real Compose behavior)
- Available in Android Studio by default
- Good learning foundation before moving to advanced mocking frameworks

**Testing Layers**:
- **Unit Tests** (JVM): Test ViewModels, repositories (fast, no device needed)
- **Instrumented Tests** (Android device): Test Compose UI, Room database
- **Manual Testing**: Exercise tracker requires hands-on feature testing (record exercise, verify display, etc.)

---

### 7. API Level & Device Support

**Decision**: Target Android API 24 (Android 7.0) and above

**Rationale**:
- Aligns with spec requirement: "Targets Android API 24+"
- Covers 95%+ of active Android devices as of 2026
- Jetpack Compose minimum is API 21, so API 24 gives safety margin
- Modern feature access (good Material Design support)
- Jetpack libraries assume API 24+ minimum

---

### 8. IDE & Build System

**Decision**: Android Studio (latest stable) + Gradle

**Rationale**:
- Android Studio: Official Google IDE, built specifically for Android development
- Gradle: Standard build system for Android (integrated into Android Studio)
- Excellent layout preview for Compose
- Free, open-source
- Massive community (all tutorials use this setup)

**Setup**: Android Studio Arctic Fox (latest LTS) or newer

---

## Project Structure

### Android Project Layout

```
android/
├── app/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/example/simplelifting/
│   │   │   │   ├── ui/
│   │   │   │   │   ├── screens/          # Composable screens
│   │   │   │   │   │   ├── RecordExerciseScreen.kt
│   │   │   │   │   │   ├── HistoryScreen.kt
│   │   │   │   │   │   ├── StatsScreen.kt
│   │   │   │   │   │   └── DashboardScreen.kt
│   │   │   │   │   ├── components/       # Reusable Composables
│   │   │   │   │   │   ├── ExerciseForm.kt
│   │   │   │   │   │   ├── ExerciseCard.kt
│   │   │   │   │   │   └── GraphDisplay.kt
│   │   │   │   │   └── theme/            # Material Design theme
│   │   │   │   │       ├── Color.kt
│   │   │   │   │       ├── Type.kt
│   │   │   │   │       └── Theme.kt
│   │   │   │   ├── viewmodel/            # ViewModel classes
│   │   │   │   │   ├── ExerciseViewModel.kt
│   │   │   │   │   ├── StatsViewModel.kt
│   │   │   │   │   └── DashboardViewModel.kt
│   │   │   │   ├── model/                # Data models
│   │   │   │   │   ├── Exercise.kt
│   │   │   │   │   ├── ExerciseSession.kt
│   │   │   │   │   ├── ExerciseSet.kt
│   │   │   │   │   └── UserPreference.kt
│   │   │   │   ├── data/                 # Data layer
│   │   │   │   │   ├── database/
│   │   │   │   │   │   ├── AppDatabase.kt
│   │   │   │   │   │   ├── ExerciseDao.kt
│   │   │   │   │   │   └── SessionDao.kt
│   │   │   │   │   └── repository/
│   │   │   │   │       ├── ExerciseRepository.kt
│   │   │   │   │       └── StatsRepository.kt
│   │   │   │   ├── util/                 # Utility functions
│   │   │   │   │   ├── DateUtils.kt
│   │   │   │   │   ├── UnitConverter.kt
│   │   │   │   │   └── CsvExporter.kt
│   │   │   │   └── MainActivity.kt       # App entry point
│   │   │   └── AndroidManifest.xml
│   │   └── test/
│   │       └── java/com/example/simplelifting/
│   │           ├── viewmodel/
│   │           │   ├── ExerciseViewModelTest.java
│   │           │   └── StatsViewModelTest.java
│   │           ├── data/
│   │           │   ├── ExerciseRepositoryTest.java
│   │           │   └── UnitConverterTest.java
│   │           └── util/
│   │               └── CsvExporterTest.java
│   └── build.gradle
├── build.gradle               # Root build config
└── settings.gradle
```

---

## Key Dependencies

### Core Framework

```gradle
dependencies {
    // Jetpack Compose
    implementation 'androidx.compose.ui:ui:1.5.0'
    implementation 'androidx.compose.material3:material3:1.0.1'
    implementation 'androidx.compose.foundation:foundation:1.5.0'
    
    // Jetpack Lifecycle/ViewModel
    implementation 'androidx.lifecycle:lifecycle-viewmodel-compose:2.6.1'
    implementation 'androidx.lifecycle:lifecycle-runtime:2.6.1'
    
    // Room Database
    implementation 'androidx.room:room-runtime:2.5.2'
    annotationProcessor 'androidx.room:room-compiler:2.5.2'
    implementation 'androidx.room:room-ktx:2.5.2'  // Coroutines support
    
    // Jetpack Navigation (for multi-screen app)
    implementation 'androidx.navigation:navigation-compose:2.7.3'
    
    // Material Icons
    implementation 'androidx.compose.material:material-icons-extended:1.5.0'
    
    // Testing
    testImplementation 'junit:junit:4.13.2'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.5.1'
}
```

### Graph Library (for Statistics)

```gradle
// For drawing progress graphs
implementation 'com.github.PhilJay:MPAndroidChart:v3.1.0'
// OR (more Jetpack-aligned)
// Compose-compatible charting library: MPAndroidChart has Compose wrapper in development
```

---

## Development Setup Checklist

1. **Install Android Studio**: https://developer.android.com/studio
2. **Install Android SDK**: Android Studio downloads automatically
3. **Create Virtual Device**: For testing (emulator) or connect physical device
4. **Clone Repository**: Already done (`/Users/joel/src/simple-lifting`)
5. **Create `/android` directory** in project root
6. **Initialize Android project** with Gradle
7. **Add dependencies** listed above
8. **Configure signing** (needed for release builds, optional for dev)

---

## Learning Roadmap

**Week 1**: Android fundamentals
- Android project structure
- Gradle build system
- Android manifest
- Activities and intents (high-level understanding)

**Week 2-3**: Jetpack Compose basics
- Composable functions
- State management with `mutableStateOf`
- Layouts (Column, Row, Box)
- Material Design 3 components
- Reusable components

**Week 4**: Room Database
- Entity classes
- DAO (Data Access Objects)
- Database creation
- Basic queries (insert, read, update, delete)

**Week 5-6**: ViewModel + LiveData
- Creating ViewModels
- LiveData for observable state
- Coroutines for async database operations
- Reactive UI updates

**Week 7**: Integration & Polish
- Combining everything
- Testing
- Performance optimization
- UI/UX refinement

---

## Known Best Practices to Follow

1. **Composable Functions**: Always keep them lightweight, fast, idempotent
2. **State Management**: Hoist state up (move to ViewModel, not Composable)
3. **Database Queries**: Use coroutines to run on background threads
4. **Original Data Format**: Per Constitution Principle VIII, store weights in original unit (never convert source data)
5. **Error Handling**: Graceful failure with user feedback
6. **Testing**: Write tests before implementation (TDD, per Constitution)

---

## Alternative Considerations (Not Chosen, But Documented)

### Why not Java for this project?
- Kotlin chosen for modern syntax and first-class Jetpack Compose support
- Java remains available (Kotlin/Java interoperability is seamless)
- Kotlin is Google's officially recommended language for Android (since 2019)
- More concise, reduces boilerplate; better learning of modern language paradigms

### Why not traditional XML layouts?
- Compose is Google's current direction
- XML is legacy (still supported, but being phased out)
- Learning Compose teaches modern Android paradigms
- XML fallback available if critical Compose issues arise

### Why not cross-platform frameworks (Flutter, React Native)?
- Project goal: learn Android development (not cross-platform)
- Native Android development gives full platform access
- Best learning experience is native tools

---

## Risks and Mitigation

| Risk | Impact | Mitigation |
|------|--------|-----------|
| Jetpack Compose learning curve | High | Follow official codelabs; reference apps; start with simple layouts |
| Room/SQLite data corruption | Medium | Comprehensive unit tests; backup export feature (CSV) |
| Performance with large dataset | Low (this app unique) | MVP targets reasonable dataset size; optimize queries if needed |
| Device compatibility issues | Low | Target Android API 24+; test on multiple devices/emulators |

---

## Conclusion

**Selected Stack Summary**:
- **Language**: Kotlin (Google-recommended, Compose-optimized)
- **UI**: Jetpack Compose (modern, Google-recommended)
- **Database**: Room (official Android persistence lib)
- **Architecture**: MVVM + LiveData (Google-recommended pattern)
- **Testing**: JUnit + Instrumented tests
- **IDE**: Android Studio
- **Target**: Android 7.0+ (API 24+)

This stack represents the modern standard for Android development as of 2026, balancing learning objectives with production best practices and Google's current recommendations.

**Next Steps**: Phase 1 will generate data model, contracts, and quickstart guide.
