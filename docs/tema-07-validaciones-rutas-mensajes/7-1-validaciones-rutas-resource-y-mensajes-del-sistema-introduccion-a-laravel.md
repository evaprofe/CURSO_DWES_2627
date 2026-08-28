# Validaciones, Rutas Resource y Mensajes del Sistema

## Validación de Datos

### ¿Qué es la validación y por qué es necesaria?

La validación es el proceso mediante el cual se verifica que los datos enviados por el usuario cumplen con unas reglas antes de ser almacenados en la base de datos.

Laravel ofrece un sistema de validación muy robusto, que nos permite validar datos tanto desde el controlador como mediante clases personalizadas.

Siempre debemos validar los datos **antes de** **almacenarlos** y **antes de actualizarlos**. Nunca debemos asumir que lo que llega del formulario es seguro o correcto.

### Validar desde el controlador usando $request->validate()

Aclaración sobre el campo `done`

En este caso vamos a utilizar un `checkbox` para marcar si la nota está completada o no. En el formulario, si el checkbox está marcado, se enviará `done=on`, y si no está marcado, no se enviará nada. Para evitar problemas con el `on` vamos a modificar la vista de creación y edición de notas para que el valor enviado sea `1` o `nada` en el caso de no estar marcado.

```text
<input
type="checkbox"
name="done"
value="1">
```

El método `$request->validate()` es una forma rápida y sencilla de validar los datos directamente en el controlador. Recibe un array con las reglas de validación para cada campo. Si la validación falla, Laravel redirige automáticamente al formulario anterior con los errores y los datos antiguos.

Este método permite validar directamente dentro del método del controlador:

```text
<?php
public function
store(Request $re
{
$validated =
$request->validat
'title' =
'required|string
max:255',
'descript
'required|string
'date' =>
'required|date',
'done' =>
'nullable|boolean
]);
```

```text
Note::create($val
return redire
>route('note.inde
>with('success',
creada correctame
}
```

Cuando la validación falla, Laravel redirige automáticamente al formulario anterior. Una buena técnica es utilizar el método `old()` para recuperar el valor anterior del campos de manera que el usuario no tenga que volver a escribirlo.

```text
<input name="title"
value="{{ old('title')
}}">
```

Haremos esto en todos los campos. Cuídado con los `textarea` que no tienen `value` y con los `checkbox`. Por ejemlo, para el `checkbox`:

```text
<input type="checkbox"
name="done" value="1" {{
old('done') ? 'checked'
: '' }}>
```

Esto mostrará el checkbox marcado si el valor anterior era `1`.

Reglas comunes de validación:

`require` El campo es obligatorio

```text
d
```

`string` Debe ser una cadena de texto

`min:n` Longitud mínima de caracteres

`max:n` Longitud máxima de caracteres `date` Debe ser una fecha válida

`boolean` true/false, 0/1, "yes"/"no"

`nullabl` El campo puede estar vacío

```text
e
```

### Crear una clase de validación personalizada

Aunque en la documentación oficial de Laravel podemos encontrar un sinfín de reglas de validación, a veces necesitamos una validación más compleja o personalizada. Para ello como ejemplo vamos a crear una vlase de validación para un DNI español, que tiene un formato específico de 8 dígitos seguidos de una letra mayúscula.

1. **1**. Primer paso, crear la clase de validación personalizada:

```text
php artisan make:rule
isDNI
```

Esto creará una clase en `app/Rules/` `isDNI.php`.

Ahora vamos a ver cómo sería la implementación de esta clase para validar un DNI español:

```text
<?php
namespace App\Rules

use Closure;
use
Illuminate\Contract

class isDNI impleme
{
public function
mixed $value, Closu
{
$value = st

if (!preg_m
$value)) {
$fail('
formato válido.');
return;
}

$numero = s
$letra = su

$letras = '
$letraCorre

if ($letra
$fail('
válido.');
}
}
}
```

Ahora ya lo podemos usar en nuestras reglas de validación:

```text
<?php
use
App\Rules\DniValido;
$request->validate([
'dni' =>
['required', new
DniValido],
]);
```

### Crear una clase FormRequest personalizada

Las clases `FormRequest` son una forma más estructurada de manejar la validación. Permiten separar la lógica de validación del controlador, lo que hace que el código sea más limpio y mantenible. Normalmente se crean para escenarios de validación compleja. Estas clases encapsulan la lógica de validación y autorización de una petición HTTP específica. Ahora, como ejemplo, vamos a crear una clase `NoteRequest` para manejar la validación de las notas.

Comando:

```text
php artisan make:request
NoteRequest
```

Esto creará una clase en `app/Http/`

`Requests/NoteRequest.php`

Este método define si el usuario tiene permiso para hacer esta petición. Para este curso lo dejaremos en `true`:

Si este método devuelve `false`, la validación no se ejecutará y se lanzará un error 403 Forbidden).

```text
<?php
public function
authorize()
{
return
true;
}
```

Ahora vamos a definir unas reglas de validación de ejmeplo para nuestro ejemplo de notas:

```text
<?php
public
function
rules()
{
return [

'title' =>
'required|
string|
max:255',

'description'
=> 'required|
string|
min:10',
'date'
=> 'required|
date',
'done'
=> 'nullable|
boolean'
];
}
```

En el controlador, en lugar de usar `Request $request`, usaremos `NoteRequest $request`. Laravel se encargará de validar automáticamente los datos antes de ejecutar el método.

```text
<?php
public function
store(NoteRequest
$request)
{
$data = $reque
>all();
$data['done']
$request->has('don
? 1: 0; // Conve
checkbox to boolea

Note::create($data
return
redirect()-
>route('note.index
}
```

Antes de modificar el `update()` recordar que en la vista de edición el checkbox se envía como `done=on` o no se envía nada. Hacemos en esta vista lo mismo que en la de creación:

```text
<input type="checkbox"
name="done" value="1" {{
($note->done) ?
'checked': '' }}>
```

Hacemos lo mismo para el método `update()`:

```text
<?php
public function
update(NoteRequest
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

## Mostrar Errores de Validación

### Mostrar errores en el formulario

Para el usuario es importante recibir feedback inmediato sobre los errores en el formulario. Laravel facilita esto con la variable `$errors` disponible en las vistas.

Puedes mostrar un error específico junto a cada campo:

```text
<input name="title"
value="{{
old('title') }}">
@error('title')
<small
style="color:red">{{
$message }}</small>
@enderror
```

### Resaltar campos con error CSS

También puedes añadir una clase CSS al campo si tiene error, para destacarlo visualmente según diseño de la aplicación:

```text
<input name="title"
class="@error('title')
is-invalid @enderror">
```

Y luego en CSS puedes estilizar la clase `.is-invalid`.

### Mostrar todos los errores juntos

```text
@if ($errors-
>any())
<div
class="alert
alert-danger">
<ul>

@foreach
($errors->all()
as $error)

<li>{{ $error
}}</li>

@endforeach
</ul>
</div>
@endif
```

### Modificar la vista de edición como lo hicimos en la de creación

La vista de creación debe haber quedado así:

```text
@extends('layouts

@section('title'
'Crear nueva Nota

@section('content
<h2>Editar No
h2>
<form action=
route('note.store
method="POST">
@csrf

<label>Título:</l
<input
type="text"
name="title" valu
old('title') }}"
required>
@error('t
<smal
style="color:red
$message }}</smal
@enderro
```

```text
<label>Descripció
label>
<textarea
name="descriptio
required>{{
old('description
textarea>

@error('descripti
<smal
style="color:red
$message }}</smal
@enderro
```

```text
@

<label>Fe
label>
<input
type="date" name=
value="{{ old('da
}}" required>
@error('d
<smal
style="color:red
$message }}</smal
@enderro
```

```text
<label>Completada
label>
<input
type="checkbox"
name="done" {{
old('done') ? 'ch
: '' }}>

