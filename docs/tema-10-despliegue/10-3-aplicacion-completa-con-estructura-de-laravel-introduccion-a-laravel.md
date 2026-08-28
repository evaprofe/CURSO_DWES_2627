# Aplicación completa con estructura de laravel

En esta sección, vamos a crear una aplicación completa de Laravel, con su estructura de carpetas y archivos necesarios para su funcionamiento. Vamos a realizar el despliegue en un servidor remoto utilizando GitHub Actions y configurando los secretos necesarios para que la aplicación funcione correctamente.

EL proceso va a ser muy similar al que hemos visto en la unidad anterior, pero con algunas diferencias debido a la estructura de Laravel y a los archivos necesarios para su funcionamiento.

## Requisitos previos

Antes de comenzar, necesitamos tener instalado Docker y Docker Compose en nuestro sistema. También necesitamos tener una cuenta en GitHub para poder utilizar GitHub Actions y un servidor remoto donde desplegar la aplicación. En este caso, utilizaremos un servidor gratuito de `InfinityFree`, pero se puede utilizar cualquier otro servidor que soporte PHP, tenga acceso a una base de datos MySQL y conexión FTP.

### Clonado o creación del proyecto laravel

Como vimos en los primeros temas, vamos a arrancar el entorno de desarrollo, Instalaremos Laravel y comprobar que todo funciona correctamente en local antes de realizar el despliegue en el servidor remoto.

Podéis acudir al tema 2, puntos 2.2, 2.3 y

2. 4 para ver como crear el entorno de desarrollo con Docker, instalar Laravel y comprobar que todo funciona correctamente en local.

También podéis descargar el proyecto inicial:

```text
git clone https://
github.com/jbeteta-ies/
phpDeployTest.git
cd phpDeployTest
```

Si habéis clonado el proyecto lo primero que debéis eliminar son los ficheros `*.txt` que encontraréis en las carpetas `mysql/data/` y 'mysql/tmp/'. Estos ficheros están únicamente para que al clonar el proyecto se suban las carpetas ya que si están vacías git no las sube.

### Crear el entorno de desarrollo

Creamos el entorno de desarrollo:

```text
docker-compose up -d --
build
```

Y accedemos al contenedor, para instalar las dependencias de Laravel con Composer:

```text
docker compose exec php
bash
composer install
```

Si al ejecutar el comando anterior, nos da un error de timeout, es posible que sea debido a que Docker no tiene suficiente memoria asignada. Para solucionarlo, podemos aumentar la memoria asignada a Docker desde la configuración de Docker Desktop. Lo ideal es asignar al menos 4 GB de memoria.

```text
# Establecer un
tiempo de espera
mayor para Composer
(2000 segundos)
composer config --
global process-
timeout 2000
composer install
```

### Crear y configurar la base de datos

Creamos la base de datos `laravel` y le damos permisos al usuario `alumno` si este no los tiene ya creados:

Para conectar:

```text
# la constraseña de
`root` con la
configuración del curso
es 'administrador'
docker compose exec
mysql mysql -u root -p
```

```text
CREATE DATABASE IF NOT
EXISTS laravel;
GRANT ALL PRIVILEGES ON
*.* TO 'alumno'@'%' WITH
GRANT OPTION;
FLUSH PRIVILEGES;
```

### Crear y modificar el fichero .env

Lo primero será entrar en el fichero `.env` y configurar los datos de conexión a la base de datos. En este caso, como estamos en local, utilizaremos los datos que tenemos configurados en el archivo.

Entramos en el contenedor `php`:

```text
docker compose exec php
bash
# copiamos el fichero
.env.example a.env
cp.env.example.env
# recuerda que nunca se
debe subir el fichero
.env a un repositorio
público
# Generamos la clave de
la aplicación
php artisan key:generate
```

Modificamos el fichero `.env`, con la siguiente configuración (la Key es la que nos ha generado el comando anterior, no hay que copiar la que aparece aquí):

```text
APP_NAME=Laravel
APP_ENV=local
APP_KEY=base64:xxxxxxxxxxxx
APP_DEBUG=true
APP_URL=http://localhost:80
# Base de datos........
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=alumno
DB_PASSWORD=alumno
```

### Migraciones

Hacemos la migración de la base de datos:

```text
php artisan migrate
```

### Verificar que todo funciona correctamente

Comprobamos que todo funciona correctamente accediendo a la siguiente URL

