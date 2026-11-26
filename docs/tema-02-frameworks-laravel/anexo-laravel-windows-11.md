# Entorno de desarrollo e instalación de Laravel en Windows 11

## Introducción

El sistema de permisos en Windows es diferente al de Linux, lo que puede causar problemas al ejecutar Laravel en un entorno de desarrollo local. En algunos casos siguiendo el manual al iniciar `laravel` aparece una pantalla de error:

!!! failure "Error al iniciar Laravel"

    tempnam(): File created in the system's temporary directory

Este error se debe a que Laravel intenta crear archivos temporales en un directorio que no tiene los permisos adecuados en Windows. Para solucionar este problema, es necesario configurar el entorno de desarrollo de manera adecuada.

Existen dos maneras de aboardar este problema:

- La primera: es, seguir la instalación de `Laravel` tal y como la encontramos en los apuntes y una vez obtenido el error, dar los permisos necesarios a algunas de las carpetas de `Laravel` para que pueda crear los archivos temporales necesarios.

    - Quitar los permisos de solo lectura `ro` de las diferentes carpetas compartidas en el fichero `docker-compose.yml`. Y volver a crear los contenedores con `docker compose up -d --build`.
    - Ahora crear el fichero de base de datos `sqlite` dentro de la carpeta `database` de Laravel, y darle permisos de escritura a ese fichero.

        ```bash
        touch database/database.sqlite
        ```
    - Dar permisos a las carpetas `storage` y `bootstrap/cache` para que Laravel pueda escribir en ellas.

        ```bash
        chown -R www-data:www-data database storage bootstrap/cache
        chmod -R 775 storage database  bootstrap/cache
        ```
    - Por último volver a ejecutar el optimizador y las migraciones de Laravel.

        ```bash
        php artisan optimize:clear
        php artisan migrate
        ```

- La segunda: es, configurar el entorno de desarrollo de manera que al levantar los contenerores se creen las carpetas y ficheros con los permisos necesario.

En el presentee documento vamos a tratar la segunda opción, ya que es la más recomendable para evitar problemas de permisos en el futuro.

## Configuración del entorno de desarrollo

### Fichero `docker-compose.yml`

Añadimos la siguiente línea dentro del servicio `php` en el fichero `docker-compose.yml`:

```yaml
user: "${UID:-1000}:${GID:-1000}"
```

Esta línea indica que el contenedor se ejecutará con el mismo usuario y grupo que el host, lo que permitirá que los archivos creados dentro del contenedor tengan los permisos adecuados en Windows.

## Modificación del `Dockerfile`

Añadimos las siguiente líneas al final del `Dockerfile` del servicio `php`:

```dockerfile
...
COPY docker/php/entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

WORKDIR /var/www/html                  # Esta línea ya la teníamos. Pero nos sirve para ubicarnos.     

ENTRYPOINT ["/entrypoint.sh"]
...
```

De esta manera, estamos copiando un script de entrada personalizado al contenedor y dándole permisos de ejecución. Este script se encargará de crear los archivos y carpetas necesarios con los permisos adecuados cada vez que se inicie el contenedor.

### Creación del script de entrada `entrypoint.sh`

Creamos el fichero `entrypoint.sh` dentro de la carpeta `docker/php/` con el siguiente contenido:

```bash
#!/bin/sh
set -e

# Carpetas típicas de Laravel (no molesta si aún no es Laravel)
mkdir -p storage bootstrap/cache 2>/dev/null || true
chmod -R ug+rw storage bootstrap/cache 2>/dev/null || true

exec "$@"
```

Esta solución es simple y efectiva. Crea las carpetas `storage` y `bootstrap/cache` si no existen, y les da permisos de lectura y escritura para el usuario y el grupo. De esta manera, cada vez que se inicie el contenedor, se asegurarán de que estas carpetas tengan los permisos adecuados.

### Levantar los contenedores

Ahora ya podemos levantar los contenedores con la nueva configuración:

```bash
docker compose up -d --build
```

### Instalar la versión adecuada de `Laraval``

Una vez que los contenedores están levantados, podemos entrar al contenedor `php` para instalar la versión adecuada de `Laravel`:

!!! note "Importante"

    Se debe eliminar todo el contenido de la carpeta `src` (en Windows) o `html` (en el contenedor) antes de ejecutar el siguiente comando, ya que `composer create-project` no funciona si la carpeta no está vacía.

```bash
docker compose exec php bash
```

Dentro del contenedor, instalamos `Laravel` con el siguiente comando:

```bash
composer create-project laravel/laravel:^12.0 .
```

En caso de que queramos instalar la última versión de `Laravel` (que en el momento de escribir este documento es la 13), podemos omitir el `^12.0` y simplemente ejecutar:

```bash
composer create-project laravel/laravel .
```

!!! note "importante"

    Al instalar Laravel nos ha cerado las carpetas de nuevo **sin** los permisos adecuados, pero no pasa nada. Ahora, reiniciamos los contenedores para que el script de entrada se ejecute y cree las carpetas con los permisos adecuados:

```bash
docker compose restart
```

### Comprobar que Laravel funciona correctamente

Ahora ya podemos comprobar que Laravel funciona correctamente accediendo a `http://localhost:8080` en nuestro navegador. Deberíamos ver la página de bienvenida de Laravel sin ningún error relacionado con los permisos.
