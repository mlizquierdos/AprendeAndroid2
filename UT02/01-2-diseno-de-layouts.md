# 1.2 Diseño de Layouts

> **Unidad 2 · Aplicaciones Android. Interfaz de usuario**
> Módulo: Programación multimedia y dispositivos móviles (2º DAM)

## Objetivos

Al terminar este apartado deberías ser capaz de:

- Distinguir `View` y `ViewGroup`, y explicar por qué conviene un árbol de vistas lo más plano posible.
- Usar correctamente los atributos de tamaño, margen, relleno, gravedad y visibilidad.
- Construir pantallas con `LinearLayout`, `FrameLayout` y, sobre todo, `ConstraintLayout`.
- Aprovechar las herramientas avanzadas de `ConstraintLayout`: sesgo, proporciones, cadenas, guías, barreras y grupos.
- Reutilizar layouts con `<include>` y `<merge>`, y crear variantes con cualificadores (`-land`, `-sw600dp`…).
- Elegir el layout más adecuado para cada situación.

---

## 1. Vistas y grupos de vistas

Toda interfaz Android se construye con dos tipos de elementos:

| Clase | Qué es | Ejemplos |
|-------|--------|----------|
| **`View`** | Elemento visible e interactivo que ocupa un área rectangular de la pantalla. | `TextView`, `Button`, `ImageView`, `EditText` |
| **`ViewGroup`** | Contenedor **invisible** que agrupa otras vistas y decide **cómo se colocan**. Hereda de `View`. | `ConstraintLayout`, `LinearLayout`, `FrameLayout` |

Los *layouts* son, por tanto, subclases de `ViewGroup`. Como un `ViewGroup` puede contener a otros `ViewGroup`, la interfaz forma un **árbol de vistas**:

```
ConstraintLayout                 ← ViewGroup raíz
 ├── TextView (título)           ← View
 ├── LinearLayout (horizontal)   ← ViewGroup anidado
 │    ├── Button (Cancelar)
 │    └── Button (Aceptar)
 └── ImageView (logo)
```

> 📌 **Relación con POO:** este es un ejemplo del patrón de diseño **Composite**: un `ViewGroup` *es una* `View` (herencia) y *contiene* `View`s (composición), lo que permite tratar igual a los elementos simples y a los contenedores.

### El coste del anidamiento

Para dibujar la pantalla, Android recorre el árbol **midiendo** y **posicionando** cada vista. Cuanto más profundo es el árbol, más trabajo cuesta cada fotograma. Por eso:

> ✅ **Diseña layouts lo más "planos" posible.** `ConstraintLayout` existe precisamente para conseguir diseños complejos **sin anidar** contenedores.

---

## 2. Atributos comunes de diseño

### 2.1 Tamaño: `layout_width` y `layout_height`

Son **obligatorios** en todas las vistas.

| Valor | Significado |
|-------|-------------|
| `wrap_content` | La vista mide **lo justo para su contenido** (texto, imagen…). |
| `match_parent` | La vista ocupa **todo el espacio disponible** del padre en ese eje. |
| `120dp` | Tamaño **fijo**. |
| `0dp` | **Depende del layout padre**: en `ConstraintLayout` significa "lo que permitan las restricciones"; en `LinearLayout` se usa junto con `layout_weight`. |

> ⚠️ En `ConstraintLayout` **no se recomienda `match_parent`** en las vistas hijas. Usa `0dp` con restricciones a ambos lados del padre.

### 2.2 Margin frente a padding

```
┌───────────── padre ─────────────┐
│        margin (exterior)        │
│   ┌─────── vista ───────────┐   │
│   │   padding (interior)    │   │
│   │   ┌── contenido ──┐     │   │
│   │   │  Texto        │     │   │
│   │   └───────────────┘     │   │
│   └─────────────────────────┘   │
└─────────────────────────────────┘
```

| Atributo | Espacio | Variantes |
|----------|---------|-----------|
| `android:layout_margin` | **Fuera** de la vista: la separa de otras vistas o del padre. | `layout_marginStart`, `layout_marginEnd`, `layout_marginTop`, `layout_marginBottom`, `layout_marginHorizontal`, `layout_marginVertical` |
| `android:padding` | **Dentro** de la vista: separa el contenido de su propio borde. | `paddingStart`, `paddingEnd`, `paddingTop`, `paddingBottom`, `paddingHorizontal`, `paddingVertical` |

