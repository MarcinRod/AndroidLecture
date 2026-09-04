# Kotlin Basics for Android

Kotlin is a modern, statically-typed programming language, officially supported by Google for creating Android applications. It is readable, concise, and fully interoperable with Java.

---

## Key Features of Kotlin

- **Conciseness** – less code than Java, no unnecessary getters/setters.
- **Null safety** – type system prevents many null-related errors.
- **Higher-order functions and lambdas** – easy to pass functions as parameters.
- **Extensions** – ability to add new functions to existing classes.
- **Java interoperability** – can use Java libraries and gradually migrate code.

---

## Basic Kotlin Syntax

### Variables and Constants

In Kotlin, two keywords are used to store data: `val` and `var`.

- **val** – declares constants (value cannot be changed after assignment). Corresponds to read-only variables.
- **var** – declares variables (value can be changed freely during program execution).

**Examples:**

```kotlin
  val pi = 3.14           // constant, cannot assign a new value
  var counter = 0         // variable, can change value

  counter = 5             // OK
  // pi = 3.1415          // Compilation error!
```

- Variable type is usually inferred automatically, but can be specified explicitly:
  ```kotlin
  val name: String = "Android"
  var age: Int = 20
  ```

- A variable can be declared without initialization, but then type must be specified:
  ```kotlin
  var result: Int
  result = 42
  ```

**Best practices:**
- Use `val` always when value doesn't change – code is then safer and more readable.
- Use `var` only when variable value actually changes.

Variables and constants in Kotlin are fundamental for working with data in Android applications.

---

## Basic Data Types

- `Int`, `Long`, `Double`, `Float` – numbers
- `Boolean` – logical values (`true`/`false`)
- `String` – text
- `Char` – single character
- `List`, `MutableList`, `Set`, `Map` – collections

```kotlin
val number: Int = 42
val text: String = "Hello"
val list: List<String> = listOf("A", "B", "C")
```

---

## Conditional Expressions

- **if / else** – like in Java, but can also be an expression (returns value)
- **when** – extended version of switch

```kotlin
val x = 10
val result = if (x > 5) "large" else "small"

when (x) {
    1 -> println("one")
    in 2..10 -> println("from 2 to 10")
    else -> println("other")
}
```

---

## Smart Casting and the `is` Operator

The `is` operator is used to check an object's type at runtime. Kotlin automatically casts the variable to the appropriate type after such a check — this mechanism is called **smart cast**.

- **Type check:**
  ```kotlin
  val obj: Any = "Hello"
  if (obj is String) {
      println(obj.length) // obj is automatically treated as String
  }
  ```

- **Negation with `!is`:**
  ```kotlin
  if (obj !is Int) {
      println("This is not an integer")
  }
  ```

- **Smart cast in `when`:**
  The `is` operator works particularly well with `when` expression:
  ```kotlin
  fun describeType(x: Any): String = when (x) {
      is Int     -> "Integer: $x"
      is String  -> "Text with length ${x.length}"
      is Boolean -> "Boolean value: $x"
      else       -> "Unknown type"
  }
  println(describeType(42))        // Integer: 42
  println(describeType("Hello"))   // Text with length 5
  ```

**Note:** Smart cast works only when the compiler can guarantee the variable won't change value between the check and usage (applies to `val` or local `var`).

---

## String Handling

- **Concatenation:** Strings can be concatenated using the `+` operator:
  ```kotlin
  val name = "John"
  val greeting = "Hello, " + name + "!"
  ```

- **Interpolation:** Most common approach – embedding variable values directly into text using `$`:
  ```kotlin
  val name = "John"
  val greeting = "Hello, $name!"
  val age = 20
  val info = "You are ${age + 1} years old"
  ```

- **Multiline strings:** For creating text containing multiple lines or special characters without escaping, use triple quotes `""" ... """`:
  ```kotlin
  val multiline = """
      This is
      text on multiple lines
      Tab character:	<- here
  """.trimIndent()
  ```

