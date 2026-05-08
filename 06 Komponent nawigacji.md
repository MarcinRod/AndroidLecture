# Komponent nawigacji w Jetpack Compose


Jetpack Compose oferuje nowoczesny sposób zarządzania nawigacją w aplikacjach Android dzięki bibliotece **Navigation Compose**. Pozwala ona w prosty sposób przechodzić między ekranami (tzw. composable destinations) oraz przekazywać dane pomiędzy nimi.

---
## Stos nawigacji (Back Stack) w Navigation Compose

Navigation Compose zarządza tzw. **stosem nawigacji** (back stack), czyli listą ekranów, które użytkownik odwiedził. Dzięki temu można łatwo obsługiwać powrót do poprzednich ekranów oraz kontrolować historię nawigacji.

### Jak działa stos?

- Każde przejście do nowego ekranu dodaje nową destynację na szczyt stosu.
- Powrót usuwa ostatni ekran ze stosu i pokazuje poprzedni.
- Można usuwać wiele ekranów naraz lub czyścić stos do wybranej destynacji.


>**Wskazówka:**  
Zrozumienie działania stosu jest kluczowe dla poprawnej obsługi nawigacji i przewidywalnego zachowania przycisku "wstecz" w aplikacji.

---

## Podstawowe elementy nawigacji


- **NavGraph**  
  Skierowany graf reprezentujący strukturę nawigacji aplikacji — wszystkie dostępne ekrany (wierzchołki) oraz możliwe przejścia między nimi. Definiuje się go deklaratywnie przez dodawanie destynacji (`composable { }`) i podgrafów (`navigation { }`).

- **NavHost**  
  Funkcja kompozycyjna odpowiedzialna za wyświetlanie aktualnie aktywnej destynacji. Wymaga wskazania destynacji startowej i definicji grafu nawigacji.

  **Przykład:**
  ```kotlin
  NavHost(navController = navController, startDestination = "ekranA") {
      composable("ekranA") { ScreenA() }
      composable("ekranB") { ScreenB() }
  }
  ```

- **NavController**  
  Obiekt sterujący nawigacją — zarządza stosem ekranów, umożliwia przechodzenie do nowych destynacji i powrót do poprzednich. Tworzony przez funkcję `rememberNavController()`.

  **Przykład:**
  ```kotlin
  val navController = rememberNavController()
  ```

- **Composable destinations**  

  Poszczególne ekrany aplikacji rejestrowane w grafie nawigacji. Każda destynacja jest identyfikowana przez unikalny klucz i powiązana z konkretną funkcją kompozycyjną.

  **Przykład:**
  ```kotlin
  composable("ekranA") { ScreenA() }
  composable("ekranB") { ScreenB() }
  ```

**Podsumowanie:**  
`NavGraph` definiuje dostępne ekrany i powiązania między nimi, `NavHost` go wyświetla, `NavController` steruje przejściami, a composable destinations to poszczególne ekrany zarejestrowane w grafie. Dzięki temu nawigacja w Compose jest przejrzysta, deklaratywna i łatwa do rozbudowy.

---

## Przykład podstawowej konfiguracji

1. **Dodaj zależność w pliku `build.gradle.kts`:**
   ```kotlin
   implementation("androidx.navigation:navigation-compose:2.7.7")
   ```

2. **Tworzenie NavController i NavHost:**
   ```kotlin
   val navController = rememberNavController()
   NavHost(navController = navController, startDestination = "ekranA") {
       composable("ekranA") { ScreenA() }
       composable("ekranB") { ScreenB() }
   }
   ```

3. **Przechodzenie między ekranami:**
   ```kotlin
   NavHost(navController = navController, startDestination = "ekranA") {
       composable("ekranA") {
           ScreenA(onNavigate = { navController.navigate("ekranB") })
       }
       composable("ekranB") { ScreenB() }
   }
   ```

---
## Przechodzenie między ekranami – podstawowe metody NavController

Do nawigacji między ekranami w Jetpack Compose służy obiekt `NavController`. Oto najważniejsze metody, które warto znać:

### 1. Przechodzenie do innego ekranu

Do przejścia do innej destynacji służy metoda `navigate`:

```kotlin
navController.navigate("ekranB")
```


### 2. Powrót do poprzedniego ekranu

Do powrótu do poprzedniego ekranu (poprzedniej destynacji na stosie) służy:

```kotlin
navController.popBackStack()
```

### 3. Powrót do konkretnej destynacji

Można wrócić do konkretnego ekranu (np. do ekranu głównego), usuwając wszystkie ekrany powyżej:

```kotlin
navController.popBackStack("ekranA", inclusive = false)
```
- Jeśli `inclusive = true`, także ekran o nazwie `"ekranA"` zostanie usunięty ze stosu.

