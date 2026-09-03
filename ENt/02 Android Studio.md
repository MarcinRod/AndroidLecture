# Android Studio Environment

When creating an application in Android Studio, the project consists of many directories and files that serve specific roles. In this chapter we will discuss:

- the most important elements of project structure,
- basics of configuration using Gradle,
- Android Studio as a development environment.

---

## Android Studio – Basics

**Android Studio** is the official IDE (integrated development environment) for creating Android applications, based on IntelliJ IDEA by JetBrains.

### Main Features of Android Studio

- Support for **Jetpack Compose**, **ViewModel**, **LiveData**, **Room**, etc.
- Advanced **debugging tools** (Logcat, Debugger, Profiler).
- **Emulator** for testing applications without a physical device.
- **Compose Preview** – preview UI components without running the application.
- Automatic **library and SDK updates**.
- Integration with **Firebase**, **Git**, **Gradle**.
- **Plugins** for many languages and technologies.

### Most Important Panels

- **Project** – directory and file structure.
- **Editor** – main code and layout editor.
- **Logcat** – system and application logs.
- **Build** – build status.
- **Device Manager** – manage emulators and physical devices.

---

## Selected Project Details

This section covers important aspects of working in Android Studio that often come up when learning to create mobile applications.

---

### Package Name

**Package name** is the unique identifier for an application in Android system. It is recorded in `AndroidManifest.xml` file and in `build.gradle` as `applicationId`.

**Example:**

```kotlin
// app/build.gradle
defaultConfig {
    applicationId "com.example.myapp"
    ...
}
```

- **`com.example.myapp`** – example of "reverse domain" convention.
- **Package name ≠ directory structure**, but they are usually aligned for readability.
- Package name affects:
  - where application is installed on device,
  - application identification in Google Play,
  - generating `BuildConfig` file and keys for Firebase.

> **Note:** Changing `applicationId` in `build.gradle` allows publishing a different version of the application without conflict with the previous one.

---

### Project Views: Android Studio vs File System

Android Studio offers different **project views** to facilitate work:

#### Android View

- Groups files logically (e.g., all layouts together).
- Hides unnecessary directories (`build`, `intermediates`).
- Most commonly used by beginners and in daily work.

#### Project View

- Shows **actual directory structure on disk**.
- Useful when working with `gradle` files, CI/CD configurations, etc.
- Facilitates navigation through the project filesystem.

#### Project Directory Structure

After creating a new project in Android Studio, the default disk structure looks roughly like this:

```
MyApp/
├── app/
│   ├── build.gradle
│   ├── src/
│   │   ├── androidTest/   
│   │   ├── main/
│   │   │   ├── java/
│   │   │   │   └── com/example/myapp/
│   │   │   │       └── MainActivity.kt
│   │   │   ├── res/
│   │   │   │   ├── layout/
│   │   │   │   ├── values/
│   │   │   │   └── drawable/
│   │   │   └── AndroidManifest.xml
│   │   └── test/         
├── build.gradle
├── settings.gradle
├── gradle.properties
└── gradle/
```

In Android view, example structure for new project is as follows:

```
MyApp (Android)
├── manifests
│   └── AndroidManifest.xml
├── java
│   ├── com.example.myapp
│   │   └── MainActivity.kt
│   └── (Tests and AndroidTest)
├── res
│   ├── drawable/       
│   ├── layout/         
│   ├── mipmap/          
│   ├── values/          
│   └── themes/           
├── Gradle Scripts
│   ├── build.gradle (Project: MyApp)
│   ├── build.gradle (Module: app)
│   ├── settings.gradle
│   └── gradle.properties
```

#### Element Descriptions

- **`manifests/AndroidManifest.xml`**  
  - Declares basic application information: package, activities, permissions.

- **`java/com.example.myapp`**  
  - Main application code space. Contains classes and functions (e.g., `MainActivity.kt`, `Repository.kt`, `ViewModel.kt`).
  - Can create subfolders by architecture: `ui`, `data`, `domain`, etc.

- **`res/`**  
  - Application resources – images, colors, texts, styles, etc.

- **`Gradle Scripts/`**  
  - Configuration scripts:
    - `build.gradle (Project)` – general project configuration (e.g., plugin versions, repositories).
    - `build.gradle (Module)` – specific module configuration (`applicationId`, dependencies, versions).
    - `settings.gradle` – defines which modules are part of project.
    - `gradle.properties` – global Gradle properties (e.g., JVM memory size, optimization flags).

> Android view doesn't show `build/`, `intermediates/`, `outputs/` directories – they are hidden to not interfere with daily work.

---

The "Android" view is the most convenient way to browse the project during daily programming, however "Project" view shows exact file layout on disk.

---

## Android Emulator

**Android Emulator** allows running an application without a physical device. All devices are managed through `Device Manager`.

### Advantages

1. **Quick and convenient testing**
   - Ability to immediately run an application without needing to connect a physical device.
  
