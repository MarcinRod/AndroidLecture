# Funkcje kompozycyjne w Jetpack Compose

## Co to są funkcje kompozycyjne?

**Funkcje kompozycyjne** (ang. *composable functions*) to podstawowy budulec interfejsu użytkownika w Jetpack Compose. Są to funkcje w języku Kotlin oznaczone adnotacją `@Composable`, które opisują, jak powinien wyglądać fragment UI.

---

## Nazewnictwo funkcji kompozycyjnych

- Funkcje kompozycyjne powinny mieć nazwy w stylu **PascalCase** (każde słowo z wielkiej litery, bez podkreśleń), np. `UserCard`, `LoginScreen`, `Greeting`.
- Nazwa funkcji powinna jasno wskazywać, **co dana funkcja wyświetla lub realizuje** – należy unikać nazw ogólnych typu `MyComposable` czy `TestFunction`.
- Jeśli funkcja kompozycyjna reprezentuje cały ekran, jej nazwa powinna kończyć się słowem `Screen`, np. `ProfileScreen`, `SettingsScreen`.
- Dla mniejszych, wielokrotnego użytku elementów zalecane jest stosowanie nazw opisujących ich rolę, np. `UserAvatar`, `ProductItem`, `ErrorMessage`.
- Jeśli funkcja jest **prywatna dla pliku** (nie powinna być używana poza plikiem), można dodać prefix `_`, np. `_UserAvatar`. To konwencja Compose, która sygnalizuje, że funkcja jest "wewnętrzna".
- Należy unikać skrótów i nieczytelnych nazw – kod Compose powinien być samoopisujący się.
- Nazwy funkcji kompozycyjnych nie powinny zawierać czasowników w trybie rozkazującym (np. `ShowUser`), lecz raczej rzeczowniki lub rzeczowniki z przymiotnikami (`UserCard`, `ErrorDialog`).

**Przykłady dobrego nazewnictwa:**
- `LoginScreen`
- `ProfileHeader`
- `ProductListItem`
- `_UserAvatar` (funkcja prywatna)
- `ErrorSnackbar`

**Przykłady złego nazewnictwa:**
- `doLoginComposable`
- `myComposable1`
- `ShowProfile`
- `testFun`

Dobre nazewnictwo ułatwia czytanie, testowanie i ponowne wykorzystanie kodu w większych projektach Compose.

---

## Rola modyfikatora (`Modifier`)

- **Modifier** to specjalny obiekt w Compose, który pozwala modyfikować wygląd, rozmiar, pozycję i zachowanie elementów UI.
- Modifier jest przekazywany jako parametr do większości funkcji kompozycyjnych i umożliwia "nakładanie" wielu efektów w łańcuchu wywołań.
- Każda funkcja kompozycyjna przyjmujący `Modifier` powinien mieć go jako pierwszy parametr domyślny, np.:
  ```kotlin
  @Composable
  fun MyButton(
      onClick: () -> Unit,
      modifier: Modifier = Modifier
  ) {
      Button(onClick = onClick, modifier = modifier) {
          Text("Kliknij mnie")
      }
  }
  ```
- Dzięki temu możliwe jest łatwe łączenie modyfikatorów i przekazywanie ich z zewnątrz, np.:
  ```kotlin
  MyButton(
      onClick = { /* ... */ },
      modifier = Modifier
          .padding(16.dp)
          .fillMaxWidth()
  )
  ```

###  Najczęściej używane funkcje Modifiera

- `padding(...)` – dodaje odstęp wewnętrzny wokół elementu.
- `fillMaxWidth()`, `fillMaxHeight()`, `fillMaxSize()` – rozciąga element do maksymalnej szerokości/wysokości/rozmiaru rodzica.
- `size(...)`, `width(...)`, `height(...)` – ustawia rozmiar elementu.
- `background(color)` – ustawia tło elementu.
- `clickable { ... }` – obsługuje kliknięcia na dowolnym elemencie.
- `weight(...)` – rozdziela przestrzeń w Row/Column proporcjonalnie.
- `offset(...)` – przesuwa element względem jego pozycji.
- `align(...)` – ustawia wyrównanie w kontenerze (np. w Box).
- `clip(shape)` – przycina obszar rysowania do podanego kształtu (np. `CircleShape`, `RoundedCornerShape`). Stosowany przed `background()`, aby wypełnienie respektowało kształt.
- `border(...)` – rysuje obrys wokół elementu.
- `wrapContentWidth()`, `wrapContentHeight()` – dopasowuje rozmiar do zawartości.
- `verticalScroll(rememberScrollState())` – umożliwia przewijanie zawartości kontenera w pionie (stosowane na `Column` lub innym kontenerze).

**Przykład łączenia modyfikatorów:**
```kotlin
Text(
    text = "Przykład",
    modifier = Modifier
        .padding(8.dp)
        .background(Color.LightGray)
        .fillMaxWidth()
        .clickable { /* obsługa kliknięcia */ }
)
```

