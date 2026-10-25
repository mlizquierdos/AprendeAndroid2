# 1.3 Elementos de la interfaz

> **Unidad 2 · Aplicaciones Android. Interfaz de usuario**
> Módulo: Programación multimedia y dispositivos móviles (2º DAM)

## Objetivos

Al terminar este apartado deberías ser capaz de:

- Obtener desde Kotlin una referencia a las vistas del layout con `findViewById()`.
- Gestionar eventos con *listeners*, escritos como lambda, como objeto anónimo o implementando la interfaz.
- Usar los controles principales: `TextView`, `EditText`/`TextInputLayout`, `Button`, `ImageView`, `CheckBox`, `RadioGroup`, `MaterialSwitch`, `SeekBar`, `ProgressBar` y `Spinner`.
- Leer y validar los datos que introduce el usuario.
- Aplicar estilos propios y las pautas básicas de accesibilidad.

---

## 1. Del XML al código: acceder a las vistas

En los puntos anteriores hemos diseñado la **parte visual**. Para que la aplicación **responda** al usuario, el código Kotlin necesita una referencia a cada vista. La forma clásica es `findViewById()`:

```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Se busca la vista por su id, indicando su tipo
        val tvSaludo = findViewById<TextView>(R.id.tvSaludo)
        val btnSaludar = findViewById<Button>(R.id.btnSaludar)

        btnSaludar.setOnClickListener {
            tvSaludo.text = getString(R.string.saludo_pulsado)
        }
    }
}
```

> ⚠️ `findViewById()` solo funciona **después** de `setContentView()`: antes, el layout aún no se ha cargado y devolvería una excepción.

> 📝 **Avance del punto 2:** `findViewById()` tiene inconvenientes (hay que repetirlo para cada vista y no comprueba en compilación que el id exista en ese layout). En el punto 2 lo sustituiremos por **View Binding**. En este punto lo usamos porque es la base sobre la que se construye View Binding y porque lo encontrarás en mucho código existente.

---

## 2. Gestión de eventos

Android sigue un modelo de **programación orientada a eventos**: la app no "pregunta" constantemente si el usuario ha pulsado un botón, sino que **registra un escuchador** (*listener*) que Android invoca cuando ocurre el evento.

> 📌 **Relación con POO:** es el patrón **Observer**. La vista es el sujeto observable y el *listener* es el observador.

### 2.1 Tres formas de escribir un listener en Kotlin

Partimos de la interfaz `View.OnClickListener`, que tiene un único método `onClick(v: View)`.

**a) Lambda (forma recomendada)** — Kotlin permite usar una lambda con interfaces de un solo método (conversión SAM):

```kotlin
btnSaludar.setOnClickListener {
    tvSaludo.text = "¡Hola!"
}
```

**b) Objeto anónimo** — necesario cuando la interfaz tiene **varios métodos** (lo veremos con `SeekBar` y `Spinner`):

```kotlin
btnSaludar.setOnClickListener(object : View.OnClickListener {
    override fun onClick(v: View?) {
        tvSaludo.text = "¡Hola!"
    }
})
```

**c) La actividad implementa la interfaz** — útil cuando muchas vistas comparten el mismo manejador:

```kotlin
class MainActivity : AppCompatActivity(), View.OnClickListener {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        findViewById<Button>(R.id.btnSumar).setOnClickListener(this)
        findViewById<Button>(R.id.btnRestar).setOnClickListener(this)
    }

    override fun onClick(v: View?) {
        when (v?.id) {
            R.id.btnSumar -> { /* ... */ }
            R.id.btnRestar -> { /* ... */ }
        }
    }
}
```

> ⚠️ **Evita el atributo XML `android:onClick="metodo"`.** Funciona, pero el nombre del método es un texto que el compilador no comprueba: si lo renombras, la app fallará en ejecución. Asigna siempre los listeners desde el código.

### 2.2 Listeners más habituales

| Evento | Método | Vistas |
|--------|--------|--------|
| Pulsación | `setOnClickListener` | Todas |
| Pulsación larga | `setOnLongClickListener` (devuelve `Boolean`) | Todas |
| Cambio de estado marcado | `setOnCheckedChangeListener` | `CheckBox`, `Switch`, `RadioGroup`, `ToggleButton` |
| Cambio de texto | `addTextChangedListener` / `doAfterTextChanged` | `EditText` |
| Cambio de progreso | `setOnSeekBarChangeListener` | `SeekBar` |
| Selección de elemento | `onItemSelectedListener` | `Spinner` |

