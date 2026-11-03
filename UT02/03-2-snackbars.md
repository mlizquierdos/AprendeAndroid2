# 3.2 Snackbars

> **Unidad 2 · Aplicaciones Android. Interfaz de usuario**
> Módulo: Programación multimedia y dispositivos móviles (2º DAM)

## Objetivos

Al terminar este apartado deberías ser capaz de:

- Explicar qué es una Snackbar y en qué se diferencia de un Toast.
- Crear y mostrar una Snackbar con `Snackbar.make()`.
- Añadir una acción, como "Deshacer", y responder a ella.
- Saber qué vista pasar a `make()` y el papel de `CoordinatorLayout`.
- Detectar cuándo y por qué se cierra una Snackbar.

---

## 1. Qué es una Snackbar

Una **Snackbar** es una barra que aparece en la parte inferior de la pantalla de **nuestra** app con un mensaje breve y, opcionalmente, **un botón de acción**. Forma parte de la librería **Material Components**, incluida por defecto en los proyectos nuevos.

```
┌───────────────────────────────┐
│                               │
│      (pantalla de la app)     │
│                               │
│ ┌───────────────────────────┐ │
│ │ Pizza eliminada  DESHACER │ │  ← Snackbar con acción
│ └───────────────────────────┘ │
└───────────────────────────────┘
```

### Toast frente a Snackbar

| | Toast | Snackbar |
|---|---|---|
| Librería | Android (`android.widget`) | Material Components |
| Dónde aparece | Flotando sobre cualquier app | Dentro de la pantalla de **nuestra** app |
| Acción | ❌ | ✅ Un botón |
| Se puede descartar deslizando | ❌ | ✅ (dentro de un `CoordinatorLayout`) |
| Duración | Corta o larga | Corta, larga o **indefinida** |
| Se muestra con la app en segundo plano | Sí (solo texto) | No |

> ✅ **Recomendación de Material Design:** para confirmaciones dentro de la app, usa preferentemente una Snackbar. Es más visible, encaja mejor con el diseño y permite deshacer.

---

## 2. Crear y mostrar una Snackbar

```kotlin
import com.google.android.material.snackbar.Snackbar

Snackbar.make(binding.root, "Pedido guardado", Snackbar.LENGTH_SHORT).show()
```

Se parece mucho a un Toast, con una diferencia importante en el primer parámetro:

| Parámetro | Qué es |
|---|---|
| `binding.root` | Una **vista** de la pantalla (no un contexto). La Snackbar busca a partir de ella dónde colocarse. |
| `"Pedido guardado"` | El texto, o un id de recurso (`R.string.pedido_guardado`). |
| `Snackbar.LENGTH_SHORT` | La duración. |
| `.show()` | La muestra. Igual que con el Toast, sin `show()` no aparece. |

### 2.1 Duración

| Constante | Comportamiento |
|---|---|
| `Snackbar.LENGTH_SHORT` | Unos 1,5 segundos. |
| `Snackbar.LENGTH_LONG` | Unos 2,75 segundos. |
| `Snackbar.LENGTH_INDEFINITE` | **No desaparece sola**: se queda hasta que el usuario pulse la acción, la descarte o se muestre otra. |

A diferencia del Toast, también admite una duración en milisegundos:

```kotlin
Snackbar.make(binding.root, R.string.pedido_guardado, 5000).show()   // 5 segundos
```

> ⚠️ Si usas `LENGTH_INDEFINITE`, añade siempre una acción. Si no, el usuario no tendrá forma evidente de cerrarla.

---

## 3. Añadir una acción

Lo que hace útil a la Snackbar es su botón de acción, que se añade con `setAction()`:

```kotlin
Snackbar.make(binding.root, R.string.pizza_eliminada, Snackbar.LENGTH_LONG)
    .setAction(R.string.deshacer) {
        // Se ejecuta si el usuario pulsa DESHACER
        restaurarPizza()
    }
    .show()
```

- El primer parámetro es el **texto del botón** (se muestra en mayúsculas).
- La lambda es lo que se ejecuta al pulsarlo.
- Al pulsar la acción, la Snackbar **se cierra sola**.
- Solo admite **una** acción. Si necesitas más opciones, usa un diálogo.

