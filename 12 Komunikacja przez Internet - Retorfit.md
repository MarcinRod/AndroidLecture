# Komunikacja przez Internet w Androidzie

Współczesne aplikacje mobilne bardzo często komunikują się z serwerami przez Internet, aby pobierać lub wysyłać dane (np. pobieranie listy produktów, logowanie użytkownika, synchronizacja danych). W Androidzie najczęściej do tego celu wykorzystuje się biblioteki takie jak **Retrofit** lub **OkHttp**.

---

## Pozwolenie na dostęp do Internetu

Aby aplikacja mogła komunikować się z serwerem przez Internet, należy dodać odpowiednie pozwolenie w pliku `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

Bez tego pozwolenia żadne zapytanie sieciowe (np. przez Retrofit, OkHttp) nie zadziała, a aplikacja otrzyma błąd połączenia.

**Ważne:**  
To pozwolenie jest wymagane w każdej aplikacji, która korzysta z Internetu – zarówno do pobierania, jak i wysyłania danych.

---


## Najważniejsze pojęcia

- **API (Application Programming Interface)** – zestaw reguł i adresów (punktów końcowych ang. *endpoint*), przez które aplikacja komunikuje się z serwerem.
- **REST API** – najpopularniejszy styl komunikacji, oparty na protokole HTTP i operacjach takich jak GET, POST, PUT, DELETE.
- **JSON** – najczęściej używany format wymiany danych między aplikacją a serwerem.

---

## Popularne biblioteki do komunikacji

- **Retrofit** – najpopularniejsza biblioteka do obsługi REST API w Androidzie. Umożliwia łatwe wykonywanie zapytań HTTP i odwzorowywanie odpowiedzi na obiekty Kotlin.
- **OkHttp** – niskopoziomowa biblioteka HTTP, na której opiera się Retrofit.

---

## Retrofit

**Retrofit** to najpopularniejsza biblioteka do komunikacji z REST API w Androidzie. Pozwala wykonywać zapytania HTTP i odwzorowywać odpowiedzi JSON na obiekty Kotlin. Obsługuje wymienne konwertery JSON — w tym Moshi oraz kotlinx-serialization.

---

### Zależności (build.gradle.kts)

**Z konwerterem Moshi:**
```kotlin
implementation("com.squareup.retrofit2:retrofit:2.11.0")
implementation("com.squareup.retrofit2:converter-moshi:2.11.0")
implementation("com.squareup.moshi:moshi:1.15.1")
implementation("com.squareup.moshi:moshi-kotlin:1.15.1")
// Rejestrowanie zapytań HTTP – tylko w wariancie debug
debugImplementation("com.squareup.okhttp3:logging-interceptor:4.12.0")
```

**Z konwerterem kotlinx-serialization:**
```kotlin
plugins {
    kotlin("plugin.serialization") version "2.0.0"
}

