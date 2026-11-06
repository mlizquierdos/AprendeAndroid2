# 4.2 Añadiendo un menú a una actividad

> **Unidad 2 · Aplicaciones Android. Interfaz de usuario**
> Módulo: Programación multimedia y dispositivos móviles (2º DAM)

## Objetivos

Al terminar este apartado deberías ser capaz de:

- Preparar la barra de la app con una `MaterialToolbar` donde se mostrará el menú.
- Cargar un recurso de menú en una actividad con `onCreateOptionsMenu()`.
- Gestionar los clics en las opciones del menú con `onOptionsItemSelected()`.
- Marcar opciones y cambiar el menú en tiempo de ejecución con `onPrepareOptionsMenu()` e `invalidateOptionsMenu()`.
- Conocer la alternativa moderna `MenuProvider`.

---

## 1. Primero, una barra de la app

El menú de opciones se muestra en la **barra de la app** (*app bar* o *toolbar*). Por eso, antes de nada, la actividad tiene que **tener** esa barra.

> ⚠️ **Las plantillas actuales no la incluyen.** Un proyecto *Empty Views Activity* usa un tema que termina en **`NoActionBar`** (en `res/values/themes.xml`):
>
> ```xml
> <style name="Base.Theme.PizzeriaMenus" parent="Theme.Material3.DayNight.NoActionBar">
> ```
>
> Si cargas un menú en una actividad sin barra, **no se ve nada** y no aparece ningún error.

Hay dos formas de tener barra:

| Opción | Cómo | Ventajas |
|---|---|---|
| **A. Toolbar en el layout** *(recomendada)* | Se mantiene el tema `NoActionBar`, se añade una `MaterialToolbar` al layout y se registra con `setSupportActionBar()`. | Control total: posición, colores, se puede ver en el editor de layouts. |
| **B. Tema con barra** | Se cambia el `parent` del tema a `Theme.Material3.DayNight` (sin `NoActionBar`). | Más rápida, pero la barra no aparece en el layout y es menos personalizable. |

### 1.1 Opción A: añadir una `MaterialToolbar`

**Layout** (`activity_main.xml`):

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <com.google.android.material.appbar.MaterialToolbar
        android:id="@+id/toolbar"
        android:layout_width="0dp"
        android:layout_height="?attr/actionBarSize"
        android:background="?attr/colorPrimaryContainer"
        app:title="@string/app_name"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <TextView
        android:id="@+id/tvContenido"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/bienvenida"
        app:layout_constraintTop_toBottomOf="@id/toolbar"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

| Atributo | Para qué |
|---|---|
| `android:layout_height="?attr/actionBarSize"` | La altura estándar de una barra de la app. El `?attr/` toma el valor del tema. |
| `android:background="?attr/colorPrimaryContainer"` | Un color del tema Material 3, para que se distinga del contenido. |
| `app:title` | El título que se muestra en la barra. |

