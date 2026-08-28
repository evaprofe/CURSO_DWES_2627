# Desarrollo de un CRUD

## Introducción a CRUD

El término **CRUD** corresponde a las operaciones básicas que se realizan en la mayoría de las aplicaciones que gestionan datos:

- reate Crear Insertar nuevos datos.

- ead Leer Consultar y visualizar datos.

- pdate Actualizar): Modificar datos existentes.

- elete Eliminar): Borrar datos.

En Laravel, la implementación de un CRUD completo nos permite comprender cómo los **Modelos**, **Controladores** y **Vistas** interactúan entre sí para ofrecer una experiencia de usuario completa.

Dominar el desarrollo de CRUDs es básico para cualquier programador web, ya que casi todas las aplicaciones web tienen que manejar datos de algún tipo, y estas son las acciones fundamentales que se realizan sobre esos datos.

Además, este tema nos va a servir para aprender el funcionamiento de los formularios en Laravel.

## Rutas Dinámicas y Controladores

### Parámetros dinámicos en rutas

En Laravel, podemos definir rutas que aceptan **parámetros dinámicos**. Estos parámetros permiten que una misma ruta atienda solicitudes distintas según el valor proporcionado. Podemos permitirnos el símil de que el nombre de la ruta es el nombre de la función y los parámetros son los argumentos que recibe esa función. Por tanto nos va a permitir generar una respuesta dinámica en función de los parámetros que reciba.

```text
<?php
Route::get('/
nota/{id}',
function ($id) {
return
"Mostrando la
nota con ID:
$id";
});
```

En este caso, `id` es un parámetro dinámico. Si accedemos a `http://` `localhost:8080/nota/5`, Laravel mostrará: "Mostrando la nota con ID 5".

```text
<?php
Route::get('/
usuario/{id}/
nota/{nota_id}',
function ($id,
$nota_id) {
return
"Usuario $id -
Nota $nota_id";
});
```

El orden de los parámetros en la URL debe coincidir exactamente con el orden de los parámetros en la función anónima o el controlador.

### Crear un circuito MVC rápido para rutas dinámicas

**Modelo**: Vamos a utilizar una tabla `notes` con los campos:

- `id` (entero, autoincremental)

- `title` (string)

- `description` (text)

- `date` (date)

- `done` (boolean)

Vamos a crear la migración de la nota y el controador:

```text
php artisan
make:migration
create_notes_table
php artisan
make:model Note
```

Podemos crearla migración y la nota con un solo comando:

```text
php artisan
make:model Note -m
```

Ahora vamos al archivo de migración `database/migrations/` `xxxx_xx_xx_create_notes_table.ph` `p` y añadimos los campos:

```text
<?php
use
Illuminate\Databa
use Illuminate\Da
use Illuminate\Su

class CreateNotes
{
/**
* Run the mi
*
* @return vo
*/
public functi
{
Schema::c
(Blueprint $table
$tabl
$tabl
$table->t
$table->d
$table->b
>default(false);
$table->t
});
}
public function d
{
Schema::drop
}
```

Por último, ejecutamos la migración:

```text
php artisan migrate
```

Si tenemos algún problema porque no hemos creado la base de datos desde 0, podemos eliminar las migraciones anteiores con:

```text
php artisan
migrate:reset
```

Ahora vamos a implementar el modelo `Note`. Vamos a definir los campos que se pueden asignar en masa y los que no:

```text
<?php
namespace App\Mode
use
Illuminate\Databas

class Note extends
{
protected $fil
'description', 'da
protected $gua
}
```

- `$fillable`: Define qué campos se pueden asignar en masa.

- `$guarded`: Define qué campos **no** se pueden asignar.

```text
php artisan
make:controller
NoteController
```

Ahora codificamos el controlador `app/` `Http/Controllers/` `NoteController.php`:

```text
<?php
public function
show($id)
{
return
view('notes.show',
compact('id'));
}
```

