# AndroidManifest.xml and Permission System in Android

## AndroidManifest.xml – What Is It For?

File **AndroidManifest.xml** is one of most important files in every Android application. Located in project root directory (`app/src/main/AndroidManifest.xml`), serves several key functions:

- **Declares application components** – such as activities (`<activity>`), services (`<service>`), receivers (`<receiver>`), and content providers (`<provider>`).
- **Specifies permissions (permissions)** – which system resources application wants to access (e.g., Internet, camera, location).
- **Defines intent filters** – allowing application components to handle specific actions or data.
- **Establishes basic application information** – such as name, icon, version, package.
- **Declares libraries and special features** – e.g., camera support, Bluetooth, NFC.

> **Note:** Minimum and target system version (`minSdkVersion`, `targetSdkVersion`) are now declared in `build.gradle` file, not in AndroidManifest.xml.

> **Note (Android 12+):** Every activity with `<intent-filter>` must have explicitly set `android:exported` attribute. Value `true` means activity can be launched by other applications; `false` – only by same application. Missing attribute causes compilation error.
> ```xml
> <activity android:name=".MainActivity" android:exported="true">
> ```

**Sample AndroidManifest.xml Fragment:**
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myapp">

    <application
        android:label="My Application"
        android:icon="@mipmap/ic_launcher">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

    <!-- Permissions -->
    <uses-permission android:name="android.permission.INTERNET" />
</manifest>
```

---

## Application Class (`android:name`)

By default, Android creates basic `Application` object at process start. Attribute `android:name` in `<application>` element allows specifying custom class inheriting from `Application` — its `onCreate()` method is called before launching any activity.

Typical usage: library initialization (e.g., Firebase, Timber, Hilt), global configuration.

```kotlin
class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()
        // Initialization code, e.g.:
        // Firebase.initialize(this)
    }
}
```

```xml
<application
    android:name=".MyApp"
    android:label="My Application"
    android:icon="@mipmap/ic_launcher">
    ...
</application>
```

---

## Permission System in Android

For application to use certain system functions or user data (e.g., camera, location, contacts), must declare appropriate **permissions** in AndroidManifest.xml file.

### Declaring Permissions

Add appropriate `<uses-permission>` entry to manifest file:
```xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

### Normal and Dangerous Permissions

- **Normal permissions** – automatically granted during installation (e.g., Internet access).
- **Dangerous permissions** – require additional user consent during application runtime (e.g., location, contacts, SMS).

### Typical Dangerous Permissions

| Permission | Usage |
|-----------|-------|
| `CAMERA` | Access to camera |
| `RECORD_AUDIO` | Sound recording (microphone) |
| `ACCESS_FINE_LOCATION` | Precise GPS location |
| `ACCESS_COARSE_LOCATION` | Approximate location (network/Wi-Fi) |
| `READ_CONTACTS` | Read contact list |
| `READ_MEDIA_IMAGES` | Access to photos (Android 13+) |
| `READ_EXTERNAL_STORAGE` | File access (Android ≤ 12) |
| `POST_NOTIFICATIONS` | Display notifications (Android 13+) |

### Runtime Permission Request (Runtime Permissions)

Since Android 6.0 (API 23), user must grant dangerous permissions consent during application use.

**Modern Approach (Kotlin, Jetpack Activity Result API):**

Best to use modern `ActivityResultContracts` API, simpler and safer than old `onRequestPermissionsResult`. Same `rememberLauncherForActivityResult` mechanism discussed in previous chapter is used here, but with `RequestPermission()` contract instead of `StartActivityForResult()`.

### Checking Permission Status Before Asking

Before displaying request, worth checking whether permission wasn't already granted – avoids unnecessary dialogs:

```kotlin
val context = LocalContext.current
val isGranted = ContextCompat.checkSelfPermission(
    context,
    Manifest.permission.CAMERA
) == PackageManager.PERMISSION_GRANTED
```

