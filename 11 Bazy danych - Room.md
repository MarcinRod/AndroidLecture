# Bazy danych w Androidzie – Room

Room to oficjalna biblioteka Jetpack, która upraszcza korzystanie z relacyjnych baz danych SQLite w aplikacjach Android. Zapewnia bezpieczny i wygodny sposób zapisu oraz odczytu danych.

---

## Dlaczego Room?

- Zapewnia warstwę abstrakcji nad SQLite.
- Automatycznie generuje kod SQL na podstawie adnotacji.
- Umożliwia typowane zapytania i migracje bazy.
- Integruje się z korutynami, Flow i StateFlow.

---

## Konfiguracja — zależności Gradle

**KSP (Kotlin Symbol Processing)** to narzędzie do przetwarzania adnotacji w języku Kotlin. Room korzysta z niego, aby na etapie kompilacji generować kod klas DAO i bazy danych — dzięki temu programista deklaruje tylko interfejsy i adnotacje, a cała logika SQL powstaje automatycznie. KSP jest następcą starszego narzędzia KAPT i działa znacznie szybciej.

Do pliku `build.gradle.kts` (moduł `app`) należy dodać plugin KSP oraz zależności Room:

```kotlin
// build.gradle.kts (app)
plugins {
    id("com.google.devtools.ksp") version "2.0.21-1.0.28"
}

dependencies {
    val roomVersion = "2.7.1"
    implementation("androidx.room:room-runtime:$roomVersion")
    implementation("androidx.room:room-ktx:$roomVersion")   // wsparcie dla korutyn i Flow
    ksp("androidx.room:room-compiler:$roomVersion")         // procesor adnotacji
}
```

Plugin `ksp` musi być również zadeklarowany w pliku `build.gradle.kts` katalogu głównego projektu:

```kotlin
// build.gradle.kts (projekt)
plugins {
    id("com.google.devtools.ksp") version "2.0.21-1.0.28" apply false
}
```

---

## Podstawowe elementy Room

1. **Entity**  
   Klasa danych oznaczona adnotacją `@Entity`, która reprezentuje tabelę w bazie danych. Każde pole klasy odpowiada kolumnie w tabeli.  
   **Klucz główny** oznacza się adnotacją `@PrimaryKey`. Może to być pojedyncze pole (np. `id`), które może być automatycznie generowane (`autoGenerate = true`), lub można zdefiniować klucz złożony (wskazując kilka pól jako klucz główny). Klucz główny gwarantuje unikalność każdego rekordu w tabeli i jest wymagany przez Room.

   **Przykład:**
   ```kotlin
   @Entity
   data class Product(
       @PrimaryKey(autoGenerate = true) val id: Int = 0, // klucz główny, autoinkrementacja
       val name: String,
       val price: Double
   )
   ```

   Można także zdefiniować klucz główny bez autoinkrementacji:
   ```kotlin
   @Entity
   data class User(
       @PrimaryKey val email: String, // klucz główny to email
       val firstName: String
   )
   ```

   Domyślna nazwa tabeli odpowiada nazwie klasy. Parametr `tableName` pozwala ją zmienić:

   ```kotlin
   @Entity(tableName = "produkty")
   data class Product(
       @PrimaryKey(autoGenerate = true) val id: Int = 0,
       val name: String,
       val price: Double
   )
   ```

   Adnotacja `@ColumnInfo` pozwala zmienić nazwę kolumny w tabeli (domyślnie odpowiada nazwie pola):

   ```kotlin
   @Entity(tableName = "produkty")
   data class Product(
       @PrimaryKey(autoGenerate = true) val id: Int = 0,
       @ColumnInfo(name = "nazwa_produktu") val name: String,
       val price: Double
   )
   ```


