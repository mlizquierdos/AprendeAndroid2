# 2.3 Uso de View Binding

> **Unidad 2 · Aplicaciones Android. Interfaz de usuario**
> Módulo: Programación multimedia y dispositivos móviles (2º DAM)

## Objetivos

Al terminar este apartado deberías ser capaz de:

- Usar View Binding en una actividad: crear la clase de vinculación, cargar el layout y acceder a las vistas.
- Asignar *listeners* y leer o modificar las vistas a través de `binding`.
- Acceder a las vistas de un layout incluido con `<include>`.
- Diferenciar `inflate()` y `bind()`, y reconocer los casos en que se usa cada uno.
- Migrar una actividad que usa `findViewById()` a View Binding.

---

## 1. View Binding en una actividad: el patrón

En una actividad, View Binding siempre se usa con los mismos tres pasos:

```kotlin
import com.dam.pruebabinding.databinding.ActivityMainBinding

class MainActivity : AppCompatActivity() {

    // 1. Propiedad para la clase de vinculación.
    //    lateinit: se inicializa en onCreate, no al crear el objeto.
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // 2. Se crea la clase de vinculación: "infla" el XML y crea todas las vistas
        binding = ActivityMainBinding.inflate(layoutInflater)

        // 3. Se muestra la vista raíz de ese árbol de vistas
        setContentView(binding.root)

        // A partir de aquí, cada vista está disponible como binding.<id>
        binding.tvSaludo.text = getString(R.string.saludo)
    }
}
```

| Paso | Qué hace |
|---|---|
| `private lateinit var binding` | Guarda la referencia para usarla en cualquier método de la clase. |
| `ActivityMainBinding.inflate(layoutInflater)` | Lee `activity_main.xml`, crea los objetos de todas sus vistas y los enlaza a las propiedades de la clase. |
| `setContentView(binding.root)` | Muestra en pantalla ese árbol de vistas. Sustituye a `setContentView(R.layout.activity_main)`. |

> ⚠️ **Error muy frecuente:** dejar también la línea `setContentView(R.layout.activity_main)` de la plantilla.
>
> ```kotlin
> binding = ActivityMainBinding.inflate(layoutInflater)
> setContentView(binding.root)
> setContentView(R.layout.activity_main)   // ❌ ¡sobra!
> ```
>
> Esa segunda llamada crea **otro** árbol de vistas distinto y lo muestra en pantalla. `binding` apunta al primero, que ya no se ve: los *listeners* no responden y los cambios de texto no aparecen. **No da ningún error**, por eso cuesta tanto encontrarlo. Debe haber **un solo** `setContentView()`, con `binding.root`.

### 1.1 Con el código *edge-to-edge* de la plantilla

La plantilla actual incluye un bloque que busca la vista raíz con `findViewById(R.id.main)`. Con View Binding se sustituye por la propiedad correspondiente:

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    enableEdgeToEdge()
    binding = ActivityMainBinding.inflate(layoutInflater)
    setContentView(binding.root)

    // Antes: findViewById(R.id.main)
    ViewCompat.setOnApplyWindowInsetsListener(binding.main) { v, insets ->
        val systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars())
        v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom)
        insets
    }
}
```

Como la vista raíz tiene `android:id="@+id/main"`, existe `binding.main`. Es la misma vista que `binding.root`, así que también puedes usar `binding.root`.

---

## 2. Trabajar con las vistas

Todo lo visto en el punto 1.3 sigue igual. Solo cambia cómo se llega a la vista: escribiendo `binding.` delante.

```kotlin
// Leer y escribir texto
val nombre = binding.etNombre.text.toString().trim()
binding.tvResumen.text = getString(R.string.saludo_nombre, nombre, 3)

// Listeners
binding.btnEnviar.setOnClickListener { enviar() }
binding.cbCondiciones.setOnCheckedChangeListener { _, marcado ->
    binding.btnEnviar.isEnabled = marcado
}

