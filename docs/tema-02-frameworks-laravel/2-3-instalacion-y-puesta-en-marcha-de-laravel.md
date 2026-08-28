# Instalación y puesta en marcha de Laravel

## Introducción

En este apartado vamos a instalar Laravel dentro del entorno de desarrollo que hemos preparado previamente.

Utilizaremos Composer para crear un nuevo proyecto Laravel y realizaremos las configuraciones necesarias en el servidor Nginx para que nuestra aplicación funcione correctamente.

El objetivo es tener un proyecto de Laravel completamente operativo y accesible desde el navegador.

> **Recordatorio importante**
>
> Si has seguido los temas 2.2 y 2.3, ya deberías tener el entorno Docker configurado. Y la estructura de carpetas y ficheros básicos creados.

## Preparar la carpeta de trabajo

Antes de instalar Laravel, es importante preparar correctamente la carpeta donde se va a ubicar el proyecto.

- Dirígete a la carpeta src/ del proyecto clonado.

```bash
cd src
```

- Si existe un archivo index.php de prueba (utilizado previamente para verificar PHP), elimínalo:

```bash
rm index.php
```

Esto nos asegura que la carpeta src/ esté vacía y lista para recibir el proyecto Laravel.

Laravel necesita que la carpeta de instalación esté vacía para crear correctamente su estructura de archivos.

## Crear el proyecto Laravel

### Acceder al contenedor PHP

Para instalar Laravel correctamente, es necesario ejecutar Composer.

Abre una terminal en Visual Studio Code. (O utiliza Docker Desktop si lo prefieres).

Ejecuta el siguiente comando para acceder al contenedor dk_php:

```bash
docker exec -it
laravel_app_php_1
bash
```

o

```bash
docker compose
exec php bash
```

Esto abrirá una consola dentro del contenedor.

### Comprobar que Composer está instalado

Antes de continuar, vamos a asegurarnos de que Composer está correctamente instalado dentro del contenedor.

Escribe:

```bash
composer --version
```

Deberías ver algo parecido a:

```text
Composer version
2.x.x 2025-XX-XX
```

Si no ves la versión de Composer, o recibes un error, revisa que tu contenedor PHP haya sido construido correctamente a partir del Dockerfile que instala Composer.

### Ir a la carpeta de trabajo src/

Una vez dentro del contenedor, asegúrate de estar en la carpeta correcta (/var/www/html): Puedes utilizar el comando pwd para saber en qué carpeta te encuentras.:

```bash
pwd
```

Si no estás en la carpeta correcta:

```bash
cd /var/www/html
```

Esta carpeta es donde se montó tu proyecto (src/) desde tu máquina local.

### Instalar Laravel

Ahora sí, desde dentro de /var/www/html, ejecuta el siguiente comando para instalar Laravel:

```bash
composer create-project laravel/
laravel:^12.0 .
```

El punto ( . ) al final del comando indica que Laravel debe instalarse en la carpeta actual (/var/www/html).

El comando laravel/laravel:^12.0 asegura que se instale la última versión estable de Laravel 12. En el momento de revisar la documentación del curso ha salido Laravel 13, pero para evitar problemas de compatibilidad con el curso, se recomienda instalar Laravel 12.

Composer descargará todo el framework, instalará las dependencias necesarias, y configurará el proyecto inicial. Puede tardar un poco, ten paciencia.

## Resumen operativo

| | |
|---|---|
| Entrar al contenedor PHP | `docker exec -it docker_php bash` |
| Comprobar Composer | `composer --version` |
| Ir a la carpeta de trabajo | `cd /var/www/html` |
| Instalar Laravel | `composer create-project laravel/laravel .` |

Al finalizar el proceso, la carpeta src/ tendrá la siguiente estructura principal:

```text
src/
├── app/
├── bootstrap/
├── config/
├── database/
├── public/
├── resources/
├── routes/
├── storage/
├── tests/
├── .env
├── artisan
└── composer.json
```

Dependiendo de tu conexión a internet y de la velocidad de tu equipo, la instalación puede tardar varios minutos.

## Configurar Nginx para Laravel

Laravel está diseñado para servir su aplicación a través de la carpeta public/. Por lo tanto, es necesario ajustar la configuración de Nginx para apuntar correctamente a esta carpeta.

Edita el archivo nginx/default.conf y modifica la directiva root:

**Antes:**

```nginx
root /var/www/html;
```

**Después:**

```nginx
root /var/www/html/public;
```

Además, asegúrate de que la regla de try_files esté configurada de esta forma:

```nginx
location / {
    try_files $uri $uri/ index.php?$query_string;
}

location ~ \.php$ {
    include fastcgi_params
    fastcgi_pass php
    fastcgi_index index.php
    fastcgi_param
    SCRIPT_FILENAME
    $document_root$fastcgi_script_name
}
```

Fíjate porque la línea fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name; ha cambiado y es crucial para el funcionamiento de Laravel. Si no la modificas, nginx no podrá encontrar los archivos PHP correctamente y devolverá un error 404.

Esta configuración permite que Laravel maneje las rutas dinámicas correctamente.

No debes servir directamente el contenido fuera de la carpeta public/ de Laravel. Es una mala práctica de seguridad.

## Levantar de nuevo los servicios Docker

Después de modificar la configuración de Nginx, es necesario reiniciar los contenedores Docker para aplicar los cambios.

Ejecuta los siguientes comandos desde la raíz del proyecto:

```bash
docker-compose down
docker-compose up -d
```

- docker-compose down detiene y elimina los contenedores actuales.
- docker-compose up -d vuelve a levantar los servicios en segundo plano.

## Verificar instalación de Laravel

Una vez que los servicios estén arriba, abre tu navegador y accede a la siguiente URL:

```text
http://localhost:8080
```

Deberías ver la pantalla de bienvenida de Laravel:

con el mensaje de "Let's get started" o similar.

Esto indica que Laravel está instalado correctamente y que la configuración del entorno es funcional.

Ahora que Laravel está instalado, el siguiente paso será aprender a trabajar con rutas, controladores y modelos dentro del framework.

IES JUAN DE GARAY - 2º DWES (Curso original de Beteta adaptado por Eva María Gómez Abad )