2. **Multiple device configurations**
   - Ability to create multiple virtual devices (AVD – Android Virtual Devices) with different Android versions, screen sizes, dpi, RAM, etc.
   - Useful for testing UI responsiveness on different device types (e.g., phone, tablet, foldable).

3. **Hardware feature simulation**
   - Ability to simulate:
     - GPS location,
     - phone calls and SMS,
     - battery level,
     - network conditions (e.g., 2G, 3G, Wi-Fi),
     - sensors (accelerometer, gyroscope, etc.),
     - cameras (front and back),
     - screen rotation.
   - These features are often difficult to test on a single physical device.

4. **Better logging and debugging**
   - Easy integration with Android Studio tools: Logcat, Debugger, Profiler.
   - Quick access to system logs and ability to inspect resources.

5. **Security and isolation**
   - Emulator runs in isolated environment – physical device is not exposed to damage or accidental data overwrite.

6. **No need to buy multiple devices**
   - Allows testing applications on different Android versions (even older) and hardware configurations without investment in real phones or tablets.

7. **Quick reset and data cleanup**
   - Ability to quickly restore emulator to initial state – useful for tests related to onboarding, permissions, or first launch.

### Emulator Disadvantages

1. **High resource consumption**
   - Emulator consumes significant amount of RAM, CPU, and disk space.
   - On weaker computers it can significantly reduce system performance.

2. **Slower performance compared to physical devices**
   - Even with hardware acceleration, emulator can run less smoothly.
   - Application startup time can be longer.

3. **Hardware feature limitations**
   - Not all hardware components are fully simulated (e.g., NFC, Bluetooth, sensors, camera).
   - Feature behavior may differ from real devices.

---

## Debugging and Logcat

Debugging is the process of finding and removing errors (bugs) from code. Android Studio offers a rich set of tools for debugging applications, with the most important being **Logcat** and **debugger with breakpoint system**.

---

### Debugging Application

Android Studio allows debugging an application step by step, stopping execution at specific locations and viewing variable values and call stack.

#### Breakpoint

- Breakpoint is stopping point – location where debugger will stop execution.
- Can be set by clicking on left margin of code editor at given line.
- Allows checking application state at given moment: variable values, method calls, etc.

#### Debugger

- Application in debug mode is launched with "Debug app" button with bug icon.
- After stopping at breakpoint you can view variable state, registers, and call stack.

---

### Logcat – Error Analysis and Exception Diagnostics

**Logcat** is the most important diagnostic tool in Android Studio – enables tracking system logs and application errors in real time. Helps detect exceptions, error messages, warnings, and system actions.

---

### Logging Levels

| Level | Meaning                          |
|-------|----------------------------------|
| `V`   | Verbose – all logs               |
| `D`   | Debug – developer logs           |
| `I`   | Info – general information       |
| `W`   | Warn – warnings                  |
| `E`   | Error – critical errors          |
| `A`   | Assert – critical conditions     |

---

### Logging Example

```kotlin
import android.util.Log

Log.d("LoginScreen", "Login button clicked")
Log.w("LoginScreen", "No server connection")
Log.e("LoginScreen", "Authorization error", exception)
```

---

### Exception Example in Logcat

Suppose application threw `NullPointerException`. In Logcat you would see something like:

```bash
E/AndroidRuntime: FATAL EXCEPTION: main
    Process: com.example.myapp, PID: 31548
    java.lang.NullPointerException: Attempt to invoke virtual method 'java.lang.String android.widget.TextView.getText()' on a null object reference
        at com.example.myapp.ui.MainActivity.onCreate(MainActivity.kt:25)
        at android.app.Activity.performCreate(Activity.java:7893)
        ...
```

---

### How to Analyze Exception

1. **Exception type**: `NullPointerException`  
   ➤ Means application tried to access an object that was `null`.

2. **Error description**:
   ```
   Attempt to invoke virtual method 'java.lang.String android.widget.TextView.getText()' on a null object reference
   ```
   ➤ `getText()` was called on uninitialized `TextView` object.

3. **Error location**:
   ```
   at com.example.myapp.ui.MainActivity.onCreate(MainActivity.kt:25)
   ```
   ➤ Error is in `MainActivity` class, line 25.

---

