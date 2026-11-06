# 4.3 Uso de menús comunes a varias actividades

> **Unidad 2 · Aplicaciones Android. Interfaz de usuario**
> Módulo: Programación multimedia y dispositivos móviles (2º DAM)

## Objetivos

Al terminar este apartado deberías ser capaz de:

- Añadir nuevas actividades a un proyecto y saber dónde quedan registradas.
- Compartir un mismo menú entre varias actividades mediante herencia, sin duplicar código.
- Navegar de una actividad a otra con un *Intent* explícito.
- Enviar datos a otra actividad con *extras* y recuperarlos.
- Mejorar el menú: ocultar la opción de la pantalla actual y añadir el botón *Arriba*.

---

## 1. Una app con varias pantallas

Hasta ahora, nuestras apps tenían una sola actividad. Pero lo normal es que una app tenga **varias pantallas**, y que el menú de opciones sirva para **navegar** entre ellas. Vamos a ampliar la *Pizzería DAM*:

```
                    ┌──────────────────┐
                    │   MainActivity   │   Hacer el pedido (formulario del punto 1.3)
                    └────────┬─────────┘
           ┌─────────────────┼──────────────────┐
           ▼                 ▼                  ▼
  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
  │  CartaActivity  │ │ CarritoActivity │ │ AjustesActivity │
  └─────────────────┘ └─────────────────┘ └─────────────────┘
     Ver la carta        Ver el pedido       Preferencias
```

Las cuatro pantallas tendrán **el mismo menú**, con las opciones *Carrito*, *Carta*, *Ajustes* y *Acerca de*.

### 1.1 Crear una nueva actividad

Clic derecho sobre el paquete de la app (`com.dam.pizzeria`) → **New → Activity → Empty Views Activity**:

- *Activity Name*: `CartaActivity`.
- Marca **Generate a Layout File** (`activity_carta.xml`).
- **No** marques *Launcher Activity*: solo la actividad principal se abre desde el icono.

Android Studio crea la clase, el layout y, además, **registra la actividad en el Manifest**:

```xml
<application ...>
    <activity android:name=".CartaActivity" android:exported="false" />

    <activity android:name=".MainActivity" android:exported="true">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
    </activity>
</application>
```

> ⚠️ **Toda actividad debe estar declarada en el `AndroidManifest.xml`.** Si creas la clase a mano (*New → Kotlin Class*) y no la declaras, al intentar abrirla la app falla con `ActivityNotFoundException`.

> 📝 **Recuerda (UT01 · estructura de la app):** el `<intent-filter>` con `MAIN` y `LAUNCHER` marca la actividad que se abre al pulsar el icono. `android:exported="false"` impide que otras apps abran esa actividad.

---

## 2. El problema: el mismo menú en cada actividad

Si cada actividad debe mostrar el menú, la solución inmediata sería copiar en las cuatro los métodos del punto 4.2:

```kotlin
// En MainActivity, en CartaActivity, en CarritoActivity, en AjustesActivity…
override fun onCreateOptionsMenu(menu: Menu): Boolean { ... }      // copiado 4 veces
override fun onOptionsItemSelected(item: MenuItem): Boolean { ... } // copiado 4 veces
```

Funciona, pero si mañana añadimos una opción al menú, hay que **cambiar cuatro ficheros**, y es fácil olvidar alguno.

> 📌 **Relación con POO (1.º DAM):** cuando varias clases comparten el mismo comportamiento, ese comportamiento se lleva a una **clase padre** y las demás **heredan** de ella. Es lo que vamos a hacer.

---

## 3. La solución: una actividad base

Creamos una clase `BaseActivity` que contiene **todo lo común al menú**. El resto de actividades heredan de ella en lugar de heredar directamente de `AppCompatActivity`:

```
          AppCompatActivity
                 ▲
                 │ hereda
           BaseActivity            ← onCreateOptionsMenu, onOptionsItemSelected
        ▲      ▲      ▲      ▲
        │      │      │      │
     Main   Carta  Carrito  Ajustes  ← solo su propio contenido
```

**`BaseActivity.kt`** (*New → Kotlin Class/File*, no *New → Activity*, porque no tiene layout ni va en el Manifest):

