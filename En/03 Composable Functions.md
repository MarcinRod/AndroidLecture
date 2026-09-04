# Composable Functions in Jetpack Compose

## What Are Composable Functions?

**Composable functions** are the fundamental building block of user interface in Jetpack Compose. They are Kotlin functions marked with the `@Composable` annotation, which describe how a UI fragment should look.

---

## Naming Composable Functions

- Composable functions should have names in **PascalCase** style (each word capitalized, no underscores), e.g., `UserCard`, `LoginScreen`, `Greeting`.
- Function name should clearly indicate **what the function displays or implements** – avoid generic names like `MyComposable` or `TestFunction`.
- If composable function represents an entire screen, its name should end with `Screen`, e.g., `ProfileScreen`, `SettingsScreen`.
- For smaller, reusable UI elements, descriptive names indicating their role are recommended, e.g., `UserAvatar`, `ProductItem`, `ErrorMessage`.
- If function is **private to file** (shouldn't be used outside the file), prefix `_` can be added, e.g., `_UserAvatar`. This is a Compose convention signaling the function is "internal".
- Abbreviations and unreadable names should be avoided – Compose code should be self-documenting.
- Composable function names should not contain imperative verbs (e.g., `ShowUser`), but rather nouns or nouns with adjectives (`UserCard`, `ErrorDialog`).

**Examples of good naming:**
- `LoginScreen`
- `ProfileHeader`
- `ProductListItem`
- `_UserAvatar` (private function)
- `ErrorSnackbar`

**Examples of poor naming:**
- `doLoginComposable`
- `myComposable1`
- `ShowProfile`
- `testFun`

Good naming facilitates code reading, testing, and reuse in larger Compose projects.

---

## The Role of Modifier

- **Modifier** is a special object in Compose that allows modifying the appearance, size, position, and behavior of UI elements.
- Modifier is passed as a parameter to most composable functions and enables "layering" multiple effects in a chain of calls.
- Each composable function accepting `Modifier` should have it as the first default parameter, e.g.:
  ```kotlin
  @Composable
  fun MyButton(
      onClick: () -> Unit,
      modifier: Modifier = Modifier
  ) {
      Button(onClick = onClick, modifier = modifier) {
          Text("Click me")
      }
  }
  ```
- This allows easy combining of modifiers and passing them from outside, e.g.:
  ```kotlin
  MyButton(
      onClick = { /* ... */ },
      modifier = Modifier
          .padding(16.dp)
          .fillMaxWidth()
  )
  ```

### Most Commonly Used Modifier Functions

- `padding(...)` – adds internal spacing around element.
- `fillMaxWidth()`, `fillMaxHeight()`, `fillMaxSize()` – stretches element to maximum width/height/size of parent.
- `size(...)`, `width(...)`, `height(...)` – sets element size.
- `background(color)` – sets element background.
- `clickable { ... }` – handles clicks on any element.
- `weight(...)` – distributes space in Row/Column proportionally.
- `offset(...)` – offsets element relative to its position.
- `align(...)` – sets alignment in container (e.g., in Box).
- `clip(shape)` – clips drawing area to given shape (e.g., `CircleShape`, `RoundedCornerShape`). Apply before `background()` so fill respects shape.
- `border(...)` – draws outline around element.
- `wrapContentWidth()`, `wrapContentHeight()` – adapts size to content.
- `verticalScroll(rememberScrollState())` – enables vertical content scrolling (applies to `Column` or other container).

**Example of combining modifiers:**
```kotlin
Text(
    text = "Example",
    modifier = Modifier
        .padding(8.dp)
        .background(Color.LightGray)
        .fillMaxWidth()
        .clickable { /* handle click */ }
)
```

> **Tip — modifier order matters:**
>
> ```kotlin
> // A: padding → background — padding area is NOT colored
> Text("A", modifier = Modifier.padding(12.dp).background(Color.Yellow))
>
> // B: background → padding — background covers padding area
> Text("B", modifier = Modifier.background(Color.Yellow).padding(12.dp))
> ```
> Both lines produce different visual results. In case A padding is "outside" background; in case B background includes padding area.

Modifiers allow building flexible, responsive, and interactive UI in Compose without inheriting from views or using complex layouts.

---

## State, Composition, and Recomposition

### Composition

**Composition** is the process where Compose runtime calls functions marked `@Composable` and builds UI element tree based on them. Composition happens only once during initial screen rendering.

### State

**State** is any value whose change can trigger UI update. Compose automatically tracks which composable functions read given state and re-calls them whenever that state changes.

#### `mutableStateOf`

Creates an object holding a value whose changes are automatically tracked by Compose. Each value modification triggers recomposition of UI fragments reading that value.

```kotlin
var count = mutableStateOf(0)   // without remember — value resets on recomposition
```

#### `remember`

Holds value **within composition** — value survives subsequent recompositions but is lost when composable function is destroyed (e.g., screen change).

```kotlin
var count by remember { mutableStateOf(0) }   // value survives recomposition
```

Both mechanisms are used together: `remember { mutableStateOf(...) }`.

> **Delegation by `by`**  
> Kotlin provides delegation operator `by` allowing direct state value reading and writing – without manual `.value` calls:
> ```kotlin
> // without by — access through .value
> val count = remember { mutableStateOf(0) }
> count.value++
> Text("${count.value}")
>
> // with by — direct access
> var count by remember { mutableStateOf(0) }
> count++
> Text("$count")
> ```
> Using `by` requires import `getValue`/`setValue` from `androidx.compose.runtime` package.

#### `rememberSaveable`

Works like `remember`, but additionally saves value to `Bundle` — state survives **configuration change** (e.g., screen rotation). Discussed in detail in Activity chapter.

```kotlin
var text by rememberSaveable { mutableStateOf("") }
```

### Recomposition

**Recomposition** is re-calling composable functions in response to state change. Compose identifies which composable functions depend on changed state and redraws **only them** – not entire screen.

```
State change → Compose detects dependencies → recomposition of minimal UI subset
```

Key recomposition properties:
- Is **selective** — affects only composable functions reading changed state.
- Can occur **multiple times** — composable functions should only describe UI appearance based on passed data, without performing operations like database writes or network calls. Such operations can be called multiple times in unpredictable order.
- Compose can **skip** recomposition of function if its parameters haven't changed (*smart recomposition*).

### Side Effects

When operations outside composition are necessary (e.g., API calls) dedicated effect functions like `LaunchedEffect` or `DisposableEffect` should be used. They guarantee safe operation execution at appropriate composition lifecycle moment.

> Side effects are discussed in detail in coroutines chapter: [Background Tasks – Coroutines](En/09%20Background%20Tasks%20-%20Coroutines.md).

### Data Flow — State Down, Events Up

Standard Compose data flow pattern: state is passed **down** through composable function parameters, events (callbacks) propagate **up** to state owner. This is foundation of *state hoisting* pattern described in next section.

```
State owner (e.g., Screen)
  │  state (down)
  ▼
ComposableChild(value, onValueChange)
                       │  event (up)
                       ▲
```

---

## State Hoisting

**State hoisting** is a pattern where state and state-change functions are passed from parent composable to child, instead of child managing state internally.

### How It Works in Practice?

Instead of:

```kotlin
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }
    Button(onClick = { count++ }) {
        Text("Clicked: $count")
    }
}
```

We apply state hoisting:

```kotlin
@Composable
fun Counter(count: Int, onIncrement: () -> Unit) {
    Button(onClick = onIncrement) {
        Text("Clicked: $count")
    }
}

@Composable
fun CounterScreen() {
    var count by remember { mutableStateOf(0) }
    Counter(count = count, onIncrement = { count++ })
}
```

Function `Counter` no longer manages its own state — receives it as parameter and accepts function for its change. Parent function (`CounterScreen`) is state owner.

### Stateful vs Stateless

Composable function managing its own internal state is **stateful composable**. Function receiving state and callbacks as parameters and maintaining no internal state is **stateless composable**.

```kotlin
// Stateful — internal state, less flexible
@Composable
fun CounterStateful() {
    var count by remember { mutableStateOf(0) }
    Button(onClick = { count++ }) { Text("$count") }
}

// Stateless — state from outside, full flexibility
@Composable
fun CounterStateless(count: Int, onIncrement: () -> Unit) {
    Button(onClick = onIncrement) { Text("$count") }
}
```

### State Hoisting Advantages

- **Reusability:** Same composable can be used in different places with different state.
- **Testability:** Easier to test composables without internal state.
- **Predictability:** State managed in one place, easing debugging and maintenance.
- **ViewModel Integration:** Hoisted state easily integrates with ViewModel or other data source.

### When NOT to Hoist State?

If state is **purely local** and doesn't affect other UI elements, it can be kept inside composable function (e.g., menu expansion, local animation effect).

---

> **More about state hoisting:**  
> [State hoisting – Official Guide](https://developer.android.com/jetpack/compose/state#hoisting)

---

## Material Design and `MaterialTheme`

**Material Design** is a UI design system developed by Google. Defines set of principles, components, and guidelines for UI appearance, animations, and behavior — so applications across different platforms are consistent, readable, and accessible.

Current version is **Material Design 3** (Material You), introducing dynamic colors adapting to user wallpaper (Android 12+) and updated components and typography.

### `MaterialTheme`

In Jetpack Compose, Material Design system is available through `MaterialTheme` object providing three main design resources:

| Resource | Description | Usage Example |
|---|---|---|
| `colorScheme` | Application color palette (background, surfaces, accents, errors) | `MaterialTheme.colorScheme.primary` |
| `typography` | Set of text styles (headings, body, labels) | `MaterialTheme.typography.bodyLarge` |
| `shapes` | Set of shapes (small, medium, large corner radius) | `MaterialTheme.shapes.medium` |

Components like `Button`, `Card`, `Surface`, or `Scaffold` automatically use these resources — changing application theme (e.g., dark mode, brand colors) automatically updates all component appearance.

### How Is Theme Defined?

Application theme is typically defined in `Theme.kt` file and wrapped around entire `MainActivity` content:

```kotlin
@Composable
fun MyAppTheme(content: @Composable () -> Unit) {
    MaterialTheme(
        colorScheme = lightColorScheme(
            primary = Color(0xFF6750A4),
            secondary = Color(0xFF625B71)
        ),
        typography = Typography(),
        content = content
    )
}
```

### Dark Mode

Theme can respond to system dark mode settings:

```kotlin
val darkTheme = isSystemInDarkTheme()
val colorScheme = if (darkTheme) darkColorScheme(...) else lightColorScheme(...)
```

> Details of creating custom theme are beyond scope of this chapter. Android Studio generates default theme when creating Compose project.

---

## Dimension Units: dp and sp

Android supports several length units, but in practice two are used:

| Unit | Name | Usage |
|------|------|-------|
| `px` | Physical pixels | Should not be used — appearance depends on screen density |
| `dp` | Density-independent pixels | UI element dimensions (margins, sizes, padding) |
| `sp` | Scale-independent pixels | Text size — additionally accounts for user font preferences |

**Practical rule:** use `dp` for all dimensions, `sp` for text size.

### What Is Density Independence?

Different devices have different screen density (DPI — dots per inch). Element of `100px` size will look different on 160 dpi (large) and 480 dpi (small — approx. 3× smaller physically).

`1 dp` is defined as one pixel on reference screen of **160 dpi** (mdpi). On higher density screens Android automatically converts:

$$\text{pixels} = dp \times \frac{DPI}{160}$$

| Density | Name | Multiplier | `16 dp` in pixels |
|---------|------|---------|---------------------|
| 160 dpi | mdpi | ×1 | 16 px |
| 240 dpi | hdpi | ×1.5 | 24 px |
| 320 dpi | xhdpi | ×2 | 32 px |
| 480 dpi | xxhdpi | ×3 | 48 px |
| 640 dpi | xxxhdpi | ×4 | 64 px |

Thanks to this, element of `48 dp` width occupies similar **physical size** (approx. 7.6 mm) regardless of device. Conversion is automatic — code always specifies value in `dp`.

In Compose values are specified directly in code with extension:

```kotlin
Modifier.padding(16.dp)
Text(text = "Hello", fontSize = 18.sp)
```

---

## User Interface Elements

Jetpack Compose provides rich set of ready-made composable functions for building user interfaces. Complete list of available Material3 components with interactive demo available at:
- [Material Design 3 – Components](https://m3.material.io/components)
- [Material Design Catalog App](https://play.google.com/store/apps/details?id=androidx.compose.material.catalog)

---

### Layout Containers

Containers serve to organize and position UI elements relative to each other.

- **Column**
  - Arranges elements vertically, one below another.
  - Most common parameters:
    - `modifier` – modifier for entire column (e.g., size, padding).
    - `verticalArrangement` – element arrangement in vertical direction (e.g., `Arrangement.SpaceBetween`, `Arrangement.Center`).
    - `horizontalAlignment` – element alignment in horizontal direction (e.g., `Alignment.CenterHorizontally`).
  - Example:
    ```kotlin
    Column(
        modifier = Modifier.fillMaxWidth(),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text("First")
        Text("Second")
    }
    ```

- **Row**
  - Arranges elements horizontally, one next to another.
  - Most common parameters:
    - `modifier`
    - `horizontalArrangement` – element arrangement in horizontal direction.
    - `verticalAlignment` – element alignment in vertical direction.
  - Example:
    ```kotlin
    Row(
        modifier = Modifier.fillMaxWidth(),
        horizontalArrangement = Arrangement.SpaceBetween,
        verticalAlignment = Alignment.CenterVertically
    ) {
        Button(onClick = {}) { Text("OK") }
        Button(onClick = {}) { Text("Cancel") }
    }
    ```

- **Box**
  - Allows layering elements on top of each other (relative positioning). Children are drawn in declaration order — **last declared child is on top** (highest z-order).
  - Most common parameters:
    - `modifier`
    - `contentAlignment` – default alignment for all children (e.g., `Alignment.Center`).
  - Example:
    ```kotlin
    Box(
        modifier = Modifier.size(100.dp),
        contentAlignment = Alignment.BottomEnd
    ) {
        Image(painter, contentDescription = null)
        Icon(Icons.Default.Favorite, contentDescription = null)
    }
    ```

- **LazyColumn**
  - Vertically scrolling list (equivalent of RecyclerView).
  - Most common parameters:
    - `modifier`
    - `contentPadding` – padding inside list.
    - `verticalArrangement`
    - `horizontalAlignment`
  - Example with real data:
    ```kotlin
    data class Person(val id: Int, val name: String)

    val people = listOf(Person(1, "Anna"), Person(2, "John"), Person(3, "Mary"))

    LazyColumn(
        modifier = Modifier.fillMaxSize(),
        contentPadding = PaddingValues(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(
            items = people,
            key = { person -> person.id }   // stable key — prevents animation errors
        ) { person ->
            Text(
                text = person.name,
                modifier = Modifier.animateItem()   // animation for add/remove/move
            )
        }
    }
    ```

  **List Item Animation (`animateItem`)**

  `Modifier.animateItem()` added to list item causes:
  - new items appear with entry animation (fade in),
  - removed items disappear with exit animation (fade out),
  - order changes are smoothly animated.

  `key` parameter is essential for animations — without it Compose can't identify which item moved vs which is new.

- **LazyRow**
  - Horizontally scrolling list.
  - Most common parameters:
    - `modifier`
    - `contentPadding`
    - `horizontalArrangement`
    - `verticalAlignment`
  - Example:
    ```kotlin
    LazyRow(
        contentPadding = PaddingValues(horizontal = 8.dp)
    ) {
        items(10) { index ->
            Card(modifier = Modifier.size(80.dp)) {
                Text("Card $index")
            }
        }
    }
    ```

---

## Documentation

- [Official Jetpack Compose Documentation](https://developer.android.com/jetpack/compose)
- [Material Design 3 in Compose](https://developer.android.com/jetpack/androidx/releases/compose-material3)

---

### **Next topic:** [Activity](En/04%20Activity.md)