Ahora que ya tenemos el controlador y el método que manejará la ruta, y el modelo que se conectará con la base de datos, vamos a definir la ruta en `routes/` `web.php`:

```text
<?php
use
App\Http\Controlle

Route::get('/note/
[NoteController::c
>name('note.show'
```

Por último nos queda crear la vista que mostrará la información de la nota con el ID recibido:

```text
<h1>Detalle de
Nota</h1>
<p>El ID de la
nota es: {{ $id
}}</p>
```

Con esto, accediendo a `/note/5` veremos "El ID de la nota es: 5".

La función `compact('variable')` crea un array asociativo `['variable' =>` `$variable]` que puede ser pasado a la vista. Es una forma rápida y limpia de pasar datos. Sólo la puedo utilizar si el valor de la clave es el mismo que el nombre de la variable.

### Parámetros opcionales y valores por defecto

Podemos definir parámetros **opcionales** añadiendo un signo de interrogación `?`:

```text
<?php
Route::get('/
saludo/
{nombre?}',
function ($nombre
= 'Invitado') {
return "Hola,
$nombre";
});
```

- Si accedemos a `/saludo/Laura`, veremos "Hola, Laura".

- Si accedemos a `/saludo`, veremos "Hola, Invitado".

- El parámetro opcional debe ser el último de la URL.

- Hay que asignar un **valor por defecto** en la función.

### Importancia del orden de las rutas

Laravel evalúa las rutas en el **orden en** **que se definen**.

```text
<?php
Route::get('/
nota/nueva',
function() {
return 'Crear
nueva nota'; });
Route::get('/
nota/{id}',
function($id) {
return "Nota ID:
$id"; });
```

- Primero debe definirse `/nota/nueva` porque si no, Laravel intentará interpretar `nueva` como un `id`.

- El orden correcto es siempre de **rutas** **más especificas a más generales**.

Primero define todas las rutas fijas y luego las rutas con parámetros dinámicos.

## Desarrollo del CRUD para notas

### Listar todas las notas

Primero creamos la ruta y el método para listar todas las notas.

```text
<?php
use
App\Http\Controlle

Route::get('/',
[NoteController::c
>name('note.index
```

```text
<?php
use App\Models\Note;
public function
index()
{
$notes =
Note::all();
return
view('notes.index',
compact('notes'));
}
```

- `@foreach` se utiliza para recorrer elementos, pero no gestiona si el array está vacío.

- `@forelse` permite recorrer elementos y además definir qué hacer si no hay elementos.

Ejemplo:

```text
@forelse ($notes
as $note)
<p>{{ $note-
>title }}</p>
@empty
<p>No hay
notas
disponibles.</p>
@endforelse
```

En el ejemplo anterior, si `$notes` está vacío, se mostrará "No hay notas disponibles."

```text
<!DOCTYPE html>
<html lang="es">
<head>
<meta
charset="UTF-8">

<title>@yield('ti
title>
</head>
<body>
<header>
<h1>Mi
Aplicación de Not
<nav>
<a h
route('note.index
}}">Inicio</a> |
<a h
route('note.creat
}}">Crear Nota</a
</nav>
</header>
<main>

@yield('content'
</main>
</body>
</html>
```

En este formulario vamos a adelantar algunas cosas que ampliaremos más adelante. Para poner el enlace a leminar una nota, vamos a utilizar un formulario con el método `POST` y la directiva `@method('DELETE')`. Esta directiva simula el método HTTP DELETE en formularios HTML (que solo permiten GET y POST. Más adelante explicaremos esto con más detalle. Hasta aquí nada especialmente llmativo. Pero también veréis que está la directiva `@csrf`. Esta directiva genera un campo oculto con un token que protege el formulario contra ataques CSRF Cross-Site Request Forgery). Más adelante también explicaremos esto con más detalle. En `laravel` esta prtección es automática, pero hay que incluir la directiva `@csrf` en los formularios para que funcione correctamente.

