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

 - **Safe cast (`as?`):**
    Returns `null` if the cast fails.
    ```kotlin
    val obj: Any = "text"
    val text: String? = obj as? String  // "text"
    val number: Int? = obj as? Int       // null (the cast failed)
    ```

 - **Practical example:**
    ```kotlin
    fun showLength(text: String?) {
      println("Length: ${text?.length ?: "no text"}")
    }
    showLength("Kotlin") // Length: 6
    showLength(null)     // Length: no text
    ```

**Tips:**
  - Avoid nullable variables when they are not necessary.
  - Null safety makes code safer and easier to maintain, which is especially important in Android applications.

---

## Classes

  In Kotlin, classes are the basic way to define your own data types.

### Class Naming

  - Kotlin class names should use **PascalCase** — each word starts with a capital letter, with no underscores, e.g., `User`, `MainActivity`, `ProductItem`.
  - A class name should clearly describe what the class represents (e.g., `User`, `Order`, `LoginViewModel`).
  - The same rules apply to data classes (`data class`); the name should be a noun.
  - Avoid abbreviations and unclear names — code should be self-explanatory.
  - Singleton objects (`object`) and companion objects should also use PascalCase, e.g., `Logger`, `DatabaseHelper`.

  **Examples of good names:**
  ```kotlin
  class UserProfile
  data class ProductItem(val name: String, val price: Double)
  object NetworkManager
  ```
  **Examples of poor names:**
  ```kotlin
  class userprofile
  class product_item
  object networkmanager
  ```
  Good class naming makes code easier to read, test, and maintain in larger Android projects.

  - **Regular class:**
    ```kotlin
    class Person(val name: String, var age: Int)
    val jan = Person("Jan", 30)
    println(jan.name) // Jan
    jan.age = 31
    ```

  - **Class with methods:**
    ```kotlin
    class Calculator {
      fun add(a: Int, b: Int): Int = a + b
    }
    val calc = Calculator()
    println(calc.add(2, 3)) // 5
    ```

### Class Constructor

  In Kotlin, a constructor is a special function used to create objects of a class. The **primary constructor** is used most often and is defined directly in the class header:

  ```kotlin
  class Person(val name: String, var age: Int)
  ```
  - Constructor parameters can immediately become class properties by using `val` or `var`, as in the example above.
  - Creating an object:
    ```kotlin
    val jan = Person("Jan", 30)
    ```

  You can also define a **secondary constructor** when you need another way to create an object:

  ```kotlin
  class Person(val name: String) {
    var age: Int = 0

    constructor(name: String, age: Int) : this(name) {
      this.age = age
    }
  }
  ```

  - A secondary constructor uses the `constructor` keyword and must always call the primary constructor (`: this(...)`).

  **Important:**
  - If a class has no properties or methods, you can omit the parentheses:
    ```kotlin
    class Empty
    ```
  - If additional logic is needed when an object is created, you can use an `init` block:
    ```kotlin
    class Person(val name: String) {
      init {
        println("Creating a person named $name")
      }
    }
    ```

## Inheritance

  In Kotlin, you can create class hierarchies and use inheritance to reuse and extend functionality.

  - By default, every Kotlin class is **final** (it cannot be inherited from). To allow inheritance, use the `open` keyword on the class and its methods.

  **Inheritance example:**
  ```kotlin
  open class Animal(val name: String) {
    open fun makeSound() {
      println("The animal makes a sound")
    }
  }

  class Dog(name: String) : Animal(name) {
    override fun makeSound() {
      println("Woof woof!")
    }
  }

  val dog = Dog("Reksio")
  dog.makeSound() // Woof woof!
  ```

  - **open** — allows a class to be inherited from or a method to be overridden.
  - **override** — overrides a method from the base class.
  - A subclass calls the base class constructor using `: Base(...)`.

### The `this` and `super` Keywords

  - **`this`** — refers to the current object (class instance). It is used inside a class to refer to its own properties and methods. It is also used in scope functions (`apply`, `run`, `with`), where it refers to the object on which the block runs:
    ```kotlin
    class Person(val name: String) {
      fun greet() = "Hello, I am ${this.name}" // this can be omitted here
    }
    ```

  - **`super`** — refers to the base class (parent class). It lets you call a parent method or constructor when a subclass overrides it:
    ```kotlin
    class Dog(name: String) : Animal(name) {
      override fun makeSound() {
        super.makeSound()   // call the implementation from Animal
        println("Woof woof!")
      }
    }
    ```

