# Quickstart: Exercise Tracker MVP

**Phase**: 1 - Design & Contracts  
**Date**: 2026-03-11  
**Target**: Get app building and running on Android device/emulator in <30 minutes

---

## Prerequisites

- **Mac (your setup)**: macOS 12+ with M1+ processor
- **Android Studio**: Latest stable version (tested with 2024.1+) - [Download](https://developer.android.com/studio)
- **Android SDK**: Automatically installed by Android Studio
  - Minimum SDK: API 24 (Android 7.0)
  - Target SDK: API 34 (Android 14) or latest available
- **Kotlin**: Included in Android Studio
- **Git**: Already installed (you're using it)

---

## Step 1: Project Setup (5 minutes)

### 1a. Create Android Project Structure

```bash
cd /Users/joel/src/simple-lifting
mkdir -p android
cd android
```

### 1b. Initialize Gradle Project

Create `build.gradle.kts` (root project):

```kotlin
plugins {
    id("java-library")
    id("com.android.application") version "8.1.0" apply false
    id("com.android.library") version "8.1.0" apply false
    id("org.jetbrains.kotlin.android") version "1.9.0" apply false
}

task("clean", Delete::class) {
    delete(rootProject.buildDir)
}
```

Create `settings.gradle.kts`:

```kotlin
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}

dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}

rootProject.name = "SimpleLiftingApp"
include(":app")
```

Create `app/build.gradle.kts`:

```kotlin
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
    id("androidx.room")
    id("com.google.devtools.ksp")
}

android {
    namespace = "com.example.simplelifting"
    compileSdk = 34

    defaultConfig {
        applicationId = "com.example.simplelifting"
        minSdk = 24
        targetSdk = 34
        versionCode = 1
        versionName = "0.1.0"

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
        vectorDrawables {
            useSupportLibrary = true
        }
    }

    buildTypes {
        release {
            isMinifyEnabled = false
            proguardFiles(getDefaultProguardFile("proguard-android-optimize.txt"), "proguard-rules.pro")
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_11
        targetCompatibility = JavaVersion.VERSION_11
    }

    kotlinOptions {
        jvmTarget = "11"
    }

    buildFeatures {
        compose = true
    }

    composeOptions {
        kotlinCompilerExtensionVersion = "1.5.1"
    }

    packagingOptions {
        resources {
            excludes += "/META-INF/{AL2.0,LGPL2.1}"
        }
    }
}

dependencies {
    // Jetpack Compose (BOM ensures compatible versions)
    val composeBom = platform("androidx.compose:compose-bom:2024.01.00")
    implementation(composeBom)
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.ui:ui-graphics")
    implementation("androidx.compose.ui:ui-tooling-preview")
    implementation("androidx.compose.material3:material3")
    implementation("androidx.compose.material:material-icons-extended")

    // Jetpack Lifecycle & ViewModel
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.7.0")
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.7.0")

    // Room Database
    implementation("androidx.room:room-runtime:2.6.1")
    implementation("androidx.room:room-ktx:2.6.1")
    ksp("androidx.room:room-compiler:2.6.1")

    // Navigation
    implementation("androidx.navigation:navigation-compose:2.7.7")

    // Core Android
    implementation("androidx.core:core-ktx:1.13.0")
    implementation("androidx.activity:activity-compose:1.8.1")

    // Testing - Unit
    testImplementation("junit:junit:4.13.2")
    testImplementation("org.mockito.kotlin:mockito-kotlin:5.1.0")
    testImplementation("org.mockito:mockito-core:5.5.1")

    // Testing - Instrumented
    androidTestImplementation(composeBom)
    androidTestImplementation("androidx.test.espresso:espresso-core:3.5.1")
    androidTestImplementation("androidx.compose.ui:ui-test-junit4")

    // Debug
    debugImplementation("androidx.compose.ui:ui-tooling")
    debugImplementation("androidx.compose.ui:ui-test-manifest")
}

room {
    schemaDirectory("$projectDir/schemas")
}
```

---

## Step 2: Create Android Project in Android Studio (5 minutes)

### Option A: Import Existing Project
1. Open Android Studio
2. File → Open → `/Users/joel/src/simple-lifting/android`
3. Android Studio detects `build.gradle.kts` and imports

### Option B: Create New Project
1. File → New → New Android Project
2. Choose **Empty Activity** template
3. Project name: `SimpleLiftingApp`
4. Package: `com.example.simplelifting`
5. Language: **Kotlin**
6. Minimum SDK: **API 24**
7. Select `build system: Gradle (Kotlin DSL)`

---

## Step 3: Set Up Virtual Device or Connect Physical Device (5 minutes)

### Option A: Virtual Device (Emulator)
1. Android Studio → Device Manager (left sidebar)
2. Click **Create Device**
3. Select **Pixel 6** (or your preferred device)
4. Choose **API 30** (or higher, but not higher than targetSdk)
5. Click **Create**
6. Launch emulator from Device Manager

### Option B: Physical Device
1. Enable Developer Mode: Settings → About Phone → tap Build Number 7x
2. Enable USB Debugging: Settings → Developer Options → USB Debugging
3. Connect to Mac via USB
4. Android Studio recognizes device automatically

---

## Step 4: Create Basic Kotlin Entity Classes (5 minutes)

Create `app/src/main/kotlin/com/example/simplelifting/model/Exercise.kt`:

```kotlin
package com.example.simplelifting.model

import androidx.room.Entity
import androidx.room.PrimaryKey
import java.time.LocalDate
import java.time.LocalDateTime

@Entity(tableName = "exercises")
data class Exercise(
    @PrimaryKey(autoGenerate = true) val id: Long = 0,
    val name: String,
    val description: String? = null,
    val personalRecord: Double? = null,
    val personalRecordUnit: String? = null,  // "lbs" or "kg"
    val lastPerformed: LocalDate? = null,
    val createdAt: LocalDateTime = LocalDateTime.now()
)
```

Create `app/src/main/kotlin/com/example/simplelifting/model/ExerciseSession.kt`:

```kotlin
package com.example.simplelifting.model

import androidx.room.Entity
import androidx.room.PrimaryKey
import java.time.LocalDate
import java.time.LocalDateTime

@Entity(tableName = "exercise_sessions")
data class ExerciseSession(
    @PrimaryKey(autoGenerate = true) val id: Long = 0,
    val sessionDate: LocalDate,
    val notes: String? = null,
    val createdAt: LocalDateTime = LocalDateTime.now(),
    val updatedAt: LocalDateTime = LocalDateTime.now()
)
```

Create `app/src/main/kotlin/com/example/simplelifting/model/ExerciseSet.kt`:

```kotlin
package com.example.simplelifting.model

import androidx.room.Entity
import androidx.room.ForeignKey
import androidx.room.Index
import androidx.room.PrimaryKey
import java.time.LocalDateTime

@Entity(
    tableName = "exercise_sets",
    foreignKeys = [
        ForeignKey(entity = Exercise::class, parentColumns = ["id"], childColumns = ["exerciseId"]),
        ForeignKey(entity = ExerciseSession::class, parentColumns = ["id"], childColumns = ["sessionId"], onDelete = ForeignKey.CASCADE)
    ],
    indices = [
        Index("exerciseId"),
        Index("sessionId"),
        Index(value = ["exerciseId", "sessionId"])
    ]
)
data class ExerciseSet(
    @PrimaryKey(autoGenerate = true) val id: Long = 0,
    val exerciseId: Long,
    val sessionId: Long,
    val weight: Double,  // numeric value
    val weightUnit: String,  // "lbs" or "kg" - IMMUTABLE
    val repsCompleted: Int,
    val setNumber: Int,
    val notes: String? = null,
    val createdAt: LocalDateTime = LocalDateTime.now()
)
```

---

## Step 5: Create Room Database (5 minutes)

Create `app/src/main/kotlin/com/example/simplelifting/data/AppDatabase.kt`:

```kotlin
package com.example.simplelifting.data

import android.content.Context
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase
import androidx.room.TypeConverters
import com.example.simplelifting.model.Exercise
import com.example.simplelifting.model.ExerciseSession
import com.example.simplelifting.model.ExerciseSet
import com.example.simplelifting.util.Converters

@Database(
    entities = [Exercise::class, ExerciseSession::class, ExerciseSet::class],
    version = 1
)
@TypeConverters(Converters::class)
abstract class AppDatabase : RoomDatabase() {
    abstract fun exerciseDao(): ExerciseDao
    abstract fun sessionDao(): SessionDao
    abstract fun exerciseSetDao(): ExerciseSetDao

    companion object {
        @Volatile
        private var Instance: AppDatabase? = null

        fun getDatabase(context: Context): AppDatabase {
            return Instance ?: synchronized(this) {
                Room.databaseBuilder(context, AppDatabase::class.java, "simple_lifting.db")
                    .fallbackToDestructiveMigration()  // For dev only; remove for production
                    .build()
                    .also { Instance = it }
            }
        }
    }
}
```

Create `app/src/main/kotlin/com/example/simplelifting/util/Converters.kt`:

```kotlin
package com.example.simplelifting.util

import androidx.room.TypeConverter
import java.time.LocalDate
import java.time.LocalDateTime
import java.time.format.DateTimeFormatter

class Converters {
    private val dateFormatter = DateTimeFormatter.ISO_LOCAL_DATE
    private val dateTimeFormatter = DateTimeFormatter.ISO_LOCAL_DATE_TIME

    @TypeConverter
    fun fromLocalDate(date: LocalDate?): String? = date?.format(dateFormatter)

    @TypeConverter
    fun toLocalDate(dateString: String?): LocalDate? = dateString?.let { LocalDate.parse(it, dateFormatter) }

    @TypeConverter
    fun fromLocalDateTime(dateTime: LocalDateTime?): String? = dateTime?.format(dateTimeFormatter)

    @TypeConverter
    fun toLocalDateTime(dateTimeString: String?): LocalDateTime? = dateTimeString?.let { LocalDateTime.parse(it, dateTimeFormatter) }
}
```

---

## Step 6: Create DAOs (Data Access Objects)

Create `app/src/main/kotlin/com/example/simplelifting/data/ExerciseDao.kt`:

```kotlin
package com.example.simplelifting.data

import androidx.room.Dao
import androidx.room.Delete
import androidx.room.Insert
import androidx.room.Query
import androidx.room.Update
import com.example.simplelifting.model.Exercise

@Dao
interface ExerciseDao {
    @Insert
    suspend fun insert(exercise: Exercise): Long

    @Update
    suspend fun update(exercise: Exercise)

    @Delete
    suspend fun delete(exercise: Exercise)

    @Query("SELECT * FROM exercises WHERE id = :id")
    suspend fun getExerciseById(id: Long): Exercise?

    @Query("SELECT * FROM exercises WHERE name = :name")
    suspend fun getExerciseByName(name: String): Exercise?

    @Query("SELECT * FROM exercises ORDER BY name ASC")
    suspend fun getAllExercises(): List<Exercise>

    @Query("SELECT * FROM exercises ORDER BY personalRecord DESC LIMIT 1")
    suspend fun getPersonalRecordExercise(): Exercise?
}
```

Create similar `SessionDao.kt` and `ExerciseSetDao.kt` following the same pattern.

---

## Step 7: First Build & Run (5 minutes)

### Build
```bash
cd /Users/joel/src/simple-lifting/android
./gradlew build
```

Expected: Green checkmark, "BUILD SUCCESSFUL"

### Run on Device/Emulator
```bash
./gradlew installDebug
```

Or in Android Studio: Run → Run 'app'

**Expected Result**: App launches with blank screen (no UI yet - that's Phase 1 Compose screens)

---

## Step 8: Create First Composable Screen (Optional Quick Test)

Create `app/src/main/kotlin/com/example/simplelifting/ui/screens/ExerciseRecorderScreen.kt`:

```kotlin
package com.example.simplelifting.ui.screens

import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.Button
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.material3.TopAppBar
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

@Composable
fun ExerciseRecorderScreen() {
    Scaffold(
        topBar = {
            TopAppBar(title = { Text("Record Exercise") })
        }
    ) { paddingValues ->
        Column(modifier = Modifier.padding(paddingValues).padding(16.dp)) {
            Text("Exercise Recorder - Coming Soon!")
            Button(onClick = { /* TODO */ }) {
                Text("Record New Exercise")
            }
        }
    }
}
```

---

## Verification Checklist

- [ ] Android Studio opens project without errors
- [ ] `build.gradle.kts` files valid (no red squigglies)
- [ ] App compiles: `./gradlew build` → "BUILD SUCCESSFUL"
- [ ] App installs on emulator/device: `./gradlew installDebug`
- [ ] App launches without crashing
- [ ] Room database schema created (check `/data/data/com.example.simplelifting/databases/`)

---

## Next Steps (Phase 1 Continued)

1. **Create UI Screens** (Jetpack Compose):
   - ExerciseRecorderScreen (record exercise session)
   - HistoryScreen (view chronological exercise history)

2. **Implement ViewModels** (business logic):
   - ExerciseRecorderViewModel (handle record action)
   - HistoryViewModel (fetch and filter history)

3. **Implement Repositories** (data layer):
   - ExerciseRepository (CRUD operations)
   - Create wiring between ViewModel → Repository → DAO → RoomDatabase

4. **Write Unit Tests**:
   - DAO tests (Room queries)
   - Repository tests (with mocked DAO)
   - ViewModel tests (with mocked repository)

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "Gradle build fails with 'cannot find symbol'" | Run `./gradlew clean build` |
| "Emulator won't start" | Choose a different API level (29-34 most stable) |
| "Android Studio can't find SDK" | File → Settings → Android SDK → check paths |
| "Room schema not generated" | Ensure `@Database` annotation present, rebuild |

---

## Command Reference

```bash
# Clean build
./gradlew clean

# Build app
./gradlew build

# Run tests
./gradlew test              # Unit tests
./gradlew connectedAndroidTest  # Instrumented tests

# Install on device
./gradlew installDebug

# View device logs
adb logcat | grep SimpleLifting
```

---

## Time Estimates (MVP Development)

| Phase | Time | Notes |
|-------|------|-------|
| Setup (Steps 1-7) | 30 min | One-time setup |
| UI Screens (Compose) | 2-3 hours | ExerciseRecorderScreen, HistoryScreen |
| ViewModels | 1-2 hours | Business logic wiring |
| Repositories | 1 hour | Data layer abstraction |
| Unit Tests | 1-2 hours | DAO, Repository, ViewModel tests |
| **Total MVP** | **6-9 hours** | Thinnest deployable slice (US1+US2) |

**Ready to build!**