```kotlin
package com.dam.pizzeria

import android.content.Intent
import android.view.Menu
import android.view.MenuItem
import androidx.appcompat.app.AppCompatActivity
import com.google.android.material.appbar.MaterialToolbar
import com.google.android.material.dialog.MaterialAlertDialogBuilder

/**
 * Actividad base de la app: todas las pantallas que heredan de ella
 * comparten la barra de la app y el menú de opciones.
 * Es "open" porque en Kotlin las clases son finales por defecto.
 */
open class BaseActivity : AppCompatActivity() {

    /** Registra la toolbar del layout como barra de la app. */
    protected fun configurarToolbar(toolbar: MaterialToolbar) {
        setSupportActionBar(toolbar)
    }

    override fun onCreateOptionsMenu(menu: Menu): Boolean {
        menuInflater.inflate(R.menu.menu_principal, menu)
        return true
    }

    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        return when (item.itemId) {
            R.id.action_carta -> { abrir(CartaActivity::class.java); true }
            R.id.action_carrito -> { abrir(CarritoActivity::class.java); true }
            R.id.action_ajustes -> { abrir(AjustesActivity::class.java); true }
            R.id.action_acerca_de -> { mostrarAcercaDe(); true }
            else -> super.onOptionsItemSelected(item)
        }
    }

    /** Abre la actividad indicada (sección 4). */
    private fun abrir(destino: Class<*>) {
        startActivity(Intent(this, destino))
    }

    private fun mostrarAcercaDe() {
        MaterialAlertDialogBuilder(this)
            .setTitle(R.string.menu_acerca_de)
            .setMessage(R.string.acerca_de_texto)
            .setPositiveButton(R.string.aceptar, null)
            .show()
    }
}
```

**Cada actividad** solo tiene que heredar de `BaseActivity` y configurar su toolbar:

```kotlin
class CartaActivity : BaseActivity() {

    private lateinit var binding: ActivityCartaBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        binding = ActivityCartaBinding.inflate(layoutInflater)
        setContentView(binding.root)
        // ... insets ...

        configurarToolbar(binding.toolbar)
        // El menú y sus clics ya los gestiona BaseActivity
    }
}
```

| Palabra clave | Por qué |
|---|---|
| `open class BaseActivity` | En Kotlin las clases son **finales** por defecto: sin `open`, no se puede heredar de ella. |
| `protected fun configurarToolbar` | Solo la pueden usar `BaseActivity` y sus subclases. |
| `private fun abrir` | Detalle interno: las subclases no lo necesitan. |

> ⚠️ Cada layout debe tener su propia `MaterialToolbar` con `android:id="@+id/toolbar"`. Un truco para no repetirla: crea `toolbar.xml` con la `MaterialToolbar` e inclúyela en cada layout con `<include android:id="@+id/toolbar" layout="@layout/toolbar" />` (puntos 1.2 y 2.3). En ese caso, `binding.toolbar` es de tipo `ToolbarBinding`, y la toolbar se obtiene con su raíz: `configurarToolbar(binding.toolbar.root)`.

### 3.1 Añadir opciones propias en una actividad

Si una pantalla necesita, además, alguna opción propia, sobrescribe los métodos y **llama a `super`** para conservar las comunes:

```kotlin
class CarritoActivity : BaseActivity() {

    override fun onCreateOptionsMenu(menu: Menu): Boolean {
        super.onCreateOptionsMenu(menu)                        // opciones comunes
        menuInflater.inflate(R.menu.menu_carrito, menu)        // + "Vaciar carrito"
        return true
    }

    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        if (item.itemId == R.id.action_vaciar) {
            vaciarCarrito()
            return true
        }
        return super.onOptionsItemSelected(item)               // el resto, a BaseActivity
    }
}
```

---

## 4. Intents y mejoras en el menú

### 4.1 Qué es un Intent

Un **Intent** es un **mensaje** que una app envía al sistema para pedir que se haga algo; lo más habitual, **abrir una actividad**. Hay dos tipos:

| Tipo | Qué indica | Ejemplo |
|---|---|---|
| **Explícito** | La **clase exacta** que se quiere abrir. | Abrir `CartaActivity` de nuestra app. |
| **Implícito** | Una **acción** genérica; el sistema busca qué app puede hacerla. | "Abrir esta web", "llamar a este número" (punto 4.4). |

Para navegar entre las pantallas de nuestra app usamos **Intents explícitos**:

```kotlin
val intent = Intent(this, CartaActivity::class.java)
startActivity(intent)
```

| Elemento | Qué es |
|---|---|
| `this` | El contexto de origen: la actividad actual. |
| `CartaActivity::class.java` | La clase de la actividad de destino. |
| `startActivity(intent)` | Pide al sistema que abra esa actividad. |

> 📝 **Recuerda (UT01 · ciclo de vida):** al abrir `CartaActivity`, `MainActivity` no se destruye: pasa por `onPause()` y `onStop()` y queda **debajo**, en la pila de actividades. Al pulsar *Atrás*, `CartaActivity` se destruye y `MainActivity` vuelve con `onRestart()`, `onStart()` y `onResume()`.

### 4.2 Enviar datos a otra actividad: *extras*

