# 4.1 ¿Cómo definimos un menú?

> **Unidad 2 · Aplicaciones Android. Interfaz de usuario**
> Módulo: Programación multimedia y dispositivos móviles (2º DAM)

## Objetivos

Al terminar este apartado deberías ser capaz de:

- Distinguir los tres tipos de menús de Android: de opciones, contextuales y emergentes.
- Crear un recurso de menú en la carpeta `res/menu`.
- Definir las opciones de un menú con `<item>` y sus atributos principales.
- Decidir qué opciones aparecen como iconos en la barra con `app:showAsAction`.
- Organizar las opciones con grupos, opciones marcables y submenús.

---

## 1. Los menús en Android

Los menús permiten al usuario **elegir una acción de una lista**. Esa acción puede afectar a toda la aplicación (buscar, abrir los ajustes, ver la información de la app) o solo a una vista concreta (borrar un elemento, compartirlo).

Android distingue tres tipos de menús:

| Tipo | Dónde aparece | Cómo se abre | Ejemplo |
|---|---|---|---|
| **Menú de opciones** | En la **barra de la app** (*app bar*), arriba | Iconos visibles o el botón ⋮ (*overflow*) | Buscar, Ajustes, Acerca de |
| **Menú contextual** | Flotando sobre una vista, o como barra de acciones superior | **Pulsación larga** sobre una vista | Copiar, Borrar, Compartir un elemento |
| **Menú emergente** (*popup*) | Una lista anclada a una vista | Pulsar un botón o icono | "Ordenar por…", más acciones de una fila |

```
┌──────────────────────────────────────┐
│ Pizzería DAM        [carta] [cesta] ⋮ │  ← barra de la app: menú de opciones
│                       ┌──────────────┐ │
│                       │ Ajustes      │ │  ← opciones del desbordamiento (⋮)
│                       │ Acerca de    │ │
│                       └──────────────┘ │
│     (contenido de la actividad)       │
└──────────────────────────────────────┘
```

En este punto nos centraremos sobre todo en el **menú de opciones**, y en el punto 4.4 veremos los **menús contextuales** y los **emergentes**.

> 📝 **Recuerda (punto 1.1):** igual que los layouts, los menús se definen en **XML** como recursos, y después se cargan desde el código. Separar la definición del menú de su lógica facilita el mantenimiento y la traducción.

---

## 2. Crear un recurso de menú

Los menús se guardan en la carpeta **`res/menu`**, que no existe en los proyectos nuevos.

**Pasos en Android Studio:**

1. Clic derecho sobre `res` → **New → Android Resource File**.
2. *File name*: `menu_principal` (minúsculas y guiones bajos).
3. *Resource type*: **Menu**. Android Studio crea la carpeta `menu` si no existe.
4. Pulsa **OK**.

```
app/src/main/res/
├── layout/
│   └── activity_main.xml
├── menu/                    ← nueva carpeta
│   └── menu_principal.xml   ← recurso de menú
└── values/
    └── strings.xml
```

El fichero se abre en un editor visual parecido al de layouts, con su propia paleta (*Menu Item*, *Search Item*, *Menu*, *Group*). Como en el punto 1, trabajaremos sobre todo en modo **Code** o **Split**.

---

## 3. Estructura de un menú

Un recurso de menú tiene tres elementos posibles:

| Etiqueta | Qué representa |
|---|---|
| `<menu>` | El **contenedor**: es la raíz del fichero. Puede contener `<item>` y `<group>`. |
| `<item>` | **Una opción** del menú. Puede contener un `<menu>` anidado para crear un **submenú**. |
| `<group>` | Un **grupo** invisible de `<item>` que comparten propiedades (visibilidad, si se pueden marcar…). |

