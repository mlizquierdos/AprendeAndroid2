# 4.4 Mejorando el comportamiento de las Intents

> **Unidad 2 · Aplicaciones Android. Interfaz de usuario**
> Módulo: Programación multimedia y dispositivos móviles (2º DAM)

## Objetivos

Al terminar este apartado deberías ser capaz de:

- Explicar cómo se apilan las actividades en la pila de actividades (*back stack*).
- Evitar pantallas duplicadas con *flags* de Intent y con `launchMode`.
- Abrir una actividad y recibir un resultado con la Activity Result API.
- Usar Intents implícitos para llamar, abrir una web o compartir.
- Crear menús contextuales flotantes y menús emergentes (*popup*).

---

## 1. El problema: la pila de actividades

### 1.1 Cómo se apilan las actividades

Android guarda las actividades abiertas en una **pila** (*back stack*): cada `startActivity()` pone una actividad **encima**, y el botón *Atrás* quita la de arriba y muestra la de debajo.

```
Main → Carta → Carrito                *Atrás*            *Atrás*
┌─────────┐                        ┌─────────┐        ┌─────────┐
│ Carrito │ ← visible               │         │        │         │
├─────────┤                        ├─────────┤        │         │
│  Carta  │                        │  Carta  │ ←      │         │
├─────────┤                        ├─────────┤        ├─────────┤
│  Main   │                        │  Main   │        │  Main   │ ←
└─────────┘                        └─────────┘        └─────────┘
```

> 📝 **Recuerda (UT01 · ciclo de vida):** las actividades de debajo están **detenidas** (`onStop()`), no destruidas. La de arriba es la única en primer plano.

### 1.2 El problema con un menú común

En el punto 4.3, cada opción del menú hace `startActivity()`. Si el usuario navega solo con el menú, la pila **no deja de crecer**:

```
Main → Carta → Carrito → Carta → Main → Carta → …

┌─────────┐
│  Carta  │ ← para salir de la app hay que pulsar Atrás seis veces
├─────────┤
│  Main   │ ← ¡segunda MainActivity!
├─────────┤
│  Carta  │ ← ¡segunda CartaActivity!
├─────────┤
│ Carrito │
├─────────┤
│  Carta  │
├─────────┤
│  Main   │
└─────────┘
```

Consecuencias:

- El usuario pulsa *Atrás* y vuelve a pantallas por las que ya pasó, en un orden confuso.
- Hay **varias copias** de la misma actividad, cada una con sus propios datos: el pedido de la primera `MainActivity` no es el de la segunda.
- Se consume más memoria.

---

## 2. *Flags* de Intent

Los ***flags*** son opciones que se añaden al Intent para indicar **cómo** debe colocarse la actividad en la pila. Se añaden con `addFlags()` o asignando la propiedad `flags`, y se combinan con `or`:

```kotlin
val intent = Intent(this, MainActivity::class.java).apply {
    addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP or Intent.FLAG_ACTIVITY_SINGLE_TOP)
}
startActivity(intent)
```

| Flag | Qué hace | Pila antes → después (abriendo B) |
|---|---|---|
| *(ninguno)* | Crea siempre una nueva instancia encima. | A-B-C → A-B-C-**B** |
| `FLAG_ACTIVITY_SINGLE_TOP` | Si la actividad **ya está arriba**, no crea otra. | A-B → A-B *(la misma)* |
| `FLAG_ACTIVITY_CLEAR_TOP` | Si ya está **en la pila**, cierra todas las que tiene encima. | A-B-C-D → A-**B** |
| `FLAG_ACTIVITY_REORDER_TO_FRONT` | Si ya está en la pila, la **mueve arriba** sin cerrar nada. | A-B-C → A-C-**B** |
| `FLAG_ACTIVITY_NEW_TASK` + `FLAG_ACTIVITY_CLEAR_TASK` | Vacía la pila y deja solo la nueva. | A-B-C → **B** |

> 📌 **Relación con el punto 3.3:** estos dos últimos *flags* son los que usamos en el `PendingIntent` de las notificaciones: al pulsarla, se abre la app con una pila limpia.

### 2.1 Aplicarlo al menú común

Para la *Pizzería DAM*, añadimos al menú común una opción *Inicio* (`action_inicio`) y aplicamos en `BaseActivity` esta combinación:

- **Ir a `MainActivity`** (la pantalla raíz): `CLEAR_TOP | SINGLE_TOP`. Se cierra todo lo que haya encima y se vuelve a la `MainActivity` **existente**, con el pedido intacto.
- **Ir a una pantalla secundaria**: `REORDER_TO_FRONT`. Si ya estaba abierta, se trae al frente en lugar de duplicarla.