```text
http://localhost:8080
```

Ahora debemos crear un repositorio en GitHub y subir el proyecto a GitHub. Ya que necesitaremos `github actions` para realizar el despliegue en el servidor remoto.

## Estructura de carpetas en local

Si hemos conseguido arrancar nuestro proyecto de laravel, la estructura de carpetas debería ser similar a la siguiente Sólo las de laravel, el resto no importan para subir a producción):

```text
phpDeployTest/
```

├── `src/` │ ├── `app/` │ ├── `config/` │ ├── `database/` │ ├── `public/` ├──

```text
index.php
```

│ ├── `resources/` │ ├── `routes/` │ ├── `storage/` │ ├── `tests/` │ ├── `vendor/` │ ├── `.env` │ ├── `artisan` │ ├── `composer.json` │ ├── `composer.lock` │ ├── `phpunit.xml`

## Estructura de carpetas en el servidor remoto

Recordemos que la situación es igual a la del punto anterior 10.2, tenemos un servidor remoto que solo nos permite subir los archivos a una carpeta llamada `htdocs` y no podemos cambiarla. Por lo que tenemos que adaptar la estructura de carpetas de laravel a esta situación.

La estructura de carpetas en el servidor remoto será similar a la de local, pero con algunas diferencias. La carpeta `public` se moverá a `htdocs` para que sea accesible desde la web. La estructura del servidor remoto será la siguiente:

```text
phpDeployTest/
```

├── `htdocs/` │ ├── `src/` │ ├── `app/` │ ├── `config/` │ ├── `database/` │ ├── `resources/` │ ├── `routes/` │ ├── `storage/` │ ├── `tests/` │ ├── `vendor/` │ ├── `.env` │ ├── `artisan` │ ├──

```text
composer.json
```

│ ├──

```text
composer.lock
```

│ ├── `phpunit.xml` ├── `index.php`

La carpeta `public` desaparece y su contenido se mueve a `htdocs` para que sea accesible desde la web.

## Despliegue en producción

### Prepacación de GitHub

Previamente a poder realizar el despliegue en producción, necesitamos configurar los secretos en GitHub. Estos secretos son necesarios para que el flujo de trabajo de GitHub Actions pueda acceder a ellos y realizar el despliegue correctamente.

Aunque en el punto anterior 10.2 ya manejamos secretos, esn este tema vamos a necesitar más secretos, información que necesiteramos traspasar durante el despliegue al servidor remoto (ìnfinityFree en este caso). Como no queremos que esta información sensible esté en el repositorio, la guardamos en los `secrets` de GitHub.

La información necesaria sobre `FTP` la tenemos en el panel de control de `InfinityFree` y la información sobre la base de datos la tenemos en la sección de `MySQL Databases` del panel de control de `InfinityFree`. Tenemos que crear la base de datos antes de continuar.

Al crear la base de datos, nos proporcionan la información necesaria para conectarnos a la base de datos. Esta información es la que necesitamos guardar en los secretos de GitHub. La contraseña es la misma que utilizamos para acceder al panel de control de `InfinityFree`.

Recordar que estos secretos los podemos configurar en el repositorio de GitHub, en la sección de `Settings` **>** `Secrets` `and variables` **>** `Actions` **>** `Repository secret`.

`APP_K` `APP_K` Clave de la `ba` aplicación

```text
EY EY 4:
```

Laravel.

```text
xx
xx
xx
xx
xx
xx
xx
xx
=
```

`APP_U` `APP_U` URL de la `ht` aplicación

```text
RL RL:/
```

en

```text
la
```

producción.

```text
el
si
e-
te
wu
.c
```

`DB_CO` `DB_CO` Tipo de `my` conexión a

```text
NNECT NNECT
```

la base de

```text
ION ION
```

datos.

`DB_HO` `DB_HO` Host de la `sq` base de

```text
ST ST 0.
```

datos.

```text
in
fr
co
```

`DB_PO` `DB_PO` Puerto de la `33` base de

```text
RT RT
```

datos.

`DB_DA` `DB_DA` Nombre de `if` la base de

```text
TABAS TABAS 01
```

datos.

```text
E E 44
ra
```

`DB_US` `DB_US` Usuario de `if` la base de

```text
ERNAM ERNAM 01
```

datos.

```text
E E 44
```

`DB_PA` `DB_PA` Contraseña `**` de la base

