# Login y roles en Laravel con Breeze

!!! quote "Autoría"

    Lección nueva escrita para 2º DAW del IES Juan de Garay (Eva María Gómez Abad). Continúa el proyecto de productos del curso [*Introducción a Laravel*](https://jbeteta-ies.github.io/2025_Laravel_32/) de **Javier Beteta** (temas 6, 7 y 9).

!!! abstract "Qué vas a conseguir"

    Al terminar esta lección tu tienda tendrá:

    - **Registro, login y logout** de usuarios, hechos con **Laravel Breeze**.
    - Un campo **`role`** en la tabla `users` con dos valores: `user` y `admin`.
    - Un **middleware** propio, `admin`, que deja pasar solo al administrador.
    - El **CRUD de productos protegido**: cualquiera puede ver el catálogo, pero solo el administrador puede crear, editar y borrar.
    - Los botones de crear, editar y borrar **ocultos** para quien no es administrador.

## Antes de empezar: dos ideas clave

**Autenticación** es saber *quién eres*: el usuario escribe su email y su contraseña y Laravel comprueba que coinciden con un registro de la tabla `users`. A partir de ese momento, Laravel recuerda al usuario en la sesión.

**Autorización** es saber *qué puedes hacer*: un usuario identificado puede ver productos, pero solo el administrador puede borrarlos.

Breeze resuelve la **autenticación**. La **autorización** la vamos a programar nosotros con un rol y un middleware.

```mermaid
flowchart LR
    A[Petición: DELETE /product/5] --> B{¿Ha iniciado sesión?<br>middleware auth}
    B -- No --> L[Redirige a /login]
    B -- Sí --> C{¿Es admin?<br>middleware admin}
    C -- No --> E[Error 403]
    C -- Sí --> D[ProductController@destroy]
```

!!! info "¿Por qué Breeze?"

    Breeze es el kit de inicio de Laravel que genera el login con **vistas Blade**, que es lo que ya sabes usar. Escribe en tu proyecto los controladores, las rutas y las vistas del login, así que puedes abrirlos y entender cómo funcionan.

    Desde Laravel 12, Laravel ya no añade funciones nuevas a Breeze: sus kits nuevos usan React, Vue o Livewire. Breeze sigue funcionando con Laravel 12, que es la versión del curso.

## Paso 0. Guarda tu trabajo (¡importante!)

Breeze **sobrescribe** varios archivos de tu proyecto al instalarse. Los que más te afectan son:

| Archivo | Qué pasa |
|---|---|
| `routes/web.php` | Se sustituye entero. **Perderías tus rutas de productos.** |
| `resources/views/layouts/app.blade.php` | Se sustituye por el layout de Breeze. **Tus vistas de productos dejarían de verse bien.** |
| `resources/css/app.css`, `resources/js/app.js`, `vite.config.js`, `tailwind.config.js` | Se sustituyen por los de Breeze. |

Por eso, antes de nada:

1. Haz un commit con todo lo que tienes:

    ```bash
    git add .
    git commit -m "Tienda antes de instalar Breeze"
    ```

2. Copia los dos archivos que vas a necesitar recuperar:

    ```bash
    cp routes/web.php routes/web.tienda.php
    cp resources/views/layouts/app.blade.php resources/views/layouts/tienda.blade.php
    ```

    El segundo comando guarda tu layout con otro nombre: `layouts/tienda.blade.php`. Breeze no lo toca.

!!! tip "Si algo sale mal"

    Con el commit del paso 1 puedes ver qué ha cambiado con `git status` y `git diff`, y recuperar cualquier archivo con `git checkout -- ruta/del/archivo`.

## Paso 1. Instalar Breeze

Todos los comandos se ejecutan **dentro del contenedor `php`**, como en el resto del curso. Puedes entrar con `docker compose exec php bash` o ponerlo delante de cada comando.

Breeze compila su CSS con **Node y npm**. Si no los instalaste en el [anexo de Tailwind](../tema-02-frameworks-laravel/anexos-tailwind-css.md), instálalos primero en el contenedor:

```bash
apt update && apt install -y nodejs npm
```

Ahora instala Breeze:

```bash
composer require laravel/breeze --dev
php artisan breeze:install blade
```

El segundo comando copia los archivos del login y después ejecuta `npm install` y `npm run build` por ti. Si te pregunta por el modo oscuro, contesta que no; si te pregunta por el framework de pruebas, elige **PHPUnit**.

Por último, crea las tablas que faltan (si ya existen, no hace nada):

```bash
php artisan migrate
```

!!! success "Comprueba"

    Abre `http://localhost:8080/register`, crea un usuario y verás el panel (`/dashboard`) de Breeze. Cierra sesión desde el menú de arriba a la derecha y vuelve a entrar desde `/login`.

### Qué ha añadido Breeze

| Dónde | Qué es |
|---|---|
| `routes/auth.php` | Rutas de registro, login, logout y recuperar contraseña. |
| `app/Http/Controllers/Auth/` | Los controladores de esas rutas. Por ejemplo, `AuthenticatedSessionController` hace el login y el logout. |
| `app/Http/Controllers/ProfileController.php` | Página para editar el perfil y cambiar la contraseña. |
| `resources/views/auth/` | Las vistas `login`, `register`, etc. |
| `resources/views/layouts/app.blade.php` | El layout de Breeze, con el menú de usuario. |
| `resources/views/layouts/navigation.blade.php` | El menú de arriba. |

Abre `app/Http/Controllers/Auth/RegisteredUserController.php`: verás que el método `store` valida los datos con `$request->validate(...)`, crea el usuario con `User::create(...)` y lo deja con la sesión iniciada con `Auth::login($user)`. Es lo mismo que hiciste en el CRUD de productos.

## Paso 2. Recuperar tus rutas y tu layout

### Las rutas

Abre `routes/web.php` (ahora es el de Breeze) y añade al final las rutas de tu tienda, copiándolas de `routes/web.tienda.php`. Por ahora déjalas tal y como estaban; en el paso 5 las protegeremos. Cuando todo funcione, borra `routes/web.tienda.php`.

### El layout

Las vistas de productos usan `@extends('layouts.app')` y `@section('content')`. El layout de Breeze funciona de otra manera (es un **componente** y usa `{{ $slot }}` en lugar de `@yield`), así que tus vistas ya no encajan con él.

La solución es apuntar tus vistas a tu layout, que guardaste como `tienda.blade.php`. En cada vista de `resources/views/product/` cambia:

```html
@extends('layouts.app')
```

por:

```html
@extends('layouts.tienda')
```

!!! success "Comprueba"

    Entra en `http://localhost:8080/product`: tu listado de productos debe verse como antes.

## Paso 3. Añadir el rol a los usuarios

### La migración

Creamos una migración que **modifica** la tabla `users` (lo viste en el tema 4):

```bash
php artisan make:migration add_role_to_users_table --table=users
```

```php title="database/migrations/xxxx_xx_xx_xxxxxx_add_role_to_users_table.php"
public function up(): void
{
    Schema::table('users', function (Blueprint $table) {
        $table->string('role')->default('user')->after('email');
    });
}

public function down(): void
{
    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn('role');
    });
}
```

```bash
php artisan migrate
```

Con `default('user')`, todo usuario que se registre será `user` sin que tengamos que hacer nada.

### El modelo `User`

Añade este método a `app/Models/User.php`:

```php title="app/Models/User.php"
public function isAdmin(): bool
{
    return $this->role === 'admin';
}
```

!!! warning "No añadas `role` a `$fillable`"

    `$fillable` indica qué campos se pueden rellenar de golpe con `User::create($request->all())` o similares. Si `role` estuviera ahí, un alumno con un poco de picardía podría añadir un campo oculto `role=admin` al formulario de registro y hacerse administrador. Dejándolo fuera, el rol solo se puede poner desde el código.

### Crear el administrador con un seeder

En el tema 9 viste los seeders. Añade el administrador en `database/seeders/DatabaseSeeder.php`:

```php title="database/seeders/DatabaseSeeder.php"
User::factory()->create([
    'name' => 'Administrador',
    'email' => 'admin@tienda.test',
    'password' => bcrypt('admin1234'),
    'role' => 'admin',
]);
```

Las factories pueden rellenar `role` aunque no esté en `$fillable`, porque crean el modelo sin esa protección.

```bash
php artisan db:seed
```

!!! note "Si el seeder falla"

    El email de `users` es único: si ejecutas `db:seed` dos veces, la segunda falla porque el administrador ya existe. Para empezar de cero, con **todas** las tablas vacías, usa `php artisan migrate:fresh --seed` (solo en tu ordenador, nunca en producción).

!!! tip "Cambiar el rol a mano"

    También puedes hacer administrador a un usuario que ya existe con Tinker:

    ```bash
    php artisan tinker
    ```

    ```php
    $u = App\Models\User::where('email', 'ana@tienda.test')->first();
    $u->role = 'admin';
    $u->save();
    ```

## Paso 4. El middleware `admin`

Un **middleware** es un filtro que se ejecuta **antes** de llegar al controlador. Ya has usado uno sin saberlo: `auth`, que manda a `/login` a quien no ha iniciado sesión. Ahora vamos a escribir el nuestro.

```bash
php artisan make:middleware EsAdmin
```

```php title="app/Http/Middleware/EsAdmin.php"
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EsAdmin
{
    public function handle(Request $request, Closure $next): Response
    {
        // $request->user() es el usuario con la sesión iniciada (o null)
        if (! $request->user() || ! $request->user()->isAdmin()) {
            abort(403, 'Solo el administrador puede hacer esto.');
        }

        // Si es admin, la petición sigue hacia el controlador
        return $next($request);
    }
}
```

Le damos un nombre corto, `admin`, en `bootstrap/app.php`:

```php title="bootstrap/app.php"
->withMiddleware(function (Middleware $middleware): void {
    $middleware->alias([
        'admin' => \App\Http\Middleware\EsAdmin::class,
    ]);
})
```

## Paso 5. Proteger las rutas de productos

Queremos que **ver** productos sea público y que **crear, editar y borrar** sea solo para el administrador. Con `Route::resource` y los métodos `only` y `except` (tema 7) queda así:

```php title="routes/web.php"
use App\Http\Controllers\ProductController;

// Solo el administrador: crear, guardar, editar, actualizar y borrar
Route::middleware(['auth', 'admin'])->group(function () {
    Route::resource('product', ProductController::class)
        ->except(['index', 'show']);
});

// Todo el mundo: listado y detalle
Route::resource('product', ProductController::class)
    ->only(['index', 'show']);
```

!!! warning "El orden importa"

    El grupo del administrador va **antes**. Si pusieras primero `show`, la dirección `/product/create` encajaría con `/product/{product}` y Laravel buscaría un producto con id `create`: verías un error 404 en lugar del formulario.

Fíjate en el orden de los middleware: primero `auth` (si no has iniciado sesión te lleva a `/login`) y después `admin` (si has iniciado sesión pero no eres administrador, error 403).

Comprueba las rutas y sus middleware con:

```bash
php artisan route:list --path=product -v
```

## Paso 6. Mostrar u ocultar botones según el rol

Que el usuario normal no pueda borrar no basta: tampoco debería **ver** el botón. Blade tiene directivas para esto:

```html title="resources/views/product/index.blade.php"
@auth
    <p>Hola, {{ auth()->user()->name }}</p>

    @if (auth()->user()->isAdmin())
        <a href="{{ route('product.create') }}">Nuevo producto</a>
    @endif
@endauth

@guest
    <a href="{{ route('login') }}">Iniciar sesión</a>
    <a href="{{ route('register') }}">Registrarse</a>
@endguest
```

- `@auth ... @endauth`: solo se muestra si hay sesión iniciada.
- `@guest ... @endguest`: solo se muestra a los visitantes sin sesión.
- `auth()->user()` devuelve el usuario conectado.

Haz lo mismo con los botones de **editar** y **borrar** de cada producto.

Para cerrar sesión desde tu layout hace falta un formulario `POST` (no un enlace), porque así lo define `routes/auth.php`:

```html title="resources/views/layouts/tienda.blade.php"
@auth
    <form method="POST" action="{{ route('logout') }}">
        @csrf
        <button type="submit">Cerrar sesión</button>
    </form>
@endauth
```

!!! danger "Ocultar no es proteger"

    Esconder un botón es solo comodidad para el usuario. Cualquiera puede escribir la URL `/product/create` a mano o enviar un formulario con Postman. La seguridad de verdad la pone el **middleware** del paso 5. Necesitas las dos cosas.

## Paso 7. Probarlo todo

| Prueba | Resultado esperado |
|---|---|
| Sin sesión, abrir `/product` | Ves el listado, sin botones de crear, editar ni borrar. |
| Sin sesión, abrir `/product/create` | Te redirige a `/login`. |
| Como usuario normal, abrir `/product/create` | Error **403**. |
| Como usuario normal, ver el listado | No aparecen los botones de administración. |
| Como `admin@tienda.test`, crear, editar y borrar un producto | Funciona y aparecen los mensajes flash del tema 7. |
| Cerrar sesión | Vuelves a ver la tienda como visitante. |

## Resumen

- **Breeze** genera registro, login, logout y perfil con vistas Blade. Antes de instalarlo, guarda `routes/web.php` y tu layout.
- El **rol** es una columna más de `users`, con valor por defecto `user`, y **no** va en `$fillable`.
- Un **middleware** filtra la petición antes del controlador; el nuestro devuelve 403 si no eres administrador.
- Las **rutas** se protegen con `Route::middleware(['auth', 'admin'])`.
- En las **vistas**, `@auth`, `@guest` y `isAdmin()` deciden qué se ve, pero la seguridad está en el middleware.
