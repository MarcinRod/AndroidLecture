# Navigation Component in Jetpack Compose

Jetpack Compose offers a modern way to manage navigation in Android applications through **Navigation Compose** library. It allows easy screen transitions (called composable destinations) and passing data between them.

---

## Navigation Back Stack in Navigation Compose

Navigation Compose manages the **navigation stack** (back stack) – list of screens visited by user. This enables easy handling of returning to previous screens and controlling navigation history.

### How Does the Stack Work?

- Each navigation to new screen adds new destination on top of stack.
- Return removes last screen from stack and shows previous one.
- Can remove multiple screens at once or clear stack to selected destination.

> **Tip:**  
> Understanding stack operation is crucial for proper navigation handling and predictable "back" button behavior in application.

---

## Basic Navigation Elements

- **NavGraph**  
  Directed graph representing application navigation structure – all available screens (vertices) and possible transitions between them. Defined declaratively by adding destinations (`composable { }`) and subgraphs (`navigation { }`).

- **NavHost**  
  Composable function responsible for displaying currently active destination. Requires specifying start destination and navigation graph definition.

  **Example:**
  ```kotlin
  NavHost(navController = navController, startDestination = "screenA") {
      composable("screenA") { ScreenA() }
      composable("screenB") { ScreenB() }
  }
  ```

- **NavController**  
  Object controlling navigation – manages screen stack, enables navigating to new destinations and returning to previous ones. Created with `rememberNavController()` function.

  **Example:**
  ```kotlin
  val navController = rememberNavController()
  ```

- **Composable destinations**  

  Individual screens registered in navigation graph. Each destination is identified by unique key and associated with specific composable function.

  **Example:**
  ```kotlin
  composable("screenA") { ScreenA() }
  composable("screenB") { ScreenB() }
  ```

**Summary:**  
`NavGraph` defines available screens and relationships, `NavHost` displays it, `NavController` controls transitions, and composable destinations are individual screens registered in graph. This makes Compose navigation transparent, declarative, and easy to expand.

---

## Basic Configuration Example

1. **Add dependency in `build.gradle.kts`:**
   ```kotlin
   implementation("androidx.navigation:navigation-compose:2.7.7")
   ```

2. **Create NavController and NavHost:**
   ```kotlin
   val navController = rememberNavController()
   NavHost(navController = navController, startDestination = "screenA") {
       composable("screenA") { ScreenA() }
       composable("screenB") { ScreenB() }
   }
   ```

3. **Navigate between screens:**
   ```kotlin
   NavHost(navController = navController, startDestination = "screenA") {
       composable("screenA") {
           ScreenA(onNavigate = { navController.navigate("screenB") })
       }
       composable("screenB") { ScreenB() }
   }
   ```

---

## Navigating Between Screens – Basic NavController Methods

For navigation between screens in Jetpack Compose, `NavController` object is used. Here are most important methods:

### 1. Navigate to Another Screen

To navigate to another destination use `navigate` method:

```kotlin
navController.navigate("screenB")
```

### 2. Return to Previous Screen

To return to previous screen (previous destination on stack) use:

```kotlin
navController.popBackStack()
```

### 3. Return to Specific Destination

Can return to specific screen (e.g., main screen), removing all screens above:

```kotlin
navController.popBackStack("screenA", inclusive = false)
```

- If `inclusive = true`, screen named `"screenA"` is also removed from stack.

### 4. Prevent Screen Duplication (singleTop)

To avoid duplicating screens on stack, use `launchSingleTop` option:

```kotlin
navController.navigate("screenA") {
    launchSingleTop = true
}
```

### 5. Navigate with Stack Clearing (e.g., After Login)

To navigate and remove all previous screens (e.g., after login):

```kotlin
navController.navigate("homeScreen") {
    popUpTo("loginScreen") { inclusive = true }
}
```

These methods allow full management of user flow in Compose application.

### Good Practice: Where to Keep NavController?

Important rule in Compose navigation: **`NavController` should be kept only at the highest level of composable tree**, and only navigation lambdas should be passed to screens.

```kotlin
// Recommended — screen doesn't know NavController
@Composable
fun LoginScreen(onLoginSuccess: () -> Unit) {
    Button(onClick = onLoginSuccess) { Text("Login") }
}

// In NavHost (only place where NavController is available):
composable("LoginScreen") {
    LoginScreen(
        onLoginSuccess = {
            navController.navigate("screenA") {
                popUpTo("loginScreen") { inclusive = true }
            }
        }
    )
}
```

Passing `navController` directly to screens makes testing harder, prevents screen reuse, and creates unnecessary dependencies.

**Summary:**  
- `navigate(route)` – navigate to screen
- `popBackStack()` – return to previous screen
- `popBackStack(route, inclusive)` – return to specific destination
- Navigation options (`launchSingleTop`, `popUpTo`) allow controlling navigation stack

---

## Defining Navigation Destinations

### Recommended: Type-Safe Navigation (Navigation 2.8+)

Since Navigation 2.8, **type-safe navigation** using `kotlinx-serialization` is available. Destinations are often defined in dedicated interface (`sealed interface`) as `@Serializable data object` or `@Serializable data class` – arguments are regular class fields, without manual route templates and `navArgument`.