```kotlin
open class BaseActivity : AppCompatActivity() {

    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        return when (item.itemId) {
            R.id.action_inicio -> { abrirInicio(); true }
            R.id.action_carta -> { abrir(CartaActivity::class.java); true }
            R.id.action_carrito -> { abrir(CarritoActivity::class.java); true }
            R.id.action_ajustes -> { abrir(AjustesActivity::class.java); true }
            // ... "Acerca de" y el resto, igual que en el punto 4.3
            else -> super.onOptionsItemSelected(item)
        }
    }

    /** Vuelve a la MainActivity existente cerrando todo lo que hay encima. */
    private fun abrirInicio() {
        startActivity(Intent(this, MainActivity::class.java).apply {
            addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP or Intent.FLAG_ACTIVITY_SINGLE_TOP)
        })
    }

    /** Abre una pantalla secundaria, o la trae al frente si ya estaba abierta. */
    private fun abrir(destino: Class<*>) {
        startActivity(Intent(this, destino).apply {
            addFlags(Intent.FLAG_ACTIVITY_REORDER_TO_FRONT)
        })
    }
}
```

Ahora, por mucho que se navegue con el menú, **cada pantalla existe una sola vez** en la pila.

### 2.2 `launchMode` en el Manifest

Algunos de estos comportamientos también pueden fijarse en la propia actividad, con el atributo `android:launchMode` del Manifest. Así se aplican **siempre**, venga de donde venga el Intent:

```xml
<activity
    android:name=".MainActivity"
    android:launchMode="singleTop"
    android:exported="true"> ...
```

| `launchMode` | Comportamiento |
|---|---|
| `standard` *(por defecto)* | Siempre una nueva instancia. |
| `singleTop` | Si ya está arriba, se reutiliza (como `FLAG_ACTIVITY_SINGLE_TOP`). |
| `singleTask` / `singleInstance` | Solo puede existir una instancia, en su propia tarea. Para casos especiales; evítalos de momento. |

### 2.3 `onNewIntent()`: cuando se reutiliza una actividad

Si una actividad se **reutiliza** en lugar de crearse (por `SINGLE_TOP`, `CLEAR_TOP` + `SINGLE_TOP` o `singleTop`), **no** se llama a `onCreate()`, porque ya existe. En su lugar, Android llama a **`onNewIntent()`** con el nuevo Intent:

```kotlin
override fun onNewIntent(intent: Intent) {
    super.onNewIntent(intent)
    setIntent(intent)                          // para que la propiedad "intent" sea la nueva
    val mensaje = intent.getStringExtra(EXTRA_MENSAJE)
    mensaje?.let { Snackbar.make(binding.root, it, Snackbar.LENGTH_SHORT).show() }
}
```

> ⚠️ Si la actividad lee sus *extras* solo en `onCreate()`, al reutilizarla **no verá los nuevos datos**. Léelos también en `onNewIntent()`.

---

## 3. Abrir una actividad y recibir un resultado

A veces se abre una pantalla **para obtener algo de ella**: elegir una pizza de la carta, editar un dato… Al cerrarse, la actividad de origen debe recibir la respuesta.

> ⚠️ En tutoriales antiguos verás `startActivityForResult()` y `onActivityResult()`. Están **obsoletos**. Se usa la **Activity Result API**, la misma que empleamos en el punto 3.3 para pedir el permiso de notificaciones.

### 3.1 En la actividad que abre y espera el resultado

```kotlin
class MainActivity : BaseActivity() {

    // 1. Se registra como propiedad: qué hacer cuando llegue el resultado
    private val elegirPizza = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) { resultado ->
        if (resultado.resultCode == RESULT_OK) {
            val pizza = resultado.data?.getStringExtra(CartaActivity.EXTRA_PIZZA)
            binding.tvPizzaElegida.text = getString(R.string.pizza_elegida, pizza)
        }
        // RESULT_CANCELED: el usuario volvió atrás sin elegir; no se hace nada
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        // ...
        // 2. Se lanza cuando haga falta
        binding.btnElegirPizza.setOnClickListener {
            elegirPizza.launch(Intent(this, CartaActivity::class.java))
        }
    }
}
```

### 3.2 En la actividad que devuelve el resultado

