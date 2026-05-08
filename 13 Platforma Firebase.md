# Platforma Firebase

**Firebase** to platforma firmy Google oferująca zestaw narzędzi i usług do tworzenia, rozwijania i zarządzania aplikacjami mobilnymi oraz webowymi. Pozwala dodać do aplikacji funkcje takie jak baza danych w chmurze, uwierzytelnianie użytkowników, powiadomienia push, analityka, hosting i wiele innych.

---

## Najważniejsze usługi Firebase

- **Firebase Authentication** – łatwe uwierzytelnianie użytkowników (e-mail, Google, Facebook, anonimowe i inne).
- **Cloud Firestore** – nowoczesna, skalowalna baza danych NoSQL w chmurze, synchronizująca dane w czasie rzeczywistym.
- **Realtime Database** – starsza baza danych czasu rzeczywistego (NoSQL).
- **Firebase Cloud Messaging (FCM)** – powiadomienia push do aplikacji mobilnych i webowych.
- **Firebase Analytics** – zaawansowana analityka użytkowników i zdarzeń w aplikacji.
- **Firebase Storage** – przechowywanie plików (np. zdjęć, dokumentów) w chmurze.
- **Firebase Hosting** – hosting statycznych stron internetowych.
- **Remote Config** – zdalna konfiguracja parametrów aplikacji bez potrzeby aktualizacji w sklepie.
- **Crashlytics** – raportowanie błędów i awarii aplikacji w czasie rzeczywistym.
- **Test Lab** – testowanie aplikacji na wielu urządzeniach w chmurze.

---

## Jak zacząć korzystać z Firebase w Androidzie?