### 4. Usuwanie ekranu z historii po przejściu (singleTop)

Aby uniknąć duplikowania ekranów na stosie, można użyć opcji `launchSingleTop`:

```kotlin
navController.navigate("ekranA") {
    launchSingleTop = true
}
```

### 5. Przechodzenie z czyszczeniem stosu (np. po logowaniu)

Aby przejść do ekranu i usunąć wszystkie poprzednie ekrany (np. po zalogowaniu):

```kotlin
navController.navigate("ekranStartowy") {
    popUpTo("ekranLogowania") { inclusive = true }
}
```



Dzięki tym metodom można w pełni zarządzać przepływem użytkownika w aplikacji Compose.


### Dobre praktyki: gdzie trzymać NavController?

W nawigacji Compose obowiązuje ważna zasada: **`NavController` należy trzymać wyłącznie na najwyższym poziomie drzewa composable**, a do ekranów przekazywać wyłącznie lambdy nawigacyjne.

```kotlin
// Zalecane — ekran nie zna NavController
@Composable
fun LoginScreen(onLoginSuccess: () -> Unit) {
    Button(onClick = onLoginSuccess) { Text("Zaloguj") }
}

// W NavHost (jedyne miejsce, gdzie NavController jest dostępny):
composable("LoginScreen") {
    LoginScreen(
        onLoginSuccess = {
            navController.navigate("ekranA") {
                popUpTo("ekranLogowania") { inclusive = true }
            }
        }
    )
}
```

Przekazywanie `navController` bezpośrednio do ekranów utrudnia testowanie, ponowne użycie ekranów i powoduje niepotrzebne zależności między warstwami.

**Podsumowanie:**  
- `navigate(route)` – przejście do ekranu
- `popBackStack()` – powrót do poprzedniego ekranu
- `popBackStack(route, inclusive)` – powrót do konkretnej destynacji
- Opcje nawigacji (`launchSingleTop`, `popUpTo`) pozwalają kontrolować stos nawigacji
---
## Definiowanie destynacji nawigacji

### Zalecane: type-safe navigation (Navigation 2.8+)

Od wersji Navigation 2.8 dostępne jest tzw. **type-safe navigation** oparte na `kotlinx-serialization`. Destynacje często definiuje się w dedykowanym interfejsie (`sealed interface`) jako `@Serializable data object` lub `@Serializable data class` — argumenty są zwykłymi polami klasy, bez ręcznych szablonów tras i `navArgument`.

**Dodatkowe zależności w `build.gradle.kts`:**
```kotlin
plugins {
    kotlin("plugin.serialization") version "2.0.0"
}

dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.7.3")
    implementation("androidx.navigation:navigation-compose:2.8.0")
}
```

**Definiowanie destynacji:**
```kotlin
sealed interface Destinations {
    @Serializable
    data object ListSelection : Destinations

    @Serializable
    data class ShoppingList(val listId: String? = null) : Destinations
}
```

**Konfiguracja NavHost i nawigacja:**
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

### Podejście tradycyjne (string-based routes)

W starszych projektach i przykładach często spotykane są dwa warianty:

**1. Enum class** — prosta iteracja, jednolita struktura:

```kotlin
enum class Destinations(val route: String) {
    ScreenA("ekranA"),
    ScreenB("ekranB"),
    Details("szczegoly/{id}")
}
```
*Zaleta:* dostęp do wszystkich destynacji przez `Destinations.entries` — przydatne przy budowie `BottomBar`.  
*Wada:* każdy wpis musi być zgodny z jednym konstruktorem; obsługa argumentów wymaga nadpisywania metod.

**2. Sealed class** — bardziej elastyczna, zalecana gdy ekrany różnią się parametrami:

```kotlin
sealed class Destinations(val baseRoute: String) {
    object ScreenA : Destinations("ekranA")
    object ScreenB : Destinations("ekranB")
    object Details : Destinations("szczegoly") {
        val routeTemplate = "szczegoly/{id}"
        fun createRoute(id: String) = "szczegoly/$id"
        const val ARG_ID = "id"
    }
}
```

*Zaleta:* poszczególne destynacje mogą mieć zróżnicowaną strukturę; wyczerpywalność `when` wymusza obsługę nowych ekranów.  
*Wada:* konieczność ręcznego definiowania `routeTemplate`, `navArgument` i `createRoute`.



### Przekazywanie argumentów (podejście tradycyjne)

W podejściu type-safe argumenty są polami `data class` — nie ma potrzeby definiowania `navArgument` ani szablonów tras. Poniższy opis dotyczy podejścia tradycyjnego (string-based), które nadal spotyka się w starszych projektach.

Dane między ekranami można przekazywać na dwa sposoby:

#### 1. Argumenty w ścieżce (path arguments)

