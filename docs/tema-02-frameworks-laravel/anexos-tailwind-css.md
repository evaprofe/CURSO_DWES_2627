# Integración de Tailwind en Laravel 12

En este documento se trata de mostrar cómo integrar `Tailwind CSS` en un proyecto `Laravel 12` que ya está funcionando dentro de un entorno Dockerizado, sin necesidad de modificar la configuración de los contenedores (docker-compose.yml).

Se ha seguido instrucciones de la documentación oficial de `tailwindcss` y `Laravel Vite Plugin`, adaptándolas al entorno Docker.

## 1) Dónde estamos y qué trae Laravel 12

- Proyecto Laravel 12 recién creado.

- La página de bienvenida se ve correctamente maquetada, si no hay build de Vite, **inyecta un CSS inline** “estilo Tailwind” solo para esa vista.

- Para que **tus vistas** (home, about, etc.) puedan utilizar `Tailwind`, debemos:

    1. Añadir Tailwind a tu **entrada CSS** `resources/css/app.css`.
    2. Hacer **un build** con Vite para generar `public/build`.
    3. Usar `@vite([...])` en tu layout.

- No vamos a tocar Nginx ni otros contenedores. Vamos a instalar `tailwindcss` como ficheros estáticos dentro de nuestra estructura de proyecto.

---

## 2) Integrar Tailwind en el proyecto

Vamos a entrar dentro de nuestro contenedor `php` para instalar dependencias y hacer el *build*.

### 2.1. Entrar al contenedor

```bash
docker compose exec php bash
```

### 2.2. Instalar dependencias de frontend

Instalar node y npm antes de continuar.

primero con `cat /etc/os-release` mira si tu imagen es `debian` o `alpine`. Si es `debian` que es lo más probable, haz:

```bash
apt update && apt install -y nodejs npm
```

Si nuestra imagen fuera

alpine

, haríamos:

```bash
apk add --no-cache nodejs npm
```

!!! tip "Nota"

    Si estamos con los contenedores del curso lo normal es que sea `debian` y que no traigan node/npm instalados, ya que no están en `Dockerfile`.

Ahora que ya tenemos instalados `nodejs` y `npm`, dentro del contenedor `php`:

```bash
npm install
npm install -D tailwindcss postcss autoprefixer
```

### 2.3. Crear/editar el CSS de entrada con Tailwind v4

Ahora debemos asegurarnos de que nuestro CSS de entrada `resources/css/app.css` importe Tailwind y defina las rutas de escaneo. En principio ya trae este fichero, solo debemos corroborar que el contenido es como este:

!!! note "resources/css/app.css"

    ```css
    @import 'tailwindcss';

    /* Tailwind v4: rutas de escaneo para generar solo las clases usadas */
    @source '../../vendor/laravel/framework/src/Illuminate/Pagination/resources/views/*.blade.php';
    @source '../../storage/framework/views/*.php';
    @source '../**/*.blade.php';
    @source '../**/*.js';

    /* Opcional: tema base */
    @theme {
    --font-sans: 'Instrument Sans', ui-sans-serif, system-ui, sans-serif,
        'Apple Color Emoji','Segoe UI Emoji','Segoe UI Symbol','Noto Color Emoji';
    }
    ```

!!! tip "Nota"

    En v4 **no es obligatorio** `tailwind.config.js`. Con `@source` y `@theme` es suficiente para proyectos sencillos.

### 2.4. Revisar Vite

En la raíz de nuestra carpeta `src`, el fichero `vite.config.js` debe estar configurado para incluir `resources/css/app.css` como entrada.

!!! note "vite.config.js"

    ```js
    import { defineConfig } from 'vite'
    import laravel from 'laravel-vite-plugin'

    export default defineConfig({
    plugins: [
        laravel({
        input: ['resources/css/app.css', 'resources/js/app.js'],
        refresh: true,
        }),
    ],
    })
    ```

### 2.5. Incluir assets en el layout

En tu layout Blade debe existir:

```text
@vite(['resources/css/app.css','resources/js/app.js'])
```

### 2.6. Generar el build estático (una sola vez)

Todavía dentro del contenedor `php`:

```bash
npm run build
```

Debería crearse:

```text
public/build/manifest.json
public/build/assets/...
```

