# 3.3 Notificaciones

> **Unidad 2 · Aplicaciones Android. Interfaz de usuario**
> Módulo: Programación multimedia y dispositivos móviles (2º DAM)

## Objetivos

Al terminar este apartado deberías ser capaz de:

- Explicar qué es una notificación y sus partes.
- Crear un canal de notificaciones y elegir su nivel de importancia.
- Declarar y solicitar el permiso `POST_NOTIFICATIONS` en Android 13 y posteriores.
- Crear y mostrar una notificación con `NotificationCompat.Builder`.
- Abrir la app al pulsar la notificación, y actualizar o eliminar notificaciones.

---

## 1. Qué es una notificación

Una **notificación** es un mensaje que Android muestra **fuera de la interfaz de la app**: en la barra de estado, en el panel de notificaciones, en la pantalla de bloqueo o, si es urgente, como un aviso flotante sobre la pantalla. Sirve para avisar al usuario **aunque no esté usando la app**.

```
Barra de estado:          [icono]  12:30

Panel de notificaciones:
┌──────────────────────────────────────┐
│ [icono] Pizzería DAM · ahora         │  ← icono pequeño, app y hora
│ Tu pedido está en camino             │  ← título
│ Llegará en unos 20 minutos           │  ← texto
│ VER PEDIDO     LLAMAR                │  ← acciones (opcionales)
└──────────────────────────────────────┘
```

### Partes de una notificación

| Parte | Obligatoria | Método |
|---|---|---|
| Icono pequeño | ✅ | `setSmallIcon()` |
| Título | Recomendada | `setContentTitle()` |
| Texto | Recomendada | `setContentText()` |
| Acción al pulsarla | Recomendada | `setContentIntent()` |
| Botones de acción | Opcional (hasta 3) | `addAction()` |
| Nombre de la app y hora | Las añade el sistema | — |

Mostrar una notificación requiere **tres pasos**, que veremos en orden:

1. **Crear un canal** de notificaciones (Android 8 y posteriores).
2. **Tener permiso** para notificar (Android 13 y posteriores).
3. **Crear y mostrar** la notificación.

---

## 2. El canal de notificaciones

### 2.1 Qué es un canal

Desde **Android 8.0 (API 26)**, toda notificación pertenece a un **canal**. Un canal agrupa notificaciones del mismo tipo, por ejemplo "Estado del pedido" u "Ofertas", y el **usuario** decide para cada canal si quiere recibirlas, con qué sonido o si deben aparecer en la pantalla de bloqueo.

```
Ajustes → Apps → Pizzería DAM → Notificaciones
  ├── Estado del pedido   [ON]   Importancia alta, con sonido
  └── Ofertas             [OFF]  ← el usuario las ha desactivado
```

Así el usuario puede silenciar las ofertas **sin dejar de recibir** los avisos de su pedido.

> ⚠️ En Android 8 o superior, una notificación **sin canal no se muestra**. No da ningún error: simplemente no aparece.

### 2.2 Niveles de importancia

Al crear el canal se indica su **importancia**, que determina cómo de intrusivas son sus notificaciones:

| Importancia | Comportamiento | Ejemplo de uso |
|---|---|---|
| `IMPORTANCE_HIGH` | Sonido y aviso flotante sobre la pantalla (*heads-up*). | Mensajes de chat, llamadas. |
| `IMPORTANCE_DEFAULT` | Sonido, sin aviso flotante. | Estado de un pedido. |
| `IMPORTANCE_LOW` | Sin sonido. Aparece en la barra de estado. | Recomendaciones. |
| `IMPORTANCE_MIN` | Sin sonido ni icono en la barra de estado. Solo en el panel. | Información poco relevante. |

### 2.3 Crear el canal

```kotlin
import android.app.NotificationChannel
import android.app.NotificationManager
import android.os.Build

class MainActivity : AppCompatActivity() {

    companion object {
        const val CANAL_PEDIDOS = "canal_pedidos"   // id del canal
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // ... binding y setContentView ...
        crearCanalNotificaciones()
    }

    private fun crearCanalNotificaciones() {
        // Los canales solo existen desde Android 8.0 (API 26)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val canal = NotificationChannel(
                CANAL_PEDIDOS,                              // id interno
                getString(R.string.canal_pedidos_nombre),   // nombre visible en Ajustes
                NotificationManager.IMPORTANCE_DEFAULT      // importancia
            ).apply {
                description = getString(R.string.canal_pedidos_descripcion)
            }

            val manager = getSystemService(NotificationManager::class.java)
            manager.createNotificationChannel(canal)
        }
    }
}
```

