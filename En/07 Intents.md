# Intents in Android Applications

**Intents** are one of Android's fundamental communication mechanisms. They allow:

- launching new activities (screens) or services,
- passing data between application components,
- invoking system functions (e.g., opening browser, sending email),
- communication between different applications.

---

## Types of Intents

- **Explicit Intent** – specifies exactly which component (e.g., activity) should be launched. Used when target class name is known.
  ```kotlin
  val intent = Intent(context, DetailActivity::class.java)
  context.startActivity(intent)
  ```

- **Implicit Intent** – describes general action (e.g., "take a photo", "send email"), system Android chooses appropriate application or component that can handle this action.
  ```kotlin
  val intent = Intent(Intent.ACTION_VIEW, Uri.parse("https://android.com"))
  context.startActivity(intent)
  ```

---

## Android Components and Intents

Intents can be directed not only to activities, but to three types of Android components:

| Component | Description | How to Launch |
|-----------|-------------|----------------|
| **Activity** | Single screen with user interface | `startActivity(intent)` |
| **Service** | Background operation without UI (e.g., music playback, file download) | `startService(intent)` |
| **BroadcastReceiver** | Listens to system or application events (e.g., network change, battery charged, custom events) | `sendBroadcast(intent)` |

**Service** is useful when task should run independently of whether user sees screen – e.g., data synchronization in background. In modern applications, background tasks often use WorkManager or coroutines (discussed in later chapters).

**BroadcastReceiver** allows responding to events without direct user invocation. Android system sends many built-in broadcasts (e.g., `ACTION_BOOT_COMPLETED`, `ACTION_BATTERY_LOW`), and application can also define and send own.

---

## Typical System Actions

Android provides set of predefined actions for creating implicit intents:

| Action | Description | Example URI / extras |
|--------|-------------|----------------------|
| `ACTION_VIEW` | Display data (webpage, map, file) | `https://...`, `geo:52.4,16.9`, `tel:...` |
| `ACTION_DIAL` | Open dialer with number (without connection) | `tel:+48123456789` |
| `ACTION_CALL` | Direct connection (requires `CALL_PHONE` permission) | `tel:+48123456789` |
| `ACTION_SEND` | Share content with another application | extras: `EXTRA_TEXT`, `EXTRA_STREAM` |
| `ACTION_SENDTO` | Open email or SMS client | `mailto:address@email.com`, `smsto:number` |
| `ACTION_PICK` | Select item (gallery, contacts) | `MediaStore.Images.Media.EXTERNAL_CONTENT_URI` |

**Example – open location on map:**
```kotlin
val uri = Uri.parse("geo:52.4064,16.9252?q=Politechnika+Poznańska")
val intent = Intent(Intent.ACTION_VIEW, uri)
context.startActivity(intent)
```

**Example – send email:**
```kotlin
val intent = Intent(Intent.ACTION_SENDTO, Uri.parse("mailto:contact@example.com"))
intent.putExtra(Intent.EXTRA_SUBJECT, "Message Subject")
context.startActivity(intent)
```

---

## Passing Data with Intents

Intents allow not only launching other components, but also passing data to them. Most commonly **extras** are used – additional information attached to intent.

### Passing Data to Activity

**Adding data to intent:**
```kotlin
val intent = Intent(context, DetailActivity::class.java)
intent.putExtra("id", 123)
intent.putExtra("name", "Product X")
context.startActivity(intent)
```

**Receiving data in activity:**
```kotlin
class DetailActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val id = intent.getIntExtra("id", 0)
        val name = intent.getStringExtra("name")
        // ...data processing
    }
}
```

### Passing Data to Other Applications

Can pass data to other applications, e.g., sending text or image:

```kotlin
val intent = Intent(Intent.ACTION_SEND)
intent.type = "text/plain"
intent.putExtra(Intent.EXTRA_TEXT, "This is a message!")
context.startActivity(Intent.createChooser(intent, "Choose application"))
```

### Passing Complex Objects

For object to be passed, it must implement `Serializable` or `Parcelable` interface:

```kotlin
val intent = Intent(context, DetailActivity::class.java)
intent.putExtra("product", product) // where product : Parcelable
context.startActivity(intent)
```

In activity receive:
```kotlin
val product = intent.getParcelableExtra<Product>("product")
```

### Extras Keys – Best Practices

Keys passed to `putExtra` / `getExtra` are plain strings — typo or inconsistency between sending and receiving class causes data loss. Recommended: define keys as constants in target activity's `companion object`:

```kotlin
class DetailActivity : AppCompatActivity() {
    companion object {
        const val ID_KEY = "id"
        const val NAME_KEY = "name"
    }
    // ...
}
```

Both sides then reference same constant:

```kotlin
// Sending:
intent.putExtra(DetailActivity.ID_KEY, 123)
intent.putExtra(DetailActivity.NAME_KEY, "Product X")

// Receiving (in DetailActivity):
val id = intent.getIntExtra(ID_KEY, 0)
val name = intent.getStringExtra(NAME_KEY)
```

### **Summary:**  
- Use `putExtra` and corresponding receive methods (`getIntExtra`, `getStringExtra`, etc.) for passing simple data.
- Define extras keys as constants in target activity's `companion object` – prevents typo-related bugs.
- Use `Parcelable` or `Serializable` for passing objects.
- Passing data through intents works both within application and between different applications.