- **Basic string operations:**
  - `length` – text length: `val length = text.length`
  - `uppercase()`, `lowercase()` – change letter case: `text.uppercase()`
  - `substring()` – extract fragment: `text.substring(0, 3)`
  - `replace()` – replace fragment: `text.replace("a", "b")`
  - `contains()` – check if text contains substring: `text.contains("cat")`
  - `split()` – split into parts: `"a,b,c".split(",")`
  - `trim()` – remove whitespace from start and end: `text.trim()`

- **Comparing strings:**
  - Standard comparison: `a == b`
  - Case-insensitive comparison: `a.equals(b, ignoreCase = true)`

- **Converting other types to string:**
  ```kotlin
  val number = 123
  val text = number.toString()
  ```

---

## Loops

Kotlin offers various types of loops and convenient ranges:

- **For loop with range:**
  ```kotlin
  for (i in 1..5) { // from 1 to 5 inclusive
      println(i)
  }
  for (i in 5 downTo 1) { // from 5 to 1 downward
      println(i)
  }
  for (i in 1 until 5) { // from 1 to 4 (5 excluded)
      println(i)
  }
  for (i in 0..10 step 2) { // every 2
      println(i)
  }
  ```

- **For loop over collection:**
  ```kotlin
  val list = listOf("A", "B", "C")
  for (element in list) {
      println(element)
  }
  ```

- **While loop:**
  ```kotlin
  var counter = 0
  while (counter < 3) {
      println(counter)
      counter++
  }
  ```

### Ranges

- Created using the `..` operator (e.g., `1..10`).
- Can use `downTo`, `until`, `step` to control the range.
- Often used in loops, but also in conditions (`if (x in 1..10)`).

**Example of range in condition:**
```kotlin
val age = 18
if (age in 13..19) {
    println("Teenager")
}
```

---

## Functions

Functions in Kotlin are the fundamental way to organize code. They allow code reuse, parameter passing, and value return.

```kotlin
fun add(a: Int, b: Int): Int {
    return a + b
}

// Function as expression (short form):
fun greet(name: String) = "Hello, $name!"
```

### Types of Function Arguments

- **Default arguments:** Parameters can be assigned a default value.
  ```kotlin
  fun greet(name: String = "Guest") {
      println("Hello, $name!")
  }
  greet() // Hello, Guest!
  greet("Anna") // Hello, Anna!
  ```

- **Named arguments:** Arguments can be passed by name, improving readability. No need to worry about argument order.
  ```kotlin
  fun order(name: String, quantity: Int, urgent: Boolean = false) { /* ... */ }
  order(name = "Coffee", quantity = 2, urgent = true)
  ```

- **Vararg arguments:** Allow passing any number of arguments of the same type.
  ```kotlin
  fun sum(vararg numbers: Int): Int = numbers.sum()
  val result = sum(1, 2, 3, 4) // 10
  ```

### Function Naming

- Function names should be verbs in indicative mood, e.g., `add`, `sendEmail`, `fetchData`.
- Functions returning boolean often start with `is`, `has`, `can`, e.g., `isActive()`, `hasPermission()`, `canEdit()`.
- Names should be short but descriptive.

---

## Lambda Functions

Lambda functions are short, anonymous functions that can be assigned to a variable or passed as an argument to another function.

- **Basic lambda syntax:**

    ```kotlin
    { parameters -> body }   // general syntax
    { x: Int -> x * 2 }      // example: lambda doubling a number
    ```

- **Lambda without parameters:**
  ```kotlin
  val greeting = { println("Hello!") }
  greeting() // prints: Hello!
  ```

- **Lambda with one parameter:**
  ```kotlin
  val double = { x: Int -> x * 2 }
  println(double(5)) // prints: 10
  ```

- **Lambda as function argument:**
  ```kotlin
  fun execute(action: () -> Unit) {
      action()
  }

  execute { println("This is a lambda!") }
  ```

