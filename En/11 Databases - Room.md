# Databases in Android – Room

Room is official Jetpack library that simplifies using relational SQLite databases in Android applications. Provides safe and convenient way to save and read data.

---

## Why Room?

- Provides abstraction layer over SQLite.
- Automatically generates SQL code based on annotations.
- Enables type-safe queries and migrations.
- Integrates with coroutines, Flow and StateFlow.

---

## Configuration – Gradle Dependencies

**KSP (Kotlin Symbol Processing)** is tool for processing annotations in Kotlin language. Room uses it to generate code for DAO classes and databases at compile time – programmer only declares interfaces and annotations, all SQL logic is generated automatically. KSP is successor to older KAPT tool and works significantly faster.

Add KSP plugin and Room dependencies to `build.gradle.kts` (app module):

```kotlin
// build.gradle.kts (app)
plugins {
    id("com.google.devtools.ksp") version "2.0.21-1.0.28"
}

dependencies {
    val roomVersion = "2.7.1"
    implementation("androidx.room:room-runtime:$roomVersion")
    implementation("androidx.room:room-ktx:$roomVersion")   // support for coroutines and Flow
    ksp("androidx.room:room-compiler:$roomVersion")         // annotation processor
}
```

Also add `ksp` plugin to project-level `build.gradle.kts`:

```kotlin
// build.gradle.kts (project)
plugins {
    id("com.google.devtools.ksp") version "2.0.21-1.0.28" apply false
}
```

---

## Basic Room Elements

1. **Entity**  
   Data class marked with `@Entity` annotation representing table in database. Each class property corresponds to column in table.  
   **Primary key** marked with `@PrimaryKey` annotation. Can be single field (e.g., `id`), which can be auto-generated (`autoGenerate = true`), or can define composite key (marking several fields as primary key). Primary key guarantees uniqueness of each record in table and is required by Room.

   **Example:**
   ```kotlin
   @Entity
   data class Product(
       @PrimaryKey(autoGenerate = true) val id: Int = 0, // primary key, auto-increment
       val name: String,
       val price: Double
   )
   ```

   Can also define primary key without auto-increment:
   ```kotlin
   @Entity
   data class User(
       @PrimaryKey val email: String, // primary key is email
       val firstName: String
   )
   ```

   Table name by default matches class name. Parameter `tableName` allows changing it:

   ```kotlin
   @Entity(tableName = "products")
   data class Product(
       @PrimaryKey(autoGenerate = true) val id: Int = 0,
       val name: String,
       val price: Double
   )
   ```

   `@ColumnInfo` annotation allows changing column name in table (by default matches property name):

   ```kotlin
   @Entity(tableName = "products")
   data class Product(
       @PrimaryKey(autoGenerate = true) val id: Int = 0,
       @ColumnInfo(name = "product_name") val name: String,
       val price: Double
   )
   ```

2. **DAO (Data Access Object)**  
   Interface marked with `@Dao` annotation. Contains methods for performing database operations: inserting (`@Insert`), updating (`@Update`), deleting (`@Delete`) and querying (`@Query`). Methods can be marked as `suspend` (for coroutines) or return `Flow` (for observing changes).

   > **Why must database operations be asynchronous?**  
   > Database access is I/O operation that may take relatively long time. Executing on main thread (UI thread) would block user interface and could cause ANR error. Room by default **forbids** executing queries on main thread and throws exception if detected. Therefore all DAO methods must either be `suspend` (called from coroutine on `Dispatchers.IO` thread) or return `Flow` (collection happens asynchronously automatically).

   - **@Insert**  
     `@Insert` annotation serves inserting new records into table. Room automatically generates appropriate SQL command based on passed object or list of objects.  
     Can specify conflict strategy (e.g., `OnConflictStrategy.REPLACE`) deciding what to do when trying to insert record with existing primary key.

     **Example:**
    ```kotlin
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(product: Product)

     @Insert
    suspend fun insertAll(products: List<Product>)
     ```

   - **@Update**  
     `@Update` annotation updates record in table based on entity primary key.

    ```kotlin
    @Update
    suspend fun update(product: Product)
    ```

   - **@Delete**  
     `@Delete` annotation deletes record from table based on entity primary key.

    ```kotlin
    @Delete
    suspend fun delete(product: Product)
     ```

   - **@Upsert**  
     `@Upsert` annotation inserts record if doesn't exist or updates if already exists. Available from Room 2.5.

    ```kotlin
    @Upsert
    suspend fun upsert(product: Product)
     ```

   - **@Query**  
     `@Query` annotation allows defining custom SQL queries (e.g., SELECT). Room validates query at compile time and automatically maps results to Kotlin objects.  
     Parameters passed to query using `:parameterName` syntax.

     **Examples with `suspend` (one-time read):**
     ```kotlin
     // get all records
     @Query("SELECT * FROM products")
     suspend fun getAll(): List<Product>

     // filter by name
     @Query("SELECT * FROM products WHERE name = :name")
     suspend fun findByName(name: String): List<Product>

     // filter by price with sorting
     @Query("SELECT * FROM products WHERE price <= :maxPrice ORDER BY price ASC")
     suspend fun getByMaxPrice(maxPrice: Double): List<Product>

     // get single record by id (null if doesn't exist)
     @Query("SELECT * FROM products WHERE id = :id")
     suspend fun getById(id: Int): Product?

     // count records
     @Query("SELECT COUNT(*) FROM products")
     suspend fun count(): Int

     // delete all records via SQL query
     @Query("DELETE FROM products")
     suspend fun deleteAll()
     ```

     **Examples with `Flow` (observe changes):**
     ```kotlin
     // observe all records — auto-update on changes
     @Query("SELECT * FROM products")
     fun observeAll(): Flow<List<Product>>

     // observe filtered records
     @Query("SELECT * FROM products WHERE price <= :maxPrice ORDER BY price ASC")
     fun observeByMaxPrice(maxPrice: Double): Flow<List<Product>>
     ```

     Method returning `Flow` allows **observing database changes in real-time**. Whenever data in table changes, all composable functions or ViewModels collecting this `Flow` automatically receive updated list without need for manual refresh.

     **When use `suspend` vs `Flow`?**

     | Case | Recommended Approach |
     |---|---|
     | One-time read (e.g., record details by id) | `suspend fun` |
     | Write, update, delete | `suspend fun` |
     | List visible in UI that should auto-rebuild | `Flow<T>` |
     | State passed through repository to ViewModel | `Flow<T>` – recommended |

     For data reads reflected in UI recommend `Flow` over `suspend fun`.

