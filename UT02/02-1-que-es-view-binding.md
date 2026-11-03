# 2.1 Qué es View Binding

> **Unidad 2 · Aplicaciones Android. Interfaz de usuario**
> Módulo: Programación multimedia y dispositivos móviles (2º DAM)

## Objetivos

Al terminar este apartado deberías ser capaz de:

- Explicar los problemas de acceder a las vistas con `findViewById()`.
- Describir qué es View Binding y qué genera a partir de cada layout.
- Deducir el nombre de la clase de vinculación y de sus propiedades a partir del fichero XML y de los ids.
- Enumerar las ventajas de View Binding: seguridad ante nulos y seguridad de tipos.
- Distinguir View Binding de otras alternativas: *Kotlin synthetics*, ButterKnife y Data Binding.

---

## 1. Punto de partida: `findViewById()`

En el punto 1.3 accedimos a las vistas del layout desde Kotlin con `findViewById()`. En la app *Pizzería DAM* tuvimos que escribir esto:

```kotlin
private lateinit var tilNombre: TextInputLayout
private lateinit var etNombre: TextInputEditText
private lateinit var rgTamano: RadioGroup
private lateinit var spMasa: Spinner
// ... y siete propiedades más

private fun inicializarVistas() {
    tilNombre = findViewById(R.id.tilNombre)
    etNombre = findViewById(R.id.etNombre)
    rgTamano = findViewById(R.id.rgTamano)
    spMasa = findViewById(R.id.spMasa)
    // ... y siete líneas más
}
```

Funciona, pero tiene varios inconvenientes:

| Problema | Qué ocurre |
|---|---|
| **Código repetitivo** | Por cada vista hay que declarar una propiedad y llamar a `findViewById()`. En una pantalla con 20 vistas son 40 líneas que no aportan lógica. |
| **El id puede no existir en ese layout** | La clase `R` es **común a toda la app**. Si usas el id de una vista de **otro** layout, el código compila, pero `findViewById()` devuelve `null` y la app falla en ejecución. |
| **El tipo puede no coincidir** | Si escribes `findViewById<Button>(R.id.tvTitulo)` y `tvTitulo` es un `TextView`, el código compila, pero la app falla con una `ClassCastException`. |
| **El orden importa** | Si se llama antes de `setContentView()`, no encuentra nada. |

Los tres últimos problemas tienen algo en común: **el compilador no puede detectarlos**. Solo aparecen al ejecutar la app, y a veces en una pantalla que no se ha probado.

---

## 2. Qué es View Binding

**View Binding** (vinculación de vistas) es una función de Android Jetpack que **genera automáticamente una clase por cada layout XML**. Esa clase, llamada **clase de vinculación**, contiene una propiedad por cada vista que tiene `android:id`, ya con el tipo correcto.

```
res/layout/activity_main.xml          →   ActivityMainBinding
├── ConstraintLayout  (id: main)            ├── root : ConstraintLayout
├── TextView          (id: tvTitulo)        ├── main : ConstraintLayout
├── EditText          (id: etNombre)        ├── tvTitulo : TextView
├── Button            (id: btnEnviar)       ├── etNombre : EditText
└── ImageView         (sin id)              └── btnEnviar : Button
                                            (la ImageView sin id no aparece)
```

Con View Binding, el código anterior se reduce a esto:

```kotlin
private lateinit var binding: ActivityMainBinding

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    binding = ActivityMainBinding.inflate(layoutInflater)
    setContentView(binding.root)

    binding.btnEnviar.setOnClickListener {
        binding.tvTitulo.text = binding.etNombre.text
    }
}
```

Ya no hace falta declarar una propiedad por vista ni llamar a `findViewById()`: **una sola propiedad, `binding`, da acceso a todas las vistas del layout**.

> 💡 **Idea clave:** View Binding no cambia nada del XML ni de cómo se dibuja la interfaz. Solo cambia **cómo llega el código Kotlin a las vistas**.

---

## 3. Reglas de nombrado