// Estado y visibilidad
binding.pbCargando.isVisible = true
binding.etNombre.error = getString(R.string.error_nombre)
```

Al escribir `binding.`, Android Studio sugiere **solo las vistas de ese layout** y con su tipo exacto. Prueba a escribir `binding.btn` y verás la lista.

### 2.1 Menos repetición con `with`

Si una función usa muchas vistas, la función de ámbito `with` de Kotlin evita repetir `binding.` en cada línea:

```kotlin
private fun limpiarFormulario() {
    with(binding) {
        etNombre.text?.clear()
        etEmail.text?.clear()
        cbCondiciones.isChecked = false
        tvResumen.text = ""
    }
}
```

> 📌 **Relación con Kotlin (UT01):** dentro del bloque `with(binding) { ... }`, `this` es `binding`, por eso se puede escribir `etNombre` en lugar de `binding.etNombre`. Úsalo con moderación: si el bloque es largo, cuesta saber qué viene del binding y qué de la actividad.

---

## 3. Layouts incluidos con `<include>`

En el punto 1.2 vimos cómo reutilizar un layout con `<include>`. View Binding también genera la clase del layout incluido, y la forma de acceder depende de cómo se incluya.

### 3.1 `<include>` con id

Si la etiqueta `<include>` tiene un **id**, la clase de vinculación principal tendrá una propiedad con ese nombre, del tipo de la clase del layout incluido:

```xml
<!-- res/layout/barra_botones.xml -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal">
    <Button android:id="@+id/btnCancelar" ... />
    <Button android:id="@+id/btnGuardar" ... />
</LinearLayout>
```

```xml
<!-- en activity_main.xml -->
<include
    android:id="@+id/barraBotones"
    layout="@layout/barra_botones" />
```

```kotlin
// binding.barraBotones es de tipo BarraBotonesBinding
binding.barraBotones.btnGuardar.setOnClickListener { guardar() }
binding.barraBotones.btnCancelar.setOnClickListener { finish() }
```

> ⚠️ Si el `<include>` **no** tiene id, sus vistas no aparecen en `ActivityMainBinding`. Ponle siempre un id.

### 3.2 `<include>` de un layout con `<merge>`

Un layout cuya raíz es `<merge>` no aporta vista propia (sus hijos pasan a formar parte del padre), así que el id del `<include>` no se puede usar. En ese caso se enlaza con **`bind()`**, a partir de la vista raíz del layout principal:

```kotlin
// barra_botones.xml empieza por <merge>
val barra = BarraBotonesBinding.bind(binding.root)
barra.btnGuardar.setOnClickListener { guardar() }
```

---

## 4. `inflate()` frente a `bind()`

Toda clase de vinculación ofrece dos formas de crearse:

| Método | Qué hace | Cuándo se usa |
|---|---|---|
| `inflate(layoutInflater)` | **Crea** las vistas a partir del XML y las enlaza. | Cuando las vistas aún no existen: en el `onCreate()` de una actividad. |
| `inflate(inflater, parent, false)` | Igual, pero indicando el contenedor padre (sin añadirse a él todavía). | Cuando la vista se va a colocar dentro de otra: fragmentos, filas de una lista, diálogos personalizados. |
| `bind(view)` | **No crea nada**: enlaza vistas que ya existen. | Cuando el layout ya se ha inflado por otro medio: `<include>` con `<merge>`, código existente. |

### Avance: dónde volveremos a ver View Binding

En los próximos puntos de la unidad usaremos las mismas clases en otros contextos. Por ahora basta con reconocer el patrón:

**Diálogo con layout propio (punto 3):**

```kotlin
val dialogBinding = DialogConfirmarBinding.inflate(layoutInflater)
AlertDialog.Builder(this)
    .setView(dialogBinding.root)
    .setPositiveButton("Aceptar") { _, _ ->
        val texto = dialogBinding.etComentario.text.toString()
    }
    .show()
```

**Fila de una lista (punto 5, RecyclerView):**

```kotlin
val filaBinding = ItemContactoBinding.inflate(
    LayoutInflater.from(parent.context), parent, false
)
filaBinding.tvNombre.text = contacto.nombre
```

> 📝 **En fragmentos** el uso cambia ligeramente: la vista del fragmento puede destruirse antes que el propio fragmento, así que la referencia al binding se pone a `null` en `onDestroyView()`. Lo veremos cuando trabajemos con fragmentos.

---

## 5. Ejemplo integrador: migrar *Pizzería DAM* a View Binding

Vamos a pasar a View Binding la app del punto 1.3. **El layout no cambia**: solo la `MainActivity`.

### 5.1 Antes: con `findViewById()`

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var tilNombre: TextInputLayout
    private lateinit var etNombre: TextInputEditText
    private lateinit var rgTamano: RadioGroup
    private lateinit var spMasa: Spinner
    private lateinit var cbQueso: CheckBox
    private lateinit var cbChampinones: CheckBox
    private lateinit var cbJamon: CheckBox
    private lateinit var tvCantidad: TextView
    private lateinit var sbCantidad: SeekBar
    private lateinit var swEnvio: MaterialSwitch
    private lateinit var tvResumen: TextView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContentView(R.layout.activity_main)
        // ... insets con findViewById(R.id.main) ...
        inicializarVistas()
        configurarListeners()
    }

    private fun inicializarVistas() {
        tilNombre = findViewById(R.id.tilNombre)
        etNombre = findViewById(R.id.etNombre)
        // ... nueve líneas más ...
    }
    // ...
}
```