```xml
<string name="canal_pedidos_nombre">Estado del pedido</string>
<string name="canal_pedidos_descripcion">Avisos sobre la preparación y el envío de tus pedidos</string>
```

**Aspectos importantes:**

- El `if` es necesario porque nuestra API mínima es la 24, y `NotificationChannel` no existe antes de la 26.
- **Crear un canal que ya existe no hace nada**, así que se puede llamar a `crearCanalNotificaciones()` en cada `onCreate()` sin problema.
- El **nombre** y la **descripción** los ve el usuario en Ajustes: deben estar en `strings.xml`.
- Una vez creado el canal, **la app ya no puede cambiar su importancia**: ahora es decisión del usuario. Si necesitas otra importancia durante las pruebas, desinstala la app o usa otro id de canal.

> 💡 En apps reales, los canales suelen crearse en una clase que hereda de `Application`, que se ejecuta una sola vez al arrancar la app. En nuestros ejemplos basta con la actividad principal.

---

## 3. El permiso de notificaciones (Android 13+)

Desde **Android 13 (API 33)**, mostrar notificaciones requiere un **permiso que el usuario debe conceder**. Es un permiso de tiempo de ejecución: no basta con declararlo, hay que **pedirlo**.

### 3.1 Declararlo en el Manifest

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />

    <application ...>
        ...
    </application>
</manifest>
```

> 📝 **Recuerda (UT01 · estructura de la app):** los permisos se declaran en `AndroidManifest.xml`, fuera de la etiqueta `<application>`.

### 3.2 Pedirlo al usuario

Se usa la API *Activity Result* con el contrato `RequestPermission`. El sistema muestra un diálogo con las opciones **Permitir** y **No permitir**:

```kotlin
import android.Manifest
import android.content.pm.PackageManager
import androidx.activity.result.contract.ActivityResultContracts
import androidx.core.content.ContextCompat

class MainActivity : AppCompatActivity() {

    // Se registra como propiedad: debe crearse ANTES de que la actividad se muestre
    private val pedirPermiso = registerForActivityResult(
        ActivityResultContracts.RequestPermission()
    ) { concedido ->
        if (concedido) {
            mostrarNotificacionPedido()
        } else {
            Snackbar.make(binding.root, R.string.sin_permiso_notificaciones, Snackbar.LENGTH_LONG).show()
        }
    }

    private fun tienePermisoNotificaciones(): Boolean =
        Build.VERSION.SDK_INT < Build.VERSION_CODES.TIRAMISU ||   // antes de Android 13 no hace falta
            ContextCompat.checkSelfPermission(
                this, Manifest.permission.POST_NOTIFICATIONS
            ) == PackageManager.PERMISSION_GRANTED

    private fun notificarPedido() {
        if (tienePermisoNotificaciones()) {
            mostrarNotificacionPedido()
        } else {
            pedirPermiso.launch(Manifest.permission.POST_NOTIFICATIONS)
        }
    }
}
```

| Elemento | Qué hace |
|---|---|
| `registerForActivityResult(...)` | Prepara la petición y define qué hacer con la respuesta (la lambda recibe `true` o `false`). |
| `checkSelfPermission(...)` | Comprueba si el permiso ya está concedido. |
| `pedirPermiso.launch(...)` | Muestra el diálogo del sistema. |

> ⚠️ Si el usuario rechaza el permiso **dos veces**, Android deja de mostrar el diálogo: al llamar a `launch()`, la respuesta llega directamente como `false`. A partir de ahí, solo puede activarlo desde los Ajustes del sistema. Por eso conviene pedirlo **en el momento en que tiene sentido** (al hacer un pedido), no nada más abrir la app sin explicación.

> 📌 Con API 32 o inferior, el permiso se concede automáticamente al instalar la app. Por eso la función devuelve `true` directamente en esas versiones.

---

## 4. Crear y mostrar una notificación

### 4.1 El icono pequeño

El icono pequeño es **obligatorio**: sin él la app falla al mostrar la notificación. Debe ser un icono **monocromo** (blanco sobre transparente), porque el sistema lo tiñe de un solo color.

La forma más sencilla de crearlo: clic derecho en `res/drawable` → **New → Vector Asset** → *Clip Art*, elige un icono (por ejemplo, *notifications*) y llámalo `ic_notificacion`.

> ⚠️ No uses el icono de la app (`R.mipmap.ic_launcher`): es a color, y en la barra de estado se verá como un cuadrado blanco.

### 4.2 Construir la notificación

Se usa `NotificationCompat.Builder`, de AndroidX, que funciona igual en todas las versiones de Android:

```kotlin
import androidx.core.app.NotificationCompat
import androidx.core.app.NotificationManagerCompat