**Additional dependencies in `build.gradle.kts`:**
```kotlin
plugins {
    kotlin("plugin.serialization") version "2.0.0"
}

dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.7.3")
    implementation("androidx.navigation:navigation-compose:2.8.0")
}
```

**Defining destinations:**
```kotlin
sealed interface Destinations {
    @Serializable
    data object ListSelection : Destinations

    @Serializable
    data class ShoppingList(val listId: String? = null) : Destinations
}
```

**NavHost Configuration and Navigation:**
```kotlin
NavHost(navController, startDestination = Destinations.ListSelection) {
    composable<Destinations.ListSelection> {
        ListSelectionScreen(
            onListSelected = { id ->
                navController.navigate(Destinations.ShoppingList(id))
            }
        )
    }
    composable<Destinations.ShoppingList> { backStackEntry ->
        val destination = backStackEntry.toRoute<Destinations.ShoppingList>()
        val listId = destination.listId ?: return@composable
        ShoppingListScreen(listId = listId)
    }
}
```

---

### Traditional Approach (String-Based Routes)

In older projects and examples, two approaches are common:

**1. Enum class** — simple iteration, uniform structure:

```kotlin
enum class Destinations(val route: String) {
    ScreenA("screenA"),
    ScreenB("screenB"),
    Details("details/{id}")
}
```

*Advantage:* access all destinations through `Destinations.entries` – useful for building `BottomBar`.  
*Disadvantage:* each entry must fit one constructor; handling arguments requires method overriding.

**2. Sealed class** — more flexible, recommended when screens have different parameters:

```kotlin
sealed class Destinations(val baseRoute: String) {
    object ScreenA : Destinations("screenA")
    object ScreenB : Destinations("screenB")
    object Details : Destinations("details") {
        val routeTemplate = "details/{id}"
        fun createRoute(id: String) = "details/$id"
        const val ARG_ID = "id"
    }
}
```

*Advantage:* individual destinations can have varied structure; `when` exhaustiveness forces new screen handling.  
*Disadvantage:* need manual `routeTemplate`, `navArgument`, and `createRoute` definition.

### Passing Arguments (Traditional Approach)

In type-safe approach arguments are `data class` fields – no need for `navArgument` or route templates. Following description applies to traditional (string-based) approach still found in older projects.

Data between screens can be passed two ways:

#### 1. Path Arguments

Arguments passed in path are part of route, e.g., `"details/{id}"`. Are required and must always be provided during navigation.

**Sealed class Definition:**
```kotlin
sealed class Destinations(val route: String) {
    object ScreenA : Destinations("screenA")
    object Details : Destinations("details/{id}") {
        fun createRoute(id: String) = "details/$id"
        const val ARG_ID = "id"
    }
}
```

**NavHost Configuration:**
```kotlin
NavHost(navController, startDestination = Destinations.ScreenA.route) {
    composable(Destinations.ScreenA.route) { ScreenA() }
    composable(
        route = Destinations.Details.route,
        arguments = listOf(navArgument(Destinations.Details.ARG_ID) { type = NavType.StringType })
    ) { backStackEntry ->
        val id = backStackEntry.arguments?.getString(Destinations.Details.ARG_ID) ?: ""
        DetailsScreen(id)
    }
}

// Navigate with argument:
navController.navigate(Destinations.Details.createRoute("123"))
```

#### 2. Query Arguments (Optional, After `?`)

Query arguments are passed after question mark in path, e.g., `"screenB?name={name}"`. Can be optional with default values.

**Sealed class Definition:**
```kotlin
sealed class Destinations(val route: String) {
    object ScreenA : Destinations("screenA")
    object ScreenB : Destinations("screenB?name={name}") {
        fun createRoute(name: String?) =
            if (name != null) "screenB?name=$name" else "screenB"
        const val ARG_NAME = "name"
    }
}
```

**NavHost Configuration:**
```kotlin
NavHost(navController, startDestination = Destinations.ScreenA.route) {
    composable(Destinations.ScreenA.route) { ScreenA() }
    composable(
        route = Destinations.ScreenB.route,
        arguments = listOf(navArgument(Destinations.ScreenB.ARG_NAME) {
            type = NavType.StringType
            defaultValue = "Guest"
            nullable = true
        })
    ) { backStackEntry ->
        val name = backStackEntry.arguments?.getString(Destinations.ScreenB.ARG_NAME)
        ScreenB(name)
    }
}

// Navigate without argument:
navController.navigate(Destinations.ScreenB.createRoute(null))
// Navigate with argument:
navController.navigate(Destinations.ScreenB.createRoute("Anna"))
```

### Approaches Comparison:

| | Type-safe (recommended) | Traditional (string-based) |
|---|---|---|
| Destination Definition | `@Serializable data object/class` | `sealed class` or `enum` with `val route: String` |
| NavHost Registration | `composable<Type>` | `composable("route")` |
| Navigation with Argument | `navigate(Dest.Screen(arg))` | `navigate("route/$arg")` |
| Argument Reading | `backStackEntry.toRoute<T>()` | `backStackEntry.arguments?.getString(key)` |
| Type Safety | Complete | Partial (typos in strings possible) |