Android Studio genera los nombres a partir del nombre del fichero y de los ids, siguiendo dos reglas.

### 3.1 Nombre de la clase

El nombre del fichero se pasa a *PascalCase* (cada palabra empieza en mayúscula, sin guiones bajos) y se le añade `Binding`:

| Fichero de layout | Clase de vinculación |
|---|---|
| `activity_main.xml` | `ActivityMainBinding` |
| `activity_login.xml` | `ActivityLoginBinding` |
| `item_contacto.xml` | `ItemContactoBinding` |
| `barra_botones.xml` | `BarraBotonesBinding` |

Las clases se generan en el paquete `<paquete de la app>.databinding`, por ejemplo `com.dam.pizzeria.databinding.ActivityMainBinding`.

### 3.2 Nombre de las propiedades

Cada id se pasa a *camelCase*:

| Id en el XML | Propiedad en la clase de vinculación |
|---|---|
| `@+id/tvTitulo` | `tvTitulo` |
| `@+id/btn_enviar` | `btnEnviar` |
| `@+id/et_nombre_usuario` | `etNombreUsuario` |

> ✅ **Por eso, en el punto 1 usamos ids en camelCase con prefijo de tipo** (`tvTitulo`, `btnEnviar`, `etNombre`): el nombre que escribes en el XML es exactamente el que usarás en Kotlin.

### 3.3 La propiedad `root`

Toda clase de vinculación tiene además la propiedad **`root`**, que es la vista raíz del layout (normalmente el `ConstraintLayout` o `LinearLayout` exterior). Es la que se pasa a `setContentView()`.

---

## 4. Ventajas

| Ventaja | Por qué |
|---|---|
| **Seguridad ante nulos** (*null safety*) | La clase solo tiene propiedades para las vistas que **existen en ese layout**. Si intentas usar un id de otro layout, **no compila**. |
| **Seguridad de tipos** (*type safety*) | Cada propiedad tiene el tipo exacto de la vista del XML. Si `tvTitulo` es un `TextView`, `binding.tvTitulo` es un `TextView`: no hay *casts* que puedan fallar. |
| **Menos código** | Una propiedad `binding` en lugar de una propiedad y un `findViewById()` por vista. |
| **Sin cambios en el XML** | No hay que añadir etiquetas ni modificar los layouts existentes. |
| **Compilación rápida** | No procesa expresiones dentro del XML (a diferencia de Data Binding). |

Los errores que con `findViewById()` aparecían en ejecución pasan a detectarse **al compilar**, que es el mejor momento para encontrarlos.

### Un caso especial: propiedades anulables

Si una vista solo existe en **algunas variantes** de un layout (por ejemplo, está en `layout-land/activity_main.xml` pero no en `layout/activity_main.xml`), su propiedad se genera como **anulable** (`TextView?`). Kotlin te obliga entonces a comprobarla antes de usarla:

```kotlin
binding.tvDetalle?.text = "Solo visible en horizontal"
```

Así View Binding refleja que, según la orientación, esa vista puede no estar.

---

## 5. Un poco de historia: otras alternativas

View Binding no fue la primera solución al problema de `findViewById()`. Encontrarás las demás en proyectos y tutoriales antiguos:

| Alternativa | Cómo funcionaba | Situación actual |
|---|---|---|
| **ButterKnife** | Librería externa con anotaciones: `@BindView(R.id.tvTitulo) lateinit var tvTitulo: TextView` | **Abandonada**: su propio autor recomienda View Binding. |
| **Kotlin synthetics** (*Kotlin Android Extensions*) | Accedía a las vistas directamente por su id: `tvTitulo.text = "Hola"` | **Obsoleta** desde Kotlin 1.4.20 y **eliminada** en Kotlin 1.8. No compila en proyectos actuales. |
| **Data Binding** | Genera clases de vinculación y además permite escribir expresiones en el XML: `android:text="@{usuario.nombre}"` | **Vigente**, pero más compleja: hay que envolver el XML en `<layout>` y compila más lento. Útil con arquitecturas como MVVM. |
| **View Binding** | Genera una clase de vinculación por layout, sin tocar el XML | **Recomendada** para sustituir a `findViewById()`. |

