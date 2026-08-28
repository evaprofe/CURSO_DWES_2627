# Práctica: Generación de Datos Aleatorios con Factory, Seeder y Faker

## Objetivo de la actividad

El objetivo de esta práctica es que el alumno aprenda a **automatizar la creación** **de datos en la base de datos** mediante el uso de **factories, seeders y Faker** en Laravel.

Al finalizar la actividad, el alumno será capaz de:

- Crear migraciones y modelos para poblar tablas con datos realistas.

- Configurar una **factory** para generar datos de prueba automáticos.

- Crear un **seeder** que utilice la factory para insertar múltiples registros.

- Utilizar **Faker** para generar nombres, correos, textos y números aleatorios.

- Ejecutar seeders desde Artisan y verificar los resultados en la base de datos.

## Instrucciones paso a paso

### 1. Crear la migración y el modelo Employee

1. **1**. Abre la terminal y crea el modelo junto con su migración:

```text
php artisan
make:model Employee -
m
```

2. **2**. En el archivo de migración generado ( `database/migrations/` xxxx_create_employees_table.p `hp`), define la estructura de la tabla:

```text
public function
{

Schema::create(
function (Bluepr
{
$table->id
$table-
>string('name');
$table-
>string('email')
$table-
>string('departm
$table-
>decimal('salary
$table->ti
});
}
```

3. **3**. Ejecuta la migración para crear la tabla:

```text
php artisan migrate
```

### 2. Configurar el modelo

`Employee`

Abre el archivo `app/Models/` `Employee.php` y modifica su contenido para permitir asignación masiva y el uso de factories:

```text
<?php

namespace App\Model

use
Illuminate\Database
use Illuminate\Data

class Employee exte
{
use HasFactory;

protected $fill
'name',
'email',
'department
'salary'
];
}
```

### 3. Crear la Factory

1. **1**. Crea una **factory** asociada al modelo `Employee`:

```text
php artisan
make:factory
EmployeeFactory --
model=Employee
```

Abre `database/factories/` `EmployeeFactory.php` y define los datos que generará **Faker**:

```text
public function
definition(): arr
{
return [
'name' =>
$this->faker->nam
'email' =>
$this->faker-
>unique()-
>safeEmail(),
'department
$this->faker-
>randomElement([
'HR', 'Marketing
'Sales', 'Finance
'salary' =>
$this->faker-
>numberBetween(20
60000),
];
}
```

**Explicacion rapida**:

- `name()`: genera nombres reales.

- `unique()->safeEmail()`: crea correos electrónicos válidos y únicos.

- `randomElement()`: selecciona un valor aleatorio de la lista dada.

- `numberBetween()`: crea un número entre los valores definidos.

### 4. Crear el Seeder

1. **1**. Crea un seeder específico para empleados:

```text
php artisan
make:seeder
EmployeeSeeder
```

Abre `database/seeders/` `EmployeeSeeder.php` y añade:

```text
use App\Models\Employee;

public function run():
void
{

Employee::factory()-
>count(100)->create();
}
```

Esto generará **100 registros aleatorios** usando la factory.

### 5. Registrar el seeder en

`DatabaseSeeder`

Abre `database/seeders/` `DatabaseSeeder.php` y añade la llamada al seeder recién creado:

```text
public function r
{
$this-
>call(EmployeeSee
}
```

### 6. Ejecutar los seeders

1. **1**. Ejecuta todos los seeders registrados:

```text
php artisan db:seed
```

2. **2**. Si deseas ejecutar solo este seeder:

```text
php artisan db:seed
--
class=EmployeeSeeder
```

3. **3**. Verifica en tu herramienta de base de datos DBeaver, VSCode o Workbench) que la tabla `employees` contiene **100 registros** generados automáticamente.

### 7. Opcional) Reiniciar las

### migraciones y seeders

Si quieres limpiar la base de datos y volver a generar los datos desde cero:

```text
php artisan
migrate:fresh --seed
```

Esto borrará todas las tablas, volverá a crearlas y ejecutará automáticamente el seeder configurado en `DatabaseSeeder`.

## Entrega

- Captura del código del modelo, factory y seeder.

- Captura de la tabla `employees` mostrando al menos 10 registros generados.

- Evidencia de ejecución del comando `php artisan db:seed`.

## Rúbrica de evaluación

**Migración de** Contiene los 20 % **employee** campos `id`, **S creada** `name`, **correctamente** `email`,

```text
departme
```

`nt`, `salary`,

```text
timestam
```

`ps`.

**Modelo** Usa 20 %

```text
$fillabl
```

**configurado** `e` y

```text
HasFacto
```

`ry`.

**Factory** Genera datos 25 % **funcional y** realistas **coherente** usando Faker (nombre, email, departamento, salario).

**Seeder** Crea 100 25 % **funcional** empleados correctamente y está registrado en

```text
Database
Seeder.p
```

`hp`.

**Verificacion y** Evidencia de 10 % **ejecución** ejecución ( `php`

```text
artisan
```

`db:seed`) y datos visibles en la base de datos.

IES JUAN DE GARAY - 2º DWES (Curso original de Beteta adaptado por Eva María Gómez Abad )
