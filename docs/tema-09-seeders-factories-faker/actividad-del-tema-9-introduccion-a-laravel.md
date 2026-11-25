# Práctica: Generación de Datos Aleatorios con Factory, Seeder y Faker

## Objetivo de la actividad

El objetivo de esta práctica es que el alumno aprenda a **automatizar la creación de datos en la base de datos** mediante el uso de **factories, seeders y Faker** en Laravel.

Al finalizar la actividad, el alumno será capaz de:

- Crear migraciones y modelos para poblar tablas con datos realistas.
- Configurar una **factory** para generar datos de prueba automáticos.
- Crear un **seeder** que utilice la factory para insertar múltiples registros.
- Utilizar **Faker** para generar nombres, correos, textos y números aleatorios.
- Ejecutar seeders desde Artisan y verificar los resultados en la base de datos.

---

## Instrucciones paso a paso

### 1. Crear la migración y el modelo `Employee`

1. Abre la terminal y crea el modelo junto con su migración:

    ```bash
    php artisan make:model Employee -m
    ```

2. En el archivo de migración generado (`database/migrations/xxxx_create_employees_table.php`), define la estructura de la tabla:

    ```php
    public function up(): void
    {
       Schema::create('employees', function (Blueprint $table) {
          $table->id();
          $table->string('name');
          $table->string('email')->unique();
          $table->string('department');
          $table->decimal('salary', 8, 2);
          $table->timestamps();
       });
    }
    ```

3. Ejecuta la migración para crear la tabla:

    ```bash
    php artisan migrate
    ```

---

### 2. Configurar el modelo `Employee`

Abre el archivo `app/Models/Employee.php` y modifica su contenido para permitir asignación masiva y el uso de factories:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Employee extends Model
{
    use HasFactory;

    protected $fillable = [
        'name',
        'email',
        'department',
        'salary'
    ];
}
```

---

### 3. Crear la Factory

1. Crea una **factory** asociada al modelo `Employee`:

    ```bash
    php artisan make:factory EmployeeFactory --model=Employee
    ```

2. Abre `database/factories/EmployeeFactory.php` y define los datos que generará **Faker**:

    ```php
    public function definition(): array
    {
       return [
          'name' => $this->faker->name(),
          'email' => $this->faker->unique()->safeEmail(),
          'department' => $this->faker->randomElement(['IT', 'HR', 'Marketing', 'Sales', 'Finance']),
          'salary' => $this->faker->numberBetween(20000, 60000),
       ];
    }
    ```

**Explicación rápida**:

- `name()`: genera nombres reales.
- `unique()->safeEmail()`: crea correos electrónicos válidos y únicos.
- `randomElement()`: selecciona un valor aleatorio de la lista dada.
- `numberBetween()`: crea un número entre los valores definidos.

---

### 4. Crear el Seeder

1. Crea un seeder específico para empleados:

    ```bash
    php artisan make:seeder EmployeeSeeder
    ```

2. Abre `database/seeders/EmployeeSeeder.php` y añade:

```php
use App\Models\Employee;

public function run(): void
{
      Employee::factory()->count(100)->create();
}
```

Esto generará **100 registros aleatorios** usando la factory.

---

### 5. Registrar el seeder en `DatabaseSeeder`

1. Abre `database/seeders/DatabaseSeeder.php` y añade la llamada al seeder recién creado:

    ```php
    public function run(): void
    {
       $this->call(EmployeeSeeder::class);
    }
    ```

---

### 6. Ejecutar los seeders

1. Ejecuta todos los seeders registrados:

    ```bash
    php artisan db:seed
    ```

2. Si deseas ejecutar solo este seeder:

    ```bash
    php artisan db:seed --class=EmployeeSeeder
    ```

3. Verifica en tu herramienta de base de datos (DBeaver, VSCode o Workbench) que la tabla `employees` contiene **100 registros** generados automáticamente.

---

### 7. (Opcional) Reiniciar las migraciones y seeders

Si quieres limpiar la base de datos y volver a generar los datos desde cero:

```bash
php artisan migrate:fresh --seed
```

Esto borrará todas las tablas, volverá a crearlas y ejecutará automáticamente el seeder configurado en `DatabaseSeeder`.

---

## Entrega

- Captura del código del modelo, factory y seeder.
- Captura de la tabla `employees` mostrando al menos 10 registros generados.
- Evidencia de ejecución del comando `php artisan db:seed`.

---

## Rúbrica de evaluación

| Criterio | Descripción | Peso |
|---|---|---|
| **Migración de `employees` creada correctamente** | Contiene los campos `id`, `name`, `email`, `department`, `salary`, `timestamps`. | 20 % |
| **Modelo `Employee` configurado correctamente** | Usa `$fillable` y `HasFactory`. | 20 % |
| **Factory funcional y coherente** | Genera datos realistas usando Faker (nombre, email, departamento, salario). | 25 % |
| **Seeder funcional** | Crea 100 empleados correctamente y está registrado en `DatabaseSeeder.php`. | 25 % |
| **Verificación y ejecución** | Evidencia de ejecución (`php artisan db:seed`) y datos visibles en la base de datos. | 10 % |

**Total: 100 %**
