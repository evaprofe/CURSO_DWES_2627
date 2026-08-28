# Creación de APIs REST en Laravel

## Rutas API

En **Laravel 12**, a diferencia de versiones anteriores, el archivo `routes/api.php` **no viene incluido por defecto**. Laravel ahora permite habilitarlo opcionalmente para mantener la aplicación más ligera si no vas a construir una API.

### Activar el sistema de rutas API

Para trabajar con rutas API, primero debemos ejecutar el siguiente comando Artisan:

```text
php artisan
install:api
```

Este comando:

- Crea automáticamente el archivo `routes/api.php`.

- Registra el archivo en el sistema de rutas.

- Aplica el middleware `api` a las rutas definidas allí.

- Añade el prefijo `/api` a todas las rutas de ese archivo.

En el proceso nos pude pedir crear una nueva migración para la tabla `api_tokens`, podemos decir que sí aunque de momento no la vamos a utilizar ya que no hemos visto la autenticación.

### ¿Dónde se registra esta configuración?

Laravel configura los archivos de rutas en `bootstrap/app.php`. Una vez activadas, podrás ver una línea como esta:

```text
<?php
->withRouting(
api:
__DIR__.'/../
routes/
api.php',
apiPrefix:
'api',
//...
)
```

Laravel no añade este prefijo automáticamente, pero su valor por defecto es `api`. Por tanto añadir esta línea o no es equivalente. Pero yo recomiendo ponerla explícitamente para tener claro que las rutas de `api.php` van con el prefijo `/api`.

Si quieres cambiar el prefijo, por ejemplo a `api/admin`, puedes modificar la clave `apiPrefix`:

```text
<?php
->withRouting(
api:
__DIR__.'/../
routes/
api.php',
apiPrefix:
'api/admin',
)
```

A partir de este momento, cualquier ruta definida en `routes/api.php` responderá a URLs que empiecen por `/api/` (o el prefijo que hayas definido).

El archivo `web.php` sigue existiendo por defecto y está pensado para rutas que devuelven vistas HTML. Por tanto, recuerda usar `api.php` exclusivamente para tu API REST.

Laravel carga cada archivo de rutas con su configuración correspondiente desde `bootstrap/app.php`:

- `routes/web.php` **-** grupo de middleware `web`

- `routes/api.php` grupo de middleware `api` y prefijo `/api`

## Introducción

Una **API** Application Programming Interface) permite a aplicaciones diferentes comunicarse entre sí, intercambiando datos en formatos como JSON. Las APIs REST usan los verbos HTTP GET, POST, PUT, DELETE) para definir operaciones sobre recursos.

Laravel ofrece todas las herramientas necesarias para construir APIs modernas, organizadas y seguras. En este tema construiremos paso a paso una API para el recurso `Note`, que ya conocemos de los temas anteriores.

## Definir Rutas de API

### Ficheros de rutas y su organización

Laravel separa las rutas para aplicación web y API

`routes/` Rutas para la interfaz web HTML

```text
web.php
```

`routes/` Rutas para la API, normalmente consumidas

```text
api.php
```

por clientes que esperan JSON

Estas rutas se cargan mediante la configuración definida en `bootstrap/` `app.php`.

Cuando activamos el soporte para API con `php artisan install:api`, Laravel registra el archivo `routes/api.php`, le aplica el prefijo `/api` y asocia estas rutas al grupo de middleware `api`.

Este grupo está pensado para rutas **stateless** (sin sesión ni CSRF) y permite, entre otras cosas, la resolución automática de parámetros de ruta a modelos mediante `SubstituteBindings`.

Por defecto, este grupo incluye la resolución de route model binding mediante `SubstituteBindings`. Si además queremos limitación de peticiones ( `throttle`) u otros middlewares, podemos añadirlos explícitamente en la configuración de middleware o en las rutas.

```text
<?php
Route::get('/
notes',
function () {
return
['mensaje' =>
'Esta es la API
de notas'];
});
```