### 3.1 Ejemplo completo: borrar y deshacer

Un patrón muy habitual: se borra el elemento **en el momento** y la Snackbar permite recuperarlo.

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding
    private val pedidos = mutableListOf("Margarita", "Barbacoa", "Cuatro quesos")

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        mostrarPedidos()
        binding.btnBorrarUltimo.setOnClickListener { borrarUltimo() }
    }

    private fun borrarUltimo() {
        if (pedidos.isEmpty()) return

        // 1. Se borra y se guarda lo borrado por si hay que deshacer
        val borrado = pedidos.removeAt(pedidos.lastIndex)
        mostrarPedidos()

        // 2. Se ofrece deshacer
        Snackbar.make(binding.root, getString(R.string.pizza_eliminada, borrado), Snackbar.LENGTH_LONG)
            .setAction(R.string.deshacer) {
                pedidos.add(borrado)
                mostrarPedidos()
            }
            .show()
    }

    private fun mostrarPedidos() {
        binding.tvPedidos.text = pedidos.joinToString("\n")
    }
}
```

```xml
<string name="pizza_eliminada">Pizza %1$s eliminada</string>
<string name="deshacer">Deshacer</string>
```

---

## 4. Qué vista pasar a `make()`

Snackbar no necesita la vista exacta donde va a aparecer: sube por el árbol de vistas a partir de la que le pasas hasta encontrar un contenedor adecuado:

1. Si encuentra un **`CoordinatorLayout`**, se coloca dentro de él.
2. Si no, se coloca en la vista raíz de la ventana.

Por eso basta con pasar `binding.root` o cualquier vista de la pantalla, como el botón pulsado.

### 4.1 Con `CoordinatorLayout`: deslizar para descartar

Si el layout raíz es un `androidx.coordinatorlayout.widget.CoordinatorLayout`, la Snackbar gana dos comportamientos:

- El usuario puede **descartarla deslizándola** hacia un lado.
- Un `FloatingActionButton` dentro del mismo `CoordinatorLayout` **sube** automáticamente para no quedar tapado.

```xml
<androidx.coordinatorlayout.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!-- Contenido de la pantalla -->
    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <!-- ... -->
    </androidx.constraintlayout.widget.ConstraintLayout>

    <com.google.android.material.floatingactionbutton.FloatingActionButton
        android:id="@+id/fabAnadir"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|end"
        android:layout_margin="16dp"
        android:contentDescription="@string/anadir"
        app:srcCompat="@drawable/ic_add" />

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

> 📌 **Relación con el punto 1.2:** `CoordinatorLayout` se comporta como un `FrameLayout` (los hijos se superponen y se colocan con `layout_gravity`), pero además **coordina** a sus hijos: el FAB "sabe" que ha aparecido una Snackbar.

### 4.2 Anclar la Snackbar sobre otra vista

Si hay una barra o un botón en la parte inferior que no quieres tapar, puedes indicar que la Snackbar aparezca **encima** de esa vista:

```kotlin
Snackbar.make(binding.root, R.string.pedido_guardado, Snackbar.LENGTH_SHORT)
    .setAnchorView(binding.btnCalcular)
    .show()
```

---

## 5. Saber cómo se ha cerrado

A veces la acción definitiva (por ejemplo, borrar en una base de datos) solo debe ejecutarse si el usuario **no** ha pulsado "Deshacer". Para eso se añade un *callback* que informa del motivo del cierre:

```kotlin
Snackbar.make(binding.root, R.string.pizza_eliminada, Snackbar.LENGTH_LONG)
    .setAction(R.string.deshacer) { restaurarPizza() }
    .addCallback(object : Snackbar.Callback() {
        override fun onDismissed(transientBottomBar: Snackbar?, event: Int) {
            if (event != DISMISS_EVENT_ACTION) {
                // El usuario NO ha pulsado Deshacer: se confirma el borrado
                borrarDefinitivamente()
            }
        }
    })
    .show()
```

| Constante de `event` | Motivo del cierre |
|---|---|
| `DISMISS_EVENT_ACTION` | Se ha pulsado la acción. |
| `DISMISS_EVENT_TIMEOUT` | Ha pasado el tiempo. |
| `DISMISS_EVENT_SWIPE` | El usuario la ha deslizado. |
| `DISMISS_EVENT_CONSECUTIVE` | Se ha mostrado otra Snackbar que la sustituye. |
| `DISMISS_EVENT_MANUAL` | Se ha cerrado desde el código con `dismiss()`. |