---

## 3. TextView

Muestra texto **no editable** por el usuario.

```xml
<TextView
    android:id="@+id/tvTitulo"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/titulo"
    android:textSize="20sp"
    android:textStyle="bold"
    android:textColor="@color/azul_primario"
    android:maxLines="2"
    android:ellipsize="end" />
```

| Atributo | Función |
|----------|---------|
| `text` | Texto mostrado (mejor como recurso `@string/...`). |
| `textSize` | Tamaño en **sp**. |
| `textStyle` | `normal`, `bold`, `italic` o `bold\|italic`. |
| `textColor` | Color del texto. |
| `textAlignment` / `gravity` | Alineación del texto dentro de la vista. |
| `maxLines` + `ellipsize` | Limita las líneas y añade "…" si el texto no cabe. |
| `textAppearance` | Aplica un estilo tipográfico predefinido, p. ej. `?attr/textAppearanceHeadlineSmall`. |

**Desde Kotlin:**

```kotlin
tvTitulo.text = "Nuevo título"                       // Asignar texto
tvTitulo.text = getString(R.string.titulo)           // Desde recursos
val actual: String = tvTitulo.text.toString()        // Leer texto
tvTitulo.setTextColor(getColor(R.color.rojo_error))  // Cambiar color
```

**Textos con parámetros** — en lugar de concatenar cadenas, usa marcadores en `strings.xml`:

```xml
<string name="saludo_nombre">Hola, %1$s. Tienes %2$d mensajes.</string>
```

```kotlin
tvSaludo.text = getString(R.string.saludo_nombre, "Ana", 3)
// → "Hola, Ana. Tienes 3 mensajes."
```

---

## 4. EditText y TextInputLayout

### 4.1 EditText

Subclase de `TextView` que permite al usuario **introducir texto**.

```xml
<EditText
    android:id="@+id/etEmail"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:hint="@string/hint_email"
    android:inputType="textEmailAddress"
    android:autofillHints="emailAddress"
    android:imeOptions="actionNext" />
```

**`android:hint`** muestra un texto de ayuda que desaparece al escribir. **No uses `text` como ayuda**: el usuario tendría que borrarlo.

**`android:inputType`** es fundamental: adapta el **teclado** y el comportamiento del campo.

| Valor | Teclado / comportamiento |
|-------|--------------------------|
| `text` | Texto general |
| `textPersonName` | Nombres propios |
| `textCapSentences` | Mayúscula al inicio de cada frase |
| `textEmailAddress` | Teclado con `@` |
| `textPassword` | Oculta los caracteres |
| `number` | Solo dígitos |
| `numberDecimal` | Números con decimales |
| `numberSigned` | Números con signo |
| `phone` | Teclado telefónico |
| `textMultiLine` | Varias líneas |

Se pueden combinar: `android:inputType="numberDecimal|numberSigned"`.

**`android:imeOptions`** cambia el botón de acción del teclado: `actionNext` (pasa al siguiente campo), `actionDone` (cierra el teclado), `actionSearch`, `actionSend`.

**Desde Kotlin:**

```kotlin
val email = etEmail.text.toString().trim()   // Leer (trim elimina espacios sobrantes)

if (email.isEmpty()) {
    etEmail.error = getString(R.string.error_campo_obligatorio)  // Muestra un aviso en el campo
    etEmail.requestFocus()
}

// Convertir a número de forma segura (devuelve null si no es un número válido)
val edad: Int? = etEdad.text.toString().toIntOrNull()
```

**Reaccionar mientras el usuario escribe** (extensión de AndroidX `core-ktx`, incluida en los proyectos nuevos):

```kotlin
etNombre.doAfterTextChanged { texto ->
    btnEnviar.isEnabled = !texto.isNullOrBlank()  // Activa el botón solo si hay texto
}
```

### 4.2 TextInputLayout (Material Design)

La librería **Material Components** (incluida por defecto en las plantillas) ofrece campos de texto más completos: etiqueta flotante, contador de caracteres, icono para mostrar la contraseña, mensajes de error y de ayuda.

```xml
<com.google.android.material.textfield.TextInputLayout
    android:id="@+id/tilPassword"
    style="@style/Widget.Material3.TextInputLayout.OutlinedBox"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:hint="@string/hint_password"
    app:counterEnabled="true"
    app:counterMaxLength="20"
    app:endIconMode="password_toggle"
    app:helperText="@string/ayuda_password"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintTop_toTopOf="parent">

    <com.google.android.material.textfield.TextInputEditText
        android:id="@+id/etPassword"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:inputType="textPassword"
        android:maxLength="20" />

</com.google.android.material.textfield.TextInputLayout>
```

