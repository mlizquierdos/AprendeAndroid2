# 3.1 Toasts

> **Unidad 2 · Aplicaciones Android. Interfaz de usuario**
> Módulo: Programación multimedia y dispositivos móviles (2º DAM)

## Objetivos

Al terminar este apartado deberías ser capaz de:

- Distinguir los tres mecanismos para informar al usuario: Toast, Snackbar y notificación.
- Crear y mostrar un Toast con `Toast.makeText()`.
- Elegir la duración adecuada y usar textos de `strings.xml`.
- Conocer las limitaciones actuales de los Toasts y cuándo no conviene usarlos.

---

## 1. Informar al usuario: tres mecanismos

Una app necesita a menudo comunicar algo al usuario: que se ha guardado un dato, que ha fallado una conexión o que ha llegado un mensaje. Android ofrece tres mecanismos, y elegir bien depende de **dónde está el usuario** y de **si tiene que hacer algo**:

| Mecanismo | Dónde aparece | ¿Admite acción? | Cuándo usarlo |
|---|---|---|---|
| **Toast** | Mensaje flotante breve, abajo en la pantalla | ❌ | Confirmación sencilla mientras se usa la app: "Guardado". |
| **Snackbar** (3.2) | Barra en la parte inferior de **nuestra** pantalla | ✅ Una acción ("Deshacer") | Confirmación con posibilidad de reaccionar: "Elemento borrado · DESHACER". |
| **Notificación** (3.3) | Barra de estado y panel de notificaciones del **sistema** | ✅ Abrir la app, acciones | Avisar de algo aunque la app **no esté en primer plano**: "Nuevo mensaje". |
| **Diálogo** (3.4) | Ventana sobre la pantalla que **interrumpe** | ✅ Botones, listas, campos | Cuando el usuario **tiene que responder**: confirmar un borrado, elegir una opción. |

> 💡 **Regla práctica:** cuanto más interrumpe un mecanismo, menos se debe usar. Un diálogo para decir "Guardado" es excesivo; un Toast para preguntar "¿Seguro que quieres borrar?" es insuficiente.

---

## 2. Qué es un Toast

Un **Toast** es un mensaje breve que aparece flotando en la parte inferior de la pantalla, sobre cualquier contenido, y desaparece solo al cabo de unos segundos. El usuario no puede interactuar con él: ni pulsarlo ni descartarlo.

```
┌─────────────────────────┐
│                         │
│      (pantalla de       │
│       la aplicación)    │
│                         │
│   ┌─────────────────┐   │
│   │ Pedido guardado │   │   ← Toast: aparece y desaparece solo
│   └─────────────────┘   │
└─────────────────────────┘
```

Se llama así por la tostada que salta de la tostadora: aparece, se ve un momento y ya está.

---

## 3. Crear y mostrar un Toast

Se usa la clase `android.widget.Toast` y su método estático `makeText()`:

```kotlin
Toast.makeText(this, "Pedido guardado", Toast.LENGTH_SHORT).show()
```

| Parámetro | Qué es |
|---|---|
| `this` | El **contexto**. En una actividad, la propia actividad (`this`). |
| `"Pedido guardado"` | El texto. Mejor como recurso: `R.string.pedido_guardado`. |
| `Toast.LENGTH_SHORT` | La duración. |
| `.show()` | **Muestra** el Toast. |

> ⚠️ **El error más típico:** olvidar `.show()`. `makeText()` solo **crea** el Toast; sin `show()` no aparece nada, y no hay ningún error que lo indique.

### 3.1 Duración

Solo hay dos valores posibles:

| Constante | Duración aproximada |
|---|---|
| `Toast.LENGTH_SHORT` | 2 segundos |
| `Toast.LENGTH_LONG` | 3,5 segundos |

No se puede indicar una duración en milisegundos. Si un mensaje necesita quedarse más tiempo, un Toast no es la herramienta adecuada: usa una Snackbar o un diálogo.

### 3.2 Texto desde recursos

`makeText()` admite directamente un id de recurso de texto:

```kotlin
Toast.makeText(this, R.string.pedido_guardado, Toast.LENGTH_SHORT).show()
```

Y si el texto lleva parámetros, se construye antes con `getString()`:

```xml
<string name="pizzas_pedidas">Has pedido %1$d pizzas</string>
```

```kotlin
val texto = getString(R.string.pizzas_pedidas, cantidad)
Toast.makeText(this, texto, Toast.LENGTH_LONG).show()
```

### 3.3 Ejemplo con View Binding

```kotlin
binding.btnGuardar.setOnClickListener {
    if (binding.etNombre.text.isNullOrBlank()) {
        Toast.makeText(this, R.string.error_nombre_vacio, Toast.LENGTH_SHORT).show()
    } else {
        guardar()
        Toast.makeText(this, R.string.guardado_ok, Toast.LENGTH_SHORT).show()
    }
}
```

### 3.4 El contexto fuera de la actividad

Dentro de una lambda o de un objeto anónimo, `this` ya no es la actividad. En esos casos hay que indicarlo de forma explícita:

```kotlin
binding.sbCantidad.setOnSeekBarChangeListener(object : SeekBar.OnSeekBarChangeListener {
    override fun onStopTrackingTouch(seekBar: SeekBar?) {
        // Aquí "this" es el objeto anónimo, no la actividad
        Toast.makeText(this@MainActivity, "Cantidad elegida", Toast.LENGTH_SHORT).show()
    }
    override fun onProgressChanged(seekBar: SeekBar?, progress: Int, fromUser: Boolean) {}
    override fun onStartTrackingTouch(seekBar: SeekBar?) {}
})
```

> 📝 En una lambda de `setOnClickListener { }` **sí** se puede usar `this`, porque las lambdas no cambian el significado de `this`. Solo cambia dentro de un `object :`.

### 3.5 Una función de extensión para simplificar

Como `Toast.makeText(...).show()` se repite mucho, es habitual crear una **función de extensión** sobre `Context`:

```kotlin
// Utilidades.kt
fun Context.toast(mensaje: String, duracion: Int = Toast.LENGTH_SHORT) {
    Toast.makeText(this, mensaje, duracion).show()
}
```

```kotlin
// En cualquier actividad
toast("Pedido guardado")
toast(getString(R.string.pizzas_pedidas, 3), Toast.LENGTH_LONG)
```

> 📌 **Relación con Kotlin (UT01):** las funciones de extensión añaden métodos a una clase existente sin heredar de ella. Aquí añadimos `toast()` a todas las clases que son `Context`, como las actividades.

---

## 4. Limitaciones actuales

Los Toasts han ido perdiendo funciones en las versiones recientes de Android:

| Limitación | Desde |
|---|---|
| Los **Toasts personalizados** (con `setView()` y un layout propio) están obsoletos. Desde segundo plano no se muestran. | Android 11 (API 30) |
| `setGravity()` (cambiar la posición) **no tiene efecto** en los Toasts de texto. | Android 11 (API 30) |
| El texto se limita a **dos líneas** y se muestra junto al icono de la app. | Android 12 (API 31) |

En la práctica: **un Toast es solo un texto corto**. Para algo más elaborado se usa una Snackbar (3.2) o un diálogo (3.4).

### Cuándo **no** usar un Toast

- Para **errores importantes**: el usuario puede no verlo, porque desaparece solo.
- Para **información que hay que leer con calma**: dura como mucho 3,5 segundos.
- Cuando el usuario **tiene que poder deshacer** algo: usa una Snackbar con acción.
- Para avisar con la app **cerrada**: usa una notificación.
- Para **depurar**: los mensajes de depuración van al Logcat con `Log.d()`, no a un Toast.

---

## 5. Practica

1. Crea un proyecto `AvisosDemo` con View Binding activado y un botón `btnToastCorto` que muestre un Toast con `LENGTH_SHORT`.
2. Añade un botón `btnToastLargo` que muestre un Toast con `LENGTH_LONG` usando un texto de `strings.xml`. Compara las duraciones.
3. Añade un `EditText` y un botón que muestre en un Toast "Hola, *nombre*", con un texto con parámetro en `strings.xml`.
4. Quita el `.show()` de uno de los Toasts y ejecuta la app. ¿Qué ocurre? ¿Aparece algún error?
5. Pulsa cinco veces seguidas el botón del Toast largo. ¿Se muestran a la vez o uno detrás de otro? ¿Cuánto tarda en desaparecer el último?
6. Crea la función de extensión `Context.toast()` de la sección 3.5 en un fichero `Utilidades.kt` y úsala en todos los botones.
7. **Reto:** escribe un Toast de cuatro líneas de texto y pruébalo en un emulador con Android 11 y otro con Android 12 o superior. ¿Qué diferencia observas?

---

## 6. Autoevaluación

1. ¿Qué diferencia hay entre un Toast, una Snackbar y una notificación? ¿Cuándo usarías cada uno?
2. ¿Qué parámetros recibe `Toast.makeText()`?
3. ¿Qué ocurre si se olvida llamar a `show()`?
4. ¿Qué duraciones admite un Toast? ¿Se puede indicar una duración en milisegundos?
5. ¿Por qué dentro de un `object : SeekBar.OnSeekBarChangeListener` hay que escribir `this@MainActivity` en lugar de `this`?
6. ¿Puede el usuario pulsar o descartar un Toast?
7. ¿Qué limitaciones tienen los Toasts desde Android 11 y Android 12?
8. ¿Por qué no es buena idea usar un Toast para mostrar un error importante?
9. ¿Qué es una función de extensión? Escribe una que muestre un Toast largo.
10. ¿Debe usarse un Toast para mostrar mensajes de depuración? ¿Qué se usa en su lugar?

---

## 📚 Para saber más (documentación oficial)

- [Descripción general de los avisos (Toast)](https://developer.android.com/guide/topics/ui/notifiers/toasts?hl=es-419) · Uso de los Toasts y alternativas recomendadas.
- [Referencia de la clase Toast](https://developer.android.com/reference/android/widget/Toast) (en inglés) · Métodos, constantes y cambios por versión.
- [Funciones de extensión (Kotlin)](https://kotlinlang.org/docs/extensions.html) (en inglés) · Cómo añadir funciones a clases existentes.

---

⬅️ [2.3 Uso de View Binding](02-3-uso-view-binding.md) · 🏠 [Índice](../README.md) · ➡️ [3.2 Snackbars](03-2-snackbars.md)