### Typical Code Causing Exception

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    val text = findViewById<TextView>(R.id.myTextView)
    val input = text.text.toString() // <- exception if `text` is null
}
```

---

### Filtering Logcat

To quickly find error or information of interest in Logcat:
- Select appropriate logging level e.g. **Error**
- In filter enter application name or tag, e.g.: `com.example.myapp`, `LoginScreen`

---

### Debugging + Logcat

After finding error in Logcat, it's worth:
- Setting **breakpoint** at suspected line
- Running application in debug mode and analyzing code

---

**Tip**: Logcat works best when relevant information is regularly logged in code – not just errors, but also application flow (e.g., screen transitions, user input, API responses).

---

### Other Debugging Tools

- **Layout Inspector** – analyze view structure in real time.
- **Network Profiler** – view network requests.
- **Memory Profiler** – monitor memory usage (especially for memory leaks).
- **CPU Profiler** – analyze application performance in real time.

---

## Gradle – Build System

**Gradle** is a build automation system that Android Studio uses for:

- compiling applications,
- managing dependencies (e.g., Jetpack libraries),
- creating different versions (e.g., debug/release),
- running tests,
- configuring production and test environments.

### How Gradle Works

- Gradle analyzes configuration files (`build.gradle.kts`) and creates **task graph**.
- Executes tasks depending on goal (e.g., `build`, `assembleDebug`).
- Fetches dependencies from repositories such as **Google** or **Maven Central**.
- Modern Android Studio projects use **Kotlin DSL** (`.gradle.kts`) – configuration written in Kotlin language.

---

## Version Management: `libs.versions.toml`

Since Gradle 7.0, library and plugin versions can be centrally managed in `libs.versions.toml` file located in `gradle/` directory:

```
MyApp/
└── gradle/
    └── libs.versions.toml
```

File is divided into three sections:

```toml
[versions]
kotlin = "1.9.22"
compose = "1.6.4"
hilt = "2.51"

[libraries]
kotlin-stdlib = { module = "org.jetbrains.kotlin:kotlin-stdlib", version.ref = "kotlin" }
compose-ui = { module = "androidx.compose.ui:ui", version.ref = "compose" }
hilt-android = { module = "com.google.dagger:hilt-android", version.ref = "hilt" }

[plugins]
android-application = { id = "com.android.application", version = "8.4.0" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
```

**Advantages:**
- Central place for version management,
- no version duplication between modules,
- easy dependency updates.

---

## `build.gradle.kts` Files

Android project has two `build.gradle.kts` files:

- **`build.gradle.kts` (main project)** – configures common settings for all modules (e.g., repositories, plugin declarations).
- **`app/build.gradle.kts` (app module)** – configures specific application module (SDK, dependencies, build types).

### `build.gradle.kts` (Main Project)

```kotlin
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.android) apply false
}
```

`apply false` means plugin is only declared at project level (to establish its version), but is not yet activated – activation happens in module file (`app/build.gradle.kts`).

---

## Sections of `app/build.gradle.kts` File

File `app/build.gradle.kts` contains module application configuration. Below are key sections.

---

### `plugins`

Declares plugins needed for module build. Using `libs.versions.toml` reference looks as follows:

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
}
```

---

### `android`

Main section configuring Android module:

```kotlin
android {
    namespace = "com.example.myapp"   // namespace for R and BuildConfig classes
    compileSdk = 34

    defaultConfig {
        applicationId = "com.example.myapp"  // unique application identifier
        minSdk = 24                           // minimum Android version (API 24 = Android 7.0)
        targetSdk = 34                        // Android version application is optimized for
        versionCode = 1                       // integer, incremented with each release
        versionName = "1.0"                   // readable version shown to user
    }

    buildTypes {
        release {
            isMinifyEnabled = false  // Proguard/R8 – code compression and obfuscation
            proguardFiles(getDefaultProguardFile("proguard-android-optimize.txt"), "proguard-rules.pro")
        }
        debug {
            applicationIdSuffix = ".debug"  // debug version installed alongside release
            isDebuggable = true
        }
    }

    buildFeatures {
        compose = true      // enables Jetpack Compose
        buildConfig = true  // generates BuildConfig class with version and build type info
    }

    composeOptions {
        kotlinCompilerExtensionVersion = "1.6.4"
    }
}
```

#### Subsection Descriptions

| Section                 | Description |
|-------------------------|-------------|
| `namespace`             | Namespace for R and BuildConfig classes. |
| `compileSdk`            | SDK version the application is compiled with. |
| `defaultConfig`         | Default application settings (ID, version, SDK, tests). |
| `buildTypes`            | Defines build types (debug, release). By default two types exist: `debug` (for development) and `release` (production version). Custom types can be defined. |
| `buildFeatures`         | Enables selected code generation features: `compose = true` enables Jetpack Compose, `buildConfig = true` generates `BuildConfig` class available in application code (contains e.g., `BuildConfig.DEBUG`, `BuildConfig.VERSION_NAME`). |
| `composeOptions`        | Jetpack Compose-specific settings. |

Differences between build types:

| Feature                   | `debug`                                    | `release`                                      |
|---------------------------|--------------------------------------------|------------------------------------------------|
| Purpose                   | Development and testing                    | Distribution (Google Play, final APK)          |
| Debugging                 | Enabled (`isDebuggable = true`)           | Disabled                                       |
| Minification / obfuscation| Disabled                                   | Optional (`isMinifyEnabled`, Proguard/R8)      |
| Signing                   | Automatic test key                         | Requires custom key (`keystore`)               |
| Performance               | Slower (additional diagnostic tools)       | Optimized                                      |
| `applicationId`           | Can have suffix (`.debug`)                | Actual application ID                          |

---

### **Next topic:** [Composable Functions](ENt/03%20Composable%20Functions.md)
