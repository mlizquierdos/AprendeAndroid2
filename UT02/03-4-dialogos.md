# 3.4 Diálogos

> **Unidad 2 · Aplicaciones Android. Interfaz de usuario**
> Módulo: Programación multimedia y dispositivos móviles (2º DAM)

## Objetivos

Al terminar este apartado deberías ser capaz de:

- Explicar qué es un diálogo y cuándo usarlo frente a un Toast o una Snackbar.
- Construir diálogos de alerta con botones, listas y layouts propios.
- Crear diálogos con `DialogFragment` y explicar por qué sobreviven a los cambios de configuración.
- Pasar datos a un diálogo mediante argumentos.
- Devolver la respuesta del usuario a la actividad con *callbacks*: una interfaz o la Fragment Result API.

---

## 1. Qué es un diálogo

Un **diálogo** es una ventana pequeña que aparece **sobre** la pantalla actual y **interrumpe** al usuario hasta que responde. Se usa cuando hace falta una decisión o un dato **antes de continuar**.

```
┌──────────────────────────────────┐
│ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │  ← la pantalla de fondo se oscurece
│ ░░ ┌──────────────────────────┐ ░ │
│ ░░ │ ¿Vaciar el pedido?       │ ░ │  ← título
│ ░░ │                          │ ░ │
│ ░░ │ Se borrarán todos los    │ ░ │  ← mensaje
│ ░░ │ datos del formulario.    │ ░ │
│ ░░ │                          │ ░ │
│ ░░ │      CANCELAR   VACIAR   │ ░ │  ← botones
│ ░░ └──────────────────────────┘ ░ │
│ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │
└──────────────────────────────────┘
```

| Úsalo para… | No lo uses para… |
|---|---|
| Confirmar una acción que no se puede deshacer ("¿Borrar la cuenta?"). | Confirmar algo que ya ha ocurrido ("Guardado"): usa un Toast o una Snackbar. |
| Pedir un dato imprescindible para continuar. | Mostrar información que no requiere respuesta. |
| Elegir entre varias opciones. | Acciones que se pueden deshacer: mejor una Snackbar con "Deshacer". |

> 💡 Los diálogos interrumpen. Si se abusa de ellos, el usuario acaba pulsando "Aceptar" sin leer.

---

## 2. Diálogos de alerta (`AlertDialog`)

### 2.1 El diálogo básico

El diálogo más habitual es el **diálogo de alerta**: título, mensaje y hasta tres botones. Con un tema Material 3, se construye con `MaterialAlertDialogBuilder`, que sigue el patrón **Builder** (como `NotificationCompat.Builder` en el punto 3.3):

```kotlin
import com.google.android.material.dialog.MaterialAlertDialogBuilder

MaterialAlertDialogBuilder(this)
    .setTitle(R.string.vaciar_titulo)                         // "¿Vaciar el pedido?"
    .setMessage(R.string.vaciar_mensaje)                      // "Se borrarán todos los datos…"
    .setNegativeButton(R.string.cancelar, null)               // null: solo cierra el diálogo
    .setPositiveButton(R.string.vaciar) { _, _ ->
        vaciarFormulario()
    }
    .show()
```

| Botón | Método | Uso | Posición |
|---|---|---|---|
| Positivo | `setPositiveButton()` | Confirma la acción: "Aceptar", "Borrar". | Derecha |
| Negativo | `setNegativeButton()` | Cancela: "Cancelar". | Izquierda del positivo |
| Neutro | `setNeutralButton()` | Una tercera opción: "Recordar más tarde". | Extremo izquierdo |

- Pulsar **cualquier** botón cierra el diálogo automáticamente.
- La lambda de cada botón recibe dos parámetros (el diálogo y el botón pulsado). Como no los usamos, se escriben `_, _`.
- El texto del botón positivo debe decir **qué hace** ("Vaciar"), no solo "Sí" o "Aceptar".

> 📝 `MaterialAlertDialogBuilder` crea un `androidx.appcompat.app.AlertDialog` con el estilo de Material 3. También existe `AlertDialog.Builder` de AppCompat, que funciona igual pero no aplica el estilo Material. En la presentación de clase verás ese segundo.

### 2.2 Impedir que se cierre al tocar fuera

Por defecto, el diálogo se cierra si el usuario pulsa fuera o el botón *Atrás*. Si la respuesta es obligatoria:

```kotlin
MaterialAlertDialogBuilder(this)
    .setTitle(R.string.condiciones_titulo)
    .setMessage(R.string.condiciones_texto)
    .setCancelable(false)
    .setPositiveButton(R.string.acepto) { _, _ -> continuar() }
    .show()
```

