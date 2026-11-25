# Actividad: login y roles en tu tienda

!!! quote "Autoría"

    Actividad nueva para 2º DAW del IES Juan de Garay (Eva María Gómez Abad), sobre el proyecto de productos del curso [*Introducción a Laravel*](https://jbeteta-ies.github.io/2025_Laravel_32/) de **Javier Beteta**.

## Objetivo

Añadir a tu tienda en Laravel el registro y el login de usuarios, y conseguir que **solo el administrador** pueda crear, editar y borrar productos. Sigue la lección [Login y roles en Laravel con Breeze](11-1-login-y-roles.md).

## Lo que tienes que hacer

1. **Guarda tu trabajo** con un commit y copia `routes/web.php` y tu layout antes de instalar Breeze (paso 0).
2. **Instala Breeze** con vistas Blade y comprueba que puedes registrarte, iniciar sesión y cerrarla (paso 1).
3. **Recupera** tus rutas de productos y haz que tus vistas usen `layouts.tienda` (paso 2).
4. **Añade el rol**: migración con la columna `role` (valor por defecto `user`), método `isAdmin()` en `User` y un administrador creado con el seeder (paso 3).
5. **Crea el middleware** `EsAdmin` y regístralo con el alias `admin` (paso 4).
6. **Protege las rutas**: `index` y `show` públicas; `create`, `store`, `edit`, `update` y `destroy` solo para `auth` + `admin` (paso 5).
7. **Adapta las vistas**: botones de administración solo para el administrador; enlaces de login y registro para visitantes; botón de cerrar sesión para usuarios con sesión (paso 6).
8. **Prueba** los seis casos de la tabla del paso 7.

## Ampliaciones (opcionales)

- **A1.** Muestra en el menú el nombre del usuario y una etiqueta «Administrador» cuando lo sea.
- **A2.** Crea una página `/admin/usuarios`, protegida con `auth` y `admin`, que liste los usuarios con su rol.
- **A3.** Escribe un test de Feature que compruebe que un usuario normal recibe 403 al entrar en `/product/create`. Pista: `$this->actingAs($user)->get('/product/create')->assertForbidden();`

## Entrega en Aules

Sube un archivo **`Readme.txt`** con:

1. El enlace a tu repositorio de GitHub (con el commit de esta actividad).
2. El email y la contraseña del administrador que crea tu seeder.
3. La tabla del paso 7 con una columna más, **«¿Funciona?»**, rellenada por ti.
4. Las ampliaciones que hayas hecho, si las hay.

## Cómo se evalúa

| Criterio | Peso |
|---|---|
| Breeze instalado; registro, login y logout funcionan; la tienda se sigue viendo bien | 20 % |
| Columna `role`, método `isAdmin()` y administrador creado con seeder (sin `role` en `$fillable`) | 20 % |
| Middleware `EsAdmin` correcto y registrado con el alias `admin` | 20 % |
| Rutas protegidas correctamente (403 para usuario normal, login para visitante) | 25 % |
| Vistas: botones según el rol, `@auth` / `@guest` y cerrar sesión con `POST` | 15 % |