### 5.2 Después: con View Binding

Primero, activa View Binding en `app/build.gradle.kts` (punto 2.2). Después:

```kotlin
package com.dam.pizzeria

import android.os.Bundle
import android.widget.RadioButton
import android.widget.SeekBar
import androidx.activity.enableEdgeToEdge
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.ViewCompat
import androidx.core.view.WindowInsetsCompat
import androidx.core.widget.doAfterTextChanged
import com.dam.pizzeria.databinding.ActivityMainBinding

class MainActivity : AppCompatActivity() {

    companion object {
        private const val PRECIO_PEQUENA = 8.0
        private const val PRECIO_MEDIANA = 10.5
        private const val PRECIO_FAMILIAR = 14.0
        private const val PRECIO_EXTRA = 1.5
        private const val PRECIO_ENVIO = 3.0
    }

    // Una sola propiedad sustituye a las once anteriores
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

        // inicializarVistas() ya no es necesaria
        actualizarTextoCantidad(binding.sbCantidad.progress)
        configurarListeners()
    }

    /** Registra los escuchadores de eventos. */
    private fun configurarListeners() {
        binding.etNombre.doAfterTextChanged { binding.tilNombre.error = null }

        binding.sbCantidad.setOnSeekBarChangeListener(object : SeekBar.OnSeekBarChangeListener {
            override fun onProgressChanged(seekBar: SeekBar?, progress: Int, fromUser: Boolean) {
                actualizarTextoCantidad(progress)
            }
            override fun onStartTrackingTouch(seekBar: SeekBar?) {}
            override fun onStopTrackingTouch(seekBar: SeekBar?) {}
        })

        // btnCalcular ya no necesita findViewById
        binding.btnCalcular.setOnClickListener { calcularPedido() }
    }

    private fun actualizarTextoCantidad(progress: Int) {
        binding.tvCantidad.text = getString(R.string.cantidad_valor, progress + 1)
    }

    /** Valida los datos, calcula el precio y muestra el resumen. */
    private fun calcularPedido() {
        with(binding) {
            // 1. Validación
            val nombre = etNombre.text.toString().trim()
            if (nombre.isEmpty()) {
                tilNombre.error = getString(R.string.error_nombre)
                etNombre.requestFocus()
                return
            }

            // 2. Lectura de los datos de la interfaz
            val precioBase = when (rgTamano.checkedRadioButtonId) {
                R.id.rbPequena -> PRECIO_PEQUENA
                R.id.rbFamiliar -> PRECIO_FAMILIAR
                else -> PRECIO_MEDIANA
            }
            // El RadioButton marcado se busca dentro del RadioGroup
            val tamano = root.findViewById<RadioButton>(rgTamano.checkedRadioButtonId).text.toString()
            val masa = spMasa.selectedItem.toString()
            val cantidad = sbCantidad.progress + 1

            val extras = listOf(cbQueso, cbChampinones, cbJamon)
                .filter { it.isChecked }
                .map { it.text.toString() }

            // 3. Cálculo
            val precioUnidad = precioBase + extras.size * PRECIO_EXTRA
            var total = precioUnidad * cantidad
            if (swEnvio.isChecked) total += PRECIO_ENVIO

            // 4. Mostrar resultado
            val textoExtras = if (extras.isEmpty()) getString(R.string.sin_extras)
                              else extras.joinToString(", ")

            tvResumen.text = getString(
                R.string.resumen_pedido,
                nombre, cantidad, tamano.lowercase(), masa.lowercase(), textoExtras, total
            )
        }
    }
}
```

### 5.3 Qué ha cambiado