> **Wskazówka — kolejność modyfikatorów ma znaczenie:**
>
> ```kotlin
> // A: padding → background — obszar padding NIE jest pokolorowany
> Text("A", modifier = Modifier.padding(12.dp).background(Color.Yellow))
>
> // B: background → padding — tło pokrywa obszar padding
> Text("B", modifier = Modifier.background(Color.Yellow).padding(12.dp))
> ```
> Obie linie produkują różny wynik wizualny. W przypadku A padding leży „na zewnątrz" tła; w przypadku B tło obejmuje również obszar padding.

Modyfikatory pozwalają budować elastyczne, responsywne i interaktywne UI w Compose bez konieczności dziedziczenia po widokach czy stosowania złożonych układów.

---

## Stan, kompozycja i rekompozycja

### Kompozycja

**Kompozycja** (ang. *composition*) to proces, w którym środowisko wykonawcze Compose wywołuje funkcje oznaczone adnotacją `@Composable` i buduje na ich podstawie drzewo elementów UI. Kompozycja zachodzi tylko raz przy pierwszym renderowaniu ekranu.

### Stan

**Stan** (ang. *state*) to każda wartość, której zmiana może powodować aktualizację UI. Compose automatycznie śledzi, które funkcje kompozycyjne odczytują dany stan, i powtarza ich wywołanie po każdej zmianie tej wartości.

#### `mutableStateOf`

Tworzy obiekt przechowujący wartość, którego zmiany są automatycznie śledzone przez Compose. Każda modyfikacja wartości powoduje rekompozycję fragmentów UI, które tę wartość odczytują.

```kotlin
var count = mutableStateOf(0)   // bez remember — wartość zostanie zresetowana przy rekompozycji
```

#### `remember`

Przechowuje wartość **wewnątrz kompozycji** — wartość przeżywa kolejne rekompozycje, lecz jest tracona przy zniszczeniu funkcji kompozycyjnej (np. przy zmianie konfiguracji).

```kotlin
var count by remember { mutableStateOf(0) }   // wartość przeżywa rekompozycję
```

Oba mechanizmy stosowane są łącznie: `remember { mutableStateOf(...) }`.

> **Delegacja przez `by`**  
> Kotlin udostępnia operator delegacji `by`, który pozwala odczytywać i zapisywać wartość stanu bezpośrednio — bez ręcznego wywoływania `.value`:
> ```kotlin
> // bez by — dostęp przez .value
> val count = remember { mutableStateOf(0) }
> count.value++
> Text("${count.value}")
>
> // z by — bezpośredni dostęp
> var count by remember { mutableStateOf(0) }
> count++
> Text("$count")
> ```
> Użycie `by` wymaga importu `getValue`/`setValue` z pakietu `androidx.compose.runtime`.

#### `rememberSaveable`

Działa jak `remember`, ale dodatkowo zapisuje wartość w `Bundle` — stan przeżywa **zmianę konfiguracji** (np. obrót ekranu). Omówiono szerzej w rozdziale o aktywności.

```kotlin
var text by rememberSaveable { mutableStateOf("") }
```

### Rekompozycja

**Rekompozycja** (ang. *recomposition*) to ponowne wywołanie funkcji kompozycyjnych w odpowiedzi na zmianę stanu. Compose identyfikuje, które funkcje kompozycyjne zależą od zmienionego stanu, i przerysowuje **tylko je** — nie cały ekran.

```
Zmiana stanu → Compose wykrywa zależności → rekompozycja minimalnego podzbioru UI
```

Kluczowe właściwości rekompozycji:
- Jest **selektywna** — dotyczy wyłącznie funkcji kompozycyjnych odczytujących zmieniony stan.
- Może zachodzić **wielokrotnie** — funkcje kompozycyjne powinny tylko opisywać wygląd UI na podstawie przekazanych danych, bez wykonywania operacji takich jak zapis do bazy czy wywołanie sieciowe. Takie operacje mogą być wywołane wielokrotnie i w nieprzewidywalnej kolejności.
- Compose może **pominąć** rekompozycję funkcji, jeśli jej parametry nie uległy zmianie (*smart recomposition*).

### Efekty uboczne

Gdy konieczne jest wykonanie operacji poza kompozycją (np. wywołanie API) należy użyć dedykowanych funkcji efektów, takich jak `LaunchedEffect`czy  `DisposableEffect`. Gwarantują one bezpieczne wykonanie operacji w odpowiednim momencie cyklu życia kompozycji.

