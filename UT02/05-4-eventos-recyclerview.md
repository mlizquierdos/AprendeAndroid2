# 5.4 Gestión de eventos en el RecyclerView

> **Unidad 2 · Aplicaciones Android. Interfaz de usuario**
> Módulo: Programación multimedia y dispositivos móviles (2º DAM)

## Objetivos

Al terminar este apartado deberías ser capaz de:

- Detectar el clic y la pulsación larga sobre un elemento de un `RecyclerView`.
- Obtener de forma segura la posición del elemento pulsado.
- Comunicar los eventos del adaptador a la actividad mediante *callbacks*: funciones lambda o interfaces.
- Gestionar eventos de vistas internas de una fila, como un botón de menú ⋮.
- Integrar en una lista lo visto en la unidad: Snackbars, diálogos, menús e Intents.

---

## 1. El punto de partida

Con el punto 5.3 tenemos una carta que se muestra y se desplaza. Pero **no responde** al tocarla. Y aquí hay una diferencia importante con `ListView`:

| | `ListView` (presentación de clase) | `RecyclerView` |
|---|---|---|
| Clic en un elemento | `listView.setOnItemClickListener { … }` ya incluido | **No existe** ese método |
| Quién programa el clic | Android | **Nosotros**, en el ViewHolder |

Esto no es un descuido: como cada fila puede tener botones, imágenes o casillas propias, `RecyclerView` deja que decidamos **qué vista** responde y **cómo**.

---

## 2. Dónde poner el *listener*

Las vistas de una fila están en el **ViewHolder**, así que el *listener* se asigna ahí. Hay dos sitios posibles:

| Dónde | Cuántas veces se ejecuta | Valoración |
|---|---|---|
| En `onBindViewHolder()` / `bind()` | **Cada vez** que la fila se rellena o se recicla | Funciona, pero crea un *listener* nuevo en cada reciclado. |
| En el bloque **`init`** del ViewHolder | **Una sola vez**, al crear la fila | ✅ **Recomendado**. |

```kotlin
class PizzaViewHolder(
    private val binding: ItemPizzaBinding
) : RecyclerView.ViewHolder(binding.root) {

    init {
        // Se ejecuta UNA vez, al crear la fila en onCreateViewHolder()
        binding.root.setOnClickListener {
            val posicion = bindingAdapterPosition
            if (posicion != RecyclerView.NO_POSITION) {
                Log.d("Carta", "Pulsada la posición $posicion")
            }
        }
    }

    fun bind(pizza: Pizza) { /* … igual que en 5.3 … */ }
}
```

### 2.1 La posición del elemento pulsado

Como la fila se recicla, **no sirve** guardar la posición al crearla: hoy muestra la pizza 0 y dentro de un momento la 23. La posición **actual** se consulta en el momento del clic con **`bindingAdapterPosition`**.

| Propiedad | Qué devuelve |
|---|---|
| `bindingAdapterPosition` | La posición del elemento en el adaptador **en este momento**. |
| `RecyclerView.NO_POSITION` (-1) | Valor que devuelve si la fila se está eliminando o aún no está colocada. **Hay que comprobarlo** siempre. |

> ⚠️ En tutoriales antiguos verás `adapterPosition` o `layoutPosition`. `adapterPosition` está **obsoleta**: usa `bindingAdapterPosition`.

### 2.2 El problema

El código anterior funciona, pero solo escribe en el Logcat. Lo que queremos es, por ejemplo, **mostrar una Snackbar**, **abrir otra actividad** o **actualizar el carrito**. Y todo eso pertenece a la **actividad**, no al ViewHolder:

- El ViewHolder no tiene acceso a `binding.root` de la actividad (para la Snackbar).
- No debe abrir actividades: entonces el mismo adaptador no se podría reutilizar en otra pantalla con otro comportamiento.

Necesitamos que el ViewHolder **avise** a la actividad de que se ha pulsado un elemento, y que sea la actividad quien decida qué hacer. Es decir, un ***callback***, igual que con los diálogos del punto 3.4.

```
  Usuario toca la fila
        │
        ▼
  ViewHolder ──avisa──► Adapter ──avisa──► Actividad
  (detecta el clic)     (traduce posición   (decide qué hacer:
                         → Pizza)            Snackbar, Intent…)
```