2. **DAO (Data Access Object)**  
   Interfejs oznaczony adnotacją `@Dao`. Zawiera metody do wykonywania operacji na bazie: wstawiania (`@Insert`), aktualizacji (`@Update`), usuwania (`@Delete`) oraz zapytań (`@Query`). Metody mogą być oznaczone jako `suspend` (dla korutyn) lub zwracać `Flow` (do obserwacji zmian).

   > **Dlaczego operacje na bazie danych muszą być asynchroniczne?**  
   > Dostęp do bazy danych to operacja wejścia/wyjścia, która może trwać stosunkowo długo. Wykonanie jej na wątku głównym (UI thread) zablokowałoby interfejs użytkownika i mogłoby spowodować błąd ANR. Room domyślnie **zabrania** wykonywania zapytań na wątku głównym i zgłosi wyjątek, jeśli zostanie to wykryte. Dlatego wszystkie metody DAO muszą być albo `suspend` (wywoływane z korutyny na wątku `Dispatchers.IO`), albo zwracać `Flow` (zbieranie odbywa się asynchronicznie automatycznie).

   - **@Insert**  
     Adnotacja `@Insert` służy do wstawiania nowych rekordów do tabeli. Room automatycznie generuje odpowiednie polecenie SQL na podstawie przekazanego obiektu lub listy obiektów.  
     Można określić strategię konfliktu (np. `OnConflictStrategy.REPLACE`), która decyduje, co zrobić w przypadku próby wstawienia rekordu z istniejącym kluczem głównym.

     **Przykład:**
    ```kotlin
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(product: Product)

     @Insert
    suspend fun insertAll(products: List<Product>)
     ```

   - **@Update**  
     Adnotacja `@Update` aktualizuje rekord w tabeli na podstawie klucza głównego encji.

    ```kotlin
    @Update
    suspend fun update(product: Product)
    ```

   - **@Delete**  
     Adnotacja `@Delete` usuwa rekord z tabeli na podstawie klucza głównego encji.

    ```kotlin
    @Delete
    suspend fun delete(product: Product)
     ```

   - **@Upsert**  
     Adnotacja `@Upsert` wstawia rekord jeśli nie istnieje lub aktualizuje go jeśli już istnieje. Dostępna od Room 2.5.

    ```kotlin
    @Upsert
    suspend fun upsert(product: Product)
     ```

   - **@Query**  
     Adnotacja `@Query` pozwala definiować własne zapytania SQL (np. SELECT, ). Room sprawdza poprawność zapytania w czasie kompilacji i automatycznie mapuje wyniki na obiekty Kotlin.  
     Parametry przekazuje się do zapytania za pomocą składni `:nazwaParametru`.

     **Przykłady z `suspend` (jednorazowy odczyt):**
     ```kotlin
     // pobranie wszystkich rekordów
     @Query("SELECT * FROM produkty")
     suspend fun getAll(): List<Product>

     // filtrowanie po nazwie
     @Query("SELECT * FROM produkty WHERE name = :name")
     suspend fun findByName(name: String): List<Product>

     // filtrowanie po cenie z sortowaniem
     @Query("SELECT * FROM produkty WHERE price <= :maxPrice ORDER BY price ASC")
     suspend fun getByMaxPrice(maxPrice: Double): List<Product>

     // pobranie pojedynczego rekordu po id (null jeśli nie istnieje)
     @Query("SELECT * FROM produkty WHERE id = :id")
     suspend fun getById(id: Int): Product?

     // zliczanie rekordów
     @Query("SELECT COUNT(*) FROM produkty")
     suspend fun count(): Int

     // usunięcie wszystkich rekordów przez zapytanie SQL
     @Query("DELETE FROM produkty")
     suspend fun deleteAll()
     ```

     **Przykłady z `Flow` (obserwowanie zmian):**
     ```kotlin
     // obserwowanie wszystkich rekordów — automatyczna aktualizacja przy zmianach
     @Query("SELECT * FROM produkty")
     fun observeAll(): Flow<List<Product>>

     // obserwowanie przefiltrowanych rekordów
     @Query("SELECT * FROM produkty WHERE price <= :maxPrice ORDER BY price ASC")
     fun observeByMaxPrice(maxPrice: Double): Flow<List<Product>>
     ```

     Metoda zwracająca `Flow` pozwala na **obserwowanie zmian w bazie danych w czasie rzeczywistym**. Gdy tylko dane w tabeli się zmienią, wszystkie funkcje kompozycyjne lub ViewModele, które zbierają ten `Flow`, automatycznie otrzymają zaktualizowaną listę bez potrzeby ręcznego odświeżania.

     **Kiedy używać `suspend`, a kiedy `Flow`?**

     | Przypadek | Zalecane podejście |
     |---|---|
     | Jednorazowy odczyt (np. szczegóły rekordu po id) | `suspend fun` |
     | Zapis, aktualizacja, usunięcie | `suspend fun` |
     | Lista widoczna w UI, która ma się odbudowywać automatycznie | `Flow<T>` |
     | Stan przekazywany przez repozytorium do ViewModelu | `Flow<T>` — zalecane |

     Dla odczytów danych odzwierciedlanych w UI zaleca się `Flow` zamiast `suspend fun`.