> 💡 **Truco:** si la vista tiene un color de fondo, el **padding queda pintado** con ese color; el **margin no**.

### 2.3 `gravity` frente a `layout_gravity`

Se confunden con frecuencia:

| Atributo | Se aplica a… | Pregunta que responde |
|----------|--------------|------------------------|
| `android:gravity` | El **contenido** de la vista. | ¿Dónde coloco el texto **dentro** de este botón? |
| `android:layout_gravity` | La **propia vista** dentro de su padre. | ¿Dónde coloco **este botón** dentro de su contenedor? |

Valores posibles: `start`, `end`, `top`, `bottom`, `center`, `center_horizontal`, `center_vertical`… combinables con `|`, por ejemplo `top|end`.

> 📝 `layout_gravity` solo funciona en padres que lo soportan, como `LinearLayout` y `FrameLayout`. En `ConstraintLayout` la posición se controla con restricciones.

### 2.4 Visibilidad: `android:visibility`

| Valor | ¿Se ve? | ¿Ocupa espacio? |
|-------|---------|-----------------|
| `visible` | Sí | Sí |
| `invisible` | No | **Sí** (deja un hueco) |
| `gone` | No | **No** (el resto de vistas se recolocan) |

Desde Kotlin: `vista.visibility = View.GONE`, o con las extensiones de AndroidX `vista.isVisible = false`.

---

## 3. LinearLayout

Coloca sus hijos **uno detrás de otro**, en **una sola fila** (horizontal) o **una sola columna** (vertical).

```xml
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/titulo" />

    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="@string/hint_nombre" />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="end"
        android:text="@string/enviar" />

</LinearLayout>
```

### 3.1 `android:orientation`

- `vertical`: los hijos se apilan de arriba abajo.
- `horizontal` (valor por defecto): los hijos se colocan de inicio a fin.

### 3.2 Repartir el espacio con `layout_weight`

`layout_weight` reparte el **espacio sobrante** entre los hijos de forma proporcional a su peso. Para que el reparto sea exacto, la dimensión en el eje de la orientación debe ser **`0dp`**:

```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal">

    <!-- Ocupa 1/3 del ancho -->
    <Button
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:text="@string/cancelar" />

    <!-- Ocupa 2/3 del ancho -->
    <Button
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="2"
        android:text="@string/aceptar" />

</LinearLayout>
```

| Pesos | Reparto |
|-------|---------|
| `1` y `1` | 50 % – 50 % |
| `1` y `2` | 33 % – 66 % |
| `1`, `1` y `2` | 25 % – 25 % – 50 % |

> ✅ **Cuándo usar LinearLayout:** listas sencillas de elementos en una dirección (un formulario básico, una fila de botones). Si empiezas a **anidar varios** `LinearLayout` para conseguir un diseño, probablemente debas usar `ConstraintLayout`.

---

## 4. FrameLayout

Apila sus hijos **unos encima de otros** en la esquina superior izquierda (o donde indique su `layout_gravity`). El último hijo declarado queda **por encima**.

```xml
<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="200dp">

    <ImageView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:scaleType="centerCrop"
        android:src="@drawable/portada"
        android:contentDescription="@string/desc_portada" />

    <!-- Texto superpuesto a la imagen, en la esquina inferior -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|start"
        android:layout_margin="16dp"
        android:background="#80000000"
        android:padding="8dp"
        android:text="@string/titulo_portada"
        android:textColor="@android:color/white" />

</FrameLayout>
```

> ✅ **Cuándo usar FrameLayout:** para **un único hijo** o para **superponer** elementos (una imagen con un texto encima, un indicador de carga sobre el contenido). También es el contenedor habitual donde se cargan **fragmentos**.

---

## 5. ConstraintLayout

Es el **layout recomendado** por Google y el que usan por defecto las plantillas de Android Studio. Permite crear diseños complejos y adaptables **sin anidar contenedores**, posicionando cada vista mediante **restricciones** (*constraints*) respecto a otras vistas o al padre.