---

## 3. Gestión de eventos mediante *callbacks*

### 3.1 Opción 1: funciones lambda *(recomendada en Kotlin)*

Kotlin permite pasar **funciones como parámetros**. El adaptador recibe en su constructor "qué hacer al pulsar una pizza", con el tipo función `(Pizza) -> Unit`: una función que recibe una `Pizza` y no devuelve nada.

**ViewHolder:** recibe las funciones y las llama con la posición.

```kotlin
class PizzaViewHolder(
    private val binding: ItemPizzaBinding,
    alPulsar: (Int) -> Unit,              // qué hacer con un clic (recibe la posición)
    alMantener: (Int) -> Unit             // qué hacer con una pulsación larga
) : RecyclerView.ViewHolder(binding.root) {

    init {
        binding.root.setOnClickListener {
            val posicion = bindingAdapterPosition
            if (posicion != RecyclerView.NO_POSITION) alPulsar(posicion)
        }
        binding.root.setOnLongClickListener {
            val posicion = bindingAdapterPosition
            if (posicion != RecyclerView.NO_POSITION) alMantener(posicion)
            true      // true: el evento queda consumido (no se lanza también el clic)
        }
    }

    fun bind(pizza: Pizza) { /* … */ }
}
```

**Adapter:** recibe las funciones de la actividad y traduce la **posición** en una **pizza**.

```kotlin
class PizzaAdapter(
    private val pizzas: MutableList<Pizza>,
    private val onPizzaClick: (Pizza) -> Unit,
    private val onPizzaLongClick: (Int) -> Unit
) : RecyclerView.Adapter<PizzaViewHolder>() {

    override fun getItemCount() = pizzas.size

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): PizzaViewHolder {
        val binding = ItemPizzaBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        return PizzaViewHolder(
            binding,
            alPulsar = { posicion -> onPizzaClick(pizzas[posicion]) },
            alMantener = { posicion -> onPizzaLongClick(posicion) }
        )
    }

    override fun onBindViewHolder(holder: PizzaViewHolder, position: Int) {
        holder.bind(pizzas[position])
    }

    // eliminar() e insertar() del punto 5.3 …
}
```

**Actividad:** decide qué ocurre.

```kotlin
private lateinit var adaptador: PizzaAdapter

private fun configurarRecyclerView() {
    adaptador = PizzaAdapter(
        FuenteDatos.pizzas.toMutableList(),
        onPizzaClick = { pizza -> anadirAlCarrito(pizza) },
        onPizzaLongClick = { posicion -> eliminarPizza(posicion) }
    )
    binding.rvPizzas.layoutManager = LinearLayoutManager(this)
    binding.rvPizzas.adapter = adaptador
}

private fun anadirAlCarrito(pizza: Pizza) {
    Snackbar.make(binding.root, getString(R.string.pizza_anadida, pizza.nombre), Snackbar.LENGTH_SHORT).show()
}
```

> 📌 **Relación con Kotlin (UT01):** `(Pizza) -> Unit` es un **tipo función**. Los argumentos con nombre (`onPizzaClick = …`) hacen la llamada mucho más legible cuando hay varias lambdas.

> 💡 **¿Por qué aquí sí se puede pasar una lambda por el constructor, y en un `DialogFragment` (punto 3.4) no?** Porque el `DialogFragment` lo **recrea Android** tras un giro, con el constructor vacío, y la lambda se pierde. El adaptador, en cambio, lo crea **nuestro código** en cada `onCreate()`, así que siempre recibe sus lambdas.

### 3.2 Opción 2: una interfaz

Es la forma clásica, y la encontrarás en mucho código existente y en Java. Sigue el mismo esquema que la comunicación entre un diálogo y su actividad (punto 3.4).

```kotlin
class PizzaAdapter(
    private val pizzas: MutableList<Pizza>,
    private val listener: OnPizzaListener
) : RecyclerView.Adapter<PizzaViewHolder>() {

    /** Lo que debe implementar quien quiera enterarse de los eventos de la lista. */
    interface OnPizzaListener {
        fun onPizzaClick(pizza: Pizza)
        fun onPizzaLongClick(posicion: Int)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): PizzaViewHolder {
        val binding = ItemPizzaBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        return PizzaViewHolder(
            binding,
            alPulsar = { posicion -> listener.onPizzaClick(pizzas[posicion]) },
            alMantener = { posicion -> listener.onPizzaLongClick(posicion) }
        )
    }
    // …
}
```