> 📝 Si en un tutorial ves código como `import kotlinx.android.synthetic.main.activity_main.*` seguido de `tvTitulo.text = ...` **sin** `binding.` delante, son *Kotlin synthetics*: no te funcionará. Tradúcelo a View Binding.

### View Binding frente a Data Binding

| | View Binding | Data Binding |
|---|---|---|
| Acceso a las vistas desde Kotlin | ✅ | ✅ |
| Requiere cambiar el XML | ❌ | ✅ (etiqueta `<layout>`) |
| Expresiones y variables en el XML | ❌ | ✅ |
| Velocidad de compilación | Más rápida | Más lenta |
| Uso recomendado | Sustituir `findViewById()` | Interfaces que se actualizan solas a partir de datos |

En este módulo usaremos **View Binding**: resuelve el problema de `findViewById()` sin añadir complejidad.

---

## 6. Practica

1. Abre la app *Pizzería DAM* del punto 1.3 y cuenta cuántas líneas dedica la `MainActivity` a declarar vistas y a llamar a `findViewById()`.
2. Escribe en tu cuaderno el nombre de la clase de vinculación que se generaría para estos ficheros: `activity_main.xml`, `activity_detalle_producto.xml`, `item_mensaje.xml` y `dialog_confirmar.xml`.
3. Escribe el nombre de la propiedad que se generaría para estos ids: `tvNombre`, `btn_guardar`, `iv_foto_perfil` y `main`.
4. En un proyecto con `findViewById()`, cambia a propósito `findViewById<TextView>(R.id.tvSaludo)` por `findViewById<Button>(R.id.tvSaludo)`. ¿Compila? Ejecuta la app y busca en el Logcat la excepción.
5. Repite el experimento con un id que pertenezca a otro layout del proyecto. ¿Qué excepción aparece ahora?
6. **Reto:** busca en Internet un tutorial antiguo de Android que use *Kotlin synthetics* o ButterKnife e identifica las líneas que habría que cambiar para pasarlo a View Binding.

---

## 7. Autoevaluación

1. Indica tres inconvenientes de acceder a las vistas con `findViewById()`.
2. ¿Por qué el compilador no detecta que un id pertenece a otro layout cuando usamos `findViewById()`?
3. ¿Qué genera View Binding a partir de cada fichero de layout?
4. ¿Cómo se llamará la clase de vinculación de `fragment_lista_tareas.xml`?
5. ¿Qué propiedad tendrá en la clase de vinculación una vista con `android:id="@+id/btn_aceptar"`?
6. ¿Aparecen en la clase de vinculación las vistas sin `android:id`? ¿Por qué?
7. ¿Qué es la propiedad `root` y para qué se usa?
8. Explica con un ejemplo qué significan *seguridad ante nulos* y *seguridad de tipos*.
9. ¿En qué caso una propiedad de la clase de vinculación es anulable?
10. ¿Qué diferencias hay entre View Binding y Data Binding? ¿Por qué usaremos View Binding en este módulo?

---

## 📚 Para saber más (documentación oficial)

- [Vinculación de vistas](https://developer.android.com/topic/libraries/view-binding?hl=es-419) · Guía oficial: qué es, cómo se activa y cómo se usa.
- [Cómo migrar de Kotlin synthetics a la vinculación de vistas de Jetpack](https://developer.android.com/topic/libraries/view-binding/migration?hl=es-419) · Para traducir código antiguo a View Binding.
- [Biblioteca de vinculación de datos](https://developer.android.com/topic/libraries/data-binding?hl=es-419) · Data Binding, para compararlo con View Binding.

---

⬅️ [1.3 Elementos de la interfaz](01-3-elementos-de-la-interfaz.md) · 🏠 [Índice](../README.md) · ➡️ [2.2 Activación de View Binding](02-2-activacion-view-binding.md)
