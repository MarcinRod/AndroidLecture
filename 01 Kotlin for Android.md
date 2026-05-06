# Podstawy języka Kotlin dla Androida

Kotlin to nowoczesny, statycznie typowany język programowania, oficjalnie wspierany przez Google do tworzenia aplikacji na Androida. Jest czytelny, zwięzły i w pełni interoperacyjny z Javą.

---

##  Najważniejsze cechy Kotlina

- **Zwięzłość** – mniej kodu niż w Javie, brak zbędnych getterów/setterów.
- **Bezpieczeństwo względem wartości null** – system typów zapobiega wielu błędom z `null`.
- **Funkcje wyższego rzędu i lambdy** – łatwe przekazywanie funkcji jako parametrów.
- **Rozszerzenia** – możliwość dodawania nowych funkcji do istniejących klas.
- **Współpraca z Javą** – można korzystać z bibliotek Java i stopniowo migrować kod.

---

##  Podstawowa składnia Kotlina

### Zmienne i stałe

W Kotlinie do przechowywania danych używa się dwóch słów kluczowych: `val` i `var`.

- **val** – służy do deklarowania stałych (wartość nie może być zmieniona po przypisaniu). Odpowiada to zmiennym tylko do odczytu (read-only).
- **var** – służy do deklarowania zmiennych (wartość można zmieniać dowolnie w trakcie działania programu).

**Przykłady:**

  ```kotlin
    val pi = 3.14           // stała, nie można przypisać nowej wartości
    var counter = 0         // zmienna, można zmieniać wartość

    counter = 5             // OK
    // pi = 3.1415          // Błąd kompilacji!
  ```

- Typ zmiennej jest zazwyczaj domniemywany automatycznie, ale można go podać jawnie:
  ```kotlin
  val name: String = "Android"
  var age: Int = 20
  ```

- Można zadeklarować zmienną bez inicjalizacji, ale wtedy należy podać typ:
  ```kotlin
  var result: Int
  result = 42
  ```

**Dobre praktyki:**
- Warto używać `val` zawsze, gdy wartość się nie zmienia – kod jest wtedy bezpieczniejszy i bardziej czytelny.
- Warto używać `var` tylko wtedy, gdy wartość zmiennej faktycznie się zmienia.

Zmienne i stałe w Kotlinie są podstawą do pracy z danymi w aplikacjach Android.

---

## Podstawowe typy danych

- `Int`, `Long`, `Double`, `Float` – liczby
- `Boolean` – wartości logiczne (`true`/`false`)
- `String` – tekst
- `Char` – pojedynczy znak
- `List`, `MutableList`, `Set`, `Map` – kolekcje

```kotlin
val number: Int = 42
val text: String = "Hello"
val list: List<String> = listOf("A", "B", "C")
```

---

## Wyrażenia warunkowe

- **if / else** – jak w Javie, ale może być też wyrażeniem (zwraca wartość)
- **when** – rozbudowana wersja switch

```kotlin
val x = 10
val result = if (x > 5) "duże" else "małe"

when (x) {
    1 -> println("jeden")
    in 2..10 -> println("od 2 do 10")
    else -> println("inne")
}
```

---

## Inteligentne rzutowanie i operator `is`

Operator `is` służy do sprawdzania typu obiektu w czasie działania programu. Kotlin automatycznie rzutuje zmienną na odpowiedni typ po takim sprawdzeniu — mechanizm ten nosi nazwę **smart cast**.

- **Sprawdzenie typu:**
  ```kotlin
  val obiekt: Any = "Hello"
  if (obiekt is String) {
      println(obiekt.length) // obiekt jest automatycznie traktowany jako String
  }
  ```

- **Negacja z `!is`:**
  ```kotlin
  if (obiekt !is Int) {
      println("To nie jest liczba całkowita")
  }
  ```

- **Smart cast w `when`:**
  Operator `is` szczególnie dobrze współpracuje z wyrażeniem `when`:
  ```kotlin
  fun opisTypu(x: Any): String = when (x) {
      is Int     -> "Liczba całkowita: $x"
      is String  -> "Tekst o długości ${x.length}"
      is Boolean -> "Wartość logiczna: $x"
      else       -> "Nieznany typ"
  }
  println(opisTypu(42))       // Liczba całkowita: 42
  println(opisTypu("Cześć")) // Tekst o długości 5
  ```

**Uwaga:** Smart cast działa tylko wtedy, gdy kompilator może zagwarantować, że zmienna nie zmieni wartości między sprawdzeniem a użyciem (dotyczy to np. zmiennych `val` lub lokalnych `var`).

---

## Obsługa stringów

- **Łączenie:** Napisy można łączyć za pomocą operatora `+`:
  ```kotlin
  val name = "Jan"
  val greeting = "Ćześć, " + name + "!"
  ```

- **Interpolacja:** Najczęściej używany sposób – wstawianie wartości zmiennych bezpośrednio do tekstu za pomocą `$`:
  ```kotlin
  val name = "Jan"
  val greeting = "Ćześć, $name!"
  val age = 20
  val info = "Masz ${age + 1} lat"
  ```

- **Wielolinijkowe stringi:** Do tworzenia tekstów zawierających wiele linii lub znaki specjalne bez konieczności ich uciekania, używa się potrójnych cudzysłowów `""" ... """`:
  ```kotlin
  val multiline = """
      To jest
      tekst w kilku liniach
      Znak tabulacji:	<- tutaj
  """.trimIndent()
  ```

