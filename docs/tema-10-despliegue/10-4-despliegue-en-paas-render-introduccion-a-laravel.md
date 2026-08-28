# Despliegue en PaaS

En este apartado hemos elegido **Render** como plataforma de despliegue para nuestra aplicación. **Render** es una plataforma de alojamiento que permite desplegar aplicaciones web, bases de datos y servicios de forma sencilla y rápida.

Principales características de **Render**:

Despliegue **Render** permite desplegar automático aplicaciones automáticamente desde un repositorio de GitHub o GitLab.

Escalabilidad **Render** permite escalar aplicaciones fácilmente, ajustando los recursos según las necesidades.

Soporte para **Render** soporta una múltiples amplia variedad de lenguajes lenguajes de programación y frameworks.

Integración con **Render** permite crear y bases de datos gestionar bases de datos de forma sencilla.

Seguridad **Render** ofrece características de seguridad como HTTPS, firewall y protección DDoS.

Precios **Render** ofrece precios competitivos competitivos y un plan gratuito para proyectos pequeños.

Soporte para **Render** permite desplegar contenedores aplicaciones en contenedores Docker.

Monitorización y **Render** proporciona logs herramientas de monitorización y acceso a logs para depurar aplicaciones.

Las dos principales características que nos interesan son el despliegue automático y la integración con bases de datos. Y la posibilidad de crear una cuenta gratuita para proyectos pequeños.

En nuestro caso, para aprovechar el despliegue automático, es necesario que el repositorio de la aplicación esté en GitHub. Si no tienes una cuenta en GitHub, puedes crear una de forma gratuita.

Por tanto para empezar tenemos un proyecto de Laravel (recien creado, sin modificaciones), si desplegamos este proyecto, el despliegue nos servirá para cualquier proyecto de Laravel que tengamos, ya que el proceso de despliegue es el mismo. Eso sí, lo tenemos que tener en GitHub.

Podemos utilizar el proyecto creado para el curso, que está en el siguiente repositorio:

```text
https://github.com/
jbeteta-ies/
phpDeployTest
```

En este caso el repositorio es privado (cada alumno debe tener su propio repositorio ya sea púbico o privado). Lo conveniente sería clonar el repositorios y subirlo a una cuenta de GitHub propia, será necesario para poder hacer el despliegue.

## * Creación de la cuenta en

Para crear una cuenta en **Render**, sigue estos pasos:

1. **1**. Accede a la página de **Render**.

2. **2**. Haz clic en el botón "Sign Up" Registrarse) en la esquina superior derecha.

3. **3**. Puedes registrarte con tu cuenta de GitHub, GitLab o correo electrónico. En este caso, selecciona "Sign up with GitHub" Registrarse con GitHub).

4. **4**. Autoriza a **Render** para acceder a tu cuenta de GitHub.

5. **5**. Completa el proceso de registro siguiendo las instrucciones en pantalla.

6. **6**. Una vez registrado, serás redirigido al panel de control de **Render**.

Yo he creado mi cuenta con GitHub, por lo que me ha redirigido a la página de autorización de **Render** en GitHub. He autorizado a **Render** para acceder a mi cuenta de GitHub y he completado el registro.

**Render** ofrece múltiples opciones. Nosotros vamos a centrarnos en el despliegue de nuestra aplicación, utilizando las mínimas opciones de **Render**. Existen otros cursos de Despliegue de aplicaciones que seguro que profundizan más en las características de este tipo de servidores.

## Creación de la base de datos

Lo primero que vamos a hacer es crear la base de datos que vamos a utilizar en nuestra aplicación. Recordemos que nuestra aplicación utiliza una base de datos MySQL. Pero **Render** es su versión gratuita solo nos permite crear bases de datos PostgreSQL. Por tanto, vamos a crear una base de datos PostgreSQL.

Opciones seleccionadas:

Name Nombre de la `pg-` instancia

```text
larave
```

Postgress

```text
l
```

Database Nombre de la `larave` base de datos

```text
l
```

Username Nombre de `alumno` usuario de la base de datos

Region Región de la `Europe` base de datos

```text
(Frank
furt)
```