1. **Utwórz konto i projekt w [konsoli Firebase](https://console.firebase.google.com/).**
2. **Dodaj aplikację Android do projektu Firebase** — pobierz plik `google-services.json` i umieść go w katalogu `app/`.
3. **Dodaj plugin i zależności do plików Gradle (Kotlin DSL):**

```kotlin
// build.gradle.kts (projekt)
plugins {
    id("com.google.gms.google-services") version "4.4.2" apply false
}
```

```kotlin
// build.gradle.kts (moduł app)
plugins {
    id("com.google.gms.google-services")
}

dependencies {
    // BOM — zarządza wersjami wszystkich bibliotek Firebase
    implementation(platform("com.google.firebase:firebase-bom:33.7.0"))

    // Dodaj wybrane usługi (bez podawania wersji — BOM ją określa)
    implementation("com.google.firebase:firebase-analytics")
    implementation("com.google.firebase:firebase-auth")
    implementation("com.google.firebase:firebase-firestore")
    // implementation("com.google.firebase:firebase-database")  // Realtime Database (opcjonalnie)
}
```

> **BOM (Bill of Materials)** — deklaracja `platform(...)` pozwala zarządzać wersjami wszystkich bibliotek Firebase centralnie. Przy aktualizacji Firebase wystarczy zmienić numer BOM — wersje poszczególnych zależności są dobierane automatycznie.

---



## Firebase Authentication — uwierzytelnianie użytkowników

**Firebase Authentication** umożliwia bezpieczne dodanie logowania do aplikacji. Obsługuje wiele metod uwierzytelniania: e-mail i hasło, Google, Facebook, konta anonimowe i inne. Konfiguracji wybranych metod logowania dokonuje się w [konsoli Firebase](https://console.firebase.google.com/).

### Obiekt `FirebaseAuth`

`FirebaseAuth` to główny obiekt obsługi uwierzytelniania. Instancję pobiera się przez `FirebaseAuth.getInstance()`. Najważniejsze właściwości i metody:

| Element | Opis |
|---|---|
| `currentUser` | Aktualnie zalogowany użytkownik lub `null` |
| `createUserWithEmailAndPassword()` | Rejestracja przez e-mail i hasło |
| `signInWithEmailAndPassword()` | Logowanie przez e-mail i hasło |
| `signOut()` | Wylogowanie |
| `sendPasswordResetEmail()` | Wysłanie e-maila do resetowania hasła |
| `addAuthStateListener()` | Nasłuchiwanie zmian stanu zalogowania |

### Podstawowe operacje — wersja z korutynami (`await`)

Biblioteka `firebase-auth` udostępnia rozszerzenia `.await()` (z pakietu `kotlinx-coroutines-play-services`), które pozwalają wywoływać operacje Firebase jak zwykłe funkcje `suspend` — bez zagnieżdżonych listenerów (stare podejście).

```kotlin
// build.gradle.kts — dodatkowa zależność dla await()
implementation("org.jetbrains.kotlinx:kotlinx-coroutines-play-services:1.8.1")
```

```kotlin
import com.google.firebase.auth.FirebaseAuth
import kotlinx.coroutines.tasks.await

val auth = FirebaseAuth.getInstance()

// Rejestracja nowego użytkownika
suspend fun register(email: String, password: String) {
    val result = auth.createUserWithEmailAndPassword(email, password).await()
    val user = result.user  // zalogowany użytkownik po rejestracji
}

// Logowanie użytkownika
suspend fun signIn(email: String, password: String) {
    val result = auth.signInWithEmailAndPassword(email, password).await()
    val user = result.user
}

// Wylogowanie
fun signOut() {
    auth.signOut()
}

// Sprawdzenie czy użytkownik jest zalogowany
val isLoggedIn = auth.currentUser != null
```

Obie metody — `createUserWithEmailAndPassword()` i `signInWithEmailAndPassword()` — zwracają `Task<AuthResult>`. Po wywołaniu `.await()` otrzymuje się obiekt `AuthResult`, który zawiera właściwość `user` typu `FirebaseUser?`. Obiekt `FirebaseUser` reprezentuje zalogowanego użytkownika i udostępnia m.in. `uid` (unikalny identyfikator), `email` oraz `displayName`.

> Funkcje `suspend` z `.await()` należy wywoływać wewnątrz korutyny (np. `viewModelScope.launch { }`) i opakowywać w `try/catch` — wyjątek jest rzucany gdy operacja się nie powiedzie (np. błędne hasło, brak połączenia).

### Obserwowanie stanu zalogowania jako `Flow`

Stan zalogowania zmienia się asynchronicznie (np. token wygasa, użytkownik wylogowuje się z innego urządzenia). Można go opakować w `callbackFlow`, aby obserwować go jako `Flow` w ViewModelu:

```kotlin
fun observeAuthState(): Flow<Boolean> = callbackFlow {
    val listener = FirebaseAuth.AuthStateListener { auth ->
        trySend(auth.currentUser != null)
    }
    FirebaseAuth.getInstance().addAuthStateListener(listener)
    awaitClose {
        FirebaseAuth.getInstance().removeAuthStateListener(listener)
    }
}
```

**Więcej informacji:**
- [Firebase Authentication — dokumentacja](https://firebase.google.com/docs/auth)

---

## Cloud Firestore — baza danych w chmurze

**Cloud Firestore** to nowoczesna, skalowalna baza danych NoSQL oferowana przez Firebase. Jest zalecanym następcą Realtime Database dla nowych projektów. Dane przechowywane są w **dokumentach** pogrupowanych w **kolekcje**, co ułatwia modelowanie złożonych struktur danych.

### Struktura danych: kolekcje i dokumenty

Zamiast jednego drzewa JSON (jak w Realtime Database), Firestore organizuje dane hierarchicznie:

```
kolekcja "produkty"
├── dokument "id1"  →  { nazwa: "Kawa", cena: 12.99 }
├── dokument "id2"  →  { nazwa: "Herbata", cena: 8.50 }
└── ...
```

- **Kolekcja** — zbiór dokumentów (odpowiednik tabeli, ale bez sztywnego schematu).
- **Dokument** — obiekt z polami i wartościami; może zawierać podkolekcje.
- Identyfikatory dokumentów mogą być generowane automatycznie lub ustalane ręcznie.

### Klasa danych dla Firestore

Firestore przechowuje dokumenty jako mapy klucz–wartość. Przy zapisie obiekt klasy danych jest automatycznie konwertowany na taką mapę (nazwy właściwości stają się kluczami pól dokumentu), a przy odczycie — odwrotnie. Klasa danych musi mieć domyślny (bezparametrowy) konstruktor i publiczne właściwości z wartościami domyślnymi:

```kotlin
data class Product(
    val id: String = "",
    val name: String = "",
    val price: Double = 0.0
)
```

> Pole `id` jest zwyczajowo przechowywane w obiekcie dla wygody, ale nie jest zapisywane jako pole dokumentu w Firestore — identyfikator dokumentu jest zarządzany osobno przez bazę. Przy odczycie przez `toObject()` / `toObjects()` pole `id` pozostaje pustym stringiem, chyba że zostanie wypełnione ręcznie na podstawie `DocumentSnapshot.id`.

### CRUD — operacje na danych

Firestore udostępnia metody zwracające `Task<T>`, które można zamieniać na korutyny przez `.await()`. Do zapisu i odczytu danych bezpośrednio używa się klasy danych — Firestore traktuje ją jak mapę.

```kotlin
import com.google.firebase.firestore.FirebaseFirestore
import kotlinx.coroutines.tasks.await

val db = FirebaseFirestore.getInstance()

// Zapis dokumentu (z automatycznym id)
suspend fun addProduct(product: Product) {
    db.collection("produkty").add(product).await()
}

// Zapis dokumentu z określonym id
suspend fun saveProduct(product: Product) {
    db.collection("produkty").document(product.id).set(product).await()
}

// Odczyt jednorazowy (pojedynczy dokument)
suspend fun getProduct(id: String): Product? {
    val snapshot = db.collection("produkty").document(id).get().await()
    return snapshot.toObject(Product::class.java)?.copy(id = snapshot.id)
}

// Aktualizacja wybranych pól (bez nadpisania całego dokumentu)
suspend fun updatePrice(id: String, newPrice: Double) {
    db.collection("produkty").document(id)
        .update("price", newPrice).await()
}

// Usunięcie dokumentu
suspend fun deleteProduct(id: String) {
    db.collection("produkty").document(id).delete().await()
}
```

### Obserwowanie zmian w czasie rzeczywistym — `callbackFlow`

Firestore umożliwia nasłuchiwanie zmian w kolekcji przez `addSnapshotListener`. Można to opakować w `callbackFlow`, aby obserwować dane jako `Flow` w architekturze MVVM:

```kotlin
fun observeProductsByMaxPrice(maxPrice: Double): Flow<List<Product>> = callbackFlow {
    val listener = db.collection("produkty")
        .whereLessThanOrEqualTo("price", maxPrice)
        .addSnapshotListener { snapshot, error ->
            if (error != null) {
                close(error)  // zamknięcie strumienia z błędem
                return@addSnapshotListener
            }
            val products = snapshot?.documents?.mapNotNull { doc ->
                doc.toObject(Product::class.java)?.copy(id = doc.id)
            } ?: emptyList()
            trySend(products)
        }

    awaitClose { listener.remove() }  // wyrejestrowanie listenera
}
```

- `whereLessThanOrEqualTo`, `whereEqualTo`, `orderBy` — metody filtrowania i sortowania zapytań.
- `toObject(Product::class.java)` — deserializacja dokumentu na klasę danych (Firestore traktuje dokument jak mapę).
- `copy(id = doc.id)` — uzupełnienie pola `id` identyfikatorem dokumentu z bazy.
- `close(error)` — zamknięcie strumienia w przypadku błędu.

#### Krótsza wersja z funkcją `snapshots()`

Biblioteka `firebase-firestore` udostępnia funkcję rozszerzającą `snapshots()`, która automatycznie opakowuje `addSnapshotListener` w `callbackFlow`. Pozwala to na uproszczenie kodu:

```kotlin
import com.google.firebase.firestore.snapshots
import kotlinx.coroutines.flow.map

fun observeProductsByMaxPrice(maxPrice: Double): Flow<List<Product>> =
    db.collection("produkty")
        .whereLessThanOrEqualTo("price", maxPrice)
        .snapshots()
        .map { snapshot ->
            snapshot.documents.mapNotNull { doc ->
                doc.toObject(Product::class.java)?.copy(id = doc.id)
            }
        }
```

> `snapshots()` jest dostępna bez dodatkowych zależności — jest częścią `firebase-firestore`. Wewnętrznie działa tak samo jak ręczne `callbackFlow` z `awaitClose { listener.remove() }`, ale eliminuje powtarzalny kod.

### Integracja z MVVM — repozytorium i ViewModel

```kotlin
// Repozytorium
interface ProductRepository {
    fun observeAll(): Flow<List<Product>>
    suspend fun add(product: Product)
    suspend fun delete(id: String)
}

class FirestoreProductRepository : ProductRepository {
    private val db = FirebaseFirestore.getInstance()

    override fun observeAll(): Flow<List<Product>> =
        db.collection("produkty")
            .snapshots()
            .map { snapshot ->
                snapshot.documents.mapNotNull { doc ->
                    doc.toObject(Product::class.java)?.copy(id = doc.id)
                }
            }

    override suspend fun add(product: Product) {
        db.collection("produkty").add(product).await()
    }

    override suspend fun delete(id: String) {
        db.collection("produkty").document(id).delete().await()
    }
}

// ViewModel
class ProductViewModel(private val repository: ProductRepository) : ViewModel() {

    val products: StateFlow<List<Product>> = repository.observeAll()
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5_000),
            initialValue = emptyList()
        )

    fun addProduct(product: Product) {
        viewModelScope.launch {
            try { repository.add(product) }
            catch (e: Exception) { /* obsługa błędu */ }
        }
    }
}
```

**Więcej informacji:**
- [Cloud Firestore — dokumentacja](https://firebase.google.com/docs/firestore)

---

## Firebase Realtime Database — starsze rozwiązanie

> **Uwaga:** Firebase Realtime Database jest starszym rozwiązaniem, poprzedzającym Cloud Firestore. Dla nowych projektów zaleca się Cloud Firestore, które oferuje lepszą skalowalność, bogatszy model zapytań i bardziej elastyczną strukturę danych. Realtime Database można stosować w projektach, które już z niej korzystają, lub gdy wymagana jest wyjątkowo niska latencja synchronizacji.

**Firebase Realtime Database** przechowuje dane w formie jednego drzewa JSON synchronizowanego w czasie rzeczywistym między wszystkimi klientami.

### Porównanie z Firestore: referencje vs kolekcje i dokumenty

Firestore i Realtime Database używają różnych modeli dostępu do danych:

| | Cloud Firestore | Realtime Database |
|---|---|---|
| Model danych | Kolekcje → dokumenty | Jedno drzewo JSON |
| Dostęp do danych | `db.collection("x").document("id")` | `db.getReference("x/id")` |
| Odpowiednik kolekcji | `db.collection("produkty")` | `db.getReference("produkty")` |
| Odpowiednik dokumentu | `db.collection("produkty").document("id1")` | `db.getReference("produkty/id1")` |
| Zapis obiektu | `.set(product).await()` | `.setValue(product).await()` |
| Nowy klucz | automatyczny przez `.add()` | `ref.push().key` |
| Odczyt jednorazowy | `.get().await()` → `toObject()` | `.get().await()` → `getValue()` |
| Nasłuchiwanie zmian | `.snapshots()` / `addSnapshotListener` | `addValueEventListener` |

Referencja w RTDB (`DatabaseReference`) pełni podobną rolę co `DocumentReference` lub `CollectionReference` w Firestore — wskazuje na konkretne miejsce w bazie i pozwala wykonywać na nim operacje.

### Obserwowanie zmian — `callbackFlow`

Nasłuchiwanie zmian można opakować w `callbackFlow`, aby korzystać z nich jako `Flow` w architekturze MVVM:

```kotlin
import com.google.firebase.database.FirebaseDatabase
import com.google.firebase.database.DataSnapshot
import com.google.firebase.database.DatabaseError
import com.google.firebase.database.ValueEventListener

fun observeProducts(): Flow<List<Product>> = callbackFlow {
    val ref = FirebaseDatabase.getInstance().getReference("produkty")

    val listener = object : ValueEventListener {
        override fun onDataChange(snapshot: DataSnapshot) {
            val products = snapshot.children.mapNotNull {
                it.getValue(Product::class.java)
            }
            trySend(products)
        }
        override fun onCancelled(error: DatabaseError) {
            close(error.toException())
        }
    }

    ref.addValueEventListener(listener)
    awaitClose { ref.removeEventListener(listener) }
}
```

### Jednorazowy odczyt z `await`

```kotlin
import kotlinx.coroutines.tasks.await

suspend fun getProduct(id: String): Product? {
    val snapshot = FirebaseDatabase.getInstance()
        .getReference("produkty/$id").get().await()
    return snapshot.getValue(Product::class.java)
}
```

### Zapis danych

```kotlin
suspend fun addProduct(product: Product) {
    val ref = FirebaseDatabase.getInstance().getReference("produkty")
    val id = ref.push().key ?: return
    ref.child(id).setValue(product).await()
}
```

**Więcej informacji:**
- [Firebase Realtime Database — dokumentacja](https://firebase.google.com/docs/database)

---

## Zalecenia i dobre praktyki

- **Do nowych projektów zaleca się Cloud Firestore** — oferuje bogatszy model zapytań, lepszą skalowalność i bardziej elastyczną strukturę danych niż Realtime Database.

- **Operacje Firebase należy wywoływać przez `.await()`** — pozwala pisać sekwencyjny, czytelny kod w korutynach zamiast zagnieżdżonych listenerów. Wymaga zależności `kotlinx-coroutines-play-services`.

- **Nasłuchiwanie zmian w czasie rzeczywistym należy opakowywać w `callbackFlow`** — zapewnia prawidłowe wyrejestrowanie listenera w `awaitClose { }` i integrację z architekturą MVVM przez `Flow`.

- **Dostęp do Firebase należy izolować w warstwie repozytorium** — ViewModel nie powinien korzystać bezpośrednio z `FirebaseFirestore` ani `FirebaseAuth`. Ułatwia to testowanie i wymianę źródła danych.

- **Operacje zapisu i odczytu należy opakowywać w `try/catch`** — Firebase rzuca wyjątki przy braku połączenia, błędach uprawnień lub nieistniejących dokumentach.

- **Reguły bezpieczeństwa Firestore należy konfigurować w konsoli Firebase** — domyślnie baza może być otwarta lub całkowicie zablokowana; odpowiednie reguły chronią dane użytkowników.

- **BOM (`firebase-bom`) upraszcza zarządzanie wersjami** — przy aktualizacji Firebase wystarczy zmienić numer BOM w jednym miejscu.

---

## Więcej informacji

- [Oficjalna dokumentacja Firebase](https://firebase.google.com/docs/android/setup)
- [Cloud Firestore — dokumentacja](https://firebase.google.com/docs/firestore)
- [Firebase Authentication — dokumentacja](https://firebase.google.com/docs/auth)
- [Firebase Realtime Database — dokumentacja](https://firebase.google.com/docs/database)