### 5.1 Restricciones básicas

Cada vista necesita, como mínimo, **una restricción horizontal y una vertical**. La sintaxis es:

```
app:layout_constraint[LadoPropio]_to[LadoDestino]Of="@id/otraVista"   (o "parent")
```

| Lado | Atributos |
|------|-----------|
| Horizontales | `Start_toStartOf`, `Start_toEndOf`, `End_toEndOf`, `End_toStartOf` |
| Verticales | `Top_toTopOf`, `Top_toBottomOf`, `Bottom_toBottomOf`, `Bottom_toTopOf` |
| Línea base del texto | `Baseline_toBaselineOf` (alinea textos de distinto tamaño) |

Ejemplo: un botón situado **debajo** de un texto y alineado con su **inicio**:

```xml
<Button
    android:id="@+id/btnAceptar"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_marginTop="24dp"
    android:text="@string/aceptar"
    app:layout_constraintStart_toStartOf="@id/tvTitulo"
    app:layout_constraintTop_toBottomOf="@id/tvTitulo" />
```

> ⚠️ Los **márgenes** solo tienen efecto si existe una **restricción** en ese mismo lado.

### 5.2 Centrado y sesgo (*bias*)

Si una vista se restringe a **dos lados opuestos**, queda centrada entre ellos. El **sesgo** desplaza esa posición:

```xml
<!-- Restringida a ambos lados del padre, pero desplazada al 25 % horizontal -->
app:layout_constraintStart_toStartOf="parent"
app:layout_constraintEnd_toEndOf="parent"
app:layout_constraintHorizontal_bias="0.25"
```

Valores de `0.0` (todo al inicio/arriba) a `1.0` (todo al final/abajo). Por defecto, `0.5`.

### 5.3 Tamaño `0dp` (*match constraints*)

Con `0dp`, la vista **se estira** hasta ocupar el espacio entre sus restricciones. Es el equivalente a `match_parent` en `ConstraintLayout`:

```xml
<!-- Campo de texto que ocupa todo el ancho, con 16dp de margen a cada lado -->
<EditText
    android:id="@+id/etNombre"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:layout_marginHorizontal="16dp"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintTop_toTopOf="parent" />
```

### 5.4 Proporción (*ratio*)

Con una de las dimensiones a `0dp`, se puede fijar la **proporción** ancho:alto:

```xml
<!-- Imagen a todo el ancho con proporción 16:9 -->
<ImageView
    android:layout_width="0dp"
    android:layout_height="0dp"
    app:layout_constraintDimensionRatio="16:9"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintTop_toTopOf="parent" />
```

### 5.5 Cadenas (*chains*)

Una **cadena** es un grupo de vistas enlazadas **en ambas direcciones** entre sí. Permite distribuirlas de forma uniforme, como haría un `LinearLayout`, pero sin anidar.

Se crea seleccionando las vistas en el editor → clic derecho → *Chains → Create Horizontal Chain*. El estilo se define en la **primera vista** de la cadena (la cabeza):

```xml
app:layout_constraintHorizontal_chainStyle="spread"
```

| Estilo | Resultado |
|--------|-----------|
| `spread` (por defecto) | Espacio repartido por igual, también en los extremos. `\|  A  B  C  \|` |
| `spread_inside` | Extremos pegados a los bordes; espacio repartido entre las vistas. `\|A    B    C\|` |
| `packed` | Vistas juntas y centradas (se puede usar sesgo). `\|   ABC   \|` |
| *weighted* | Con vistas a `0dp` y `layout_constraintHorizontal_weight`, reparte el espacio como `layout_weight`. |

### 5.6 Elementos auxiliares (*Helpers*)

No se ven en la aplicación; solo ayudan a posicionar otras vistas.

**Guideline** — línea guía fija, en `dp` o en porcentaje:

```xml
<androidx.constraintlayout.widget.Guideline
    android:id="@+id/guiaVertical50"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    app:layout_constraintGuide_percent="0.5" />
```