Ejemplo de menú de opciones para la *Pizzería DAM*:

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/action_carrito"
        android:icon="@drawable/ic_carrito"
        android:title="@string/menu_carrito"
        app:showAsAction="ifRoom" />

    <item
        android:id="@+id/action_carta"
        android:icon="@drawable/ic_carta"
        android:title="@string/menu_carta"
        app:showAsAction="ifRoom" />

    <item
        android:id="@+id/action_ajustes"
        android:title="@string/menu_ajustes"
        app:showAsAction="never" />

    <item
        android:id="@+id/action_acerca_de"
        android:title="@string/menu_acerca_de"
        app:showAsAction="never" />

</menu>
```

```xml
<!-- res/values/strings.xml -->
<string name="menu_carrito">Carrito</string>
<string name="menu_carta">Carta</string>
<string name="menu_ajustes">Ajustes</string>
<string name="menu_acerca_de">Acerca de</string>
```

> ⚠️ **No olvides `xmlns:app`.** El atributo `showAsAction` pertenece a la librería AppCompat, por eso se escribe `app:showAsAction`. Si escribes `android:showAsAction`, el editor no da error, pero **se ignora** y todas las opciones irán al desbordamiento ⋮.

---

## 4. Atributos de `<item>`

| Atributo | Para qué sirve |
|---|---|
| `android:id` | Identificador de la opción. **Imprescindible** para saber en el código qué opción se ha pulsado. |
| `android:title` | Texto de la opción. Siempre como recurso `@string/...`. |
| `android:icon` | Icono que se muestra si la opción aparece en la barra. |
| `app:showAsAction` | **Dónde** aparece la opción: en la barra como icono o en el desbordamiento ⋮. |
| `android:orderInCategory` | Número que fija el **orden** de las opciones (menor = antes). |
| `android:visible` | Si la opción se muestra (`true` por defecto). |
| `android:enabled` | Si la opción se puede pulsar (`true` por defecto). |
| `android:checkable` | Si la opción se puede **marcar** (aparece con una casilla). |
| `android:checked` | Si empieza marcada. |
| `app:actionViewClass` | Sustituye el icono por una **vista** (por ejemplo, un `SearchView` para buscar). |

> 💡 **Iconos:** usa *New → Vector Asset → Clip Art* (punto 1.3) para crear `ic_carrito` e `ic_carta`. En la barra, los iconos se tiñen automáticamente con el color del tema.

---

## 5. `app:showAsAction`: barra o desbordamiento

La barra de la app tiene un espacio limitado. `showAsAction` decide qué opciones aparecen **como icono en la barra** y cuáles quedan **escondidas** en el menú de desbordamiento ⋮.

| Valor | Comportamiento |
|---|---|
| `never` | Siempre en el desbordamiento ⋮. **Es el valor por defecto.** |
| `ifRoom` | En la barra **si cabe**; si no, en el desbordamiento. **El más recomendable.** |
| `always` | Siempre en la barra, aunque no quepa (puede tapar el título). Úsalo con moderación. |
| `withText` | Se combina con los anteriores para mostrar también el texto junto al icono, si cabe: `ifRoom\|withText`. |
| `collapseActionView` | Para opciones con una vista de acción (como la búsqueda): se muestra como icono y se expande al pulsarlo. |

> ✅ **Recomendación de Material Design:** muestra en la barra solo las **dos o tres** acciones más usadas (con `ifRoom`), y deja el resto en el desbordamiento. Opciones como *Ajustes* o *Acerca de* van siempre al desbordamiento.

> ⚠️ Si una opción aparece en la barra pero no tiene `android:icon`, se muestra su **texto** en mayúsculas. Si está en el desbordamiento, el icono **no** se muestra.

---

## 6. Grupos, opciones marcables y submenús

### 6.1 Grupos

Un `<group>` agrupa opciones que comparten comportamiento. Desde el código se puede, por ejemplo, ocultar todo el grupo de una vez con `menu.setGroupVisible(R.id.grupo_admin, false)`.

### 6.2 Opciones marcables

Con `android:checkableBehavior` en el grupo, sus opciones se muestran con casillas o botones de opción:

| Valor | Comportamiento |
|---|---|
| `none` | Ninguna se puede marcar (por defecto). |
| `all` | Se pueden marcar varias (**casillas**, como `CheckBox`). |
| `single` | Solo una a la vez (**botones de opción**, como `RadioButton`). |

```xml
<item
    android:id="@+id/action_ordenar"
    android:title="@string/menu_ordenar"
    app:showAsAction="never">
    <!-- Submenú: un <menu> dentro de un <item> -->
    <menu>
        <group android:checkableBehavior="single">
            <item
                android:id="@+id/orden_nombre"
                android:title="@string/orden_nombre"
                android:checked="true" />
            <item
                android:id="@+id/orden_precio"
                android:title="@string/orden_precio" />
        </group>
    </menu>
