# Creación de datos iniciales y prueba: Seeders, Factories y Faker

## Introducción

En los temas anteriores aprendimos a crear tablas con migraciones y a manipular datos manualmente. Sin embargo, en muchos proyectos reales necesitamos:

- Crear **datos iniciales**, como usuarios, roles o productos básicos.

- Generar **datos masivos de prueba** para desarrollo y pruebas funcionales.

Laravel nos ofrece tres herramientas clave para ello:

- **Seeders**: para insertar datos conocidos y permanentes.

- **Factories**: para generar datos aleatorios.

- **Faker**: para crear contenido realista de forma automática.

Utilizaremos la base de datos `laravel` y actualizaremos el archivo `.env` para garantizar la conexión correcta.

## Preparación del Entorno

vamos a comenzar desde un proyecto Laravel nuevo y la base de datos también nueva. La base de datos podemos eliminar la que teníamos `drop database` `laravel;` y crear una nueva `create` `database laravel;`.

Para el proyecto Laravel:

```text
composer create-project
laravel/laravel example-
app
cd example-app
```

o podemosusar el proyecto que teníamos eliminando todos los archivos, controladores, migraciones recursos, vistas etc, que hemos creado en los temas anteriores. De esta manera evitamos conflictos con los archivos que vamos a crear en este tema.

Una vez creado o limpiado el proyecto, editamos el archivo `.env` para configurar la conexión a la base de datos:

```text
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=alumno
DB_PASSWORD=alumno
```

Luego hacemos la migración inicial:

```text
php artisan migrate
```

### Crear la tabla y el modelo

`Product`

En este tema vamos manejar una tabla de productos. Creamos la migración y el modelo:

```text
php artisan make:model
Product -m
```

Este comando crea el modelo `Product.php` en `app/Models` y la migración en `database/migrations`.

Editamos la migración:

```text
<?php
Schema::create('p
function (Blueprin
{
$table->id();
$table->string
50);
$table-
>string('short_des
200);
$table-
>string('descripti
$table->float
>default(20);
$table->timest
});
```

Editamos el modelo `Product.php` para permitir asignación masiva:

```text
<?php
class Product
extends Model
{
protected
$guarded = [];
}
```

Ejecutamos la migración:

```text
php artisan migrate
```

### Controlador, ruta y vista

Ahora que ya tenemos la tabla y el modelo, vamos a crear el circuito completo para mostrar los productos en una vista.

Creamos el controlador:

```text
php artisan
make:controller
ProductController
```

Ruta en `web.php`:

```text
<?php
use
App\Http\Controlle

Route::get('/produ
[ProductControlle
>name('product.ind
```

Método `index()` en el controlador:

```text
<?php
use
App\Models\Product
use
Illuminate\View\Vi

public function
index(): View
{
$products =
Product::all();
return
view('products.ind
compact('products
}
```

Antes de crear la vista vamos a crear el layout `layouts/app.blade.php` para que todas las vistas tengan un diseño común.

```text
<!DOCTYPE html>
<html lang="es">
<head>
<meta
charset="UTF-8">
<meta
name="viewport"
content="width=de
width, initial-
scale=1.0">

<title>@yield('ti
'Mi Aplicación')<
title>
</head>
<body>
<header>
<h1>Mi
Tienda</h1>
<!-- Aquí
podría ir una ba
navegación -->
</header>
<main>

@yield('content'
</main>

<footer>
<p>&copy
date('Y') }} Mi
Tienda</p>
</footer>
</body>
</html>
```

Ahora que ya tenemos el layout, podemos crear la vista `index.blade.php` para listar los productos. Creamos la carpeta `products` dentro de `resources/views` y dentro de ella el archivo `index.blade.php`.

Creamos `resources/views/products/` `index.blade.php`:

En este caso la vista es sencilla, solo muestra una lista de productos, en forma de `cards`. Más adelante con CSS ya le cambiaremos el aspecto. Utilizamos `@forelse` para mostrar un mensaje si no hay productos.

```text
@extends
('layouts.app')

@section('title'
'Products')

@section('content
<div
class="product-
container">
@forelse
($products as
$product)
<div
class="card">

<h2>{{ $product-
>name }}</h2>

<p>{{ $product-
>short_descriptio
}}</p>

<p><strong>${{
$product->price }
</strong></p>
</div
@empty
<p>No
hay productos
disponibles.</p>
@endforel
</div>
@endsection
```

Añadimos un par de reglas CSS en `public/assets/css/app.css` para mejorar la presentación de la vista:

```text
.product-
container {
display:
flex;
flex-wrap:
wrap;
gap: 20px;
}

.card {
border: 1px
solid #ccc;
padding:
15px;
width:
200px;
box-shadow:
2px 2px 10px
rgba(0,0,0,0.1);
}
```

Nos falta añadir el CSS a la vista. Editamos el layout `app.blade.php` para incluir el CSS

```text
<head>
...
<link
rel="stylesheet"
href="{{
asset('css/
app.css') }}">
...
</head>
```

De momento esta es la vista, sin datos. Vamos a crear datos iniciales y de prueba para que se vea mejor.

## Seeders

### Introducción a Seeders

Los **seeders** permiten poblar la base de datos con datos iniciales que queremos tener siempre disponibles.

### Crear y registrar un seeder

```text
php artisan make:seeder
ProductSeeder
```

En el seeder:

```text
<?php

namespace Databas

use
Illuminate\Databa
use Illuminate\Da
use App\Models\P

class ProductSeed
{
/**
* Run the dat
*/
public functi
{
// Vamos
Product:
'name
'sho
producto 1',
'desc
producto 1',
'pric
]);
Product:
'name
'sho
producto 2',
'desc
producto 2',
'pric
]);
Product:
'name
'sho
producto 3',
'desc
producto 3',
```

