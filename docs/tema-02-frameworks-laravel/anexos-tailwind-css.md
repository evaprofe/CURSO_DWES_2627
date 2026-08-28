# Tailwind CSS - Introducción a Laravel

Integración de
Tailwind en Laravel 12
En este documento se trata de mostrar
cómo integrar Tailwind CSS en un
proyecto Laravel 12 que ya está
funcionando dentro de un entorno
Dockerizado, sin necesidad de modificar
la configuración de los contenedores
(docker-compose.yml).
Se ha seguido instrucciones de la
documentación oficial de tailwindcss y
Laravel Vite Plugin , adaptándolas al
entorno Docker.
1� Dónde estamos y qué
trae Laravel 12
• Proyecto Laravel 12 recién creado.
• La página de bienvenida se ve
correctamente maquetada, si no hay
build de Vite,
“estilo Tailwind” solo para esa vista.
• Para que (home, about,
etc.) puedan utilizar Tailwind ,
debemos:
2.5 Estructura de un proyecto
Laravel

---

a��Añadir Tailwind a tu
resources/css/app.css .
b��Hacer con Vite para
generar public/build .
c��Usar @vite([...]) en tu layout.
• No vamos a tocar Nginx ni otros
contenedores. Vamos a instalar
tailwindcss como ficheros
estáticos dentro de nuestra estructura
de proyecto.
2� Integrar Tailwind en el
proyecto
Vamos a entrar dentro de nuestro
contenedor php para instalar
dependencias y hacer el build.
2.1. Entrar al contenedor
2.2. Instalar dependencias de
frontend
docker compose exec php
bash

---

Instalar node y npm antes de continuar.
primero con cat /etc/os-release
mira si tu imagen es debian o alpine .
Si es debian que es lo más probable,
haz:
Si nuestra imagen fuera alpine ,
haríamos:
Si estamos con los contenedores del curso lo
normal es que sea debian y que no traigan
node/npm instalados, ya que no están en
Dockerfile .
Ahora que ya tenemos instalados nodejs
y npm , dentro del contenedor php :
apt update && apt
install -y nodejs npm
apk add --no-cache
nodejs npm
npm install
npm install -D
tailwindcss postcss
autoprefixer

---

2.3. Crear/editar el CSS de
entrada con Tailwind v4
Ahora debemos asegurarnos de que
nuestro CSS de entrada resources/
css/app.css importe Tailwind y defina
las rutas de escaneo. En principio ya trae
este fichero, solo debemos corroborar que
el contenido es como este:

---

@import
'tailwindcss';
/* Tailwind v4: rutas
de escaneo para
generar solo las
clases usadas */
@source '../../
vendor/laravel/
framework/src/
Illuminate/
Pagination/resources/
views/*.blade.php';
@source '../../
storage/framework/
views/*.php';
@source '../**/
*.blade.php';
@source '../**/*.js';
/* Opcional: tema
base */
@theme {
--font-sans:
'Instrument Sans',
ui-sans-serif,
system-ui, sansserif,
'Apple Color
Emoji','Segoe UI
Emoji','Segoe UI
Symbol','Noto Color
Emoji';
}

---