Un Intent puede llevar datos adjuntos, los **extras**, como pares clave–valor:

```kotlin
// En MainActivity: al pulsar "Ver carrito", se envía el resumen del pedido
val intent = Intent(this, CarritoActivity::class.java).apply {
    putExtra(CarritoActivity.EXTRA_CLIENTE, nombre)
    putExtra(CarritoActivity.EXTRA_CANTIDAD, cantidad)
    putExtra(CarritoActivity.EXTRA_TOTAL, total)
}
startActivity(intent)
```

```kotlin
// En CarritoActivity: se recuperan en onCreate
class CarritoActivity : BaseActivity() {

    companion object {
        // Las claves se definen como constantes en la actividad que las recibe
        const val EXTRA_CLIENTE = "extra_cliente"
        const val EXTRA_CANTIDAD = "extra_cantidad"
        const val EXTRA_TOTAL = "extra_total"
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        // ... binding, setContentView, configurarToolbar ...

        val cliente = intent.getStringExtra(EXTRA_CLIENTE) ?: getString(R.string.cliente_anonimo)
        val cantidad = intent.getIntExtra(EXTRA_CANTIDAD, 0)       // 0: valor por defecto
        val total = intent.getDoubleExtra(EXTRA_TOTAL, 0.0)

        binding.tvResumen.text = getString(R.string.resumen_carrito, cliente, cantidad, total)
    }
}
```

| Para enviar | Para recibir |
|---|---|
| `putExtra(clave, valor)` | `intent.getStringExtra(clave)` → `String?` |
| | `intent.getIntExtra(clave, porDefecto)` → `Int` |
| | `intent.getDoubleExtra(clave, porDefecto)` → `Double` |
| | `intent.getBooleanExtra(clave, porDefecto)` → `Boolean` |

- Dentro de la actividad, la propiedad **`intent`** es el Intent que la abrió.
- `getStringExtra()` puede devolver `null` (si nadie envió esa clave), por eso se combina con `?:`.
- Los tipos numéricos y booleanos necesitan un **valor por defecto**.
- Las claves van como **constantes**, para que emisor y receptor no se equivoquen al escribirlas.

> 📌 Los extras viajan en un **`Bundle`**, el mismo tipo de objeto que en `onSaveInstanceState()` (UT01) y en los argumentos de un `DialogFragment` (punto 3.4).

### 4.3 Mejora 1: no abrir la pantalla en la que ya estamos

Con el menú común aparece un problema: estando en *Carta*, el menú también ofrece *Carta*. Si el usuario la pulsa, se abre **otra** `CartaActivity` encima de la actual.

La solución: cada actividad indica qué opción del menú le corresponde, y `BaseActivity` la **oculta** en `onPrepareOptionsMenu()` (punto 4.2):

```kotlin
open class BaseActivity : AppCompatActivity() {

    /** Id de la opción del menú que corresponde a esta pantalla (null si ninguna). */
    protected open val opcionMenuActual: Int? = null

    override fun onPrepareOptionsMenu(menu: Menu): Boolean {
        opcionMenuActual?.let { menu.findItem(it)?.isVisible = false }
        return super.onPrepareOptionsMenu(menu)
    }

    // ... resto igual ...
}
```

```kotlin
class CartaActivity : BaseActivity() {
    override val opcionMenuActual = R.id.action_carta
    // ...
}
```

> 📌 **Relación con POO:** `protected open val` es una propiedad que las subclases pueden **sobrescribir** (`override val`). Es una forma sencilla de que cada hija "configure" el comportamiento del padre.

### 4.4 Mejora 2: el botón *Arriba*

Las pantallas secundarias deben ofrecer una flecha **←** en la barra (*botón Arriba*) para volver a la pantalla principal. Se necesitan dos pasos:

**1. Indicar en el Manifest cuál es la actividad padre:**

```xml
<activity
    android:name=".CartaActivity"
    android:exported="false"
    android:parentActivityName=".MainActivity" />
```

**2. Mostrar la flecha en la barra** (en `BaseActivity`, solo si la actividad tiene padre):

```kotlin
protected fun configurarToolbar(toolbar: MaterialToolbar, mostrarArriba: Boolean = true) {
    setSupportActionBar(toolbar)
    supportActionBar?.setDisplayHomeAsUpEnabled(mostrarArriba)
}
```

```kotlin
// En MainActivity, que no tiene padre:
configurarToolbar(binding.toolbar, mostrarArriba = false)
```

Al pulsar la flecha, Android llama a `onOptionsItemSelected()` con el id **`android.R.id.home`**. Como nuestra rama `else` llama a `super`, `AppCompatActivity` lo gestiona y **vuelve a la actividad padre** indicada en el Manifest. Por eso era importante esa rama `else`.

