# Introducción al desarrollo web en el servidor

## 1. ¿Qué es un servidor web?

Un **servidor web** es un software (y también puede ser el hardware que lo aloja) que **recibe peticiones** de los navegadores web (clientes) y **responde** enviando recursos: archivos HTML, hojas de estilo, imágenes o incluso respuestas generadas dinámicamente.

!!! info "Servidor vs Cliente"
    - El **cliente** es normalmente el navegador del usuario.
    - El **servidor** es el software que atiende esas peticiones.

### 1.1 Ciclo de vida de una petición web

```
Navegador  ──── Solicita página (ej: index.php) ────►  Servidor
                                                         Procesa la petición
Navegador  ◄─── Devuelve HTML generado ──────────────  Servidor
```

Por ejemplo, al visitar `https://midominio.com/productos.php`:

- El navegador envía una petición al servidor.
- El servidor interpreta y ejecuta `productos.php`.
- Devuelve una página HTML generada en tiempo real.

---

## 2. Lenguajes del lado servidor

En este apartado hacemos un repaso a los lenguajes más comunes del lado servidor y del cliente. En este curso nos vamos a centrar en **PHP**, pero existen otros lenguajes.

En el cliente (*Frontend*) se ha estandarizado el uso de: `HTML`, `CSS` y `JavaScript`. Prácticamente todas las páginas web usan estos lenguajes. Cada uno cumple una función:

- **HTML** → Estructura y contenido
- **CSS** → Estilos visuales
- **JavaScript** → Comportamiento e interactividad

De esta manera se ha separado la estructura, los estilos y el comportamiento. Cada archivo cumple una funcionalidad concreta, lo que permite mantener el código más organizado y reutilizable.

En cambio, en el servidor (*Backend*) existen muchos lenguajes diferentes. Algunos de los más populares son:

| Lenguaje | Descripción |
|----------|-------------|
| **PHP** | Muy usado en desarrollo web |
| **Python** | Lenguaje versátil, con frameworks como Django o Flask |
| **Node.js** | JavaScript en el servidor |
| **Ruby** | Conocido por el framework Ruby on Rails |
| **Java** | Usado en grandes aplicaciones empresariales |

Normalmente para cada aplicación utilizamos un solo lenguaje en el servidor, aunque es posible combinar varios.

### 2.1 Cliente vs Servidor

| Cliente (Frontend) | Servidor (Backend) |
|--------------------|-------------------|
| HTML, CSS, JavaScript | PHP, Python, Node.js, Java... |
| Corre en el navegador | Corre en el servidor |
| Muestra datos | Accede a bases de datos |
| Interfaz de usuario | Lógica de negocio, seguridad |

### 2.2 ¿Por qué usamos lenguajes en el servidor?

Los lenguajes del lado servidor permiten (entre otras cosas):

- Leer y escribir datos en bases de datos.
- Controlar el acceso a usuarios (login, permisos).
- Generar contenido dinámico adaptado al usuario.
- Validar datos sensibles.

!!! info "Contenido estático vs dinámico"
    - `index.html` muestra siempre lo mismo.
    - `index.php` puede mostrar contenido diferente dependiendo de quién acceda.

Inicialmente los servidores web solo servían archivos estáticos (HTML, imágenes...). Con el tiempo se añadieron lenguajes del lado servidor para hacer las páginas más interactivas y personalizadas. De esta manera los `servidores web` cada vez que reciben la petición de un recurso dinámico (por ejemplo un archivo `.php`) lo procesan y generan el contenido en tiempo real. En este proceso pueden acceder a bases de datos, a otros servidores, leer archivos, etc., lo que genera una página web adaptada a cada usuario.

---

## 3. Lenguajes embebidos

Como ya hemos visto, existen gran variedad de lenguajes en el lado del servidor. Una característica que los diferencia son aquellos que son **embebidos** y los que no.

- **Embebidos**: `PHP`, `ASP`, `JSP`
- **No embebidos**: `Python`, `Ruby`, `Node.js`

Un lenguaje **embebido** es aquel que se puede incluir dentro de otro. Por ejemplo:

!!! example "Ejemplo de código embebido en PHP"
    ```php
    <h1>Bienvenido</h1>
    <p>Hoy es <?php echo date("d/m/Y"); ?></p>
    ```

Aquí PHP está embebido dentro del HTML. Esto permite generar contenido dinámico de forma sencilla.