Postgres Versión de `16` Version PostgreSQL

Datalog Región para `EU` Region enviar datos al Datalog

Instance Tipo de `Free` Type instancia

Una vez seleccionadas las opciones, hacemos clic en el botón "Create Database" Crear base de datos). **Render** creará la base de datos y nos proporcionará las credenciales de acceso.

La base de datos se creará en la región de Europa Frankfurt) por defecto. Si necesitas cambiar la región, puedes hacerlo en las opciones avanzadas.

En el plan gratuito las base de datos caducan a los 30 días. Por tanto tenemos este tiempo para realizar el despliegue de nuestra aplicación y probarla. Posteriormente hay que volver a crear la base de datos o actualizar a un plan de pago.

Una vez creada, la aplicación nos mostrará las credenciales de acceso a la base de datos. Estas credenciales son necesarias para configurar nuestra aplicación Laravel.

Host `dpg-d0qdmsjuibrs73ehedm`

Port `5432`

Database `laravel_6idw`

Username `alumno`

Password `p3DmIdbZEBtmzcuMflpDHoE`

URL `postgresql:// Externa alumno:p3DmIdbZEBtmzcuMflpDHoEp d0qdmsjuibrs73ehedmg-a.frankfurt- postgres.com/laravel_6idw'

PSQL `PGPASSWORD=p3DmIdbZEBtm` Command

```text
Epxp8uToLh psql -h dpg-
d0qdmsjuibrs73ehedmg-a.
postgres.**Render**.com
```

```text
laravel_6idw
```

Nosotros necesitaremos el `Host`, `Port`, `Database`, `Username` y `Password` para configurar nuestra aplicación Laravel.

## Configuración de la aplicación Laravel

Antes de comenzar con la creación del servicio en **Render**, debemos preparar nuestra aplicación para que funcione correctamente en el entorno de **Render**. Para ello, debemos realizar los siguientes

1. **1**. Crear una rama `Render` esta rama será la que utilicemos para accionar el desplegue automático.

2. **2**. Configurar el archivo `.env` de Laravel.

3. **3**. Crear el archivo `Dockerfile` para la aplicación.

4. **4**. Crear el archivo `src/public/` `migrate.php` para ejecutar las migraciones de la base de datos.

Para ello y para no mezclar el código de la aplicación con el código de `Render` vamos a crear una carpeta llamada `/`

`Render` en la raíz de nuestro proyecto. En esta carpeta vamos a crear los archivos necesarios para el despliegue en **Render**.

### Crear la rama Render

El nombre de `Render` es completamenta aleatorio. Simplemente es tener una rama sobre la que guardar los cambios necesarios y activar el despliegue automático. Por ejemplo, en nuesta aplicación podemos trabajar con una rama `develop` y una rama `main`. Durante el desarrlllo y pruebebas de aplicaciones podemos trabajar en la rama `develop` y cuando tengamos una versión estable, la pasamos a `main`.

Cuando queramos desplegar la aplicación, simplemente hacemos un `merge` de `develop` a `Render` y **Render** se encargará de desplegar la aplicación automáticamente.

Yo voy a trabajar directamente en la rama `Render`. El manejo de ramas es algo que dejo para cursos más específicos sobre control de versiones y Git.

Lo primero que tenemos que hacer antes de crear la rama es tener nustro proyecto limpio, es decir, hemos hecho `commit` de todos los cambios y no tenemos cambios pendientes.

Crear la rama `Render` es sencillo, simplemente ejecutamos el siguiente comando en la terminal:

```text
git checkout -b Render
```

Esto creará una nueva rama llamada `Render` y nos cambiará a esa rama. A partir de ahora, todos los cambios que realicemos se guardarán en esta rama. Y si estamos en local y queremos subir los cambios a GitHub, simplemente hacemos un `push`:

```text
git push -u origin
Render
```

Para los que manejan vsCode si tenemos nuestro editor conectado podemos manejar las ramas y las actualizaiones desde el propio editor. En la parte inferior izquierda de la pantalla, podemos ver el nombre de la rama actual y al hacer clic en ella, podemos cambiar de rama o crear una nueva. Pero yo voy a dar las explicaiones utilizano la terminal.

Ahora creamos la carpeta `Render` en la raíz de nuestro proyecto:

```text
mkdir Render
```

Y dentro de esta carpeta vamos a crear los archivos necesarios para el despliegue en **Render**.

### Configuración del archivo.env

Como ya sabremos el fichero `.env` es el archivo de configuración de nuestra aplicación Laravel. En este archivo vamos a configurar la conexión a la base de datos que hemos creado en **Render**. Pero si tenemos bien configurado nuestro `.gitignore`, este archivo no se subirá al repositorio. Por tanto, vamos a crear un archivo `.env.**Render**` dentro de la carpeta `Render` que será el que subamos al repositorio y que **Render** utilizará para configurar la aplicación. De esta manera también podemos tener un `.env` local para nuestro desarrollo y un `.env.**Render**` para el despliegue en **Render**.

Creamos el archivo `.env.**Render**` dentro de la carpeta `Render`:

```text
touch Render/.env.Render
```

Y editamos el archivo para configurar la conexión a la base de datos. Vamos a utilizar las credenciales que nos ha proporcionado **Render** al crear la base de datos. El contenido del archivo `.env.Render` será el siguiente:

```text
APP_NAME=Laravel
APP_ENV=production
APP_KEY=base64:F3DJSvM2e
APP_DEBUG=false
APP_URL=https://phpdeplo