- **Podstawowe operacje na stringach:**
  - `length` – długość tekstu: `val length = text.length`
  - `uppercase()`, `lowercase()` – zmiana wielkości liter: `text.uppercase()`
  - `substring()` – wycinanie fragmentu: `text.substring(0, 3)`
  - `replace()` – zamiana fragmentu: `text.replace("a", "b")`
  - `contains()` – sprawdzenie, czy tekst zawiera podciąg: `text.contains("kot")`
  - `split()` – dzielenie na części: `"a,b,c".split(",")`
  - `trim()` – usuwanie białych znaków z początku i końca: `text.trim()`

- **Porównywanie stringów:**
  - Standardowe porównanie: `a == b`
  - Porównanie bez rozróżniania wielkości liter: `a.equals(b, ignoreCase = true)`

- **Konwersja innych typów do stringa:**
  ```kotlin
  val number = 123
  val text = number.toString()
  ```

---

## Pętle

W Kotlinie można korzystać z różnych rodzajów pętli, a także z wygodnych zakresów (range):

- **Pętla for z zakresem:**
  ```kotlin
  for (i in 1..5) { // od 1 do 5 włącznie
      println(i)
  }
  for (i in 5 downTo 1) { // od 5 do 1 w dół
      println(i)
  }
  for (i in 1 until 5) { // od 1 do 4 (5 wyłączone)
      println(i)
  }
  for (i in 0..10 step 2) { // co 2
      println(i)
  }
  ```

- **Pętla for po kolekcji:**
  ```kotlin
  val list = listOf("A", "B", "C")
  for (element in list) {
      println(element)
  }
  ```

- **Pętla while:**
  ```kotlin
  var counter = 0
  while (counter < 3) {
      println(counter)
      counter++
  }
  ```

### Zakresy (range)

- tworzone za pomocą operatora `..` (np. `1..10`).
- Można użyć `downTo`, `until`, `step` do sterowania zakresem.
- Zakresy są często wykorzystywane w pętlach, ale także w warunkach (`if (x in 1..10)`).

**Przykład użycia zakresu w warunku:**
```kotlin
val age = 18
if (age in 13..19) {
    println("Nastolatek")
}
```
---


## Funkcje

Funkcje w Kotlinie są podstawowym sposobem organizacji kodu. Pozwalają na wielokrotne użycie logiki, przekazywanie parametrów i zwracanie wartości.

```kotlin
fun add(a: Int, b: Int): Int {
    return a + b
}

// Funkcja jako wyrażenie (skrótowa forma):
fun greet(name: String) = "Ćześć, $name!"
```

### Rodzaje argumentów funkcji

- **Argumenty domyślne:** Parametrom można przypisać domyślną wartość.
  ```kotlin
  fun greet(name: String = "Gość") {
      println("Ćześć, $name!")
  }
  greet() // Ćześć, Gość!
  greet("Anna") // Ćześć, Anna!
  ```

- **Argumenty nazwane:** Argumenty można przekazywać po nazwie, co zwiększa czytelność. Nie trzeba też przejmować się kolejnością argumentów.
  ```kotlin
  fun order(name: String, quantity: Int, urgent: Boolean = false) { /* ... */ }
  order(name = "Kawa", quantity = 2, urgent = true)
  ```

- **Argumenty vararg:** Pozwalają przekazać dowolną liczbę argumentów tego samego typu.
  ```kotlin
  fun sum(vararg numbers: Int): Int = numbers.sum()
  val result = sum(1, 2, 3, 4) // 10
  ```

### Nazewnictwo funkcji

- Nazwy funkcji powinny być czasownikami w trybie oznajmującym, np. `add`, `sendEmail`, `fetchData`.
- Funkcje zwracające wartość logiczną często zaczynają się od `is`, `has`, `can`, np. `isActive()`, `hasPermission()`, `canEdit()`.
- Nazwy powinny być krótkie, ale opisowe.

---

##  Funkcje lambda


Funkcje lambda to krótkie, anonimowe funkcje, które można przypisać do zmiennej lub przekazać jako argument do innej funkcji.
- **Podstawowa składnia funkcji lambda**:

    ```kotlin
    { parametry -> ciało }   // ogólna składnia
    { x: Int -> x * 2 }      // przykład: lambda podwajająca liczbę
    ```

- **Lambda bez parametrów:**
  ```kotlin
  val greeting = { println("Ćześć!") }
  greeting() // wypiśe: Ćześć!
  ```

- **Lambda z jednym parametrem:**
  ```kotlin
  val double = { x: Int -> x * 2 }
  println(double(5)) // wypiśe: 10
  ```

- **Lambda jako argument funkcji:**
  ```kotlin
  fun execute(action: () -> Unit) {
      action()
  }

  execute { println("To jest lambda!") }
  ```

- **Domyślny parametr `it`:** gdy lambda przyjmuje dokładnie jeden parametr i nie nadano mu jawnej nazwy, można odwołać się do niego przez `it`:
  ```kotlin
  val double = { x: Int -> x * 2 }   // jawna nazwa parametru: x
  val double2: (Int) -> Int = { it * 2 } // to samo, z użyciem it

  val numbers = listOf(1, 2, 3, 4)
  val even = numbers.filter { it % 2 == 0 }  // it = bieżący element listy
  val doubled = numbers.map { it * 2 }        // it = bieżący element listy
  ```

## Kolekcje

Kotlin oferuje wygodne i bezpieczne kolekcje, które są szeroko wykorzystywane w aplikacjach Android. Najczęściej używane typy kolekcji to:

- **List** – lista tylko do odczytu (niemutowalna)
- **MutableList** – lista, którą można modyfikować (dodawać, usuwać elementy)
- **Set** / **MutableSet** – zbiór unikalnych elementów
- **Map** / **MutableMap** – para klucz-wartość

**Przykłady deklaracji:**
```kotlin
val list = listOf("A", "B", "C")                  // niemutowalna lista
val mutableList = mutableListOf(1, 2, 3)          // mutowalna lista
val set = setOf("kot", "pies", "kot")             // zbiór (duplikaty ignorowane)
val map = mapOf("klucz" to 1, "drugi" to 2)       // mapa tylko do odczytu
val mutableMap = mutableMapOf("a" to 1, "b" to 2)
```

**Podstawowe operacje na kolekcjach:**
- Odczyt elementu z listy: `list[0]`
- Dodanie elementu do mutowalnej listy: `mutableList.add(4)`
- Usunięcie elementu: `mutableList.remove(2)`
- Sprawdzenie, czy lista zawiera element: `list.contains("A")`
- Iteracja po kolekcji:
  ```kotlin
  for (element in list) {
      println(element)
  }
  ```

**Wyrażenia lambda i kolekcje:**
Kotlin pozwala na wygodne przetwarzanie kolekcji za pomocą funkcji wyższego rzędu i lambd. Poniżej najczęściej używane funkcje:

- **`filter`** — zwraca nową kolekcję zawierającą tylko elementy spełniające warunek.
  Przekazywana lambda przyjmuje jeden argument (element kolekcji, domyślnie `it`) i musi zwracać `Boolean`:
  ```kotlin
  // składnia ogólna:
  kolekcja.filter { element -> warunek_zwracający_Boolean }

  val numbers = listOf(1, 2, 3, 4, 5, 6)
  val even = numbers.filter { it % 2 == 0 }           // [2, 4, 6]

  data class Product(val name: String, val price: Double)
  val products = listOf(Product("Sok", 3.99), Product("Kawa", 12.99), Product("Herbata", 6.49))
  val cheap = products.filter { it.price < 7.0 }      // [Sok, Herbata]
  ```

- **`map`** — przekształca każdy element kolekcji i zwraca nową listę wyników.
  Lambda przyjmuje element i zwraca dowolną wartość — typ wynikowej listy zależy od tego, co lambda zwraca:
  ```kotlin
  // składnia ogólna:
  kolekcja.map { element -> przekształcona_wartość }

  val numbers = listOf(1, 2, 3, 4, 5)
  val doubled = numbers.map { it * 2 }                // [2, 4, 6, 8, 10]

  // map może zmienić też typ elementów:
  val products = listOf(Product("Sok", 3.99), Product("Kawa", 12.99))
  val names = products.map { it.name }                // ["Sok", "Kawa"]  (List<String>)
  val prices = products.map { it.price }              // [3.99, 12.99]    (List<Double>)
  ```

- **`forEach`** — wykonuje operację dla każdego elementu (nie zwraca wartości):
  ```kotlin
  val products = listOf("Kawa", "Herbata", "Sok")
  products.forEach { println("Produkt: $it") }
  ```

- **`any` / `all` / `none`** — sprawdzają, czy warunek jest spełniony:
  ```kotlin
  val numbers = listOf(1, 2, 3, 4, 5)
  println(numbers.any { it > 4 })   // true  — przynajmniej jeden > 4
  println(numbers.all { it > 0 })   // true  — wszystkie > 0
  println(numbers.none { it > 10 }) // true  — żaden nie > 10
  ```

- **`find` / `firstOrNull`** — zwraca pierwszy element spełniający warunek lub `null`:
  ```kotlin
  val numbers = listOf(1, 2, 3, 4, 5)
  val first = numbers.find { it > 3 }       // 4
  val missing = numbers.find { it > 10 }    // null
  ```

- **`sortedBy` / `sortedByDescending`** — sortuje kolekcję według wybranej właściwości.
  Lambda zwraca klucz sortowania (wartość porównywalną, np. `Int`, `String`, `Double`):
  ```kotlin
  // składnia ogólna:
  kolekcja.sortedBy { element -> klucz_sortowania }

  data class Product(val name: String, val price: Double)
  val products = listOf(Product("Sok", 3.99), Product("Kawa", 12.99), Product("Herbata", 6.49))

  val byPrice = products.sortedBy { it.price }
  // [Sok(3.99), Herbata(6.49), Kawa(12.99)]

  val byName = products.sortedBy { it.name }
  // [Herbata, Kawa, Sok]  (alfabetycznie)

  val byPriceDesc = products.sortedByDescending { it.price }
  // [Kawa(12.99), Herbata(6.49), Sok(3.99)]
  ```

- **`groupBy`** — grupuje elementy według klucza, zwraca `Map<K, List<V>>`.
  Lambda zwraca klucz grupowania — wszystkie elementy z tym samym kluczem trafiają do jednej listy:
  ```kotlin
  // składnia ogólna:
  kolekcja.groupBy { element -> klucz_grupy }   // → Map<TypKlucza, List<TypElementu>>

  data class Product(val name: String, val category: String, val price: Double)
  val products = listOf(
      Product("Kawa", "napój", 12.99),
      Product("Herbata", "napój", 6.49),
      Product("Chleb", "pieczywo", 4.99),
      Product("Bułka", "pieczywo", 1.50)
  )

  val byCategory = products.groupBy { it.category }
  // { "napój" -> [Kawa, Herbata], "pieczywo" -> [Chleb, Bułka] }

  // dostęp do konkretnej grupy:
  val drinks = byCategory["napój"]   // [Kawa, Herbata]
  ```