En v4
tailwind.config.js . Con @source
y @theme es suficiente para proyectos
sencillos.
2.4. Revisar Vite
En la raíz de nuestra carpeta src , el
fichero vite.config.js debe estar
configurado para incluir resources/
css/app.css como entrada.
import { defineConfig
} from 'vite'
import laravel from
'laravel-vite-plugin'
export default
defineConfig({
plugins: [
laravel({
input:
['resources/css/
app.css', 'resources/
js/app.js'],
refresh: true,
}),
],
})

---

2.5. Incluir assets en el layout
En tu layout Blade debe existir:
2.6. Generar el build estático
(una sola vez)
Todavía dentro del contenedor php :
Debería crearse:
Con esto, la app deja de usar el CSS inline
de la “welcome” y pasa a servir
desde public/build , que
Nginx ya lo entrega como estático. De
esta manera no tenemos que tocar nada
en Nginx ni en Docker.
@vite(['resources/css/
app.css','resources/js/
app.js'])
npm run build
public/build/
manifest.json
public/build/assets/...

---

3� Vistas de ejemplo:
Home y About con Tailwind
3.1. Rutas
Archivo: routes/web.php
3.2. Layout base
Como aún no hemos visto layouts, cada
vista es independiente. De momento no
hace falta un layout común, cargaremos
@vite([...]) en cada vista.
3.3. Home
Archivo: resources/views/
home.blade.php
use
Illuminate\Support\Facades\Route;
Route::view('/', 'home')-
>name('home');
Route::view('/about', 'about')-
>name('about');

---

<!doctype html>
<html lang="es">
<head>
<meta
charset="utf-8">
<meta
name="viewport"
content="width=devicewidth,initialscale=1">
<title> Página de
Inicio</title>
@vite(['resources/
css/app.css',
'resources/js/
app.js'])
</head>
<body class="min-hscreen bg-gray-50
text-gray-900
antialiased">
<main class="mxauto max-w-3xl px-4
py-12">
<section
class="flex flex-col
items-center textcenter gap-6">
<img
src="https://
picsum.photos/300/350"
alt="Imagen de inicio"
class="w-40 h-40
object-cover roundedfull shadow" />

---

<h1
class="text-3xl
md:text-4xl fontsemibold">Bienvenido a
la Home</h1>
<p
class="max-w-prose
border rounded-lg
shadow-sm p-4 bgwhite">
Ejemplo con Tailwind:
todo centrado, imagen
con sombra y párrafo
con borde y sombra.
</p>
</section>
</main>
<footer
class="mt-12 bordert">
<div
class="mx-auto maxw-3xl px-4 py-6 textsm text-gray-500">
&copy; {{
date('Y') }} {{
config('app.name',
'Demo') }}
</div>
</footer>
</body>
</html>
3.4. About

---

Archivo: resources/views/
about.blade.php

---

<!DOCTYPE html>
<html lang="es">
<head>
<meta
charset="utf-8">
<meta
name="viewport"
content="width=devicewidth,initialscale=1">
<title> Página de
Acerca de</title>
@vite(['resources/
css/app.css',
'resources/js/
app.js'])
</head>
<body class="min-hscreen bg-gray-50
text-gray-900
antialiased">
<main class="mxauto max-w-3xl px-4
py-12">
<section
class="flex flex-col
items-center textcenter gap-6">
<img
src="https://
picsum.photos/300/350"
alt="Imagen de about"
class="w-40 h-40
object-cover roundedlg shadow" />

---

<h1
class="text-3xl
md:text-4xl fontsemibold">Sobre este
proyecto</h1>
<p
class="max-w-prose
border rounded-lg
shadow-sm p-4 bgwhite">
Integración básica de
utilidades Tailwind
directamente en Blade
para mejorar el
aspecto sin escribir
CSS
tradicional.
</p>
</section>
</main>
<footer
class="mt-12 bordert">
<div
class="mx-auto maxw-3xl px-4 py-6 textsm text-gray-500">
&copy; {{
date('Y') }} {{
config('app.name',
'Demo') }}
</div>
</footer>
</body>
</html>

---

3.5. Imágenes de demo
Para no tener que descargar imágenes,
usamos https://picsum.photos/ que
genera imágenes aleatorias de prueba.
4� Resumen de pasos
��� docker
compose exec php bash
��� npm install npm
install -D tailwindcss
postcss autoprefixer
���
con @import
'tailwindcss' y @source .
��� tiene
resources/css/app.css y
resources/js/app.js .
��� con @vite([...]) .
��� : npm run build Verifica
public/build/manifest.json .
��� copiadas arriba.
��� en public/img .
Hecho. A partir de aquí Nginx sirve
public/build y las vistas cargan
Tailwind.

IES JUAN DE GARAY - 2º DWES (Curso original de Beteta adaptado por Eva María Gómez Abad )
