# Actividad: "Agencia de Viajes"

## Objetivo

Construir un mini-sitio con **Home (landing), Contact y Services** utilizando **Blade** con **layout común** y **partials** y **componentes**, aplicando un **CSS básico** (puedes reutilizar el del tema anterior).

## Enunciado

### 1. Estructura común (todas las páginas)

- Usa un **layout Blade** compartido con:
  - **Título de la página.**
  - **Barra de navegación** (enlace a Home, Services, Contact).
  - **Footer**.

- Implementa la navbar y el footer como **partials**

- Implementa los servicios como **componentes anónimos** (`<x-card ... />`).

- Aplica **CSS** para dar formato (puedes reutilizar el CSS de la práctica anterior).

### 2. Home (landing)

- Muestra el **nombre de la empresa (H1)**.

- Incluye una **imagen centrada** de portada.

### 3. Contact

- Presenta un **mapa** (imagen estática).

- Añade datos típicos de contacto: **teléfono** (`tel:`), **email** (`mailto:`), **dirección** (texto)...

### 4. Services

- Crea una **rejilla de tarjetas** (mín. 3).

- Cada **tarjeta** (componente anónimo `<x-card …>`) representará una **ciudad**, con:
  - **Imagen.**
  - **Precio.**
  - **Duración del viaje (nº de días).**
  - **Descripción breve.**
  - **Botón/enlace a alguna página de interés de la ciudad.**

### 5. Rutas

- Define las rutas `/`, `/contact` y `/services` para mostrar las vistas con Blade.

### 6. Rúbrica de evaluación

- **Estructura común: layout. (20%)**

- **partials: uso de partials para la navbar y footer. (20%)**

- **Uso de componentes: tarjetas de ciudades como componentes. (20%)**

- **Contenido y formato: imágenes, textos, enlaces y CSS básico. (10%)**

- **Rutas y navegación: rutas definidas y navegación funcional. (20%)**

- **Calidad del código: buenas prácticas en Blade y PHP. (10%)**

**Requisitos de calidad:** estructura semántica (encabezados, listas, enlaces), textos alternativos en imágenes, coherencia visual entre páginas y navegación funcional.

Busca en el aula virtual la tarea de la actividad del tema 3 y sube el documento pdf con la actividad realizada.

