# Background Tasks in Android Applications (Jetpack Compose and Coroutines)

In modern Android applications, including those built with Jetpack Compose, many operations (e.g., fetching network data, saving to database, heavy computations) should be executed **in background** to not block main UI thread.

## UI Thread (Main Thread) in Android

All operations related to displaying user interface (UI) in Android must be executed on **main thread** (called **UI thread** or **Main thread**). This means:

- View rendering, click handling, animations, and UI updates must happen on same thread.
- Executing long operation (e.g., fetching network data, database write) on main thread causes application to stop responding, which may lead to **ANR (Application Not Responding)** error.

---

## Why Background Work Is Important

- **User interface is not blocked** – application remains responsive.
- **Avoids ANR (Application Not Responding) errors** – system may close application if operations take too long on main thread.
- **Better user experience** – e.g., loading network data doesn't stop animations or scrolling.

---

## Coroutines

In Jetpack Compose, background work is most often realized using **coroutines** (Kotlin Coroutines):

- Coroutines allow easy launching of asynchronous tasks.
- They are lightweight and well-integrated with Compose architecture and ViewModel.

## Coroutine System Elements

### Suspend Functions

- Functions marked with `suspend` keyword can be called only from coroutine or another suspend function.
- Allow executing asynchronous operations sequentially without blocking thread.

**Example:**
```kotlin
suspend fun fetchDataFromNetwork(): String {
    // fetching data from network
    return "Data"
}
```

### Scope (Coroutine Scope)

- **Scope** manages coroutines – determines which part of application they're assigned to and when they should be cancelled.
- Most commonly used scopes in Compose:
  - **viewModelScope** – coroutines are tied to ViewModel and cancelled when `ViewModel` is destroyed (more on `ViewModel` in [Application Architecture](ENt/10%20Application%20Architecture.md) chapter).
  - **lifecycleScope** – coroutines are tied to component lifecycle (e.g., activity).
  - **rememberCoroutineScope** – scope tied to composable function, manages coroutines in its context.

### Dispatchers (Thread Selection)

- **Dispatchers** determine on which thread coroutine executes:
  - `Dispatchers.Main` – main UI thread (default in Compose and ViewModel).
  - `Dispatchers.IO` – thread for input/output operations (e.g., network, files, database).
  - `Dispatchers.Default` – thread for heavy computations.
- Can switch between dispatchers using `withContext`.

**Example with `withContext`:**
```kotlin
// Suspend function automatically switches context — caller doesn't need to do this
suspend fun loadAndProcess(): String {
    val raw = withContext(Dispatchers.IO) {
        fetchDataFromNetwork()     // network operation on IO thread
    }
    return withContext(Dispatchers.Default) {
        processData(raw)           // heavy computation on Default thread
    }
    // result returns to thread that called function (e.g., Main)
}
```

**Summary:**
- Suspend functions allow writing asynchronous code simply.
- Scope manages coroutine lifecycle and cancellation.
- Dispatchers select appropriate thread for task.

---

## Launching Coroutines

To execute background task, launch coroutine in appropriate scope. Most common ways:

### `launch`

- Most popular function for launching coroutines.
- Used for tasks that don't need to return result.
- Most often called in `viewModelScope`, `lifecycleScope`, or `rememberCoroutineScope`.

**Example in ViewModel:**
```kotlin
viewModelScope.launch {
    val data = fetchDataFromNetwork()
    // update UI state
}
```

**Example in composable with own scope:**
```kotlin
val coroutineScope = rememberCoroutineScope()
Button(onClick = {
    coroutineScope.launch {
        // background task, e.g., save to database
    }
}) {
    Text("Save")
}
```

### `async`

- Used when task should be launched in parallel and return result as `Deferred`.
- Allows executing multiple tasks in parallel and retrieve results using `await()`.
- For grouping parallel tasks inside suspend function recommend using `coroutineScope { }` – creates isolated scope that cancels all child tasks if one fails.

**Example:**
```kotlin
val coroutineScope = rememberCoroutineScope()
coroutineScope.launch {
    val deferred1 = async { fetchData1() }
    val deferred2 = async { fetchData2() }
    val result1 = deferred1.await()
    val result2 = deferred2.await()
    // use both results
}
```

**Example with `coroutineScope { }`:**
```kotlin
suspend fun fetchBothResults(): Pair<String, String> = coroutineScope {
    val deferred1 = async { fetchData1() }
    val deferred2 = async { fetchData2() }
    deferred1.await() to deferred2.await()
}
```

## Cancelling Coroutines

Coroutine launched by `launch` or `async` can be explicitly cancelled using `Job` object. Cancellation is **cooperative** – coroutine must itself check cancellation status (by calling suspend functions or explicitly checking `isActive`).

```kotlin
val job = coroutineScope.launch {
    fetchDataFromNetwork()
}
// explicit cancellation:
job.cancel()
```

In long-running loops or computations where no suspend functions are called, must manually check `isActive`:

```kotlin
coroutineScope.launch {
    for (item in largeList) {
        if (!isActive) break   // coroutine was cancelled – break loop
        process(item)
    }
}
```

> **`CancellationException`** – cancelling coroutine internally throws `CancellationException`. This is normal behavior, not application error. In `catch` block shouldn't suppress this exception:
> ```kotlin
> coroutineScope.launch {
>     try {
>         doLongWork()
>     } catch (e: Exception) {
>         if (e is CancellationException) throw e   // cancellation must propagate
>         handleError(e)
>     }
> }
> ```

---

## Side Effects

