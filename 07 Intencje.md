# System intencji (Intent) w aplikacjach Android

**Intencje** (ang. *Intents*) to jeden z podstawowych mechanizmów komunikacji w Androidzie. Pozwalają na:

- uruchamianie nowych aktywności (ekranów) lub usług,
- przekazywanie danych między komponentami aplikacji,
- wywoływanie funkcji systemowych (np. otwarcie przeglądarki, wysłanie e-maila),
- komunikację między różnymi aplikacjami.

---

## Rodzaje intencji

- **Explicit Intent** – wskazuje dokładnie, który komponent (np. aktywność) ma zostać uruchomiony. Stosuje się ją, gdy znana jest nazwa klasy docelowej.
  ```kotlin
  val intent = Intent(context, DetailActivity::class.java)
  context.startActivity(intent)
  ```

- **Implicit Intent** – opisuje ogólną akcję (np. „zrób zdjęcie”, „wyślij e-mail”), a system Android sam wybiera odpowiednią aplikację lub komponent, który może obsłużyć tę akcję.
  ```kotlin
  val intent = Intent(Intent.ACTION_VIEW, Uri.parse("https://android.com"))
  context.startActivity(intent)
  ```

---

## Komponenty Androida a intencje

Intencje mogą być kierowane nie tylko do aktywności, ale do trzech rodzajów komponentów Androida:

| Komponent | Opis | Jak uruchomić |
|-----------|------|---------------|
| **Activity** | Pojedynczy ekran z interfejsem użytkownika | `startActivity(intent)` |
| **Service** | Operacja działająca w tle, bez interfejsu (np. odtwarzanie muzyki, pobieranie pliku) | `startService(intent)` |
| **BroadcastReceiver** | Nasłuchuje na zdarzenia systemowe lub aplikacyjne (np. zmiana sieci, naładowanie baterii, własne zdarzenia) | `sendBroadcast(intent)` |

**Service** jest przydatny, gdy zadanie ma trwać niezależnie od tego, czy użytkownik widzi ekran — np. synchronizacja danych w tle. W nowoczesnych aplikacjach do zadań w tle częściej używa się WorkManager lub coroutines (omówione w dalszych rozdziałach).

**BroadcastReceiver** pozwala reagować na zdarzenia bez bezpośredniego wywołania przez użytkownika. System Android wysyła wiele wbudowanych broadcastów (np. `ACTION_BOOT_COMPLETED`, `ACTION_BATTERY_LOW`), a aplikacja może też definiować i wysyłać własne.

---

## Typowe akcje systemowe

System Android udostępnia szereg predefiniowanych akcji, z których można korzystać przy tworzeniu implicit intents:

| Akcja | Opis | Przykładowe URI / extras |
|-------|------|---------------------------|
| `ACTION_VIEW` | Wyświetlenie danych (strona, mapa, plik) | `https://...`, `geo:52.4,16.9`, `tel:...` |
| `ACTION_DIAL` | Otwarcie dialera z numerem (bez połączenia) | `tel:+48123456789` |
| `ACTION_CALL` | Bezpośrednie połączenie (wymaga pozwolenia `CALL_PHONE`) | `tel:+48123456789` |
| `ACTION_SEND` | Udostępnienie treści innej aplikacji | extras: `EXTRA_TEXT`, `EXTRA_STREAM` |
| `ACTION_SENDTO` | Otwarcie klienta e-mail lub SMS | `mailto:adres@email.com`, `smsto:numer` |
| `ACTION_PICK` | Wybór elementu (galeria, kontakty) | `MediaStore.Images.Media.EXTERNAL_CONTENT_URI` |

**Przykład – otwarcie lokalizacji na mapie:**
```kotlin
val uri = Uri.parse("geo:52.4064,16.9252?q=Politechnika+Poznańska")
val intent = Intent(Intent.ACTION_VIEW, uri)
context.startActivity(intent)
```

**Przykład – wysłanie e-maila:**
```kotlin
val intent = Intent(Intent.ACTION_SENDTO, Uri.parse("mailto:kontakt@example.com"))
intent.putExtra(Intent.EXTRA_SUBJECT, "Temat wiadomości")
context.startActivity(intent)
```

---
## Przekazywanie danych z intencjami

Intencje pozwalają nie tylko uruchamiać inne komponenty, ale także przekazywać do nich dane. Najczęściej używa się do tego tzw. **extras** – czyli dodatkowych informacji dołączanych do intencji.

### Przekazywanie danych do aktywności

**Dodawanie danych do intencji:**
```kotlin
val intent = Intent(context, DetailActivity::class.java)
intent.putExtra("id", 123)
intent.putExtra("nazwa", "Produkt X")
context.startActivity(intent)
```

**Odbieranie danych w aktywności:**
```kotlin
class DetailActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val id = intent.getIntExtra("id", 0)
        val name = intent.getStringExtra("nazwa")
        // ...przetwarzanie danych
    }
}
```