- **`reduce` / `fold`** — redukuje kolekcję do jednej wartości, przetwarzając elementy po kolei.
  Lambda przyjmuje dwa argumenty: **akumulator** (`acc` — wynik dotychczasowego przetwarzania) oraz **bieżący element** (`n`).
  - `reduce` — wartość startowa akumulatora to pierwszy element kolekcji
  - `fold` — wartość startową akumulatora podaje się jawnie jako argument; działa też na pustej kolekcji
  ```kotlin
  // składnia ogólna:
  kolekcja.reduce { acc, element -> nowa_wartość_akumulatora }
  kolekcja.fold(wartość_startowa) { acc, element -> nowa_wartość_akumulatora }

  val numbers = listOf(1, 2, 3, 4, 5)

  // reduce: acc zaczyna od 1, potem: 1+2=3, 3+3=6, 6+4=10, 10+5=15
  val sum = numbers.reduce { acc, n -> acc + n }           // 15

  // fold: acc zaczyna od 100, potem: 100+1=101, ..., 110+5=115
  val sumFrom100 = numbers.fold(100) { acc, n -> acc + n } // 115

  // fold do budowania stringa:
  val text = numbers.fold("Liczby:") { acc, n -> "$acc $n" }
  // "Liczby: 1 2 3 4 5"
  ```

- **Łączenie operacji w łańcuch:**
  ```kotlin
  val numbers = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
  val result = numbers
      .filter { it % 2 == 0 }   // [2, 4, 6, 8, 10]
      .map { it * it }           // [4, 16, 36, 64, 100]
      .sum()                     // 220
  ```

**Wskazówki:**
- Domyślnie kolekcje są niemutowalne — jeśli zachodzi potrzeba modyfikacji kolekcji, należy użyć typu `MutableList`, `MutableSet` lub `MutableMap`.
- Operacje na kolekcjach (`filter`, `map` itp.) nie modyfikują oryginalnej kolekcji — zwracają nową.


## Null safety

Null safety to jedna z najważniejszych cech języka Kotlin, która pomaga unikać błędów związanych z wartością `null` (tzw. NullPointerException).

- **Domyślnie zmienne nie mogą być nullem:**
  ```kotlin
  var text: String = "Hello"
  text = null // Błąd kompilacji!
  ```

- **Aby zmienna mogła przyjmować wartość null, należy dodać znak zapytania do typu:**
  ```kotlin
  var text: String? = null
  text = "Cześć"
  ```

- **Bezpieczne wywołanie (safe call operator `?.`):**
  Pozwala wywołać metodę lub uzyskać właściwość tylko, jeśli obiekt nie jest nullem.
  ```kotlin
  println(text?.length) // jeśli text == null, wynik to null, nie ma błędu
  ```

- **Operator Elvis (`?:`):**
  Pozwala ustawić wartość domyślną, gdy zmienna jest nullem.
  ```kotlin
  val length = text?.length ?: 0 // jeśli text == null, length = 0
  ```

- **Wymuszenie nie-null (`!!`):**
  Rzuca wyjątek, jeśli zmienna jest nullem (należy stosować tylko wtedy, gdy mamy pewność, że obiekt nie jest null).
  ```kotlin
  val length = text!!.length // jeśli text == null, zostanie rzucony wyjątek NullPointerException
  ```

- **Bezpieczne rzutowanie (`as?`):**
  Zwraca null, jeśli rzutowanie się nie powiedzie.
  ```kotlin
  val obj: Any = "tekst"
  val text: String? = obj as? String  // "tekst"
  val number: Int? = obj as? Int       // null (rzutowanie się nie powiodło)
  ```

- **Przykład praktyczny:**
  ```kotlin
  fun showLength(text: String?) {
      println("Długość: ${text?.length ?: "brak tekstu"}")
  }
  showLength("Kotlin") // Długość: 6
  showLength(null)     // Długość: brak tekstu
  ```

**Wskazówki:**
- Warto unikać zmiennych nullable, jeśli nie jest to konieczne.
- Null safety sprawia, że kod jest bezpieczniejszy i łatwiejszy w utrzymaniu, co jest szczególnie ważne w aplikacjach Android.

---

## Klasy

W Kotlinie klasy są podstawowym sposobem definiowania własnych typów danych.

### Nazewnictwo klas

- Nazwy klas w Kotlinie powinny być w stylu **PascalCase** – każde słowo zaczyna się wielką literą, bez podkreśleń, np. `User`, `MainActivity`, `ProductItem`.
- Nazwa klasy powinna jasno opisywać, co reprezentuje dana klasa (np. `User`, `Order`, `LoginViewModel`).
- Dla klas danych (`data class`) należy stosować te same zasady – nazwa powinna być rzeczownikiem.
- Zaleca się unikanie skrótów i nieczytelnych nazw – kod powinien być samoopisujący się.
- Dla obiektów singleton (`object`) i obiektów companion również należy stosować PascalCase, np. `Logger`, `DatabaseHelper`.

**Przykłady dobrego nazewnictwa:**
```kotlin
class UserProfile
data class ProductItem(val name: String, val price: Double)
object NetworkManager
```
**Przykłady złego nazewnictwa:**
```kotlin
class userprofile
class product_item
object networkmanager
```
Dobre nazewnictwo klas ułatwia czytanie, testowanie i utrzymanie kodu w większych projektach Android.