**Actividad**: se registra la toolbar como barra de la actividad con **`setSupportActionBar()`**:

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        ViewCompat.setOnApplyWindowInsetsListener(binding.main) { v, insets ->
            val systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars())
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom)
            insets
        }

        // La toolbar del layout pasa a ser la barra de la app
        setSupportActionBar(binding.toolbar)
    }
}
```

> 📌 **Relación con el punto 2:** gracias a View Binding, la toolbar está disponible directamente como `binding.toolbar`.

> 📝 **Recuerda (presentación de clase):** si usas `setSupportActionBar()` con un tema que **ya** tiene barra (sin `NoActionBar`), la app falla al arrancar: *"This Activity already has an action bar supplied by the window decor"*. Elige **una** de las dos opciones.

---

## 2. Cargar el menú: `onCreateOptionsMenu()`

Una vez hay barra, el menú se carga **sobrescribiendo** el método `onCreateOptionsMenu()` de la actividad. Android lo llama automáticamente cuando necesita mostrar el menú:

```kotlin
override fun onCreateOptionsMenu(menu: Menu): Boolean {
    // "Infla" el recurso XML y añade sus opciones al objeto menu
    menuInflater.inflate(R.menu.menu_principal, menu)
    return true    // true: el menú se muestra
}
```

| Elemento | Qué es |
|---|---|
| `menu: Menu` | Objeto menú, vacío, que nos pasa Android para que lo rellenemos. |
| `menuInflater` | Propiedad de la actividad que convierte un recurso XML de menú en objetos `MenuItem`. |
| `inflate(R.menu.menu_principal, menu)` | Lee `res/menu/menu_principal.xml` y añade sus opciones a `menu`. |
| `return true` | Indica que el menú debe mostrarse. Con `false`, no se muestra. |

> 📌 **Relación con el punto 2.3:** "inflar" es el mismo concepto que en `ActivityMainBinding.inflate()`: convertir un XML en objetos. Para layouts se usa `layoutInflater` y para menús, `menuInflater`.

Ejecuta la app: en la barra aparecen los iconos con `ifRoom` y el botón ⋮ con el resto de opciones. **Todavía no hacen nada al pulsarlas**: eso es lo siguiente.

---

## 3. Gestionando los clics en el menú

Cuando el usuario pulsa **cualquier** opción del menú (un icono de la barra o una opción del desbordamiento), Android llama al método **`onOptionsItemSelected()`** y le pasa la opción pulsada (`MenuItem`). Con su **id** sabemos cuál es:

```kotlin
override fun onOptionsItemSelected(item: MenuItem): Boolean {
    return when (item.itemId) {
        R.id.action_carrito -> {
            Snackbar.make(binding.root, R.string.abriendo_carrito, Snackbar.LENGTH_SHORT).show()
            true
        }
        R.id.action_carta -> {
            Snackbar.make(binding.root, R.string.abriendo_carta, Snackbar.LENGTH_SHORT).show()
            true
        }
        R.id.action_ajustes -> {
            Snackbar.make(binding.root, R.string.abriendo_ajustes, Snackbar.LENGTH_SHORT).show()
            true
        }
        R.id.action_acerca_de -> {
            mostrarAcercaDe()
            true
        }
        else -> super.onOptionsItemSelected(item)
    }
}

private fun mostrarAcercaDe() {
    MaterialAlertDialogBuilder(this)
        .setTitle(R.string.menu_acerca_de)
        .setMessage(R.string.acerca_de_texto)
        .setPositiveButton(R.string.aceptar, null)
        .show()
}
```

### 3.1 Qué devuelve

| Valor devuelto | Significado |
|---|---|
| `true` | "**He gestionado** este clic". Android no hace nada más. |
| `super.onOptionsItemSelected(item)` | "**No es mío**": la clase padre lo gestiona. Necesario para opciones que añade el propio sistema, como el botón *Arriba* (punto 4.3). |

> ✅ **Patrón:** `when` sobre `item.itemId`, cada rama termina devolviendo `true`, y la rama `else` llama a `super`. Es el mismo esquema que en la presentación de clase.

> 📌 **Relación con Kotlin (UT01):** en Kotlin, `when` es una **expresión**: devuelve el valor de la última línea de la rama elegida. Por eso podemos escribir `return when (...) { ... }` y que cada rama termine en `true`.

> ⚠️ En código Java antiguo verás `switch (item.getItemId()) { case R.id.…: }`. Desde el Android Gradle Plugin 8, los `R.id` ya no son constantes y ese `switch` **no compila**. El `when` de Kotlin no tiene ese problema, porque no exige valores constantes.

### 3.2 Opciones marcables

En el punto 4.1 creamos un submenú "Ordenar" con opciones de selección única. Android **no** cambia la marca al pulsarlas: hay que hacerlo en el código:

```kotlin
R.id.orden_nombre, R.id.orden_precio -> {
    item.isChecked = true      // en un grupo "single", desmarca automáticamente las demás
    ordenarPor(item.itemId)
    true
}
```

Para una opción suelta con `android:checkable="true"` (una casilla), se invierte su estado:

```kotlin
R.id.action_modo_oscuro -> {
    item.isChecked = !item.isChecked
    true
}
```

---

## 4. Cambiar el menú en tiempo de ejecución

`onCreateOptionsMenu()` solo se ejecuta **una vez**, al crear el menú. Si después queremos cambiarlo (ocultar una opción, desactivarla o cambiar su texto), se usan dos métodos:

| Método | Qué hace |
|---|---|
| `onPrepareOptionsMenu(menu)` | Se sobrescribe. Android lo llama **justo antes** de mostrar el menú: es el sitio para ajustar sus opciones. |
| `invalidateOptionsMenu()` | Se llama desde nuestro código. Indica a Android que el menú ha cambiado: vuelve a llamar a `onPrepareOptionsMenu()` y redibuja la barra. |

**Ejemplo:** ocultar el carrito mientras esté vacío.

```kotlin
private var pizzasEnCarrito = 0

