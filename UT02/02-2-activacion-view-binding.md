# 2.2 Activación de View Binding

> **Unidad 2 · Aplicaciones Android. Interfaz de usuario**
> Módulo: Programación multimedia y dispositivos móviles (2º DAM)

## Objetivos

Al terminar este apartado deberías ser capaz de:

- Activar View Binding en el fichero de Gradle del módulo, en Kotlin DSL y en Groovy.
- Sincronizar el proyecto y comprobar que se han generado las clases de vinculación.
- Importar una clase de vinculación en el código Kotlin.
- Excluir un layout de la generación con `tools:viewBindingIgnore`.
- Resolver los errores más habituales al activar View Binding.

---

## 1. View Binding se activa por módulo

View Binding **no está activo por defecto** en los proyectos creados con la plantilla *Empty Views Activity*. Hay que activarlo de forma explícita, y se hace **por módulo**: en cada módulo donde quieras usarlo.

En nuestros proyectos solo hay un módulo, `app`, así que basta con activarlo una vez.

> 📝 **Recuerda (UT01 · estructura del proyecto):** un proyecto Android tiene **dos** ficheros `build.gradle(.kts)`. Uno es el del **proyecto** (raíz) y otro el del **módulo** (`app/build.gradle.kts`). View Binding se activa en el del **módulo**.

```
MiProyecto/
├── build.gradle.kts            ← del proyecto: aquí NO
├── settings.gradle.kts
└── app/
    └── build.gradle.kts        ← del módulo: aquí SÍ
```

En la vista **Android** del panel *Project*, aparecen dentro de *Gradle Scripts* como `build.gradle.kts (Project: MiProyecto)` y `build.gradle.kts (Module :app)`.

---

## 2. Activarlo en Gradle

Dentro del bloque `android { }` del fichero del módulo, se añade el bloque `buildFeatures` con la opción `viewBinding`.

### 2.1 Kotlin DSL (`build.gradle.kts`) · plantilla actual

Es el formato que usan los proyectos nuevos de Android Studio:

```kotlin
android {
    namespace = "com.dam.pizzeria"
    compileSdk = 36

    defaultConfig {
        // ...
    }

    // Activa la generación de clases de vinculación
    buildFeatures {
        viewBinding = true
    }

    // ...
}
```

### 2.2 Groovy (`build.gradle`) · proyectos antiguos

Si el proyecto es antiguo y el fichero se llama `build.gradle` (sin `.kts`), la sintaxis cambia ligeramente: no hay signo `=`.

```groovy
android {
    // ...
    buildFeatures {
        viewBinding true
    }
}
```

> ⚠️ Si el bloque `buildFeatures` ya existe (por ejemplo, con `compose = true` u otra opción), **añade** la línea `viewBinding = true` dentro de él. No crees un segundo bloque.

---

## 3. Sincronizar el proyecto

Al modificar un fichero de Gradle, Android Studio muestra arriba una barra amarilla:

> *Gradle files have changed since last project sync. A project sync may be necessary for the IDE to work properly.* **Sync Now**

Pulsa **Sync Now** (o el icono del elefante con la flecha 🐘 de la barra de herramientas, o *File → Sync Project with Gradle Files*).

Durante la sincronización, Gradle genera **una clase de vinculación por cada layout** de `res/layout/`. A partir de ese momento ya puedes usarlas en el código.

> 💡 Si después de sincronizar el editor sigue sin reconocer las clases, ejecuta **Build → Make Project** (`Ctrl + F9`). Fuerza la generación del código.

---

## 4. Comprobar que funciona

### 4.1 Desde el código

Abre la `MainActivity` y escribe `ActivityMainBinding`. Si View Binding está activo, Android Studio la reconoce y propone importarla (`Alt + Enter`):

```kotlin
import com.dam.pizzeria.databinding.ActivityMainBinding
```

El paquete es siempre `<namespace de la app>.databinding`. El `namespace` es el que aparece en `build.gradle.kts`.

### 4.2 Ver el código generado

Las clases se generan en la carpeta `build` del módulo, no en `src`. **Nunca se editan a mano**: se vuelven a generar cada vez que cambias el layout.

Para verlas, mantén pulsado `Ctrl` y haz clic sobre `ActivityMainBinding` en el código. Verás algo parecido a esto (simplificado):

```java
// Código GENERADO por Android Studio (en Java)
public final class ActivityMainBinding implements ViewBinding {
    private final ConstraintLayout rootView;
    public final ConstraintLayout main;
    public final TextView tvTitulo;
    public final EditText etNombre;
    public final Button btnEnviar;

    public static ActivityMainBinding inflate(LayoutInflater inflater) { ... }
    public static ActivityMainBinding bind(View rootView) { ... }

    @Override
    public ConstraintLayout getRoot() { return rootView; }
}
```

Fíjate en que:

- Hay una propiedad pública por cada vista con id, con su **tipo exacto**.
- Tiene dos formas de crearse: `inflate()` (crea las vistas a partir del XML) y `bind()` (las enlaza a partir de una vista ya creada). Las veremos en el punto 2.3.
- Está escrita en Java, pero desde Kotlin se usa con normalidad: `getRoot()` se convierte en la propiedad `root`.

