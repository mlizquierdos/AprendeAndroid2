# AprendeAndroid2

Repositorio de apuntes del módulo **Programación Multimedia y Dispositivos Móviles** (2º DAM).

## UT02 · Aplicaciones Android. Interfaz de usuario

### Objetivos de aprendizaje

1. Crear aplicaciones móviles con diferentes diseños y elementos de interfaz.
2. Enlazar la interfaz con la lógica de la aplicación.
3. Utilizar clases para el modelado de alertas y notificaciones.
4. Crear aplicaciones con varias actividades que se comuniquen entre ellas.
5. Crear aplicaciones con listas dinámicas.

### Índice de contenidos

| Punto | Contenido | Apuntes |
|------|-----------|---------|
| **1** | **Diseñando interfaces de usuario en Android** | |
| 1.1 | El editor de Layouts de Android Studio | [01-1-editor-de-layouts.md](UT02/01-1-editor-de-layouts.md) |
| 1.2 | Diseño de Layouts | [01-2-diseno-de-layouts.md](UT02/01-2-diseno-de-layouts.md) |
| 1.3 | Elementos de la interfaz | [01-3-elementos-de-la-interfaz.md](UT02/01-3-elementos-de-la-interfaz.md) |
| **2** | **View Binding: Vinculación de vistas** | |
| 2.1 | Qué es View Binding | [02-1-que-es-view-binding.md](UT02/02-1-que-es-view-binding.md) |
| 2.2 | Activación de View Binding | [02-2-activacion-view-binding.md](UT02/02-2-activacion-view-binding.md) |
| 2.3 | Uso de View Binding | [02-3-uso-view-binding.md](UT02/02-3-uso-view-binding.md) |
| **3** | **Notificaciones y diálogos** | |
| 3.1 | Toasts | [03-1-toasts.md](UT02/03-1-toasts.md) |
| 3.2 | Snackbars | [03-2-snackbars.md](UT02/03-2-snackbars.md) |
| 3.3 | Notificaciones: canal de notificaciones · crear y mostrar notificaciones | [03-3-notificaciones.md](UT02/03-3-notificaciones.md) |
| 3.4 | Diálogos: diálogos con DialogFragment · diálogos y callbacks | [03-4-dialogos.md](UT02/03-4-dialogos.md) |
| **4** | **Navegación entre actividades: Intents y Menús** | |
| 4.1 | ¿Cómo definimos un menú? | [04-1-definir-menu.md](UT02/04-1-definir-menu.md) |
| 4.2 | Añadiendo un menú a una actividad · gestionando los clics en el menú | [04-2-menu-en-actividad.md](UT02/04-2-menu-en-actividad.md) |
| 4.3 | Uso de menús comunes a varias actividades · Intents y mejoras en el menú | [04-3-menus-comunes-intents.md](UT02/04-3-menus-comunes-intents.md) |
| 4.4 | Mejorando el comportamiento de las Intents · menús contextuales | [04-4-intents-y-menus-contextuales.md](UT02/04-4-intents-y-menus-contextuales.md) |
| **5** | **El componente RecyclerView** | |
| 5.1 | Listas con RecyclerView | [05-1-listas-recyclerview.md](UT02/05-1-listas-recyclerview.md) |
| 5.2 | Elementos para crear un RecyclerView | [05-2-elementos-recyclerview.md](UT02/05-2-elementos-recyclerview.md) |
| 5.3 | Creación del adaptador: la clase ViewHolder · la clase Adapter · creación del RecyclerView | [05-3-creacion-adaptador.md](UT02/05-3-creacion-adaptador.md) |
| 5.4 | Gestión de eventos en el RecyclerView · mediante callbacks | [05-4-eventos-recyclerview.md](UT02/05-4-eventos-recyclerview.md) |

### Estructura del repositorio

```
AprendeAndroid2/
├── README.md
└── UT02/
    ├── 01-1-editor-de-layouts.md
    ├── 01-2-diseno-de-layouts.md
    ├── 01-3-elementos-de-la-interfaz.md
    ├── 02-1-que-es-view-binding.md
    ├── 02-2-activacion-view-binding.md
    ├── 02-3-uso-view-binding.md
    ├── 03-1-toasts.md
    ├── 03-2-snackbars.md
    ├── 03-3-notificaciones.md
    ├── 03-4-dialogos.md
    ├── 04-1-definir-menu.md
    ├── 04-2-menu-en-actividad.md
    ├── 04-3-menus-comunes-intents.md
    ├── 04-4-intents-y-menus-contextuales.md
    ├── 05-1-listas-recyclerview.md
    ├── 05-2-elementos-recyclerview.md
    ├── 05-3-creacion-adaptador.md
    ├── 05-4-eventos-recyclerview.md
    └── img/
        ├── editor-layout-android-studio.png
        ├── recyclerview-componentes.jpg
        ├── recyclerview-metodos-adaptador.jpg
        └── recyclerview-reciclado.jpg
```

> **Requisitos:** Android Studio (versión estable actual), Kotlin y el sistema de vistas basado en **XML** (no Jetpack Compose).

---

## 🔗 Enlaces web

En esta sección encontrarás aquellos enlaces de interés necesarios para ampliar e investigar los contenidos de la unidad.

### 1. Diseñando interfaces de usuario en Android

- **Página oficial Desarrolladores de Android. Cómo desarrollar una UI con views (editor de diseño).** (developer.android.com)
  https://developer.android.com/studio/views/layout-editor?hl=es-419
- **Página oficial Desarrolladores de Android. Diseños en Views.** (developer.android.com)
  https://developer.android.com/guide/topics/ui/declaring-layout?hl=es-419