override fun onPrepareOptionsMenu(menu: Menu): Boolean {
    menu.findItem(R.id.action_carrito)?.isVisible = pizzasEnCarrito > 0
    return super.onPrepareOptionsMenu(menu)
}

private fun anadirPizza() {
    pizzasEnCarrito++
    invalidateOptionsMenu()     // fuerza a llamar otra vez a onPrepareOptionsMenu()
}
```

| Propiedad de `MenuItem` | Para qué |
|---|---|
| `isVisible` | Mostrar u ocultar la opción. |
| `isEnabled` | Activarla o desactivarla (se ve atenuada). |
| `title` | Cambiar su texto. |
| `setIcon(R.drawable.x)` | Cambiar su icono. |
| `isChecked` | Marcarla o desmarcarla. |

---

## 5. La opción de búsqueda

Si en el punto 4.1 añadiste una opción con `app:actionViewClass="androidx.appcompat.widget.SearchView"`, al pulsar el icono se despliega un campo de búsqueda en la barra. Para leer lo que escribe el usuario, se obtiene esa vista en `onCreateOptionsMenu()`:

```kotlin
override fun onCreateOptionsMenu(menu: Menu): Boolean {
    menuInflater.inflate(R.menu.menu_principal, menu)

    val buscador = menu.findItem(R.id.action_buscar)?.actionView as? SearchView
    buscador?.queryHint = getString(R.string.buscar_pizza)
    buscador?.setOnQueryTextListener(object : SearchView.OnQueryTextListener {
        override fun onQueryTextSubmit(texto: String?): Boolean {
            buscarPizza(texto.orEmpty())
            return true
        }
        override fun onQueryTextChange(texto: String?): Boolean = false
    })
    return true
}
```

> ⚠️ Importa `androidx.appcompat.widget.SearchView`, no `android.widget.SearchView`: tiene que coincidir con la clase indicada en el XML.

---

## 6. La alternativa moderna: `MenuProvider`

Los métodos `onCreateOptionsMenu()` y `onOptionsItemSelected()` siguen siendo válidos en las actividades y los usaremos en el resto del punto. Pero AndroidX ofrece una forma más moderna de declarar un menú: la interfaz **`MenuProvider`**, que agrupa todo en un objeto y se ata al ciclo de vida.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    // ... binding, setContentView, setSupportActionBar ...

    addMenuProvider(object : MenuProvider {
        override fun onCreateMenu(menu: Menu, menuInflater: MenuInflater) {
            menuInflater.inflate(R.menu.menu_principal, menu)
        }

        override fun onMenuItemSelected(item: MenuItem): Boolean = when (item.itemId) {
            R.id.action_ajustes -> { abrirAjustes(); true }
            else -> false
        }
    }, this)   // "this": el menú se elimina automáticamente al destruir la actividad
}
```