```text
@extends('layouts

@section('title'
'Listado de Notas

@section('content
<h2>Listado d
Notas</h2>

@forelse ($no
$note)
<div>
<h3><
href="{{
route('note.show
$note->id) }}">{{
>title }}</a></h3
<p>{{
>description }}</
<smal
$note->date }}</s
<div>
<
href="{{
route('note.edit
$note->id) }}">Ed
a>
<
action="{{
route('note.dest
$note->id) }}"
method="POST"
style="display:in

@csrf

@method('DELETE'

<button
```

```text
type="submit">Eli
button>
<
</div
</div>
@empty
<p>No hay
disponibles.</p>
@endforelse
@endsection
```

### Crear una Nueva Nota

Para poder crear una nota necesitamos tres cosas:

1. **1**. Una ruta que muestre el formulario de creación, `/note/create`.

2. **2**. Un método en el controlador que maneje esa ruta y muestre la vista con el formulario: `function create()`.

3. **3**. Una vista que contenga el formulario de creación: `resources/views/` `notes/create.blade.php`.

```text
<?php
Route::get('/note/
create',
[NoteController::class,
'create'])-
>name('note.create');
```

```text
<?php
public function
create()
{
return
view('notes.create
}
```

```text
@extends('layouts

@section('title'
'Crear Nueva Nota

@section('content
<h2>Crear Not
<form action=
route('note.store
method="POST">
@csrf

<label>Título:</l
<input
type="text"
name="title" requ
```

```text
<label>Descripció
label>
<textarea
name="descriptio
required></texta

<label>Fe
label>
<input
type="date" name=
required>
```

```text
<label>Completada
label>
<input
type="checkbox"
name="done">

<button
type="submit">Gua
```

```text
yp
button>
<a href=
route('note.index
}}">Cancelar</a>
</form>
@endsection
```

### Guardar la Nueva Nota

Al igual que en el caso anterior, para guardar la nota necesitamos dos cosas. Una ruta que maneje el envío del formulario y un método en el controlador que procese los datos y guarde la nota en la base de datos. Al final, redirigiremos a la lista de notas.

```text
<?php
Route::post('/note
store',
[NoteController::c
'store'])-
>name('note.store
```

Para guardar la nota, podemos usar diferentes métodos. Aquí mostramos dos formas:

```text
<?php
public function
store(Request
$request)
{
$note = new
Note();
$note->title
$request-
>input('title');
$note->descri
= $request-
>input('descripti
$note->date =
$request-
>input('date');
$note->done =
$request-
>input('done') ?
0;
$note->save(
// Redirigir
lista de notas
return redire
>route('note.inde
}
```

O usando el método `create`:

```text
<?php
public function
store(Request
$request)
{

Note::create($requ
>all());
return redirec
>route('note.index
}
```

En el caso de los campos `checkbox`, si no se marca, no se envía ningún valor, es decir, en el request no existirá la clave `done` y por tanto este campo no se actualizará. Para solucionarlo, podemos usar una expresión ternaria:

```text
$done = $request-
>input('done') ? 1:
0;
```

para asignar un valor por defecto de `0` cuando no se marque el checkbox. O simplememnte comprobar si el campo existe en el request y asignar `1` si existe o `0` si no existe

Por tanto tenemos que modificar el método `store` para que el campo `done` se actualice correctamente:

```text
<?php
public function
store(Request
$request)
{
$data = $requ
>all();
$data['done'
$request-
>has('done');
```

```text
Note::create($dat

return
redirect()-
>route('note.inde
}
```

- `@csrf` protege contra ataques CSRF Cross-Site Request Forgery).

- `$request->all()` devuelve todos los datos enviados en el formulario.

- Laravel valida automáticamente que el token CSRF esté presente. Si no lo está, lanzará un error.

¿Cómo funciona CSRF?

- Laravel genera un token único para cada sesión de usuario.

- Este token se incluye en cada formulario generado por Laravel.

