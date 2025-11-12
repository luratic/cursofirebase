# Firebase Analytics (Android) · Integración Analytics

Este README está pensado para **copiar y pegar** en un proyecto Android (Kotlin) y cubrir, en orden, la puesta a punto y los ejemplos de:

- Inicialización de Firebase Analytics
- **User ID** y **User Properties**
- **Default Event Parameters**
- **Eventos simples** (custom y recomendados)
- **Eventos de e-commerce** (view_item, add_to_cart, purchase)
- **DebugView, Logcat y ADB** (para ver eventos en tiempo real)

> Todos los snippets usan **KTX** y la misma Activity del proyecto **Basic Views Activity** de Android Studio.

---

## 0) Requisitos y configuración rápida

1. La forma que usaremos para añadir Firebase Analytics será desde Android Studio > Tools > Firebase > Analytics > **Get started with Google Analytics**
2. Connect your app to Firebase > Connect To Firebase > New Project or Choose Project
3. Connect 
4. Add the Analytics SDK to your app > Accept Changes


### Método Alternativo

1. **Agrega Firebase** al proyecto (consola de Firebase → añade app Android → descarga `google-services.json`).
2. **Gradle raíz (`settings.gradle` o `build.gradle` del proyecto)**

```gradle
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.google.gms.google.services) apply false
}
```

3. **Gradle módulo app (`app/build.gradle`)**

```gradle
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.google.gms.google.services)
}

dependencies {
    implementation(libs.firebase.analytics)
}
```


---

## 1) Imports (Kotlin)

```kotlin
// Analytics
import com.google.firebase.Firebase
import com.google.firebase.analytics.analytics
import com.google.firebase.analytics.FirebaseAnalytics
import com.google.firebase.analytics.logEvent
```

---

## 2) Inicialización en tu Activity

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var appBarConfiguration: AppBarConfiguration
    private lateinit var binding: ActivityMainBinding

    private lateinit var analytics: FirebaseAnalytics

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // 1) Obtener instancia de Analytics 
        analytics = Firebase.analytics

        //Envío de evento de arranque simple
        analytics.logEvent("app_open_manual") {
            param(FirebaseAnalytics.Param.CONTENT_TYPE, "view")
        }

        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        setSupportActionBar(binding.toolbar)

        val navController = findNavController(R.id.nav_host_fragment_content_main)
        appBarConfiguration = AppBarConfiguration(navController.graph)
        setupActionBarWithNavController(navController, appBarConfiguration)

        binding.fab.setOnClickListener { view ->
            //Envío evento al pulsar en el botón de email
            analytics.logEvent("send_email") {
                param(FirebaseAnalytics.Param.METHOD, "email")
                param(type, "boton")
            }
            Snackbar.make(view, "Replace with your own action", Snackbar.LENGTH_LONG)
                .setAction("Action", null)
                .setAnchorView(R.id.fab).show()
        }
    }
}
```

---

## 3) User ID (identificar usuarios logados)

```kotlin
analytics.setUserId("user_12345")
```

**Buenas prácticas**
- No uses emails en claro. ID estable, no PII.
- Llama al método cada vez que se abra la aplicación si el usuario sigue autenticado.

[Documentación Oficial](https://firebase.google.com/docs/analytics/userid)

---

## 4) User Properties

```kotlin
analytics.setUserProperty("plan", "premium")
analytics.setUserProperty("country", "ES")
analytics.setUserProperty("app_tier", "beta")
analytics.setUserProperty("app_tier", null) // eliminar
```
[Documentación Oficial](https://firebase.google.com/docs/analytics/user-properties?platform=android)


---

## 5) Default Event Parameters

```kotlin
val defaults = bundleOf(
    "currency" to "EUR",
    "environment" to "staging",
    "app_version" to BuildConfig.VERSION_NAME
)
analytics.setDefaultEventParameters(defaults)
```
> Se aplican a todos los eventos futuros, salvo que se sobrescriban.
[Documentación Oficial](https://firebase.google.com/docs/analytics/events?platform=android#set_default_event_parameters_2)

---

## 6) Eventos simples

```kotlin
analytics.logEvent("mi_evento") {
    param(FirebaseAnalytics.Param.CONTENT_TYPE, "boton")
    param("cta_label", "email")
}