companion object {
    const val CANAL_PEDIDOS = "canal_pedidos"
    const val ID_NOTIF_PEDIDO = 1
}

private fun mostrarNotificacionPedido() {
    val notificacion = NotificationCompat.Builder(this, CANAL_PEDIDOS)
        .setSmallIcon(R.drawable.ic_notificacion)              // obligatorio
        .setContentTitle(getString(R.string.notif_titulo))     // "Tu pedido está en camino"
        .setContentText(getString(R.string.notif_texto))       // "Llegará en unos 20 minutos"
        .setPriority(NotificationCompat.PRIORITY_DEFAULT)      // para Android 7.1 o inferior
        .setAutoCancel(true)                                   // se borra al pulsarla
        .build()

    // Comprobación obligatoria: el compilador avisa si falta
    if (tienePermisoNotificaciones()) {
        NotificationManagerCompat.from(this).notify(ID_NOTIF_PEDIDO, notificacion)
    }
}
```

| Elemento | Qué hace |
|---|---|
| `Builder(this, CANAL_PEDIDOS)` | Crea la notificación **en ese canal**. El id debe coincidir con el del canal creado. |
| `setPriority()` | En Android 7.1 o inferior no hay canales: la prioridad cumple esa función. Desde Android 8 se ignora y manda la importancia del canal. |
| `setAutoCancel(true)` | La notificación desaparece cuando el usuario la pulsa. |
| `build()` | Crea el objeto `Notification`. |
| `notify(id, notificacion)` | La muestra. El **id** identifica la notificación para actualizarla o borrarla después. |

> 📌 **Relación con POO:** `NotificationCompat.Builder` sigue el patrón **Builder**: en lugar de un constructor con muchos parámetros, cada método configura una parte y devuelve el propio *builder*, para encadenar llamadas. Lo mismo veremos con `AlertDialog.Builder` en el punto 3.4.

### 4.3 Abrir la app al pulsar la notificación

Por defecto, pulsar la notificación no hace nada. Para que abra una actividad hay que indicarlo con un **`PendingIntent`**: una "orden aplazada" que el sistema ejecutará en nombre de nuestra app cuando el usuario la pulse.

```kotlin
import android.app.PendingIntent
import android.content.Intent

val intent = Intent(this, MainActivity::class.java).apply {
    flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
}
val pendingIntent = PendingIntent.getActivity(
    this, 0, intent, PendingIntent.FLAG_IMMUTABLE
)

val notificacion = NotificationCompat.Builder(this, CANAL_PEDIDOS)
    .setSmallIcon(R.drawable.ic_notificacion)
    .setContentTitle(getString(R.string.notif_titulo))
    .setContentText(getString(R.string.notif_texto))
    .setContentIntent(pendingIntent)     // ← qué abrir al pulsarla
    .setAutoCancel(true)
    .build()
```

> 📝 **Avance del punto 4:** un `Intent` es un mensaje para abrir otra actividad. Lo estudiaremos a fondo en el punto 4. Por ahora, basta con saber que este código abre `MainActivity`.
>
> `FLAG_IMMUTABLE` es **obligatorio** desde Android 12: indica que nadie podrá modificar el intent antes de ejecutarlo.

---

## 5. Más opciones

### 5.1 Texto largo

`setContentText()` muestra **una sola línea**. Para un texto largo que se despliegue, se usa un estilo:

```kotlin
.setStyle(
    NotificationCompat.BigTextStyle()
        .bigText(getString(R.string.notif_texto_largo))
)
```

### 5.2 Botones de acción

```kotlin
.addAction(
    R.drawable.ic_llamar,                // icono (no se ve en todas las versiones)
    getString(R.string.llamar),          // texto del botón
    pendingIntentLlamar                  // qué hacer al pulsarlo
)
```

### 5.3 Actualizar y eliminar

- **Actualizar:** llama a `notify()` con el **mismo id**: la notificación se sustituye en lugar de duplicarse.
- **Eliminar:** `NotificationManagerCompat.from(this).cancel(ID_NOTIF_PEDIDO)`.
- **Eliminar todas las de la app:** `NotificationManagerCompat.from(this).cancelAll()`.

```kotlin
// Simulación del estado de un pedido: la misma notificación cambia de texto
private fun actualizarEstado(estado: String) {
    val notificacion = NotificationCompat.Builder(this, CANAL_PEDIDOS)
        .setSmallIcon(R.drawable.ic_notificacion)
        .setContentTitle(getString(R.string.app_name))
        .setContentText(estado)
        .setOnlyAlertOnce(true)    // suena solo la primera vez, no en cada actualización
        .build()
    if (tienePermisoNotificaciones()) {
        NotificationManagerCompat.from(this).notify(ID_NOTIF_PEDIDO, notificacion)
    }
}
```

---

## 6. Resumen del proceso

```
1. Canal          crearCanalNotificaciones()          (API 26+, una vez; repetirlo no afecta)
       │