Argumenty przekazywane w ścieżce są częścią trasy, np. `"szczegoly/{id}"`. Są one wymagane i muszą być zawsze podane podczas nawigacji.

**Definicja sealed class:**
```kotlin
sealed class Destinations(val route: String) {
    object ScreenA : Destinations("ekranA")
    object Details : Destinations("szczegoly/{id}") {
        fun createRoute(id: String) = "szczegoly/$id"
        const val ARG_ID = "id"
    }
}
```

**Konfiguracja NavHost:**
```kotlin
NavHost(navController, startDestination = Destinations.ScreenA.route) {
    composable(Destinations.ScreenA.route) { ScreenA() }
    composable(
        route = Destinations.Details.route,
        arguments = listOf(navArgument(Destinations.Details.ARG_ID) { type = NavType.StringType })
    ) { backStackEntry ->
        val id = backStackEntry.arguments?.getString(Destinations.Details.ARG_ID) ?: ""
        SzczegolyEkran(id)
    }
}

// Przejście z argumentem:
navController.navigate(Destinations.Details.createRoute("123"))
```

#### 2. Argumenty jako query (opcjonalne, po znaku `?`)

Argumenty query są przekazywane po znaku zapytania w ścieżce, np. `"ekranB?imie={imie}"`. Mogą być opcjonalne i mieć wartości domyślne.

**Definicja sealed class:**
```kotlin
sealed class Destinations(val route: String) {
    object ScreenA : Destinations("ekranA")
    object ScreenB : Destinations("ekranB?imie={imie}") {
        fun createRoute(imie: String?) =
            if (imie != null) "ekranB?imie=$imie" else "ekranB"
        const val ARG_IMIE = "imie"
    }
}
```

**Konfiguracja NavHost:**
```kotlin
NavHost(navController, startDestination = Destinations.ScreenA.route) {
    composable(Destinations.ScreenA.route) { ScreenA() }
    composable(
        route = Destinations.ScreenB.route,
        arguments = listOf(navArgument(Destinations.ScreenB.ARG_IMIE) {
            type = NavType.StringType
            defaultValue = "Gość"
            nullable = true
        })
    ) { backStackEntry ->
        val imie = backStackEntry.arguments?.getString(Destinations.ScreenB.ARG_IMIE)
        ScreenB(imie)
    }
}

// Przejście bez argumentu:
navController.navigate(Destinations.ScreenB.createRoute(null))
// Przejście z argumentem:
navController.navigate(Destinations.ScreenB.createRoute("Anna"))
```



### Porównanie podejść:

| | Type-safe (zalecane) | Tradycyjne (string-based) |
|---|---|---|
| Definicja destynacji | `@Serializable data object/class` | `sealed class` lub `enum` z `val route: String` |
| Rejestracja w NavHost | `composable<Type>` | `composable("route")` |
| Nawigacja z argumentem | `navigate(Dest.Screen(arg))` | `navigate("route/$arg")` |
| Odczyt argumentów | `backStackEntry.toRoute<T>()` | `backStackEntry.arguments?.getString(key)` |
| Bezpieczeństwo typów | pełne | częściowe (możliwe literówki w łańcuchach) |

---

## Zagnieżdżanie grafów nawigacji (Nested Navigation Graphs)

W większych aplikacjach Jetpack Compose często stosuje się **zagnieżdżone grafy nawigacji** (nested navigation graphs), aby lepiej organizować trasy i zarządzać złożonymi przepływami ekranów. Pozwala to na podział aplikacji na logiczne moduły (np. logowanie, główny ekran, ustawienia), z których każdy ma własny podgraf nawigacji.

### Jak zdefiniować zagnieżdżony graf?

W podejściu type-safe każdy podgraf jest identyfikowany przez obiekt destynacji oznaczony `@Serializable`. Funkcja `navigation<T>` przyjmuje typ obiektu-trasy grafu zamiast łańcucha znaków:

**Definicja destynacji:**
```kotlin
// Obiekty-klucze podgrafów (nie są ekranami — tylko identyfikatorami)
@Serializable data object AuthGraph
@Serializable data object MainGraph

// Ekrany podgrafu logowania
@Serializable data object Login
@Serializable data object Register

// Ekrany podgrafu głównego
@Serializable data object Home
@Serializable data class Settings(val userId: String)
```

Klucze podgrafów (`AuthGraph`, `MainGraph`) nie są ekranami — identyfikują jedynie grupę destynacji. Ekrany mogą być prostymi `data object` (bez argumentów) lub `data class` (z argumentami jako pola).

**Konfiguracja NavHost:**
```kotlin
NavHost(navController, startDestination = AuthGraph) {
    // Podgraf logowania
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
    // Podgraf główny aplikacji
    navigation<MainGraph>(startDestination = Home) {
        composable<Home> { HomeScreen() }
        composable<Settings> { backStackEntry ->
            val dest = backStackEntry.toRoute<Settings>()
            SettingsScreen(userId = dest.userId)
        }
    }
}
```