```kotlin
class CartaActivity : BaseActivity(), PizzaAdapter.OnPizzaListener {

    private fun configurarRecyclerView() {
        adaptador = PizzaAdapter(FuenteDatos.pizzas.toMutableList(), this)
        // …
    }

    override fun onPizzaClick(pizza: Pizza) = anadirAlCarrito(pizza)
    override fun onPizzaLongClick(posicion: Int) = eliminarPizza(posicion)
}
```

### 3.3 ¿Cuál usar?

| | Lambdas | Interfaz |
|---|---|---|
| Código necesario | Menos: no hay que declarar nada | Hay que declarar la interfaz y que la actividad la implemente |
| Legibilidad | Muy clara con argumentos con nombre | Clara cuando hay muchos eventos distintos |
| Idiomático en | Kotlin | Java (y código Kotlin antiguo) |
| Patrón de diseño | **Observer** en ambos casos | **Observer** en ambos casos |

Las dos son correctas. En este módulo usaremos **lambdas**, pero conviene reconocer las dos.

---

## 4. Eventos en vistas internas de la fila

Una fila puede tener sus propios botones: un corazón para marcar favorita, una papelera, un botón ⋮… Cada uno se programa igual, en el `init` del ViewHolder, sobre **su** vista.

### 4.1 Un menú ⋮ en cada fila

En el punto 4.4 vimos que el menú contextual flotante (pulsación larga) no es evidente para el usuario. En las listas, lo habitual hoy es un **botón ⋮** en cada fila que abre un **`PopupMenu`**.

**1. Añadir el botón al layout de la fila** (`item_pizza.xml`):

```xml
<ImageButton
    android:id="@+id/btnOpciones"
    android:layout_width="48dp"
    android:layout_height="48dp"
    android:background="?attr/selectableItemBackgroundBorderless"
    android:src="@drawable/ic_more_vert"
    android:contentDescription="@string/mas_opciones"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintBottom_toBottomOf="parent" />
```

**2. Un menú** (`res/menu/menu_item_pizza.xml`) con las opciones *Ver ingredientes*, *Compartir* y *Eliminar*.

**3. En el ViewHolder**, el botón abre el `PopupMenu` y avisa a la actividad con la opción elegida:

```kotlin
class PizzaViewHolder(
    private val binding: ItemPizzaBinding,
    alPulsar: (Int) -> Unit,
    alElegirOpcion: (posicion: Int, idOpcion: Int) -> Unit
) : RecyclerView.ViewHolder(binding.root) {

    init {
        binding.root.setOnClickListener {
            val posicion = bindingAdapterPosition
            if (posicion != RecyclerView.NO_POSITION) alPulsar(posicion)
        }

        binding.btnOpciones.setOnClickListener { boton ->
            PopupMenu(boton.context, boton).apply {
                menuInflater.inflate(R.menu.menu_item_pizza, menu)
                setOnMenuItemClickListener { item ->
                    val posicion = bindingAdapterPosition
                    if (posicion != RecyclerView.NO_POSITION) alElegirOpcion(posicion, item.itemId)
                    true
                }
                show()
            }
        }
    }
    // …
}
```

**4. En la actividad**, se decide qué hacer con cada opción:

```kotlin
private fun alElegirOpcion(posicion: Int, idOpcion: Int) {
    val pizza = adaptador.obtener(posicion)
    when (idOpcion) {
        R.id.opc_ingredientes -> mostrarIngredientes(pizza)       // diálogo (punto 3.4)
        R.id.opc_compartir -> compartir(pizza)                    // Intent implícito (punto 4.4)
        R.id.opc_eliminar -> eliminarPizza(posicion)              // Snackbar con Deshacer (sección 5)
    }
}
```

```kotlin
// En PizzaAdapter
fun obtener(posicion: Int): Pizza = pizzas[posicion]
```

> 📌 Fíjate en que el ViewHolder **no sabe** qué significa cada opción: solo informa de **qué fila** y **qué opción**. Toda la lógica está en la actividad.