### 2.3 Diálogos con listas

En lugar de un mensaje, el diálogo puede mostrar una lista de opciones:

```kotlin
val masas = resources.getStringArray(R.array.masas)
```

**Lista simple** (al pulsar una opción, el diálogo se cierra):

```kotlin
MaterialAlertDialogBuilder(this)
    .setTitle(R.string.elige_masa)
    .setItems(masas) { _, posicion ->
        binding.tvMasa.text = masas[posicion]
    }
    .show()
```

**Selección única** (con `RadioButton`; se confirma con un botón):

```kotlin
var elegida = 0
MaterialAlertDialogBuilder(this)
    .setTitle(R.string.elige_masa)
    .setSingleChoiceItems(masas, elegida) { _, posicion -> elegida = posicion }
    .setPositiveButton(R.string.aceptar) { _, _ -> binding.tvMasa.text = masas[elegida] }
    .setNegativeButton(R.string.cancelar, null)
    .show()
```

**Selección múltiple** (con `CheckBox`):

```kotlin
val extras = resources.getStringArray(R.array.extras)
val marcados = BooleanArray(extras.size)   // todos a false

MaterialAlertDialogBuilder(this)
    .setTitle(R.string.elige_extras)
    .setMultiChoiceItems(extras, marcados) { _, posicion, marcado ->
        marcados[posicion] = marcado
    }
    .setPositiveButton(R.string.aceptar) { _, _ ->
        binding.tvExtras.text = extras.filterIndexed { i, _ -> marcados[i] }.joinToString(", ")
    }
    .show()
```

> ⚠️ `setMessage()` y las listas son **incompatibles**: si pones un mensaje, la lista no se muestra. Usa solo el título.

### 2.4 Diálogos con un layout propio

Si necesitas campos de texto u otras vistas, puedes diseñar un layout y ponerlo en el diálogo con `setView()`. Con View Binding (punto 2.3):

```xml
<!-- res/layout/dialog_nombre.xml -->
<com.google.android.material.textfield.TextInputLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:paddingHorizontal="24dp"
    android:paddingTop="8dp"
    android:hint="@string/hint_nombre">

    <com.google.android.material.textfield.TextInputEditText
        android:id="@+id/etNombreDialogo"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:inputType="textPersonName|textCapWords" />

</com.google.android.material.textfield.TextInputLayout>
```

```kotlin
val dialogBinding = DialogNombreBinding.inflate(layoutInflater)

MaterialAlertDialogBuilder(this)
    .setTitle(R.string.tu_nombre)
    .setView(dialogBinding.root)
    .setPositiveButton(R.string.aceptar) { _, _ ->
        val nombre = dialogBinding.etNombreDialogo.text.toString().trim()
        binding.tvSaludo.text = getString(R.string.saludo_nombre, nombre)
    }
    .setNegativeButton(R.string.cancelar, null)
    .show()
```

---

## 3. Diálogos con `DialogFragment`

### 3.1 El problema: girar la pantalla

Prueba esto: muestra cualquiera de los diálogos anteriores y **gira el emulador**. El diálogo **desaparece**.

> 📝 **Recuerda (UT01 · ciclo de vida):** al girar la pantalla, la actividad se **destruye y se vuelve a crear**. El diálogo pertenecía a la actividad destruida, así que desaparece con ella. Si el usuario estaba escribiendo su nombre, lo pierde.

La solución recomendada por Android es crear el diálogo dentro de un **`DialogFragment`**.

### 3.2 Qué es un `DialogFragment`

Un **fragmento** es una porción reutilizable de interfaz con su propio ciclo de vida, que vive dentro de una actividad. Un `DialogFragment` es un fragmento especializado en mostrar un diálogo.

Su ventaja clave es que lo gestiona el **`FragmentManager`** de la actividad, que lo **vuelve a crear** automáticamente cuando la actividad se recrea. El diálogo sobrevive al giro.

| | Diálogo creado en la actividad | `DialogFragment` |
|---|---|---|
| Sobrevive al giro de pantalla | ❌ | ✅ |
| Reutilizable desde varias actividades | ❌ (código copiado) | ✅ (es una clase) |
| Recomendado por Android | — | ✅ |

### 3.3 Crear un `DialogFragment`

Se crea una clase que hereda de `DialogFragment` y se sobrescribe **`onCreateDialog()`**, que debe **devolver** el diálogo ya construido:

```kotlin
import android.app.Dialog
import android.os.Bundle
import androidx.fragment.app.DialogFragment
import com.google.android.material.dialog.MaterialAlertDialogBuilder

class VaciarPedidoDialog : DialogFragment() {

    override fun onCreateDialog(savedInstanceState: Bundle?): Dialog {
        return MaterialAlertDialogBuilder(requireContext())
            .setTitle(R.string.vaciar_titulo)
            .setMessage(R.string.vaciar_mensaje)
            .setNegativeButton(R.string.cancelar, null)
            .setPositiveButton(R.string.vaciar) { _, _ ->
                // ¿Cómo avisamos a la actividad? → sección 4
            }
            .create()      // create(), no show(): lo muestra el DialogFragment
    }

    companion object {
        const val TAG = "VaciarPedidoDialog"
    }
}
```

Fíjate en las diferencias con la sección 2:

- El contexto ya no es `this` (el fragmento no es un `Context`), sino **`requireContext()`**.
- Se termina con **`create()`**, no con `show()`: `onCreateDialog()` solo construye el diálogo.

### 3.4 Mostrarlo

Desde la actividad, se crea el objeto y se llama a `show()` pasando el `FragmentManager` y una **etiqueta** que lo identifica:

```kotlin
binding.btnVaciar.setOnClickListener {
    VaciarPedidoDialog().show(supportFragmentManager, VaciarPedidoDialog.TAG)
}
```

### 3.5 Pasar datos al diálogo: argumentos

A menudo el diálogo necesita datos, por ejemplo el nombre de la pizza que se va a borrar. La tentación es pasarlos por el constructor:

```kotlin
class BorrarPizzaDialog(private val pizza: String) : DialogFragment()   // ❌ NO
```

> ⚠️ **No funciona bien.** Cuando Android recrea el fragmento tras un giro, lo hace con el **constructor sin parámetros**, así que el dato se pierde (y en algunos casos la app falla).

La forma correcta es guardarlos en los **argumentos** del fragmento, un `Bundle` que Android conserva al recrearlo. Se suele hacer con una función de fábrica en el `companion object`:

```kotlin
import androidx.core.os.bundleOf

class BorrarPizzaDialog : DialogFragment() {

    override fun onCreateDialog(savedInstanceState: Bundle?): Dialog {
        val pizza = requireArguments().getString(ARG_PIZZA)

        return MaterialAlertDialogBuilder(requireContext())
            .setTitle(getString(R.string.borrar_pizza_titulo, pizza))   // "¿Borrar la pizza %1$s?"
            .setNegativeButton(R.string.cancelar, null)
            .setPositiveButton(R.string.borrar) { _, _ -> /* sección 4 */ }
            .create()
    }

    companion object {
        const val TAG = "BorrarPizzaDialog"
        private const val ARG_PIZZA = "pizza"

        // Función de fábrica: crea el diálogo con sus argumentos
        fun newInstance(pizza: String) = BorrarPizzaDialog().apply {
            arguments = bundleOf(ARG_PIZZA to pizza)
        }
    }
}
```

```kotlin
// En la actividad
BorrarPizzaDialog.newInstance("Barbacoa").show(supportFragmentManager, BorrarPizzaDialog.TAG)
```

> 📌 **Relación con la UT01:** es el mismo `Bundle` que usamos en `onSaveInstanceState()`. Android lo guarda y lo devuelve al recrear el fragmento.

---

## 4. Diálogos y *callbacks*

### 4.1 El problema

El diálogo es una clase **distinta** de la actividad. Cuando el usuario pulsa "Vaciar", el diálogo tiene que **avisar a la actividad** para que vacíe el formulario, porque es ella la que tiene acceso a sus vistas. A ese aviso se le llama ***callback***: la actividad deja una forma de "llamarla de vuelta".

> ⚠️ **Lo que no hay que hacer:** pasar una lambda por el constructor, como `VaciarPedidoDialog { vaciarFormulario() }`. Tras un giro, el fragmento se recrea sin esa lambda y el botón deja de hacer nada. Es el mismo problema que el de la sección 3.5.

Veremos dos soluciones correctas.

### 4.2 Solución 1: interfaz y `onAttach()`

Es la forma clásica, la que aparece en la presentación de clase.

**1. El diálogo define una interfaz** con los métodos que la actividad debe implementar:

```kotlin
class VaciarPedidoDialog : DialogFragment() {

    interface VaciarPedidoListener {
        fun onVaciarConfirmado()
    }

    private var listener: VaciarPedidoListener? = null

    // 2. Al unirse a la actividad, guarda una referencia a ella como listener
    override fun onAttach(context: Context) {
        super.onAttach(context)
        listener = context as? VaciarPedidoListener
    }

    override fun onCreateDialog(savedInstanceState: Bundle?): Dialog {
        return MaterialAlertDialogBuilder(requireContext())
            .setTitle(R.string.vaciar_titulo)
            .setMessage(R.string.vaciar_mensaje)
            .setNegativeButton(R.string.cancelar, null)
            .setPositiveButton(R.string.vaciar) { _, _ ->
                listener?.onVaciarConfirmado()      // 3. Avisa a la actividad
            }
            .create()
    }

    override fun onDetach() {
        super.onDetach()
        listener = null     // evita guardar una referencia a una actividad destruida
    }

    companion object {
        const val TAG = "VaciarPedidoDialog"
    }
}
```

**4. La actividad implementa la interfaz:**

```kotlin
class MainActivity : AppCompatActivity(), VaciarPedidoDialog.VaciarPedidoListener {

    // ...

    override fun onVaciarConfirmado() {
        vaciarFormulario()
        Snackbar.make(binding.root, R.string.pedido_vaciado, Snackbar.LENGTH_SHORT).show()
    }
}
```

| Paso | Dónde | Qué ocurre |
|---|---|---|
| 1 | Diálogo | Define la interfaz `VaciarPedidoListener`. |
| 2 | Diálogo · `onAttach()` | Recibe la actividad (`context`) y la guarda como listener. |
| 3 | Diálogo · botón | Llama a `listener?.onVaciarConfirmado()`. |
| 4 | Actividad | Implementa la interfaz y reacciona. |

Funciona tras un giro porque `onAttach()` se ejecuta **otra vez** con la actividad nueva.

> 📌 **Relación con POO:** es el patrón **Observer** que vimos con los *listeners* del punto 1.3: el diálogo no sabe qué actividad lo usa, solo que implementa la interfaz. Por eso `as?`: si la actividad no la implementa, `listener` es `null` y no falla.

### 4.3 Solución 2: la Fragment Result API

La forma **moderna** que recomienda Android. No necesita interfaces: el diálogo **publica un resultado** con una clave, y la actividad **escucha** esa clave.

**En el diálogo:**

```kotlin
import androidx.fragment.app.setFragmentResult

class NombreDialog : DialogFragment() {

    override fun onCreateDialog(savedInstanceState: Bundle?): Dialog {
        val dialogBinding = DialogNombreBinding.inflate(layoutInflater)

        return MaterialAlertDialogBuilder(requireContext())
            .setTitle(R.string.tu_nombre)
            .setView(dialogBinding.root)
            .setNegativeButton(R.string.cancelar, null)
            .setPositiveButton(R.string.aceptar) { _, _ ->
                val nombre = dialogBinding.etNombreDialogo.text.toString().trim()
                // Publica el resultado con una clave y un Bundle
                setFragmentResult(CLAVE_RESULTADO, bundleOf(EXTRA_NOMBRE to nombre))
            }
            .create()
    }

    companion object {
        const val TAG = "NombreDialog"
        const val CLAVE_RESULTADO = "resultado_nombre"
        const val EXTRA_NOMBRE = "nombre"
    }
}
```

**En la actividad** (en `onCreate()`, para que el listener exista también tras un giro):

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    binding = ActivityMainBinding.inflate(layoutInflater)
    setContentView(binding.root)

    supportFragmentManager.setFragmentResultListener(
        NombreDialog.CLAVE_RESULTADO, this
    ) { _, resultado ->
        val nombre = resultado.getString(NombreDialog.EXTRA_NOMBRE)
        binding.tvSaludo.text = getString(R.string.saludo_nombre, nombre)
    }

    binding.btnPedirNombre.setOnClickListener {
        NombreDialog().show(supportFragmentManager, NombreDialog.TAG)
    }
}
```

### 4.4 ¿Cuál usar?

| | Interfaz + `onAttach()` | Fragment Result API |
|---|---|---|
| Código en la actividad | Implementa una interfaz | Registra un listener por clave |
| Datos que devuelve | Cualquier tipo, con métodos a medida | Lo que quepa en un `Bundle` |
| Acoplamiento | La actividad debe implementar la interfaz | Solo comparten una clave |
| Dónde lo encontrarás | Código y tutoriales existentes | Código nuevo; recomendado por Android |

Las dos son correctas y sobreviven al giro. Conviene **conocer las dos**: la primera para entender código existente y la segunda para escribir código nuevo.

---

## 5. Diálogos de fecha y hora

Android incluye diálogos ya preparados para elegir una fecha (`DatePickerDialog`) o una hora (`TimePickerDialog`). Se usan con el mismo esquema: un `DialogFragment` que los devuelve en `onCreateDialog()` y un *callback* para enviar el resultado.

```kotlin
class FechaDialog : DialogFragment(), DatePickerDialog.OnDateSetListener {