| | Con `findViewById()` | Con View Binding |
|---|---|---|
| Propiedades de vistas | 11 `lateinit var` | 1 (`binding`) |
| Método `inicializarVistas()` | Necesario | Eliminado |
| `setContentView()` | `R.layout.activity_main` | `binding.root` |
| Insets *edge-to-edge* | `findViewById(R.id.main)` | `binding.main` |
| Botón *Calcular* | `findViewById<Button>(R.id.btnCalcular)` | `binding.btnCalcular` |
| Un id mal escrito | Falla al **ejecutar** | Falla al **compilar** |

**Aspectos a destacar:**

- `calcularPedido()` envuelve su cuerpo en `with(binding) { ... }`, así que dentro del bloque se escribe `etNombre`, no `binding.etNombre`. El `return` de la validación sigue saliendo de la función, porque `with` es una función `inline`.
- El `RadioButton` marcado se obtiene con `root.findViewById(...)`, porque su id se conoce **en ejecución** (`checkedRadioButtonId`). Es el único caso en que sigue teniendo sentido `findViewById()`: cuando el id no se conoce al escribir el código. Si prefieres evitarlo, puedes usar un `when` sobre `checkedRadioButtonId`, como hacemos con el precio.

---

## 6. Practica

1. En el proyecto `PruebaBinding` del punto 2.2, escribe el patrón completo de la sección 1: propiedad `binding`, `inflate()` y `setContentView(binding.root)`.
2. Al pulsar un botón `btnSaludar`, muestra en `tvSaludo` el texto "Hola, " seguido de lo escrito en `etNombre`. Si el campo está vacío, muestra un error en el propio `EditText`.
3. Añade a propósito una segunda línea `setContentView(R.layout.activity_main)` después de la tuya. Ejecuta la app y pulsa el botón. ¿Qué ocurre? ¿Aparece algún error en el Logcat? Quita la línea y vuelve a probar.
4. Crea un layout `barra_botones.xml` con dos botones, `btnLimpiar` y `btnSalir`, inclúyelo en `activity_main.xml` con un id y programa sus clics a través de `binding.barraBotones`.
5. Reescribe la función que limpia el formulario usando `with(binding)`.
6. **Migración:** pasa a View Binding la app *Pizzería DAM* siguiendo la sección 5. Comprueba que funciona igual que antes.
7. **Reto:** migra también la actividad práctica *InscripcionCursos* del punto 1.3. Cuenta cuántas líneas de código te has ahorrado.

---

## 7. Autoevaluación

1. ¿Cuáles son los tres pasos para usar View Binding en una actividad?
2. ¿Por qué la propiedad `binding` se declara como `lateinit var`?
3. ¿Qué se le pasa a `setContentView()` cuando usamos View Binding?
4. ¿Qué ocurre si después de `setContentView(binding.root)` se llama también a `setContentView(R.layout.activity_main)`? ¿Por qué es un error difícil de detectar?
5. En el código *edge-to-edge* de la plantilla, ¿por qué podemos escribir `binding.main`?
6. ¿Qué ventaja aporta `with(binding) { ... }`? ¿Qué inconveniente puede tener?
7. ¿Cómo se accede a un botón `btnGuardar` de un layout incluido con `<include android:id="@+id/barraBotones" ... />`?
8. ¿Qué diferencia hay entre `inflate()` y `bind()`? Pon un ejemplo en el que se use cada uno.
9. ¿En qué caso sigue teniendo sentido `findViewById()` aunque usemos View Binding?
10. Al migrar *Pizzería DAM*, ¿qué partes del código desaparecen y cuáles no cambian?

---

## 📚 Para saber más (documentación oficial)

- [Vinculación de vistas · Uso](https://developer.android.com/topic/libraries/view-binding?hl=es-419#usage) · Uso en actividades y fragmentos, con ejemplos.
- [Cómo migrar de Kotlin synthetics a la vinculación de vistas de Jetpack](https://developer.android.com/topic/libraries/view-binding/migration?hl=es-419) · Pasos para migrar código existente.
- [Funciones de alcance (Kotlin)](https://kotlinlang.org/docs/scope-functions.html) (en inglés) · `with`, `apply`, `let`, `run` y `also`.
- [Cómo reutilizar diseños con &lt;include&gt;](https://developer.android.com/develop/ui/views/layout/improving-layouts/reusing-layouts?hl=es-419) · `<include>` y `<merge>`.

---

⬅️ [2.2 Activación de View Binding](02-2-activacion-view-binding.md) · 🏠 [Índice](../README.md) · ➡️ [3.1 Toasts](03-1-toasts.md)
