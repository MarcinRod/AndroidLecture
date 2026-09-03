# Internet Communication in Android

Modern mobile applications frequently communicate with servers over Internet to fetch or send data (e.g., getting product list, user login, data synchronization). In Android, most commonly use libraries such as **Retrofit** or **OkHttp** for this purpose.

---

## Permission for Internet Access

For application to communicate with server over Internet, must add appropriate permission in `AndroidManifest.xml` file:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

Without this permission, no network request (e.g., through Retrofit, OkHttp) will work, and application receives connection error.

**Important:**  
This permission required in every application using Internet – both for downloading and sending data.

---

## Key Concepts

- **API (Application Programming Interface)** – set of rules and addresses (endpoints) through which application communicates with server.
- **REST API** – most popular communication style, based on HTTP protocol and operations like GET, POST, PUT, DELETE.
- **JSON** – most commonly used data exchange format between application and server.

---

## Popular Communication Libraries

- **Retrofit** – most popular library for handling REST API in Android. Enables easy execution of HTTP requests and mapping responses to Kotlin objects.
- **OkHttp** – low-level HTTP library on which Retrofit is based.

---

## Retrofit

**Retrofit** is most popular library for communicating with REST API in Android. Allows executing HTTP requests and mapping JSON responses to Kotlin objects. Supports interchangeable JSON converters – including Moshi and kotlinx-serialization.

---

### Dependencies (build.gradle.kts)

**With Moshi Converter:**
```kotlin
implementation("com.squareup.retrofit2:retrofit:2.11.0")
implementation("com.squareup.retrofit2:converter-moshi:2.11.0")
implementation("com.squareup.moshi:moshi:1.15.1")
implementation("com.squareup.moshi:moshi-kotlin:1.15.1")
// Register HTTP requests – debug variant only
debugImplementation("com.squareup.okhttp3:logging-interceptor:4.12.0")
```

**With kotlinx-serialization Converter:**
```kotlin
plugins {
    kotlin("plugin.serialization") version "2.0.0"
}

dependencies {
    implementation("com.squareup.retrofit2:retrofit:2.11.0")
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.7.3")
    implementation("com.jakewharton.retrofit:retrofit2-kotlinx-serialization-converter:1.0.0")
    // Register HTTP requests – debug variant only
    debugImplementation("com.squareup.okhttp3:logging-interceptor:4.12.0")
}
```

---

### Data Model

#### With Moshi Converter

```kotlin
import com.squareup.moshi.Json
import com.squareup.moshi.JsonClass

@JsonClass(generateAdapter = true)
data class Product(
    val id: Int,
    @Json(name = "product_name") val name: String,
    val price: Double,
    val description: String? = null
)
```

**Moshi Annotations:**
- `@JsonClass(generateAdapter = true)` – generates adapter for converting object to/from JSON. Required for correct Kotlin data class handling.
- `@Json(name = "key")` – maps class property to JSON field with different name.
- Properties with `String?` type and default value `null` are optional – if missing in JSON, get value `null`.

**Important:** if property not nullable and has no default value, and missing from JSON, Moshi throws error during reading.

#### With kotlinx-serialization

```kotlin
import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class Product(
    val id: Int,
    @SerialName("product_name") val name: String,
    val price: Double,
    val description: String? = null
)
```

**kotlinx-serialization Annotations:**
- `@Serializable` – marks class for serialization; adapter generated at compile time by compiler plugin.
- `@SerialName("key")` – equivalent to `@Json(name = ...)` from Moshi.
- Properties with default value are optional – if missing from JSON, get default value.

---

### Defining API Interface

API interface in Retrofit contains methods corresponding to individual HTTP requests. Each method marked with annotation specifying request type and endpoint path.

**Most Important Retrofit Annotations:**
- `@GET("endpoint")` – fetching request.
- `@POST("endpoint")` – sending request.
- `@PUT("endpoint")` – updating request.
- `@DELETE("endpoint")` – deleting request.
- `@Query("param")` – parameter passed in URL (e.g., `?id=1`).
- `@Path("param")` – dynamic parameter in URL path (e.g., `/products/{id}`).
- `@Body` – object sent as request body (e.g., in POST).

**API Interface Example:**
```kotlin
import retrofit2.Response
import retrofit2.http.*

interface ApiService {
    @GET("products")
    suspend fun getProducts(): Response<List<Product>>

    @GET("products/{id}")
    suspend fun getProductById(@Path("id") id: Int): Response<Product>

    @POST("products")
    suspend fun addProduct(@Body product: Product): Response<Product>

    @PUT("products/{id}")
    suspend fun updateProduct(@Path("id") id: Int, @Body product: Product): Response<Product>

    @DELETE("products/{id}")
    suspend fun deleteProduct(@Path("id") id: Int): Response<Unit>
}
```

Wrapping returned type in `Response<T>` allows reading HTTP status code (`response.code()`) and error body (`response.errorBody()`). Retrofit doesn't throw exception for server errors (e.g., `404`, `500`) – without `Response<T>` can't distinguish them from successful response.

---

### Creating Retrofit Instance

Best to keep Retrofit instance as singleton (e.g., in companion object or `Application` class) to avoid creating it multiple times.

#### With Moshi Converter