3. **Database**  
   Abstract class inheriting from `RoomDatabase`, marked with `@Database` annotation. Specifies list of entities (`entities`) and database version (`version`). Must contain abstract methods returning DAO for each entity used in application.

   - No methods implemented in this class – Room generates entire database access logic based on DAO declarations.
   - Database class should be singleton in application (created only once).

   #### General Process for Creating `RoomDatabase` Instance
   1. Create public abstract class (abstract class) extending `RoomDatabase`. Class is abstract because Room library creates its implementation automatically.
   2. Mark class with `@Database` annotation. In arguments list database entities and set version number.
   3. Define abstract method or property returning DAO instance.
   4. Application should have only one `RoomDatabase` instance – create as singleton.
   5. Database created through `Room.databaseBuilder` only when instance doesn't exist yet. Otherwise return existing instance.

   **Example:**
   **Creating database instance (using companion object):**
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
                       "products.db"
                   ).build()
                   INSTANCE = instance
                   instance
               }
           }
       }
   }
   ```

   With companion object, guaranteed only one database instance exists throughout application (singleton).

`@Volatile` in Kotlin (and Java) means variable can be shared between different threads and its value always current for all threads.  

Thanks to `@Volatile`:
- Each thread always sees newest value of `INSTANCE` variable.
- Prevents situation where two threads create two different database instances.
- Important for implementing singleton pattern in multithreaded environment.

`synchronized` is keyword in Kotlin and Java allowing synchronization of code section access by multiple threads. Only one thread at a time can execute code inside `synchronized` block.

---

## Example: Saving Data with Room

### 1. Entity Definition

```kotlin
import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "products")
data class Product(
    @PrimaryKey(autoGenerate = true) val id: Int = 0,
    val name: String,
    val price: Double
)
```

### 2. DAO – Data Access Interface

```kotlin
import androidx.room.*

@Dao
interface ProductDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(product: Product)

    @Query("SELECT * FROM products")
    suspend fun getAll(): List<Product>

    @Query("SELECT * FROM products")
    fun observeAll(): Flow<List<Product>>
}
```

### 3. Database Class (Singleton with Companion Object)

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
                    "products.db"
                ).build()
                INSTANCE = instance
                instance
            }
        }
    }
}
```

### 4. Save Data Through Repository (Recommended)

Following MVVM architecture principles, ViewModel shouldn't access DAO directly. Instead – like with other data sources – should communicate through **repository** (see [Repository](#repository--integration-with-mvvm-architecture) below).

Simple example without repository (illustration only):

```kotlin
// Direct from DAO — only for simple examples
val db = AppDatabase.getInstance(context)
val productDao = db.productDao()

viewModelScope.launch {
    val newProduct = Product(name = "Coffee", price = 12.99)
    productDao.insert(newProduct)
}
```

### 5. Observing Database Changes (e.g., in ViewModel)

```kotlin
val productsFlow: Flow<List<Product>> = productDao.observeAll()
```

---

## Repository – Integration with MVVM Architecture

Following good architecture practices (described in detail in [Application Architecture](/En/10%20Application%20Architecture.md) chapter), ViewModel shouldn't access DAO directly. Introduce **repository** – intermediary layer hiding database access details.

Repository often defined through **interface and separate implementing class**. Such division has two key advantages:

- **Testability** – in ViewModel unit tests can substitute real implementation (using Room) with simplified test version (e.g., storing data in memory), without changing ViewModel code.
- **Easy data source swap** – if future data comes not from local database but e.g. from network or different database, just write new interface implementation. ViewModel remains unchanged.