```kotlin
class CartaActivity : BaseActivity() {

    companion object {
        const val EXTRA_PIZZA = "extra_pizza"
    }

    private fun devolverPizza(nombre: String) {
        val respuesta = Intent().putExtra(EXTRA_PIZZA, nombre)
        setResult(RESULT_OK, respuesta)   // 3. Prepara el resultado
        finish()                           // 4. Se cierra: el resultado llega a MainActivity
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        // ...
        binding.btnMargarita.setOnClickListener { devolverPizza("Margarita") }
        binding.btnBarbacoa.setOnClickListener { devolverPizza("Barbacoa") }
    }
}
```

| Paso | Dónde | Qué hace |
|---|---|---|
| 1 | Origen | `registerForActivityResult(StartActivityForResult()) { … }` como propiedad. |
| 2 | Origen | `launcher.launch(intent)` abre la actividad. |
| 3 | Destino | `setResult(RESULT_OK, intentConDatos)` prepara la respuesta. |
| 4 | Destino | `finish()` cierra la actividad; se ejecuta la lambda del paso 1. |

> 📝 Si el usuario pulsa *Atrás* sin elegir, el resultado llega con `RESULT_CANCELED` y `data` es `null`. Por eso se comprueba siempre `resultCode`.

---

## 4. Intents implícitos

Un Intent **implícito** no dice qué clase abrir, sino **qué acción** se quiere hacer. Android busca las apps capaces de hacerla y, si hay varias, deja elegir al usuario. Así nuestra app puede **llamar por teléfono, abrir una web o compartir** sin programar nada de eso.

```kotlin
// Abrir el marcador con el teléfono de la pizzería (no llama: el usuario confirma)
val llamar = Intent(Intent.ACTION_DIAL, Uri.parse("tel:600123456"))

// Abrir una página web en el navegador
val web = Intent(Intent.ACTION_VIEW, Uri.parse("https://developer.android.com"))

// Abrir una ubicación en la app de mapas
val mapa = Intent(Intent.ACTION_VIEW, Uri.parse("geo:0,0?q=Pizzería+DAM+Madrid"))

// Compartir un texto (WhatsApp, correo, Telegram…)
val compartir = Intent(Intent.ACTION_SEND).apply {
    type = "text/plain"
    putExtra(Intent.EXTRA_TEXT, getString(R.string.texto_compartir, total))
}
```

| Acción | Para qué | Dato |
|---|---|---|
| `ACTION_DIAL` | Abrir el marcador con un número | `tel:…` |
| `ACTION_VIEW` | Ver algo: una web, un mapa… | `https://…`, `geo:…` |
| `ACTION_SEND` | Compartir contenido | `type` + `EXTRA_TEXT` |
| `ACTION_SENDTO` | Escribir un correo | `mailto:…` |

### 4.1 Lanzarlos de forma segura

Puede que el dispositivo **no tenga ninguna app** capaz de hacer la acción (un emulador sin navegador, una tablet sin teléfono). En ese caso `startActivity()` lanza una `ActivityNotFoundException` y la app falla. Por eso se protege con `try`/`catch`:

```kotlin
private fun lanzarSeguro(intent: Intent) {
    try {
        startActivity(intent)
    } catch (e: ActivityNotFoundException) {
        Snackbar.make(binding.root, R.string.sin_app_para_accion, Snackbar.LENGTH_LONG).show()
    }
}
```

> 📝 En tutoriales antiguos verás la comprobación `if (intent.resolveActivity(packageManager) != null)`. Desde Android 11, por la **visibilidad de paquetes**, puede devolver `null` aunque sí haya apps instaladas, salvo que se declaren en el Manifest con `<queries>`. El `try`/`catch` es más sencillo y fiable.

### 4.2 Forzar el selector de apps

Para compartir, conviene mostrar **siempre** la lista de apps, aunque el usuario tenga una marcada por defecto:

```kotlin
lanzarSeguro(Intent.createChooser(compartir, getString(R.string.compartir_con)))
```

> 📌 **Relación con el menú:** estos Intents encajan muy bien como opciones del menú de opciones (*Llamar*, *Compartir pedido*) o del menú contextual de la sección siguiente.

---

## 5. Menús contextuales

El menú de opciones afecta a **toda la pantalla**. Un **menú contextual**, en cambio, ofrece acciones sobre **un elemento concreto**: una foto, un texto, una fila de una lista. Android ofrece dos formas:

| Tipo | Cómo se ve | Cómo se abre |
|---|---|---|
| **Menú contextual flotante** | Una lista flotante, como un diálogo, sobre la vista. | **Pulsación larga** sobre la vista. |
| **Modo de acción contextual** | Una barra que sustituye a la barra de la app, con acciones sobre los elementos seleccionados. | Normalmente, pulsación larga para empezar a seleccionar. |

