# Zasoby aplikacji w Androidzie i Compose

## Co to są zasoby?

**Zasoby** (resources) to wszystkie dodatkowe pliki i dane, które aplikacja wykorzystuje poza kodem źródłowym. Przykłady to: teksty, kolory, obrazy, rozmiary. W Androidzie przechowuje się je w katalogu `res/`.

---


## Typy zasobów w Androidzie

Android obsługuje wiele rodzajów zasobów, które można wykorzystać w aplikacji:

- **Teksty** (`res/values/strings.xml`) – napisy, komunikaty, etykiety, opisy, komunikaty błędów, teksty przycisków.
- **Kolory** (`res/values/colors.xml`) – definicje kolorów używanych w aplikacji.
- **Obrazy rastrowe** (`res/drawable/`) – pliki PNG, JPG, WebP.
- **Obrazy wektorowe** (`res/drawable/`) – pliki XML typu VectorDrawable.
- **Ikony aplikacji** (`res/mipmap/`) – ikony launchera w różnych rozdzielczościach.
- **Wymiary** (`res/values/dimens.xml`) – marginesy, paddingi, rozmiary tekstu, szerokości, wysokości. *(W Compose wartości `Dp` podaje się bezpośrednio w kodzie, `dimens.xml` jest rzadko używany)*
- **Style** (`res/values/styles.xml`) – definicje stylów dla widoków (np. czcionki, kolory, marginesy). *(Raczej nieużywane w Compose)*
- **Motywy** (`res/values/themes/`) – ogólne motywy aplikacji (np. jasny/ciemny). *(W Compose korzysta się z własnego systemu themingu)*
- **Tablice** (`res/values/arrays.xml`) – listy tekstów, kolorów, liczb.
- **Pliki surowe** (`res/raw/`) – dowolne pliki binarne, np. dźwięki, filmy, pliki tekstowe.
- **Czcionki** (`res/font/`) – pliki fontów TTF/OTF.
- **Animacje** (`res/anim/`, `res/animator/`) – animacje XML (np. przejścia, przesunięcia). *(Nie są wykorzystywane w Compose, animacje realizuje się w kodzie)*
- **Menu** (`res/menu/`) – definicje menu aplikacji. *(Nie są używane w Compose)*
- **Layouty** (`res/layout/`) – pliki XML z układami widoków (głównie w klasycznym Androidzie, w Compose rzadziej). *(Nie są używane w Compose)*

> **Uwaga:**  
> Zasoby oznaczone kursywą *(...)* są typowe dla klasycznego Androida (View System) i nie są wykorzystywane lub są wykorzystywane marginalnie w Jetpack Compose. W Compose UI, układy, animacje i style definiuje się bezpośrednio w kodzie Kotlin.

## Identyfikator zasobu i klasa `R`

Każdy zasób w Androidzie otrzymuje **unikalny identyfikator** (ID), który jest generowany automatycznie podczas kompilacji i przechowywany w klasie `R`.

- Klasa `R` jest generowana przez Androida i zawiera zagnieżdżone klasy odpowiadające typom zasobów, np. `R.string`, `R.drawable`, `R.color`, `R.dimen`.
- Dzięki temu można odwoływać się do zasobów w kodzie w bezpieczny sposób, np.:
  - `R.string.app_name`
  - `R.drawable.ic_launcher`
  - `R.color.primary`
  - `R.dimen.padding_large`

**Przykład użycia w Compose:**
```kotlin
Text(text = stringResource(id = R.string.greeting))
Image(painter = painterResource(id = R.drawable.avatar), contentDescription = null)
val color = colorResource(id = R.color.my_color)
val padding = dimensionResource(id = R.dimen.padding_large)
```
> **Informacja:**  
> Można także pobierać zasoby z wykorzystaniem kontekstu:  
> ```kotlin
> val context = LocalContext.current
> val appName = context.getString(R.string.app_name)
> val drawable = ContextCompat.getDrawable(context, R.drawable.avatar)
> ```

**Jak to działa?**  
Podczas kompilacji Android przypisuje każdemu zasobowi unikalny numer ID i generuje klasę `R`, która pozwala na łatwe i bezpieczne odwoływanie się do zasobów w kodzie – bez konieczności podawania ścieżek czy nazw plików jako tekst.


## Przechowywanie tekstu w zasobach

Wszystkie napisy, które pojawiają się w aplikacji (np. tytuły, opisy, komunikaty, teksty przycisków), powinny być przechowywane w pliku `res/values/strings.xml`. Dzięki temu:

- łatwo wspierać wiele języków (internacjonalizacja),
- można zarządzać wszystkimi tekstami w jednym miejscu,
- zmiana tekstu nie wymaga modyfikacji kodu źródłowego.