> 📌 **Relación con lo visto en el punto 1:** internamente, `bind()` sigue usando `findViewById()`, pero **una sola vez** y en código generado y probado. Ya no lo escribimos nosotros, y por eso no nos podemos equivocar.

---

## 5. Excluir un layout

Si hay un layout para el que **no** quieres que se genere clase (por ejemplo, uno que no se usa desde el código), añade este atributo a su **vista raíz**:

```xml
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:viewBindingIgnore="true">
    <!-- ... -->
</LinearLayout>
```

Es poco habitual. Lo normal es dejar que se generen todas.

---

## 6. Problemas frecuentes

| Síntoma | Causa probable | Solución |
|---|---|---|
| `Unresolved reference: ActivityMainBinding` | No se ha sincronizado o no se ha compilado. | *Sync Now* y después *Build → Make Project*. |
| Sigue sin reconocerla tras sincronizar | Se activó en el `build.gradle.kts` **del proyecto**. | Muévelo al del **módulo** (`app`). |
| `Unresolved reference: databinding` en el `import` | El paquete no coincide con el `namespace`. | Revisa el `namespace` en `build.gradle.kts` y borra el import para que Android Studio lo vuelva a proponer. |
| `Unresolved reference` en una vista concreta (`binding.tvTitulo`) | La vista no tiene `android:id` o el id está escrito de otra forma. | Añade el id o usa el nombre en camelCase. |
| Error `Expecting '='` o `Unresolved reference: viewBinding` en Gradle | Sintaxis de Groovy en un fichero `.kts` (o al revés). | En `.kts`: `viewBinding = true`. En Groovy: `viewBinding true`. |
| La clase tiene un nombre inesperado | El nombre del fichero de layout no sigue `snake_case`. | Renombra el layout (`Shift + F6`) en minúsculas y con guiones bajos. |

---

## 7. Practica

1. Crea un proyecto **Empty Views Activity** llamado `PruebaBinding` (Kotlin, API mínima 24).
2. Abre los dos ficheros `build.gradle.kts` y localiza el del módulo `app`. ¿Qué `namespace` tiene?
3. Activa View Binding en el fichero correcto y sincroniza el proyecto.
4. Añade en `activity_main.xml` un `TextView` con id `tvSaludo`, un `EditText` con id `et_nombre` y un `Button` **sin id**.
5. En la `MainActivity`, escribe `ActivityMainBinding`, impórtala con `Alt + Enter` y abre el código generado con `Ctrl + clic`. ¿Qué propiedades tiene? ¿Aparece el botón? ¿Cómo se llama la propiedad del `EditText`?
6. Crea un segundo layout `activity_ajustes.xml` y comprueba, sin escribir nada más, que ya existe `ActivityAjustesBinding`.
7. Añade `tools:viewBindingIgnore="true"` a la raíz de `activity_ajustes.xml`, compila y comprueba qué ocurre con su clase.
8. **Reto:** activa View Binding en el proyecto *Pizzería DAM* del punto 1.3. Lo usaremos en el punto 2.3 para migrarlo.

---

## 8. Autoevaluación

1. ¿Está View Binding activado por defecto en los proyectos *Empty Views Activity*?
2. ¿En cuál de los dos ficheros `build.gradle.kts` se activa? ¿Por qué en ese?
3. Escribe el bloque que activa View Binding en Kotlin DSL y en Groovy.
4. ¿Qué hay que hacer después de modificar un fichero de Gradle para que los cambios tengan efecto?
5. ¿En qué paquete se generan las clases de vinculación?
6. ¿Dónde se guardan las clases generadas? ¿Se pueden modificar a mano?
7. ¿Qué dos métodos estáticos tiene toda clase de vinculación y qué hace cada uno?
8. ¿Para qué sirve `tools:viewBindingIgnore="true"` y dónde se coloca?
9. El editor marca `Unresolved reference: ActivityMainBinding` después de activar View Binding. Indica dos posibles causas y cómo resolverlas.
10. ¿Deja de usarse `findViewById()` por completo al usar View Binding? Razona la respuesta.

---

## 📚 Para saber más (documentación oficial)

- [Vinculación de vistas · Configuración](https://developer.android.com/topic/libraries/view-binding?hl=es-419#setup) · Cómo activarla en Gradle y excluir layouts.
- [Configura tu compilación](https://developer.android.com/build?hl=es-419) · Qué son los ficheros de Gradle de un proyecto Android y cómo se organizan.
- [Cómo migrar tu configuración de compilación de Groovy a Kotlin](https://developer.android.com/build/migrate-to-kotlin-dsl?hl=es-419) · Diferencias de sintaxis entre `build.gradle` y `build.gradle.kts`.

---

⬅️ [2.1 Qué es View Binding](02-1-que-es-view-binding.md) · 🏠 [Índice](../README.md) · ➡️ [2.3 Uso de View Binding](02-3-uso-view-binding.md)
