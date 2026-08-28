¿Quién eres? Comprobar email y contraseña ¿Qué puedes hacer? Permitir editar solo tus notas **im** es el sistema ligero de autenticación de Laravel para:

POST /login (email + password) comprobar credenciales usuario válido token de acceso GET /api/notes + Bearer Token auth:sanctum valida el token

respuesta JSON

**_|** Usuario autenticado `<?php` `use App\Models\User;` `use Illuminate\Support\Facades\Hash;` `User::create([` `'name' => 'Ana',` `'email' => 'ana@example.com',` `'password' => Hash::make('secret123')` `]);` `User::create([` `'name' => 'Luis',` `'email' => 'luis@example.com',` `'password' => Hash::make('secret123')` `]);` `return 'Usuarios creados';`

`<?php` `> User::All();` `= Illuminate\Database\Eloquent\Collection {#8294`

```text
all: [
App\Models\User {#8291
id: 1,
name: "Ana",
email: "ana@example.com",
email_verified_at: null,
<?php
#password: "\$2y\$12\
use App\Models\User;
$WOjR5Cv4XvZgVSSi99mW8OhGX3Mg5ucYbP7LwYI/Os/
use Illuminate\Support\Facades\Hash;
OGlRdpk.0O",
use Illuminate\Support\Facades\Route;
#remember_token: null,
created_at: "2026-04-26 08:28:54",
Route::get('/create-users', function () {
updated_at: "2026-04-26 08:28:54",
User::create([
},
'name' => 'Ana',
App\Models\User {#8298
'email' => 'ana@example.com',
id: 3,
'password' => Hash::make('secret123')
name: "Luis",
]);
email: "luis@example.com",
email_verified_at: null,
User::create([
#password: "\$2y\$12\
'name' => 'Luis',
$G9Vh/2xINmOfDI9G5oXgweRx2tBly9tarxuoC75plbB/8ha6XI2/
'email' => 'luis@example.com',
i",
'password' => Hash::make('secret123')
#remember_token: null,
]);
created_at: "2026-04-26 08:37:32",
updated_at: "2026-04-26 08:37:32",
return 'Usuarios creados';
},
});
],
}
```

`<?php` `use Illuminate\Database\Migrations\Migration;` `use Illuminate\Database\Schema\Blueprint;` `use Illuminate\Support\Facades\Schema;` `return new class extends Migration` `{` `public function up(): void` `{`

```text
Schema::table('notes', function (Blueprint
$table) {
$table->foreignId('user_id')-
>after('id')->constrained()->onDelete('cascade');
});
}
public function down(): void
{
Schema::table('notes', function (Blueprint
$table) {
$table-
>dropConstrainedForeignId('user_id');
});
}
};
```

`<?php` `class Note extends Model` `{` `protected $fillable = ['title', 'description',` `'date', 'done', 'user_id'];` `public function user()` `{`

```text
return $this->belongsTo(User::class);
}
}
```

`<?php` `use Laravel\Sanctum\HasApiTokens;` `class User extends Authenticatable` `{` `use HasApiTokens; // Trait necesario para` `Sanctum` `public function notes()` `{`

```text
return $this->hasMany(Note::class);
}
}
```

`<?php` `trait HasSampleTrait` `{` `public function traitMethod()` `{`

```text
return 'Este es un método del trait
HasSampleTrait. ';
}
}
class Sample
{
use HasSampleTrait; // Podemos usar varios
traits
function example()
{
return "Este es un método de la clase
Sample. ";
}
//...
}
$sample = new Sample();
echo $sample->example(); // Este es un método de la
clase Sample."
echo $sample->traitMethod(); // Este es un método
del trait HasSampleTrait.
```

`<?php` `namespace App\Http\Controllers\Api;` `use App\Http\Controllers\Controller;` `use Illuminate\Http\JsonResponse;` `use Illuminate\Http\Request;` `use Illuminate\Support\Facades\Hash;` `use App\Models\User;` `class AuthController extends Controller` `{` `public function login(Request $request):` `JsonResponse` `{`

```text
$request->validate([
'email' => 'required|email',
'password' => 'required|string'
]);
$user = User::where('email', $request-
>email)->first();
if (! $user || ! Hash::check($request-
>password, $user->password)) {
return response()->json([
'success' => false,
'message' => 'Credenciales
incorrectas.'
], 401);
}
$token = $user->createToken('api-token')-
>plainTextToken;
return response()->json([
'success' => true,
'message' => 'Login correcto.',
'token' => $token,
'user' => $user
], 200);
}
public function me(Request $request):
JsonResponse
{
return response()->json([
'success' => true,
'data' => $request->user()
], 200);
}
public function logout(Request $request):
JsonResponse
{
$request->user()->currentAccessToken()-
>delete();
return response()->json([
'success' => true,
'message' => 'Logout correcto.'
], 200);
}
}
```

`<?php` `use App\Http\Controllers\Api\AuthController;` `use App\Http\Controllers\Api\NoteController;` `use Illuminate\Support\Facades\Route;` `Route::post('/login', [AuthController::class,` `'login']);` `Route::middleware('auth:sanctum')->group(function` `() {` `Route::get('/me', [AuthController::class,` `'me']);` `Route::post('/logout', [AuthController::class,` `'logout']);` `Route::apiResource('notes',` `NoteController::class);` `});`

**zed**.