**Barrier** — barrera que se coloca automáticamente en el borde **más extremo** de un grupo de vistas. Útil cuando no sabemos qué texto será más largo (por ejemplo, en otro idioma):

```xml
<androidx.constraintlayout.widget.Barrier
    android:id="@+id/barreraEtiquetas"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:barrierDirection="end"
    app:constraint_referenced_ids="tvEtiquetaNombre,tvEtiquetaEmail" />
```

Después, los campos se restringen a `@id/barreraEtiquetas` y nunca se solaparán con la etiqueta más larga.

**Group** — agrupa vistas para cambiar su **visibilidad a la vez**:

```xml
<androidx.constraintlayout.widget.Group
    android:id="@+id/grupoDireccion"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:visibility="gone"
    app:constraint_referenced_ids="tvCalle,etCalle,tvCiudad,etCiudad" />
```

### 5.7 Ejemplo completo: pantalla de inicio de sesión

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="24dp"
    tools:context=".LoginActivity">

    <!-- Logo: a todo el ancho, proporción 2:1 -->
    <ImageView
        android:id="@+id/ivLogo"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:contentDescription="@string/desc_logo"
        android:src="@mipmap/ic_launcher"
        app:layout_constraintDimensionRatio="2:1"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/tvTitulo"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:text="@string/titulo_login"
        android:textSize="24sp"
        android:textStyle="bold"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/ivLogo" />

    <EditText
        android:id="@+id/etUsuario"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="32dp"
        android:autofillHints="username"
        android:hint="@string/hint_usuario"
        android:inputType="text"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/tvTitulo" />

    <EditText
        android:id="@+id/etPassword"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:autofillHints="password"
        android:hint="@string/hint_password"
        android:inputType="textPassword"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/etUsuario" />

    <!-- Dos botones en una cadena "spread_inside", con el mismo ancho gracias a los pesos -->
    <Button
        android:id="@+id/btnRegistro"
        style="@style/Widget.Material3.Button.OutlinedButton"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="32dp"
        android:layout_marginEnd="8dp"
        android:text="@string/registro"
        app:layout_constraintEnd_toStartOf="@id/btnEntrar"
        app:layout_constraintHorizontal_chainStyle="spread_inside"
        app:layout_constraintHorizontal_weight="1"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/etPassword" />

    <Button
        android:id="@+id/btnEntrar"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="8dp"
        android:text="@string/entrar"
        app:layout_constraintBaseline_toBaselineOf="@id/btnRegistro"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_weight="1"
        app:layout_constraintStart_toEndOf="@id/btnRegistro" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

Fíjate en que **todas las vistas son hijas directas** del `ConstraintLayout`: el árbol tiene **un solo nivel** de profundidad.

---

## 6. Otros layouts: RelativeLayout, TableLayout y GridLayout

| Layout | Descripción | Situación actual |
|--------|-------------|------------------|
| **RelativeLayout** | Posiciona vistas respecto al padre o a otras vistas (`layout_below`, `layout_toEndOf`, `layout_alignParentBottom`…). | **Obsoleto en la práctica**: `ConstraintLayout` hace lo mismo con más potencia. Conviene conocerlo porque aparece en proyectos y tutoriales antiguos. |
| **TableLayout** | Organiza las vistas en filas (`TableRow`) y columnas, como una tabla HTML. | Uso puntual: formularios tabulares sencillos. |
| **GridLayout** | Rejilla de filas y columnas con posibilidad de que una celda ocupe varias (`layout_rowSpan`, `layout_columnSpan`). | Uso puntual: teclados numéricos, calculadoras. |

> 📝 Para mostrar **listas o cuadrículas de datos** (que pueden ser largas o variables), **no** se usan estos layouts, sino **`RecyclerView`**, que veremos en el punto 5.

---

## 7. Contenido desplazable: ScrollView

Si el contenido no cabe en pantalla (formularios largos, pantallas en horizontal), hay que envolverlo en un `ScrollView` (desplazamiento vertical) o `HorizontalScrollView`.

Reglas:

- Un `ScrollView` admite **un único hijo directo**, que suele ser un layout con el resto de vistas.
- El hijo debe tener `layout_height="wrap_content"` (es lo que se desplaza).