Y, relacionado con ellos, el **menú emergente** (*popup*): una lista anclada a una vista que se abre con un **clic normal**.

### 5.1 Menú contextual flotante

Se necesitan cuatro pasos. Usaremos como ejemplo la imagen de una pizza en la carta.

**1. Definir el menú en XML** (`res/menu/menu_contextual_pizza.xml`):

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/ctx_anadir" android:title="@string/ctx_anadir" />
    <item android:id="@+id/ctx_ingredientes" android:title="@string/ctx_ingredientes" />
    <item android:id="@+id/ctx_compartir" android:title="@string/ctx_compartir" />
</menu>
```

**2. Registrar la vista** que tendrá el menú, en `onCreate()`:

```kotlin
registerForContextMenu(binding.ivMargarita)
```

**3. Crear el menú** sobrescribiendo `onCreateContextMenu()`. Android lo llama al hacer una pulsación larga sobre una vista registrada:

```kotlin
override fun onCreateContextMenu(
    menu: ContextMenu, v: View, menuInfo: ContextMenu.ContextMenuInfo?
) {
    super.onCreateContextMenu(menu, v, menuInfo)
    menuInflater.inflate(R.menu.menu_contextual_pizza, menu)
    menu.setHeaderTitle(R.string.pizza_margarita)     // título opcional
}
```

**4. Responder a la opción elegida** sobrescribiendo `onContextItemSelected()`:

```kotlin
override fun onContextItemSelected(item: MenuItem): Boolean {
    return when (item.itemId) {
        R.id.ctx_anadir -> { anadirAlCarrito("Margarita"); true }
        R.id.ctx_ingredientes -> { mostrarIngredientes("Margarita"); true }
        R.id.ctx_compartir -> {
            lanzarSeguro(Intent.createChooser(
                Intent(Intent.ACTION_SEND).apply {
                    type = "text/plain"
                    putExtra(Intent.EXTRA_TEXT, getString(R.string.mira_esta_pizza, "Margarita"))
                },
                getString(R.string.compartir_con)
            ))
            true
        }
        else -> super.onContextItemSelected(item)
    }
}
```

| Paso | Método | Equivalente en el menú de opciones |
|---|---|---|
| Registrar la vista | `registerForContextMenu(vista)` | — (no hace falta) |
| Crear el menú | `onCreateContextMenu()` | `onCreateOptionsMenu()` |
| Responder | `onContextItemSelected()` | `onOptionsItemSelected()` |

> 💡 **Varias vistas con el mismo menú:** si registras varias imágenes, el parámetro `v` de `onCreateContextMenu()` indica cuál se pulsó. Guárdalo en una propiedad (`pizzaSeleccionada = v.id`) para saber en `onContextItemSelected()` sobre qué pizza actuar.

> 📝 **En listas:** con un `ListView`, se registra la lista entera y la fila pulsada se obtiene de `item.menuInfo as AdapterView.AdapterContextMenuInfo`, como en la presentación de clase. Con `RecyclerView` (punto 5) se usa otro enfoque.

### 5.2 Menú emergente (*popup*)

Un `PopupMenu` muestra una lista **anclada a una vista** al pulsarla. Es útil para un botón "⋮" junto a un elemento, o para opciones como "Ordenar por…". No necesita registrar nada ni sobrescribir métodos:

```kotlin
binding.btnOrdenar.setOnClickListener { boton ->
    PopupMenu(this, boton).apply {
        menuInflater.inflate(R.menu.menu_ordenar, menu)
        setOnMenuItemClickListener { item ->
            when (item.itemId) {
                R.id.orden_nombre -> { ordenarPorNombre(); true }
                R.id.orden_precio -> { ordenarPorPrecio(); true }
                else -> false
            }
        }
        show()
    }
}
```

> ⚠️ Importa `androidx.appcompat.widget.PopupMenu`, para que tenga el mismo aspecto en todas las versiones.

### 5.3 Modo de acción contextual

El **modo de acción contextual** sustituye temporalmente la barra de la app por una barra con acciones sobre lo seleccionado, como cuando se seleccionan varios mensajes en una app de chat. Se inicia con `startSupportActionMode()` y un objeto `ActionMode.Callback` con cuatro métodos (crear, preparar, pulsar una acción y destruir).

La presentación de clase lo desarrolla con un `ListView`. Lo retomaremos con `RecyclerView` en el punto 5, donde la selección múltiple tiene más sentido.

### 5.4 ¿Qué menú uso?

| Situación | Menú |
|---|---|
| Acción que afecta a toda la pantalla o a la app (Ajustes, Buscar, Compartir pedido) | **Menú de opciones** |
| Acciones sobre **un** elemento, sin botones visibles | **Menú contextual flotante** (pulsación larga) |
| Acciones sobre **varios** elementos seleccionados | **Modo de acción contextual** |
| Más opciones de un elemento o de un botón, con un clic | **Menú emergente** |

> 💡 La pulsación larga no es evidente para el usuario: **no escondas ahí acciones importantes** que no estén disponibles de otra forma.

---

## 6. Practica

1. En `PizzeriaMenus`, navega solo con el menú: *Main → Carta → Carrito → Carta → Main*. Después cuenta cuántas veces tienes que pulsar *Atrás* para salir de la app.
2. Añade los *flags* de la sección 2.1 a `BaseActivity` y repite la prueba. ¿Cuántas veces hay que pulsar *Atrás* ahora?
3. Escribe en la `MainActivity` un dato en el formulario, navega a *Carta* y vuelve con la opción *Inicio* del menú. ¿Se conserva el dato? ¿Por qué?
4. Añade un `Log.d()` en `onCreate()` y en `onNewIntent()` de `MainActivity`. Comprueba en el Logcat cuál se llama al volver con *Inicio*.
5. Haz que `CartaActivity` devuelva la pizza elegida a `MainActivity` con la Activity Result API. Comprueba qué ocurre si vuelves con *Atrás* sin elegir.
6. Añade al menú común las opciones *Llamar a la pizzería* (`ACTION_DIAL`) y *Visitar web* (`ACTION_VIEW`), lanzadas con `lanzarSeguro()`.
7. Añade un botón *Compartir pedido* que comparta el resumen con `ACTION_SEND` y `createChooser()`.
8. En `CartaActivity`, añade un menú contextual flotante a la imagen de cada pizza con las opciones *Añadir al carrito*, *Ver ingredientes* y *Compartir*.
9. Añade un botón *Ordenar* con un `PopupMenu` que ordene la carta por nombre o por precio.
10. **Reto:** registra las tres imágenes de la carta con el **mismo** menú contextual y haz que cada opción actúe sobre la pizza pulsada.

---

## 7. Autoevaluación

1. ¿Qué es la pila de actividades? ¿Qué ocurre en ella al llamar a `startActivity()` y al pulsar *Atrás*?
2. ¿Qué problema aparece al navegar solo con un menú común sin usar *flags*?
3. Explica qué hacen `FLAG_ACTIVITY_SINGLE_TOP`, `FLAG_ACTIVITY_CLEAR_TOP` y `FLAG_ACTIVITY_REORDER_TO_FRONT`, con un ejemplo de pila de cada uno.
4. ¿Qué diferencia hay entre indicar el comportamiento con un *flag* y con `android:launchMode`?
5. ¿Cuándo se llama a `onNewIntent()` en lugar de a `onCreate()`? ¿Qué hay que hacer en él?
6. ¿Qué pasos hay que seguir para abrir una actividad y recibir un resultado con la Activity Result API?
7. ¿Qué es un Intent implícito? Pon tres ejemplos de acciones.
8. ¿Por qué hay que proteger con `try`/`catch` el lanzamiento de un Intent implícito?
9. ¿Qué cuatro pasos hacen falta para crear un menú contextual flotante?
10. ¿Qué diferencias hay entre un menú contextual flotante y un menú emergente? ¿Cuándo usarías cada uno?

---

## 📚 Para saber más (documentación oficial)

- [Tareas y la pila de actividades](https://developer.android.com/guide/components/activities/tasks-and-back-stack?hl=es-419) · *Flags* de Intent, `launchMode` y comportamiento del botón *Atrás*.
- [Cómo obtener el resultado de una actividad](https://developer.android.com/training/basics/intents/result?hl=es-419) · La Activity Result API.
- [Intents comunes](https://developer.android.com/guide/components/intents-common?hl=es-419) · Catálogo de Intents implícitos: llamar, mapas, correo, cámara…
- [Cómo agregar menús · Menús contextuales y emergentes](https://developer.android.com/guide/topics/ui/menus?hl=es-419#context-menu) · Menú contextual flotante, modo de acción contextual y `PopupMenu`.

---

⬅️ [4.3 Menús comunes a varias actividades](04-3-menus-comunes-intents.md) · 🏠 [Índice](../README.md) · ➡️ [5.1 Listas con RecyclerView](05-1-listas-recyclerview.md)