### Przekazywanie danych do innych aplikacji

Można przekazywać dane do innych aplikacji, np. wysyłając tekst lub obraz:

```kotlin
val intent = Intent(Intent.ACTION_SEND)
intent.type = "text/plain"
intent.putExtra(Intent.EXTRA_TEXT, "To jest wiadomość!")
context.startActivity(Intent.createChooser(intent, "Wybierz aplikację"))
```

### Przekazywanie złożonych obiektów

Aby przekazać obiekt, musi on implementować interfejs `Serializable` lub `Parcelable`:

```kotlin
val intent = Intent(context, DetailActivity::class.java)
intent.putExtra("produkt", produkt) // gdzie produkt : Parcelable
context.startActivity(intent)
```
W aktywności odbiera się:
```kotlin
val product = intent.getParcelableExtra<Produkt>("produkt")
```


### Klucze extras — dobre praktyki

Klucze przekazywane do `putExtra` / `getExtra` to zwykłe ciągi znaków — literówka lub niespójność między klasą wysyłającą a odbierającą powoduje, że dane nie dotrą. Zaleca się definiowanie kluczy jako stałych w `companion object` klasy docelowej aktywności:

```kotlin
class DetailActivity : AppCompatActivity() {
    companion object {
        const val ID_KEY = "id"
        const val NAME_KEY = "nazwa"
    }
    // ...
}
```

Dzięki temu obie strony odwołują się do tej samej stałej:

```kotlin
// Wysyłanie:
intent.putExtra(DetailActivity.ID_KEY, 123)
intent.putExtra(DetailActivity.NAME_KEY, "Produkt X")

// Odbieranie (w DetailActivity):
val id = intent.getIntExtra(ID_KEY, 0)
val name = intent.getStringExtra(NAME_KEY)
```

### **Podsumowanie:**  
- Do przekazywania prostych danych należy używać `putExtra` i odpowiednich metod odbioru (`getIntExtra`, `getStringExtra` itd.).
- Klucze extras należy definiować jako stałe w `companion object` klasy docelowej — unika się w ten sposób błędów wynikających z literówek.
- Do przekazywania obiektów należy używać `Parcelable` lub `Serializable`.
- Przekazywanie danych przez intencje działa zarówno wewnątrz aplikacji, jak i między różnymi aplikacjami.

---

## Odbieranie wyniku z aktywności

Do odbierania wyników z uruchomionej aktywności należy używać `ActivityResultLauncher` wraz z `registerForActivityResult`. Jest to nowoczesne API, które zastąpiło zdeprecjonowane `startActivityForResult`.

**Launcher rejestruje się poza funkcją obsługi zdarzenia** (np. poza `onClick`), ponieważ musi być zainicjowany przed uruchomieniem composable:

```kotlin
// Wybór zdjęcia z galerii — gotowy kontrakt
val launcher = rememberLauncherForActivityResult(
    contract = ActivityResultContracts.GetContent()
) { uri ->
    // uri — wybrany plik lub null, jeśli anulowano
    if (uri != null) {
        // obsługa wybranego pliku
    }
}

Button(onClick = { launcher.launch("image/*") }) {
    Text("Wybierz zdjęcie")
}
```

Do uruchamiania dowolnej aktywności i odbierania jej wyniku służy kontrakt `StartActivityForResult`:

```kotlin
val launcher = rememberLauncherForActivityResult(
    contract = ActivityResultContracts.StartActivityForResult()
) { result ->
    if (result.resultCode == Activity.RESULT_OK) {
        val dane = result.data?.getStringExtra("wynik")
    }
}

Button(onClick = {
    val intent = Intent(context, WyborActivity::class.java)
    launcher.launch(intent)
}) {
    Text("Otwórz ekran wyboru")
}
```

Aktywność uruchomiona przez launcher kończy się wywołaniem `setResult` i `finish`:

```kotlin
// W uruchamianej aktywności:
val resultIntent = Intent().putExtra("wynik", "wybranaWartość")
setResult(Activity.RESULT_OK, resultIntent)
finish()
```

**Wbudowane kontrakty** (`ActivityResultContracts.*`) upraszczają typowe scenariusze:

| Kontrakt | Zastosowanie |
|----------|--------------|
| `GetContent()` | Wybór pliku lub obrazu z galerii |
| `TakePicture()` | Zrobienie zdjęcia aparatem |
| `RequestPermission()` | Żądanie pojedynczego pozwolenia |
| `RequestMultiplePermissions()` | Żądanie wielu pozwoleń naraz |
| `StartActivityForResult()` | Uruchomienie dowolnej aktywności |

---

### System filtrów intencji (Intent Filters)