```xml
<ScrollView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp">

        <!-- ... muchos campos de formulario ... -->

    </LinearLayout>
</ScrollView>
```

> ⚠️ **Nunca** metas un `RecyclerView` dentro de un `ScrollView`: ambos intentan gestionar el desplazamiento y el rendimiento se resiente. Si necesitas combinarlos, usa `NestedScrollView`.

---

## 8. Reutilizar layouts: include y merge

Igual que en programación extraemos código repetido a funciones, podemos extraer partes de la interfaz a **layouts independientes** y reutilizarlas.

**`res/layout/barra_botones.xml`**

```xml
<merge xmlns:android="http://schemas.android.com/apk/res/android">
    <Button
        android:id="@+id/btnCancelar"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:text="@string/cancelar" />
    <Button
        android:id="@+id/btnGuardar"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:text="@string/guardar" />
</merge>
```

**Uso en otro layout:**

```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal">

    <include layout="@layout/barra_botones" />

</LinearLayout>
```

- `<include>` inserta el contenido de otro layout.
- `<merge>` como raíz del layout incluido evita añadir un contenedor extra: sus hijos se integran **directamente** en el padre (en el ejemplo, en el `LinearLayout` horizontal). Así no aumentamos el anidamiento.

---

## 9. Recursos al servicio del diseño

Un buen layout **no contiene valores "a fuego"**. Textos, colores y medidas se definen como **recursos** en `res/values/`:

| Fichero | Contenido | Referencia en XML | En Kotlin |
|---------|-----------|-------------------|-----------|
| `strings.xml` | Textos | `@string/aceptar` | `getString(R.string.aceptar)` |
| `colors.xml` | Colores | `@color/azul_primario` | `getColor(R.color.azul_primario)` |
| `dimens.xml` | Medidas | `@dimen/margen_estandar` | `resources.getDimension(R.dimen.margen_estandar)` |
| `themes.xml` | Tema de la app | `@style/Theme.EjemplosUI` | — |

```xml
<!-- res/values/dimens.xml -->
<resources>
    <dimen name="margen_estandar">16dp</dimen>
    <dimen name="texto_titulo">24sp</dimen>
</resources>
```

```xml
android:layout_margin="@dimen/margen_estandar"
android:textSize="@dimen/texto_titulo"
```

> ✅ Si mañana el cliente pide márgenes de 20dp en toda la app, **solo cambias una línea**.

---

## 10. Layouts alternativos

Android elige automáticamente el recurso más adecuado según la **configuración** del dispositivo, gracias a los **cualificadores** en el nombre de la carpeta:

```
res/
├── layout/              ← diseño por defecto (vertical, móviles)
│   └── activity_main.xml
├── layout-land/         ← orientación horizontal
│   └── activity_main.xml
├── layout-sw600dp/      ← pantallas con ancho mínimo de 600dp (tablets)
│   └── activity_main.xml
├── values/strings.xml       ← textos por defecto
└── values-en/strings.xml    ← textos en inglés
```

Los ficheros alternativos deben tener **el mismo nombre** y contener **las vistas con los mismos ids**, para que el código Kotlin funcione igual con cualquiera de ellos.

Para crear una variante: en el editor, botón de orientación de la barra superior → **Create Landscape Qualifier**, o clic derecho en `res/layout` → *New → Layout Resource File* → añadir cualificador.

> 📝 **Recuerda (UT01 · ciclo de vida):** al girar el dispositivo, la actividad se **destruye y se recrea**. Precisamente en ese momento Android carga el layout correspondiente a la nueva orientación.

---

## 11. ¿Qué layout elijo?

```
¿Contiene un único hijo o elementos superpuestos?
 └─ Sí → FrameLayout
 └─ No → ¿Es una simple fila o columna, sin anidar?
          └─ Sí → LinearLayout
          └─ No → ConstraintLayout
¿Es una lista de datos de longitud variable?
 └─ RecyclerView (punto 5)
¿El contenido puede no caber en pantalla?
 └─ Envolverlo en ScrollView
```