```kotlin
// El error se asigna al TextInputLayout, no al EditText
if (password.length < 8) {
    tilPassword.error = getString(R.string.error_password_corta)
} else {
    tilPassword.error = null   // Quita el error
}
```

> 📝 El `hint` se pone en el `TextInputLayout` (para que flote) y el `inputType` en el `TextInputEditText`.

---

## 5. Button e ImageButton

### 5.1 Button

```xml
<Button
    android:id="@+id/btnEnviar"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/enviar"
    app:icon="@drawable/ic_send" />
```

Con un tema **Material 3** (el de las plantillas actuales), cada `<Button>` se convierte automáticamente en un `MaterialButton`, lo que permite atributos como `app:icon` o `app:cornerRadius` y distintos estilos:

| Estilo | Uso |
|--------|-----|
| *(por defecto)* — relleno | Acción principal |
| `@style/Widget.Material3.Button.OutlinedButton` | Acción secundaria |
| `@style/Widget.Material3.Button.TextButton` | Acción poco destacada (p. ej. "Cancelar") |
| `@style/Widget.Material3.Button.TonalButton` | Intermedio entre relleno y contorno |

```kotlin
btnEnviar.setOnClickListener { enviarFormulario() }

btnEnviar.isEnabled = false   // Desactiva el botón (se muestra atenuado y no responde)
```

### 5.2 ImageButton

Botón que muestra **solo una imagen**. Como no tiene texto, es **obligatorio** describirlo para accesibilidad:

```xml
<ImageButton
    android:id="@+id/ibFavorito"
    android:layout_width="48dp"
    android:layout_height="48dp"
    android:src="@drawable/ic_favorite_border"
    android:background="?attr/selectableItemBackgroundBorderless"
    android:contentDescription="@string/desc_favorito" />
```

> 💡 **Iconos:** clic derecho en `res/drawable` → *New → Vector Asset* → *Clip Art* da acceso a todos los iconos de Material Design como vectores (se ven nítidos en cualquier densidad).

---

## 6. ImageView

Muestra una imagen (de `res/drawable` o `res/mipmap`).

```xml
<ImageView
    android:id="@+id/ivFoto"
    android:layout_width="match_parent"
    android:layout_height="200dp"
    android:src="@drawable/paisaje"
    android:scaleType="centerCrop"
    android:contentDescription="@string/desc_paisaje" />
```

**`android:scaleType`** indica cómo se ajusta la imagen si su proporción no coincide con la de la vista:

| Valor | Comportamiento |
|-------|----------------|
| `fitCenter` (por defecto) | Imagen completa, centrada, sin deformar (puede dejar bandas vacías). |
| `centerCrop` | Rellena toda la vista sin deformar, **recortando** lo que sobra. Muy usado en fotos. |
| `centerInside` | Como `fitCenter`, pero nunca amplía imágenes pequeñas. |
| `fitXY` | Estira la imagen a la vista **deformándola**. Rara vez es buena idea. |
| `center` | Tamaño original, centrada, sin escalar. |

**Desde Kotlin:**

```kotlin
ivFoto.setImageResource(R.drawable.otra_foto)
```

> 📝 **Imágenes de Internet:** `ImageView` no descarga imágenes por sí solo. Para ello se usan librerías como **Coil** o **Glide**, que trataremos más adelante en el módulo.

---

## 7. CheckBox

Casilla de verificación para opciones **independientes** (se pueden marcar varias).

```xml
<CheckBox
    android:id="@+id/cbAceptoCondiciones"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/acepto_condiciones" />
```

```kotlin
// Consultar el estado
if (cbAceptoCondiciones.isChecked) { /* ... */ }

// Reaccionar al cambio
cbAceptoCondiciones.setOnCheckedChangeListener { _, isChecked ->
    btnRegistrar.isEnabled = isChecked
}
```

> 📝 El guion bajo `_` indica en Kotlin un parámetro de la lambda que **no vamos a usar** (aquí, la propia vista).

---

## 8. RadioButton y RadioGroup

Para opciones **mutuamente excluyentes** (solo se puede elegir una). Los `RadioButton` deben estar dentro de un `RadioGroup`, que se encarga de desmarcar los demás. `RadioGroup` hereda de `LinearLayout`, así que admite `orientation`.