dependencies {
    implementation("com.squareup.retrofit2:retrofit:2.11.0")
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.7.3")
    implementation("com.jakewharton.retrofit:retrofit2-kotlinx-serialization-converter:1.0.0")
    // Rejestrowanie zapytań HTTP – tylko w wariancie debug
    debugImplementation("com.squareup.okhttp3:logging-interceptor:4.12.0")
}
```

---

### Model danych

#### Z konwerterem Moshi

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

**Adnotacje Moshi:**
- `@JsonClass(generateAdapter = true)` – powoduje wygenerowanie adaptera do zamiany obiektu na JSON i odwrotnie. Jest wymagana do poprawnej obsługi klas danych w Kotlinie.
- `@Json(name = "klucz")` – przypisuje pole klasy do pola o innej nazwie w JSON-ie.
- Pola z typem `String?` i wartością domyślną `null` są opcjonalne — jeśli nie pojawią się w JSON-ie, przyjmą wartość `null`.

**Ważne:** jeśli pole nie jest nullable i nie ma wartości domyślnej, a zabraknie go w JSON-ie, Moshi zgłosi błąd podczas odczytu.

#### Z kotlinx-serialization

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

**Adnotacje kotlinx-serialization:**
- `@Serializable` – oznacza klasę do serializacji; adapter jest generowany w czasie kompilacji przez wtyczkę kompilatora.
- `@SerialName("klucz")` – odpowiednik `@Json(name = ...)` z Moshi.
- Pola z wartością domyślną są opcjonalne — jeśli nie pojawią się w JSON-ie, przyjmą podaną wartość.

---

### Definicja interfejsu API

Interfejs API w Retrofit zawiera metody odpowiadające poszczególnym zapytaniom HTTP. Każda metoda jest oznaczona adnotacją określającą typ zapytania i ścieżkę punktu końcowego.

**Najważniejsze adnotacje Retrofit:**
- `@GET("endpoint")` – zapytanie pobierające dane.
- `@POST("endpoint")` – zapytanie wysyłające dane.
- `@PUT("endpoint")` – zapytanie aktualizujące dane.
- `@DELETE("endpoint")` – zapytanie usuwające dane.
- `@Query("param")` – parametr przekazywany w adresie URL (np. `?id=1`).
- `@Path("param")` – parametr dynamiczny w ścieżce URL (np. `/produkty/{id}`).
- `@Body` – obiekt przesyłany jako treść zapytania (np. w POST).

**Przykład interfejsu API:**
```kotlin
import retrofit2.Response
import retrofit2.http.*

interface ApiService {
    @GET("produkty")
    suspend fun getProducts(): Response<List<Product>>

    @GET("produkty/{id}")
    suspend fun getProduktById(@Path("id") id: Int): Response<Product>

    @POST("produkty")
    suspend fun addProduct(@Body product: Product): Response<Product>

    @PUT("produkty/{id}")
    suspend fun updateProduct(@Path("id") id: Int, @Body product: Product): Response<Product>

    @DELETE("produkty/{id}")
    suspend fun deleteProduct(@Path("id") id: Int): Response<Unit>
}
```

Opakowanie zwracanego typu w `Response<T>` pozwala odczytać kod statusu HTTP (`response.code()`) oraz treść błędu (`response.errorBody()`). Retrofit nie zgłasza wyjątku dla błędów serwera (np. `404`, `500`) — bez `Response<T>` nie można ich odróżnić od prawidłowej odpowiedzi.

---

### Tworzenie instancji Retrofit

Instancję Retrofit najlepiej przechowywać jako singleton (np. w obiekcie towarzyszącym lub klasie `Application`), aby nie tworzyć jej wielokrotnie.

#### Z konwerterem Moshi

```kotlin
import retrofit2.Retrofit
import retrofit2.converter.moshi.MoshiConverterFactory
import com.squareup.moshi.Moshi
import com.squareup.moshi.kotlin.reflect.KotlinJsonAdapterFactory

val moshi = Moshi.Builder()
    .addLast(KotlinJsonAdapterFactory())
    .build()

val retrofit = Retrofit.Builder()
    .baseUrl("https://twoje-api.pl/api/") // adres musi kończyć się ukośnikiem /
    .addConverterFactory(MoshiConverterFactory.create(moshi))
    .build()

val api = retrofit.create(ApiService::class.java)
```

#### Z kotlinx-serialization

```kotlin
import com.jakewharton.retrofit2.converter.kotlinx.serialization.asConverterFactory
import kotlinx.serialization.json.Json
import okhttp3.MediaType.Companion.toMediaType

val json = Json {
    ignoreUnknownKeys = true  // ignoruje nieznane pola w JSON-ie
    coerceInputValues = true  // dla brakujących pól non-nullable używa wartości domyślnej
}

val retrofit = Retrofit.Builder()
    .baseUrl("https://twoje-api.pl/api/")
    .addConverterFactory(json.asConverterFactory("application/json; charset=UTF8".toMediaType()))
    .build()