- **Zwykła klasa:**
  ```kotlin
  class Person(val name: String, var age: Int)
  val jan = Person("Jan", 30)
  println(jan.name) // Jan
  jan.age = 31
  ```

- **Klasa z metodami:**
  ```kotlin
  class Calculator {
      fun add(a: Int, b: Int): Int = a + b
  }
  val calc = Calculator()
  println(calc.add(2, 3)) // 5
  ```
### Konstruktor klasy

W Kotlinie konstruktor to specjalna funkcja, która służy do tworzenia obiektów danej klasy. Najczęściej używa się **konstruktora głównego** (primary constructor), który definiuje się bezpośrednio w nagłówku klasy:

```kotlin
class Person(val name: String, var age: Int)
```
- Parametry konstruktora mogą być od razu właściwościami klasy (z `val` lub `var`), jak w przykładzie powyżej.
- Tworzenie obiektu:  
  ```kotlin
  val jan = Person("Jan", 30)
  ```

Można też zdefiniować **konstruktor dodatkowy** (secondary constructor), gdy potrzebny jest inny sposób tworzenia obiektu:

```kotlin
class Person(val name: String) {
    var age: Int = 0

    constructor(name: String, age: Int) : this(name) {
        this.age = age
    }
}
```

- Konstruktor dodatkowy używa słowa kluczowego `constructor` i zawsze musi wywołać konstruktor główny (`: this(...)`).

**Ważne:**
- Jeśli klasa nie ma żadnych właściwości ani metod, można pominąć nawiasy:
  ```kotlin
  class Pusta
  ```
- Jeśli potrzebna jest dodatkowa logika podczas tworzenia obiektu, można użyć bloku `init`:
  ```kotlin
  class Person(val name: String) {
      init {
          println("Tworzę osobę o imieniu $name")
      }
  }
  ```

## Dziedziczenie

W Kotlinie można tworzyć hierarchie klas i korzystać z dziedziczenia, aby ponownie wykorzystywać kod i rozszerzać funkcjonalność.

- Domyślnie każda klasa w Kotlinie jest **finalna** (nie można po niej dziedziczyć). Aby umożliwić dziedziczenie, należy użyć słowa kluczowego `open` przy definicji klasy i metod.

**Przykład dziedziczenia:**
```kotlin
open class Animal(val name: String) {
    open fun makeSound() {
        println("Zwierzę wydaje dźwięk")
    }
}

class Dog(name: String) : Animal(name) {
    override fun makeSound() {
        println("Hau hau!")
    }
}

val dog = Dog("Reksio")
dog.makeSound() // Hau hau!
```

- **open** – pozwala na dziedziczenie po klasie lub nadpisywanie metody.
- **override** – służy do nadpisywania metod z klasy bazowej.
- Konstruktor klasy pochodnej wywołuje konstruktor klasy bazowej za pomocą `: Base(...)`.

### Słowa kluczowe: `this` i `super`

- **`this`** — odwołanie do bieżącego obiektu (instancji klasy). Używane w ciele klasy do odwołania się do jej własnych właściwości i metod, a także w funkcjach zakresu (`apply`, `run`, `with`), gdzie odwołuje się do obiektu, na którym blok jest wykonywany:
  ```kotlin
  class Person(val name: String) {
      fun greet() = "Cześć, jestem ${this.name}" // this można tu pominąć
  }
  ```

- **`super`** — odwołanie do klasy bazowej (nadklasy). Pozwala wywołać metodę lub konstruktor rodzica, gdy klasa pochodna go nadpisuje:
  ```kotlin
  class Dog(name: String) : Animal(name) {
      override fun makeSound() {
          super.makeSound()   // wywołanie implementacji z Animal
          println("Hau hau!")
      }
  }
  ```

### Interfejsy

Interfejsy w Kotlinie definiuje się za pomocą słowa kluczowego `interface`. Interfejs może zawierać deklaracje metod (bez implementacji) oraz domyślne implementacje metod.

- Klasa może implementować dowolną liczbę interfejsów (w przeciwieństwie do dziedziczenia po klasie, które jest pojedyncze).
- Interfejsy są często używane do definiowania kontraktów, które muszą być spełnione przez klasy (np. obsługa kliknięć, komunikacja między komponentami).

**Przykład prostego interfejsu:**
```kotlin
interface Clickable {
    fun click()
}

class Button : Clickable {
    override fun click() {
        println("Kliknięto przycisk")
    }
}

val button = Button()
button.click() // Kliknięto przycisk
```

**Interfejs z domyślną implementacją:**
```kotlin
interface Greetable {
    fun greet(name: String) {
        println("Cześć, $name!")
    }
}

class User : Greetable

val user = User()
user.greet("Anna") // Cześć, Anna!
```

**Implementacja wielu interfejsów:**
```kotlin
interface Clickable { fun click() }
interface Flyable { fun fly() }

class SuperBird : Clickable, Flyable {
    override fun click() { println("Ptak kliknięty") }
    override fun fly() { println("Ptak leci") }
}
```

**Ważne cechy interfejsów w Kotlinie:**
- Interfejs może mieć właściwości (bez stanu), np. `val name: String`
- Interfejs nie może przechowywać stanu (nie może mieć pól z wartościami)
- W jednej klasie można implementować wiele interfejsów

Interfejsy są szeroko wykorzystywane w Androidzie, np. do obsługi zdarzeń (kliknięcia, callbacki), komunikacji między fragmentami, adapterami itp.

### Klasy abstrakcyjne

