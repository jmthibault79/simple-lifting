# Quickstart: Exercise Tracker MVP

**Phase**: 1 - Design & Contracts  
**Date**: 2026-03-11  
**Last Updated**: 2026-03-12 (Critical: Use Android Studio for gradle, not manual config)  
**Target**: Get app building and running on Android device/emulator in <30 minutes

## CRITICAL LESSON

**Android Studio and AGP (Android Gradle Plugin) are domain-specific tools designed to handle gradle bootstrap and dependency resolution automatically.** Start by opening the IDE, not by manually configuring gradle files. The IDE's error messages are your guide; they point directly to solutions. Avoid spending hours on manual gradle debugging—it's a sign you're not using the right tool.

The quickest path to "app runs": **Use Android Studio's New Project Wizard → Let it generate gradle files → Gradle sync succeeds automatically.**

### What's Updated (3/11/2026)
- ✅ Root `build.gradle.kts`: Removed deprecated `task()` syntax, updated AGP to 8.3.0
- ✅ App `build.gradle.kts`: Updated SDK targets to API 35, Kotlin to 1.9.23, Java to 17, all dependencies current
- ✅ `packagingOptions` → `packaging` (new syntax)
- ✅ Compose BOM to 2025.01.00, Lifecycle to 2.8.0, Room to 2.6.2, Navigation to 2.8.0

---

## Prerequisites

- **Mac (your setup)**: macOS 12+ with M1+ processor
- **Android Studio**: Panda 2 (2025.3.2) or later - [Download](https://developer.android.com/studio)
- **Android SDK**: Automatically installed by Android Studio
  - Minimum SDK: API 24 (Android 7.0)
  - Target SDK: API 35 (Android 15) or latest available
- **Kotlin**: Included in Android Studio (1.9.23+, auto-configured)
- **Git**: Already installed (you're using it)

---

## Step 1: Create Android Project in Android Studio (5 minutes)

**Option A: Fastest — Use Android Studio's New Project Wizard** (RECOMMENDED)

1. Open Android Studio → File → New → New Android Project
2. Select **Empty Activity** template
3. Fill in:
   - Project name: `SimpleLiftingApp`
   - Package: `com.example.simplelifting`
   - Save location: `/Users/joel/src/simple-lifting/android`
   - Language: **Kotlin**
   - Minimum SDK: **API 24**
   - Build system: **Gradle (Kotlin DSL)**
4. Click **Create**
5. **Wait for gradle sync to complete** (Android Studio handles all gradle bootstrap automatically)
6. Green checkmark appears → gradle sync successful

Android Studio has generated all gradle files, configured repositories, and resolved versions automatically. No manual gradle editing needed.

**Option B: Import Existing Project**

1. Open Android Studio → File → Open → `/Users/joel/src/simple-lifting/android`
2. If gradle sync fails, read the error message in the **Build** tab
3. Make the small fix suggested by the error (usually a version update)
4. The IDE re-syncs automatically; green checkmark appears when successful

**Why Option A is faster**: The wizard generates gradle files tailored to your environment. Option B requires fixing errors that the wizard would have prevented.

---

## Step 2: Set Up Virtual Device or Connect Physical Device (5 minutes)

### Option A: Virtual Device (Emulator)
1. Android Studio → Device Manager (left sidebar)
2. Click **Create Device**
3. Select **Pixel 6** (or your preferred device)
4. Choose **API 35** (or API 34, but not lower than API 30)
5. Click **Create**
6. Launch emulator from Device Manager

### Option B: Physical Device
1. Enable Developer Mode: Settings → About Phone → tap Build Number 7x
2. Enable USB Debugging: Settings → Developer Options → USB Debugging
3. Connect to Mac via USB
4. Android Studio recognizes device automatically

---

## Step 3: Create Basic Kotlin Entity Classes (5 minutes)

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

## Step 4: Create Room Database (5 minutes)

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

## Step 5: Create DAOs (Data Access Objects)

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

## Step 6: First Build & Run (5 minutes)

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

## Step 7: Create First Composable Screen (Optional Quick Test)

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
| "'task()' is deprecated" error | Ensure root `build.gradle.kts` uses only plugin declarations (no custom tasks) |
| "Cannot add task 'clean'" error | Don't manually define clean task; Gradle provides it by default |
| "packagingOptions deprecated" | Updated to `packaging {}` block (see gradle files above) |
| "Emulator won't start" | Choose a different API level (34-35 most stable); ensure sufficient disk space |
| "Android Studio can't find SDK" | File → Settings → Android SDK → check paths, ensure SDK is installed |
| "Compose compiler extension mismatch" | Ensure Kotlin version matches `kotlinCompilerExtensionVersion` in build.gradle |
| "Room schema not generated" | Ensure `@Database` annotation present, rebuild with `./gradlew clean build` |

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