LOG_CHANNEL=stack
LOG_LEVEL=error

DB_CONNECTION=pgsql
DB_HOST=dpg-d0qdmsjuibrs
DB_PORT=5432
DB_DATABASE=laravel_6idw
DB_USERNAME=alumno
DB_PASSWORD=p3DmIdbZEBtm

BROADCAST_DRIVER=log
CACHE_DRIVER=file
QUEUE_CONNECTION=sync
SESSION_DRIVER=file
SESSION_LIFETIME=120

MAIL_MAILER=smtp
MAIL_HOST=smtp.mailtrap.
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS=hello@
MAIL_FROM_NAME="${APP_NA
```

Los valores utilizados en el archivo `.env.Render` son los que nos ha proporcionado **Render** al crear la base de datos. Cada alumno debe utilizar sus propias credenciales. Asegúrate de sustituir los valores por los que te ha proporcionado **Render** al crear tu base de datos.

Como vimos ya en el tema anterior, sería recomendabla modificar el acrivo anterior y utilizar `secretos` para las credenciales de la base de datos. Pero para simplificar el proceso, vamos a utilizar el archivo `.env.Render` directamente. En puntos anteriores ya hemos visto como utilizar secretos en GitHub Actions.

### Creación del archivo

`Dockerfile`

**Render** utiliza Docker para desplegar aplicaciones. Por tanto, necesitamos crear un archivo `Dockerfile` que contenga las instrucciones para construir la imagen de nuestra aplicación Laravel. Este archivo para que no tenga conflictos con otros ficheros de nuestra aplicación, lo vamos a crear también dentro de la carpeta `Render`.

```text
touch Render/Dockerfile
```

Creamos el archivo `Dockerfile` dentro de la carpeta `Render`:

```text
touch Render/Dockerfile
```

Y editamos el archivo para configurar la imagen de Docker. El contenido del archivo `Dockerfile` será el siguiente:

```text
FROM php:8.4-fpm

# Instalar depend
necesarias para M
etc.
RUN apt-get updat
get install -y \
git \
unzip \
curl \
libzip-dev \
libpng-dev \
libxml2-dev \
libonig-dev \
libssl-dev \
libpq-dev \
libmongoc-1.0
libjemalloc2
&& rm -rf /va
apt/lists/*

# Instalar extens
necesarias
RUN docker-php-ex
pdo pdo_mysql pdo
xml ctype zip

# Instalar MongoD
extension
# RUN pecl instal
&& docker-php-ext
mongodb

# Instalar Xdebug
RUN pecl install
docker-php-ext-en
xdebug

# Instalar Compos
```

```text
p
RUN curl -sS http
getcomposer.org/i
| php -- --instal
usr/local/bin --
filename=compose

# Establecer dire
trabajo
WORKDIR /var/www/

# Copiar el códig
desde /src del re
contenedor
COPY src/ /var/ww

# Copiar.env.**R
COPY
**Render**/.env.*
/var/www/html/.en