---

## 5. Ejemplo integrador: la carta de la *Pizzería DAM*

Este ejemplo reúne lo visto en **toda la unidad**: una lista con `RecyclerView` y View Binding, Snackbars con acción, un diálogo, el menú de la barra que se actualiza y un Intent implícito.

| Evento | Qué ocurre | Punto de la unidad |
|---|---|---|
| Clic en una pizza | Se añade al carrito, Snackbar y el título del menú *Carrito (n)* se actualiza. | 3.2 · 4.2 |
| ⋮ → *Ver ingredientes* | Diálogo con los ingredientes. | 3.4 |
| ⋮ → *Compartir* | Selector de apps para compartir la pizza. | 4.4 |
| ⋮ → *Eliminar* o pulsación larga | Se elimina de la carta, con Snackbar *Deshacer*. | 3.2 · 5.3 |

### 5.1 ViewHolder y adaptador completos

```kotlin
// PizzaViewHolder.kt
class PizzaViewHolder(
    private val binding: ItemPizzaBinding,
    alPulsar: (Int) -> Unit,
    alMantener: (Int) -> Unit,
    alElegirOpcion: (posicion: Int, idOpcion: Int) -> Unit
) : RecyclerView.ViewHolder(binding.root) {

    init {
        binding.root.setOnClickListener {
            conPosicion(alPulsar)
        }
        binding.root.setOnLongClickListener {
            conPosicion(alMantener)
            true
        }
        binding.btnOpciones.setOnClickListener { boton ->
            PopupMenu(boton.context, boton).apply {
                menuInflater.inflate(R.menu.menu_item_pizza, menu)
                setOnMenuItemClickListener { item ->
                    conPosicion { posicion -> alElegirOpcion(posicion, item.itemId) }
                    true
                }
                show()
            }
        }
    }

    /** Ejecuta la acción solo si la fila tiene una posición válida. */
    private fun conPosicion(accion: (Int) -> Unit) {
        val posicion = bindingAdapterPosition
        if (posicion != RecyclerView.NO_POSITION) accion(posicion)
    }

    fun bind(pizza: Pizza) {
        val contexto = binding.root.context
        binding.tvNombre.text = pizza.nombre
        binding.tvIngredientes.text = pizza.ingredientes
        binding.tvPrecio.text = contexto.getString(R.string.precio_euros, pizza.precio)
        binding.ivPizza.setImageResource(pizza.imagen)
        binding.tvVegetariana.isVisible = pizza.vegetariana
    }
}
```

```kotlin
// PizzaAdapter.kt
class PizzaAdapter(
    private val pizzas: MutableList<Pizza>,
    private val onPizzaClick: (Pizza) -> Unit,
    private val onPizzaLongClick: (Int) -> Unit,
    private val onOpcion: (posicion: Int, idOpcion: Int) -> Unit
) : RecyclerView.Adapter<PizzaViewHolder>() {

    override fun getItemCount() = pizzas.size

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): PizzaViewHolder {
        val binding = ItemPizzaBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        return PizzaViewHolder(
            binding,
            alPulsar = { posicion -> onPizzaClick(pizzas[posicion]) },
            alMantener = onPizzaLongClick,
            alElegirOpcion = onOpcion
        )
    }

    override fun onBindViewHolder(holder: PizzaViewHolder, position: Int) {
        holder.bind(pizzas[position])
    }

    fun obtener(posicion: Int): Pizza = pizzas[posicion]

    fun eliminar(posicion: Int): Pizza {
        val borrada = pizzas.removeAt(posicion)
        notifyItemRemoved(posicion)
        return borrada
    }

    fun insertar(posicion: Int, pizza: Pizza) {
        pizzas.add(posicion, pizza)
        notifyItemInserted(posicion)
    }
}
```

> 💡 La función auxiliar `conPosicion()` evita repetir tres veces la comprobación de `NO_POSITION`. Es un ejemplo de **función de orden superior**: recibe otra función como parámetro.

### 5.2 La actividad