<button
type="submit">Gua
button>
<a href=
route('note.index
}}">Cancelar</a>
@error('d
<smal
style="color:red
$message }}</smal
@enderro
</form>
@endsection
```

también modificamos la vista del formulario de edición para mostrar los mensajes de error:

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
@error('t
<smal
style="color:red
$message }}</smal
@enderro
```

```text
<label>Descripció
label>
<textarea
name="descriptio
required>{{ $note
>description }}</

@error('descripti
<smal
style="color:red
$message }}</smal
@enderro

<label>Fe
```

```text
label>
<input ty
name="date" value
$note->date }}"
@error('d
<smal
style="color:red
$message }}</smal
@enderro
```

```text
<label>Completada
<input
type="checkbox"
name="done" {{ $n
? 'checked': ''

<button
type="submit">Act
button>
<a href=
route('note.index
}}">Cancelar</a>
@error('d
<smal
style="color:red
$message }}</smal
@enderro
</form>
@endsection
```

Algunos métodos interesantes de laravel que no hemos utilizado pero que se usan a menudo son:

`Illum` `make(` Crear un validador

```text
inate)
\Supp
ort\F
acade
s\Val
idato
```

```text
r
```

`Illum` `fails` Verificar si la validación falló

```text
inate ()
\Supp
ort\F
acade
s\Val
idato
r
```

`Illum` `error` Obtener los errores de

```text
inate s()
```

validación

```text
\Supp
ort\F
acade
s\Val
idato
r
```

`Illum` `valid` Validar y redirigir automáticamente

```text
inate ate()
\Supp
ort\F
acade
s\Val
idato
r
```

`Illum` `old()` Obtener el valor anterior de un

```text
inate
```

campo

```text
\Supp
ort\F
acade
s\Req
uest
```

`Illum` `flash` Guardar datos en la sesión para la

```text
inate ()
```

siguiente

```text
\Supp
```

petición

```text
ort\F
acade
s\Req
uest
```

Ahora vamos a probar a crear una nota con una descripción inferir a los 10 caracteres y veremos el mensaje de error.

{ width=400px }

Ahora corregimos el error y vemos que la nota se crea correctamente. Además nos aparece un mensaje de éxito.

{ width=400px }

Ahora podéis comprobar que:

- En la edición las validaciones y mensajes de éxito funcionan igual.

- El campo ´done´ funciona correctamente tanto en creación como en edición.

Si has seguido al pie de la letra las instrucciones, el campo `done` no va a funcionar. Al añadir la clase `NoteRequest` pedimos que el campo sea `boolean` pero a este punto está llegando un `on` cuando el checkbox está marcado, o nada cuando no lo está. Y por tanto no llegaremos nunca a que se ejecute el método `store()` o `update()`, que es donde estábamos manejando el valor del checkbox.

Necesitamos que cuando se valida el campo `done`, si está marcado llegue como `true` y si no está marcado como `false`. Ya que en la validación hemos especificado que debe ser un booleano. Para ello vamos a usar el método `prepareForValidation()` que nos permite modificar los datos antes de que se apliquen las reglas de validación.

Añadimos este método en la clase `NoteRequest`:

```text
<?php
protected function
prepareForValidati
{
$this->merge(
'done' =>
$this->done ? true
false,
]);
}
```

De esta manera nos aseguramos que el campo `done` siempre llega como `1` o `0`, y por tanto la validación funciona correctamente.

Este método es muy útil para modificar cualquier dato antes de que se aplique la validación. Por ejemplo, podríamos usarlo para formatear fechas, convertir cadenas a mayúsculas/minúsculas. No siempre la información que llega del formulario está en el formato que necesitamos para validar o almacenar. Y que no esté en el formato no significa que no sea válido. Pero la validación de `laravel` es estricta y debemos asegurarnos que los datos cumplen las reglas que hemos definido.

### Traducir los mensajes de error

