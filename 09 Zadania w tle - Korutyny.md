# Praca w tle w aplikacjach Android (Jetpack Compose i korutyny)

W nowoczesnych aplikacjach Android, także tych tworzonych z użyciem Jetpack Compose, wiele operacji (np. pobieranie danych z sieci, zapisywanie do bazy, długotrwałe obliczenia) powinno być wykonywanych **w tle**, aby nie blokować głównego wątku UI.

## Wątek UI (Main Thread) w Androidzie

Wszystkie operacje związane z wyświetlaniem interfejsu użytkownika (UI) w Androidzie muszą być wykonywane na **wątku głównym** (tzw. **UI thread** lub **Main thread**). Oznacza to, że:

- Renderowanie widoków, obsługa kliknięć, animacje i aktualizacje stanu UI muszą odbywać się na tym samym wątku.
- Wykonanie długotrwałej operacji (np. pobieranie danych z sieci, zapis do bazy) na wątku głównym powoduje, że aplikacja przestaje reagować, co może prowadzić do błędu **ANR (Application Not Responding)**.
  
---

## Dlaczego praca w tle jest ważna?

- **Interfejs użytkownika nie jest blokowany** – aplikacja pozostaje responsywna.
- **Unika się błędów ANR (Application Not Responding)** – system może zamknąć aplikację, jeśli operacje trwają zbyt długo na głównym wątku.
- **Lepsze doświadczenie użytkownika** – np. ładowanie danych z sieci nie zatrzymuje animacji czy przewijania.

---

## Korutyny

W Jetpack Compose najczęściej do realizacji pracy w tle używa się **korutyn** (Kotlin Coroutines):

- Korutyny pozwalają łatwo uruchamiać zadania asynchroniczne.
- Są lekkie i dobrze integrują się z architekturą Compose oraz ViewModel.

## Elementy systemu korutyn

### Funkcje suspend

- Funkcje oznaczone słowem kluczowym `suspend` mogą być wywoływane tylko z korutyny lub innej funkcji suspend.
- Pozwalają na wykonywanie operacji asynchronicznych w sposób sekwencyjny, bez blokowania wątku.

**Przykład:**
```kotlin
suspend fun fetchDataFromNetwork(): String {
    // pobieranie danych z sieci
    return "Dane"
}
```

### Scope (zakres korutyny)