```text
p,
'pric
]);
Product:
'name
'sho
producto 4',
'desc
producto 4',
'pric
]);
}
}
```

Editamos `DatabaseSeeder.php` para incluirlo:

```text
<?php
public function run
{
$this-
>call(ProductSeeder:

}
```

Ejecutamos:

```text
php artisan db:seed
```

Ahora ya tenemos productos en la base de datos y se muestran en la vista. Puedes comprobar que esxisten en la base de datos con `select *` `from products;`.

### Ejecutar un seeder específico

Si en alguna ocasión queremos ejecutar un seeder específico, podemos usar:

```text
php artisan db:seed --
class=ProductSeeder
```

### Vista de los productos

Vamos al navegador, accedemos a http:// localhost:8080/products y vemos la lista de productos que hemos creado con el seeder.

Ahora ya tenemos productos en la base de datos y se muestran en la vista.

## Factories

### Introducción a Factories

Al probar su aplicación o sembrar su base de datos, es posible que necesites insertar algunos registros en ella. En lugar de especificar manualmente el valor de cada columna, `Laravel` te permite definir un conjunto de atributos predeterminados para cada uno de sus modelos `Eloquent` mediante fábricas de modelos.

Una **factory** es una clase que define un modelo y cómo generar datos de prueba para ese modelo. Las **factories** permiten generar muchos datos automáticamente para pruebas y desarrollo.

### = Crear una factory

```text
php artisan make:factory
ProductFactory --
model=Product
```

Este comando debe crear el archivo `database/factories/` `ProductFactory.php`.

Definición simple:

Antes de modificar la función hay que importar `Str`:

```text
<?php
use
Illuminate\Support\Str;
```

```text
<?php
public function
definition(): ar
{
return [
'name' =>
Str::random(10),

'short_descriptio
=> Str::random(30

'description' =>
Str::random(50),
'price' =
random_int(5, 100
];
}
```

Es necesario modificar el modelo para que use la factory.

Primero importamos `HasFactory`, y luego modificamos la clase para usar el trait `HasFactory`:

```text
<?php
use
Illuminate\Database\Eloq
```

```text
<?php
class Product
extends Model
{
use
HasFactory;

protected
$guarded = [];
}
```

### Usar factory en un seeder

Modificamos el contenido del seeder `ProductSeeder.php` para usar la factory. En la función `run()` quitamos los prodcutos que habíamos creado manualmente y añadimos la factory.

Primero agregamos la importación del modelo `Product`:

```text
<?php
use
App\Models\Product;
```

```text
<?php
public function
run(): void
{
// Crear 10
productos usando l
factory

Product::factory(
>count(10)-
>create();
}
```

Ahora como no tenemos nada importante en la base de datos vamos a ejecutar las migraciones de nuevo, para eliminar los datos que teníamos y crear la tabla de nuevo:

```text
php artisan
migrate:fresh
php artisan db:seed
```

Esto nos borrará todos los datos y creará la tabla de nuevo, luego ejecutará el seeder que a su vez usará la factory para crear 10 productos con datos aleatorios.

Si vamos a la base de datos veremos que tenemos 10 productos con datos aleatorios:

Hemos conseguido tener 10 productos en la base de datos con datos aleatorios en segundos. Pero los datos no son muy realistas, ya que son cadenas aleatorias. Para conseguir unos datos más realistas veremos Faker en el siguiente apartado.

## Faker

### Introducción a Faker

**Faker** genera datos aleatorios que parecen reales (nombres, direcciones, textos.). Laravel lo usa integrado en `Factory`.

### Tabla de métodos útiles de Faker

`name(` Nombre Juan Pérez completo

```text
)
```

`email` Email juan@mail.com realista

```text
()
```

`sente` Frase con Lorem ipsum `n` palabras dolor sit amet.

```text
nce(n
)
```

`parag` Párrafo de Lorem ipsum texto dolor sit

```text
raph(
```

amet.

```text
)
```

`text(` Texto de Texto aleatorio longitud `n`

```text
n)
```

`word(` Una palabra laptop aleatoria

```text
)
```

`numbe` Número 47 entre `a` y

```text
rBetw
b
een(a
, b)
```

`rando` Valor de un 'IT' array

```text
mElem
ent([
...])
```

`boole` true o false true aleatorio

```text
an()
```

`date(` Fecha 2023 01 15 aleatoria

```text
)
```

`compa` Nombre de Grupo Pérez empresa SL

```text
ny()
```

### Aplicar Faker a ProductFactory

```text
<?php
public function
definition(): ar
{
return [
'name' =>
$this->faker-
>word(),

'short_descriptio
=> $this->faker-
>sentence(5),

'description' =>
$this->faker-
>paragraph(),
'price' =
$this->faker-
>numberBetween(10
100)
];
}
```

Ahora, al igual que en el punto anterior como no tenemos nada importante en la base de datos vamos a ejecutar las migraciones de nuevo, para eliminar los datos que teníamos y crear la tabla de nuevo, esta vez lo hacemos en un solo comando:

```text
php artisan
migrate:fresh --seed
```

Siguen siendo datos aleatorios, pero ahora son más realistas. Y a la hora de ver cómo quedan en una vista, para un prototipo, es mucho mejor.

Comprobamos la vista: http:// localhost:8080/products, vamos a ver los productos con datos más realistas:

## Conclusiones

- Los **seeders** son útiles para poblar la base de datos con datos estáticos.

- Las **factories** permiten generar cientos de registros en segundos.

- **Faker** mejora la calidad de los datos de prueba.

- Todo esto acelera y mejora el desarrollo realista de tu aplicación.

