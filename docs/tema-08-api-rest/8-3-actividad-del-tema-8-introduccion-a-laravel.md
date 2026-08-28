# Práctica: Creación de una API REST para Products

## Objetivo de la actividad

En esta práctica vas a aplicar todo lo aprendido sobre

**de APIs REST** para desarrollar un servicio completo que gestione productos ( Products). Al finalizar, habrás implementado todos los elementos esenciales de una API moderna:

- Definición de rutas API ( `api.php`)

- Creación de un controlador con métodos CRUD ( `index`, `show`, `store`, `update`, `destroy`)

- Uso de modelos y migraciones

- Validación de datos mediante Form Request

- Personalización de la salida con API Resources

- Pruebas de los endpoints con Postman o REST Client

Este ejercicio te permitirá consolidar los conceptos vistos con el recurso `Note` y demostrar que puedes aplicarlos de forma autónoma en un nuevo caso.

## Instrucciones

Sigue los pasos **en orden**, comprobando el funcionamiento de cada parte antes de pasar a la siguiente. Puedes basarte en el ejemplo del tema anterior sobre Notes, adaptándolo al nuevo recurso Product. O continuar con el proyecto de las prácticas anteriores que ya trata sobre productos.

### #1. 1. Preparar el entorno

1. **1**. Abre el proyecto de Laravel que has elegido.

2. **2**. Asegúrate de tener las rutas API activadas. Si no lo hiciste antes, ejecuta:

```text
php artisan
install:api
```

3. **3**. Verifica que los contenedores de Docker estén funcionando:

```text
docker compose up -d
```

### 2. Crear la tabla y el modelo de productos

Si no tienes ya una tabla `products`, crea una nueva migración y el modelo asociado. Si ya lo tienes continúa con el siguiente paso.

1. **1**. Genera la migración para la tabla `products`:

```text
php artisan
make:migration
create_products_table
```

2. Edita la migración para que incluya los siguientes campos:

```text
$table->id();
$table-
>string('name');
$table-
>text('description')-
>nullable();
$table-
>decimal('price', 8,
2);
$table-
>integer('stock');
$table->timestamps();
```

3. Ejecuta las migraciones:

```text
php artisan migrate
```

4. Crea el modelo `Product`:

```text
php artisan
make:model Product
```

5. Define en el modelo los campos permitidos para asignación masiva:

```text
protected $fillable =
['name',
'description',
'price', 'stock'];
```

### 3. Crear el controlador API

1. **1**. Crea el controlador `ProductController` en el espacio de nombres `api`.

2. **2**. Implementa en él los cinco métodos principales ( `index`, `store`, `show`, `update`, `destroy`) para manejar el CRUD. Puedes basarte en el ejemplo del recurso `Note` del tema.

### 4. Definir las rutas

1. **1**. Crea las rutas para manejar los productos, utiliza `apiResource`:

2. **2**. Comprueba que se hayan creado las rutas con:

```text
php artisan
route:list --
path=api/products
```

### 5. Crear el API Resource

1. **1**. Genera la clase `ProductResource`. Esta clase te permitirá personalizar la estructura de los datos JSON que devuelve la API. Un ejemplo de estructura sería: ``json { "id": 1, "nombre": "Camiseta", "precio": "$19.99", "stock": 25, "descripcion": "Camiseta de algodón" } \`\`\` 2. Usa esta clase en el controlador para las respuestas JSON.

### 6. Validar los datos con una clase Form Request

1. **1**. Crea la clase `ProductRequest` para validar los datos de entrada al crear o actualizar un producto. Las reglas de validación podrían ser:

- `name`: obligatorio, cadena de texto, mínimo 3 y máximo 255 caracteres

- `description`: obligatorio, cadena de texto, mínimo 10 caracteres

- `price`: obligatorio, numérico, mínimo 0, máximo 9999.99

- `stock`: obligatorio, entero, mínimo 0, máximo 10000

```text
2. Modifica los
```

`métodos` store() `y` update()` del controlador para usar esta clase:

### 7. Probar la API

Crea un archivo `products.rest` en la raíz del proyecto (o usa Postman).

2. **2**. Escribe las peticiones para probar todos los endpoints:

- **GET** `/api/products` → Listar todos los productos

- **POST** `/api/products` → Crear un nuevo producto

- **GET** `/api/products/{id}` → Mostrar un producto

- **PUT** `/api/products/{id}` → Modificar un producto

- **DELETE** `/api/products/{id}` → Eliminar un producto

3. **3**. Comprueba que todas las operaciones devuelven los **cédigos HTTP** **correctos** ( `200`, `201`, `204`, etc.) y que la respuesta JSON tiene el formato definido en `ProductResource`.

## Entrega

Crea un pdf con los siguientes contenidos:

- Capturas de las peticiones y respuestas en REST Client, o la extensión que utilices

- Un breve archivo `README.md` explicando las rutas y cómo probarlas

- Añade al pdf el código de la migración, el modelo, el controlador, la clase Resource y la clase Request

## Rúbrica de evaluación

**Estructura del** Configura 15 % **proyecto y** correctamente **rutas** las rutas API y el controlador con los métodos CRUD.

**Migración y** La tabla y el 20 % **modelo** modelo

```text
Product
```

están correctamente definidos y funcionales.

**de** Implementa 20 % **datos** correctamente

```text
ProductR
```

`equest` y usa

```text
$request
-
>validat
```

`ed()` en lugar de

```text
$request
```

- `->all()`.

**Formato de** Devuelve las 20 % **salida** respuestas **(Resource)** formateadas correctamente usando

```text
ProductR
```

`esource`.

**Pruebas y** Todas las 25 % **funcionamiento** operaciones **general** CRUD funcionan y devuelven los códigos HTTP adecuados.

**Total**: 100 %