- **Default parameter `it`:** When lambda accepts exactly one parameter and no explicit name is given, you can refer to it through `it`:
  ```kotlin
  val double = { x: Int -> x * 2 }   // explicit parameter name: x
  val double2: (Int) -> Int = { it * 2 } // same, using it

  val numbers = listOf(1, 2, 3, 4)
  val even = numbers.filter { it % 2 == 0 }  // it = current list element
  val doubled = numbers.map { it * 2 }        // it = current list element
  ```

## Collections

Kotlin offers convenient and safe collections widely used in Android applications. Most commonly used collection types:

- **List** – read-only list (immutable)
- **MutableList** – list that can be modified (add, remove elements)
- **Set** / **MutableSet** – set of unique elements
- **Map** / **MutableMap** – key-value pairs

**Declaration examples:**
```kotlin
val list = listOf("A", "B", "C")                  // immutable list
val mutableList = mutableListOf(1, 2, 3)          // mutable list
val set = setOf("cat", "dog", "cat")              // set (duplicates ignored)
val map = mapOf("key" to 1, "second" to 2)        // read-only map
val mutableMap = mutableMapOf("a" to 1, "b" to 2)
```

**Basic collection operations:**
- Read element from list: `list[0]`
- Add element to mutable list: `mutableList.add(4)`
- Remove element: `mutableList.remove(2)`
- Check if list contains element: `list.contains("A")`
- Iterate over collection:
  ```kotlin
  for (element in list) {
      println(element)
  }
  ```

**Lambdas and collections:**
Kotlin allows convenient collection processing with higher-order functions and lambdas. Below are most commonly used functions:

- **`filter`** — returns new collection containing only elements meeting the condition.
  Lambda accepts one argument (collection element, default `it`) and must return `Boolean`:
  ```kotlin
  // general syntax:
  collection.filter { element -> condition_returning_Boolean }

  val numbers = listOf(1, 2, 3, 4, 5, 6)
  val even = numbers.filter { it % 2 == 0 }           // [2, 4, 6]

  data class Product(val name: String, val price: Double)
  val products = listOf(Product("Juice", 3.99), Product("Coffee", 12.99), Product("Tea", 6.49))
  val cheap = products.filter { it.price < 7.0 }      // [Juice, Tea]
  ```

- **`map`** — transforms each collection element and returns new list of results.
  Lambda accepts element and returns any value — resulting list type depends on what lambda returns:
  ```kotlin
  // general syntax:
  collection.map { element -> transformed_value }

  val numbers = listOf(1, 2, 3, 4, 5)
  val doubled = numbers.map { it * 2 }                // [2, 4, 6, 8, 10]

  // map can also change element types:
  val products = listOf(Product("Juice", 3.99), Product("Coffee", 12.99))
  val names = products.map { it.name }                // ["Juice", "Coffee"]  (List<String>)
  val prices = products.map { it.price }              // [3.99, 12.99]    (List<Double>)
  ```