Klasa abstrakcyjna (`abstract class`) to klasa, której nie można bezpośrednio instancjonować — służy jako szablon dla klas pochodnych. Może zawierać zarówno metody abstrakcyjne (bez implementacji), jak i metody z implementacją, a także pola przechowujące stan.

```kotlin
abstract class Shape {
    // abstract — każda podklasa MUSI dostarczyć własną implementację;
    // Shape nie wie, jak liczyć pole (inaczej robi to koło, prostokąt, trójkąt)
    abstract fun area(): Double

    // zwykła metoda z implementacją — wspólna dla wszystkich podklas, 
    // ale można je też nadpisywać
    fun describe() {
        println("Pole powierzchni: ${area()}")
    }
}

class Circle(val radius: Double) : Shape() {
    override fun area() = Math.PI * radius * radius
}

class Square(val edge: Double) : Shape() {
    override fun area() = edge * edge
    override fun describe() {
       super.describe() // dostęp do funkcji describe z klasy nadrzędnej
       println("To jest kwadrat")
    }
}

val circle = Circle(5.0)
circle.describe() // Pole powierzchni: 78.53...

val square = Square(5.0)
square.describe()
// Pole powierzchni: 25.0
// To jest kwadrat
```

**Różnice między `abstract class` a `interface`:**

| Cecha | `abstract class` | `interface` |
|---|---|---|
| Instancjonowanie | nie można | nie można |
| Pola ze stanem | tak | nie |
| Konstruktor | tak | nie |
| Implementacja metod | tak — metody mogą być abstrakcyjne (bez ciała) lub konkretne (z ciałem) | tak — metody mogą mieć domyślną implementację, ale nie muszą; bez ciała klasa implementująca musi dostarczyć własną |
| Dziedziczenie | tylko po jednej klasie | wiele interfejsów jednocześnie |

**Kiedy stosować które rozwiązanie:**
- `abstract class` — gdy klasy pochodne mają wspólny stan (pola) lub logikę bazową; gdy hierarchia klas reprezentuje relację „jest rodzajem" (np. `Circle` jest `Shape`)
- `interface` — gdy zależy wyłącznie na zdefiniowaniu kontraktu (co klasa potrafi robić); gdy jedna klasa powinna spełniać wiele niezależnych kontraktów



##  Klasy specjalne

- **Data class** – specjalny typ klasy do przechowywania danych. Automatycznie generuje metody `equals()`, `hashCode()`, `toString()`, `copy()` i `componentN()`:
  ```kotlin
  data class Product(val name: String, val price: Double)
  val coffee = Product("Kawa", 12.99)
  println(coffee) // Product(name=Kawa, price=12.99)
  val cheaperCoffee = coffee.copy(price = 10.99)
  ```

- **object** – słowo kluczowe do tworzenia singletonów (pojedynczych instancji klas) lub obiektów anonimowych.

  * **Singleton:**
  ```kotlin
  object Logger {
      fun log(msg: String) {
          println("LOG: $msg")
      }
  }
  Logger.log("Start aplikacji")
  ```

  *  **Obiekt anonimowy** – przydatny np. do implementacji interfejsów „w locie”:
  ```kotlin
  val listener = object : Clickable {
      override fun click() {
          println("Kliknięto anonimowy obiekt")
      }
  }
  listener.click()
  ```

  * **companion object** – obiekt towarzyszący w klasie, pozwala na tworzenie statycznych metod/pól (bez konieczności tworzenia oddzielnej instancji):
  ```kotlin
  class User(val name: String) {
      companion object {
          fun createAnonymous() = User("Anonim")
      }
  }
  val anonymous = User.createAnonymous()
  ```

- **enum class** – typ wyliczeniowy, pozwala zdefiniować ograniczony zbiór stałych wartości. Przydatny np. do reprezentowania stanów, typów, kategorii:
  ```kotlin
  enum class Status {
      LOADING, SUCCESS, ERROR
  }

  val status = Status.SUCCESS
  when (status) {
      Status.LOADING -> println("Trwa ładowanie...")
      Status.SUCCESS -> println("Sukces!")
      Status.ERROR   -> println("Wystąpił błąd")
  }
  ```
    Enum w Kotlinie może mieć własne właściwości, metody oraz konstruktory. Dzięki temu enumy mogą przechowywać dodatkowe dane i zachowanie.

    ```kotlin
    enum class OrderStatus(val description: String, val isFinal: Boolean) {
        NEW("Nowe zamówienie", false),
        IN_PROGRESS("W realizacji", false),
        COMPLETED("Zrealizowane", true),
        CANCELLED("Anulowane", true); // zwróć uwagę na średnik
    }
    ```