**Intent Filter** to mechanizm, dzięki któremu aplikacja może zadeklarować, jakie akcje, dane lub kategorie jest w stanie obsłużyć. Filtry intencji są definiowane w pliku `AndroidManifest.xml` i pozwalają systemowi Android wybrać odpowiednią aplikację lub komponent do obsługi danej intencji.

**Jak to działa?**
- Gdy wysyłana jest implicit intent, system przeszukuje wszystkie zainstalowane aplikacje i ich filtry intencji.
- Jeśli znajdzie komponent (np. aktywność), który pasuje do akcji, typu danych lub kategorii, wyświetli użytkownikowi listę wyboru lub uruchomi odpowiednią aplikację.

**Przykład filtra intencji w AndroidManifest.xml:**
```xml
<activity android:name=".SzczegolyActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:scheme="https" android:host="android.com" />
    </intent-filter>
</activity>
```

**Co można filtrować?**
- **Akcje** (np. `android.intent.action.SEND`)
- **Kategorie** (np. `android.intent.category.DEFAULT`)
- **Typy danych** (np. `image/*`, `text/plain`)
- **Schematy i hosty URI** (np. `https://android.com`)


---

## Przykład użycia intencji

**Uruchomienie nowej aktywności z przekazaniem danych:**
```kotlin
val intent = Intent(context, DetailsActivity::class.java)
intent.putExtra("id", 123)
context.startActivity(intent)
```

**Wywołanie akcji systemowej (np. otwarcie strony):**
```kotlin
val intent = Intent(Intent.ACTION_VIEW, Uri.parse("https://android.com"))
context.startActivity(intent)
```

---

## Intencje a Navigation Compose

W tradycyjnych aplikacjach Android przechodzenie między ekranami (aktywnościami) odbywa się za pomocą intencji. W Jetpack Compose, zamiast intencji, do nawigacji **wewnątrz jednej aktywności** używa się Navigation Compose i `NavController`.

- **Navigation Compose** zarządza przejściami między composable (ekranami) w ramach jednej aktywności.
- **Intencje** są nadal używane do:
  - uruchamiania innych aktywności (np. przejście do ekranu logowania w innej aplikacji),
  - wywoływania funkcji systemowych,
  - komunikacji między aplikacjami.

**Przykład: przejście do innej aktywności z poziomu composable:**
```kotlin
val context = LocalContext.current
Button(onClick = {
    val intent = Intent(context, DetailsActivity::class.java)
    intent.putExtra("id", 123)
    context.startActivity(intent)
}) {
    Text("Otwórz szczegóły (nowa aktywność)")
}
```

**Przykład: nawigacja w Compose (bez intencji):**
```kotlin
Button(onClick = { navController.navigate("szczegoly/123") }) {
    Text("Przejdź do szczegółów (w Compose)")
}
```

---

## PendingIntent

`PendingIntent` to opakowanie zwykłej intencji, które pozwala innemu komponentowi systemu (np. powiadomieniu, widgetowi, alarmowi) wykonać tę intencję **w imieniu aplikacji** — nawet gdy aplikacja nie jest uruchomiona.

```kotlin
val intent = Intent(context, MainActivity::class.java)
val pendingIntent = PendingIntent.getActivity(
    context,
    0,
    intent,
    PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
)
```

`PendingIntent` jest wymagany m.in. przy:
- **powiadomieniach** — akcja po kliknięciu w powiadomienie,
- **widgetach ekranu głównego** — obsługa kliknięcia,
- **alarmach** (`AlarmManager`) — uruchomienie kodu w zaplanowanym czasie.

> Flagę `FLAG_IMMUTABLE` należy ustawiać zawsze, gdy zawartość intencji nie musi być modyfikowana przez zewnętrzny komponent (wymagane od Android 12).

---

## Podsumowanie

- **Explicit intents** wskazują konkretny komponent; **implicit intents** opisują akcję i pozwalają systemowi wybrać odpowiednią aplikację.
- Do przekazywania danych między aktywnosćciami służą **extras** (`putExtra`/`getExtra`); klucze należy definiować jako stałe w `companion object`.
- Do odbierania wyniku z aktywności należy używać **`ActivityResultLauncher`** z wbudowanymi kontraktami (`GetContent`, `TakePicture`, `RequestPermission` i in.).
- **Filtry intencji** w `AndroidManifest.xml` deklarują, jakie akcje i dane potrafi obsłużyć dana aktywność.
- **`PendingIntent`** umożliwia wykonanie intencji przez inny komponent systemu (powiadomienie, widget, alarm) w imieniu aplikacji.
- **Navigation Compose** obsługuje nawigację wewnątrz jednej aktywności; intencje nadal są niezbędne do komunikacji z innymi aplikacjami i systemem.
---
### **Następny temat:** [Manifest i pozwolenia](https://github.com/MarcinRod/AndroidLecture2025/blob/main/08%20Manifest%20i%20pozwolenia.md)