# Ejecutar compos
install sin depen
de desarrollo
RUN composer inst
optimize-autoload
dev

# Exponer puerto
el que uses)
EXPOSE 1000

# Comando para a
Laravel con servi
embebido PHP
CMD ["php", "arti
"serve", "--host=
"--port=1000"]
```

Analicemos paso a paso el contenido del `Dockerfile`:

```text
FROM php:8.4-fpm
```

Esta línea indica que estamos utilizando la imagen base de PHP 8.4 con FPM FastCGI Process Manager). Esta es una imagen oficial dePHP que incluye las extensiones necesarias para ejecutar aplicaciones Laravel.

```text
RUN apt-get update &&
apt-get install -y \
git \
unzip \
curl \
libzip-dev \
libpng-dev \
libxml2-dev \
libonig-dev \
libssl-dev \
libpq-dev \
libmongoc-1.0-0 \
libjemalloc2 \
&& rm -rf /var/lib/
apt/lists/*
```

Esta sección instala las dependencias necesarias para nuestra aplicación Laravel. Incluye herramientas como `git`, `unzip`, `curl` y las bibliotecas necesarias para las extensiones de PHP que vamos a utilizar. También limpia la caché de apt para reducir el tamaño de la imagen. En este caso ya hemos añadido las dependencias necesarias para `MongoDB` y `PostgreSQL`. Si en nuestro proyecto local utiliamos más dependencias, las debemos añadir aquí. Recordad que este Docker debe ser igual al que tenemos en nuestro proyecto local, para que no tengamos problemas de dependencias al desplegar la aplicación.

```text
RUN docker-php-ext-
install pdo pdo_mysql
pdo_pgsql xml ctype zip
```

En este caso he mantenido las extensiones para mysql y pgsql. Seguramente ya que en `Render` vamos a utilizar `PostgreSQL`, podemos eliminar la extensión `pdo_mysql` si no la vamos a utilizar en el proyecto. Pero si la tenemos en nuestro proyecto local, es mejor dejarla para evitar problemas de dependencias.

```text
# Instalar MongoDB
extension
# RUN pecl install
mongodb && docker-php-
ext-enable mongodb
```

Esta línea está comentada, pero si en nuestro proyecto utilizamos MongoDB, debemos descomentarla para instalar la extensión de MongoDB. Si no la utilizamos, podemos eliminar esta línea.

```text
# Instalar Xdebug
# RUN pecl install
xdebug && docker-php-
ext-enable xdebug
```

Esta línea también está comentada, no es normal que utilicemos `Xdebug` en producción.

```text
# Instalar Composer
RUN curl -sS https://
getcomposer.org/
installer | php -- --
install-dir=/usr/local/
bin --filename=composer
```

Esta línea instala Composer, el gestor de dependencias de PHP. Esto nos permitirá instalar las dependencias de nuestra aplicación Laravel. Recordad que la carpeta `vendor` no se sube al repositorio, por lo que debemos instalar las dependencias en el contenedor.

```text
# Establecer directorio
de trabajo
WORKDIR /var/www/html
```

Esta línea establece el directorio de trabajo dentro del contenedor. Todas las siguientes instrucciones se ejecutarán en este directorio.

```text
# Copiar el código
Laravel desde /src del
repo al contenedor
COPY src/ /var/www/html/
```

Esta línea copia el código de nuestra aplicación Laravel desde la carpeta `src` del repositorio al directorio de trabajo del contenedor. Asegúrate de que tu código Laravel está en la carpeta `src` del repositorio. 'src\' es la carpeta donde debemos tener nuestro proyecto de Laravel. Si hemos cambiado el nombre de la carpeta, debemos cambiar esta línea para que apunte a la carpeta correcta.

```text
# Copiar.env.Render
COPY Render/.env.Render
/var/www/html/.env
```

Esta línea copia el archivo `.env.Render` que hemos creado anteriormente al directorio de trabajo del contenedor y lo renombra a `.env`. Esto es necesario para que Laravel pueda leer la configuración de la base de datos y otras variables de entorno.

Estamos pasando datos con información sensible al contenedor. Esto normalmente no se hace así, se deben crear unas variables de entorno en el servidor y no pasar los datos directamente al contenedor. Pero de momento elegimos la opción más simple.

```text
# Ejecutar composer
install sin dependencias
de desarrollo
RUN composer install --
optimize-autoloader --
no-dev
```

Esta línea ejecuta Composer para instalar las dependencias de nuestra aplicación Laravel. Utiliza la opción `--optimize-` `autoloader` para optimizar el autoloading y `--no-dev` para no instalar las dependencias de desarrollo. Esto es importante para reducir el tamaño de la imagen y mejorar el rendimiento en producción.

```text
# Exponer puerto 1000 (o
el que uses)
EXPOSE 1000
```

Esta línea expone el puerto 1000 del contenedor. Este es el puerto en el que Laravel servirá la aplicación. Asegúrate de que este puerto está configurado correctamente en tu aplicación Laravel.

1000 es el puerto que pide `Render` para exponer la aplicación, no está elegido al azar. Para modificar el puerto debes mirar la documentación de **Render** y ver si se puede modificar.

```text
# Comando para arrancar
Laravel con servidor
embebido PHP
CMD ["php", "artisan",
"serve", "--
host=0.0.0.0", "--
port=1000"]
```

Esta línea define el comando que se ejecutará cuando se inicie el contenedor. En este caso, estamos utilizando el servidor embebido de Laravel para servir la aplicación en el puerto 1000. Esto es suficiente para desplegar una aplicación Laravel en **Render**.

### Creación del archivo

`migrate.php`

Nuestro servidor, como nos pasará en la mayoría de los servidores gratuitos, no permite ejecutar comandos de Artisan directamente. Por tanto, debemos crear un archivo PHP que se encargue de ejecutar las migraciones de la base de datos. Este archivo lo vamos a llamar `migrate.php` y lo vamos a crear dentro de la carpeta `src/public/`. Es importante que esté en esta carpeta para que podamos acceder desde el navegador y ejecutar las migraciones.

De momento este fichero solo nos permite ejecutar las migraciones de la base de datos, hacer un `fresh` o un `reset`. Si fuera necesario ampliarlo, por ejemplo para ejecutar `db:seed` o `queue:work`, deberíamos añadir las opciones necesarias en el fichero.

Creamos el archivo `migrate.php` dentro de la carpeta `src/public/`:

```text
<?php
ini_set('display_
ini_set('display_
error_reporting(

use Illuminate\Fo
use Illuminate\Co
ArtisanConsole;

$basePath = __DI

require $basePath

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
echo "Ejecutado c
echo $output;
```

Le hemos añadido una clave secreta para evitar que cualquier persona pueda ejecutar las migraciones. Para ejecutar las migraciones, debemos acceder a la URL

```text
https://phpdeploytest-
sh8f.on**Render**.com/
migrate.php?
key=123456&action=migrate
```

Si queremos hacer un `fresh` o un `reset`, simplemente cambiamos el valor de `action` a `fresh` o `reset`.

## Creación del servicio en

Ahora que tenemos nuestra aplicación configurada y lista para desplegar, vamos a crear el servicio en **Render**. Para ello, sigue estos pasos:

Primero en el panel de control de **Render**, haz clic en el botón "New" Nuevo y selecciona "Web Service" Servicio web).

En la siguiente página me debe dejar seleccionar el repositorio de GitHub donde tengo mi proyecto. En este caso, selecciono el repositorio `phpDeployTest` que he creado anteriormente. Si no ves tu repositorio, asegúrate de que has autorizado a **Render** para acceder a tu cuenta de GitHub y que el repositorio es público o tienes acceso a él.

Una vez seleccionado el repositorio, **Render** te pedirá que configures el servicio. Aquí debes completar los siguientes campos:

Name Nombre del `phpDep` servicio

```text
loyTes
t
```

Language Lenguaje de `Docker` programación PHP no está)

Branch Rama del `Render` repositorio que se utilizará para el despliegue

Region Región donde `Europe` se desplegará

```text
(Frank
```

el servicio

```text
furt)
```

Root Directorio raíz `.` Directory del servicio

Dockerfile Ruta al archivo `./` Path Dockerfile

```text
**Rend
er**/
Docker
file
```

Intace type Tipo de `Free` instancia

En la parte inferior de la página, encontrarás una sección llamada "Environment Variables" Variables de entorno). Aquí podemos añadir las variables de entorno, incluso nos permite subirlas desde un archico `.env`. En nuestro caso, vamos a subir el archivo `.env.**Render**` que hemos creado anteriormente. Dejamos esta opción vacía y para pruebas posteriores ya que es más segura que tener nuestro archivo con datos sensibles en el repositorio.

Una vez completados todos los campos, haz clic en el botón "Create Web Service" Crear servicio web). **Render** comenzará a construir la imagen de Docker y desplegará tu aplicación.

Nos pueden aparecer otros campos como:

Docker Comando `` Command para construir la imagen de Docker

Pre deploy Comando que `` Command se ejecutará antes de desplegar la aplicación

Auto Activar el `on` deploy despliegue

```text
Commit
```

automático al hacer push a la rama seleccionada Deploy URL del `https:` hook webhook para

```text
//
```

el despliegue

```text
api.**
```

automático

```text
Render
**.com
/
deploy
/srv-
xxxxxx
```

Principalmente no hay que tocar nada de esto. **Render** se encargará de construir la imagen de Docker y desplegar tu aplicación automáticamente. Como en la últia línea de `Dockerfile` hemos indicado que se ejecute el comando `php` `artisan serve`, **Render** iniciará el servidor embebido de Laravel en el puerto

1000. 

## * Puesta en marcha del despliegue

Tenemos dos opcioenes para iniciar el despliegue de nuestra aplicación:

1. **1**. Despliegue manual: Para ello, en el panel de control de **Render**, haz clic en el botón "Manual Deploy" Despliegue manual) y selecciona la rama `Render`. **Render** comenzará a construir la imagen de Docker y desplegará tu aplicación.

2. **2**. Despliegue automático: Si has activado el despliegue automático, **Render** desplegará tu aplicación automáticamente cada vez que hagas un push a la rama `Render`. Para ello, simplemente haz un commit y push a la rama `Render` y **Render** se encargará del resto.

El primer despliegue puede tardar un poco más de lo habitual, ya que **Render** debe construir la imagen de Docker y desplegar tu aplicación por primera vez. Los siguientes despliegues serán más rápidos, ya que **Render** utilizará la caché de Docker.

Durante el despliegue, **Render** mostrará el progreso en tiempo real en el panel de control. Una vez que el despliegue se haya completado, podrás acceder a tu aplicación Laravel en la URL proporcionada por **Render**.

En la imagen anterior podemos ver el progreso del despliegue de nuestra aplicación. Una vez que el despliegue se haya completado, **Render** nos proporcionará una URL para acceder a nuestra aplicación.

Cuando el despliegue termine (podemos seguirlo en los logs), **Render** nos mostrará la URL de nuestra aplicación. En mi caso, la URL es:

```text
https://phpdeploytest-
sh8f.on**Render**.com
```

Esta es la imagen del proyecto inicial de Laravel que hemos desplegado en **Render**. Si accedemos a la URL, veremos la página de bienvenida de Laravel.

## Primera migración

Una vez que hemos desplegado nuestra aplicación, es importante realizar la primera migración de la base de datos para crear las tablas necesarias. Para ello, debemos acceder al archivo `migrate.php` que hemos creado anteriormente.

Para ejecutar las migraciones, simplemente accedemos a la URL

```text
https://phpdeploytest-
sh8f.onRender.com/
migrate.php?
key=123456&action=migrate
```

Recuerda que debes sustituir `123456` por la clave secreta que has definido en el archivo `migrate.php`. Si no lo haces, obtendrás un error de acceso denegado.

Si todo ha ido bien, deberías ver un mensaje indicando que se ha ejecutado el comando `migrate` y por tanto las tablas necesarias se han creado en la base de datos. Si hay algún error, se mostrará un mensaje de error con los detalles del problema.