2. Permiso        <uses-permission POST_NOTIFICATIONS>  en el Manifest
       │          checkSelfPermission / launch()        (API 33+)
       │
3. Notificación   NotificationCompat.Builder(ctx, CANAL)
                    .setSmallIcon(...)       ← obligatorio
                    .setContentTitle(...)
                    .setContentText(...)
                    .setContentIntent(...)   ← al pulsarla
                    .build()
                  NotificationManagerCompat.from(ctx).notify(ID, notificacion)
```

### Por qué no aparece mi notificación

| Causa | Cómo comprobarlo |
|---|---|
| No se ha creado el canal, o el id del `Builder` no coincide con el del canal. | Revisa que se use la misma constante en los dos sitios. |
| No se ha concedido el permiso (Android 13+). | *Ajustes → Apps → tu app → Notificaciones*. |
| El usuario ha desactivado el canal. | En la misma pantalla de Ajustes, dentro del canal. |
| Falta `setSmallIcon()`. | La app falla con una excepción: mira el Logcat. |
| El modo *No molestar* está activado en el emulador. | Desliza el panel de ajustes rápidos. |

---

## 7. Practica

1. En el proyecto `AvisosDemo`, crea un icono `ic_notificacion` con *Vector Asset*.
2. Crea un canal "Avisos generales" con importancia `IMPORTANCE_DEFAULT` y comprueba en *Ajustes → Apps → AvisosDemo → Notificaciones* que aparece.
3. Declara el permiso `POST_NOTIFICATIONS` y pídelo al pulsar un botón "Activar avisos". Prueba a concederlo y a denegarlo en un emulador con Android 13 o superior.
4. Añade un botón que muestre una notificación con título y texto. Pulsa el botón varias veces: ¿aparecen varias notificaciones o una sola? ¿Por qué?
5. Haz que la notificación abra la app al pulsarla y que desaparezca después.
6. Crea un segundo canal "Urgentes" con `IMPORTANCE_HIGH` y una notificación en ese canal. ¿Qué diferencia observas?
7. Intenta cambiar la importancia del primer canal en el código y vuelve a ejecutar la app. ¿Cambia algo? ¿Por qué?
8. **Reto:** en la app *Pizzería DAM*, al pulsar *Calcular precio* con un pedido válido, muestra una notificación "Pedido recibido" con el resumen en un `BigTextStyle`. Añade un botón "Simular envío" que actualice **la misma** notificación a "Tu pedido está en camino".

---

## 8. Autoevaluación

1. ¿Qué es una notificación? ¿En qué se diferencia de un Toast o una Snackbar?
2. ¿Qué es un canal de notificaciones y desde qué versión de Android es obligatorio?
3. Indica los cuatro niveles de importancia de un canal y qué comportamiento tiene cada uno.
4. ¿Qué ocurre si se crea un canal que ya existe? ¿Puede la app cambiar después su importancia?
5. ¿Por qué hace falta un `if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O)` al crear el canal?
6. ¿Qué dos pasos hay que dar para poder notificar en Android 13 o superior?
7. ¿Qué ocurre si el usuario rechaza el permiso dos veces?
8. ¿Qué elemento de una notificación es obligatorio? ¿Qué requisito tiene?
9. ¿Qué es un `PendingIntent` y para qué se usa en una notificación?
10. ¿Cómo se actualiza una notificación ya mostrada sin crear otra nueva?

---

## 📚 Para saber más (documentación oficial)

- [Creación de notificaciones](https://developer.android.com/develop/ui/views/notifications/build-notification?hl=es-419) · Guía completa: contenido, acciones, actualizar y eliminar.
- [Cómo crear y administrar canales de notificaciones](https://developer.android.com/develop/ui/views/notifications/channels?hl=es-419) · Canales, importancia y ajustes del usuario.
- [Permiso de tiempo de ejecución para notificaciones](https://developer.android.com/develop/ui/views/notifications/notification-permission?hl=es-419) · Cómo y cuándo pedir `POST_NOTIFICATIONS`.
- [Descripción general de las notificaciones](https://developer.android.com/develop/ui/views/notifications?hl=es-419) · Partes de una notificación y dónde aparece.

---

⬅️ [3.2 Snackbars](03-2-snackbars.md) · 🏠 [Índice](../README.md) · ➡️ [3.4 Diálogos](03-4-dialogos.md)