---

## Receiving Result from Activity

For receiving results from launched activity use `ActivityResultLauncher` with `registerForActivityResult`. This is modern API that replaced deprecated `startActivityForResult`.

**Launcher registers outside event handler** (e.g., outside `onClick`) because it must be initialized before composable runs:

```kotlin
// Select photo from gallery — ready-made contract
val launcher = rememberLauncherForActivityResult(
    contract = ActivityResultContracts.GetContent()
) { uri ->
    // uri — selected file or null if cancelled
    if (uri != null) {
        // handle selected file
    }
}

Button(onClick = { launcher.launch("image/*") }) {
    Text("Select photo")
}
```

For launching any activity and receiving result use `StartActivityForResult` contract:

```kotlin
val launcher = rememberLauncherForActivityResult(
    contract = ActivityResultContracts.StartActivityForResult()
) { result ->
    if (result.resultCode == Activity.RESULT_OK) {
        val data = result.data?.getStringExtra("result")
    }
}

Button(onClick = {
    val intent = Intent(context, SelectionActivity::class.java)
    launcher.launch(intent)
}) {
    Text("Open selection screen")
}
```

Activity launched by launcher ends with `setResult` and `finish`:

```kotlin
// In launched activity:
val resultIntent = Intent().putExtra("result", "selectedValue")
setResult(Activity.RESULT_OK, resultIntent)
finish()
```

**Built-in contracts** (`ActivityResultContracts.*`) simplify typical scenarios:

| Contract | Usage |
|----------|-------|
| `GetContent()` | Select file or image from gallery |
| `TakePicture()` | Take photo with camera |
| `RequestPermission()` | Request single permission |
| `RequestMultiplePermissions()` | Request multiple permissions at once |
| `StartActivityForResult()` | Launch any activity |

---

## Intent Filters System

**Intent Filter** is mechanism allowing application to declare what actions, data, or categories it can handle. Defined in `AndroidManifest.xml`, allows Android system to choose appropriate application or component to handle given intent.

**How Does It Work?**
- When implicit intent is sent, system searches all installed applications and their intent filters.
- If finds component (e.g., activity) matching action, data type, or category, shows user selection list or launches appropriate application.

**Example Intent Filter in AndroidManifest.xml:**
```xml
<activity android:name=".DetailsActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:scheme="https" android:host="android.com" />
    </intent-filter>
</activity>
```

**What Can Be Filtered?**
- **Actions** (e.g., `android.intent.action.SEND`)
- **Categories** (e.g., `android.intent.category.DEFAULT`)
- **Data types** (e.g., `image/*`, `text/plain`)
- **URI schemes and hosts** (e.g., `https://android.com`)

---

## Intent Usage Example

**Launch new activity with data passing:**
```kotlin
val intent = Intent(context, DetailsActivity::class.java)
intent.putExtra("id", 123)
context.startActivity(intent)
```

**Invoke system action (e.g., open webpage):**
```kotlin
val intent = Intent(Intent.ACTION_VIEW, Uri.parse("https://android.com"))
context.startActivity(intent)
```

---

## Intents and Navigation Compose

In traditional Android applications, screen transitions (between activities) are done using intents. In Jetpack Compose, instead of intents, **Navigation Compose** and `NavController` are used **within single activity**.

- **Navigation Compose** manages transitions between composables (screens) within single activity.
- **Intents** are still used for:
  - launching other activities (e.g., navigate to login screen in different application),
  - invoking system functions,
  - communication between applications.

**Example: navigate to another activity from composable:**
```kotlin
val context = LocalContext.current
Button(onClick = {
    val intent = Intent(context, DetailsActivity::class.java)
    intent.putExtra("id", 123)
    context.startActivity(intent)
}) {
    Text("Open details (new activity)")
}
```

**Example: navigation in Compose (without intents):**
```kotlin
Button(onClick = { navController.navigate("details/123") }) {
    Text("Go to details (in Compose)")
}
```

---

## PendingIntent

`PendingIntent` is wrapper around regular intent, allowing another system component (e.g., notification, widget, alarm) to execute that intent **on behalf of application** — even when application isn't running.

```kotlin
val intent = Intent(context, MainActivity::class.java)
val pendingIntent = PendingIntent.getActivity(
    context,
    0,
    intent,
    PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
)
```

`PendingIntent` is required for:
- **Notifications** — action after notification click,
- **Home screen widgets** — handling click,
- **Alarms** (`AlarmManager`) — run code at scheduled time.

> Always set `FLAG_IMMUTABLE` flag when intent content doesn't need modification by external component (required from Android 12).

---

## Summary

- **Explicit intents** target specific component; **implicit intents** describe action and let system choose appropriate application.
- **Extras** (`putExtra`/`getExtra`) pass data between activities; define keys as constants in target activity's `companion object`.
- **`ActivityResultLauncher`** with built-in contracts (`GetContent`, `TakePicture`, `RequestPermission`, etc.) receives results from activity.
- **Intent Filters** in `AndroidManifest.xml` declare what actions and data given activity can handle.
- **`PendingIntent`** enables execution of intent by another system component (notification, widget, alarm) on behalf of application.
- **Navigation Compose** handles navigation within single activity; intents remain necessary for communication with other applications and system.

---

### **Next topic:** [Manifest and Permissions](/En/08%20Manifest%20and%20Permissions.md)