```xml
<RadioGroup
    android:id="@+id/rgTamano"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    android:checkedButton="@id/rbMediana">

    <RadioButton
        android:id="@+id/rbPequena"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/pequena" />

    <RadioButton
        android:id="@+id/rbMediana"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/mediana" />

    <RadioButton
        android:id="@+id/rbFamiliar"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/familiar" />

</RadioGroup>
```

`android:checkedButton` marca una opción por defecto.

```kotlin
// Consultar la opción marcada (-1 si no hay ninguna)
val precioBase = when (rgTamano.checkedRadioButtonId) {
    R.id.rbPequena -> 8.0
    R.id.rbMediana -> 10.5
    R.id.rbFamiliar -> 14.0
    else -> 0.0
}

// Reaccionar al cambio de selección
rgTamano.setOnCheckedChangeListener { _, checkedId ->
    // checkedId es el id del RadioButton seleccionado
}
```

> ✅ `when` con los ids es la forma idiomática en Kotlin de tratar la opción elegida.

---

## 9. Switch y ToggleButton

Representan un estado **activado/desactivado**.

```xml
<com.google.android.material.materialswitch.MaterialSwitch
    android:id="@+id/swEnvio"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/envio_a_domicilio" />
```

```kotlin
swEnvio.setOnCheckedChangeListener { _, isChecked ->
    grupoDireccion.isVisible = isChecked   // Muestra u oculta los campos de dirección
}
```

| Componente | Observaciones |
|------------|---------------|
| `MaterialSwitch` | Interruptor de Material 3. **Recomendado.** |
| `Switch` / `SwitchCompat` | Versiones anteriores; siguen funcionando. |
| `ToggleButton` | Botón con texto distinto en cada estado (`textOn` / `textOff`). Hoy se usa poco. |

> 💡 **CheckBox o Switch:** un `Switch` aplica el cambio **inmediatamente** (activar el Wi-Fi); un `CheckBox` suele marcar una opción que se **confirma después** con un botón (aceptar condiciones de un formulario).

---

## 10. SeekBar y ProgressBar

### 10.1 SeekBar

Barra deslizante para elegir un valor numérico dentro de un rango.

```xml
<SeekBar
    android:id="@+id/sbCantidad"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:min="1"
    android:max="10"
    android:progress="1" />
```

Su listener tiene **tres métodos**, por lo que se implementa con un **objeto anónimo**:

```kotlin
sbCantidad.setOnSeekBarChangeListener(object : SeekBar.OnSeekBarChangeListener {
    override fun onProgressChanged(seekBar: SeekBar?, progress: Int, fromUser: Boolean) {
        tvCantidad.text = getString(R.string.cantidad_valor, progress)
    }
    override fun onStartTrackingTouch(seekBar: SeekBar?) { /* El usuario empieza a arrastrar */ }
    override fun onStopTrackingTouch(seekBar: SeekBar?) { /* El usuario suelta */ }
})
```

> 📝 `android:min` requiere API 26. Con una API mínima inferior, usa `max` y suma el desplazamiento en el código (`progress + 1`).
> Como alternativa moderna, Material ofrece `com.google.android.material.slider.Slider`.

### 10.2 ProgressBar

Indica que hay una tarea en curso.

```xml
<!-- Indeterminada: rueda girando, no se sabe cuánto falta -->
<ProgressBar
    android:id="@+id/pbCargando"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:visibility="gone" />

<!-- Determinada: barra horizontal con progreso conocido -->
<ProgressBar
    android:id="@+id/pbDescarga"
    style="?android:attr/progressBarStyleHorizontal"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:max="100"
    android:progress="40" />
```

```kotlin
pbCargando.isVisible = true   // Mostrar mientras se carga
pbDescarga.progress = 75      // Actualizar el progreso
```

---

## 11. Spinner

Lista desplegable para elegir **una opción** entre varias. Las opciones pueden venir de un recurso de tipo *array*:

```xml
<!-- res/values/strings.xml -->
<string-array name="masas">
    <item>Fina</item>
    <item>Clásica</item>
    <item>Rellena de queso</item>
</string-array>
```

**Opción A — directamente en XML** (sencillo, para listas fijas):

```xml
<Spinner
    android:id="@+id/spMasa"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:entries="@array/masas" />
```

**Opción B — mediante un adaptador** (necesario si los datos se generan en el código):

```kotlin
val masas = listOf("Fina", "Clásica", "Rellena de queso")
val adaptador = ArrayAdapter(this, android.R.layout.simple_spinner_item, masas)
adaptador.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item)
spMasa.adapter = adaptador
```