    override fun onCreateDialog(savedInstanceState: Bundle?): Dialog {
        val hoy = Calendar.getInstance()
        return DatePickerDialog(
            requireContext(), this,
            hoy.get(Calendar.YEAR), hoy.get(Calendar.MONTH), hoy.get(Calendar.DAY_OF_MONTH)
        )
    }

    override fun onDateSet(view: DatePicker?, year: Int, month: Int, dayOfMonth: Int) {
        // month empieza en 0 (enero = 0)
        setFragmentResult("resultado_fecha", bundleOf(
            "fecha" to "%02d/%02d/%d".format(dayOfMonth, month + 1, year)
        ))
    }
}
```

> 📝 La presentación de clase desarrolla este ejemplo con la solución de la interfaz y `onAttach()`.

---

## 6. Practica

1. En el proyecto `AvisosDemo`, muestra un diálogo de alerta con título, mensaje y los botones "Cancelar" y "Aceptar". Al pulsar "Aceptar", muestra una Snackbar.
2. Añade un botón neutro "Más tarde" y comprueba su posición. Después, haz que el diálogo no se pueda cerrar tocando fuera.
3. Crea tres diálogos con la lista de masas de la *Pizzería DAM*: lista simple, selección única y selección múltiple de extras. Muestra el resultado en un `TextView`.
4. Abre cualquiera de los diálogos y gira el emulador. ¿Qué ocurre?
5. Pasa el diálogo de confirmación a una clase `ConfirmarDialog : DialogFragment()`. Repite el giro: ¿qué ocurre ahora?
6. Crea `BorrarPizzaDialog` con la función `newInstance(pizza)` y comprueba que el nombre de la pizza se mantiene tras girar la pantalla.
7. Haz que `ConfirmarDialog` avise a la actividad con una interfaz y `onAttach()`.
8. Crea un `NombreDialog` con un layout propio (`dialog_nombre.xml` y View Binding) que devuelva el nombre a la actividad con la Fragment Result API.
9. **Reto:** en la *Pizzería DAM*, añade un botón "Vaciar pedido" que pida confirmación con un `DialogFragment` y, al confirmar, vacíe el formulario y muestre una Snackbar con "Deshacer" que lo restaure.

---

## 7. Autoevaluación

1. ¿Cuándo es adecuado usar un diálogo y cuándo es mejor una Snackbar?
2. ¿Qué tres tipos de botones puede tener un diálogo de alerta? ¿Dónde aparece cada uno?
3. ¿Para qué sirve `setCancelable(false)`?
4. ¿Qué diferencia hay entre `setItems()`, `setSingleChoiceItems()` y `setMultiChoiceItems()`? ¿Qué ocurre si además se usa `setMessage()`?
5. ¿Cómo se añade un layout propio a un diálogo usando View Binding?
6. ¿Qué le ocurre a un diálogo creado directamente en la actividad al girar la pantalla? ¿Por qué?
7. ¿Qué método se sobrescribe en un `DialogFragment`? ¿Por qué termina con `create()` y no con `show()`?
8. ¿Por qué no se deben pasar datos a un `DialogFragment` por el constructor? ¿Cómo se pasan?
9. Explica los pasos de la solución con interfaz y `onAttach()`. ¿Por qué sigue funcionando tras un giro?
10. ¿Qué diferencias hay entre esa solución y la Fragment Result API?

---

## 📚 Para saber más (documentación oficial)

- [Diálogos](https://developer.android.com/develop/ui/views/components/dialogs?hl=es-419) · `AlertDialog`, listas, layouts propios y `DialogFragment`.
- [Cómo mostrar diálogos con DialogFragment](https://developer.android.com/guide/fragments/dialogs?hl=es-419) · Ciclo de vida de un `DialogFragment` y cómo mostrarlo.
- [Cómo comunicarse con fragmentos](https://developer.android.com/guide/fragments/communicate?hl=es-419) · La Fragment Result API.
- [Dialogs · Material Components for Android](https://github.com/material-components/material-components-android/blob/master/docs/components/Dialog.md) (en inglés) · `MaterialAlertDialogBuilder` y estilos.
- [Agrega selectores a tu app](https://developer.android.com/guide/topics/ui/controls/pickers?hl=es-419) · `DatePickerDialog` y `TimePickerDialog`.

---

⬅️ [3.3 Notificaciones](03-3-notificaciones.md) · 🏠 [Índice](../README.md) · ➡️ [4.1 ¿Cómo definimos un menú?](04-1-definir-menu.md)