```kotlin
import retrofit2.Retrofit
import retrofit2.converter.moshi.MoshiConverterFactory
import com.squareup.moshi.Moshi
import com.squareup.moshi.kotlin.reflect.KotlinJsonAdapterFactory

val moshi = Moshi.Builder()
    .addLast(KotlinJsonAdapterFactory())
    .build()

val retrofit = Retrofit.Builder()
    .baseUrl("https://your-api.com/api/") // must end with /
    .addConverterFactory(MoshiConverterFactory.create(moshi))
    .build()

val api = retrofit.create(ApiService::class.java)
```

#### With kotlinx-serialization

```kotlin
import com.jakewharton.retrofit2.converter.kotlinx.serialization.asConverterFactory
import kotlinx.serialization.json.Json
import okhttp3.MediaType.Companion.toMediaType

val json = Json {
    ignoreUnknownKeys = true  // ignore unknown fields in JSON
    coerceInputValues = true  // use default value for missing non-nullable fields
}

val retrofit = Retrofit.Builder()
    .baseUrl("https://your-api.com/api/")
    .addConverterFactory(json.asConverterFactory("application/json; charset=UTF8".toMediaType()))
    .build()

val api = retrofit.create(ApiService::class.java)
```

> `ignoreUnknownKeys = true` particularly useful when server returns more fields than defined in model.

#### Logging HTTP Requests

During app development useful to log HTTP request and response content. Use `HttpLoggingInterceptor` from OkHttp library:

```kotlin
import okhttp3.OkHttpClient
import okhttp3.logging.HttpLoggingInterceptor

val clientBuilder = OkHttpClient.Builder()

if (BuildConfig.DEBUG) {
    val interceptor = HttpLoggingInterceptor().apply {
        level = HttpLoggingInterceptor.Level.BODY
    }
    clientBuilder.addInterceptor(interceptor)
}

val client = clientBuilder.build()

val retrofit = Retrofit.Builder()
    .baseUrl("https://your-api.com/api/")
    .addConverterFactory(MoshiConverterFactory.create(moshi))
    .client(client)
    .build()
```

`BuildConfig.DEBUG` is constant automatically generated by build system – is `true` in `debug` variant and `false` in `release` variant. Thanks to this, interceptor added only during app creation and doesn't reach production version.

> Even with `BuildConfig.DEBUG`, `logging-interceptor` dependency should be declared as `debugImplementation` – `HttpLoggingInterceptor` class won't be available in `release` variant, preventing accidental use.

---

### Repository Pattern

Following [Google architecture guidelines](https://developer.android.com/topic/architecture), ViewModel shouldn't directly use `ApiService`. **Repository** pattern – intermediary layer between ViewModel and data source – is recommended:

```
ViewModel → Repository → ApiService (Retrofit)
```

Repository responsible for fetching data and providing it to view model. HTTP error code handling should be closed inside Repository – ViewModel should receive ready data or exception:

```kotlin
class ProductsRepository(private val api: ApiService) {

    suspend fun getProducts(): List<Product> {
        val response = api.getProducts()
        if (response.isSuccessful) {
            return response.body() ?: emptyList()
        } else {
            throw Exception("HTTP Error ${response.code()}")
        }
    }

    suspend fun addProduct(product: Product): Product {
        val response = api.addProduct(product)
        if (response.isSuccessful) {
            return response.body() ?: throw Exception("Empty server response")
        } else {
            throw Exception("HTTP Error ${response.code()}")
        }
    }
}
```

---

### Example Usage in ViewModel

```kotlin
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.launch

class ProductsViewModel(private val repository: ProductsRepository) : ViewModel() {

    private val _products = MutableStateFlow<List<Product>>(emptyList())
    val products: StateFlow<List<Product>> = _products

    private val _error = MutableStateFlow<String?>(null)
    val error: StateFlow<String?> = _error

    fun getProducts() {
        viewModelScope.launch {
            try {
                _products.value = repository.getProducts()
                _error.value = null
            } catch (e: Exception) {
                _error.value = e.message
            }
        }
    }
}
```

State observed in composable function through `collectAsStateWithLifecycle()`:

```kotlin
val products by viewModel.products.collectAsStateWithLifecycle()
val error by viewModel.error.collectAsStateWithLifecycle()

if (error != null) {
    Text("Error: $error")
} else {
    LazyColumn {
        items(products) { product ->
            Text(product.name)
        }
    }
}
```

---

## Best Practices

- In API interfaces recommend using `suspend fun` with `Response<T>` wrapping to distinguish network errors (exception) from HTTP errors (codes 4xx/5xx).
- HTTP error code handling should be closed in Repository layer – ViewModel should receive ready data or exception.
- Keep Retrofit instance as singleton.
- Recommend using Repository pattern as intermediary layer between ViewModel and `ApiService`.
- `HttpLoggingInterceptor` should be added only in `debug` variant.

---

**More Information:**  
- [Official Retrofit Documentation](https://square.github.io/retrofit/)
- [Retrofit Usage Examples](https://github.com/square/retrofit/tree/master/samples)
- [Official Moshi Documentation](https://github.com/square/moshi)

---

### **Next topic:** [Firebase Platform](ENt/13%20Firebase%20Platform.md)