```kotlin
class CartaActivity : BaseActivity() {

    private lateinit var binding: ActivityCartaBinding
    private lateinit var adaptador: PizzaAdapter
    private var pizzasEnCarrito = 0

    override val opcionMenuActual = R.id.action_carta       // punto 4.3

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        binding = ActivityCartaBinding.inflate(layoutInflater)
        setContentView(binding.root)
        ViewCompat.setOnApplyWindowInsetsListener(binding.main) { v, insets ->
            val barras = insets.getInsets(WindowInsetsCompat.Type.systemBars())
            v.setPadding(barras.left, barras.top, barras.right, barras.bottom)
            insets
        }
        configurarToolbar(binding.toolbar.root)
        configurarRecyclerView()
    }

    private fun configurarRecyclerView() {
        adaptador = PizzaAdapter(
            FuenteDatos.pizzas.toMutableList(),
            onPizzaClick = ::anadirAlCarrito,                 // referencia a función
            onPizzaLongClick = ::eliminarPizza,
            onOpcion = ::alElegirOpcion
        )
        with(binding.rvPizzas) {
            layoutManager = LinearLayoutManager(this@CartaActivity)
            adapter = adaptador
            setHasFixedSize(true)
        }
    }

    // ── Clic: añadir al carrito ──────────────────────────────────────────
    private fun anadirAlCarrito(pizza: Pizza) {
        pizzasEnCarrito++
        invalidateOptionsMenu()                                // actualiza "Carrito (n)"
        Snackbar.make(binding.root, getString(R.string.pizza_anadida, pizza.nombre), Snackbar.LENGTH_SHORT)
            .show()
    }

    override fun onPrepareOptionsMenu(menu: Menu): Boolean {
        menu.findItem(R.id.action_carrito)?.title =
            getString(R.string.menu_carrito_cantidad, pizzasEnCarrito)
        return super.onPrepareOptionsMenu(menu)
    }

    // ── Pulsación larga o ⋮ → Eliminar: borrar con Deshacer ──────────────
    private fun eliminarPizza(posicion: Int) {
        val borrada = adaptador.eliminar(posicion)
        Snackbar.make(binding.root, getString(R.string.pizza_eliminada, borrada.nombre), Snackbar.LENGTH_LONG)
            .setAction(R.string.deshacer) { adaptador.insertar(posicion, borrada) }
            .show()
    }

    // ── Menú ⋮ de cada fila ──────────────────────────────────────────────
    private fun alElegirOpcion(posicion: Int, idOpcion: Int) {
        val pizza = adaptador.obtener(posicion)
        when (idOpcion) {
            R.id.opc_ingredientes -> mostrarIngredientes(pizza)
            R.id.opc_compartir -> compartir(pizza)
            R.id.opc_eliminar -> eliminarPizza(posicion)
        }
    }

    private fun mostrarIngredientes(pizza: Pizza) {
        MaterialAlertDialogBuilder(this)
            .setTitle(pizza.nombre)
            .setMessage(pizza.ingredientes)
            .setPositiveButton(R.string.aceptar, null)
            .show()
    }

    private fun compartir(pizza: Pizza) {
        val intent = Intent(Intent.ACTION_SEND).apply {
            type = "text/plain"
            putExtra(Intent.EXTRA_TEXT, getString(R.string.mira_esta_pizza, pizza.nombre))
        }
        try {
            startActivity(Intent.createChooser(intent, getString(R.string.compartir_con)))
        } catch (e: ActivityNotFoundException) {
            Snackbar.make(binding.root, R.string.sin_app_para_accion, Snackbar.LENGTH_LONG).show()
        }
    }
}
```

Textos que usa (`strings.xml`):

```xml
<string name="pizza_anadida">%1$s añadida al carrito</string>
<string name="pizza_eliminada">Pizza %1$s eliminada</string>
<string name="menu_carrito_cantidad">Carrito (%1$d)</string>
<string name="mira_esta_pizza">¡Mira esta pizza de la Pizzería DAM: %1$s!</string>
<string name="compartir_con">Compartir con…</string>
<string name="sin_app_para_accion">No hay ninguna app para hacer esto</string>
<string name="mas_opciones">Más opciones</string>
<string name="deshacer">Deshacer</string>
<string name="aceptar">Aceptar</string>
```

**Aspectos a destacar:**