`### Login` `POST http://localhost:8080/api/login HTTP/1.1` `Accept: application/json` `content-type: application/json` `{` `"email": "ana@example.com",` `"password": "secret123"` `}`

`{` `"success": true,` `"message": "Login correcto.",` `"token": "1|xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",` `"user": {`

```text
"id": 1,
"name": "Ana",
"email": "ana@example.com"
}
}
```

`### Usuario autenticado` `GET http://localhost:8080/api/me` `Accept: application/json` `Authorization: Bearer 1|` `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

```text
"success": true,
"data": {
"id": 1,
"name": "Ana",
"email": "ana@example.com"
}
}
```

`<?php` `public function store(NoteRequest $request):` `JsonResponse` `{` `$note = Note::create([`

```text
...$request->validated(),
'user_id' => $request->user()->id,
]);
return response()->json([
'success' => true,
'message' => 'Nota creada correctamente.',
'data' => new NoteResource($note)
], 201);
}
```

`<?php` `public function toArray($request): array` `{` `return [`

```text
'id' => $this->id,
'titulo' => $this->title,
'descripcion' => $this->description,
'fecha' => $this->date,
'estado' => $this->done ? 'Completada':
'Pendiente',
'usuario' => [
'id' => $this->user->id,
'name' => $this->user->name,
],
];
}
```

`<?php` `public function index(): JsonResponse` `{` `return response()->json([`

```text
'success' => true,
'data' =>
NoteResource::collection(Note::with('user')->get())
], 200);
}
```

`{` `"success": true,` `"data": [`

```text
{
"id": 1,
"titulo": "Nota de Ana",
"descripcion": "Texto de ejemplo",
"fecha": "2026-04-22",
"estado": "Pendiente",
"usuario": {
"id": 1,
"name": "Ana"
}
},
{
"id": 2,
"titulo": "Nota de Luis",
"descripcion": "Otra nota de ejemplo",
"fecha": "2026-04-23",
"estado": "Completada",
"usuario": {
"id": 3,
"name": "Luis"
}
}
]
}
```

`GET http://localhost:8080/api/notes/1` `Accept: application/json` `Authorization: Bearer TU_TOKEN`

`<?php` `public function show(Note $note): JsonResponse` `{` `$note->load('user');` `return response()->json([`

```text
'success' => true,
'data' => new NoteResource($note)
], 200);
}
```

`{` `"success": true,` `"data": {`

```text
"id": 1,
"titulo": "Nota de Ana",
"descripcion": "Texto de ejemplo",
"fecha": "2026-04-22",
"estado": "Pendiente",
"usuario": {
"id": 1,
"name": "Ana"
}
}
}
```

`<?php` `namespace App\Policies;` `use App\Models\Note;` `use App\Models\User;` `class NotePolicy` `{` `public function update(User $user, Note $note):` `bool` `{`

```text
return $user->id === $note->user_id;
}
public function delete(User $user, Note $note):
bool
{
return $user->id === $note->user_id;
}
}
```

`<?php` `use` `Illuminate\Foundation\Auth\Access\AuthorizesRequests;` `class NoteController extends Controller` `{` `use AuthorizesRequests;` `//...` `}`

`<?php` `public function update(NoteRequest $request, Note` `$note): JsonResponse` `{` `$this->authorize('update', $note);` `$note->update($request->validated());` `return response()->json([`

```text
'success' => true,
'message' => 'Nota actualizada
correctamente.',
'data' => new NoteResource($note)
], 200);
}
```

`<?php` `public function destroy(Note $note): JsonResponse` `{` `$this->authorize('delete', $note);` `$note->delete();` `return response()->json([`

```text
'success' => true,
'message' => 'Nota eliminada
correctamente.'
], 200);
}
```

Cliente hace login

Laravel valida credenciales

Sanctum genera token

Cliente guarda token

Petición con Bearer Token

Sanctum identifica al usuario

Controlador recibe request user

Acción permitida por Policy?

`POST http://localhost:8080/api/login HTTP/1.1` `Accept: application/json` `content-type: application/json` `{` `"email": "ana@example.com",` `"password": "secret123"` `}`

`GET http://localhost:8080/api/notes` `Accept: application/json` `Authorization: Bearer TU_TOKEN`

`POST http://localhost:8080/api/notes HTTP/1.1` `Accept: application/json` `Authorization: Bearer TU_TOKEN` `content-type: application/json` `{` `"title": "Nota de Ana",` `"description": "Texto de ejemplo",` `"date": "2026-04-22",` `"done": false` `}`

`PUT http://localhost:8080/api/notes/1 HTTP/1.1` `Accept: application/json` `Authorization: Bearer TU_TOKEN` `content-type: application/json` `{` `"title": "Nota modificada",` `"description": "Texto actualizado",` `"date": "2026-04-22",` `"done": true` `}`

`PUT http://localhost:8080/api/notes/2 HTTP/1.1` `Accept: application/json` `Authorization: Bearer TU_TOKEN` `content-type: application/json` `{` `"title": "Intento no permitido",` `"description": "No debería funcionar",` `"date": "2026-04-22",` `"done": false` `}`

`POST http://localhost:8080/api/logout` `Accept: application/json` `Authorization: Bearer TU_TOKEN`

**+ policies**.

IES JUAN DE GARAY - 2º DWES (Curso original de Beteta adaptado por Eva María Gómez Abad )
