# Actividad del Tema 2 - Introducción a Laravel

Ahora que hemos aprendido sobre la
estructura y los elementos básicos de un
proyecto Laravel, es momento de poner
en práctica lo aprendido con una actividad
que consolidará tus conocimientos.
Objetivo de
la actividad
El objetivo de esta
actividad es que puedas crear un proyecto
Laravel desde cero, entender su
estructura y familiarizarte con los
elementos clave que componen un
proyecto Laravel. Puedes crear un nuevo
proyecto o utilizar el proyecto que has
creado siguiendo el tema.
Instrucciones
��� :
• Si no lo has hecho ya, crea un
nuevo proyecto Laravel utilizando
2.5 Estructura de un proyecto
Laravel

---

Composer. Puedes hacerlo
ejecutando el siguiente comando
en tu terminal:
• Asegúrate de reemplazar
nombre-del-proyecto con el
nombre que desees para tu
proyecto. Sigue las instrucciones
del tema 2.4 para crear el
proyecto dentro del entorno
Docker.
��� :
• Crea tres vistas Blade en la
carpeta resources/views .
Nómbralas home.blade.php ,
about.blade.php y
contact.blade.php .
• Cada vista debe contener un título
y un párrafo descriptivo. Por
ejemplo, en home.blade.php .
Deben ser páginas HTML5
básicas.
��� :
• Abre el archivo routes/web.php
y define rutas para cada una de
las vistas que has creado. Utiliza
composer createproject laravel/
laravel nombredel-proyecto

---

el método Route::view() para
asociar cada ruta con su
respectiva vista. Por ejemplo:
��� :
• Entrega un documento pdf con el
siguiente contenido:
• Nombre, apellidos y título de la
actividad.
• El código de las rutas
definidas en routes/
web.php .
• El código HTML de cada una
de las vistas creadas.
• Pantallazos de las tres vistas
renderizadas en el navegador.
��� :
• La actividad será evaluada en
base a los siguientes criterios:
• Estructura adecuada de las
Route::view('/',
'home')-
>name('home');
Route::view('/
about', 'about')-
>name('about');
Route::view('/
contact',
'contact')-
>name('contact');

---

vistas Blade. �20%�
• Definición correcta de las
rutas en web.php . �30%�
• Acceso a las vistas a través de
las rutas definidas. �30%�
• Presentación clara y
organizada del documento
entregado. �20%�
Busca en el aula virtual la tarea de la
actividad del tema 2 y sube el documento
pdf con la actividad realizada.

IES JUAN DE GARAY - 2º DWES (Curso original de Beteta adaptado por Eva María Gómez Abad )
