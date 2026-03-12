# DETAILED: How to Run Your Android App

## START HERE: Trust Android Studio

**Android Studio is the canonical tool for Android development.** It handles:
- Gradle bootstrap (automatic)
- Dependency resolution (automatic)
- Version compatibility checking (automatic)
- Build configuration (automatic)

**Your fastest path**:
1. Open Android Studio
2. File → Open → `/Users/joel/src/simple-lifting/android`
3. Wait for gradle sync (green checkmark = success)
4. Click **Run 'app'** button (green play icon)
5. App launches on your emulator/device

If gradle sync shows an error in the **Build** tab, read it carefully—it tells you exactly what to fix. Fix it, let the IDE re-sync, and you're done.

**Don't manually edit gradle unless Android Studio tells you to.** The IDE is specifically designed to prevent this.

---

## What "Run App" Means

In Android development, "running your app" involves these layers:

1. **Compilation** (Java/Kotlin → machine code)
2. **Packaging** (code + resources → APK file)
3. **Installation** (APK → device/emulator)
4. **Execution** (app launches and talks to Android OS)

Android Studio handles all of this for you. This guide covers what's actually happening under the hood.

---

## PART 1: Prerequisite Setup (Do This First)

### Step 1: Install Android Studio

Download **Android Studio Panda 2 (2025.3.2+)** from [developer.android.com/studio](https://developer.android.com/studio)

Why Panda 2? Your project uses modern Jetpack libraries (Compose, Lifecycle 2.8.0) that assume Panda 2's tooling. Each Android Studio version is optimized for a specific AGP/gradle/Kotlin ecosystem.

**After installation:**
1. Open Android Studio
2. Go to **Settings → Languages & Frameworks → Android SDK**
3. Ensure you have:
   - **SDK Platforms**: API 34, API 35
   - **SDK Tools**: Android Emulator, Android SDK Platform-Tools, Kotlin Plugin 1.9.23+

### Step 2: Set Up Java 17

Verify your Java version (Android requires 17+):

```bash
java -version
# Expected output: "17.0.x" or higher
```

If you get an error or old version, install Temurin JDK 17:
```bash
# Mac with Homebrew
brew install temurin@17
echo 'export JAVA_HOME=$(/usr/libexec/java_home -v 17)' >> ~/.zshrc
source ~/.zshrc
```

Verify again:
```bash
java -version  # Should show 17.x.x
echo $JAVA_HOME  # Should point to Temurin JDK 17
```

### Step 3: Create or Attach an Android Emulator

**Option A: Use Android Studio's Device Manager (Easiest)**

1. Open Android Studio
2. Click **Device Manager** (right side toolbar)
3. Click **Create Device**
4. Select **Pixel 6** (or any phone device)
5. Select **API 35** or **API 34**
6. Name it `Pixel_6_API_35`
7. Click **Create**
8. Click the **Play** button to launch it

Wait 2-3 minutes for the emulator to boot. You'll see an Android lock screen.

**Option B: Use Physical Device (More Complex)**

If you have an Android phone:
1. Connect via USB cable to your Mac
2. Enable Developer Mode: Settings → About Phone → Tap "Build Number" 7x
3. Enable USB Debugging: Settings → Developer Options → USB Debugging → Allow
4. On your Mac:
   ```bash
   adb devices
   # Should show your device listed as "device" (not "offline")
   ```

If showing "offline", restart ADB:
```bash
adb kill-server
adb start-server
adb devices
```

---

## PART 2: Open Android Studio and Let It Handle Gradle

### The Easy Path

1. **Open Android Studio**
2. **File → Open** → `/Users/joel/src/simple-lifting/android`
3. **Wait for gradle to sync** (you'll see "Gradle: Syncing" at bottom)
4. When syncing completes, a green checkmark appears → success
5. Click the green **Run** button (top toolbar)
6. Select your device/emulator
7. App launches

That's it. Android Studio has handled everything automatically.

### If Gradle Sync Fails

1. Look at the **Build** tab (usually appears automatically with red error text)
2. Read the error message carefully — it tells you exactly what's wrong
3. **Common errors**:
   - "Could not find androidx.something:something:X.Y.Z" → Version doesn't exist, use an older version or newer version
   - "Plugin 'kotlin-android' not found" → Missing kotlin plugin declaration
   - "AGP version X requires Gradle Y" → Gradle/AGP mismatch
4. Make the fix suggested by the error
5. Click **Sync Now** (button appears in the editor)
6. Wait for sync to complete again

**That sync error message is your teacher.** It's pointing you to the exact fix. Don't try to manually debug gradle—let the IDE guide you.

---

## PART 3: Advanced — Understanding Gradle Manually (Optional)

If you want to understand what's happening under the hood, or if Android Studio's sync doesn't work:

### Manual Gradle Verification

### Step A: Update app/build.gradle.kts Dependencies (Only if IDE Tells You To)

If Android Studio's gradle sync error says a dependency version doesn't exist, replace it with a version that does. The versions below are **known to work as of March 2026**:

```kotlin
dependencies {
    // Jetpack Compose (use compatible BOM version)
    val composeBom = platform("androidx.compose:compose-bom:2024.12.00")
    implementation(composeBom)
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.ui:ui-graphics")
    implementation("androidx.compose.ui:ui-tooling-preview")
    implementation("androidx.compose.material3:material3")
    implementation("androidx.compose.material:material-icons-extended")

    // Jetpack Lifecycle & ViewModel
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.7.0")
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.7.0")

    // Room Database (use 2.5.x which is stable)
    implementation("androidx.room:room-runtime:2.5.2")
    implementation("androidx.room:room-ktx:2.5.2")
    ksp("androidx.room:room-compiler:2.5.2")

    // Navigation
    implementation("androidx.navigation:navigation-compose:2.7.7")

    // Core Android
    implementation("androidx.core:core-ktx:1.12.0")
    implementation("androidx.activity:activity-compose:1.8.1")

    // Testing - Unit
    testImplementation("junit:junit:4.13.2")
    testImplementation("org.mockito.kotlin:mockito-kotlin:5.1.0")
    testImplementation("org.mockito:mockito-core:5.6.1")

    // Testing - Instrumented
    androidTestImplementation(composeBom)
    androidTestImplementation("androidx.test.espresso:espresso-core:3.5.1")
    androidTestImplementation("androidx.compose.ui:ui-test-junit4")

    // Debug
    debugImplementation("androidx.compose.ui:ui-tooling")
    debugImplementation("androidx.compose.ui:ui-test-manifest")
}
```

### Step B: Update Kotlin Compiler Extension

In your `app/build.gradle.kts`, find the `composeOptions` block and change it:

```kotlin
composeOptions {
    kotlinCompilerExtensionVersion = "1.5.11"  // Keep this - compatible with Kotlin 1.9.23
}
```

### Step C: Verify gradle.properties

Ensure `/android/gradle.properties` contains:

```properties
# AndroidX Support
android.useAndroidX=true
android.enableJetifier=true

# Gradle Memory (safe for Java 17)
org.gradle.jvmargs=-Xmx2048m

# Kotlin
kotlin.code.style=official
```

Do **NOT** include `-XX:MaxPermSize` — it doesn't exist in Java 17.

---

## PART 4: Command-Line Build (If You Prefer Terminal)

**Prefer Android Studio for building (it's more visual).** But if you want to build from the command line:

```bash
cd /Users/joel/src/simple-lifting/android

# Clean old artifacts
./gradlew clean

# Full build
./gradlew build

# Expected output (last 3 lines):
# > Task :app:bundleDebug
# > Task :app:assembleDebug
# BUILD SUCCESSFUL in XXs
```

**If build fails, DON'T try to debug gradle manually.** Instead:
1. Open Android Studio
2. File → Open → this project
3. Look at the Build tab error message
4. Make the fix it suggests

The IDE is much better at diagnosing gradle problems than terminal error messages.

---

## PART 5: Install and Run App

**Easiest Method: Android Studio UI**

1. Click the green **Run** button (top toolbar) or press `Ctrl + R` (Mac)
2. Select your device/emulator from the popup
3. Android Studio handles build → install → launch automatically
4. Watch the **Logcat** panel (bottom) for app output

**If you prefer command line:**

```bash
cd /Users/joel/src/simple-lifting/android

# Build and install
./gradlew installDebug

# Launch
adb shell am start -n com.example.simplelifting/.MainActivity

# Watch logs
adb logcat | grep simplelifting
```

Expected: App appears on device with "Hello Simple Lifting!" screen.

---

## PART 6: Debug If App Crashes

### Scenario A: "App Keeps Crashing"

1. Open Logcat in Android Studio
2. Search for "FATAL" or "Exception"
3. Look for lines like:
   ```
   java.lang.RuntimeException: Unable to start activity ComponentInfo{com.example.simplelifting/.MainActivity}: android.view.InflateException: ...
   ```

4. Common causes:
   - **Missing layout file**: Check that `res/layout/activity_main.xml` exists
   - **Theme mismatch**: Verify `@style/Theme.SimpleLiftingApp` exists in `res/values/styles.xml`
   - **Null pointer in onCreate()**: Check MainActivity.kt onCreate() method

### Scenario B: "App Won't Install"

```bash
./gradlew installDebug 2>&1 | grep -i error
```

Common errors:
- `INSTALL_FAILED_INVALID_APK` → Rebuild with `./gradlew clean build`
- `INSTALL_FAILED_INSUFFICIENT_STORAGE` → Clear emulator cache or use larger device
- `INSTALL_FAILED_VERSION_DOWNGRADE` → Uninstall first: `adb uninstall com.example.simplelifting`

### Scenario C: "Device Not Found"

```bash
adb devices
# Output should show something like:
# emulator-5554    device
# OR
# XXXXXXXXXXXXXX   device (if physical phone)

# If emulator not showing:
adb kill-server
adb start-server
adb devices
```

---

## PART 7: Troubleshooting Gradle Sync (Read This Only If Sync Fails)

| Issue | Fix |
|-------|-----|
| `Could not find androidx.core:core-ktx:X.X.X` | Reduce version number by 1 minor version in `app/build.gradle.kts` and rebuild |
| `Unrecognized VM option 'MaxPermSize'` | Remove `-XX:MaxPermSize` from `gradle.properties` (Java 17 doesn't support it) |
| `Plugin [id: 'com.google.devtools.ksp'] not found` | Add `id("com.google.devtools.ksp")` to root `build.gradle.kts` `plugins {}` block |
| `Task 'installDebug' not found` | Ensure you're running from `/android` directory, not root `/simple-lifting` |
| `Gradle build daemon got disconnected` | Run `./gradlew --stop` then rebuild |

---

## PART 9: Emulator-Specific Tips

**Emulator is slow:**
- Use **API 34** instead of API 35 (slightly faster)
- Close other apps on your Mac (frees RAM)
- Enable GPU acceleration: In Device Manager, click ⚙️ settings on your device → Graphics → **Hardware**

**Changing default device:**
1. Device Manager → Click device → Click gear icon
2. Change Startup Options
3. Restart emulator

**Emulator shows black screen for 5+ minutes:**
- This is normal on first boot
- Don't force-quit
- Wait for Android to fully boot (you'll see the lock screen)

---

## PART 10: Understanding What Each Step Actually Does

### `./gradlew build`
- Compiles `.kt` files → bytecode
- Validates for Android API compatibility
- Packages resources (strings.xml, drawables, etc.)
- Generates APK file at `app/build/outputs/apk/debug/app-debug.apk`

### `./gradlew installDebug`
- Takes the APK from build step
- Copies it to device storage via ADB
- Android OS extracts and installs it
- App is registered in device's app list

### `adb shell am start -n ...`
- Tells Android OS to launch MainActivity
- Android creates  process
- MainActivity.onCreate() runs
- Your Compose UI renders

---

## SUMMARY CHECKLIST

Before claiming "app is running":

- ✅ Java 17 is installed and $JAVA_HOME is set
- ✅ Android Studio Panda 2 has SDK API 34/35 installed
- ✅ Emulator or physical device is connected and visible in `adb devices`
- ✅  `./gradlew clean build` completes with "BUILD SUCCESSFUL"
- ✅ `./gradlew installDebug` completes with no errors
- ✅ App icon appears on device/emulator home screen
- ✅ Tapping app shows "Hello Simple Lifting!" screen
- ✅ Logcat shows no FATAL exceptions

If all boxes checked, your app is **successfully running**.

---

## NEXT: Modify the App

Once the app runs, you can:
1. Edit code in `MainActivity.kt`
2. Save file (Ctrl+S / Cmd+S)
3. Android Studio auto-rebuilds in background
4. Press `Shift+F10` to reinstall and rerun
5. See changes instantly on device

This live-reload cycle is crucial to Android development.