- **sealed class / sealed interface** – klasa lub interfejs reprezentujące **zamkniętą hierarchię typów**: wszystkie podtypy muszą być zdefiniowane w tym samym pliku (w Kotlinie od wersji 1.5 wymóg ogranicza się do tego samego pakietu w obrębie tego samego modułu kompilacji). Kompilator zna więc wszystkie możliwe podtypy, dzięki czemu wyrażenie `when` może być wyczerpujące — bez konieczności dodawania gałęzi `else`.

   
  > **Moduł kompilacji** to zbiór plików Kotlina kompilowanych razem w jednym kroku (np. jeden moduł Gradle w projekcie Android). Podtyp sealed class nie może być zdefiniowany w innej bibliotece czy module — hierarchia jest zamknięta dla zewnętrznego kodu.


  ```kotlin
  sealed class Result {
      data class Success(val value: Int) : Result() // jeżeli klasa ma niepusty konstruktor  używamy class
      data class Failure(val error: String) : Result()
      data object Loading : Result() // jeżeli pusty konstruktor to object
  }

  fun handle(result: Result) {
      when (result) {
          is Result.Success -> println("Wynik: ${result.value}")
          is Result.Failure -> println("Błąd: ${result.error}")
          is Result.Loading -> println("Ładowanie...")
          // nie trzeba else — kompilator wie, że to wszystkie przypadki
      }
  }
  ```

  `sealed interface` działa analogicznie, ale klasy implementujące mogą jednocześnie dziedziczyć po innej klasie (sealed class tego nie umożliwia, bo Kotlin nie ma wielokrotnego dziedziczenia klas):

  ```kotlin
  sealed interface Result {
      data class Success(val value: Int) : Result
      data class Failure(val error: String) : Result
      data object Loading : Result
  }

  fun handle(result: Result) {
      when (result) {
          is Result.Success -> println("Wynik: ${result.value}")
          is Result.Failure -> println("Błąd: ${result.error}")
          is Result.Loading -> println("Ładowanie...")
      }
  }
  ```

  Kluczowa różnica: `data class Success` może tutaj jednocześnie dziedziczyć po innej klasie, np. `class Success(...) : SomeBaseClass(), Result`, co przy `sealed class` nie jest możliwe.

  **Porównanie `enum class` vs `sealed class`:**

  | Cecha | `enum class` | `sealed class` |
  |---|---|---|
  | Liczba instancji | stała, jedna na wartość | dowolna liczba obiektów danego podtypu |
  | Dane w wariantach | wspólna struktura dla wszystkich | każdy podtyp może mieć inne pola |
  | Podtypy | tylko stałe wyliczeniowe | pełnoprawne klasy (`data class`, `object`, klasy z logiką) |
  | Wyczerpujące `when` | tak | tak |
  | Zastosowanie | stały zbiór prostych stałych (np. `Direction.NORTH`) | hierarchia typów z różnymi danymi (np. wynik operacji, stan) |

---

## `lateinit` i `by lazy`

W Kotlinie zmienne wymagają inicjalizacji przy deklaracji. Istnieją jednak dwa mechanizmy pozwalające odłożyć inicjalizację na później.

### `lateinit`

Słowo kluczowe `lateinit` stosuje się do zmiennych `var` typów nieopcjonalnych (non-null), gdy inicjalizacja nie może nastąpić w momencie deklaracji, lecz zostanie wykonana przed pierwszym użyciem. Działa wyłącznie dla typów obiektowych (nie dla `Int`, `Boolean` itp.).

```kotlin
class UserRepository {
    lateinit var database: Database  // zainicjalizowana później, np. przez framework DI

    fun setup(db: Database) {
        database = db
    }

    fun findUser(id: Int) = database.query(id)
}
```

- Próba odczytu zmiennej `lateinit` przed inicjalizacją rzuca wyjątek `UninitializedPropertyAccessException`.
- Można sprawdzić, czy zmienna była już zainicjalizowana: `::database.isInitialized`.

### `by lazy`

Delegacja `by lazy` służy do **leniwej inicjalizacji** zmiennych `val` — wartość jest obliczana dopiero przy pierwszym odczycie, a następnie zapamiętywana.

```kotlin
val processedData: List<String> by lazy {
    println("Inicjalizacja listy...")
    listOf("A", "B", "C").map { it.lowercase() }
}

// Blok lazy nie jest jeszcze uruchomiony
println(processedData) // dopiero teraz następuje inicjalizacja
println(processedData) // drugi raz: wynik zapamiętany, blok nie uruchamia się ponownie
```

**Porównanie:**

| Cecha | `lateinit` | `by lazy` |
|---|---|---|
| Typ zmiennej | `var` | `val` |
| Inicjalizacja | ręczna, w dowolnym miejscu | automatyczna, przy pierwszym użyciu |
| Dozwolone typy | tylko obiektowe | wszystkie |
| Bezpieczeństwo wątkowe | nie | domyślnie tak |

---

## Destrukturyzacja

Destrukturyzacja pozwala rozłożyć obiekt na kilka zmiennych w jednej instrukcji. Jest dostępna dla `data class`, par (`Pair`), wpisów mapy i innych typów.

- **Destrukturyzacja `data class`:**
  ```kotlin
  data class Product(val name: String, val price: Double)
  val coffee = Product("Kawa", 12.99)

  val (name, price) = coffee
  println("$name kosztuje $price zł") // Kawa kosztuje 12.99 zł
  ```

- **Destrukturyzacja w pętli po mapie:**
  ```kotlin
  val map = mapOf("a" to 1, "b" to 2) // słowo "to" tworzy parę, np. "a" to 1 == Pair("a",1)
  for ((key, value) in map) {
      println("$key = $value")
  }
  ```

- **Pomijanie wartości znakiem `_`:**
  Jeśli któraś ze zmiennych nie jest potrzebna, można ją pominąć:
  ```kotlin
  val (_, price) = Product("Herbata", 8.50) // name pominięta
  ```

- **Destrukturyzacja w lambdach:**
  ```kotlin
  val products = listOf(Product("Kawa", 12.99), Product("Sok", 5.00))
  products.forEach { (name, price) ->
      println("$name: $price zł")
  }
  ```


---

## Funkcje zakresu

Funkcje zakresu (ang. *scope functions*) to wbudowane funkcje Kotlina, które pozwalają wykonać blok kodu w kontekście danego obiektu. Dzięki nim kod staje się bardziej zwięzły, szczególnie przy inicjalizacji obiektów i obsłudze wartości nullable.