| Situación | Layout recomendado |
|-----------|--------------------|
| Pantalla completa con varios elementos | `ConstraintLayout` |
| Fila de botones o formulario muy sencillo | `LinearLayout` |
| Imagen con texto superpuesto / contenedor de fragmento | `FrameLayout` |
| Lista o cuadrícula de datos | `RecyclerView` |
| Contenido más largo que la pantalla | `ScrollView` + layout interior |

---

## 12. Practica

Vas a diseñar **la misma pantalla de dos maneras** en un proyecto llamado `FichaContacto`: una ficha con una imagen de perfil de `120dp × 120dp` centrada arriba, el nombre en grande (`22sp`, negrita), dos filas de etiqueta y valor ("Teléfono" y "Email") y una fila final con tres botones del **mismo ancho**: "Llamar", "Mensaje" y "Email".

1. En `activity_main_linear.xml`, construye la ficha usando **solo `LinearLayout`** (puedes anidarlos). Reparte el ancho de los botones con `layout_weight`.
2. En `activity_main.xml`, construye la misma ficha con **un único `ConstraintLayout`, sin anidar** ningún contenedor. Alinea los valores con una **`Barrier`** y reparte los botones con una **cadena con pesos**.
3. Pasa todos los textos a `strings.xml` y todos los márgenes a `dimens.xml`.
4. Crea una variante **horizontal** (`layout-land`) de la versión con `ConstraintLayout`: imagen a la izquierda y datos a la derecha, separados por una `Guideline` al 35 %.
5. Oculta la fila del teléfono primero con `invisible` y después con `gone`. ¿Qué diferencia observas?
6. Cuenta en el árbol de componentes los **niveles de anidamiento** de cada versión.
7. **Reto:** sustituye el texto "Teléfono" por uno muy largo, como si tradujeras la app a otro idioma. ¿Qué versión mantiene los valores alineados? ¿Por qué?

---

## 13. Autoevaluación

1. ¿Qué diferencia hay entre `View` y `ViewGroup`? ¿Qué patrón de diseño ilustra la relación entre ambos?
2. ¿Qué significa `0dp` en un `ConstraintLayout`? ¿Y en un `LinearLayout`?
3. ¿Qué diferencia hay entre *margin* y *padding*? ¿Cuál de los dos queda pintado con el color de fondo de la vista?
4. ¿Qué diferencia hay entre `gravity` y `layout_gravity`?
5. ¿Qué diferencia hay entre `invisible` y `gone`?
6. Dos botones de un `LinearLayout` horizontal tienen pesos `1` y `3`. ¿Qué porcentaje del ancho ocupa cada uno? ¿Qué valor debe tener su `layout_width`?
7. ¿Cuántas restricciones necesita como mínimo una vista dentro de un `ConstraintLayout`?
8. ¿Qué diferencia hay entre una `Guideline` y una `Barrier`? Pon un ejemplo de uso de cada una.
9. ¿Qué estilos de cadena existen y cómo distribuye las vistas cada uno?
10. ¿Para qué sirve la carpeta `res/layout-land/`? ¿Qué deben cumplir los ficheros que contiene para que el código Kotlin siga funcionando?

---

## 📚 Para saber más (documentación oficial)

- [Diseños en Views](https://developer.android.com/guide/topics/ui/declaring-layout?hl=es-419) · `View`, `ViewGroup`, atributos de diseño y tipos de layout.
- [Cómo compilar una IU responsiva con ConstraintLayout](https://developer.android.com/training/constraint-layout?hl=es-419) · Restricciones, sesgo, cadenas, guías y barreras con ejemplos.
- [Diseño responsivo/adaptable con vistas](https://developer.android.com/develop/ui/views/layout/responsive-adaptive-design-with-views?hl=es-419) · Cómo adaptar los layouts a distintos tamaños y orientaciones de pantalla.
- [Cómo crear listas dinámicas con RecyclerView](https://developer.android.com/develop/ui/views/layout/recyclerview?hl=es-419) · Avance del punto 5: la alternativa correcta para mostrar listas de datos.

---

⬅️ [1.1 El editor de Layouts](01-1-editor-de-layouts.md) · 🏠 [Índice](../README.md) · ➡️ [1.3 Elementos de la interfaz](01-3-elementos-de-la-interfaz.md)