```text
SSWOR SSWOR **
```

de datos.

```text
D D
```

Asegúrate de que los datos son correctos y que la base de datos está configurada correctamente. Estos datos son necesarios para que Laravel pueda conectarse a la base de datos en producción. Los datos mostrados son un ejemplo de un servidor de pruebas gratuito, `InfinityFree`, que permite desplegar aplicaciones web de forma gratuita.

### Archivo de despliegue deploy.yaml

Para realizar el despliegue en producción, utilizaremos GitHub Actions para automatizar el proceso. El archivo `deploy.yaml` se encargará de subir los archivos al servidor remoto.

Si han clonado el repositorio el archivo debe estar en la carpeta `.github/` `workflows/deploy.yaml`. Si no es así, lo creamos.

El contenido del archivo `deploy.yaml` es el siguiente:

```text
name: Deploy to
on:
push:
branches:
- main

jobs:
ftp-deploy:
runs-on: ubun

steps:
# 1. Checko
código
- name: Che
uses: act
checkout@v3

# 2. Instal
Composer (solo pa
vendor la primera
- name: Set
with composer
uses: shi
setup-php@v2
with:
php-ve
extensi
mbstring, intl
tools:

- name: Ins
dependencies with
working-d
./src
run: comp
install --no-dev
autoloader

# 3. Subir
```

```text
archivos públicos
- name: Upl
files to htdocs/
uses: Sam
FTP-Deploy-Action
with:
server
secrets.FTP_SERV
usernam
secrets.FTP_USERN
passwo
secrets.FTP_PASSW
local-d
public/
server
htdocs/
dangero
slate: false #
archivo de estado
log-lev

# 4. Prepa
código fuente sin
"public"
- name: Pre
without public
run: |
mkdir t
shopt
cp -r s
(public) temp_src
shell: ba

# 5. Subir
del proyecto Lara
htdocs/src/
- name: Upl
(except public) t
src/
uses: Sam
FTP Deploy Action
```

```text
FTP-Deploy-Action
with:
server
secrets.FTP_SERV
usernam
secrets.FTP_USERN
passwo
secrets.FTP_PASSW
local-d
temp_src/
server
htdocs/src/
dangero
slate: false #
borrar y mantiene
incremental
log-lev

# 6. Crear
.env
- name: Cre
file in temp fold
run: |
mkdir t
echo
"APP_NAME=Laravel
temp_env/.env
echo
"APP_ENV=producti
temp_env/.env
echo "A
secrets.APP_KEY }
temp_env/.env
echo
"APP_DEBUG=false
temp_env/.env
echo "A
secrets.APP_URL }
temp_env/.env
echo "
temp_env/.env
```

```text
echo
"LOG_CHANNEL=stac
temp_env/.env
echo "
temp_env/.env
echo
"DB_CONNECTION=${
secrets.DB_CONNEC
temp_env/.env
echo "
secrets.DB_HOST }
temp_env/.env
echo "
secrets.DB_PORT }
temp_env/.env
echo
"DB_DATABASE=${{
secrets.DB_DATABA
temp_env/.env
echo
"DB_USERNAME=${{
secrets.DB_USERNA
temp_env/.env
echo
"DB_PASSWORD=${{
secrets.DB_PASSWO
temp_env/.env
echo "
temp_env/.env
echo
"BROADCAST_DRIVER
temp_env/.env
echo
"CACHE_DRIVER=fil
temp_env/.env
echo
"FILESYSTEM_DISK=
temp_env/.env
echo
"QUEUE_CONNECTION
temp env/.env
```

```text
temp_env/.env
echo
"SESSION_DRIVER=f
temp_env/.env
echo
"SESSION_LIFETIM
temp_env/.env
echo "
temp_env/.env
echo
"MAIL_MAILER=smtp
temp_env/.env
echo
"MAIL_HOST=smtp.m
>> temp_env/.env
echo
"MAIL_PORT=2525"
temp_env/.env
echo
"MAIL_USERNAME=nu
temp_env/.env
echo
"MAIL_PASSWORD=nu
temp_env/.env
echo
"MAIL_ENCRYPTION=
temp_env/.env
echo
"MAIL_FROM_ADDRES
temp_env/.env
echo
"MAIL_FROM_NAME=\
${APP_NAME}\"" >>
temp_env/.env

# 7. Subir
tocar el resto
- name: Upl
via FTP manually
run: |
curl -T
temp env/ env ftp
```