> Efekty uboczne są omówione szczegółowo w rozdziale poświęconym korutynom: [Zadania w tle – Korutyny](https://github.com/MarcinRod/AndroidLecture2025/blob/main/10%20Zadania%20w%20tle%20-%20Korutyny.md).

### Przepływ danych — stan w dół, zdarzenia w górę

Standardowy wzorzec przepływu danych w Compose: stan przekazywany jest **w dół** przez parametry funkcji kompozycyjnych, a zdarzenia (callbacki) propagują **w górę** ku właścicielowi stanu. Jest to fundament wzorca *state hoisting* opisanego w kolejnej sekcji.

```
Właściciel stanu (np. Screen)
  │  stan (w dół)
  ▼
ComposableChild(value, onValueChange)
                       │  zdarzenie (w górę)
                       ▲
```

---

## Wynoszenie stanu (State Hoisting)

**State hoisting** to wzorzec polegający na przekazywaniu stanu i funkcji do jego zmiany z nadrzędnej funkcji kompozycyjnej do podrzędnej, zamiast przechowywania stanu wewnętrznie.

### Jak to wygląda w praktyce?

Zamiast:

```kotlin
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }
    Button(onClick = { count++ }) {
        Text("Kliknięto: $count")
    }
}
```

Stosujemy wynoszenie stanu:

```kotlin
@Composable
fun Counter(count: Int, onIncrement: () -> Unit) {
    Button(onClick = onIncrement) {
        Text("Kliknięto: $count")
    }
}

@Composable
fun CounterScreen() {
    var count by remember { mutableStateOf(0) }
    Counter(count = count, onIncrement = { count++ })
}
```

Funkcja `Counter` nie zarządza już swoim stanem — otrzymuje go jako parametr oraz przyjmuje funkcję do jego zmiany. Funkcja nadrzędna (`CounterScreen`) jest właścicielem stanu.

### Stateful vs Stateless

Funkcja kompozycyjna zarządzająca własnym stanem wewnętrznym to **stateful composable**. Funkcja, która otrzymuje stan i callbacki jako parametry i sama nie przechowuje żadnego stanu, to **stateless composable**.

```kotlin
// Stateful — stan wewnętrzny, mniejsza elastyczność
@Composable
fun CounterStateful() {
    var count by remember { mutableStateOf(0) }
    Button(onClick = { count++ }) { Text("$count") }
}

// Stateless — stan z zewnątrz, pełna elastyczność
@Composable
fun CounterStateless(count: Int, onIncrement: () -> Unit) {
    Button(onClick = onIncrement) { Text("$count") }
}
```

### Zalety state hoistingu

- **Reużywalność:** Ta sama funkcja kompozycyjna może być wykorzystana w różnych miejscach z różnym stanem.
- **Testowalność:** Łatwiej testować funkcje kompozycyjne, które nie mają własnego stanu.
- **Przewidywalność:** Stan jest zarządzany w jednym miejscu, co ułatwia debugowanie i utrzymanie kodu.
- **Integracja z ViewModel:** Wyniesiony stan łatwo powiązać z ViewModel lub innym źródłem danych.

### Kiedy NIE wynosić stanu?

Jeśli stan jest **czysto lokalny** i nie wpływa na inne elementy UI, można go przechowywać wewnątrz funkcji kompozycyjnej (np. rozwinięcie menu, lokalny efekt animacji).

---

> **Więcej o state hoisting:**  
> [State hoisting – oficjalny poradnik](https://developer.android.com/jetpack/compose/state#hoisting)

---

## Material Design i `MaterialTheme`

**Material Design** to system projektowania interfejsów użytkownika opracowany przez Google. Definiuje zestaw zasad, komponentów i wytycznych dotyczących wyglądu, animacji i zachowania UI — tak aby aplikacje na różnych platformach były spójne, czytelne i dostępne.

Aktualna wersja to **Material Design 3** (Material You), która wprowadza m.in. dynamiczne kolory dostosowujące się do tapety użytkownika (Android 12+) oraz zaktualizowane komponenty i typografię.

### `MaterialTheme`

W Jetpack Compose system Material Design jest dostępny przez obiekt `MaterialTheme`, który dostarcza trzy główne zasoby projektowe:

| Zasób | Opis | Przykład użycia |
|---|---|---|
| `colorScheme` | Paleta kolorów aplikacji (tło, powierzchnie, akcenty, błędy) | `MaterialTheme.colorScheme.primary` |
| `typography` | Zestaw stylów tekstowych (nagłówki, ciało, etykiety) | `MaterialTheme.typography.bodyLarge` |
| `shapes` | Zestaw kształtów (małe, średnie, duże zaokrąglenia) | `MaterialTheme.shapes.medium` |

Komponenty takie jak `Button`, `Card`, `Surface` czy `Scaffold` automatycznie korzystają z tych zasobów — zmiana motywu aplikacji (np. tryb ciemny, inne kolory marki) automatycznie aktualizuje wygląd wszystkich komponentów.

### Jak wygląda definicja motywu?

Motyw aplikacji definiowany jest typowo w pliku `Theme.kt` i owijany wokół całej zawartości `MainActivity`:

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

### Tryb ciemny

Motyw może reagować na ustawienia systemowe trybu ciemnego:

```kotlin
val darkTheme = isSystemInDarkTheme()
val colorScheme = if (darkTheme) darkColorScheme(...) else lightColorScheme(...)
```

> Szczegóły tworzenia własnego motywu są poza zakresem tego rozdziału. Android Studio generuje domyślny motyw przy tworzeniu projektu Compose.

---

## Jednostki wymiarów: dp i sp

Android obsługuje kilka jednostek długości, ale w praktyce używa się dwóch:

| Jednostka | Nazwa | Zastosowanie |
|-----------|-------|--------------|
| `px` | Piksele fizyczne | Nie należy używać — wygląd zależy od gęstości ekranu |
| `dp` | Density-independent pixels | Wymiary elementów UI (marginesy, rozmiary, paddingi) |
| `sp` | Scale-independent pixels | Rozmiar tekstu — uwzględnia dodatkowo preferencje czcionki użytkownika |

**Praktyczna zasada:** do wszystkich wymiarów używać `dp`, do rozmiaru tekstu `sp`.

### Na czym polega niezależność od gęstości?

Różne urządzenia mają różną gęstość ekranu (DPI — dots per inch). Element o rozmiarze `100px` będzie wyglądał inaczej na ekranie 160 dpi (duży) i 480 dpi (mały — ok. 3× mniejszy fizycznie).

`1 dp` jest zdefiniowany jako jeden piksel na ekranie referencyjnym o gęstości **160 dpi** (mdpi). Na ekranach o wyższej gęstości Android automatycznie przelicza:

$$\text{piksele} = dp \times \frac{DPI}{160}$$

| Gęstość | Nazwa | Mnożnik | `16 dp` w pikselach |
|---------|-------|---------|---------------------|
| 160 dpi | mdpi | ×1 | 16 px |
| 240 dpi | hdpi | ×1,5 | 24 px |
| 320 dpi | xhdpi | ×2 | 32 px |
| 480 dpi | xxhdpi | ×3 | 48 px |
| 640 dpi | xxxhdpi | ×4 | 64 px |

Dzięki temu element o szerokości `48 dp` zajmuje zbliżony **rozmiar fizyczny** (ok. 7,6 mm) niezależnie od urządzenia. Przeliczenie odbywa się automatycznie — w kodzie zawsze podaje się wartość w `dp`.

W Compose wartości podaje się bezpośrednio w kodzie z rozszerzeniem:

```kotlin
Modifier.padding(16.dp)
Text(text = "Witaj", fontSize = 18.sp)
```

---

## Elementy interfejsu użytkownika

Jetpack Compose udostępnia bogaty zestaw gotowych funkcji kompozycyjnych do budowania interfejsów użytkownika. Pełna lista dostępnych komponentów Material3 wraz z interaktywnym demo dostępna jest w:
- [Material Design 3 – komponenty](https://m3.material.io/components)
- [Material Design Catalog App](https://play.google.com/store/apps/details?id=androidx.compose.material.catalog)

---

### Kontenery układu

Kontenery służą do organizowania i pozycjonowania elementów UI względem siebie.

- **Column**
  - Układa elementy pionowo, jeden pod drugim.
  - Najczęstsze parametry:
    - `modifier` – modyfikator dla całej kolumny (np. rozmiar, padding).
    - `verticalArrangement` – sposób rozmieszczenia elementów w pionie (np. `Arrangement.SpaceBetween`, `Arrangement.Center`).
    - `horizontalAlignment` – wyrównanie elementów w poziomie (np. `Alignment.CenterHorizontally`).
  - Przykład:
    ```kotlin
    Column(
        modifier = Modifier.fillMaxWidth(),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text("Pierwszy")
        Text("Drugi")
    }
    ```

- **Row**
  - Układa elementy poziomo, jeden obok drugiego.
  - Najczęstsze parametry:
    - `modifier`
    - `horizontalArrangement` – rozmieszczenie elementów w poziomie.
    - `verticalAlignment` – wyrównanie elementów w pionie.
  - Przykład:
    ```kotlin
    Row(
        modifier = Modifier.fillMaxWidth(),
        horizontalArrangement = Arrangement.SpaceBetween,
        verticalAlignment = Alignment.CenterVertically
    ) {
        Button(onClick = {}) { Text("OK") }
        Button(onClick = {}) { Text("Anuluj") }
    }
    ```

- **Box**
  - Pozwala nakładać elementy na siebie (pozycjonowanie względem siebie). Dzieci są rysowane w kolejności deklaracji — **ostatnie zadeklarowane dziecko jest na wierzchu** (najwyższy z-order).
  - Najczęstsze parametry:
    - `modifier`
    - `contentAlignment` – domyślne wyrównanie wszystkich dzieci (np. `Alignment.Center`).
  - Przykład:
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
  - Lista przewijana pionowo (odpowiednik RecyclerView).
  - Najczęstsze parametry:
    - `modifier`
    - `contentPadding` – padding wewnątrz listy.
    - `verticalArrangement`
    - `horizontalAlignment`
  - Przykład z rzeczywistą listą danych:
    ```kotlin
    data class Person(val id: Int, val name: String)

    val people = listOf(Person(1, "Anna"), Person(2, "Jan"), Person(3, "Maria"))

    LazyColumn(
        modifier = Modifier.fillMaxSize(),
        contentPadding = PaddingValues(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(
            items = people,
            key = { person -> person.id }   // stabilny klucz — zapobiega błędom animacji
        ) { person ->
            Text(
                text = person.name,
                modifier = Modifier.animateItem()   // animacja dodawania/usuwania/przesuwania elementów
            )
        }
    }
    ```

  **Animacja elementów listy (`animateItem`)**

  `Modifier.animateItem()` dodany do elementu listy powoduje, że:
  - nowe elementy pojawiają się z animacją wejścia (fade in),
  - usunięte elementy znikają z animacją wyjścia (fade out),
  - przesunięcia kolejności elementów są animowane płynnie.

  Parametr `key` jest niezbędny przy animacjach — bez niego Compose nie może zidentyfikować, który element się przemieścił, a który jest nowy.

- **LazyRow**
  - Lista przewijana poziomo.
  - Najczęstsze parametry:
    - `modifier`
    - `contentPadding`
    - `horizontalArrangement`
    - `verticalAlignment`
  - Przykład:
    ```kotlin
    LazyRow(
        contentPadding = PaddingValues(horizontal = 8.dp)
    ) {
        items(10) { index ->
            Card(modifier = Modifier.size(80.dp)) {
                Text("Karta $index")
            }
        }
    }
    ```

---

### Kontenery Material3

Kontenery Material3 automatycznie korzystają z motywu aplikacji (`colorScheme`, `shapes`), co zapewnia wizualną spójność z systemem Material Design.

- **Card**, **ElevatedCard**, **OutlinedCard**
  - Kontenery Material3 z wbudowaną elewacją, kształtem i kolorem tła z motywu. Służą do grupowania powiązanych treści w wizualnie wyróżniony kafelek.
  - Warianty:
    - `Card` – wypełniony, domyślna elewacja tonalna.
    - `ElevatedCard` – jak `Card`, ale z wyraźniejszym cieniem (element wizualnie unosi się nad tłem).
    - `OutlinedCard` – obrys zamiast cienia, płaski wygląd.
    - `Card(onClick = …)` – klikalny wariant z efektem ripple.
  - Kluczowe parametry:
    - `modifier`
    - `shape` – kształt karty (np. `RoundedCornerShape(16.dp)`).
    - `colors` – `CardDefaults.cardColors(containerColor = …)` — nadpisuje kolor tła z motywu.
    - `elevation` – `CardDefaults.cardElevation(defaultElevation = …)` — kontroluje cień.
    - `onClick` – jeśli podany, cały Card staje się interaktywny.
  - Przykład:
    ```kotlin
    // Wariant wypełniony
    Card(modifier = Modifier.fillMaxWidth()) {
        Text("Basic Card", modifier = Modifier.padding(16.dp))
    }

    // Wariant z obramowaniem
    OutlinedCard(modifier = Modifier.fillMaxWidth()) {
        Text("Outlined Card", modifier = Modifier.padding(16.dp))
    }

    // Klikalny wariant ze zmianą koloru
    var selected by remember { mutableStateOf(false) }
    Card(
        onClick = { selected = !selected },
        modifier = Modifier.fillMaxWidth(),
        colors = CardDefaults.cardColors(
            containerColor = if (selected)
                MaterialTheme.colorScheme.primaryContainer
            else
                MaterialTheme.colorScheme.surfaceContainerHigh
        )
    ) {
        Text(
            text = if (selected) "Zaznaczono ✓" else "Kliknij, aby zaznaczyć",
            modifier = Modifier.padding(16.dp)
        )
    }
    ```

- **Surface**
  - Bardziej podstawowy kontener niż `Card` — daje pełną kontrolę nad kolorem, kształtem i elewacją.
  - Kluczowe parametry:
    - `modifier`
    - `color` – kolor tła (domyślnie `colorScheme.surface`).
    - `shape` – kształt (np. `RoundedCornerShape`, `CircleShape`).
    - `tonalElevation` – nakłada odcień koloru z motywu proporcjonalny do elewacji (efekt Material You). Nie tworzy cienia.
    - `shadowElevation` – głębokość fizycznego cienia w dp (wizualny efekt uniesienia).
    - `contentColor` – kolor propagowany do dzieci przez `LocalContentColor`.
    - `onClick` – jeśli podany, Surface staje się klikalna z efektem ripple.
  - Przykład:
    ```kotlin
    Surface(
        modifier = Modifier.fillMaxWidth(),
        shape = RoundedCornerShape(12.dp),
        color = MaterialTheme.colorScheme.tertiaryContainer,
        shadowElevation = 4.dp,
        tonalElevation = 4.dp
    ) {
        Text("Zawartość", modifier = Modifier.padding(16.dp))
    }
    ```

  **Card vs Surface**

  | Komponent | Kiedy stosować |
  |-----------|----------------|
  | `Card` | Gotowe domyślne ustawienia Material3 — kafelki z treścią |
  | `Surface` | Pełna kontrola stylu — niestandardowe kontenery, nakładki, tła ekranów |

- **Spacer**
  - Element do tworzenia odstępów między innymi elementami.
  - Najczęstsze parametry:
    - `modifier` (najczęściej `Modifier.height(...)` lub `Modifier.width(...)`)
  - Przykład:
    ```kotlin
    Spacer(modifier = Modifier.height(16.dp))
    ```

- **Scaffold**
  - `Scaffold` to kontener, który ułatwia budowanie standardowych układów aplikacji zgodnych z Material Design.
  - Pozwala łatwo dodać pasek górny (`TopAppBar`), dolny (`BottomAppBar`), przycisk FAB, szufladę nawigacyjną i inne elementy.

```kotlin
@Composable
fun MainScreen() {
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Tytuł aplikacji") }
            )
        },
        floatingActionButton = {
            FloatingActionButton(onClick = { /* akcja */ }) {
                Icon(Icons.Default.Add, contentDescription = "Dodaj")
            }
        },
        bottomBar = {
            BottomAppBar {
                Text("Dolny pasek")
            }
        }
    ) { innerPadding ->
        // Główna zawartość ekranu
        Column(modifier = Modifier.padding(innerPadding)) {
            Text("Witaj w aplikacji!")
        }
    }
}
```

**Często używane parametry Scaffold:**
- `topBar` – pasek górny (np. `TopAppBar`)
- `bottomBar` – pasek dolny (np. `BottomAppBar`)
- `floatingActionButton` – przycisk FAB
- `drawerContent` – zawartość szuflady nawigacyjnej
- `snackbarHost` – obsługa snackbarów
- `content` – główna zawartość ekranu (przyjmuje padding od Scaffolda)

---

### Elementy UI

- **Text**
  - Służy do wyświetlania tekstu.
  - Najważniejsze parametry:
    - `text` – wyświetlany tekst.
    - `modifier` – modyfikator wyglądu i położenia.
    - `style` – styl tekstu (np. `MaterialTheme.typography.bodyLarge`).
    - `color` – kolor tekstu.
    - `maxLines`, `overflow` – kontrola liczby linii i zachowania przy przepełnieniu.
  - Przykład:
    ```kotlin
    Text(
        text = "Witaj w Compose!",
        style = MaterialTheme.typography.titleLarge,
        color = Color.Blue,
        maxLines = 1,
        overflow = TextOverflow.Ellipsis
    )
    ```

- **Button**
  - Klasyczny przycisk.
  - Najważniejsze parametry:
    - `onClick` – obsługa kliknięcia.
    - `modifier`
    - `enabled` – czy przycisk jest aktywny.
    - `content` – zawartość przycisku (np. `Text`, `Icon`).
  - Przykład:
    ```kotlin
    Button(onClick = { /* akcja */ }) {
        Text("Zatwierdź")
    }
    ```

- **OutlinedButton**, **IconButton**
  - Warianty przycisków: z obramowaniem lub tylko z ikoną.
  - Przykład:
    ```kotlin
    OutlinedButton(onClick = { }) { Text("Anuluj") }
    IconButton(onClick = { }) { Icon(Icons.Default.Favorite, contentDescription = null) }
    ```

- **TextField**
  - Pole do wprowadzania tekstu.
  - Najważniejsze parametry:
    - `value` – aktualna wartość tekstu.
    - `onValueChange` – reakcja na zmianę tekstu.
    - `label` – etykieta pola.
    - `placeholder` – podpowiedź.
    - `singleLine` – czy pole jest jednoliniowe.
    - `modifier`
  - Przykład:
    ```kotlin
    var text by remember { mutableStateOf("") }
    TextField(
        value = text,
        onValueChange = { text = it },
        label = { Text("Imię") },
        singleLine = true
    )
    ```

- **Checkbox**, **Switch**, **RadioButton**
  - Elementy wyboru.
  - Najważniejsze parametry:
    - `checked` – stan zaznaczenia.
    - `onCheckedChange` – reakcja na zmianę stanu.
    - `modifier`
  - Przykład:
    ```kotlin
    var checked by remember { mutableStateOf(false) }
    Checkbox(checked = checked, onCheckedChange = { checked = it })
    Switch(checked = checked, onCheckedChange = { checked = it })
    RadioButton(selected = checked, onClick = { checked = !checked })
    ```

- **Image**, **Icon**
  - Wyświetlanie obrazów i ikon.
  - Najważniejsze parametry:
    - `painter` (dla Image), `imageVector` (dla Icon)
    - `contentDescription` – opis dla dostępności.
    - `modifier`
    - `contentScale` – sposób skalowania obrazu.
  - Przykład:
    ```kotlin
    Image(
        painter = painterResource(id = R.drawable.avatar),
        contentDescription = "Avatar",
        modifier = Modifier.size(64.dp),
        contentScale = ContentScale.Crop
    )
    Icon(Icons.Default.Home, contentDescription = "Strona główna")
    ```

- **HorizontalDivider** / **VerticalDivider**
  - Linia podziału, np. między elementami listy. W Material3 `Divider` jest przestarzały — zastępuje go `HorizontalDivider`.
  - Najważniejsze parametry:
    - `modifier`
    - `color`
    - `thickness`
  - Przykład:
    ```kotlin
    HorizontalDivider(color = Color.Gray, thickness = 1.dp)
    ```


---

## Slots API — przekazywanie zawartości przez lambdy

**Slots API** to wzorzec projektowy w Jetpack Compose polegający na przekazywaniu funkcji kompozycyjnych jako parametrów (`@Composable` lambda). Zamiast definiować sztywną zawartość kontenera, wywołujący dostarcza własną treść do wyznaczonych "slotów".

Jest to mechanizm leżący u podstaw wszystkich standardowych kontenerów Compose — `Card`, `Column`, `Button`, `Scaffold` i inne przyjmują swoje dzieci właśnie w ten sposób.

### Przykład

```kotlin
// Definicja funkcji ze slotem
@Composable
fun InfoCard(
    title: String,
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit          // slot — wywołujący dostarcza zawartość
) {
    Card(modifier = modifier.fillMaxWidth()) {
        Column(modifier = Modifier.padding(16.dp)) {
            Text(text = title, style = MaterialTheme.typography.titleMedium)
            Spacer(modifier = Modifier.height(8.dp))
            content()                        // miejsce, w którym zostanie umieszczona treść
        }
    }
}

// Użycie — wywołujący decyduje, co trafi do slotu
InfoCard(title = "Ważna informacja") {
    Text("Treść karty zdefiniowana przez wywołującego.")
    Button(onClick = { }) { Text("Akcja") }
}
```

### Wiele slotów

Funkcja może mieć kilka slotów, jak np. `Scaffold`:

```kotlin
Scaffold(
    topBar = { TopAppBar(title = { Text("Tytuł") }) },   // slot nagłówka
    floatingActionButton = { FloatingActionButton(onClick = {}) { } },  // slot FAB
    content = { padding -> /* główna treść */ }           // slot zawartości
)
```

Slots API umożliwia tworzenie elastycznych, wielokrotnego użytku kontenerów, których wygląd i zachowanie są definiowane przez wywołującego.

---

## Podgląd funkcji kompozycyjnych (Preview)

Jetpack Compose umożliwia szybki podgląd UI bez uruchamiania aplikacji na emulatorze lub urządzeniu.

- Użyj adnotacji `@Preview` nad funkcją kompozycyjną (lub jej wywołaniem).
- Funkcja podglądu musi być bezparametrowa lub mieć wartości domyślne.

**Przykład:**
```kotlin
@Preview(showBackground = true, name = "Podgląd powitania", widthDp = 320)
@Composable
fun GreetingPreview() {
    Greeting(name = "Compose")
}
```
- Podgląd pojawi się w panelu "Design" w Android Studio.

### Najważniejsze parametry adnotacji `@Preview`

- **`name`** – nazwa podglądu wyświetlana w Android Studio (przydatne, gdy masz kilka podglądów).
- **`showBackground`** – czy pokazać tło wokół funkcji kompozycyjnej (domyślnie `false`). Wartość `true` pozwala lepiej zobaczyć kształt i marginesy elementu.
- **`backgroundColor`** – kolor tła podglądu w formacie ARGB (np. `0xFFFF0000` dla czerwonego). Działa tylko, gdy `showBackground = true`.
- **`widthDp`** i **`heightDp`** – wymusza rozmiar podglądu w dp (przydatne do testowania responsywności).
- **`group`** – pozwala grupować kilka podglądów razem (np. różne warianty kolorystyczne).
- **`uiMode`** – pozwala wymusić tryb jasny/ciemny (`Configuration.UI_MODE_NIGHT_YES` lub `NO`).
- **`locale`** – pozwala wymusić język/region (np. `"pl"` dla polskiego, `"en"` dla angielskiego).
- **`fontScale`** – pozwala przetestować UI przy różnych rozmiarach czcionek (np. `1.5f`).
- **`showSystemUi`** – wyświetla podgląd w otoczeniu pełnego UI systemowego (pasek statusu, pasek nawigacji). Przydatne do testowania układów pełnoekranowych.

**Przykłady:**
```kotlin
@Preview(name = "Tryb ciemny", uiMode = Configuration.UI_MODE_NIGHT_YES, showBackground = true)
@Composable
fun DarkModePreview() {
    MyScreen()
}

@Preview(locale = "pl", showBackground = true, widthDp = 400)
@Composable
fun PolishPreview() {
    Greeting(name = "Użytkownik")
}
```
> **Wskazówka:**  
> Dla jednej funkcji można zdefiniować wiele podglądów, co pozwala szybko weryfikować różne warianty (np. tryb ciemny, różne języki, rozmiary ekranu).
> 
Podgląd w Compose znacznie przyspiesza pracę nad UI, pozwala testować różne warianty i szybciej wychwytywać błędy wizualne.



---

## Animacje

Jetpack Compose udostępnia wbudowane API animacji, które pozwala płynnie reagować na zmiany stanu bez ręcznego zarządzania `Animator` czy `ValueAnimator`.

### `animate*AsState`

Animuje pojedynczą wartość (kolor, rozmiar, pozycję) przy każdej jej zmianie. Wystarczy zastąpić bezpośrednią wartość wywołaniem `animate*AsState`.

```kotlin
var expanded by remember { mutableStateOf(false) }
val targetHeight by animateDpAsState(
    targetValue = if (expanded) 200.dp else 60.dp,
    label = "height"
)

Box(modifier = Modifier.fillMaxWidth().height(targetHeight).background(Color.Blue)) {
    Button(onClick = { expanded = !expanded }) {
        Text(if (expanded) "Zwiń" else "Rozwiń")
    }
}
```

Dostępne warianty: `animateDpAsState`, `animateFloatAsState`, `animateColorAsState`, `animateIntAsState` i inne.

### `rememberInfiniteTransition`

Służy do tworzenia animacji działających w sposób ciągły — bez końca, niezależnie od zmian stanu. Typowe zastosowania to efekty pulsowania, migania, obracania lub płynnego przechodzenia kolorów.

```kotlin
val infiniteTransition = rememberInfiniteTransition(label = "infinite")

val alpha by infiniteTransition.animateFloat(
    initialValue = 1f,
    targetValue = 0.2f,
    animationSpec = infiniteRepeatable(
        animation = tween(durationMillis = 800),
        repeatMode = RepeatMode.Reverse       // wraca do wartości początkowej
    ),
    label = "alpha"
)

Box(
    modifier = Modifier
        .size(64.dp)
        .background(MaterialTheme.colorScheme.primary.copy(alpha = alpha))
)
```

Dostępne metody `InfiniteTransition`: `animateFloat`, `animateColor`, `animateValue`.

### `AnimatedVisibility`

Animuje pojawienie się i zniknięcie elementu UI.

```kotlin
var visible by remember { mutableStateOf(true) }

AnimatedVisibility(visible = visible) {
    Text("Ten tekst jest animowany")
}

Button(onClick = { visible = !visible }) {
    Text("Przełącz")
}
```

Domyślnie używa efektu fade + slide. Wejście i wyjście można dostosować przez parametry `enter` i `exit`.

### `AnimatedContent`

Animuje przejście między różnymi treściami w zależności od stanu.

```kotlin
var count by remember { mutableStateOf(0) }

AnimatedContent(targetState = count) { targetCount ->
    Text("Wartość: $targetCount")
}
```



> Bardziej zaawansowane animacje (własne przejścia, `updateTransition`, animacje nawigacyjne) są poza zakresem tego rozdziału.
> [Oficjalna dokumentacja animacji w Compose](https://developer.android.com/jetpack/compose/animation/introduction)

---

## Dobre praktyki

- **Nazewnictwo:** nazwy funkcji kompozycyjnych powinny być w stylu PascalCase i opisywać to, co wyświetlają — bez czasowników w trybie rozkazującym.
- **Modifier jako parametr:** każda funkcja kompozycyjna powinna przyjmować `Modifier = Modifier` jako pierwszy parametr domyślny, aby wywołujący mógł swobodnie wpływać na jej wygląd i układ.
- **Kolejność modyfikatorów:** ma znaczenie wizualne — `padding → background` daje inny efekt niż `background → padding`.
- **State hoisting:** stan powinien być wynoszony na możliwie najwyższy wspólny poziom. Preferowane są bezstanowe (stateless) funkcje kompozycyjne, które otrzymują dane i callbacki z zewnątrz.
- **Slots API:** kontenery wielokrotnego użytku powinny przyjmować zawartość przez lambdy `@Composable`, co zwiększa ich elastyczność.
- **Podział na małe funkcje:** duże ekrany należy dzielić na mniejsze, wyspecjalizowane funkcje kompozycyjne, co ułatwia czytanie, testowanie i ponowne wykorzystanie kodu.
- **Podgląd (`@Preview`):** warto definiować wiele podglądów dla jednej funkcji (tryb ciemny, różne rozmiary, różne języki), aby szybko wychwytywać błędy wizualne bez uruchamiania aplikacji.
-  **Do wymiarów używać `dp`, do rozmiaru tekstu `sp`** – Android automatycznie przelicza wartości dla ekranów o różnej gęstości. Wartości `px` są zależne od sprzętu i nie należy ich stosować.
- **Animacje oparte na stanie:** do animowania zmian w UI należy używać `animate*AsState` i `AnimatedVisibility` zamiast ręcznego zarządzania wartościami; ciągłe animacje realizuje się przez `rememberInfiniteTransition`.


## Dokumentacja

- [Oficjalna dokumentacja Jetpack Compose – Composables](https://developer.android.com/jetpack/compose/composables)
- [Compose Pathway – podstawy Compose](https://developer.android.com/jetpack/compose/tutorial)
- [Compose – Preview](https://developer.android.com/jetpack/compose/tooling/preview)
- [State hoisting – oficjalny poradnik](https://developer.android.com/jetpack/compose/state#hoisting)

---
### **Następny temat:** [Aktywność i cykl życia](https://github.com/MarcinRod/AndroidLecture2025/blob/main/04%20Aktywno%C5%9B%C4%87.md)