### Interfaces

  Interfaces in Kotlin are defined using the `interface` keyword. An interface can contain method declarations (without implementations) as well as default method implementations.

  - A class can implement any number of interfaces (unlike class inheritance, which is limited to one base class).
  - Interfaces are often used to define contracts that classes must fulfill (e.g., handling clicks or communication between components).

  **Simple interface example:**
  ```kotlin
  interface Clickable {
    fun click()
  }

  class Button : Clickable {
    override fun click() {
      println("Button clicked")
    }
  }

  val button = Button()
  button.click() // Button clicked
  ```

  **Interface with a default implementation:**
  ```kotlin
  interface Greetable {
    fun greet(name: String) {
      println("Hello, $name!")
    }
  }

  class User : Greetable

  val user = User()
  user.greet("Anna") // Hello, Anna!
  ```

  **Implementing multiple interfaces:**
  ```kotlin
  interface Clickable { fun click() }
  interface Flyable { fun fly() }

  class SuperBird : Clickable, Flyable {
    override fun click() { println("Bird clicked") }
    override fun fly() { println("Bird is flying") }
  }
  ```

  **Important features of Kotlin interfaces:**
  - An interface can have properties without state, e.g., `val name: String`.
  - An interface cannot store state (it cannot have fields with values).
  - A class can implement multiple interfaces.

  Interfaces are widely used in Android, e.g., for handling events (clicks, callbacks), communication between fragments, adapters, and more.

### Abstract Classes

  An abstract class (`abstract class`) cannot be instantiated directly — it serves as a template for subclasses. It can contain both abstract methods (without implementations) and implemented methods, as well as fields that store state.

  ```kotlin
  abstract class Shape {
    // abstract — every subclass MUST provide its own implementation;
    // Shape does not know how to calculate the area (a circle, rectangle, and triangle do it differently)
    abstract fun area(): Double

    // regular method with an implementation — shared by all subclasses,
    // but it can also be overridden
    fun describe() {
      println("Area: ${area()}")
    }
  }

  class Circle(val radius: Double) : Shape() {
    override fun area() = Math.PI * radius * radius
  }

  class Square(val edge: Double) : Shape() {
    override fun area() = edge * edge
    override fun describe() {
       super.describe() // access the describe function from the parent class
       println("This is a square")
    }
  }

  val circle = Circle(5.0)
  circle.describe() // Area: 78.53...

  val square = Square(5.0)
  square.describe()
  // Area: 25.0
  // This is a square
  ```

  **Differences between `abstract class` and `interface`:**

  | Feature | `abstract class` | `interface` |
  |---|---|---|
  | Instantiation | not possible | not possible |
  | Fields with state | yes | no |
  | Constructor | yes | no |
  | Method implementation | yes — methods can be abstract (without a body) or concrete (with a body) | yes — methods can have a default implementation, but do not have to; a class implementing a method without a body must provide its own implementation |
  | Inheritance | only one class | multiple interfaces at the same time |

  **When to use each option:**
  - `abstract class` — when subclasses share state (fields) or base logic; when the class hierarchy represents an “is a kind of” relationship (e.g., a `Circle` is a `Shape`).
  - `interface` — when you only need to define a contract (what a class can do); when one class should fulfill multiple independent contracts.