> 📌 **Adaptador:** objeto que actúa de **puente** entre los datos (una lista) y la vista que los muestra. Es el patrón **Adapter**, y será la pieza central cuando trabajemos con **RecyclerView** en el punto 5.

**Leer y reaccionar a la selección:**

```kotlin
val masaElegida = spMasa.selectedItem.toString()
val posicion = spMasa.selectedItemPosition

spMasa.onItemSelectedListener = object : AdapterView.OnItemSelectedListener {
    override fun onItemSelected(parent: AdapterView<*>?, view: View?, position: Int, id: Long) {
        // position: índice de la opción elegida
    }
    override fun onNothingSelected(parent: AdapterView<*>?) { }
}
```

> 📝 Material Design propone como alternativa visual un `TextInputLayout` con estilo *ExposedDropdownMenu* y un `AutoCompleteTextView`. `Spinner` sigue siendo perfectamente válido y más sencillo para empezar.

---

## 12. Estilos y temas

Cuando varias vistas comparten los mismos atributos, repetirlos es propenso a errores. Los **estilos** agrupan atributos con un nombre, igual que una clase CSS.

```xml
<!-- res/values/styles.xml (puedes crearlo; también vale themes.xml) -->
<resources>
    <style name="TextoEtiqueta">
        <item name="android:textSize">14sp</item>
        <item name="android:textStyle">bold</item>
        <item name="android:textColor">@color/gris_oscuro</item>
        <item name="android:layout_marginTop">16dp</item>
    </style>

    <!-- Herencia: toma todo de TextoEtiqueta y cambia el color -->
    <style name="TextoEtiqueta.Destacada">
        <item name="android:textColor">@color/azul_primario</item>
    </style>
</resources>
```

```xml
<TextView
    style="@style/TextoEtiqueta"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/etiqueta_nombre" />
```

| Concepto | Ámbito | Dónde se aplica |
|----------|--------|-----------------|
| **Estilo** | Una vista | `style="@style/..."` en la vista |
| **Tema** | Toda la app o una actividad | `android:theme` en `AndroidManifest.xml` |

El tema de la app se define en `res/values/themes.xml` (y `values-night/themes.xml` para el modo oscuro). Cambiando sus colores (`colorPrimary`, `colorSecondary`…) se actualiza el aspecto de **todos** los componentes Material de la aplicación a la vez.

> 📌 **Relación con POO:** los estilos admiten **herencia** (`TextoEtiqueta.Destacada` hereda de `TextoEtiqueta`), igual que las clases.

---

## 13. Accesibilidad

Una app profesional debe poder usarla **todo el mundo**, incluidas las personas que usan lectores de pantalla (TalkBack) o necesitan textos grandes.

| Pauta | Cómo cumplirla |
|-------|----------------|
| Describir los elementos gráficos | `android:contentDescription` en `ImageView` e `ImageButton`. Si la imagen es **decorativa**, usa `android:importantForAccessibility="no"`. |
| Áreas táctiles suficientes | Mínimo **48dp × 48dp** para elementos pulsables. |
| Texto escalable | Tamaños de texto siempre en **sp**. |
| Contraste suficiente | Texto legible sobre su fondo (el editor avisa de contrastes bajos). |
| Etiquetar los campos | `hint` en los `EditText`, o `android:labelFor` en la etiqueta que los acompaña. |

> 💡 Activa **TalkBack** en el emulador (*Ajustes → Accesibilidad*) y navega por tu app con los ojos cerrados. Es la mejor forma de detectar problemas.

---

## 14. Ejemplo integrador: Pizzería DAM

Aplicación que calcula el precio de un pedido combinando la mayoría de los componentes vistos.