</item>
```

> ⚠️ Android **no marca ni desmarca** la opción automáticamente al pulsarla: lo haremos en el código con `item.isChecked = true` (punto 4.2).

### 6.3 Submenús

Como se ve en el ejemplo anterior, un submenú se crea poniendo un `<menu>` **dentro** de un `<item>`. Al pulsar la opción "Ordenar", se abre una segunda lista con sus opciones. Solo se admite **un nivel** de submenú.

---

## 7. Practica

1. Crea un proyecto **Empty Views Activity** llamado `PizzeriaMenus`, con View Binding activado.
2. Crea la carpeta `res/menu` y el recurso `menu_principal.xml` siguiendo los pasos de la sección 2.
3. Crea con *Vector Asset* los iconos `ic_carrito` e `ic_carta`.
4. Copia el menú de la sección 3 y pasa todos los títulos a `strings.xml`.
5. Abre el menú en modo **Split** y comprueba en la vista previa qué opciones salen como icono y cuáles en el desbordamiento.
6. Cambia el `showAsAction` de *Carta* a `never`, después a `always` y por último a `ifRoom|withText`. Anota qué ocurre en cada caso.
7. Cambia `app:showAsAction` por `android:showAsAction` en una opción. ¿Qué ocurre? ¿Por qué?
8. Añade el submenú "Ordenar" de la sección 6 con dos opciones de selección única.
9. **Reto:** añade una opción "Buscar" con `app:actionViewClass="androidx.appcompat.widget.SearchView"` y `app:showAsAction="ifRoom|collapseActionView"`. Lo probaremos en ejecución en el punto 4.2.

---

## 8. Autoevaluación

1. ¿Qué tres tipos de menús existen en Android? ¿Dónde aparece y cómo se abre cada uno?
2. ¿En qué carpeta se guardan los recursos de menú? ¿Cómo se crea desde Android Studio?
3. ¿Qué etiquetas pueden aparecer en un recurso de menú y qué representa cada una?
4. ¿Por qué es imprescindible el atributo `android:id` en cada `<item>`?
5. ¿Qué valores puede tomar `app:showAsAction`? ¿Cuál es el valor por defecto y cuál el más recomendable?
6. ¿Qué ocurre si se escribe `android:showAsAction` en lugar de `app:showAsAction`?
7. ¿Qué se muestra en la barra si una opción tiene `ifRoom` pero no tiene icono?
8. ¿Para qué sirve `android:orderInCategory`?
9. ¿Qué diferencia hay entre `checkableBehavior="all"` y `checkableBehavior="single"`?
10. ¿Cómo se crea un submenú? ¿Cuántos niveles de submenú se admiten?

---

## 📚 Para saber más (documentación oficial)

- [Cómo agregar menús](https://developer.android.com/guide/topics/ui/menus?hl=es-419) · Tipos de menús, recurso XML y atributos de cada opción.
- [Recurso de menú](https://developer.android.com/guide/topics/resources/menu-resource?hl=es-419) · Referencia completa de las etiquetas `<menu>`, `<item>` y `<group>`.
- [Cómo agregar y administrar acciones](https://developer.android.com/training/appbar/actions?hl=es-419) · Acciones en la barra de la app y `showAsAction`.

---

⬅️ [3.4 Diálogos](03-4-dialogos.md) · 🏠 [Índice](../README.md) · ➡️ [4.2 Añadiendo un menú a una actividad](04-2-menu-en-actividad.md)
