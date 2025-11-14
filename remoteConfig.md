# Firebase Remote Config + Analytics (Android/Kotlin)

Ejemplo simple para integrar **Remote Config** y **Firebase Analytics** en una `MainActivity`.

### Configuración rápida

1. La forma que usaremos para añadir Firebase Analytics será desde Android Studio > Tools > Firebase > Remote Config > **Get started with Google Analytics**
2. Connect your app to Firebase > Connect To Firebase > New Project or Choose Project
3. Connect 
4. Add the Analytics SDK to your app > Accept Changes

## 1. Parámetros que vams Firebase Remote Config

| Parámetro              | Ejemplo                          | Descripción |
|------------------------|----------------------------------|-------------|
| snackbar_text          | Mensaje desde Remote Config     | Texto del Snackbar |
| ga4_remote     | valor_ejemplo                         | Parámetro dinámico enviado a GA4 |

---

## 2. Dependencias necesarias 

```gradle
// Firebase Remote Config → obtener valores dinámicos desde Firebase
    implementation(libs.firebase.config)
```

---

## 3. Imports necesarios

```kotlin
//RemoteConfig
import com.google.firebase.remoteconfig.remoteConfig
import com.google.firebase.remoteconfig.remoteConfigSettings
```

---

## 4. Ejemplo simple en MainActivity

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    binding = ActivityMainBinding.inflate(layoutInflater)
    setContentView(binding.root)

    val analytics = Firebase.analytics
    //Variable RemoteConfig
    val remoteConfig = Firebase.remoteConfig

    // Configuración de Remote Config (intervalo mínimo = 0 → para pruebas)
    val configSettings = remoteConfigSettings {
        minimumFetchIntervalInSeconds = 0
    }
    remoteConfig.setConfigSettingsAsync(configSettings)

    // Valores por defecto si no hay red o primera ejecución
    remoteConfig.setDefaultsAsync(
        mapOf(
            "snackbar_text" to "Texto por defecto 📱",
            "ga4_remote" to "default"
        )
    )

    //Primera vez - descarga de Firebase
    remoteConfig.fetchAndActivate()

    // Leer valores activos
    val message = remoteConfig.getString("snackbar_text")
    val ga4Remote = remoteConfig.getString("ga4_remote")

    // Enviar a GA4 un parámetro dinámico
    analytics.logEvent("remote_config_loaded") {
        param("ga4Remote", variant)
    }

    // Usar valores remotos en la UI
    binding.fab.setOnClickListener { view ->
        analytics.logEvent("click_fab") {
            param("ga4Remote", variant)
        }
        Snackbar.make(view, message, Snackbar.LENGTH_LONG)
            .setAction("Action") {
                analytics.logEvent("snackbar_action_click") {
                    param("experiment_variant", variant)
                }
            }
            .setAnchorView(binding.fab)
            .show()
    }
}
```

---

## 5. DebugView (ver los eventos en GA4)

```bash
adb shell setprop debug.firebase.analytics.app com.example.brais
```

Firebase Console → Analytics → **DebugView**.

---

## Resultado

✔ Texto del Snackbar configurable desde Remote Config  
✔ Texto del botón de acción también remoto  
✔ Parámetro dinámico enviado a GA4  
✔ Ejemplo ultra simple y fácil de copiar  