Accediendo a `http://` `localhost:8000/api/notes`, obtendrás: `{ "mensaje": "Esta es`

```text
la API de notas" }
```

Hay que fijarse que no es necesario añadir `/api` en la ruta, Laravel lo añade automáticamente. Pero sí hay que ponerlo en las peticiones.

### Middleware aplicado a las rutas API

Las rutas definidas en `routes/api.php` quedan asociadas automáticamente al grupo de middleware `api`.

En la estructura moderna de Laravel, este grupo se configura en `bootstrap/` `app.php`.

Por ejemplo, si definimos una ruta como:

```text
<?php
Route::get('/notes/
{note}',
[NoteController::class,
'show']);
```

y en el controlador usamos:

```text
<?php
public function
show(Note $note):
JsonResponse
{
return response()-
>json([
'success' =>
true,
'data' => $note
], 200);
}
```

Este comportamiento se basa en el route model binding de Laravel, que permite convertir automáticamente los parámetros de la URL en modelos de Eloquent.

Laravel intentará buscar automáticamente la nota por su identificador. Si no existe, devolverá un error 404.

Si queremos añadir limitación de peticiones ( `throttle`) al grupo `api`, podemos hacerlo explícitamente en la configuración de middleware o en rutas concretas.

### Control de errores en la API

En una API REST es importante devolver siempre respuestas en formato JSON, también cuando ocurre un error.

Uso de route model binding

Cuando usamos parámetros tipados en el controlador:

```text
<?php
public function
show(Note $note):
JsonResponse
```

Laravel resuelve automáticamente el modelo. Si no existe, devuelve un error **404 Not Found**.

Este mismo comportamiento se aplica en métodos como `update()` y `destroy()`:

```text
<?php
public function
update(NoteRequest
$request, Note
$note):
JsonResponse
{
$note-
>update($request-
>validated());

return
response()->json([
'success'
=> true,
'message'
=> 'Nota
actualizada
correctamente.',
'data' =>
$note
], 200);
}

public function
destroy(Note
$note):
JsonResponse
{
$note-
>delete();

return
response()->json([
'success'
=> true,
'message'
=> 'Nota eliminada
correctamente.'
])
```

```text
], 200);
}
```

Uso alternativo con findOrFail

También podemos usar:

```text
<?php
$note =
Note::findOrFail($id);
```

Si no existe, Laravel lanza una excepción y devuelve un 404.

Asegurar respuestas JSON en errores

En clientes como REST Client, Postman o frontend, es recomendable enviar:

```text
Accept: application/json
```

Además, podemos forzar que todas las rutas de la API devuelvan JSON configurando `bootstrap/app.php`:

```text
<?php
->withExceptions(fun
(Exceptions $excepti
{
$exceptions-
>shouldRenderJsonWhe
(Request $request, T
$e) {
return $requ
>is('api/*') || $req
>expectsJson();
});
})
```

Conclusión

- Usar **route model binding** es la opción más limpia.

- Laravel devuelve automáticamente **404** si el recurso no existe.

- No es necesario usar `try-catch` en cada método.

- La mejor práctica es centralizar el formato JSON de errores.

## Crear Controlador para la API

Antes de generar el controlador, debemos asegurarnos de que existe la tabla `notes`

en nuestra base de datos.En caso de no tenerla, podemos crearla con el siguiente comando:

```text
php artisan
make:migration
create_notes_table
```

Esto generará un archivo de migración en `database/migrations` que podemos editar para definir la estructura de la tabla `notes`.

Asegúrate de que la migración tenga el siguiente contenido:

```text
<?php
use
Illuminate\Databa
use Illuminate\Da
use Illuminate\Su
class CreateNotes
{
public functi
{
Schema::c
(Blueprint $table
$tabl
$tabl
$tabl
$tabl
$tabl
>default(false);
$tabl
});
}

public functi
{
Schema::d
}
}
```