- **`::anadirAlCarrito`** es una **referencia a función**: en lugar de escribir `{ pizza -> anadirAlCarrito(pizza) }`, se pasa la función directamente. Vale porque su firma coincide con el tipo esperado `(Pizza) -> Unit`.
- La actividad **no sabe nada de filas ni de ViewHolders**: solo recibe pizzas y posiciones. Y el adaptador **no sabe nada de Snackbars ni de menús**. Cada clase tiene su responsabilidad.
- El "Deshacer" funciona porque `eliminar()` devuelve la pizza borrada, que se guarda en la lambda de la acción (punto 3.2).

> 📌 **Cierre de la unidad:** con esta pantalla has usado layouts y vistas (punto 1), View Binding (punto 2), avisos y diálogos (punto 3), menús, Intents y navegación (punto 4) y listas dinámicas (punto 5). Son los cinco objetivos de la unidad.

---

## 6. Practica

1. En el proyecto de la carta, añade en el `init` del ViewHolder un `setOnClickListener` que escriba la posición en el Logcat. Comprueba que la posición es correcta después de desplazarte por la lista.
2. Cambia el adaptador para que reciba una lambda `onPizzaClick: (Pizza) -> Unit` y haz que la actividad muestre una Snackbar con el nombre de la pizza pulsada.
3. Añade la pulsación larga para eliminar una pizza con Snackbar *Deshacer*. Prueba a eliminar varias seguidas y deshacer la última.
4. Reescribe los *callbacks* con una **interfaz** `OnPizzaListener` implementada por la actividad. Compara cuántas líneas has escrito con cada opción y quédate con la que prefieras.
5. Añade el botón ⋮ a cada fila con un `PopupMenu` y sus tres opciones.
6. Haz que el título de la opción *Carrito* del menú de la barra muestre cuántas pizzas se han añadido.
7. Añade un `ImageButton` con un corazón para marcar una pizza como favorita. Pista: añade `var favorita: Boolean` al modelo, cambia el icono en `bind()` (¡siempre, en los dos casos!) y avisa al adaptador con `notifyItemChanged()`.
8. **Reto (integración con el punto 4.4):** haz que `MainActivity` abra la carta con `registerForActivityResult()`. Al pulsar una pizza, la carta debe devolver su nombre con `setResult()` y cerrarse, y `MainActivity` debe mostrarlo en el formulario.

---

## 7. Autoevaluación

1. ¿Por qué `RecyclerView` no tiene un método `setOnItemClickListener` como `ListView`?
2. ¿Dónde es preferible asignar el *listener* de una fila: en `onBindViewHolder()` o en el `init` del ViewHolder? ¿Por qué?
3. ¿Por qué no se puede guardar la posición de una fila al crearla? ¿Cómo se obtiene en el momento del clic?
4. ¿Qué es `RecyclerView.NO_POSITION` y por qué hay que comprobarlo?
5. ¿Por qué el ViewHolder no debe mostrar Snackbars ni abrir actividades?
6. ¿Qué significa el tipo `(Pizza) -> Unit`?
7. ¿Por qué en un adaptador se puede pasar una lambda por el constructor y en un `DialogFragment` no?
8. Explica cómo se gestionan los eventos con una interfaz. ¿Qué patrón de diseño sigue?
9. ¿Qué devuelve el *listener* de pulsación larga y qué significa ese valor?
10. ¿Cómo se añade un menú ⋮ a cada fila? ¿Qué información debe enviar el ViewHolder a la actividad?

---

## 📚 Para saber más (documentación oficial)

- [Cómo crear listas dinámicas con RecyclerView](https://developer.android.com/develop/ui/views/layout/recyclerview?hl=es-419) · Guía oficial completa.
- [Referencia de RecyclerView.ViewHolder](https://developer.android.com/reference/androidx/recyclerview/widget/RecyclerView.ViewHolder) (en inglés) · `bindingAdapterPosition` y el resto de propiedades.
- [Funciones de orden superior y lambdas (Kotlin)](https://kotlinlang.org/docs/lambdas.html) (en inglés) · Tipos función, lambdas y referencias a funciones.
- [Cómo agregar menús · Menú emergente](https://developer.android.com/guide/topics/ui/menus?hl=es-419) · `PopupMenu`.

---

⬅️ [5.3 Creación del adaptador](05-3-creacion-adaptador.md) · 🏠 [Índice](../README.md) · 🏁 Fin de la Unidad 2