**Przykład `strings.xml`:**
```xml
<resources>
    <string name="app_name">Moja Aplikacja</string>
    <string name="greeting">Witaj, użytkowniku!</string>
    <string name="button_ok">OK</string>
    <string name="error_network">Brak połączenia z internetem</string>
</resources>
```

**W Compose:**
```kotlin
Text(text = stringResource(id = R.string.greeting))
```
---

## Formatowanie tekstu z parametrami

Ciągi znaków w `strings.xml` mogą zawierać **placeholdery** – miejsca, w które wstawiane są wartości w czasie działania aplikacji.

**Przykład `strings.xml`:**
```xml
<string name="welcome_user">Witaj, %1$s!</string>
<string name="item_count">Znaleziono %1$d elementów.</string>
```

- `%1$s` – pierwszy argument typu tekstowego
- `%1$d` – pierwszy argument typu całkowitego
- Numerowanie (`%1$`, `%2$`) pozwala na zmianę kolejności argumentów przy tłumaczeniu.

**W Compose:**
```kotlin
Text(text = stringResource(R.string.welcome_user, userName))
Text(text = stringResource(R.string.item_count, results.size))
```

---

## Pluralizacja tekstu

Android obsługuje formy liczby mnogiej przez element `<plurals>` w `strings.xml`. Jest to niezbędne, gdy wyświetlana wiadomość zależy od liczby (np. "1 wynik" vs "5 wyników").

**Przykład `strings.xml`:**
```xml
<plurals name="search_results">
    <item quantity="one">Znaleziono %1$d wynik.</item>
    <item quantity="few">Znaleziono %1$d wyniki.</item>
    <item quantity="many">Znaleziono %1$d wyników.</item>
    <item quantity="other">Znaleziono %1$d wyników.</item>
</plurals>
```

Dostępne kategorie: `zero`, `one`, `two`, `few`, `many`, `other` – Android dobiera odpowiednią formę w zależności od języka systemu.

**W Compose:**
```kotlin
val context = LocalContext.current
val text = context.resources.getQuantityString(R.plurals.search_results, count, count)
Text(text = text)
```

> Jetpack Compose nie posiada dedykowanej funkcji kompozycyjnych dla `plurals` – należy sięgnąć po zasoby przez kontekst.

## Obrazy: drawable vs mipmap
- **`drawable/`** – tutaj umieszczamy obrazy używane w aplikacji (np. ilustracje, tła, ikony przycisków). Obsługuje grafiki rastrowe (PNG, JPG) i wektorowe (XML).
- **`mipmap/`** – katalog przeznaczony wyłącznie na ikony aplikacji. Android automatycznie wybiera odpowiednią wersję w zależności od rozdzielczości ekranu.

**W Compose:**
```kotlin
Image(
    painter = painterResource(id = R.drawable.my_image),
    contentDescription = "Opis obrazka"
)
```
Ikona aplikacji jest ustawiana w pliku manifestu i nie powinna być używana w UI aplikacji.

---

## Zasoby alternatywne

Android pozwala tworzyć **alternatywne wersje zasobów** dla różnych języków, rozdzielczości, orientacji ekranu, trybu ciemnego/jasnego i innych cech urządzenia. Dzięki temu aplikacja może automatycznie dopasować się do urządzenia i preferencji użytkownika, bez konieczności pisania dodatkowego kodu do obsługi tych wariantów.

**Przykłady katalogów alternatywnych:**
- `res/values-pl/strings.xml` – teksty po polsku,
- `res/values-en/strings.xml` – teksty po angielsku,
- `res/drawable-hdpi/` – obrazy dla ekranów o wysokiej rozdzielczości,
- `res/values-land/` – zasoby dla trybu poziomego,
- `res/values-night/` – kolory i style dla trybu ciemnego,
- `res/values-sw600dp/` – zasoby dla tabletów i dużych ekranów.

**Jak to działa?**  
Android automatycznie wybiera odpowiedni zasób na podstawie ustawień systemu i parametrów urządzenia (język, rozdzielczość, orientacja, tryb nocny itp.).  
Np. jeśli użytkownik ma ustawiony język polski, system użyje `strings.xml` z katalogu `values-pl`. Jeśli urządzenie jest w trybie nocnym, pobierze kolory z `values-night/colors.xml`.

**Zalety alternatywnych zasobów:**
- Ułatwiają internacjonalizację (wielojęzyczność) aplikacji.
- Pozwalają na optymalizację wyglądu i działania na różnych urządzeniach (telefony, tablety, foldables).
- Umożliwiają wsparcie dla trybu ciemnego bez dodatkowego kodu.
- Pozwalają na dostosowanie UI do orientacji ekranu lub rozmiaru wyświetlacza.