W Kotlinie wyróżnia się pięć funkcji zakresu: `let`, `apply`, `run`, `also`, `with`. Różnią się sposobem odwoływania się do obiektu (`this` lub `it`) oraz tym, co zwracają.

| Funkcja | Odwołanie do obiektu | Zwraca |
|---|---|---|
| `let` | `it` | wynik bloku |
| `apply` | `this` | obiekt |
| `run` | `this` | wynik bloku |
| `also` | `it` | obiekt |
| `with` | `this` | wynik bloku |

### `apply` — konfiguracja obiektu

Stosowany głównie do inicjalizacji lub konfiguracji obiektu. Wewnątrz bloku obiekt jest dostępny jako `this`. Zwraca sam obiekt.

```kotlin
data class Config(var host: String = "", var port: Int = 0, var timeout: Int = 0)

val config = Config().apply {
    host = "localhost"
    port = 8080
    timeout = 30
}
```

### `let` — operacje na wartości, obsługa null

Często używany z operatorem `?.` do bezpiecznej obsługi wartości nullable. Obiekt jest dostępny jako `it`.

```kotlin
val text: String? = fetchText()
text?.let {
    println("Długość: ${it.length}") // wykonane tylko gdy text != null
}

// let do transformacji wartości:
val doubleLength = "Kotlin".let { it.length * 2 } // 12
```

### `run` — blok operacji zwracający wynik

Występuje w dwóch wariantach:

- **Na obiekcie** — podobny do `apply`, ale zwraca wynik bloku (nie obiekt). Obiekt dostępny jako `this`:
  ```kotlin
  val result = StringBuilder().run {
      append("Kotlin ")
      append("jest świetny")
      toString() // ta wartość zostanie zwrócona
  }
  println(result) // Kotlin jest świetny
  ```

- **Bez obiektu** — blok kodu wykonywany w miejscu, zwracający wartość. Przydatny do wyodrębnienia fragmentu logiki lub inicjalizacji zmiennej wymagającej kilku kroków:
  ```kotlin
  val value = run {
      val base = 10
      val factor = 3
      base * factor // wynik bloku przypisany do value
  }
  println(value) // 30
  ```

### `also` — efekty uboczne

Stosowany gdy potrzebne jest wykonanie dodatkowej operacji (np. logowanie) bez modyfikowania obiektu. Zwraca obiekt.

```kotlin
val list = mutableListOf(1, 2, 3)
    .also { println("Lista przed: $it") }
list.add(4)
```

### `with` — operacje na obiekcie bez rozszerzenia

Przyjmuje obiekt jako argument (nie jako odbiorcę wywołania). Przydatny gdy obiekt jest już znany i nie jest wynikiem wyrażenia.

```kotlin
val person = Person("Anna", 25)
with(person) {
    println("Imię: $name")
    println("Wiek: $age")
}
```

**Wskazówki — kiedy używać której funkcji:**
- `apply` — konfiguracja lub inicjalizacja obiektu.
- `let` — operacje na wartości nullable lub ograniczenie zakresu zmiennej.
- `also` — efekt uboczny (np. logowanie) bez zmiany obiektu.
- `run` / `with` — ciąg operacji na obiekcie, gdy potrzebny jest wynik bloku.

---

## Funkcje i właściwości rozszerzające

Funkcje i właściwości rozszerzające (extension functions, extension properties) pozwalają dodać nowe metody lub właściwości do istniejących klas – nawet tych, których nie można modyfikować (np. klasy z bibliotek lub Javy).

### Funkcje rozszerzające

- Definiuje się je poza klasą, poprzedzając nazwą typu, który jest rozszerzany:
  ```kotlin
  fun String.reverse(): String = this.reversed()

  val text = "Kotlin"
  println(text.reverse()) // "niltok"
  ```

- Można rozszerzać dowolny typ, także klasy Androida:
  ```kotlin
  fun Context.toast(msg: String) =
      Toast.makeText(this, msg, Toast.LENGTH_SHORT).show()
  ```

- Funkcje rozszerzające mają dostęp do publicznych metod i właściwości rozszerzanego typu przez słowo kluczowe `this`.

- Funkcje rozszerzające nie modyfikują oryginalnej klasy

### Właściwości rozszerzające

- Pozwalają dodać „pseudo-właściwości” do istniejących klas:
  ```kotlin
  val String.reversed: String
      get() = this.reversed()

  println("Android".reversed) // "diordnA"
  ```

- Właściwości rozszerzające nie mogą mieć stanu (nie można zadeklarować pola), tylko getter.




**Podsumowanie:**
- Funkcje i właściwości rozszerzające poprawiają czytelność kodu i pozwalają pisać bardziej idiomatyczne API.
- Nie modyfikują oryginalnych klas – są bezpieczne i wygodne.

---

## 📱Aplikacja pokazowa:
- [KotlinShowcase](https://github.com/MarcinRod/KotlinShowcase)

## 📚 Dokumentacja i materiały

- [Oficjalna dokumentacja Kotlin](https://kotlinlang.org/docs/home.html)
- [Kotlin for Android Developers](https://developer.android.com/kotlin)
- [Podstawy Kotlin na Android Developers](https://developer.android.com/kotlin/learn)
---
### 🧭 **Następny temat:** [Android Studio](https://github.com/MarcinRod/AndroidLecture2025/blob/main/02%20Android%20Studio.md)
