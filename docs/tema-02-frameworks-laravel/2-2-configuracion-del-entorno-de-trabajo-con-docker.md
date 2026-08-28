Perfecto, ahora te entiendo 👍  
Quieres **UN ÚNICO .md completo**, limpio, continuo y listo para meter directamente en MkDocs como página (sin cortes, sin explicaciones, sin fragmentación).

Te lo construyo ya 👇 (esto lo puedes copiar y pegar tal cual en un `.md`)

***

# Configuración del entorno de trabajo con Docker

## Introducción

Para todo desarrollo web, es fundamental contar con un entorno de trabajo adecuado que facilite la creación, prueba y despliegue de aplicaciones.  
Al final, lo que necesitamos básicamente es:

* Un servidor web (Nginx o Apache)
* Un intérprete de PHP (PHP-FPM)
* Un sistema de gestión de bases de datos (MySQL, PostgreSQL, etc.)

Existen múltiples soluciones para crear este entorno de trabajo:

* Instalar cada componente por separado → complejo y propenso a errores
* Usar soluciones empaquetadas (XAMPP, WAMP, MAMP) → fáciles pero poco flexibles
* Usar máquinas virtuales o contenedores (Docker) → entornos aislados y reproducibles ✅

En este curso vamos a utilizar **Docker**.

Docker es una plataforma que permite crear, desplegar y ejecutar aplicaciones en contenedores.  
Los contenedores son entornos ligeros que incluyen todo lo necesario para ejecutar una aplicación.

***

## Requisitos previos

* Docker Desktop
* Visual Studio Code
  * Extensión recomendada: Docker

### Base de datos (MySQL)

Opciones:

* Crear contenedor propio
* Reutilizar uno existente
* Definirlo en `docker-compose.yml` ✅ (opción usada)

Credenciales usadas en el curso:

* Host: `localhost` o `host.docker.internal`
* Usuario: `alumno`
* Password: `alumno`
* Base de datos: `test`

***

## Paso 1. Preparación del proyecto

```bash
mkdir mi-proyecto-php
cd mi-proyecto-php
```

Crear estructura:

```bash
mkdir -p docker/mysql/init docker/nginx docker/php docker/scripts src

touch docker-compose.yml
touch docker/mysql/init/01-grants.sql
touch docker/nginx/default.conf
touch docker/php/Dockerfile
touch docker/php/xdebug.ini
touch docker/scripts/.gitkeep
touch src/index.php
```

Estructura final:

```text
.
├── docker-compose.yml
├── docker/
│   ├── nginx/
│   │   └── default.conf
│   ├── php/
│   │   ├── Dockerfile
│   │   └── xdebug.ini
│   ├── scripts/
│   │   └── .gitkeep
│   └── mysql/
│       └── init/
│           └── 01-grants.sql
└── src/
    └── index.php
```

***

## Paso 2. Configuración de docker-compose.yml

```yaml
services:
  php:
    build:
      context: .
      dockerfile: docker/php/Dockerfile
    working_dir: /var/www/html
    volumes:
      - ./src:/var/www/html
      - ./docker/scripts:/scripts
    environment:
      XDEBUG_CONFIG: "client_host=host.docker.internal client_port=9003"
      MYSQL_HOST: mysql
      MYSQL_USER: alumno
      MYSQL_PASSWORD: alumno
      MYSQL_DB: test
    extra_hosts:
      - "host.docker.internal:host-gateway"
    depends_on:
      - mysql

  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
    depends_on:
      - php
    volumes:
      - ./src:/var/www/html:ro
      - ./docker/nginx/default.conf:/etc/nginx/conf.d/default.conf:ro

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: administrador
      MYSQL_DATABASE: test
      MYSQL_USER: alumno
      MYSQL_PASSWORD: alumno
    ports:
      - "3306:3306"
    volumes:
      - dbdata:/var/lib/mysql
      - ./docker/scripts:/scripts
      - ./docker/mysql/init:/docker-entrypoint-initdb.d:ro
    tmpfs:
      - /tmp

volumes:
  dbdata:
```

***

## Paso 3. Configuración de Nginx

Archivo: `docker/nginx/default.conf`

```nginx
server {
    listen 80;
    server_name localhost;

    root /var/www/html;
    index index.php index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME /var/www/html$fastcgi_script_name;
    }
}
```

***

## Paso 4. Dockerfile para PHP

Archivo: `docker/php/Dockerfile`

```dockerfile
FROM php:8.3-fpm

RUN apt-get update && apt-get install -y \
    git curl unzip zip libzip-dev libpng-dev \
    && docker-php-ext-install pdo_mysql zip gd \
    && pecl install xdebug \
    && docker-php-ext-enable xdebug \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

COPY --from=composer:2 /usr/bin/composer /usr/bin/composer
COPY docker/php/xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

WORKDIR /var/www/html

CMD ["php-fpm"]
```

***

## Configuración Xdebug

Archivo: `docker/php/xdebug.ini`

```ini
xdebug.mode=debug
xdebug.start_with_request=yes
xdebug.client_host=host.docker.internal
xdebug.client_port=9003
xdebug.log_level=0
```

***

## Paso 5. Archivo PHP de prueba

Archivo: `src/index.php`

```php
<?php
$host = getenv('MYSQL_HOST');
$db = getenv('MYSQL_DB');
$user = getenv('MYSQL_USER');
$pass = getenv('MYSQL_PASSWORD');

try {
    $pdo = new PDO("mysql:host=$host;dbname=$db", $user, $pass);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

    echo "<p>Conexión exitosa a la base de datos!</p>";
} catch (PDOException $e) {
    echo "<p>Error al conectar: " . $e->getMessage() . "</p>";
}

phpinfo();
?>
```

***

## Paso 6. Permisos MySQL

Archivo: `docker/mysql/init/01-grants.sql`

```sql
GRANT ALL PRIVILEGES ON *.* TO 'alumno'@'%' IDENTIFIED BY 'alumno' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

***

## Paso 7. Levantar el entorno

Primera vez:

```bash
docker compose up --build
```

Modo segundo plano:

```bash
docker compose up -d
```

Parar entorno:

```bash
docker compose down
```

Eliminar imágenes:

```bash
docker compose down --rmi all
```

***

## Paso 8. Verificación

Abrir en navegador:

```
http://localhost:8080
```

Resultado esperado:

```
Conexión exitosa a la base de datos!
```

***

## Paso 9. Acceso a MySQL

```bash
docker compose exec mysql mysql -uroot -p
```

Ejemplo:

```sql
SHOW DATABASES;
```

***

## Paso 10. Configuración de depuración en VSCode

Archivo: `.vscode/launch.json`

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Listen for XDebug",
      "type": "php",
      "request": "launch",
      "port": 9003,
      "pathMappings": {
        "/var/www/html": "${workspaceFolder}/src"
      },
      "log": true
    }
  ]
}
```

***

## Uso de Visual Studio Code con Docker

Con la extensión Docker puedes:

* Ver contenedores activos
* Arrancar y parar servicios
* Ver logs
* Ejecutar comandos dentro del contenedor
* Abrir el contenedor en VSCode

***

## Conclusión

Has configurado un entorno completo con:

* Nginx
* PHP-FPM
* MySQL
* Docker

Ventajas:

* Entorno aislado
* Reproducible
* Portátil
* Escalable

La carpeta principal de trabajo es:

```
src/
```

donde desarrollarás tu aplicación (Laravel).