If `isGranted` is `true`, no need to launch launcher.

### Justifying Permission Request

If user previously rejected permission, Android recommends explaining reason before asking again (`shouldShowRequestPermissionRationale`). Can check this flag in activity:

```kotlin
val activity = LocalContext.current as Activity
if (activity.shouldShowRequestPermissionRationale(Manifest.permission.CAMERA)) {
    // show explanation dialog, then launch the request
} else {
    launcher.launch(Manifest.permission.CAMERA)
}
```

### Permanent Permission Rejection

If user checked "Don't ask again", `shouldShowRequestPermissionRationale` returns `false`, and launcher doesn't show any dialog. Only option is redirecting to application settings:

```kotlin
val intent = Intent(
    Settings.ACTION_APPLICATION_DETAILS_SETTINGS,
    Uri.fromParts("package", context.packageName, null)
)
context.startActivity(intent)
```

In this case should inform user (e.g., through `Snackbar` or dialog) why permission is required and how to enable it manually.

**Example:**
```kotlin
// In composable or activity:
val launcher = rememberLauncherForActivityResult(
    contract = ActivityResultContracts.RequestPermission()
) { isGranted: Boolean ->
    if (isGranted) {
        // Permission granted
    } else {
        // Permission denied
    }
}

// Call permission request, e.g., after button click:
Button(onClick = {
    launcher.launch(Manifest.permission.CAMERA)
}) {
    Text("Request camera access")
}
```

**For Multiple Permissions:**
```kotlin
val launcher = rememberLauncherForActivityResult(
    contract = ActivityResultContracts.RequestMultiplePermissions()
) { permissions ->
    val cameraGranted = permissions[Manifest.permission.CAMERA] ?: false
    val locationGranted = permissions[Manifest.permission.ACCESS_FINE_LOCATION] ?: false
    // handle response
}

launcher.launch(arrayOf(
    Manifest.permission.CAMERA,
    Manifest.permission.ACCESS_FINE_LOCATION
))
```

---

## Declaring Required Hardware (`<uses-feature>`)

In addition to permissions, application can declare required hardware or device features. Unlike permissions, `<uses-feature>` informs Google Play which devices application should appear on:

```xml
<uses-feature android:name="android.hardware.camera" android:required="true" />
<uses-feature android:name="android.hardware.location.gps" android:required="false" />
```

- `required="true"` – application won't be installed on devices without this feature.
- `required="false"` – feature is optional; application should check its availability in code through `PackageManager.hasSystemFeature()`.

> **Note:** Some permissions (e.g., `CAMERA`) automatically imply corresponding `<uses-feature>`. Worth adding explicitly with `required="false"` if application can work without camera.

---

## Summary

- **AndroidManifest.xml** is central application configuration file – components, permissions, and other key information are declared here.
- From Android 12, activities with `<intent-filter>` require explicit `android:exported` attribute.
- **Normal permissions** granted automatically during installation; **dangerous permissions** require user consent during application runtime (from Android 6.0).
- Before displaying request, check permission status through `ContextCompat.checkSelfPermission`.
- Use `ActivityResultContracts.RequestPermission()` – same launcher mechanism as for receiving activity results.
- If user previously rejected permission, worth checking `shouldShowRequestPermissionRationale`; on permanent rejection – redirect to settings through `Settings.ACTION_APPLICATION_DETAILS_SETTINGS`.
- `<uses-feature>` declares required hardware and decides visibility on Google Play; add explicitly with `required="false"` if feature is optional.
- Custom `Application` class (attribute `android:name`) allows global library initialization before activity launch.

---

## More Information:

- [Permissions Overview – Android Developers](https://developer.android.com/guide/topics/permissions/overview)
- [Request App Permissions – Android Developers](https://developer.android.com/training/permissions/requesting)

---

### **Next topic:** [Background Tasks – Coroutines](/En/09%20Background%20Tasks%20-%20Coroutines.md)