---

## Nested Navigation Graphs

In larger applications, Jetpack Compose often uses **nested navigation graphs** to better organize routes and manage complex screen flows. Allows dividing application into logical modules (e.g., login, main screen, settings), each with own subgraph.

### How to Define Nested Graph?

In type-safe approach, each subgraph is identified by destination object marked `@Serializable`. `navigation<T>` function accepts graph-route object type instead of string:

**Destination Definition:**
```kotlin
// Subgraph key objects (not screens — only identifiers)
@Serializable data object AuthGraph
@Serializable data object MainGraph

// Auth subgraph screens
@Serializable data object Login
@Serializable data object Register

// Main subgraph screens
@Serializable data object Home
@Serializable data class Settings(val userId: String)
```

Subgraph keys (`AuthGraph`, `MainGraph`) are not screens – only identify destination group. Screens can be simple `data object` (no arguments) or `data class` (arguments as fields).

**NavHost Configuration:**
```kotlin
NavHost(navController, startDestination = AuthGraph) {
    // Auth subgraph
    navigation<AuthGraph>(startDestination = Login) {
        composable<Login> {
            LoginScreen(onLoginSuccess = {
                navController.navigate(Home) {
                    popUpTo(AuthGraph) { inclusive = true }
                }
            })
        }
        composable<Register> { RegisterScreen() }
    }
    // Main application subgraph
    navigation<MainGraph>(startDestination = Home) {
        composable<Home> { HomeScreen() }
        composable<Settings> { backStackEntry ->
            val dest = backStackEntry.toRoute<Settings>()
            SettingsScreen(userId = dest.userId)
        }
    }
}
```

### Navigating Between Subgraphs

Navigation to screen in different subgraph is identical to regular navigation – through destination object. No need for full paths:

```kotlin
// After login: navigate to main graph and clear login stack
navController.navigate(Home) {
    popUpTo(AuthGraph) { inclusive = true }
}
```

### Nested Graphs Advantages

- **Better code organization** – each module has own, readable subgraph.
- **Easier flow management** – login stack can be cleared after login via `popUpTo(AuthGraph)` and navigate to main graph.
- **Reusability** – same settings subgraph can be embedded in different places in application.

**More about Nested Graphs:**  
[Official Documentation – Nested Navigation Graphs](https://developer.android.com/jetpack/compose/navigation#nested-nav)

---

## Deep Links

**Deep link** is mechanism allowing opening specific application screen from outside – e.g., from notification, widget, system shortcut, or another application. Registered directly at destination in `NavHost`, by providing URI pattern:

```kotlin
composable<Destinations.ShoppingList>(
    deepLinks = listOf(navDeepLink { uriPattern = "myapp://list/{listId}" })
) { backStackEntry ->
    val dest = backStackEntry.toRoute<Destinations.ShoppingList>()
    ShoppingListScreen(listId = dest.listId ?: return@composable)
}
```

For system Android to redirect to application after link click, appropriate entry in `AndroidManifest.xml` is required:

```xml
<activity android:name=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="myapp" android:host="list" />
    </intent-filter>
</activity>
```

> Practical deep link usage (e.g., opening screens from notifications) is shown in sample application about architecture: [ReminderShowcase](https://github.com/MarcinRod/ReminderShowcase).

---

## Navigation Compose Advantages

- **Declarative Route and Screen Management**  
  All routes and screens defined in one place, easing reading and code maintenance.

- **Type-safe Navigation (Navigation 2.8+)**  
  Destinations as `@Serializable data object/class` eliminate route name typos and manual `navArgument` definition. Arguments are regular class fields — read through `toRoute<T>()`.

- **Argument Passing in Traditional Approach**  
  In older string-based API, arguments can be passed in path (required) or as query (optional), with type enforcement through `navArgument` and default values.

- **Navigation Stack Handling**  
  Navigation Compose automatically manages screen history (back stack), enabling intuitive "back" button use and user flow control.

- **Animation and Navigation Element Integration**  
  Navigation can be combined with transition animations, bottom navigation bar (`BottomNavigation`), tabs (`Tabs`), or side drawer (`Drawer`).

- **Nested Graphs Support**  
  Applications can be organized into logical modules and manage complex screen flows through nested navigation graphs.

- **Architecture Integration**  
  Navigation Compose integrates well with ViewModels, State Hoisting, and other Compose architectural patterns.

---

Navigation Compose is modern and recommended way to handle navigation in Jetpack Compose applications. Since version 2.8, type-safe navigation eliminates manual route string management.

---

## Documentation

- [Official Navigation Compose Documentation](https://developer.android.com/jetpack/compose/navigation)
- [Type-safe Navigation – Migration Guide](https://developer.android.com/guide/navigation/design/type-safety)
- [Examples on GitHub](https://github.com/android/compose-samples/tree/main/NavigationAdvancedSample)

---

### **Next topic:** [Intents](En/07%20Intents.md)