- **Scope** zarządza korutynami – określa, do jakiej części aplikacji są one przypisane i kiedy powinny zostać anulowane.
- Najczęściej używane scope w Compose:
  - **viewModelScope** – korutyny są powiązane z ViewModelem i są anulowane, gdy `ViewModel` jest usuwany (więcej o `ViewModel` w sekcji [Architektura aplikacji](https://github.com/MarcinRod/AndroidLecture2025/blob/main/10%20Architektura%20aplikacji.md)).
  - **lifecycleScope** – korutyny są powiązane z cyklem życia komponentu (np. aktywności).
  - **rememberCoroutineScope** – scope powiązany z funkcją kompozycyjną, zarządza korutynami w jej kontekście.


### Dispatchers (wybór wątku)

- **Dispatchers** określają, na jakim wątku wykonywana jest korutyna:
  - `Dispatchers.Main` – główny wątek UI (domyślny w Compose i ViewModelu).
  - `Dispatchers.IO` – wątek do operacji wejścia/wyjścia (np. sieć, pliki, baza).
  - `Dispatchers.Default` – wątek do ciężkich obliczeń.
- Można przełączać się między dispatcherami za pomocą `withContext`.

**Przykład z `withContext`:**
```kotlin
// Funkcja suspend automatycznie przełącza kontekst — wywołujący nie musi tego robić
suspend fun loadAndProcess(): String {
    val raw = withContext(Dispatchers.IO) {
        fetchDataFromNetwork()     // operacja sieciowa na wątku IO
    }
    return withContext(Dispatchers.Default) {
        processData(raw)           // ciężkie obliczenia na wątku Default
    }
    // wynik wraca na wątek, który wywołał funkcję (np. Main)
}
```

**Podsumowanie:**
- Funkcje suspend pozwalają pisać asynchroniczny kod w prosty sposób.
- Scope zarządza cyklem życia korutyn i ich anulowaniem.
- Dispatchers wybierają odpowiedni wątek do zadania.

  
## Uruchamianie korutyn

Aby wykonać zadanie w tle, należy uruchomić korutynę w odpowiednim zakresie. Najczęściej używane sposoby:

### `launch`

- Najpopularniejsza funkcja do uruchamiania korutyn.
- Używana do zadań, które nie muszą zwracać wyniku.
- Najczęściej wywoływana w `viewModelScope`, `lifecycleScope` lub `rememberCoroutineScope`.

**Przykład w ViewModelu:**
```kotlin
viewModelScope.launch {
    val data = fetchDataFromNetwork()
    // aktualizacja stanu UI
}
```

**Przykład w composable z własnym scope:**
```kotlin
val coroutineScope = rememberCoroutineScope()
Button(onClick = {
    coroutineScope.launch {
        // zadanie w tle, np. zapis do bazy
    }
}) {
    Text("Zapisz")
}
```


### `async`

- Używana gdy zadanie ma być uruchomione równolegle i zwrócony wynik jako `Deferred`.
- Pozwala na wykonywanie wielu zadań równolegle i pobranie wyników za pomocą `await()`.
- Do grupowania równoległych zadań wewnątrz funkcji suspend zaleca się `coroutineScope { }` — tworzy izolowany scope, który anuluje wszystkie zadania podrzędne w razie błędu jednego z nich.

**Przykład:**
```kotlin
val coroutineScope = rememberCoroutineScope()
coroutineScope.launch {
    val deferred1 = async { fetchData1() }
    val deferred2 = async { fetchData2() }
    val result1 = deferred1.await()
    val result2 = deferred2.await()
    // użyj obu wyników
}
```

**Przykład z `coroutineScope { }`:**
```kotlin
suspend fun fetchBothResults(): Pair<String, String> = coroutineScope {
    val deferred1 = async { fetchData1() }
    val deferred2 = async { fetchData2() }
    deferred1.await() to deferred2.await()
}
```

## Anulowanie korutyn

Korutyna uruchomiona przez `launch` lub `async` może zostać anulowana jawnie za pomocą obiektu `Job`. Anulowanie jest **współpracujące** — korutyna musi sama sprawdzać stan anulowania (poprzez wywoływanie funkcji suspend lub jawne sprawdzanie `isActive`).

```kotlin
val job = coroutineScope.launch {
    fetchDataFromNetwork()
}
// jawne anulowanie korutyny:
job.cancel()
```

W długo działających pętlach lub obliczeniach, w których nie są wywoływane żadne funkcje suspend, należy samodzielnie sprawdzać `isActive`:

```kotlin
coroutineScope.launch {
    for (item in largeList) {
        if (!isActive) break   // korutyna została anulowana — przerywamy pętlę
        process(item)
    }
}
```

> **`CancellationException`** — anulowanie korutyny wewnętrznie rzuca `CancellationException`. Jest to normalne zachowanie, które nie oznacza błędu aplikacji. W bloku `catch` nie należy tłumić tego wyjątku:
> ```kotlin
> coroutineScope.launch {
>     try {
>         doLongWork()
>     } catch (e: Exception) {
>         if (e is CancellationException) throw e   // anulowanie musi się propagować
>         handleError(e)
>     }
> }
> ```

---

## Efekty uboczne (Side Effects)

W Jetpack Compose **efekty uboczne** (ang. side effects) to operacje wykonywane poza procesem kompozycji — np. uruchomienie korutyny, rejestracja listenera, logowanie czy nawigacja. Compose udostępnia dedykowane funkcje kompozycyjne zapewniające, że efekty są wykonywane w odpowiednim momencie i są poprawnie czyszczone.


### `LaunchedEffect`

- Specjalna funkcja kompozycyjna w Compose, która uruchamia korutynę przy starcie lub po zmianie klucza.
- Idealna do inicjalizacji danych lub reagowania na zmiany parametrów.

**Przykład:**
```kotlin
@Composable
fun MyScreen() {
    LaunchedEffect(Unit) {
        loadData()
    }
    // UI...
}
```
`LaunchedEffect` uruchamia korutynę, gdy funkcja kompozycyjna pojawia się na ekranie **lub** gdy zmieni się wartość klucza (key). Jeśli jako klucz podana jest wartość `Unit`, efekt wykona się tylko raz przy pierwszym wyświetleniu. Jeśli podana zostanie inna wartość (np. identyfikator, stan), efekt wykona się za każdym razem, gdy ta wartość się zmieni.

**Przykład:**
```kotlin
@Composable
fun UserScreen(userId: String) {
    LaunchedEffect(userId) {
        loadUser(userId)
    }
    // UI...
}
```
W tym przykładzie:
- Jeśli `userId` się zmieni (np. użytkownik wybierze inny profil), `LaunchedEffect` ponownie uruchomi korutynę i załaduje dane nowego użytkownika.
- Gdyby jako klucza użyto `Unit`, efekt wykonałby się tylko raz, niezależnie od zmian `userId`.

---

> **`rememberCoroutineScope` vs `LaunchedEffect`**  
> `LaunchedEffect` uruchamia korutynę automatycznie — przy wejściu funkcji kompozycyjnej w kompozycję lub przy zmianie klucza. Służy do efektów, które mają się wykonać "same z siebie" (np. załadowanie danych przy starcie ekranu).  
> `rememberCoroutineScope` daje dostęp do scope, w którym korutynę uruchamia się ręcznie — np. w reakcji na kliknięcie przycisku. Scope jest powiązany z cyklem życia funkcji kompozycyjnej i jest anulowany gdy opuszcza ona kompozycję.
> 
### `DisposableEffect` — efekt z czyszczeniem zasobów

Służy do efektów, które wymagają jawnego sprzątania po sobie — np. rejestracja i wyrejestrowanie listenera. Blok `onDispose` jest wywoływany gdy funkcja kompozycyjna opuszcza kompozycję lub gdy zmienia się klucz.

```kotlin
@Composable
fun LocationTracker(onLocationUpdate: (Location) -> Unit) {
    val context = LocalContext.current

    DisposableEffect(Unit) {
        val manager = context.getSystemService(LocationManager::class.java)
        val listener = LocationListener { location -> onLocationUpdate(location) }

        manager.requestLocationUpdates(LocationManager.GPS_PROVIDER, 0L, 0f, listener)

        onDispose {
            manager.removeUpdates(listener)  // sprzątanie przy wyjściu z kompozycji
        }
    }
}
```


### Powiązanie efektów z cyklem życia

Efekty Compose są powiązane z cyklem życia **kompozycji**, nie z cyklem życia aktywności. Oznacza to:

| Zdarzenie | `LaunchedEffect` / `DisposableEffect` |
|---|---|
| Funkcja kompozycyjna wchodzi w kompozycję | Efekt uruchamiany |
| Zmiana klucza | Poprzedni efekt anulowany/czyszczony, nowy uruchamiany |
| Funkcja kompozycyjna opuszcza kompozycję | Efekt anulowany / `onDispose` wywoływany |
| Rotacja ekranu (rekreacja Activity) | Efekt anulowany i uruchomiony ponownie |

Gdy ekran jest zasłonięty przez inny (np. dialog), funkcja kompozycyjna pozostaje w kompozycji — efekty **nie** są anulowane. 

#### Obserwowanie zdarzeń cyklu życia aktywności przez `DisposableEffect`

Gdy funkcja kompozycyjna musi reagować na zdarzenia cyklu życia aktywności (np. `onResume`, `onPause`), można to osiągnąć przez `DisposableEffect` z `LifecycleEventObserver`:

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

- `LocalLifecycleOwner.current` — dostarcza `LifecycleOwner` (najczęściej bieżącą aktywność) z drzewa kompozycji.
- Observer jest rejestrowany gdy funkcja kompozycyjna wchodzi w kompozycję i wyrejestrowany w `onDispose` — nie pozostawia wycieków przy rotacji ani nawigacji.



---

## Obsługa błędów w korutynach

Wyjątki rzucone wewnątrz korutyny nie propagują się automatycznie do wątku wywołującego — należy je obsługiwać jawnie.

### `try/catch` — zalecane podejście

Najprostszy sposób: opakowanie kodu korutyny w blok `try/catch`. Sprawdza się w ViewModelu przy aktualizacji stanu UI.

```kotlin

coroutineScope.launch {
    try {
        val result = fetchDataFromNetwork()  
    } catch (e: Exception) {
      // obsłuż wyjątek
    }
}
```

---


## Praca w tle a długotrwałe zadania

- Do bardzo długich zadań (np. synchronizacja w tle, powiadomienia) należy używać **WorkManager** lub **Foreground Service**.
- Korutyny są idealne do krótkich operacji powiązanych z życiem ekranu.



## Zalecenia i dobre praktyki  

- **Korutyny należy uruchamiać w odpowiednim zakresie**  
  Dzięki temu korutyny są automatycznie anulowane, gdy komponent (ViewModel, funkcja kompozycyjna, aktywność) przestaje istnieć. Zapobiega to wyciekom pamięci i niepotrzebnemu wykonywaniu zadań w tle.

- **Należy wybierać odpowiedni dyspozytor**  
  Do operacji wejścia/wyjścia (sieć, pliki, baza) należy używać `Dispatchers.IO`, do ciężkich obliczeń `Dispatchers.Default`, a do aktualizacji UI `Dispatchers.Main`.

- **Nie należy blokować wątku głównego**  
  Długotrwałe operacje należy zawsze wykonywać w tle (np. z użyciem `withContext(Dispatchers.IO)`).

- **Zaleca się stosowanie funkcji suspend do operacji asynchronicznych**  
  Pozwala to pisać czytelny, sekwencyjny kod bez blokowania wątków.

- **Należy używać `LaunchedEffect` do inicjalizacji i reagowania na zmiany parametrów w funkcji kompozycyjnej**  
  Dzięki temu zadanie wykona się tylko wtedy, gdy jest to potrzebne.

  
## Więcej informacji

- [Kotlin Coroutines – Android Developers](https://developer.android.com/kotlin/coroutines)
- [WorkManager – Android Developers](https://developer.android.com/topic/libraries/architecture/workmanager)
- [Praca w tle w Compose – dokumentacja](https://developer.android.com/jetpack/compose/side-effects#launchedeffect)

---

### **Następny temat:** [Architektura aplikacji](https://github.com/MarcinRod/AndroidLecture2025/blob/main/10%20Architektura%20aplikacji.md)