3. **Database**  
   Klasa abstrakcyjna dziedzicząca po `RoomDatabase`, oznaczona adnotacją `@Database`. Określa się w niej listę encji (`entities`) oraz wersję bazy (`version`). Musi zawierać abstrakcyjne metody zwracające DAO dla każdej encji używanej w aplikacji.

   - W tej klasie nie implementuje się żadnych metod – Room generuje całą logikę dostępu do bazy na podstawie deklaracji DAO.
   - Klasa bazy danych powinna być singletonem w aplikacji (tworzona tylko raz).


   #### Ogólny proces tworzenia instancji `RoomDatabase`
   1. Należy utworzyć publiczną klasę abstrakcyjną (abstract class), która rozszerza `RoomDatabase`. Klasa jest abstrakcyjna, ponieważ biblioteka Room tworzy jej implementację automatycznie.
   2. Klasę opatruje się adnotacją `@Database`. W argumentach wymienia się encje bazy danych i ustawia numer wersji.
   3. Należy zdefiniować abstrakcyjną metodę lub właściwość, która zwraca instancję DAO.
   4. W aplikacji powinna istnieć tylko jedna instancja `RoomDatabase` — należy ją tworzyć jako singleton.
   5. Bazę danych tworzy się przez `Room.databaseBuilder` tylko wtedy, gdy nie istnieje jeszcze instancja. W przeciwnym razie zwracana jest istniejąca.

   **Przykład:**
   **Tworzenie instancji bazy (z użyciem companion object):**
   ```kotlin
   @Database(entities = [Product::class], version = 1)
   abstract class AppDatabase : RoomDatabase() {
       abstract fun productDao(): ProductDao

       companion object {
           @Volatile
           private var INSTANCE: AppDatabase? = null

           fun getInstance(context: Context): AppDatabase {
               return INSTANCE ?: synchronized(this) {
                   val instance = Room.databaseBuilder(
                       context.applicationContext,
                       AppDatabase::class.java,
                       "produkty.db"
                   ).build()
                   INSTANCE = instance
                   instance
               }
           }
       }
   }
   ```
   Dzięki companion object zapewnione jest, że w całej aplikacji istnieje tylko jedna instancja bazy danych (singleton).



Adnotacja `@Volatile` w Kotlinie (i Javie) oznacza, że zmienna może być współdzielona między różnymi wątkami i jej wartość będzie zawsze aktualna dla wszystkich wątków.  


Dzięki `@Volatile`:
- Każdy wątek zawsze widzi najnowszą wartość zmiennej `INSTANCE`.
- Zapobiega to sytuacji, w której dwa wątki utworzą dwie różne instancje bazy danych.
- Jest to ważne przy implementacji wzorca singleton w środowisku wielowątkowym.