Después de editar la migración, ejecuta el siguiente comando para crear la tabla:

```text
php artisan migrate
```

Esto creará la tabla `notes` en tu base de datos.

También podemos crear el modelo `Note` con el siguiente comando:

```text
php artisan make:model
Note
```

Esto generará el modelo `Note` en `app/` `Models/Note.php`.

### Generar un controlador API

Usamos el flag `--api` para generar un controlador que sólo incluye los métodos necesarios para una API CRUD

```text
php artisan
make:controller Api/
NoteController --api
```

Esto creará el archivo en `app/Http/` `Controllers/Api/` `NoteController.php` con los métodos: `index`, `store`, `show`, `update`, `destroy`.

```text
<?php
namespace
App\Http\Controll
use
App\Http\Controll
use App\Models\No
use
Illuminate\Http\J
use Illuminate\Ht
class NoteControl
Controller
{
public functi
JsonResponse
{
// Lista
notas
}

public functi
$note): JsonRespo
{
// Mostra
ID
}

public functi
store(Request $re
JsonResponse
{
// Crear
}

public functi
update(Request $
$note): JsonRespo
{
// Actual
existente
```

```text
}

public functi
$note): JsonRespo
{
// Elimin
}
}
```

El controlador `NoteController` extiende de `Controller` y usa el modelo `Note` para interactuar con la base de datos.

### = Crear las rutas API para Notes

En `routes/api.php`:

```text
<?php
use
App\Http\Controlle

Route::apiResource
NoteController::cl
```

Por seguridad, podemos utilizar `only()` para definir las rutas que queremos habilitar:

```text
<?php
Route::apiResource('notes',
NoteController::class)-
>only([
'index', 'show',
'store', 'update',
'destroy'
]);
```

Esto generará automáticamente las rutas necesarias para el controlador `NoteController` usando el método `apiResource`. Laravel se encarga de crear las rutas RESTful para los métodos del controlador.

Esto define rutas como:

- GET `/api/notes`

- GET `/api/notes/{note}`

- POST `/api/notes`

- PUT/PATCH `/api/notes/{note}`

- DELETE `/api/notes/{note}`

Puedes comprobarlas con:

```text
php artisan route:list
--path=api/notes
```

## Implementar el CRUD API para Notes

### = Modelo Note

Asegúrate de que el modelo `Note` está correctamente definido con `$fillable`:

```text
<?php
class Note
extends Model
{
protected
$fillable =
['title',
'description',
'date',
'done'];
}
```

## Códigos de Estado HTTP en APIs

En una API REST, es importante devolver **cédigos de estado HTTP apropiados** para indicar si la operación fue exitosa o si ocurrió un error.

A continuación, una tabla con los códigos más comunes y su uso recomendado:

`200 OK` Éxito La petición fue exitosa (por ejemplo, GET, PUT, DELETE

`201` Recurso Se ha creado creado un nuevo

```text
Create
```

recurso

```text
d
```

correctamente (por ejemplo, POST

`204 No` Sin La petición fue contenido exitosa pero no

```text
Conten
```

se devuelve

```text
t
```

ningún contenido (opcional tras DELETE

`400` Petición Cuando el incorrecta cliente envía

```text
Bad
```

datos inválidos

```text
Reques
t
```

`401` No Cuando el autorizado usuario no está

```text
Unauth
```

autenticado

```text
orized
```

`403` Prohibido El usuario está autenticado

```text
Forbid
```

pero no tiene

```text
den
```

permisos `404` No El recurso encontrado solicitado no

```text
Not
```

existe

```text
Found
```

`422` Entidad no Validaciones procesable fallidas en los

```text
Unproc
```

datos enviados

```text
essabl
e
Entity
```

`500` Error del Error servidor inesperado en

```text
Intern
```

el servidor

```text
al
Server
Error
```

- Usamos **200 OK** en respuestas normales donde devolvemos datos GET, PUT, DELETE.

- Usamos **201 Created** al crear un recurso con POST para indicar que se creó exitosamente.

- Usaríamos **204 No Content** si quisiéramos responder a un DELETE sin mensaje (aunque aquí devolvemos mensaje con 200.

## Implementar métodos

## del controlador

Antes de implementar los métodos del controlador, asegúrate de importar las clases necesarias:

```text
<?php
use
Illuminate\Http\Js
use Illuminate\Htt
```

### Método index() – Listar notas

```text
<?php
public function
index():
JsonResponse
{
return
response()-
>json([

'success' =>
true,
'data'
=> Note::all()
], 200);
}
```

### EH Método show() – Mostrar una nota

```text
<?php
public function
show(Note
$note):
JsonResponse
{
return
response()-
>json([

'success' =>
true,
'data'
=> $note
], 200);
}
```

### Método store() – Crear una nota

```text
<?php
public function
store(Request
$request):
JsonResponse
{
$note =
Note::create($req
>all());
return respo
>json([
'success
true,
'message
'Nota creada
correctamente.',
'data' =>
$note
], 201);
}
```

### Método update() – Modificar nota

```text
<?php
public function
update(Request
$request, Note
$note):
JsonResponse
{
$note-
>update($request
>all());
return
response()-
>json([
'success
=> true,
'message
=> 'Nota
actualizada
correctamente.',
'data' =>
$note
], 200);
}
```

### Método destroy() – Eliminar nota

```text
<?php
public function
destroy(Note
$note):
JsonResponse
{
$note-
>delete();
return
response()-
>json([

'success' =>
true,

'message' =>
'Nota eliminada
correctamente.'
], 200);
}
```

### Testing de la API

Para probar la API vamos a usar una extensión de Code llamada `REST` `Client` que permite hacer peticiones HTTP directamente desde el editor. También puedes usar herramientas como `Postman` o `Insomnia`.

Para ello vamos a crear un archivo `notes.rest` en la raíz del proyecto por ejemplo.

Vamos a escribir nuestra primera petición para listar todas las notas:

```text
###
GET http://
localhost:8080/
api/notes
Accept:
application/
json
```

Para lanzar la petición, sitúate en la línea `GET.` y pulsa el botón `Send` `Request` que aparece encima.

Debes obtener una respuesta como esta:

En este caso teníamos dos notas creadas previamente, de un tema anterior.

Vamos a crear una nueva nota con el método `POST`:

```text
###
POST http://
localhost:8080/
api/notes
HTTP/1.1
Accept:
application/
json
content-type:
application/
json

{
"title":
"Nueva Nota",

"description":
"Descripción de
la nueva nota",
"date":
"2023-10-01",
"done":
false
}
```

Debemos recibir una respuesta como esta:

En este código hay que observar varias cosas:

La URL de la petición es `http://` `localhost:8080/api/notes`, que es la ruta que hemos definido en nuestro archivo de rutas. El método HTTP utilizado es `POST`, lo que indica que estamos creando un nuevo recurso.

3. **3**. Hemos especificado la cabecera content-type: application/ `json` para indicar que el cuerpo de la petición es un JSON.

4. **4**. En el cuerpo de la petición, estamos enviando un objeto JSON con los datos de la nueva nota que queremos crear.

5. **5**. Hemos dejado una línea en blanco entre las cabeceras y el cuerpo de la petición, que es obligatorio en HTTP.

Ahora vamos a modificar la nota creada anteriormente con el método `PUT`. Pero antes necesitamos su ID, para ello repetimos la petición `GET` para listar todas las notas y ver el ID de la nota que acabamos de crear. En mi caso es la nota con ID 5.

```text
###
PUT http://
localhost:8080/
api/notes/5
HTTP/1.1
Accept:
application/
json
content-type:
application/
json

{
"id": 5,
"title":
"Nota
Modificada",

"description":
"Descripción de
la nota
modificada",
"date":
"2023-10-01",
"done":
true
}
```

La estructura es similar a la petición `POST`, pero en este caso el método es `PUT` y la URL incluye el ID de la nota que queremos modificar.

Debemos recibir una respuesta como esta:

Finalmente vamos a eliminar la nota con ID 5 usando el método `DELETE`:

```text
###
DELETE http://
localhost:8080/
api/notes/5
Accept:
application/
json
```

La petición es muy sencilla, sólo necesitamos el método `DELETE` y la URL con el ID de la nota que queremos eliminar.

Debemos recibir una respuesta como esta:

Con esto hemos probado todas las operaciones CRUD de nuestra API REST para el recurso `Note`. En el siguiente apartado vamos a ver cómo mejorar la salida de los datos usando `API` `Resources`, podemos tomar el control del formato de los datos que devolvemos.

## API Resources

Laravel permite transformar la salida de tus APIs con clases Resource que te dan control sobre el formato.

### Crear un API Resource

```text
php artisan
make:resource
NoteResource
```

Crea el archivo en `App\Http\Resources\NoteResource.` `php`

### Personalizar la transformación

Vamos a modificar la salida de los datos en `NoteResource.php`. Por ejemplo, podemos cambiar los nombres de los campos y añadir un campo calculado `estado` que indique si la nota está completada o pendiente:

```text
<?php
public function
toArray($request
{
return [
'id' =>
$this->id,
'titulo'
=> $this->title,

'descripcion' =>
$this-
>description,
'fecha'
=> $this->date,
'estado'
=> $this->done ?
'Completada':
'Pendiente'
];
}
```

### Usar el recurso en el controlador

Añadimos la importación al controlador `NoteController`:

```text
<?php
use
App\Http\Resources\NoteR
```

Modificamos el método `index()` para devolver una colección de `NoteResource`:

```text
<?php
return response()
'success' => t
'data' =>
NoteResource::coll
], 200);
```

De esta manera, la respuesta incluirá el estado y el mensaje de éxito. Vamos a comprobarlo con una petición `GET` a `/` `api/notes`:

En los demás métodos no devolvemos una colección sino un solo elemento. Por ello por ejemplo para el método `show()` podemos hacer lo siguiente:

```text
<?php
public function
show(Note $note):
JsonResponse
{
return
response()->json(
'success'
=> true,
'data' =>
new
NoteResource($note
], 200);
}
```

Ahora sería aplicable a todos los métodos que devuelven un solo elemento, como `store()` y `update()`. Al `destroy()` no puesto que no devuelve los datos de la nota eliminada.

## Validación de los datos

Al igual que en los formularios, es importante validar los datos que recibimos en la API. `Laravel` ofrece un sistema de validación muy potente. Empezaremos por validar los datos en el método `store()` y `update()`. Para ello vamos a crear la clase `NoteRequest`:

En el tema anterior creamos la clase `NoteRequest` para validar los datos del formulario. Si la tienes creada, elimínala para evitar conflictos.

```text
php artisan
make:request
NoteRequest
```

Esto generará el archivo en `app/Http/` `Requests/NoteRequest.php`.

```text
<?php
namespace App\Htt
use Illuminate\Fo
use Illuminate\Co
use
Illuminate\Http\
use Illuminate\Va

class NoteRequest
{
public functi
{
return t
}

public functi
{
return [
'titl
'desc
'date
'done
];
}

}
```

esta función al igual que en los formularios, define las reglas de validación. En este caso:

- `title`: requerido, cadena de texto, máximo 255 caracteres.

- `description`: requerido, cadena de texto.

- `date`: requerido, debe ser una fecha válida.

- `done`: booleano (opcional).

En caso de error en la validación, Laravel devolverá automáticamente un error 422 con los detalles del error.

```text
{
"message":
"The given data
was invalid.",
"errors": {

"title": [

"The title
field is
required."
]
}
}
```

Podemos personalizar la respuesta de error en el método `failedValidation()` en la clase `NoteRequest`:

Debemos añadir las importaciones necesarias al principio del archivo:

```text
<?php
use Illuminate\Co
use
Illuminate\Http\Ex
use Illuminate\Val
```

```text
<?php
protected function
failedValidation(V
$validator)
{
throw new
HttpResponseExcept
>json([
'success'
'message'
validación',
'errors' =
>errors()
], 422, [],
JSON_UNESCAPED_UN
}
```

En los parámetros de `json()` podemos añadir el tercer parámetro `JSON_UNESCAPED_UNICODE` para evitar que los caracteres especiales se escapen. Esto es útil si estás trabajando con caracteres no ASCII. Sino los acentos y caracteres especiales se escaparán y no se verán correctamente en la respuesta.

Ahora para que las validaciones funcionen debemos ajustar los métodos `store()` y `update()` del controlador para usar `NoteRequest` en lugar de `Request`:

```text
<?php
use
App\Http\Requests\NoteRe
```

```text
<?php
public function
store(NoteRequest
$request):
JsonResponse
{
$note =
Note::create($req
>validated());
return respo
>json([
'success
true,
'message
'Nota creada
correctamente.',
'data' =>
NoteResource($not
], 201);
}

public function
update(NoteReques
$request, Note $n
JsonResponse
{
$note-
>update($request
>validated());
return respo
>json([
'success
true,
'message
'Nota actualizada
correctamente.',
'data' =>
NoteResource($not
], 200);
```

```text
],);
}
```

Se puede observar en los dos métodos que hemos cambiado `$request->all()` por `$request->validated()`. Esto asegura que sólo los datos que pasaron la validación se usan para crear o actualizar la nota.

Por ejemplo, imaginemos que tenemos la siguiente validación, una clase que solo admite los campos `name` y `email`:

```text
<?php
class
StoreUserRequest
extends
FormRequest
{
public
function rules()
{
return [

'name' =>
['required',
'string'],

'email' =>
['required',
'email'],
];
}
}
```

Ahora el cliente envía el siguiente JSON

```text
{
"name": "John Do
"email":
"john.doe@example.co
"role": "admin"
}
```

El campo `role` no está definido en las reglas de validación, por lo que será ignorado cuando usemos `$request-`

`>validated()`. Esto ayuda a prevenir la asignación masiva de campos no deseados.

En resumen:

```text
$request->validated(
// => ['name' => 'Jo
Doe', 'email' =>
'john.doe@example.co

$request->all();
// => ['name' => 'Jo
Doe', 'email' =>
'john.doe@example.co
'role' => 'admin']
```

Recordemos que si `role` no está en `$fillable` en el modelo, no se asignará de todas formas. Pero es una buena práctica usar `validated()` para asegurarnos de que sólo los datos permitidos se procesan.

## Ejemplos de peticiones

Si todo ha ido bien aquí tenemos una API REST completa para el recurso `Note` que podemos probar con herramientas como `Postman` o `RestClient`:

GET api/notes POST api/notes

```text
### listar
todas las notas
GET http://
localhost:8080/
api/notes
Accept:
application/
json
```

```text
,
"done":
false
}
```

```text
### Modificamos
la primera nota
PUT http://
localhost:8080/
api/notes/1
HTTP/1.1
Accept:
application/
json
content-type:
application/
json

{
"id": 1,
"title":
"First Note",

"description":
"This is the
first note,
modified",
"date":
"2023-10-01",
"done":
true
}
```

```text
api/notes/1
Accept:
application/
json
```

```text
### Eliminar la
nota con id 1
DELETE http://
localhost:8080/
api/notes/1
Accept:
application/
json
```

## Conclusiones

- Las **APIs REST** son ideales para aplicaciones SPA, móviles o integraciones.

- Laravel permite definir rutas específicas para API con **prefijos** **automaticos** y **middleware** personalizado.

- Los controladores API están enfocados a **respuestas JSON**.

- **ApiResource** permite estructurar y controlar la salida de tus datos.