!!! note "Ventajas del código embebido"
    - Ideal para principiantes
    - Permite ver rápidamente el resultado en el navegador
    - Muy utilizado en proyectos reales

---

## 4. Introducción a PHP

### 4.1 ¿Qué es PHP?

PHP es un lenguaje de programación del lado del servidor, diseñado para generar contenido web dinámico.

!!! info "Significado de PHP"
    PHP originalmente significaba "Personal Home Page", pero hoy se interpreta como **"PHP: Hypertext Preprocessor"**.

### 4.2 Historia breve

- Creado en 1994 por Rasmus Lerdorf.
- Desde entonces ha evolucionado mucho: orientación a objetos, frameworks modernos, tipado estricto...
- Hoy en día PHP sigue siendo uno de los lenguajes más usados para la web.

### 4.3 Características destacadas

- Fácil de aprender y usar.
- Multiplataforma y de código abierto.
- Muy rápido en ejecución.
- Amplia documentación.

### 4.4 Ejemplos de aplicaciones reales en PHP

| Aplicación | Descripción |
|------------|-------------|
| **Moodle** | Plataforma de educación online |
| **WordPress** | CMS más usado del mundo |
| **Prestashop** | Comercio electrónico |
| **phpBB** | Foros web |
| **Laravel** | Framework moderno para PHP |

### 4.5 Web oficial y documentación

- Web oficial: [https://www.php.net](https://www.php.net)
- Manual en español: [https://www.php.net/manual/es](https://www.php.net/manual/es)

---

## 5. Última versión estable de PHP

A la fecha de esta unidad (agosto 2025), la última versión estable es **PHP 8.4**.

Entre sus novedades destacadas:

- Tipado más estricto y seguro.
- Mejoras de rendimiento.
- Nuevas funciones y sintaxis moderna.

!!! info "¿Cómo saber la versión instalada?"
    Puedes ejecutar este código PHP:

    ```php title="version.php"
    <?php
    phpinfo();
    ?>
    ```

    O bien en consola:

    ```bash title="Comprobar versión en consola"
    php -v
    ```

---

## 6. Ejercicios propuestos

### Ejercicio 1: Diferencia entre cliente y servidor

Explica con tus propias palabras la diferencia entre:

- Lenguaje del lado cliente (ejemplo)
- Lenguaje del lado servidor (ejemplo)

??? success "Solución"
    - Un lenguaje del lado **cliente** se ejecuta en el navegador del usuario. Ejemplo: JavaScript para validar un formulario.
    - Un lenguaje del lado **servidor** se ejecuta en el servidor antes de que se genere la página. Ejemplo: PHP para obtener datos de una base de datos y mostrar un listado.

---

### Ejercicio 2: Petición HTTP

Describe con un esquema o dibujo cómo se produce una petición web desde que el usuario accede a una página hasta que se genera la respuesta.

??? success "Solución"
    - El usuario accede a una URL en el navegador.
    - El navegador (cliente) envía una petición HTTP al servidor.
    - El servidor recibe la petición, ejecuta el código PHP si es necesario.
    - El servidor genera una respuesta en HTML y la envía al navegador.
    - El navegador muestra la página al usuario.

---

### Ejercicio 3: Apps en PHP

Investiga uno de los siguientes proyectos PHP (Moodle, WordPress, Prestashop) y responde:

- ¿Para qué sirve?
- ¿Dónde se usa?
- ¿Por qué crees que está hecho en PHP?

??? success "Solución"
    - **WordPress** sirve para crear páginas web y blogs.
    - Se usa en millones de sitios web en todo el mundo.
    - Está hecho en PHP porque es un lenguaje muy extendido, de fácil despliegue en servidores y permite generar contenido dinámico fácilmente.

---

### Ejercicio 4: Versiones de PHP

Busca en la web oficial de PHP la última versión disponible.

- ¿Es la misma que tenemos instalada?
- ¿Qué mejoras trae respecto a versiones anteriores?

??? success "Solución"
    - Según [php.net](https://www.php.net), la última versión estable es PHP 8.3.
    - Si usamos `php -v`, podemos comprobar si tenemos esa misma versión.
    - Algunas mejoras son: mayor rendimiento, validación de tipos más estricta, y nuevas funciones del lenguaje que facilitan la escritura de código más limpio.

IES JUAN DE GARAY - 2º DWES (Curso original de Beteta adaptado por Eva María Gómez Abad )