- Cuando se envía el formulario, Laravel verifica que el token enviado coincida con el de la sesión.

- Si no coinciden, Laravel lanza un error 419 Page Expired).

- Esto previene que un atacante envíe formularios en nombre del usuario sin su consentimiento.

### Editar una Nota

```text
Route::get('/note/edit/
{note}',
[NoteController::class,
'edit'])-
>name('note.edit');
```

Tenemos varias formas de recibir el parámetro `note`. En esta primer recibimos el ID y buscamos la nota, para poder pasarla a la vista:

```text
<?php
public function
edit($id)
{
$note =
Note::findOrFail($
return
view('notes.edit'
compact('note'));
}
```

En esta segunda forma, recibimos el modelo `Note`. De esta manera es Laravel el que se encarga de buscar la nota. Cuando trabajamos con modelos, esta es la forma recomendada:

```text
<?php
public function
edit(Note $note)
{
return
view('notes.edit'
compact('note'));
}
```

En este caso la ruta la hemos definido con el método `PUT`. Este método es el que se utiliza para actualizar los datos de un recurso existente. Pero ¿cómo hacerlo si las opciones de `form` solo permiten `GET` y `POST` ?. Laravel nos ofrece una solución sencilla: la directiva `@method('PUT')`. Esta directiva simula el método PUT en formularios HTML. Esta directiva debe estar dentro del formulario y antes de los inputs.

```text
<form id="sample-
form"
action="somepage.php
method="POST">
@csrf
@method('PUT')
<!-- Otros campo
del formulario -->
</form>
```

Con este formato el formulario se enviará como un PUT, aunque el método del formulario sea POST.

```text
@extends('layouts

@section('title'
Nota')

@section('content
<h2>Editar No
<form action=
route('note.updat
$note->id) }}"
method="POST">
@csrf
@method(
<label>Tí
label>
<input ty
name="title" valu
$note->title }}"
```

```text
<label>Descripció
label>
<textarea
name="descriptio
required>{{ $note
>description }}</

<label>Fe
label>
<input ty
name="date" value
$note->date }}"
```

```text
<label>Completada
<input
type="checkbox"
name="done" {{ $n
? 'checked': ''
```

```text
<button
type="submit">Act
button>
<a href=
route('note.index
}}">Cancelar</a>
</form>
@endsection
```