analytics.logEvent(FirebaseAnalytics.Event.SCREEN_VIEW) {
    param(FirebaseAnalytics.Param.SCREEN_NAME, "Home")
    param(FirebaseAnalytics.Param.SCREEN_CLASS, "MainActivity")
}
```
[Documentación Oficial](https://firebase.google.com/docs/analytics/events?platform=android)

---

## 7) E-commerce (GA4 recomendado)

Algunos ejemplos:
- `view_item`
- `add_to_cart`
- `begin_checkout`
- `add_payment_info`
- `purchase`

### view_item
```kotlin
val item = Bundle().apply {
    putString("item_id", "SKU_123")
    putString("item_name", "Zapatilla SuperRun")
    putDouble("price", 79.99)
    putString("item_brand", "Datola")
    putString("item_category", "Calzado")
}
Bundle().apply {
    putString("currency", "EUR")
    putParcelableArrayList("items", arrayListOf(item))
}.also { analytics.logEvent("view_item", it) }
```

### add_to_cart
```kotlin
val cartItem = Bundle().apply {
    putString("item_id", "SKU_123")
    putString("item_name", "Zapatilla SuperRun")
    putDouble("price", 79.99)
    putLong("quantity", 1)
}
Bundle().apply {
    putString("currency", "EUR")
    putDouble("value", 79.99)
    putParcelableArrayList("items", arrayListOf(cartItem))
}.also { analytics.logEvent("add_to_cart", it) }
```

### purchase
```kotlin
val items = arrayListOf(
    Bundle().apply { putString("item_id","SKU_123"); putString("item_name","Zapatilla SuperRun"); putDouble("price",79.99); putLong("quantity",1) },
    Bundle().apply { putString("item_id","SKU_456"); putString("item_name","Calcetines Pro"); putDouble("price",9.99); putLong("quantity",2) }
)
val subtotal = 79.99 + 2 * 9.99
val shipping = 3.99
val tax = 6.00
val total = subtotal + shipping + tax

Bundle().apply {
    putString("transaction_id", "T_${System.currentTimeMillis()}")
    putString("currency", "EUR")
    putDouble("value", total)
    putDouble("shipping", shipping)
    putDouble("tax", tax)
    putParcelableArrayList("items", items)
}.also { analytics.logEvent("purchase", it) }
```
[Documentación Oficial](https://firebase.google.com/docs/analytics/measure-ecommerce)

---

## 8) Dónde colocar cada cosa

- `setUserId(...)`: tras login o al inicio si sigue autenticado.
- `setUserProperty(...)`: al cambiar atributo (plan, país...). 
- `setDefaultEventParameters(...)`: al iniciar app, cuando se tenga el valor o se actualice.
- Eventos e-commerce: en el punto real del flujo (detalle → carrito → checkout → compra).

---

## 9) DebugView (ver eventos en tiempo real)

```bash
adb shell setprop debug.firebase.analytics.app <nombre_paquete_aplicacion> 
# Desactivar
adb shell setprop debug.firebase.analytics.app .none.
```

Abre Firebase Console → Analytics → DebugView.

### Extensión para facilitar la visualización del DebugView en la consola del navegador
[GA4 Enhanced DebugView by Luratic](https://chromewebstore.google.com/detail/hgkkhcgpdigijpbclngegpckcabpjjej)

---

## 10) Logcat: ver trazas de Firebase Analytics

```bash
# Activar logs verbosos
adb shell setprop log.tag.FA VERBOSE
adb shell setprop log.tag.FA-SVC VERBOSE
# Ver sólo Analytics
adb logcat -v time -s FA FA-SVC 

```

---

## 11) Añadir `adb` al PATH

### Windows (CMD)

```cmd
setx PATH "%PATH%;C:\Users\%USERNAME%\AppData\Local\Android\Sdk\platform-tools"
```

### macOS (zsh)

```bash
echo 'export PATH="$PATH:$HOME/Library/Android/sdk/platform-tools"' >> ~/.zshrc
source ~/.zshrc
```

Verifica:
```bash
adb version
```
---

## 12) Troubleshooting rápido

- **No veo eventos** → activa DebugView y Logcat.
- **Valores `null`** → evita enviar `null`, mejor omitir.
- **Default params** → sólo aplican a eventos futuros.