### 14.1 Layout — `activity_main.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<ScrollView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="@dimen/margen_estandar">

        <TextView
            android:id="@+id/tvTitulo"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/titulo_pizzeria"
            android:textAppearance="?attr/textAppearanceHeadlineSmall"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

        <!-- Nombre del cliente -->
        <com.google.android.material.textfield.TextInputLayout
            android:id="@+id/tilNombre"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginTop="@dimen/margen_estandar"
            android:hint="@string/hint_nombre"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/tvTitulo">

            <com.google.android.material.textfield.TextInputEditText
                android:id="@+id/etNombre"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:inputType="textPersonName|textCapWords" />
        </com.google.android.material.textfield.TextInputLayout>

        <!-- Tamaño -->
        <TextView
            android:id="@+id/tvEtiquetaTamano"
            style="@style/TextoEtiqueta"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/etiqueta_tamano"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/tilNombre" />

        <RadioGroup
            android:id="@+id/rgTamano"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:checkedButton="@id/rbMediana"
            android:orientation="horizontal"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/tvEtiquetaTamano">

            <RadioButton
                android:id="@+id/rbPequena"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/pequena" />

            <RadioButton
                android:id="@+id/rbMediana"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/mediana" />

            <RadioButton
                android:id="@+id/rbFamiliar"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/familiar" />
        </RadioGroup>

        <!-- Masa -->
        <TextView
            android:id="@+id/tvEtiquetaMasa"
            style="@style/TextoEtiqueta"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/etiqueta_masa"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/rgTamano" />

        <Spinner
            android:id="@+id/spMasa"
            android:layout_width="wrap_content"
            android:layout_height="48dp"
            android:entries="@array/masas"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/tvEtiquetaMasa" />

        <!-- Ingredientes extra -->
        <TextView
            android:id="@+id/tvEtiquetaExtras"
            style="@style/TextoEtiqueta"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/etiqueta_extras"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/spMasa" />

        <CheckBox
            android:id="@+id/cbQueso"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/extra_queso"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/tvEtiquetaExtras" />

        <CheckBox
            android:id="@+id/cbChampinones"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/extra_champinones"
            app:layout_constraintStart_toEndOf="@id/cbQueso"
            app:layout_constraintTop_toTopOf="@id/cbQueso" />

        <CheckBox
            android:id="@+id/cbJamon"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/extra_jamon"
            app:layout_constraintStart_toEndOf="@id/cbChampinones"
            app:layout_constraintTop_toTopOf="@id/cbQueso" />

        <!-- Cantidad -->
        <TextView
            android:id="@+id/tvCantidad"
            style="@style/TextoEtiqueta"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/cbQueso"
            tools:text="Cantidad: 1" />

        <SeekBar
            android:id="@+id/sbCantidad"
            android:layout_width="0dp"
            android:layout_height="48dp"
            android:max="9"
            android:progress="0"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/tvCantidad" />

        <!-- Envío a domicilio -->
        <com.google.android.material.materialswitch.MaterialSwitch
            android:id="@+id/swEnvio"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginTop="@dimen/margen_estandar"
            android:text="@string/envio_a_domicilio"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/sbCantidad" />

        <Button
            android:id="@+id/btnCalcular"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginTop="@dimen/margen_estandar"
            android:text="@string/calcular"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/swEnvio" />

        <TextView
            android:id="@+id/tvResumen"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginTop="@dimen/margen_estandar"
            android:textAppearance="?attr/textAppearanceBodyLarge"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/btnCalcular"
            tools:text="Pedido de Ana: 2 pizzas medianas…" />

    </androidx.constraintlayout.widget.ConstraintLayout>
</ScrollView>
```

> 📝 El `SeekBar` usa `max="9"` y en el código se suma 1, para obtener valores de 1 a 10 sin depender de `android:min` (API 26).

### 14.2 Recursos — `strings.xml` (extracto)

```xml
<resources>
    <string name="app_name">Pizzería DAM</string>
    <string name="titulo_pizzeria">Haz tu pedido</string>
    <string name="hint_nombre">Nombre</string>
    <string name="etiqueta_tamano">Tamaño</string>
    <string name="pequena">Pequeña</string>
    <string name="mediana">Mediana</string>
    <string name="familiar">Familiar</string>
    <string name="etiqueta_masa">Masa</string>
    <string name="etiqueta_extras">Ingredientes extra (1,50 € c/u)</string>
    <string name="extra_queso">Queso</string>
    <string name="extra_champinones">Champiñones</string>
    <string name="extra_jamon">Jamón</string>
    <string name="cantidad_valor">Cantidad: %1$d</string>
    <string name="envio_a_domicilio">Envío a domicilio (+3 €)</string>
    <string name="calcular">Calcular precio</string>
    <string name="error_nombre">Introduce tu nombre</string>
    <string name="resumen_pedido">Pedido de %1$s:\n%2$d × pizza %3$s, masa %4$s\nExtras: %5$s\nTotal: %6$.2f €</string>
    <string name="sin_extras">ninguno</string>

    <string-array name="masas">
        <item>Fina</item>
        <item>Clásica</item>
        <item>Rellena de queso</item>
    </string-array>
</resources>
```

### 14.3 Recursos — `dimens.xml`, `colors.xml` y `styles.xml`

```xml
<!-- res/values/dimens.xml -->
<resources>
    <dimen name="margen_estandar">16dp</dimen>