- **`forEach`** — executes operation for each element (doesn't return value):
  ```kotlin
  val products = listOf("Coffee", "Tea", "Juice")
  products.forEach { println("Product: $it") }
  ```

- **`any` / `all` / `none`** — check if condition is met:
  ```kotlin
  val numbers = listOf(1, 2, 3, 4, 5)
  println(numbers.any { it > 4 })   // true  — at least one > 4
  println(numbers.all { it > 0 })   // true  — all > 0
  println(numbers.none { it > 10 }) // true  — none > 10
  ```

- **`find` / `firstOrNull`** — returns first element meeting condition or `null`:
  ```kotlin
  val numbers = listOf(1, 2, 3, 4, 5)
  val first = numbers.find { it > 3 }       // 4
  val missing = numbers.find { it > 10 }    // null
  ```

- **`sortedBy` / `sortedByDescending`** — sorts collection by selected property.
  Lambda returns sorting key (comparable value like `Int`, `String`, `Double`):
  ```kotlin
  // general syntax:
  collection.sortedBy { element -> sorting_key }

  data class Product(val name: String, val price: Double)
  val products = listOf(Product("Juice", 3.99), Product("Coffee", 12.99), Product("Tea", 6.49))

  val byPrice = products.sortedBy { it.price }
  // [Juice(3.99), Tea(6.49), Coffee(12.99)]

  val byName = products.sortedBy { it.name }
  // [Coffee, Juice, Tea]  (alphabetically)

  val byPriceDesc = products.sortedByDescending { it.price }
  // [Coffee(12.99), Tea(6.49), Juice(3.99)]
  ```

- **`groupBy`** — groups elements by key, returns `Map<K, List<V>>`.
  Lambda returns grouping key — all elements with same key go into one list:
  ```kotlin
  // general syntax:
  collection.groupBy { element -> group_key }   // → Map<KeyType, List<ElementType>>

  data class Product(val name: String, val category: String, val price: Double)
  val products = listOf(
      Product("Coffee", "beverage", 12.99),
      Product("Tea", "beverage", 6.49),
      Product("Bread", "bakery", 4.99),
      Product("Roll", "bakery", 1.50)
  )

  val byCategory = products.groupBy { it.category }
  // { "beverage" -> [Coffee, Tea], "bakery" -> [Bread, Roll] }

  // access specific group:
  val drinks = byCategory["beverage"]   // [Coffee, Tea]
  ```

- **`reduce` / `fold`** — reduces collection to single value, processing elements sequentially.
  Lambda accepts two arguments: **accumulator** (`acc` — result of previous processing) and **current element** (`n`).
  - `reduce` — accumulator initial value is first element
  - `fold` — accumulator initial value is provided explicitly as argument; works on empty collection too
  ```kotlin
  // general syntax:
  collection.reduce { acc, element -> new_accumulator_value }
  collection.fold(initial_value) { acc, element -> new_accumulator_value }

  val numbers = listOf(1, 2, 3, 4, 5)

  // reduce: acc starts at 1, then: 1+2=3, 3+3=6, 6+4=10, 10+5=15
  val sum = numbers.reduce { acc, n -> acc + n }           // 15

  // fold: acc starts at 100, then: 100+1=101, ..., 110+5=115
  val sumFrom100 = numbers.fold(100) { acc, n -> acc + n } // 115

  // fold for building string:
  val text = numbers.fold("Numbers:") { acc, n -> "$acc $n" }
  // "Numbers: 1 2 3 4 5"
  ```

- **Chaining operations:**
  ```kotlin
  val numbers = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
  val result = numbers
      .filter { it % 2 == 0 }   // [2, 4, 6, 8, 10]
      .map { it * it }           // [4, 16, 36, 64, 100]
      .sum()                     // 220
  ```

**Tips:**
- By default, collections are immutable — if modification is needed, use `MutableList`, `MutableSet` or `MutableMap`.
- Collection operations (`filter`, `map`, etc.) don't modify original collection — they return new one.

## Null Safety

Null safety is one of Kotlin's most important features, helping avoid `null`-related errors (so-called NullPointerException).

- **By default, variables cannot be null:**
  ```kotlin
  var text: String = "Hello"
  text = null // Compilation error!
  ```

- **For variable to accept null, add question mark to type:**
  ```kotlin
  var text: String? = null
  text = "Hello"
  ```

- **Safe call operator `?.`:**
  Allows calling method or accessing property only if object is not null.
  ```kotlin
  println(text?.length) // if text == null, result is null, no error
  ```

- **Elvis operator `?:`:**
  Allows setting default value when variable is null.
  ```kotlin
  val length = text?.length ?: 0 // if text == null, length = 0
  ```

- **Non-null assertion `!!`:**
  Throws exception if variable is null (use only when you're sure object is not null).
  ```kotlin
  val length = text!!.length // throws exception if text == null
  ```

---

## Documentation

- [Official Kotlin documentation](https://kotlinlang.org/docs/)
- [Google's Kotlin style guide for Android](https://developer.android.com/kotlin/style-guide)

---

### **Next topic:** [Android Studio](/En/02%20Android%20Studio.md)