Con esto, la app deja de usar el CSS inline de la “welcome” y pasa a servir **tu Tailwind real** desde `public/build`, que `Nginx` ya lo entrega como estático. De esta manera no tenemos que tocar nada en Nginx ni en Docker.

## 3) Vistas de ejemplo: Home y About con Tailwind

### 3.1. Rutas

Archivo: `routes/web.php`

!!! note "routes/web.php"

    ```php
    use Illuminate\Support\Facades\Route;

    Route::view('/', 'home')->name('home');
    Route::view('/about', 'about')->name('about');
    ```

### 3.2. Layout base

Como aún no hemos visto layouts, cada vista es independiente. De momento no hace falta un layout común, cargaremos `@vite([...])` en cada vista.

### 3.3. Home

Archivo: `resources/views/home.blade.php`

!!! note "resources/views/home.blade.php"

    ```html
    <!doctype html>
    <html lang="es">

    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width,initial-scale=1">
        <title> Página de Inicio</title>
        @vite(['resources/css/app.css', 'resources/js/app.js'])
    </head>

    <body class="min-h-screen bg-gray-50 text-gray-900 antialiased">

        <main class="mx-auto max-w-3xl px-4 py-12">
            <section class="flex flex-col items-center text-center gap-6">
                <img src="https://picsum.photos/300/350" alt="Imagen de inicio" class="w-40 h-40 object-cover rounded-full shadow" />

                <h1 class="text-3xl md:text-4xl font-semibold">Bienvenido a la Home</h1>

                <p class="max-w-prose border rounded-lg shadow-sm p-4 bg-white">
                    Ejemplo con Tailwind: todo centrado, imagen con sombra y párrafo con borde y sombra.
                </p>
            </section>
        </main>

        <footer class="mt-12 border-t">
            <div class="mx-auto max-w-3xl px-4 py-6 text-sm text-gray-500">
                &copy; {{ date('Y') }} {{ config('app.name', 'Demo') }}
            </div>
        </footer>
    </body>

    </html>
    ```

### 3.4. About

Archivo: `resources/views/about.blade.php`

!!! note "resources/views/about.blade.php"

    ```html
    <!DOCTYPE html>
    <html lang="es">

    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width,initial-scale=1">
        <title> Página de Acerca de</title>
        @vite(['resources/css/app.css', 'resources/js/app.js'])
    </head>

    <body class="min-h-screen bg-gray-50 text-gray-900 antialiased">
        <main class="mx-auto max-w-3xl px-4 py-12">
            <section class="flex flex-col items-center text-center gap-6">
                <img src="https://picsum.photos/300/350" alt="Imagen de about"
                    class="w-40 h-40 object-cover rounded-lg shadow" />

                <h1 class="text-3xl md:text-4xl font-semibold">Sobre este proyecto</h1>

                <p class="max-w-prose border rounded-lg shadow-sm p-4 bg-white">
                    Integración básica de utilidades Tailwind directamente en Blade para mejorar el aspecto sin escribir CSS
                    tradicional.
                </p>
            </section>
        </main>

        <footer class="mt-12 border-t">
            <div class="mx-auto max-w-3xl px-4 py-6 text-sm text-gray-500">
                &copy; {{ date('Y') }} {{ config('app.name', 'Demo') }}
            </div>
        </footer>
    </body>

    </html>
    ```

### 3.5. Imágenes de demo

Para no tener que descargar imágenes, usamos `https://picsum.photos/` que genera imágenes aleatorias de prueba.

---

## 4) Resumen de pasos

1. **Entrar al contenedor php** `docker compose exec php bash`
2. **Instalar** `npm install` `npm install -D tailwindcss postcss autoprefixer`
3. **Configurar `resources/css/app.css`** con `@import 'tailwindcss'` y `@source`.
4. **Comprobar `vite.config.js`** tiene `resources/css/app.css` y `resources/js/app.js`.
5. **Layout** con `@vite([...])`.
6. **Build**: `npm run build` Verifica `public/build/manifest.json`.
7. **Rutas + Vistas** copiadas arriba.
8. **Imágenes** en `public/img`.

Hecho. A partir de aquí Nginx sirve `public/build` y las vistas cargan Tailwind.