> 📌 **Relación con POO:** `Snackbar.Callback` es una **clase abstracta**, no una interfaz. Por eso se escribe `object : Snackbar.Callback()` **con paréntesis**: estamos heredando de ella y llamando a su constructor.

---

## 6. Personalizar el aspecto

El aspecto de la Snackbar lo define el tema Material de la app, y es preferible no cambiarlo. Aun así, se pueden ajustar algunos detalles:

```kotlin
Snackbar.make(binding.root, R.string.error_conexion, Snackbar.LENGTH_INDEFINITE)
    .setAction(R.string.reintentar) { conectar() }
    .setActionTextColor(getColor(R.color.amarillo))
    .setBackgroundTint(getColor(R.color.rojo_error))
    .setTextMaxLines(3)
    .show()
```

> ⚠️ Usa los colores con moderación: si cada Snackbar tiene un color distinto, el usuario deja de percibirlos como un mismo tipo de aviso.

---

## 7. Practica

1. En el proyecto `AvisosDemo`, añade un botón que muestre una Snackbar corta con el texto "Snackbar de prueba".
2. Cambia la duración a `LENGTH_INDEFINITE` sin acción. ¿Cómo la cierras? Añade una acción "Cerrar" que no haga nada más.
3. Implementa el ejemplo "borrar y deshacer" de la sección 3.1 con una lista de tres pizzas que se muestran en un `TextView`.
4. Cambia el layout raíz a `CoordinatorLayout` y comprueba que ahora la Snackbar se puede deslizar para descartarla.
5. Añade un `FloatingActionButton` abajo a la derecha. Muestra una Snackbar y observa qué hace el botón. Repite la prueba con un `ConstraintLayout` como raíz.
6. Añade el *callback* `onDismissed` de la sección 5 y muestra en el Logcat con `Log.d()` el motivo del cierre. Prueba los cuatro casos: pulsar la acción, esperar, deslizar y mostrar otra Snackbar encima.
7. **Reto:** en la app *Pizzería DAM*, sustituye los Toasts de confirmación por Snackbars. Añade un botón "Vaciar pedido" que limpie el formulario y ofrezca "Deshacer" para recuperar los datos.

---

## 8. Autoevaluación

1. ¿Qué diferencias hay entre un Toast y una Snackbar?
2. ¿A qué librería pertenece la clase `Snackbar`?
3. ¿Qué se pasa como primer parámetro de `Snackbar.make()`? ¿Por qué no es un contexto?
4. ¿Qué duraciones admite una Snackbar? ¿Qué ocurre con `LENGTH_INDEFINITE`?
5. ¿Cómo se añade una acción? ¿Cuántas acciones admite como máximo?
6. ¿Qué ventajas aporta usar un `CoordinatorLayout` como raíz del layout?
7. ¿Para qué sirve `setAnchorView()`?
8. ¿Cómo sabes si el usuario ha pulsado la acción "Deshacer" o si la Snackbar se ha cerrado por tiempo?
9. ¿Por qué se escribe `object : Snackbar.Callback()` con paréntesis, y en cambio `object : View.OnClickListener` sin ellos?
10. ¿Se puede mostrar una Snackbar con la app en segundo plano? ¿Qué usarías en ese caso?

---

## 📚 Para saber más (documentación oficial)

- [Cómo crear y mostrar un mensaje emergente](https://developer.android.com/develop/ui/views/notifications/snackbar/showing?hl=es-419) · Snackbar y `CoordinatorLayout`.
- [Cómo agregar una acción a un mensaje](https://developer.android.com/develop/ui/views/notifications/snackbar/action?hl=es-419) · `setAction()` con ejemplos.
- [Snackbar · Material Components for Android](https://github.com/material-components/material-components-android/blob/master/docs/components/Snackbar.md) (en inglés) · Todas las opciones de configuración y estilo.

---

⬅️ [3.1 Toasts](03-1-toasts.md) · 🏠 [Índice](../README.md) · ➡️ [3.3 Notificaciones](03-3-notificaciones.md)