```kotlin
// Repository interface
interface ProductRepository {
    fun observeAll(): Flow<List<Product>>
    suspend fun insert(product: Product)
    suspend fun delete(product: Product)
}

// Implementation using Room
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

> **Note:** Room 2.1+ automatically switches suspend DAO functions to `Dispatchers.IO` thread internally. Calling `dao.insert()` directly from `viewModelScope` (running on `Dispatchers.Main`) won't cause error. Still explicit `withContext(Dispatchers.IO)` in repository layer recommended – makes intent clear and ensures any other interface implementation (e.g., test, different database) works correctly regardless of Room's internal mechanisms.

ViewModel uses only repository interface – has no knowledge of data source (Room, network, cache):

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

Repository created with DAO access and passed to ViewModel through manual dependency injection:

```kotlin
val db = AppDatabase.getInstance(context)
val repository = ProductRepositoryImpl(db.productDao())

// In composable function:
val viewModel: ProductViewModel = viewModel(
    factory = viewModelFactory {
        initializer { ProductViewModel(repository) }
    }
)
```

---

## TypeConverter – Saving Custom Types

SQLite supports only basic data types: integers, floats, text and bytes. Kotlin types like `LocalDate`, `List<String>` or custom classes can't be directly saved in database. `TypeConverter` allows defining conversion between Kotlin type and type supported by SQLite.

### Creating Converter

Class with methods marked `@TypeConverter` – one converts to SQLite type (e.g., `Long`), another back:

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

### Registering Converter in Database Class

Converter classes registered with `@TypeConverters` annotation on `RoomDatabase` class:

```kotlin
@Database(entities = [Product::class], version = 1)
@TypeConverters(Converters::class)
abstract class AppDatabase : RoomDatabase() {
    abstract fun productDao(): ProductDao
    // ...
}
```

After registration, Room automatically applies conversion when saving and reading fields of that type in all database entities.

### Example Usage in Entity

```kotlin
@Entity(tableName = "products")
data class Product(
    @PrimaryKey(autoGenerate = true) val id: Int = 0,
    val name: String,
    val price: Double,
    val expiryDate: LocalDate?   // Room will use Converters automatically
)
```

---

## Database Migrations

When database schema changes between application versions (e.g., adding column, renaming table), Room requires providing **migration** – instructions how to transform old structure to new. Without migration, Room throws exception on startup.

Version number specified in `@Database(version = ...)` annotation and should be incremented with each schema change.

### Creating Migration

Migration is `Migration(startVersion, endVersion)` object containing SQL commands transforming schema:

```kotlin
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(db: SupportSQLiteDatabase) {
        // add new column to existing table
        db.execSQL("ALTER TABLE products ADD COLUMN expiryDate INTEGER")
    }
}
```

### Registering Migration in Database Builder

```kotlin
val instance = Room.databaseBuilder(
    context.applicationContext,
    AppDatabase::class.java,
    "products.db"
)
.addMigrations(MIGRATION_1_2)
.build()
```

Room automatically selects appropriate migration based on installed database version and current application version.

### `fallbackToDestructiveMigration()` – Only During App Creation

During app creation, when test data isn't important, can skip migrations and let Room delete and recreate database on version change:

```kotlin
Room.databaseBuilder(
    context.applicationContext,
    AppDatabase::class.java,
    "products.db"
)
.fallbackToDestructiveMigration(dropAllTables = true)
.build()
```

> **Note:** `fallbackToDestructiveMigration()` destroys all user data. Use only during app creation – never in production app.

---

## Summary and Best Practices

### Data Layer Architecture with Room

```
Composable function
       ↕ (StateFlow / events)
   ViewModel
       ↕ (Flow / suspend fun)
  Repository (interface)
       ↕
  Repository implementation
       ↕ (Flow / suspend fun)
      DAO
       ↕
    Room (SQLite)
```

### Best Practices

- **Database operations must be asynchronous** – Room by default throws exception accessing from main thread. DAO methods should be `suspend` or return `Flow`.

- **Use `Flow` for observing lists in UI** – allows automatically refreshing interface on every database change, without manual re-querying.

- **ViewModel shouldn't access DAO directly** – introduce repository as intermediary layer. Eases testing and data source swapping.

- **Define repository through interface** – enables substituting implementation in tests (e.g., version storing data in memory) without ViewModel changes.

- **`withContext(Dispatchers.IO)` in suspend repository methods** – explicit thread specification makes intent clear and makes code independent of Room's internal mechanisms.

- **`AppDatabase` class should be singleton** – creating multiple `RoomDatabase` instances is costly and can cause data inconsistencies.

- **Database version (`version`) should be incremented on each schema change** – without this Room throws exception on startup. On schema change must provide migration or use `fallbackToDestructiveMigration()` (destroys data – only during app creation).

- **Control table and column names through `tableName` and `@ColumnInfo`** – eases writing SQL queries and reduces refactoring error risk.

---

## More Information

- [Official Room Documentation](https://developer.android.com/training/data-storage/room)
- [Room Persistence Library – Guide](https://developer.android.com/jetpack/androidx/releases/room)

---

### **Next topic:** [Internet Communication - Retrofit](/En/12%20Internet%20Communication%20-%20Retrofit.md)