> **Wskazówka:**  
> Tworząc alternatywne zasoby, wystarczy dodać odpowiedni katalog w folderze `res/` i umieścić tam plik o tej samej nazwie co w katalogu podstawowym. Android sam wybierze właściwy plik w czasie działania aplikacji.


---

## Kolory i motywy w Compose

W aplikacjach opartych na Jetpack Compose kolory definiuje się dwoma sposobami:

| Sposób | Kiedy stosować |
|--------|----------------|
| `MaterialTheme.colorScheme` | Podstawowe kolory UI (tło, powierzchnia, kolor główny, tekst). Zalecane w Compose. |
| `colorResource(R.color.xxx)` | Gdy potrzebny jest konkretny kolor z `colors.xml`, np. kolor marki spoza palety Material. |

```xml
<!-- res/values/colors.xml -->
<resources>
    <color name="brand_primary">#6200EE</color>
    <color name="brand_secondary">#03DAC5</color>
</resources>
```

```kotlin
// ui/theme/Color.kt
val BrandPrimary = Color(0xFF6200EE)
val BrandSecondary = Color(0xFF03DAC5)
```

System Material 3 generuje pełną paletę kolorów (jasną i ciemną) na podstawie jednego koloru bazowego. Dlatego w typowej aplikacji Compose nie trzeba ręcznie definiować wielu kolorów – wystarczy skonfigurować motyw.

**Przykład użycia `MaterialTheme` w Compose:**
```kotlin
Text(
    text = "Witaj",
    color = MaterialTheme.colorScheme.primary
)
Surface(
    color = MaterialTheme.colorScheme.background
) { ... }
```

**`colors.xml` nadal ma sens** do przechowywania kolorów markowych (hex), które następnie przekazuje się do konfiguracji motywu:





## Kształty w Material 3

Podobnie jak kolory, Material 3 udostępnia systemowy zestaw kształtów przez `MaterialTheme.shapes`. Zamiast definiować `RoundedCornerShape` ręcznie w każdym komponencie, warto korzystać z predefiniowanych rozmiarów:

| Token | Domyślna wartość | Przykładowe zastosowanie |
|-------|-----------------|--------------------------|
| `MaterialTheme.shapes.small` | `RoundedCornerShape(4.dp)` | Przyciski, pola tekstowe |
| `MaterialTheme.shapes.medium` | `RoundedCornerShape(8.dp)` | Karty, dialogi |
| `MaterialTheme.shapes.large` | `RoundedCornerShape(16.dp)` | Dolne arkusze, duże karty |
| `MaterialTheme.shapes.extraLarge` | `RoundedCornerShape(28.dp)` | Duże powierzchnie |

**Przykład:**
```kotlin
Card(
    shape = MaterialTheme.shapes.medium
) { ... }

Button(
    shape = MaterialTheme.shapes.small,
    onClick = { }
) { Text("Kliknij") }
```

Kształty można dostosować globalnie w konfiguracji motywu — zmiana w jednym miejscu wpływa na cały UI.

---

## Wskazówki
- **Do pobierania zasobów w Compose** należy korzystać z dedykowanych funkcji (`stringResource`, `colorResource`, `painterResource`, `dimensionResource`).
- **Nie należy przechowywać referencji do `Context`** poza funkcjami kompozycyjnymi – więcej w rozdziale o Aktywności.
- **Wszystkie teksty należy przechowywać w `strings.xml`** – ułatwia tłumaczenie i zarządzanie treścią. Do tekstu z parametrami używać `%1$s` / `%1$d`, do liczebników — `<plurals>`.
- **Warto tworzyć alternatywne zasoby** dla różnych języków i rozdzielczości, aby aplikacja była uniwersalna.

- **Kolory i kształty najlepiej pobierać z `MaterialTheme`** (`colorScheme`, `shapes`) – zapewnia spójność z motywem aplikacji i ułatwia jego personalizację.
- **`colors.xml` stosować do przechowywania kolorów markowych (hex)**, które następnie przekazuje się do konfiguracji motywu – nie odwoływać się do nich bezpośrednio w funkcjach kompzycyjnych.

---

## Dokumentacja

- [Oficjalna dokumentacja Android – Resources](https://developer.android.com/guide/topics/resources/providing-resources)
- [Jetpack Compose – Resources](https://developer.android.com/jetpack/compose/resources)

---
### **Następny temat:** [Komponent nawigacji](https://github.com/MarcinRod/AndroidLecture2025/blob/main/06%20Komponent%20nawigacji.md)