val api = retrofit.create(ApiService::class.java)
```

> `ignoreUnknownKeys = true` jest szczególnie przydatne, gdy serwer zwraca więcej pól niż zdefiniowano w modelu.

#### Rejestrowanie zapytań HTTP

Podczas tworzenia aplikacji przydatne jest rejestrowanie treści zapytań i odpowiedzi HTTP. Służy do tego `HttpLoggingInterceptor` z biblioteki OkHttp:

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
    .baseUrl("https://twoje-api.pl/api/")
    .addConverterFactory(MoshiConverterFactory.create(moshi))
    .client(client)
    .build()
```

`BuildConfig.DEBUG` to stała generowana automatycznie przez system budowania — przyjmuje wartość `true` w wariancie `debug` i `false` w wariancie `release`. Dzięki temu interceptor jest dodawany tylko podczas tworzenia aplikacji i nie trafia do wersji produkcyjnej.

> Nawet przy użyciu `BuildConfig.DEBUG`, zależność `logging-interceptor` powinna być zadeklarowana jako `debugImplementation` — klasa `HttpLoggingInterceptor` nie będzie wtedy dostępna w wariancie `release`, co uniemożliwia jej przypadkowe użycie.


### Wzorzec Repository

Zgodnie z [wytycznymi architektury Google](https://developer.android.com/topic/architecture) ViewModel nie powinien bezpośrednio korzystać z `ApiService`. Zalecanym podejściem jest wzorzec **Repository**, który stanowi warstwę pośrednią między ViewModelem a źródłem danych:

```
ViewModel → Repository → ApiService (Retrofit)
```

Repository odpowiada za pobieranie danych i udostępnianie ich modelowi widoku. Obsługę kodów błędów HTTP należy zamknąć wewnątrz Repository — ViewModel powinien otrzymywać gotowe dane lub wyjątek:

```kotlin
class ProductsRepository(private val api: ApiService) {

    suspend fun getProducts(): List<Product> {
        val response = api.getProducts()
        if (response.isSuccessful) {
            return response.body() ?: emptyList()
        } else {
            throw Exception("Błąd HTTP ${response.code()}")
        }
    }

    suspend fun addProduct(product: Product): Product {
        val response = api.addProduct(product)
        if (response.isSuccessful) {
            return response.body() ?: throw Exception("Pusta odpowiedź serwera")
        } else {
            throw Exception("Błąd HTTP ${response.code()}")
        }
    }
}
```

---

### Przykład użycia w ViewModel

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

Stan jest obserwowany w funkcji kompozycyjnej przez `collectAsStateWithLifecycle()`:

```kotlin
val products by viewModel.products.collectAsStateWithLifecycle()
val error by viewModel.error.collectAsStateWithLifecycle()

if (error != null) {
    Text("Błąd: $error")
} else {
    LazyColumn {
        items(products) { product ->
            Text(product.name)
        }
    }
}
```

---

## Dobre praktyki

- W interfejsach API zalecane jest stosowanie `suspend fun` z opakowaniem `Response<T>`, aby odróżnić błędy sieciowe (wyjątek) od błędów HTTP (kody 4xx/5xx).
- Obsługę kodów HTTP należy zamknąć w warstwie Repository — ViewModel powinien otrzymywać gotowe dane lub wyjątek.
- Instancję Retrofit należy przechowywać jako singleton.
- Zalecane jest stosowanie wzorca Repository jako warstwy pośredniej między ViewModelem a `ApiService`.
- `HttpLoggingInterceptor` należy dodawać wyłącznie w wariancie `debug`.


---

**Więcej informacji:**  
- [Oficjalna dokumentacja Retrofit](https://square.github.io/retrofit/)
- [Przykłady użycia Retrofit](https://github.com/square/retrofit/tree/master/samples)
- [Oficjalna dokumentacja Moshi](https://github.com/square/moshi)

---

### **Następny temat:** [Platforma Firebase](https://github.com/MarcinRod/AndroidLecture2025/blob/main/13%20Platforma%20Firebase.md)