`synchronized` to słowo kluczowe w Kotlinie i Javie, które pozwala na synchronizację dostępu do fragmentu kodu przez wiele wątków. Dzięki temu tylko jeden wątek na raz może wykonać kod znajdujący się w bloku `synchronized`.


---

## Przykład: zapis danych z Room

### 1. Definicja encji

```kotlin
import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "produkty")
data class Product(
    @PrimaryKey(autoGenerate = true) val id: Int = 0,
    val name: String,
    val price: Double
)
```

### 2. DAO – interfejs dostępu do danych

```kotlin
import androidx.room.*

@Dao
interface ProductDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(product: Product)

    @Query("SELECT * FROM produkty")
    suspend fun getAll(): List<Product>

    @Query("SELECT * FROM produkty")
    fun observeAll(): Flow<List<Product>>
}
```

### 3. Klasa bazy danych (singleton z companion object)

```kotlin
import android.content.Context
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase

@Database(entities = [Product::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun productDao(): ProductDao

    companion object {
        @Volatile
        private var INSTANCE: AppDatabase? = null

        fun getInstance(context: Context): AppDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    "produkty.db"
                ).build()
                INSTANCE = instance
                instance
            }
        }
    }
}
```

### 4. Zapis danych przez repozytorium (zalecane)

