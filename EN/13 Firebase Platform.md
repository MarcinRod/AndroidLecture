# Firebase Platform

**Firebase** is platform by Google offering set of tools and services for building, developing and managing mobile and web applications. Allows adding features such as cloud database, user authentication, push notifications, analytics, hosting and many others.

---

## Most Important Firebase Services

- **Firebase Authentication** – easy user authentication (email, Google, Facebook, anonymous and others).
- **Cloud Firestore** – modern, scalable NoSQL cloud database with real-time data synchronization.
- **Realtime Database** – older real-time database (NoSQL).
- **Firebase Cloud Messaging (FCM)** – push notifications for mobile and web applications.
- **Firebase Analytics** – advanced user and event analytics in application.
- **Firebase Storage** – storing files (e.g., photos, documents) in cloud.
- **Firebase Hosting** – hosting static websites.
- **Remote Config** – remote application parameter configuration without store update.
- **Crashlytics** – real-time app error and crash reporting.
- **Test Lab** – app testing on multiple devices in cloud.

---

## Getting Started with Firebase on Android

1. **Create account and project in [Firebase Console](https://console.firebase.google.com/).**
2. **Add Android application to Firebase project** – download `google-services.json` file and place in `app/` directory.
3. **Add plugin and dependencies to Gradle files (Kotlin DSL):**

```kotlin
// build.gradle.kts (project)
plugins {
    id("com.google.gms.google-services") version "4.4.2" apply false
}
```

```kotlin
// build.gradle.kts (app module)
plugins {
    id("com.google.gms.google-services")
}

dependencies {
    // BOM – manages versions of all Firebase libraries
    implementation(platform("com.google.firebase:firebase-bom:33.7.0"))

    // Add selected services (without specifying version – BOM determines it)
    implementation("com.google.firebase:firebase-analytics")
    implementation("com.google.firebase:firebase-auth")
    implementation("com.google.firebase:firebase-firestore")
    // implementation("com.google.firebase:firebase-database")  // Realtime Database (optional)
}
```

> **BOM (Bill of Materials)** – `platform(...)` declaration allows managing all Firebase libraries versions centrally. When updating Firebase, just change BOM number – individual dependency versions selected automatically.

---

## Firebase Authentication – User Authentication

**Firebase Authentication** enables safe adding login to application. Supports multiple authentication methods: email and password, Google, Facebook, anonymous accounts and others. Configuration of selected login methods done in [Firebase Console](https://console.firebase.google.com/).

### `FirebaseAuth` Object

`FirebaseAuth` is main object handling authentication. Get instance through `FirebaseAuth.getInstance()`. Most important properties and methods:

| Element | Description |
|---|---|
| `currentUser` | Currently logged in user or `null` |
| `createUserWithEmailAndPassword()` | Register through email and password |
| `signInWithEmailAndPassword()` | Login through email and password |
| `signOut()` | Logout |
| `sendPasswordResetEmail()` | Send email for password reset |
| `addAuthStateListener()` | Listen to login state changes |

### Basic Operations – Version with Coroutines (`await`)

`firebase-auth` library provides `.await()` extensions (from `kotlinx-coroutines-play-services` package) allowing calling Firebase operations like regular `suspend` functions – without nested listeners (old approach).

```kotlin
// build.gradle.kts – additional dependency for await()
implementation("org.jetbrains.kotlinx:kotlinx-coroutines-play-services:1.8.1")
```

```kotlin
import com.google.firebase.auth.FirebaseAuth
import kotlinx.coroutines.tasks.await

val auth = FirebaseAuth.getInstance()

// Register new user
suspend fun register(email: String, password: String) {
    val result = auth.createUserWithEmailAndPassword(email, password).await()
    val user = result.user  // logged in user after registration
}

// Login user
suspend fun signIn(email: String, password: String) {
    val result = auth.signInWithEmailAndPassword(email, password).await()
    val user = result.user
}

// Logout
fun signOut() {
    auth.signOut()
}

// Check if user logged in
val isLoggedIn = auth.currentUser != null
```

Both methods – `createUserWithEmailAndPassword()` and `signInWithEmailAndPassword()` – return `Task<AuthResult>`. After calling `.await()`, get `AuthResult` object containing `user` property of type `FirebaseUser?`. `FirebaseUser` object represents logged in user and provides `uid` (unique identifier), `email` and `displayName`.

> `suspend` functions with `.await()` should be called inside coroutine (e.g., `viewModelScope.launch { }`) and wrapped in `try/catch` – exception thrown when operation fails (e.g., wrong password, no connection).

### Observing Login State as `Flow`

Login state changes asynchronously (e.g., token expires, user logs out from another device). Can wrap in `callbackFlow` to observe as `Flow` in ViewModel:

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

**More Information:**
- [Firebase Authentication – Documentation](https://firebase.google.com/docs/auth)

---

## Cloud Firestore – Cloud Database

**Cloud Firestore** is modern, scalable NoSQL cloud database offered by Firebase. Recommended successor to Realtime Database for new projects. Data stored in **documents** grouped in **collections**, which eases modeling complex data structures.

### Data Structure: Collections and Documents

Instead of single JSON tree (like in Realtime Database), Firestore organizes data hierarchically:

```
"products" collection
├── document "id1"  →  { name: "Coffee", price: 12.99 }
├── document "id2"  →  { name: "Tea", price: 8.50 }
└── ...
```

- **Collection** – set of documents (equivalent to table, but without strict schema).
- **Document** – object with fields and values; can contain sub-collections.
- Document identifiers can be auto-generated or set manually.

### Data Class for Firestore

Firestore stores documents as key-value maps. When saving, data class object automatically converted to such map (property names become document field keys), and vice versa on read. Data class must have default (parameterless) constructor and public properties with default values:

```kotlin
data class Product(
    val id: String = "",
    val name: String = "",
    val price: Double = 0.0
)
```

> `id` field conventionally stored in object for convenience, but not saved as document field in Firestore – document identifier managed separately by database. On read through `toObject()` / `toObjects()`, `id` field remains empty string unless filled manually from `DocumentSnapshot.id`.

### CRUD – Data Operations

Firestore provides methods returning `Task<T>`, convertible to coroutines through `.await()`. For saving and reading data directly use data class – Firestore treats it as map.

```kotlin
import com.google.firebase.firestore.FirebaseFirestore
import kotlinx.coroutines.tasks.await

val db = FirebaseFirestore.getInstance()

// Save document (with auto-generated id)
suspend fun addProduct(product: Product) {
    db.collection("products").add(product).await()
}

// Save document with specified id
suspend fun saveProduct(product: Product) {
    db.collection("products").document(product.id).set(product).await()
}

// One-time read (single document)
suspend fun getProduct(id: String): Product? {
    val snapshot = db.collection("products").document(id).get().await()
    return snapshot.toObject(Product::class.java)?.copy(id = snapshot.id)
}

// Update selected fields (without overwriting entire document)
suspend fun updatePrice(id: String, newPrice: Double) {
    db.collection("products").document(id)
        .update("price", newPrice).await()
}

// Delete document
suspend fun deleteProduct(id: String) {
    db.collection("products").document(id).delete().await()
}
```

### Observing Changes in Real-Time – `callbackFlow`

Firestore enables listening to collection changes through `addSnapshotListener`. Can wrap in `callbackFlow` to observe as `Flow` in MVVM architecture:

```kotlin
fun observeProductsByMaxPrice(maxPrice: Double): Flow<List<Product>> = callbackFlow {
    val listener = db.collection("products")
        .whereLessThanOrEqualTo("price", maxPrice)
        .addSnapshotListener { snapshot, error ->
            if (error != null) {
                close(error)  // close stream with error
                return@addSnapshotListener
            }
            val products = snapshot?.documents?.mapNotNull { doc ->
                doc.toObject(Product::class.java)?.copy(id = doc.id)
            } ?: emptyList()
            trySend(products)
        }

    awaitClose { listener.remove() }  // unregister listener
}
```

- `whereLessThanOrEqualTo`, `whereEqualTo`, `orderBy` – methods for filtering and sorting queries.
- `toObject(Product::class.java)` – deserialize document to data class (Firestore treats document as map).
- `copy(id = doc.id)` – fill `id` field with document identifier from database.
- `close(error)` – close stream in case of error.

#### Shorter Version with `snapshots()` Function

`firebase-firestore` library provides `snapshots()` extension function automatically wrapping `addSnapshotListener` in `callbackFlow`. Allows simplifying code:

```kotlin
import com.google.firebase.firestore.snapshots
import kotlinx.coroutines.flow.map

fun observeProductsByMaxPrice(maxPrice: Double): Flow<List<Product>> =
    db.collection("products")
        .whereLessThanOrEqualTo("price", maxPrice)
        .snapshots()
        .map { snapshot ->
            snapshot.documents.mapNotNull { doc ->
                doc.toObject(Product::class.java)?.copy(id = doc.id)
            }
        }
```

> `snapshots()` available without additional dependencies – part of `firebase-firestore`. Internally works same as manual `callbackFlow` with `awaitClose { listener.remove() }`, but eliminates repetitive code.

### MVVM Integration – Repository and ViewModel

```kotlin
// Repository
interface ProductRepository {
    fun observeAll(): Flow<List<Product>>
    suspend fun add(product: Product)
    suspend fun delete(id: String)
}

class FirestoreProductRepository : ProductRepository {
    private val db = FirebaseFirestore.getInstance()

    override fun observeAll(): Flow<List<Product>> =
        db.collection("products")
            .snapshots()
            .map { snapshot ->
                snapshot.documents.mapNotNull { doc ->
                    doc.toObject(Product::class.java)?.copy(id = doc.id)
                }
            }

    override suspend fun add(product: Product) {
        db.collection("products").add(product).await()
    }

    override suspend fun delete(id: String) {
        db.collection("products").document(id).delete().await()
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
            catch (e: Exception) { /* handle error */ }
        }
    }
}
```

**More Information:**
- [Cloud Firestore – Documentation](https://firebase.google.com/docs/firestore)

---

## Firebase Realtime Database – Older Solution

> **Note:** Firebase Realtime Database is older solution, preceding Cloud Firestore. For new projects recommend Cloud Firestore, which offers better scalability, richer query model and more flexible data structure. Realtime Database can be used in projects already using it, or when extremely low synchronization latency required.

**Firebase Realtime Database** stores data as single JSON tree synchronized in real-time between all clients.

### Comparison with Firestore: References vs Collections and Documents

Firestore and Realtime Database use different data access models:

| | Cloud Firestore | Realtime Database |
|---|---|---|
| Data Model | Collections → documents | Single JSON tree |
| Data Access | `db.collection("x").document("id")` | `db.getReference("x/id")` |
| Collection Equivalent | `db.collection("products")` | `db.getReference("products")` |
| Document Equivalent | `db.collection("products").document("id1")` | `db.getReference("products/id1")` |
| Save Object | `.set(product).await()` | `.setValue(product).await()` |
| New Key | Auto-generated through `.add()` | `ref.push().key` |
| One-time Read | `.get().await()` → `toObject()` | `.get().await()` → `getValue()` |
| Listen to Changes | `.snapshots()` / `addSnapshotListener` | `addValueEventListener` |

Reference in RTDB (`DatabaseReference`) plays similar role to `DocumentReference` or `CollectionReference` in Firestore – points to specific database location and enables performing operations.

### Observing Changes – `callbackFlow`

Can wrap listening to changes in `callbackFlow` to use as `Flow` in MVVM architecture:

```kotlin
import com.google.firebase.database.FirebaseDatabase
import com.google.firebase.database.DataSnapshot
import com.google.firebase.database.DatabaseError
import com.google.firebase.database.ValueEventListener

fun observeProducts(): Flow<List<Product>> = callbackFlow {
    val ref = FirebaseDatabase.getInstance().getReference("products")

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

### One-time Read with `await`

```kotlin
import kotlinx.coroutines.tasks.await

suspend fun getProduct(id: String): Product? {
    val snapshot = FirebaseDatabase.getInstance()
        .getReference("products/$id").get().await()
    return snapshot.getValue(Product::class.java)
}
```

### Saving Data

```kotlin
suspend fun addProduct(product: Product) {
    val ref = FirebaseDatabase.getInstance().getReference("products")
    val id = ref.push().key ?: return
    ref.child(id).setValue(product).await()
}
```

**More Information:**
- [Firebase Realtime Database – Documentation](https://firebase.google.com/docs/database)

---

## Recommendations and Best Practices

- **Recommend Cloud Firestore for new projects** – offers richer query model, better scalability and more flexible data structure than Realtime Database.

- **Call Firebase operations through `.await()`** – allows writing sequential, readable code in coroutines instead of nested listeners. Requires `kotlinx-coroutines-play-services` dependency.

- **Wrap real-time listening in `callbackFlow`** – ensures proper listener unregistration in `awaitClose { }` and MVVM integration through `Flow`.

- **Isolate Firebase access in repository layer** – ViewModel shouldn't directly use `FirebaseFirestore` or `FirebaseAuth`. Eases testing and data source swapping.

- **Wrap write and read operations in `try/catch`** – Firebase throws exceptions on connection failures, permission errors or nonexistent documents.

- **Configure Firestore security rules in Firebase Console** – by default database may be open or completely locked; appropriate rules protect user data.

- **BOM (`firebase-bom`) simplifies version management** – when updating Firebase, just change BOM number in one place.

---

## More Information

- [Official Firebase Documentation](https://firebase.google.com/docs/android/setup)
- [Cloud Firestore – Documentation](https://firebase.google.com/docs/firestore)
- [Firebase Authentication – Documentation](https://firebase.google.com/docs/auth)
- [Firebase Realtime Database – Documentation](https://firebase.google.com/docs/database)