Laravel trae sus mensajes de error por defecto en inglés. Hasta la versión los ficheros de traducción se encontraban en `resources/lang/en/validation.php`. A partir de Laravel 10 es necesario activar el sistema de traducción para que se publiquen los archivos de idioma. Para ello ejecutamos el siguiente comando:

```text
php artisan lang:publish
```

Esto creará los archivos de traducción en `src/lang/`. Ahora si queremos traducir los mensajes de error al castellano, por ejemplo, podemos copiar el archivo `src/` `lang/en/validation.php` a `src/` `lang/es/validation.php` y traducir los mensajes.

Para que Laravel use el idioma español, debemos configurar el valor de `locale` en el archivo `config/app.php`:

```text
'locale' => 'es',
```

## Mensajes de Éxito

Para mejorar la experiencia del usuario, es buena práctica mostrar mensajes de éxito o error después de operaciones como crear, actualizar o eliminar. PPara ello vamos a usar los mensajes flash, que permiten mandar un mensaje entre peticiones almacenándolo en la sesión.

### Flash de sesión con with()

Este método permite guardar un mensaje en la sesión que se mostrará en la siguiente petición. Vamos a usarlo para mostrar un mensaje de éxito después de crear o actualizar o eliminar una nota.

En la función `store()` del controlador:

```text
<?php
return redirect()-
>route('note.index')-
>with('success', 'Nota
guardada
correctamente.');
```

En la función `update()`:

```text
<?php
return redirect()-
>route('note.index')-
>with('success', 'Nota
actualizada
correctamente.');
```

En la función `destroy()`:

```text
<?php
return redirect()-
>route('note.index')-
>with('danger', 'Nota
eliminada
correctamente.');
```

### Mostrar el mensaje en la vista (por ejemplo, en layout):

```text
@if
(session('success
<div
class="alert ale
success">
{{
session('success
}}
</div>
@endif
@if
(session('danger
<div
class="alert ale
danger">
{{
session('danger'
</div>
@endif
```

O incluirlo como partial:

```text
@if
(session('success
<div
class="alert ale
success"

style="padding:
10px; margin-bott
20px;

background-color
#d4edda;

color: #155724;

border-color:
#c3e6cb;

border-radius:
5px;">
{{
session('success
}}
</div>
@endif
@if
(session('danger
<div
class="alert ale
danger"

style="padding:
10px; margin-bott
20px;

background-color
#f8d7da;

color: #721c24;
```

```text
;

border-color:
#f5c6cb;

border-radius:
5px;">
{{
session('danger'
</div>
@endif
```

Y en el layout:

```text
@include('_partials.message
```

## Rutas Resource

### ¿Qué son?

Las rutas `Route:resource()` generan automáticamente todas las rutas necesarias para un CRUD completo. De esta manera nos ahorramos definir todas las rutas para todos los modelos.

```text
<?php
Route::resource('note',
NoteController::class);
```

### Acciones generadas

GET /note index Mostrar todas las notas

GET /note/ create Formulario create para nueva nota

POST /note store Guardar nueva nota

GET /note/ show Mostrar una {note} nota

GET /note/ edit Formulario {note}/edit para editar

PUT/PATCH / update Actualizar note/{note} nota

DELETE / destroy Eliminar nota note/{note}

### Personalización

- Solo algunas rutas:

```text
<?php
Route::resource('note',
NoteController::class)-
>only(['index',
'show']);
```

- Excluir algunas:

```text
<?php
Route::resource('note',
NoteController::class)-
>except(['destroy']);
```

### Ver rutas disponibles

```text
php artisan route:list
```

Te muestra todas las rutas definidas, su método, URI y nombre.

### Crear un controlador tipo resource

```text
php artisan
make:controller
NoteController --
resource
```

Este comando crea todos los métodos básicos ( `index`, `create`, `store`, `show`, `edit`, `update`, `destroy`).

## Conclusiones

- Validar datos es esencial para proteger la integridad de la base de datos.

- Los mensajes de error y éxito mejoran la experiencia del usuario.

- Las rutas resource simplifican la estructura del código.