> 📝 Es la forma **obligatoria en fragmentos**, donde los métodos clásicos están obsoletos. En actividades puedes usar cualquiera de las dos, pero **no mezcles** ambas para el mismo menú.

---

## 7. Practica

1. En el proyecto `PizzeriaMenus` del punto 4.1, abre `res/values/themes.xml` y comprueba de qué tema hereda el de la app.
2. Añade una `MaterialToolbar` al layout y regístrala con `setSupportActionBar(binding.toolbar)`. Ejecuta la app: ¿aparece el título?
3. Sobrescribe `onCreateOptionsMenu()` para cargar `menu_principal`. Comprueba qué opciones salen en la barra y cuáles en el desbordamiento ⋮.
4. Gira el emulador a horizontal. ¿Caben ahora más iconos en la barra?
5. Sobrescribe `onOptionsItemSelected()` para que cada opción muestre una Snackbar distinta, y *Acerca de* muestre un diálogo con el nombre de la app y tu nombre.
6. Haz que las opciones del submenú "Ordenar" se marquen al pulsarlas.
7. Añade un botón "Añadir pizza" al layout y haz que la opción *Carrito* solo sea visible cuando haya al menos una pizza. Cambia también su título para que muestre la cantidad: "Carrito (3)".
8. Cambia temporalmente el tema a `Theme.Material3.DayNight` (sin `NoActionBar`), sin quitar `setSupportActionBar()`. ¿Qué ocurre? Deja el tema como estaba.
9. **Reto:** implementa la búsqueda de la sección 5. Al enviar el texto, muestra en el `TextView` del contenido "Buscando: *texto*". Después, reescribe el menú con `MenuProvider`.

---

## 8. Autoevaluación

1. ¿Por qué en un proyecto *Empty Views Activity* actual el menú de opciones no se ve si no se hace nada más?
2. ¿Qué dos formas hay de tener barra de la app? ¿Cuál se recomienda y por qué?
3. ¿Para qué sirve `setSupportActionBar()`? ¿Qué ocurre si se usa con un tema que ya tiene barra?
4. ¿Qué método se sobrescribe para cargar el menú? ¿Qué hace `menuInflater.inflate()`?
5. ¿Qué significa devolver `true` en `onCreateOptionsMenu()`?
6. ¿Qué método recibe los clics del menú? ¿Cómo se sabe qué opción se ha pulsado?
7. ¿Por qué la rama `else` de `onOptionsItemSelected()` debe llamar a `super`?
8. ¿Se marca automáticamente una opción `checkable` al pulsarla? ¿Cómo se marca?
9. ¿Para qué sirven `onPrepareOptionsMenu()` e `invalidateOptionsMenu()`? Pon un ejemplo.
10. ¿Qué es `MenuProvider`? ¿En qué caso es obligatorio usarlo?

---

## 📚 Para saber más (documentación oficial)

- [Cómo agregar menús · Menú de opciones](https://developer.android.com/guide/topics/ui/menus?hl=es-419#options-menu) · `onCreateOptionsMenu()`, `onOptionsItemSelected()` y cambios en tiempo de ejecución.
- [Cómo configurar la barra de la app](https://developer.android.com/training/appbar/setting-up?hl=es-419) · Toolbar, `setSupportActionBar()` y temas `NoActionBar`.
- [Cómo agregar y administrar acciones](https://developer.android.com/training/appbar/actions?hl=es-419) · Responder a las acciones de la barra.
- [Cómo usar vistas y proveedores de acciones](https://developer.android.com/training/appbar/action-views?hl=es-419) · `SearchView` como vista de acción.

---

⬅️ [4.1 ¿Cómo definimos un menú?](04-1-definir-menu.md) · 🏠 [Índice](../README.md) · ➡️ [4.3 Menús comunes a varias actividades](04-3-menus-comunes-intents.md)