- **Página oficial Desarrolladores de Android. Cómo compilar una IU responsiva con ConstraintLayout.** (developer.android.com)
  https://developer.android.com/training/constraint-layout?hl=es-419
- **Página oficial Desarrolladores de Android. Diseño responsivo/adaptable con vistas.** (developer.android.com)
  https://developer.android.com/develop/ui/views/layout/responsive-adaptive-design-with-views?hl=es-419
- **Página oficial Desarrolladores de Android. Descripción general de los eventos de entrada.** (developer.android.com)
  https://developer.android.com/guide/topics/ui/ui-events?hl=es-419
- **Página oficial Desarrolladores de Android. Cómo agregar spinners a tu app.** (developer.android.com)
  https://developer.android.com/guide/topics/ui/controls/spinner?hl=es-419
- **Página oficial Desarrolladores de Android. Estilos y temas.** (developer.android.com)
  https://developer.android.com/guide/topics/ui/themes?hl=es-419
- **Página oficial Desarrolladores de Android. Cómo hacer que las apps sean más accesibles.** (developer.android.com)
  https://developer.android.com/guide/topics/ui/accessibility/apps?hl=es-419
- **Material Design. Primeros pasos con Material Components para Android** (en inglés). (material.io)
  https://material.io/develop/android/docs/getting-started

### 2. View Binding: Vinculación de vistas

- **Página oficial Desarrolladores de Android. Vinculación de vistas.** (developer.android.com)
  https://developer.android.com/topic/libraries/view-binding?hl=es-419
- **Página oficial Desarrolladores de Android. Cómo migrar de Kotlin synthetics a la vinculación de vistas de Jetpack.** (developer.android.com)
  https://developer.android.com/topic/libraries/view-binding/migration?hl=es-419
- **Página oficial Desarrolladores de Android. Biblioteca de vinculación de datos (Data Binding).** (developer.android.com)
  https://developer.android.com/topic/libraries/data-binding?hl=es-419

### 3. Notificaciones y diálogos

- **Página oficial Desarrolladores de Android. Descripción general de los avisos (Toast).** (developer.android.com)
  https://developer.android.com/guide/topics/ui/notifiers/toasts?hl=es-419
- **Página oficial Desarrolladores de Android. Cómo crear y mostrar un mensaje emergente (Snackbar).** (developer.android.com)
  https://developer.android.com/develop/ui/views/notifications/snackbar/showing?hl=es-419
- **Página oficial Desarrolladores de Android. Creación de notificaciones.** (developer.android.com)
  https://developer.android.com/develop/ui/views/notifications/build-notification?hl=es-419
- **Página oficial Desarrolladores de Android. Cómo crear y administrar canales de notificaciones.** (developer.android.com)
  https://developer.android.com/develop/ui/views/notifications/channels?hl=es-419
- **Página oficial Desarrolladores de Android. Permiso de tiempo de ejecución para notificaciones.** (developer.android.com)
  https://developer.android.com/develop/ui/views/notifications/notification-permission?hl=es-419
- **Página oficial Desarrolladores de Android. Diálogos.** (developer.android.com)
  https://developer.android.com/develop/ui/views/components/dialogs?hl=es-419
- **Página oficial Desarrolladores de Android. Cómo comunicarse con fragmentos (Fragment Result API).** (developer.android.com)
  https://developer.android.com/guide/fragments/communicate?hl=es-419

### 4. Navegación entre actividades: Intents y Menús

- **Página oficial Desarrolladores de Android. Cómo agregar menús.** (developer.android.com)
  https://developer.android.com/guide/topics/ui/menus?hl=es-419
- **Página oficial Desarrolladores de Android. Recurso de menú.** (developer.android.com)
  https://developer.android.com/guide/topics/resources/menu-resource?hl=es-419
- **Página oficial Desarrolladores de Android. Cómo agregar y administrar acciones (barra de la app).** (developer.android.com)
  https://developer.android.com/training/appbar/actions?hl=es-419
- **Página oficial Desarrolladores de Android. Intents y filtros de intents.** (developer.android.com)
  https://developer.android.com/guide/components/intents-filters?hl=es-419
- **Página oficial Desarrolladores de Android. Intents comunes.** (developer.android.com)
  https://developer.android.com/guide/components/intents-common?hl=es-419
- **Página oficial Desarrolladores de Android. Tareas y la pila de actividades.** (developer.android.com)
  https://developer.android.com/guide/components/activities/tasks-and-back-stack?hl=es-419
- **Página oficial Desarrolladores de Android. Cómo obtener el resultado de una actividad.** (developer.android.com)
  https://developer.android.com/training/basics/intents/result?hl=es-419

### 5. El componente RecyclerView

- **Página oficial Desarrolladores de Android. Cómo crear listas dinámicas con RecyclerView.** (developer.android.com)
  https://developer.android.com/develop/ui/views/layout/recyclerview?hl=es-419
- **Página oficial Desarrolladores de Android. Referencia de RecyclerView.Adapter** (en inglés). (developer.android.com)
  https://developer.android.com/reference/androidx/recyclerview/widget/RecyclerView.Adapter
- **Página oficial Desarrolladores de Android. ListAdapter y DiffUtil** (en inglés), para ir más allá. (developer.android.com)
  https://developer.android.com/reference/androidx/recyclerview/widget/ListAdapter
- **Kotlin. Funciones de orden superior y lambdas** (en inglés). (kotlinlang.org)
  https://kotlinlang.org/docs/lambdas.html

> 💡 Si alguna página no está disponible en español, cambia `?hl=es-419` por `?hl=es` o elimínalo para ver la versión original en inglés, que suele estar más actualizada.