| | Botón *Atrás* (sistema) | Botón *Arriba* (←, barra) |
|---|---|---|
| Dónde está | Gesto o botón del sistema | En la barra de la app |
| A dónde lleva | A la pantalla **anterior** en el tiempo | A la pantalla **padre** en la jerarquía de la app |
| Se configura | Funciona solo | `parentActivityName` + `setDisplayHomeAsUpEnabled(true)` |

### 4.5 Mejora 3: cerrar la actividad desde el código

Para volver a la pantalla anterior desde un botón propio ("Volver", "Cancelar"), basta con **cerrar** la actividad actual:

```kotlin
binding.btnVolver.setOnClickListener { finish() }
```

> ⚠️ **No vuelvas atrás abriendo otra vez la actividad anterior** con `startActivity(Intent(this, MainActivity::class.java))`: crearías una **nueva** `MainActivity` encima de la pila, en lugar de volver a la que ya existía. Lo veremos en el punto 4.4.

---

## 5. Practica

1. En el proyecto `PizzeriaMenus`, crea tres actividades con *New → Activity → Empty Views Activity*: `CartaActivity`, `CarritoActivity` y `AjustesActivity`. Comprueba cómo han quedado declaradas en el Manifest.
2. Crea `toolbar.xml` con una `MaterialToolbar` e inclúyela en los cuatro layouts.
3. Crea `BaseActivity` con el menú común y haz que las cuatro actividades hereden de ella. Comprueba que el menú aparece en todas.
4. Añade una opción al menú común (por ejemplo, *Ofertas*). ¿Cuántos ficheros has tenido que cambiar?
5. Navega con el menú: *Main → Carta → Carrito → Ajustes*. Después pulsa *Atrás* varias veces y anota el orden en que aparecen las pantallas.
6. Implementa la mejora de la sección 4.3 y comprueba que en cada pantalla desaparece su propia opción.
7. Añade el botón *Arriba* a las tres pantallas secundarias. Compara qué ocurre al pulsar *Arriba* y al pulsar *Atrás* después de navegar *Main → Carta → Carrito*.
8. Haz que, al pulsar *Calcular precio* en `MainActivity`, se abra `CarritoActivity` con el nombre, la cantidad y el total como *extras*, y que esta muestre el resumen.
9. **Reto:** añade a `CarritoActivity` una opción de menú propia "Vaciar carrito" que pida confirmación con un `DialogFragment` (punto 3.4), sin perder las opciones comunes.

---

## 6. Autoevaluación

1. ¿Qué crea Android Studio al añadir una actividad con *New → Activity*? ¿Qué ocurre si una actividad no está en el Manifest?
2. ¿Qué problema aparece al copiar el mismo menú en varias actividades?
3. ¿Cómo se comparte un menú entre varias actividades? ¿Por qué `BaseActivity` debe ser `open`?
4. ¿Por qué `BaseActivity` no se crea con *New → Activity* ni se declara en el Manifest?
5. Si una actividad quiere añadir opciones propias al menú común, ¿qué debe hacer en `onCreateOptionsMenu()` y en `onOptionsItemSelected()`?
6. ¿Qué diferencia hay entre un Intent explícito y uno implícito?
7. ¿Cómo se envía un dato a otra actividad? ¿Cómo se recupera? ¿Por qué los números necesitan un valor por defecto?
8. ¿Por qué conviene definir las claves de los *extras* como constantes?
9. ¿Qué dos pasos hacen falta para mostrar el botón *Arriba*? ¿En qué se diferencia del botón *Atrás*?
10. ¿Por qué no se debe volver a la pantalla anterior abriéndola con `startActivity()`? ¿Qué se usa en su lugar?

---

## 📚 Para saber más (documentación oficial)

- [Intents y filtros de intents](https://developer.android.com/guide/components/intents-filters?hl=es-419) · Intents explícitos e implícitos, *extras* y cómo se resuelven.
- [Introducción a las actividades](https://developer.android.com/guide/components/activities/intro-activities?hl=es-419) · Cómo se declaran en el Manifest y cómo se abren unas desde otras.
- [Cómo configurar la barra de la app](https://developer.android.com/training/appbar/setting-up?hl=es-419) · Toolbar y `setSupportActionBar()`.
- [Herencia (Kotlin)](https://kotlinlang.org/docs/inheritance.html) (en inglés) · Clases `open`, `override` y `super`.

---

⬅️ [4.2 Añadiendo un menú a una actividad](04-2-menu-en-actividad.md) · 🏠 [Índice](../README.md) · ➡️ [4.4 Mejorando el comportamiento de las Intents](04-4-intents-y-menus-contextuales.md)