</resources>
```

```xml
<!-- res/values/colors.xml (añadir a los existentes) -->
<color name="gris_oscuro">#FF424242</color>
<color name="azul_primario">#FF1565C0</color>
```

```xml
<!-- res/values/styles.xml -->
<resources>
    <style name="TextoEtiqueta">
        <item name="android:textSize">14sp</item>
        <item name="android:textStyle">bold</item>
        <item name="android:textColor">@color/gris_oscuro</item>
        <item name="android:layout_marginTop">16dp</item>
    </style>
</resources>
```

### 14.4 Lógica — `MainActivity.kt`

```kotlin
package com.dam.pizzeria

import android.os.Bundle
import android.widget.Button
import android.widget.CheckBox
import android.widget.RadioButton
import android.widget.RadioGroup
import android.widget.SeekBar
import android.widget.Spinner
import android.widget.TextView
import androidx.activity.enableEdgeToEdge
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.ViewCompat
import androidx.core.view.WindowInsetsCompat
import androidx.core.widget.doAfterTextChanged
import com.google.android.material.materialswitch.MaterialSwitch
import com.google.android.material.textfield.TextInputEditText
import com.google.android.material.textfield.TextInputLayout

class MainActivity : AppCompatActivity() {

    // Constantes de precios agrupadas en un companion object
    companion object {
        private const val PRECIO_PEQUENA = 8.0
        private const val PRECIO_MEDIANA = 10.5
        private const val PRECIO_FAMILIAR = 14.0
        private const val PRECIO_EXTRA = 1.5
        private const val PRECIO_ENVIO = 3.0
    }

