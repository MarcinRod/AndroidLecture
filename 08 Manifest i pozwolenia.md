# AndroidManifest.xml i system pozwoleń w Androidzie

## AndroidManifest.xml – do czego służy?

Plik **AndroidManifest.xml** to jeden z najważniejszych plików w każdej aplikacji Android. Znajduje się w katalogu głównym projektu (`app/src/main/AndroidManifest.xml`) i pełni kilka kluczowych funkcji:

- **Deklaruje komponenty aplikacji** – takie jak aktywności (`<activity>`), serwisy (`<service>`), odbiorniki (`<receiver>`) i dostawcy treści (`<provider>`).
- **Określa uprawnienia (permissions)** – czyli do jakich zasobów systemowych aplikacja chce mieć dostęp (np. Internet, aparat, lokalizacja).
- **Definiuje filtry intencji (intent filters)** – pozwalające na obsługę określonych akcji lub danych przez komponenty aplikacji.
- **Ustala podstawowe informacje o aplikacji** – takie jak nazwa, ikona, wersja, pakiet.
- **Deklaruje biblioteki i funkcje specjalne** – np. obsługę kamer, Bluetooth, NFC.

> **Uwaga:** Minimalna i docelowa wersja systemu (`minSdkVersion`, `targetSdkVersion`) są obecnie deklarowane w pliku `build.gradle`, a nie w AndroidManifest.xml.

> **Uwaga (Android 12+):** Każda aktywność posiadająca `<intent-filter>` musi mieć jawnie ustawiony atrybut `android:exported`. Wartość `true` oznacza, że aktywność może być uruchomiona przez inne aplikacje; `false` — tylko przez tę samą aplikację. Brak tego atrybutu powoduje błąd kompilacji.
> ```xml
> <activity android:name=".MainActivity" android:exported="true">
> ```

**Przykładowy fragment AndroidManifest.xml:**
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myapp">

    <application
        android:label="Moja Aplikacja"
        android:icon="@mipmap/ic_launcher">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

    <!-- Uprawnienia -->
    <uses-permission android:name="android.permission.INTERNET" />
</manifest>
```

---

## Klasa Application (`android:name`)

Domyślnie Android tworzy podstawowy obiekt `Application` przy starcie procesu. Atrybut `android:name` w elemencie `<application>` pozwala wskazać własną klasę, która dziedziczy po `Application` — jej metoda `onCreate()` wywoływana jest przed uruchomieniem jakiejkolwiek aktywności.

Typowe zastosowania: inicjalizacja bibliotek (np. Firebase, Timber, Hilt), konfiguracja globalna.

```kotlin
class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()
        // Kod inicjalizujący, np.:
        // Firebase.initialize(this)
    }
}
```

```xml
<application
    android:name=".MyApp"
    android:label="Moja Aplikacja"
    android:icon="@mipmap/ic_launcher">
    ...
</application>
```

---

## System pozwoleń (permissions) w Androidzie

Aby aplikacja mogła korzystać z niektórych funkcji systemowych lub danych użytkownika (np. aparatu, lokalizacji, kontaktów), musi zadeklarować odpowiednie **pozwolenia** w pliku AndroidManifest.xml.

### Deklarowanie pozwoleń

Należy dodać odpowiedni wpis `<uses-permission>` do pliku manifestu, np.:
```xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

### Pozwolenia zwykłe i niebezpieczne

- **Pozwolenia zwykłe** (normal permissions) – są automatycznie przyznawane podczas instalacji (np. dostęp do Internetu).
- **Pozwolenia niebezpieczne** (dangerous permissions) – wymagają dodatkowej zgody użytkownika w trakcie działania aplikacji (np. lokalizacja, kontakty, SMS).

### Typowe pozwolenia niebezpieczne

| Pozwolenie | Zastosowanie |
|------------|--------------|
| `CAMERA` | Dostęp do aparatu fotograficznego |
| `RECORD_AUDIO` | Nagrywanie dźwięku (mikrofon) |
| `ACCESS_FINE_LOCATION` | Dokładna lokalizacja GPS |
| `ACCESS_COARSE_LOCATION` | Przybliżona lokalizacja (sieć/Wi-Fi) |
| `READ_CONTACTS` | Odczyt listy kontaktów |
| `READ_MEDIA_IMAGES` | Dostęp do zdjęć (Android 13+) |
| `READ_EXTERNAL_STORAGE` | Dostęp do plików (Android ≤ 12) |
| `POST_NOTIFICATIONS` | Wyświetlanie powiadomień (Android 13+) |

### Prośba o pozwolenie w czasie działania (runtime permissions)

Od Androida 6.0 (API 23) użytkownik musi wyrazić zgodę na niebezpieczne pozwolenia podczas korzystania z aplikacji.

**Aktualne podejście (Kotlin, Jetpack Activity Result API):**

Najlepiej korzystać z nowoczesnego API `ActivityResultContracts`, które jest prostsze i bezpieczniejsze niż stare `onRequestPermissionsResult`. Ten sam mechanizm `rememberLauncherForActivityResult` był omówiony w poprzednim rozdziale w kontekście odbierania wyników z aktywności — tutaj używany jest z kontraktem `RequestPermission()` zamiast `StartActivityForResult()`.

### Sprawdzanie stanu pozwolenia przed pytaniem

Przed wyświetleniem prośby warto sprawdzić, czy pozwolenie nie zostało już wcześniej przyznane — unika się w ten sposób niepotrzebnych okienek dialogowych:

```kotlin
val context = LocalContext.current
val isGranted = ContextCompat.checkSelfPermission(
    context,
    Manifest.permission.CAMERA
) == PackageManager.PERMISSION_GRANTED
```

Jeśli `isGranted` ma wartość `true`, nie ma potrzeby uruchamiania launchera.

### Uzasadnienie prośby o pozwolenie

Jeśli użytkownik wcześniej odrzucił pozwolenie, Android zaleca wyjaśnienie powodu przed ponownym pytaniem (`shouldShowRequestPermissionRationale`). Można sprawdzić tę flagę w aktywności:

```kotlin
val activity = LocalContext.current as Activity
if (activity.shouldShowRequestPermissionRationale(Manifest.permission.CAMERA)) {
    // show explanation dialog, then launch the request
} else {
    launcher.launch(Manifest.permission.CAMERA)
}
```

### Permanentne odrzucenie pozwolenia

Jeśli użytkownik zaznaczył opcję ‚Nie pytaj ponownie’, `shouldShowRequestPermissionRationale` zwraca `false`, a launcher nie wyświetla żadnego okna dialogowego. Jedynym wyjściem jest przekierowanie do ustawień aplikacji:

```kotlin
val intent = Intent(
    Settings.ACTION_APPLICATION_DETAILS_SETTINGS,
    Uri.fromParts("package", context.packageName, null)
)
context.startActivity(intent)
```

W takim przypadku należy poinformować użytkownika (np. przez `Snackbar` lub dialog), dlaczego pozwolenie jest wymagane i jak je włączyć ręcznie.

**Przykład:**
```kotlin
// W composable lub aktywności:
val launcher = rememberLauncherForActivityResult(
    contract = ActivityResultContracts.RequestPermission()
) { isGranted: Boolean ->
    if (isGranted) {
        // Pozwolenie przyznane
    } else {
        // Pozwolenie odrzucone
    }
}

// Wywołanie prośby o pozwolenie, np. po kliknięciu przycisku:
Button(onClick = {
    launcher.launch(Manifest.permission.CAMERA)
}) {
    Text("Poproś o dostęp do kamery")
}
```

**W przypadku wielu pozwoleń:**
```kotlin
val launcher = rememberLauncherForActivityResult(
    contract = ActivityResultContracts.RequestMultiplePermissions()
) { permissions ->
    val cameraGranted = permissions[Manifest.permission.CAMERA] ?: false
    val locationGranted = permissions[Manifest.permission.ACCESS_FINE_LOCATION] ?: false
    // obsługa odpowiedzi
}

launcher.launch(arrayOf(
    Manifest.permission.CAMERA,
    Manifest.permission.ACCESS_FINE_LOCATION
))
```


---

## Deklarowanie wymaganego sprzętu (`<uses-feature>`)

Oprócz pozwoleń aplikacja może deklarować wymagany sprzęt lub funkcje urządzenia. W odrożnieniu od pozwoleń, `<uses-feature>` informuje Sklep Play, na jakich urządzeniach aplikacja ma się pojawiać:

```xml
<uses-feature android:name="android.hardware.camera" android:required="true" />
<uses-feature android:name="android.hardware.location.gps" android:required="false" />
```

- `required="true"` — aplikacja nie zostanie zainstalowana na urządzeniach bez danej funkcji.
- `required="false"` — funkcja jest opcjonalna; aplikacja powinna w kodzie sprawdzać jej dostępność przez `PackageManager.hasSystemFeature()`.

> **Uwaga:** Niektóre pozwolenia (np. `CAMERA`) automatycznie implikują odpowiadający wpis `<uses-feature>`. Warto go jawnie dodać z `required="false"`, jeśli aplikacja może działać bez aparatu.

---

## Podsumowanie

- **AndroidManifest.xml** to centralny plik konfiguracyjny aplikacji – deklarowane są w nim komponenty, uprawnienia i inne kluczowe informacje.
- Od Androida 12 aktywności z `<intent-filter>` wymagają jawnego atrybutu `android:exported`.
- **Pozwolenia zwykłe** przyznawane są automatycznie przy instalacji; **pozwolenia niebezpieczne** wymagają zgody użytkownika w czasie działania aplikacji (od Android 6.0).
- Przed wyświetleniem prośby należy sprawdzić stan pozwolenia przez `ContextCompat.checkSelfPermission`.
- Do żądania pozwoleń należy używać `ActivityResultContracts.RequestPermission()` — tego samego mechanizmu launchera co przy odbieraniu wyników z aktywności.
- Jeśli użytkownik wcześniej odrzucił pozwolenie, warto sprawdzić `shouldShowRequestPermissionRationale`; przy permanentnym odrzuceniu — przekierować do ustawień przez `Settings.ACTION_APPLICATION_DETAILS_SETTINGS`.
- `<uses-feature>` deklaruje wymagany sprzęt i decyduje o widoczności w Sklepie Play; należy go dodać jawnie z `required="false"`, jeśli funkcja jest opcjonalna.
- Własna klasa `Application` (atrybut `android:name`) pozwala na globalną inicjalizację bibliotek przed uruchomieniem aktywności.



## Więcej informacji:
- [Permissions overview – Android Developers](https://developer.android.com/guide/topics/permissions/overview)
- [Request app permissions – Android Developers](https://developer.android.com/training/permissions/requesting)
---
### **Następny temat:** [Praca w tle: Korutyny](https://github.com/MarcinRod/AndroidLecture2025/blob/main/09%20Zadania%20w%20tle%20-%20Korutyny.md)