## Special Classes

  - **Data class** — a special class type for storing data. It automatically generates `equals()`, `hashCode()`, `toString()`, `copy()`, and `componentN()` methods:
    ```kotlin
    data class Product(val name: String, val price: Double)
    val coffee = Product("Kawa", 12.99)
    println(coffee) // Product(name=Kawa, price=12.99)
    val cheaperCoffee = coffee.copy(price = 10.99)
    ```

  - **object** — a keyword for creating singletons (single instances of classes) or anonymous objects.

    * **Singleton:**
    ```kotlin
    object Logger {
      fun log(msg: String) {
        println("LOG: $msg")
      }
    }
    Logger.log("Application started")
    ```

    * **Anonymous object** — useful, for example, for implementing interfaces on the fly:
    ```kotlin
    val listener = object : Clickable {
      override fun click() {
        println("Anonymous object clicked")
      }
    }
    listener.click()
    ```

    * **companion object** — an object associated with a class. It lets you create static-like methods and fields without creating a separate instance:
    ```kotlin
    class User(val name: String) {
      companion object {
        fun createAnonymous() = User("Anonymous")
      }
    }
    val anonymous = User.createAnonymous()
    ```

  - **enum class** — an enumeration type that defines a limited set of constant values. It is useful for representing states, types, and categories:
    ```kotlin
    enum class Status {
      LOADING, SUCCESS, ERROR
    }

    val status = Status.SUCCESS
    when (status) {
      Status.LOADING -> println("Loading...")
      Status.SUCCESS -> println("Success!")
      Status.ERROR   -> println("An error occurred")
    }
    ```
    Kotlin enums can have their own properties, methods, and constructors. This lets them store additional data and behavior.

    ```kotlin
    enum class OrderStatus(val description: String, val isFinal: Boolean) {
      NEW("New order", false),
      IN_PROGRESS("In progress", false),
      COMPLETED("Completed", true),
      CANCELLED("Cancelled", true); // note the semicolon
    }
    ```

  - **sealed class / sealed interface** — a class or interface representing a **closed type hierarchy**: all direct subtypes must be defined in the same package and compilation module (in Kotlin before 1.5, they had to be in the same file). The compiler therefore knows all possible subtypes, so a `when` expression can be exhaustive without an `else` branch.

    > A **compilation module** is a set of Kotlin files compiled together in one step (e.g., one Gradle module in an Android project). A sealed class subtype cannot be defined in another library or module — the hierarchy is closed to external code.

    ```kotlin
    sealed class Result {
      data class Success(val value: Int) : Result() // use class when the constructor is non-empty
      data class Failure(val error: String) : Result()
      data object Loading : Result() // use object when there are no constructor parameters
    }

    fun handle(result: Result) {
      when (result) {
        is Result.Success -> println("Result: ${result.value}")
        is Result.Failure -> println("Error: ${result.error}")
        is Result.Loading -> println("Loading...")
        // no else is needed — the compiler knows these are all cases
      }
    }
    ```

    A `sealed interface` works similarly, but implementing classes can also inherit from another class (a sealed class cannot do this because Kotlin does not support multiple class inheritance):

    ```kotlin
    sealed interface Result {
      data class Success(val value: Int) : Result
      data class Failure(val error: String) : Result
      data object Loading : Result
    }

    fun handle(result: Result) {
      when (result) {
        is Result.Success -> println("Result: ${result.value}")
        is Result.Failure -> println("Error: ${result.error}")
        is Result.Loading -> println("Loading...")
      }
    }
    ```

    The key difference: `data class Success` can also inherit from another class here, e.g., `class Success(...) : SomeBaseClass(), Result`, which is not possible with a `sealed class`.

    **Comparison of `enum class` and `sealed class`:**

    | Feature | `enum class` | `sealed class` |
    |---|---|---|
    | Number of instances | fixed, one per value | any number of objects of each subtype |
    | Data in variants | same structure for all | each subtype can have different fields |
    | Subtypes | only enumeration constants | full classes (`data class`, `object`, classes with logic) |
    | Exhaustive `when` | yes | yes |
    | Use case | fixed set of simple constants (e.g., `Direction.NORTH`) | type hierarchy with different data (e.g., operation result or state) |

  ---

## `lateinit` and `by lazy`

  In Kotlin, variables must be initialized when they are declared. However, two mechanisms let you defer initialization until later.

### `lateinit`

  The `lateinit` keyword is used with non-null `var` properties when initialization cannot happen at the declaration, but will happen before the first use. It works only with object types (not `Int`, `Boolean`, etc.).

  ```kotlin
  class UserRepository {
    lateinit var database: Database  // initialized later, e.g., by a DI framework

    fun setup(db: Database) {
      database = db
    }

    fun findUser(id: Int) = database.query(id)
  }
  ```

  - Reading a `lateinit` property before it has been initialized throws an `UninitializedPropertyAccessException`.
  - You can check whether a property has been initialized: `::database.isInitialized`.

### `by lazy`

  The `by lazy` delegate provides **lazy initialization** for `val` properties — the value is computed on first access and then cached.

  ```kotlin
  val processedData: List<String> by lazy {
    println("Initializing the list...")
    listOf("A", "B", "C").map { it.lowercase() }
  }

  // The lazy block has not run yet
  println(processedData) // initialization happens now
  println(processedData) // second time: cached result; the block does not run again
  ```

  **Comparison:**

  | Feature | `lateinit` | `by lazy` |
  |---|---|---|
  | Property type | `var` | `val` |
  | Initialization | manual, at any point | automatic, on first use |
  | Allowed types | object types only | all types |
  | Thread safety | no | yes, by default |

  ---