- `@method('PUT')` simula el método HTTP PUT en formularios HTML (que solo permiten GET y POST.

### Actualizar la Nota

```text
<?php
Route::put('/note/
update/{note}',
[NoteController::c
'update'])-
>name('note.update
```

En este caso también tenemos dos formas de recibir el parámetro `Note $note`. En esta primer caso recibimos la nota y `laravel` por inyección de modelo la busca por nosotros y en el segundo caso recibimos el ID y buscamos la nota nosotros.

```text
<?php
public function
update(Request
$request, Note $no
{
$note-
>update($request-
>all());
return
redirect()-
>route('note.index
}
```

```text
<?php
public function
update(Request
$request, $id)
{
$note =
Note::findOrFail($
$note-
>update($request-
>all());
return redirec
>route('note.index
}
```

En este caso tenemos el mismo problema que en el método `store` con el campo `done`. Si no se marca el checkbox, no se envía ningún valor y por tanto este campo no se actualizará. Para solucionarlo, podemos usar una expresión ternaria para asignar un valor por defecto de `0` cuando no se marque el checkbox. O simplememnte comprobar si el campo existe en el request y asignar `1` si existe o `0` si no existe.

```text
<?php
public function
update(Request
$request, Note $no
{
$data = $reque
>all();
$data['done']
$request-
>has('done');
$note-
>update($data);
return
redirect()-
>route('note.index
}
```

### Mostrar una Nota Individual

```text
<?php
Route::get('/note/show/
{note}',
[NoteController::class,
'show'])-
>name('note.show');
```

En este caso también tenemos dos formas de recibir el parámetro `Note $note`. En esta primer caso recibimos la nota y `laravel` por inyección de modelo la busca por nosotros y en el segundo caso recibimos el ID y buscamos la nota nosotros.

```text
<?php
public function
show($id)
{
$note =
Note::findOrFail($
return
view('notes.show'
compact('note'));
}
```

```text
<?php
public function
show(Note $note)
{
return
view('notes.show'
compact('note'));
}
```

```text
@extends('layouts

@section('title'
'Detalle de Nota

@section('content
<h2>{{ $note
}}</h2>
<p>{{ $note-
>description }}</
<p>Fecha: {{
>date }}</p>
<p>Estado: {{
$note->done ?
'Completada':
'Pendiente' }}</p
<a href="{{
route('note.index
}}">Volver</a>
@endsection
```

### Eliminar una Nota

```text
<?php
Route::delete('/note/
destroy/{note}',
[NoteController::class,
'destroy'])-
>name('note.destroy');
```

Como en los casos anteriores, tenemos dos formas de recibir el parámetro `Note` `$note`. En esta primer caso recibimos la nota y `laravel` por inyección de modelo la busca por nosotros y en el segundo caso recibimos el ID y buscamos la nota nosotros.

```text
<?php
public function
destroy($id)
{
$note =
Note::findOrFail($
$note->delete
return redirec
>route('note.index
}
```

```text
<?php
public function
destroy(Note $note
{
$note->delete
return
redirect()-
>route('note.index
}
```

- El método `delete()` elimina el registro de la base de datos.

### Prueba completa del CRUD

Ahora vamos probar todas las funcionalidades del CRUD

Al no utilizar nada de CSS, el aspecto es muy básico. En un proyecto real, se debería aplicar estilos CSS para mejorar la apariencia y usabilidad.

1. **Listar Notas**: Accede a la ruta `/` para ver el listado de notas.

2. **Crear Nota**: Haz clic en "Crear Nota", rellena el formulario y envíalo.

Al hacer click en "Crear Nota" se accede a `/note/create`:

Una vez rellenado el formulario lo enviamos al servidor (ruta `/note/` `store`) y volvemos al listado de notas.

Podemos ver la nota creada y como aparecen los enlaces para editar y eliminar.

3. **Editar Nota**: Haz clic en "Editar" junto a una nota, modifica los datos y envía el formulario.

Al hacer click en "Editar" se accede a `/note/edit/{id}`:

Una vez modificado el formulario lo enviamos al servidor (ruta `/note/`

`update/{id}`) que actualiza la nota y nos redirecciona al listado de notas.

4. **Mostrar Nota**: Haz clic en el título de una nota para ver sus detalles.

Al hacer click en el título de una nota se accede a `/note/show/{id}`:

Una vez vista pulsamos volver y nos redirecciona al listado de notas.

5. **Eliminar Nota**: Haz clic en "Eliminar" junto a una nota. Al hacer click en "Eliminar" se envía un formulario con método `DELETE` a la ruta `/note/` `destroy/{id}` que elimina la nota y nos redirecciona al listado de notas.

En un proyecto real, es recomendable añadir una confirmación antes de eliminar una nota para evitar eliminaciones accidentales.

### Tipado en los métodos del controlador

```text
<?php
public function
update(
Request
$request,
Note
$note
):
RedirectResponse
{
$note-
>update($request-
>all());
return
redirect()-
>route('note.index')
}
```

- Tipar los parámetros mejora la legibilidad y control de errores.

- Tipar el tipo de retorno ayuda a Laravel a validar internamente las respuestas.

- `View` para devolver vistas. `use` Illuminate\View\View

- `RedirectResponse` para redirecciones. `use` Illuminate\Http\RedirectRes ponse

- `JsonResponse` para APIs. `use` Illuminate\Http\JsonRespons e

IES JUAN DE GARAY - 2º DWES (Curso original de Beteta adaptado por Eva María Gómez Abad )