### Przechodzenie między podgrafami

Nawigacja do ekranu w innym podgrafie odbywa się identycznie jak zwykła nawigacja — przez obiekt destynacji. Nie ma potrzeby podawania pełnych ścieżek:

```kotlin
// Po zalogowaniu: przejście do głównego grafu i wyczyszczenie stosu logowania
navController.navigate(Home) {
    popUpTo(AuthGraph) { inclusive = true }
}
```


### Zalety zagnieżdżonych grafów

- **Lepsza organizacja kodu** – każdy moduł ma własny, czytelny podgraf.
- **Łatwiejsze zarządzanie przepływem** – stos logowania można wyczyścić po zalogowaniu przez `popUpTo(AuthGraph)` i przejść do głównego grafu.
- **Możliwość ponownego użycia** – ten sam podgraf ustawień można osadzić w różnych miejscach aplikacji.

**Więcej o zagnieżdżaniu grafów:**  
[Oficjalna dokumentacja – Nested Navigation Graphs](https://developer.android.com/jetpack/compose/navigation#nested-nav)

---

## Deep Links

**Deep link** to mechanizm pozwalający otworzyć konkretny ekran aplikacji z zewnątrz — np. z powiadomienia, widżetu, skrótu systemowego lub innej aplikacji. Rejestruje się go bezpośrednio przy destynacji w `NavHost`, podając wzorzec URI:

```kotlin
composable<Destinations.ShoppingList>(
    deepLinks = listOf(navDeepLink { uriPattern = "myapp://lista/{listId}" })
) { backStackEntry ->
    val dest = backStackEntry.toRoute<Destinations.ShoppingList>()
    ShoppingListScreen(listId = dest.listId ?: return@composable)
}
```

Aby system Android mógł przekierować do aplikacji po kliknięciu w link, wymagany jest odpowiedni wpis w `AndroidManifest.xml`:

```xml
<activity android:name=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="myapp" android:host="lista" />
    </intent-filter>
</activity>
```

> Praktyczne zastosowanie deep links (m.in. otwieranie ekranów z powiadomień) pokazane jest w aplikacji pokazowej dotyczącej architektury: [ReminderShowcase](https://github.com/MarcinRod/ReminderShowcase) .

---

## Zalety Navigation Compose

- **Deklaratywne zarządzanie trasami i ekranami**  
  Wszystkie trasy i ekrany definiowane są w jednym miejscu, co ułatwia czytanie i utrzymanie kodu.

- **Type-safe navigation (Navigation 2.8+)**  
  Destynacje jako `@Serializable data object/class` eliminują literówki w nazwach tras i ręczne definiowanie `navArgument`. Argumenty to zwykłe pola klasy — odczytywane przez `toRoute<T>()`.

- **Przekazywanie argumentów w podejściu tradycyjnym**  
  W starszym, string-based API argumenty można przekazywać w ścieżce (path) lub jako query, z możliwością wymuszenia typu przez `navArgument` i ustawienia wartości domyślnych.

- **Obsługa stosu nawigacji**  
  Navigation Compose automatycznie zarządza historią ekranów (back stack), co pozwala na intuicyjne korzystanie z przycisku "wstecz" i kontrolowanie przepływu użytkownika.

- **Integracja z animacjami i typowymi elementami nawigacji**  
  Można połączyć nawigację z animacjami przejść, dolnym paskiem nawigacji (BottomNavigation), zakładkami (Tabs) czy boczną szufladą (Drawer).

- **Wsparcie dla zagnieżdżonych grafów**  
  Aplikację można organizować w logiczne moduły i zarządzać złożonymi przepływami ekranów dzięki nested navigation graphs.

- **Łatwa integracja z architekturą aplikacji**  
  Navigation Compose dobrze współpracuje z ViewModelami, State Hoisting i innymi wzorcami architektonicznymi Compose.

---

Navigation Compose to nowoczesny i zalecany sposób obsługi nawigacji w aplikacjach Jetpack Compose. Od wersji 2.8 type-safe navigation eliminuje potrzebę ręcznego zarządzania łańcuchami tras.

---

## Dokumentacja

- [Oficjalna dokumentacja Navigation Compose](https://developer.android.com/jetpack/compose/navigation)
- [Type-safe navigation – Migration Guide](https://developer.android.com/guide/navigation/design/type-safety)
- [Przykłady na GitHub](https://github.com/android/compose-samples/tree/main/NavigationAdvancedSample)

---
### **Następny temat:** [Intencje](https://github.com/MarcinRod/AndroidLecture2025/blob/main/07%20Intencje.md)