## Destructuring

  Destructuring lets you split an object into several variables in one statement. It is available for `data class` instances, pairs (`Pair`), map entries, and other types.

  - **Destructuring a `data class`:**
    ```kotlin
    data class Product(val name: String, val price: Double)
    val coffee = Product("Kawa", 12.99)

    val (name, price) = coffee
    println("$name costs $price zł") // Kawa costs 12.99 zł
    ```

  - **Destructuring while iterating over a map:**
    ```kotlin
    val map = mapOf("a" to 1, "b" to 2) // the `to` keyword creates a pair, e.g., "a" to 1 == Pair("a", 1)
    for ((key, value) in map) {
      println("$key = $value")
    }
    ```

  - **Ignoring a value with `_`:**
    If a variable is not needed, you can omit it:
    ```kotlin
    val (_, price) = Product("Tea", 8.50) // name is ignored
    ```

  - **Destructuring in lambdas:**
    ```kotlin
    val products = listOf(Product("Kawa", 12.99), Product("Sok", 5.00))
    products.forEach { (name, price) ->
      println("$name: $price zł")
    }
    ```

  ---

## Scope Functions

  Scope functions are built-in Kotlin functions that execute a block of code in the context of a given object. They make code more concise, especially when initializing objects and handling nullable values.

  Kotlin has five scope functions: `let`, `apply`, `run`, `also`, and `with`. They differ in how you refer to the object (`this` or `it`) and what they return.

  | Function | Object reference | Returns |
  |---|---|---|
  | `let` | `it` | block result |
  | `apply` | `this` | object |
  | `run` | `this` | block result |
  | `also` | `it` | object |
  | `with` | `this` | block result |

### `apply` — configuring an object

  Used mainly to initialize or configure an object. Inside the block, the object is available as `this`. It returns the object itself.

  ```kotlin
  data class Config(var host: String = "", var port: Int = 0, var timeout: Int = 0)

  val config = Config().apply {
    host = "localhost"
    port = 8080
    timeout = 30
  }
  ```

### `let` — working with a value and handling null

  Often used with the `?.` operator to safely handle nullable values. The object is available as `it`.

  ```kotlin
  val text: String? = fetchText()
  text?.let {
    println("Length: ${it.length}") // runs only when text != null
  }

  // let for transforming a value:
  val doubleLength = "Kotlin".let { it.length * 2 } // 12
  ```

### `run` — a block of operations that returns a result

  There are two forms:

  - **On an object** — similar to `apply`, but returns the block result (not the object). The object is available as `this`:
    ```kotlin
    val result = StringBuilder().run {
      append("Kotlin ")
      append("is great")
      toString() // this value is returned
    }
    println(result) // Kotlin is great
    ```

  - **Without an object** — a code block executed in place that returns a value. Useful for extracting a piece of logic or initializing a variable that requires several steps:
    ```kotlin
    val value = run {
      val base = 10
      val factor = 3
      base * factor // block result assigned to value
    }
    println(value) // 30
    ```

### `also` — side effects

  Used when an additional operation (e.g., logging) is needed without modifying the object. It returns the object.

  ```kotlin
  val list = mutableListOf(1, 2, 3)
    .also { println("List before: $it") }
  list.add(4)
  ```

### `with` — operating on an object without an extension call

  Takes an object as an argument (rather than as the call receiver). Useful when the object is already known and is not the result of an expression.

  ```kotlin
  val person = Person("Anna", 25)
  with(person) {
    println("Name: $name")
    println("Age: $age")
  }
  ```

  **Guidelines — when to use each function:**
  - `apply` — configuring or initializing an object.
  - `let` — working with nullable values or limiting a variable's scope.
  - `also` — a side effect (e.g., logging) without changing the object.
  - `run` / `with` — a sequence of operations on an object when you need the block result.

  ---

## Extension Functions and Properties

  Extension functions and properties let you add new methods or properties to existing classes — even classes you cannot modify (e.g., library or Java classes).

### Extension Functions

  - Define them outside the class, prefixing the function name with the type being extended:
    ```kotlin
    fun String.reverse(): String = this.reversed()

    val text = "Kotlin"
    println(text.reverse()) // "niltok"
    ```

  - You can extend any type, including Android classes:
    ```kotlin
    fun Context.toast(msg: String) =
      Toast.makeText(this, msg, Toast.LENGTH_SHORT).show()
    ```

  - Extension functions can access the extended type's public methods and properties using `this`.

  - Extension functions do not modify the original class.

### Extension Properties

  - They let you add “pseudo-properties” to existing classes:
    ```kotlin
    val String.reversed: String
      get() = this.reversed()

    println("Android".reversed) // "diordnA"
    ```

  - Extension properties cannot have state (they cannot declare a backing field); they can only have a getter.

  **Summary:**
  - Extension functions and properties improve readability and help you write more idiomatic APIs.
  - They do not modify the original classes — they are safe and convenient.

  ---

## Documentation

- [Official Kotlin documentation](https://kotlinlang.org/docs/)
- [Google's Kotlin style guide for Android](https://developer.android.com/kotlin/style-guide)

---

### **Next topic:** [Android Studio](/En/02%20Android%20Studio.md)