In Jetpack Compose **side effects** are operations executed outside composition process – e.g., launching coroutine, registering listener, logging, or navigation. Compose provides dedicated composable functions ensuring side effects execute at appropriate time and are properly cleaned up.

### `LaunchedEffect`

- Special composable function in Compose that launches coroutine on startup or after key change.
- Perfect for initialization or reacting to parameter changes.

**Example:**
```kotlin
@Composable
fun MyScreen() {
    LaunchedEffect(Unit) {
        loadData()
    }
    // UI...
}
```

`LaunchedEffect` launches coroutine when composable appears on screen **or** when key value changes. If `Unit` passed as key, effect executes only once on first display. If other value passed (e.g., ID, state), effect executes whenever that value changes.

**Example:**
```kotlin
@Composable
fun UserScreen(userId: String) {
    LaunchedEffect(userId) {
        loadUser(userId)
    }
    // UI...
}
```

In this example:
- If `userId` changes (e.g., user selects different profile), `LaunchedEffect` relaunches coroutine and loads new user data.
- If `Unit` used as key instead, effect would execute only once, regardless of `userId` changes.

---

> **`rememberCoroutineScope` vs `LaunchedEffect`**  
> `LaunchedEffect` launches coroutine automatically – on composable entry or key change. Used for effects that should execute "by themselves" (e.g., loading data on screen startup).  
> `rememberCoroutineScope` provides access to scope for manually launching coroutine – e.g., in response to button click. Scope is tied to composable lifecycle and cancelled when composable leaves composition.

### `DisposableEffect` – Side Effect with Resource Cleanup

Used for effects requiring explicit cleanup – e.g., registering and unregistering listener. `onDispose` block executes when composable leaves composition or key changes.

```kotlin
@Composable
fun LocationTracker(onLocationUpdate: (Location) -> Unit) {
    val context = LocalContext.current

    DisposableEffect(Unit) {
        val manager = context.getSystemService(LocationManager::class.java)
        val listener = LocationListener { location -> onLocationUpdate(location) }

        manager.requestLocationUpdates(LocationManager.GPS_PROVIDER, 0L, 0f, listener)

        onDispose {
            manager.removeUpdates(listener)  // cleanup on composition exit
        }
    }
}
```

### Connecting Effects to Lifecycle

Effects in Compose are tied to **composition** lifecycle, not activity lifecycle. This means:

| Event | `LaunchedEffect` / `DisposableEffect` |
|-------|---------------------------------------|
| Composable enters composition | Effect launched |
| Key changes | Previous effect cancelled/cleaned, new launched |
| Composable leaves composition | Effect cancelled / `onDispose` called |
| Screen rotation (Activity recreation) | Effect cancelled and relaunched |

When screen is covered by another (e.g., dialog), composable remains in composition – effects are **not** cancelled.

#### Observing Activity Lifecycle Events with `DisposableEffect`

When composable must react to activity lifecycle events (e.g., `onResume`, `onPause`), achievable with `DisposableEffect` and `LifecycleEventObserver`:

```kotlin
@Composable
fun LifecycleAwareScreen(onResume: () -> Unit, onPause: () -> Unit) {
    val lifecycleOwner = LocalLifecycleOwner.current

    DisposableEffect(lifecycleOwner) {
        val observer = LifecycleEventObserver { _, event ->
            when (event) {
                Lifecycle.Event.ON_RESUME -> onResume()
                Lifecycle.Event.ON_PAUSE  -> onPause()
                else -> {}
            }
        }

        lifecycleOwner.lifecycle.addObserver(observer)

        onDispose {
            lifecycleOwner.lifecycle.removeObserver(observer)
        }
    }
}
```

- `LocalLifecycleOwner.current` – provides `LifecycleOwner` (usually current activity) from composition tree.
- Observer is registered when composable enters composition and unregistered in `onDispose` – no memory leaks on rotation or navigation.

---

## Error Handling in Coroutines

Exceptions thrown inside coroutine don't automatically propagate to calling thread – must handle explicitly.

### `try/catch` – Recommended Approach

Simplest way: wrap coroutine code in `try/catch` block. Works well in ViewModel for UI state updates.

```kotlin
coroutineScope.launch {
    try {
        val result = fetchDataFromNetwork()  
    } catch (e: Exception) {
        // handle exception
    }
}
```

---

## Background Work and Long-Running Tasks

- For very long tasks (e.g., background synchronization, notifications) use **WorkManager** or **Foreground Service**.
- Coroutines ideal for short operations tied to screen lifecycle.

---

## Recommendations and Best Practices

- **Launch coroutines in appropriate scope**  
  Coroutines are automatically cancelled when component (ViewModel, composable function, activity) ceases to exist. Prevents memory leaks and unnecessary background task execution.

- **Choose appropriate dispatcher**  
  For I/O operations (network, files, database) use `Dispatchers.IO`, for heavy computations use `Dispatchers.Default`, for UI updates use `Dispatchers.Main`.

- **Don't block main thread**  
  Always execute long operations in background (e.g., using `withContext(Dispatchers.IO)`).

- **Use suspend functions for asynchronous operations**  
  Allows writing readable, sequential code without blocking threads.

- **Use `LaunchedEffect` for initialization and reacting to parameter changes in composable function**  
  Ensures task executes only when needed.

---

## More Information

- [Kotlin Coroutines – Android Developers](https://developer.android.com/kotlin/coroutines)
- [WorkManager – Android Developers](https://developer.android.com/topic/libraries/architecture/workmanager)
- [Background Work in Compose – Documentation](https://developer.android.com/jetpack/compose/side-effects#launchedeffect)

---

### **Next topic:** [Application Architecture](ENt/10%20Application%20Architecture.md)