    // Referencias a las vistas. "lateinit" porque se inicializan en onCreate,
    // después de setContentView (en el punto 2 las sustituiremos por View Binding)
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
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main)) { v, insets ->
            val systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars())
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom)
            insets
        }

        inicializarVistas()
        configurarListeners()
    }

    /** Obtiene las referencias a las vistas del layout. */
    private fun inicializarVistas() {
        tilNombre = findViewById(R.id.tilNombre)
        etNombre = findViewById(R.id.etNombre)
        rgTamano = findViewById(R.id.rgTamano)
        spMasa = findViewById(R.id.spMasa)
        cbQueso = findViewById(R.id.cbQueso)
        cbChampinones = findViewById(R.id.cbChampinones)
        cbJamon = findViewById(R.id.cbJamon)
        tvCantidad = findViewById(R.id.tvCantidad)
        sbCantidad = findViewById(R.id.sbCantidad)
        swEnvio = findViewById(R.id.swEnvio)
        tvResumen = findViewById(R.id.tvResumen)

        actualizarTextoCantidad(sbCantidad.progress)
    }

    /** Registra los escuchadores de eventos. */
    private fun configurarListeners() {
        // Quita el error en cuanto el usuario empieza a escribir
        etNombre.doAfterTextChanged { tilNombre.error = null }

        // Interfaz con tres métodos → objeto anónimo
        sbCantidad.setOnSeekBarChangeListener(object : SeekBar.OnSeekBarChangeListener {
            override fun onProgressChanged(seekBar: SeekBar?, progress: Int, fromUser: Boolean) {
                actualizarTextoCantidad(progress)
            }
            override fun onStartTrackingTouch(seekBar: SeekBar?) {}
            override fun onStopTrackingTouch(seekBar: SeekBar?) {}
        })

        // Interfaz con un único método → lambda
        findViewById<Button>(R.id.btnCalcular).setOnClickListener { calcularPedido() }
    }

    private fun actualizarTextoCantidad(progress: Int) {
        tvCantidad.text = getString(R.string.cantidad_valor, progress + 1)
    }

    /** Valida los datos, calcula el precio y muestra el resumen. */
    private fun calcularPedido() {
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
        val tamano = findViewById<RadioButton>(rgTamano.checkedRadioButtonId).text.toString()
        val masa = spMasa.selectedItem.toString()
        val cantidad = sbCantidad.progress + 1

        // Lista con los textos de los CheckBox marcados
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
```

**Aspectos a destacar del código:**

- La lógica está **separada en funciones** con una sola responsabilidad (`inicializarVistas`, `configurarListeners`, `calcularPedido`).
- Los precios son **constantes** en un `companion object`, no "números mágicos" repartidos por el código.
- Se usan funciones de colecciones de Kotlin (`filter`, `map`, `joinToString`) para tratar los `CheckBox` de forma compacta, en lugar de tres `if` consecutivos.
- La validación **corta la ejecución** con `return` antes de calcular nada.

> 📌 **Reflexión:** toda la lógica de negocio (precios, cálculo) está dentro de la `Activity`. Funciona, pero mezcla interfaz y lógica. Más adelante extraeremos esta lógica a una clase `Pedido` independiente, siguiendo el principio de separación de responsabilidades.

---

## 15. Practica

Crea una aplicación `InscripcionCursos`: un formulario de inscripción a un curso, con un `ConstraintLayout` dentro de un `ScrollView`.

1. Diseña el formulario con estos campos: nombre y apellidos y email (`TextInputLayout`), edad (`EditText` numérico), curso (`Spinner` con al menos 4 cursos desde un `string-array`), modalidad (`RadioGroup`: Presencial / Online / Semipresencial), turno (`RadioGroup`: Mañana / Tarde), material adicional (3 `CheckBox`: libro, kit de prácticas y certificado), horas semanales (`SeekBar` de 2 a 20), recibir novedades (`MaterialSwitch`), aceptar condiciones (`CheckBox`), un botón *Inscribirse* y un `TextView` para el resumen.
2. Haz que el botón *Inscribirse* solo esté **habilitado** cuando se marque *Aceptar condiciones*.
3. Muestra en un `TextView` el valor del `SeekBar` mientras se desliza. Recuerda que `android:min` requiere API 26 (apartado 10).
4. Al pulsar *Inscribirse*, **valida** los datos: nombre no vacío, email con `@` y edad entre 16 y 99. Muestra cada error en su propio campo con `error`.
5. Si todo es correcto, muestra en el `TextView` un **resumen** completo de la inscripción.
6. Si se elige la modalidad *Online*, oculta el grupo de turno con `gone`.
7. Pon todos los textos en `strings.xml`, añade `contentDescription` a las imágenes y aplica **un estilo propio** a las etiquetas. Activa **TalkBack** en el emulador y recorre el formulario.
8. **Reto:** añade un `ImageView` en la cabecera que cambie de imagen según el curso elegido en el `Spinner` (usa `onItemSelectedListener`).

---

## 16. Autoevaluación

1. ¿Por qué hay que llamar a `findViewById()` después de `setContentView()`?
2. ¿Qué patrón de diseño sigue el modelo de eventos de Android? Explica cómo funciona.
3. ¿Cuándo se puede escribir un *listener* como lambda y cuándo hace falta un objeto anónimo? Pon un ejemplo de cada caso.
4. ¿Por qué se desaconseja el atributo XML `android:onClick`?
5. ¿Para qué sirve `inputType`? Indica tres valores y su efecto.
6. ¿Cómo se sabe qué `RadioButton` de un `RadioGroup` está marcado? ¿Qué devuelve si no hay ninguno?
7. ¿Cuándo es más adecuado un `CheckBox` y cuándo un `Switch`?
8. ¿Qué es un adaptador y qué papel cumple en un `Spinner`?
9. ¿Qué diferencia hay entre un estilo y un tema? ¿Cómo se aplica cada uno?
10. Indica tres pautas de accesibilidad que debe cumplir una interfaz y cómo se aplican.

---

## 📚 Para saber más (documentación oficial)

- [Descripción general de los eventos de entrada](https://developer.android.com/guide/topics/ui/ui-events?hl=es-419) · *Listeners*, gestión de eventos y foco.
- [Cómo agregar spinners a tu app](https://developer.android.com/guide/topics/ui/controls/spinner?hl=es-419) · Opciones del `Spinner` y uso de adaptadores.
- [Estilos y temas](https://developer.android.com/guide/topics/ui/themes?hl=es-419) · Cómo crear estilos, la herencia entre estilos y el tema de la app.
- [Cómo hacer que las apps sean más accesibles](https://developer.android.com/guide/topics/ui/accessibility/apps?hl=es-419) · `contentDescription`, áreas táctiles, contraste y pruebas con TalkBack.
- [Primeros pasos con Material Components para Android](https://material.io/develop/android/docs/getting-started) (en inglés) · `TextInputLayout`, `MaterialButton`, `MaterialSwitch` y el resto de componentes de Material.
- [Vinculación de vistas](https://developer.android.com/topic/libraries/view-binding?hl=es-419) · Avance del punto 2: la alternativa moderna a `findViewById()`.

---

⬅️ [1.2 Diseño de Layouts](01-2-diseno-de-layouts.md) · 🏠 [Índice](../README.md) · ➡️ 2. View Binding *(próximamente)*