```text
temp_env/.env ftp
secrets.FTP_SERV
htdocs/src/.env \
--use
secrets.FTP_USERN
secrets.FTP_PASSW
```

También en GitHub necesitamos crear un **token de acceso personal** PAT) con permisos de `repo` y `workflow` para que el flujo de trabajo pueda acceder a los secretos y realizar el despliegue. Si lo hiciste en el punto anterior 10.2 no es necesario que lo vuelvas a hacer.

### Explicación del archivo deploy.yaml

Este archivo YAML define un **flujo de** **trabajo (workflow)** de **GitHub Actions** que se ejecuta automáticamente cada vez que se hace un push a la rama `main`. El objetivo es **desplegar el proyecto PHP/** **Laravel** en el hosting gratuito **InfinityFree** mediante FTP.

Configuración general

- **name: Deploy to InfinityFree** Nombre del flujo de trabajo.

- **on: push -> branches: main** Indica que el flujo se ejecuta

**cambios a la rama principal ( main**.

- **runs-on: ubuntu-Latest** GitHub usa una **máquina virtual Ubuntu** para realizar todas las acciones.

Paso 1 Checkout del código

```text
- name: Checkout code
uses: actions/
checkout@v3
```

**Qué hace**: Descarga el código fuente del repositorio en la máquina virtual de GitHub. Es el paso inicial para que las siguientes acciones puedan acceder a los archivos del proyecto.

**Referencia**: actions/checkout

Paso 2 **-** Instalar PHP y Composer

```text
- name: Set up PHP with
composer
uses: shivammathur/
setup-php@v2
```

**Qué hace**: Instala una versión específica de **PHP (8.4)** y **Composer**, la herramienta que maneja las dependencias de PHP. También activa extensiones necesarias ( `mbstring`, `intl`) para Laravel.

**Objetivo**: preparar el entorno para poder ejecutar `composer install`.

**Referencia**: shivammathur/setup-php

Paso 3 **-** Instalar dependencias del proyecto

```text
- name: Install
dependencies with
composer
working-directory:./
src
run: composer install
--no-dev --optimize-
autoloader
```

**Qué hace**: Ejecuta `composer install` dentro de la carpeta `src` para generar la carpeta `vendor` (que contiene las librerías PHP. La opción `--no-dev` evita instalar dependencias de desarrollo, y `--` `optimize-autoloader` mejora el rendimiento del autoload de Laravel.

Paso 4 Subir archivos públicos a `/` `htdocs/`

```text
- name: Upload public
files to htdocs/
uses: SamKirkland/FTP-
Deploy-Action@4.1.0
```

**Qué hace**: Sube **solo la carpeta public/** al directorio `/htdocs/` del servidor (la raíz pública de InfinityFree). Usa el **plugin** **FTP Deploy Action** que sincroniza los archivos por FTP.

- `${{ secrets.FTP_SERVER }}` → dirección del servidor FTP

- `${{ secrets.FTP_USERNAME }}`, `${{ secrets.FTP_PASSWORD }}` → credenciales guardadas en GitHub Secrets

```text
dangerous-clean-slate:
```

`false` significa que **no borra todo antes** **de subir**, sino que solo actualiza los archivos modificados.

Paso 5 Preparar el código fuente sin la carpeta `public`

```text
- name: Prepare src
without public
run: |
mkdir temp_src
shopt -s extglob
cp -r src/!(public)
temp_src/
```

**Qué hace**: Crea una carpeta temporal ( `temp_src`) que **copia todo el contenido** **de src excepto lacarpeta public/**. Esto se hace porque los archivos públicos ya se subieron en el paso anterior y no deben duplicarse.

```text
shopt -s extglob
```

activa una opción de Bash para usar patrones avanzados de exclusión.

Paso 6 **-** Subir el resto del código a `/` `htdocs/src/`

```text
- name: Upload src
(except public) to
htdocs/src/
uses: SamKirkland/FTP-
Deploy-Action@4.1.0
```

**Qué hace**: Sube el contenido de `temp_src/` al servidor en la ruta `/` `htdocs/src/`. Ahí queda alojada la parte “interna” del proyecto Laravel (controladores, modelos, vistas, etc.).

**Importante**: De nuevo, `dangerous-` `clean-slate: false` asegura que **solo** **se sincronizan los cambios**, sin borrar todo.

Paso 7 **-** Crear y subir el archivo `.env`

Primero se **genera un archivo.env** **temporal** con las variables del entorno (datos de configuración y credenciales), luego se **sube manualmente por FTP**.

```text
- name: Create.env file
in temp folder
run: |
mkdir temp_env
echo
"APP_NAME=Laravel" >
temp_env/.env
...
```

y después:

```text
- name: Upload.env via
FTP manually
run: |
curl -T./
temp_env/.env ftp://${{
secrets.FTP_SERVER }}/
htdocs/src/.env \
--user "${{
secrets.FTP_USERNAME }}:
${{
secrets.FTP_PASSWORD}}"
```

- Crea un `.env` nuevo con las variables que Laravel necesita APP_KEY, DB, MAIL, etc.), usando los valores almacenados como **secrets** en GitHub.

- Sube ese archivo al servidor mediante **curl** y el protocolo FTP.

**Por qué no se sube con el resto**: El `.env` contiene **información sensible** (contraseñas, claves API, etc.), y se maneja aparte para mayor control.

Resultado final

Al terminar, el servidor de InfinityFree tendrá esta estructura:

```text
/htdocs/
```

│ ├── `index.php` ←

```text
Archivos accesibles
desde la web
```

│ └── `src/` ←

```text
Código del proyecto
Laravel
```

├── `app/` ├── `vendor/` ├── `routes/` ├── `.` └── `.env` ←

```text
Configuración del
entorno
```

El sitio queda actualizado automáticamente cada vez que se hace un push a la rama `main`.

## Página principal de laravel

la página `index.php` de laravel se encuentra en la carpeta `public`, por lo que al acceder a la URL del servidor remoto deberíamos ver la página de bienvenida de laravel. En el servidor la tendremos en `htdocs`. Hemos tenido que modificar la página `index.php` que viene por defecto con laravel para que funcione en nuestro servidor remoto y en local sin tener que mantener dos versiones diferentes.

En este caso el script lo que hace es detectar si estamos en un entorno local o en producción, y cargar el autoloader de Composer y el bootstrap de Laravel desde la ruta correcta. El contenido del archivo `index.php` es el siguiente:

```text
<?php
use
Illuminate\Founda
use Illuminate\Ht

define('LARAVEL_S
microtime(true))

// Detectar ento
producción según
carpeta o archivo
if (file_exists(_
vendor/autoload.p
// Estamos e
(estructura están
$basePath =
'/../');
//echo "Ejecu
local<br>";
} elseif (file_ex
src/vendor/autolo
// Estamos e
InfinityFree con
modificada
$basePath =
'/src/');
//echo "Ejecu
producción<br>";
} else {
die("No se ha
el entorno de eje
realpath(__DIR__
}

// Modo mantenimi
if (file_exists($
$basePath. 'sto
maintenance.php'
require $mai
```

```text
q $
}

// Autoloader de
require $basePath
autoload.php';

// Bootstrap Lara
la petición
/** @var Applicat
$app = require_on
bootstrap/app.php

// Capturar la pe
manejarla
$response = $app
$request = Re
);

$response->send(

$app->terminate($
$response);
```

## Realizar el commit y push a GitHub

Una vez que tenemos el archivo `deploy.yaml` y hemos configurado los secretos en GitHub, podemos realizar el commit y push a GitHub para que se inicie el flujo de trabajo de GitHub Actions y se realice el despliegue en el servidor remoto.

```text
git add.
git commit -m
"Despliegue inicial de
Laravel"
git push origin main
```

Esta acción debe iniciar el flujo de trabajo de GitHub Actions y realizar el despliegue en el servidor remoto. Podemos ver el progreso del despliegue en la sección de `Actions` del repositorio de GitHub. La primera vez que se realiza el despliegue puede tardar un poco más, ya que tiene que instalar las dependencias de Laravel con Composer y aunque el peso no es grande son muchos archivos.

`GitHub Actions` lleva un control de los archivos subidos y de las versiones, por lo que en los siguientes despliegues solo subirá los archivos que hayan cambiado, haciendo el proceso mucho más rápido.

Una vez podamos ver en `github` `actions` que el flujo de trabajo ha terminado correctamente, podemos pasar al siguiente punto ya qu antes de acceder a la aplicación necesitamos ejecutar las migraciones para crear las tablas necesarias en la base de datos.

## Migraciones

Llegados a este punto, tenemos la aplicación desplegada y la base de datos creada. Pero la base de datos está vacía ya que necesitamos ejecutar las migraciones para crear las tablas necesarias. Tenemos dos maneras de hacerlo, la primera manualmente, crear las tablas y datos necesarios en la base de datos remota. Esta la descartamos porque no automatiza nada y está expuest a múltiples errores humanos. La segunda es crear un script que se encargue de ejecutar las migraciones de Laravel de forma automática. Para ello, creamos un archivo `migrate.php` en la carpeta `htdocs` con el siguiente contenido:

```text
<?php
ini_set('display_
ini_set('display_
error_reporting(

use Illuminate\Fo
use Illuminate\Co
ArtisanConsole;

require __DIR__

// Definir la cla
define('SECRET_K

// Validar key y
$key = $_GET['key
$action = $_GET[

if ($key !== SECR
http_response
exit('Acceso
}

$allowedActions =

if (!in_array($ac
http_response
exit('Acción
}

$basePath = __DI

// Bootstrap de l
$app = require_on
app.php';

$kernel = $app-
>make(Illuminate\
```

```text
// Capturar la sa
ob_start();

try {
switch ($acti
case 'mig
$exit
['--force' => tru
brea
case 'res
$exit
>call('migrate:re
brea
case 'fre
$exit
>call('migrate:f
brea
}
} catch (Exceptio
ob_end_clean
http_response
exit("Error e
$e->getMessage()
}

$output = ob_get_

// Enviar resulta
header('Content-T
echo "Ejecutando
echo $output;
```

Este script permite ejecutar las migraciones, con los siguientes parámetros:

- `key`: Clave de acceso para ejecutar el script. En este caso, `123456`.

- `action`: Acción a realizar. Puede ser `migrate`, `reset` o `fresh`. Para ejecutar las migraciones, simplemente accedemos a la URL del script con los parámetros necesarios. Por ejemplo:

```text
http://localhost:8080/
migrate.php?
key=123456&action=migrate
```

Esto ejecutará las migraciones en la base de datos configurada en el archivo `.env` de Laravel. Asegúrate de que la base de datos está configurada correctamente y que los datos son correctos.

Igual que hemos hecho con migrate podemos crar otros scripts o ampliar este para ejecutar comandos de Artisan, como `db:seed` o `cache:clear`, siguiendo la misma estructura.

Por último ir a la base de datos de InfinityFree y comprobar que las tablas se han creado correctamente.

Una vez que hemos ejecutado las migraciones, podemos acceder a la aplicación y ver que las tablas se han creado correctamente en la base de datos. Si todo ha ido bien, deberíamos ver la página de bienvenida de Laravel y no deberíamos tener ningún error.

Por último accediendo a nuestro dominio deberíamos ver la página de bienvenida de Laravel: https://laravel-simple- test.wuaze.com

## protección del sitio

El servidor elegido `InfinityFree` nos obliga a colocar los archivos en la carpeta `htdocs`. Por lo que quedan expuestos, cosa que va en contra de la filosofia de Laravel. Ahora nos queda pendiente como protecger la carpeta `src` y su contenido. Solo deben ser accesibles los scripts que se encuentren en la carpeta `htdocs`. Para ello, podemos crear un archivo `.htaccess` en la carpeta `htdocs` con el siguiente contenido:

## Resumen.

Hemos creado una aplicación Laravel completa, con su estructura de carpetas y archivos necesarios para su funcionamiento. Hemos realizado el despliegue en un servidor remoto utilizando GitHub Actions y hemos configurado los secretos necesarios para que la aplicación funcione correctamente.

No perdamos de vista el objetivo inicial, que es conseguir que la aplicación que tenemos en local y que estamos desarrollando, se pueda desplegar en un servidor remoto de forma automática y sin necesidad de realizar cambios manuales en el código. Esto nos permitirá tener una aplicación siempre actualizada y lista para ser utilizada por los usuarios.

Aun que nosotros por abreviar hemos desplegado los cambios en la rama `main`, Lo ideal sería tener una rama de desarrollo y una rama de producción. De esta forma, podemos realizar los cambios en la rama de desarrollo y, una vez que estén listos, hacer un merge a la rama de producción para que se desplieguen automáticamente en el servidor remoto. Pero eso queda para los cursos de `git`.

