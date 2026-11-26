# Problemas frecuentes al montar el entorno

Esta página recoge los errores que aparecen con más frecuencia al seguir los apartados 2.2 (entorno Docker) y 2.3 (instalación de Laravel) en un ordenador con Windows, con su causa y su solución. Si te sale un error, búscalo aquí antes de nada.

!!! tip "Cómo buscar"

    Pulsa `Ctrl+F` en el navegador y escribe una parte del mensaje de error, por ejemplo `port is already allocated` o `readonly database`.

## Terminal y comandos

### `"touch" no se reconoce como un comando` · `mkdir -p` crea carpetas raras

**Causa:** los comandos del curso son de Linux (bash) y los estás escribiendo en CMD o PowerShell.

**Solución:** abre la terminal de VS Code con **Git Bash** (flecha ˅ junto al **+** de la terminal). Si prefieres PowerShell, en el paso 1 del 2.2 tienes los comandos equivalentes.

### `"grep" no se reconoce como un comando interno o externo`

**Causa:** `grep` es de Linux; no existe en CMD ni en PowerShell.

**Solución:** en CMD usa `findstr` (por ejemplo `docker compose exec php php -i | findstr xdebug.mode`) o ejecuta el `grep` dentro del contenedor: `docker compose exec php sh -c "php -i | grep xdebug.mode"`.

### `Error response from daemon: No such container: mysql`

**Causa:** se han mezclado las dos formas de entrar en un contenedor.

| Comando | Qué nombre necesita | En nuestro entorno |
|---|---|---|
| `docker compose exec` | el **servicio** del `docker-compose.yml` | `php`, `nginx`, `mysql` |
| `docker exec -it` | el **contenedor** real (`docker ps`) | `<carpeta>-php-1`, `<carpeta>-mysql-1`… |

**Solución:** usa siempre la primera forma, que no depende del nombre de tu carpeta:

```bash
docker compose exec mysql mysql -uroot -p
docker compose exec php bash
```

### `"composer" no se reconoce como un comando` · `"php" no se reconoce` · `artisan` no funciona

**Causa:** PHP y Composer están instalados **dentro del contenedor `php`**, no en Windows. La terminal de VS Code, al abrirse, está en Windows.

**Solución:** entra antes en el contenedor y ejecuta ahí los comandos:

```bash
docker compose exec php bash      # el indicador cambia a root@...:/var/www/html#
composer --version
php artisan route:list
exit                              # para volver a Windows
```

O, sin entrar, anteponiendo `docker compose exec php` a cada comando: `docker compose exec php composer --version`.

## Docker y MySQL

### `Bind for 0.0.0.0:3306 failed: port is already allocated`

**Causa:** otro MySQL ya usa el puerto 3306 de tu ordenador: el de XAMPP o el contenedor de otro proyecto.

**Solución:** mira quién es con `docker ps --filter "publish=3306"`. Páralo o, mejor, cambia el puerto de este entorno en `docker-compose.yml` a `"3307:3306"`. Laravel no se entera, porque se conecta por dentro de Docker. Lo mismo vale para el puerto `8080`.

### El contenedor `mysql` sale como `Exited (1)` · `ERROR 1064 ... near 'IDENTIFIED BY 'alumno''`

**Causa:** el `01-grants.sql` usa una sintaxis de MySQL 5.7. En MySQL 8 no se puede poner `IDENTIFIED BY` dentro de un `GRANT`.

**Solución:** deja el archivo así y borra la base de datos a medio crear para que el script se vuelva a ejecutar:

```sql
GRANT ALL PRIVILEGES ON *.* TO 'alumno'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

```bash
docker compose down -v
docker compose up -d
```

### La página de prueba dice `getaddrinfo for mysql failed: Name or service not known`

**Causa:** el contenedor de MySQL no está en marcha, así que PHP no encuentra ningún equipo llamado `mysql`.

**Solución:** `docker compose ps -a` para ver su estado y `docker compose logs mysql --tail 30` para ver por qué se ha parado (normalmente, uno de los dos errores anteriores).

!!! info "Para ver qué pasa en cualquier contenedor"

    ```bash
    docker compose ps -a              # estado de los tres contenedores
    docker compose logs mysql --tail 30   # últimas líneas del registro de uno
    ```

## Laravel

### Error 500: `tempnam(): file created in the system's temporary directory` · `Permission denied` · `attempt to write a readonly database`

**Causa:** Laravel necesita escribir en `storage/`, `bootstrap/cache/` y, mientras use SQLite, en `database/`. Composer creó esas carpetas como `root` y PHP atiende las páginas como `www-data`.

**Solución:**

```bash
docker compose exec php chown -R www-data:www-data storage bootstrap/cache database
docker compose exec php chmod -R 775 storage bootstrap/cache database
```

Si sigue fallando en Windows, `docker compose exec php chmod -R 777 storage bootstrap/cache database` (solo en tu ordenador, nunca en producción).

### Sale un `index.php` que no es el tuyo · los cambios no se ven

**Causa:** el archivo está fuera de `src/`. Solo `src/` se comparte con los contenedores (`./src:/var/www/html`); lo que pongas en otra carpeta, por ejemplo `docker/php/`, no se ejecuta nunca.

## VS Code, Git y Xdebug

### El panel de Git dice «The folder currently open doesn't have a Git repository», pero el repositorio existe

**Causa:** VS Code está abierto **como administrador** (el título dice `[Administrator]`). Git no se fía de una carpeta cuyo dueño es otro usuario (*dubious ownership*) y VS Code la trata como si no tuviera repositorio.

**Solución:** abre VS Code de forma normal. Si el acceso directo tiene marcado «Ejecutar como administrador» (Propiedades → Compatibilidad), desmárcalo. No pulses «Initialize Repository»: crearías otro repositorio encima.

### Los commits salen con otro nombre en GitHub («No contributors»)

**Causa:** el email de tu configuración de Git no es uno de los de tu cuenta de GitHub.

**Solución:**

```bash
git config --global user.name "tu-usuario-de-github"
git config --global user.email "el-email-de-tu-cuenta-de-github"
```

### Antes de cada commit: ¿se va a subir algo que no debe?

Revisa la lista de cambios (o `git status`). **No** deben aparecer `src/vendor/` ni `src/.env`: el `.gitignore` de Laravel ya los excluye. Si aparecen, es que estás haciendo el commit desde una carpeta equivocada.

### En «Run and Debug» no aparece la opción PHP · VS Code ofrece una extensión de SQL

**Causa:** falta la extensión **PHP Debug** (autor *Xdebug*). Si además tienes abierto un `.sql`, VS Code intenta depurar ese archivo.

**Solución:** instala PHP Debug (`Ctrl+Shift+X`) y crea el `launch.json` con un archivo `.php` abierto (paso 10 del 2.2). Cierra el aviso de SQL: no hace falta.

### Pulso F5, recargo la página y no se detiene en el punto de interrupción

Comprueba, por este orden:

1. **`docker/php/xdebug.ini` no está vacío** (contenido en el paso 2 del 2.2). Si lo cambias, reconstruye: `docker compose up -d --build php`.
2. **Xdebug está en modo depuración:** `docker compose exec php php -r "echo ini_get('xdebug.mode');"` debe responder `debug`.
3. **VS Code está escuchando:** abajo, en la barra de estado, aparece «Listen for XDebug».
4. **El punto rojo está en un archivo de `src/`**, que es el que se ejecuta, y no en una copia en otra carpeta.