Zgodnie z zasadami architektury MVVM ViewModel nie powinien korzystać bezpośrednio z DAO. Zamiast tego — podobnie jak z innymi źródłami danych — powinien komunikować się przez **repozytorium** (patrz [Repozytorium](#repozytorium--integracja-z-architekturą-mvvm) niżej).

Prosty przykład bez repozytorium (tylko dla ilustracji):

```kotlin
// Bezpośrednio z DAO — tylko dla prostych przykładów
val db = AppDatabase.getInstance(context)
val productDao = db.productDao()

viewModelScope.launch {
    val newProduct = Product(name = "Kawa", price = 12.99)
    productDao.insert(newProduct)
}
```

### 5. Obserwowanie zmian w bazie (np. w ViewModelu)

```kotlin
val productsFlow: Flow<List<Product>> = productDao.observeAll()
```

---

## Repozytorium — integracja z architekturą MVVM

Zgodnie z dobrymi praktykami architektury (opisanymi szczegółowo w rozdziale [Architektura aplikacji](https://github.com/MarcinRod/AndroidLecture2025/blob/main/10%20Architektura%20aplikacji.md)), ViewModel nie powinien korzystać bezpośrednio z DAO. Wprowadza się **repozytorium** — warstwę pośrednią, która ukrywa szczegóły dostępu do bazy danych.

Repozytorium często definiuje się przez **interfejs i osobną klasę implementującą** ten interfejs. Taki podział ma dwie kluczowe zalety:

- **Testowalność** — w testach jednostkowych ViewModelu można podmienić prawdziwą implementację (korzystającą z Room) na uproszczoną wersję testową (np. przechowującą dane w pamięci), bez zmiany kodu ViewModelu.
- **Łatwość wymiany źródła danych** — jeśli w przyszłości dane będą pochodzić nie z lokalnej bazy, lecz np. z sieci lub z innej bazy, wystarczy napisać nową implementację interfejsu. ViewModel pozostaje bez zmian.

```kotlin
// Interfejs repozytorium
interface ProductRepository {
    fun observeAll(): Flow<List<Product>>
    suspend fun insert(product: Product)
    suspend fun delete(product: Product)
}

// Implementacja korzystająca z Room
class ProductRepositoryImpl(
    private val dao: ProductDao
) : ProductRepository {
    override fun observeAll(): Flow<List<Product>> = dao.observeAll()

    override suspend fun insert(product: Product) = withContext(Dispatchers.IO) {
        dao.insert(product)
    }

    override suspend fun delete(product: Product) = withContext(Dispatchers.IO) {
        dao.delete(product)
    }
}
```

> **Uwaga:** Room 2.1+ automatycznie przełącza `suspend` funkcje DAO na wątek `Dispatchers.IO` wewnętrznie. Wywołanie `dao.insert()` bezpośrednio z `viewModelScope` (który działa na `Dispatchers.Main`) nie spowoduje błędu. Mimo to jawne `withContext(Dispatchers.IO)` w warstwie repozytorium jest zalecane — czyni intencję oczywistą i sprawia, że każda inna implementacja interfejsu (np. testowa, z inną bazą) będzie działać poprawnie niezależnie od wewnętrznych mechanizmów Room.


ViewModel korzysta wyłącznie z interfejsu repozytorium — nie ma wiedzy o tym, skąd dane pochodzą (Room, sieć, pamięć podręczna):

```kotlin
class ProductViewModel(
    private val repository: ProductRepository
) : ViewModel() {

    val products: StateFlow<List<Product>> = repository.observeAll()
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5_000),
            initialValue = emptyList()
        )

    fun addProduct(name: String, price: Double) {
        viewModelScope.launch {
            repository.insert(Product(name = name, price = price))
        }
    }

    fun deleteProduct(product: Product) {
        viewModelScope.launch {
            repository.delete(product)
        }
    }
}
```

Repozytorium tworzy się z dostępem do DAO i przekazuje do ViewModelu przez manualne wstrzykiwanie zależności:

```kotlin
val db = AppDatabase.getInstance(context)
val repository = ProductRepositoryImpl(db.productDao())

// W funkcji kompozycyjnej:
val viewModel: ProductViewModel = viewModel(
    factory = viewModelFactory {
        initializer { ProductViewModel(repository) }
    }
)
```
---

## TypeConverter — zapisywanie niestandardowych typów

SQLite obsługuje tylko podstawowe typy danych: liczby całkowite, liczby zmiennoprzecinkowe, tekst i bajty. Typy Kotlina takie jak `LocalDate`, `List<String>` czy własne klasy nie mogą być bezpośrednio zapisane w bazie. `TypeConverter` pozwala zdefiniować konwersję między typem Kotlina a typem obsługiwanym przez SQLite.

### Tworzenie konwertera

Klasa z metodami oznaczonymi `@TypeConverter` — jedna konwertuje do typu SQLite (np. `Long`), druga z powrotem:

```kotlin
import androidx.room.TypeConverter
import java.time.LocalDate

class Converters {

    @TypeConverter
    fun localDateToLong(data: LocalDate?): Long? =
        data?.toEpochDay()

    @TypeConverter
    fun longToLocalDate(epochDay: Long?): LocalDate? =
        epochDay?.let { LocalDate.ofEpochDay(it) }
}
```

### Rejestracja konwertera w klasie bazy

Klasy konwerterów rejestruje się adnotacją `@TypeConverters` na klasie `RoomDatabase`:

```kotlin
@Database(entities = [Product::class], version = 1)
@TypeConverters(Converters::class)
abstract class AppDatabase : RoomDatabase() {
    abstract fun productDao(): ProductDao
    // ...
}
```

Po rejestracji Room automatycznie stosuje konwersję przy zapisie i odczycie pól danego typu we wszystkich encjach bazy.

### Przykład użycia w encji

```kotlin
@Entity(tableName = "produkty")
data class Product(
    @PrimaryKey(autoGenerate = true) val id: Int = 0,
    val name: String,
    val price: Double,
    val expiryDate: LocalDate?   // Room użyje Converters automatycznie
)
```

---

## Migracje bazy danych

Gdy schemat bazy zmienia się między wersjami aplikacji (np. dodanie kolumny, zmiana nazwy tabeli), Room wymaga dostarczenia **migracji** — instrukcji, jak przekształcić starą strukturę bazy w nową. Bez migracji Room zgłosi wyjątek przy uruchomieniu.

Numer wersji określany jest w adnotacji `@Database(version = ...)` i należy go inkrementować przy każdej zmianie schematu.

### Tworzenie migracji

Migracja to obiekt `Migration(startVersion, endVersion)` zawierający polecenia SQL transformujące schemat:

```kotlin
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(db: SupportSQLiteDatabase) {
        // dodanie nowej kolumny do istniejącej tabeli
        db.execSQL("ALTER TABLE produkty ADD COLUMN expiryDate INTEGER")
    }
}
```

### Rejestracja migracji w konstruktorze bazy

```kotlin
val instance = Room.databaseBuilder(
    context.applicationContext,
    AppDatabase::class.java,
    "produkty.db"
)
.addMigrations(MIGRATION_1_2)
.build()
```

Room automatycznie wybierze odpowiednią migrację na podstawie wersji zainstalowanej bazy i aktualnej wersji aplikacji.

### `fallbackToDestructiveMigration()` — tylko na etapie tworzenia aplikacji

Podczas tworzenia aplikacji, gdy dane testowe nie są istotne, można pominąć migracje i pozwolić Room na usunięcie i ponowne utworzenie bazy przy zmianie wersji:

```kotlin
Room.databaseBuilder(
    context.applicationContext,
    AppDatabase::class.java,
    "produkty.db"
)
.fallbackToDestructiveMigration(dropAllTables = true)
.build()
```

> **Uwaga:** `fallbackToDestructiveMigration()` niszczy wszystkie dane użytkownika. Należy go używać wyłącznie na etapie tworzenia aplikacji — nigdy w aplikacji produkcyjnej.

---

## Podsumowanie i dobre praktyki

### Architektura warstwy danych z Room

```
Funkcja kompozycyjna
       ↕ (StateFlow / zdarzenia)
   ViewModel
       ↕ (Flow / suspend fun)
  Repozytorium (interfejs)
       ↕
  Implementacja repozytorium
       ↕ (Flow / suspend fun)
      DAO
       ↕
    Room (SQLite)
```

### Dobre praktyki

- **Operacje na bazie muszą być asynchroniczne** — Room domyślnie zgłasza wyjątek przy dostępie z wątku głównego. Metody DAO powinny być `suspend` lub zwracać `Flow`.

- **Do obserwowania list w UI należy używać `Flow`** — pozwala to automatycznie odświeżać interfejs przy każdej zmianie w bazie, bez ręcznego odpytywania.

- **ViewModel nie powinien korzystać bezpośrednio z DAO** — należy wprowadzić repozytorium jako warstwę pośrednią. Ułatwia to testowanie i wymianę źródła danych.

- **Repozytorium należy definiować przez interfejs** — umożliwia podmianę implementacji w testach (np. wersja przechowująca dane w pamięci) bez zmian w ViewModelu.

- **`withContext(Dispatchers.IO)` w metodach `suspend` repozytorium** — jawne określenie wątku IO czyni intencję oczywistą i uniezależnia kod od wewnętrznych mechanizmów Room.

- **Klasa `AppDatabase` powinna być singletonem** — tworzenie wielu instancji `RoomDatabase` jest kosztowne i może prowadzić do niespójności danych.

- **Numer wersji bazy (`version`) należy inkrementować przy każdej zmianie schematu** — bez tego Room zgłosi wyjątek przy uruchomieniu. Przy zmianie schematu należy dostarczyć migrację lub użyć `fallbackToDestructiveMigration()` (niszczy dane — tylko na etapie tworzenia aplikacji).

- **Nazwy tabel i kolumn warto kontrolować przez `tableName` i `@ColumnInfo`** — ułatwia to pisanie zapytań SQL i zmniejsza ryzyko błędów przy refaktoryzacji.


## Więcej informacji

- [Oficjalna dokumentacja Room](https://developer.android.com/training/data-storage/room)
- [Room Persistence Library – przewodnik](https://developer.android.com/jetpack/androidx/releases/room)

---

### **Następny temat:** [Komunikacja przez Internet - Retrofit](https://github.com/MarcinRod/AndroidLecture2025/blob/main/12%20Komunikacja%20przez%20Internet%20-%20Retorfit.md)