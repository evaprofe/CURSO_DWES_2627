# Práctica: Migraciones y Modelos en Laravel

## Objetivo de la actividad

El objetivo de esta práctica es aprender a

**migraciones** y a

**Eloquent** aplicando las convenciones y propiedades adecuadas de Laravel.

Al finalizar la práctica deberás ser capaz de:

- Crear y modificar tablas utilizando migraciones.

- Comprender el funcionamiento de los métodos `up()` y `down()`.

- Definir claves primarias personalizadas.

- Asociar un modelo Eloquent con su tabla correspondiente.

- Usar correctamente las propiedades `$table`, `$primaryKey`, `$incrementing`, `$keyType` y `$fillable`.

## 1. Crear la migración inicial de employees

El primer paso será crear una migración que defina la tabla **employees**, encargada de almacenar los datos básicos de los empleados.

### Pasos

1. **1**. Accede al entorno PHP del proyecto (contenedor o terminal). Si trabajas con Docker, entra al contenedor de PHP con el comando correspondiente.

2. **2**. Crea una nueva migración utilizando el comando de Artisan para generar la estructura base. Llama a la migración `create_employees_table`.

3. **3**. Abre el archivo de migración generado y define los campos indicados en la siguiente tabla:

`emp_i` `integ` Clave primaria. No

```text
d er
```

autoincremental.

`emp_f` `strin` Nombre del empleado. No

```text
irstn g(100
```

nulo.

```text
ame)
```

`emp_l` `strin` Apellidos del empleado. No

```text
astna g(100
```

nulo.

```text
me)
```

`emp_b` `date` Fecha de nacimiento. No

```text
irth_
```

nulo.

```text
date
```

`emp_h` `date` Fecha de contratación. No

```text
ire_d
```

nulo.

```text
ate
```

`creat` `times` Generados automáticamente `ed_at`, `tamp` con

```text
updat
timestamp
ed_at
```

`s()`.

1. **1**. Guarda los cambios y ejecuta la migración para crear la tabla en la base de datos. Si todo es correcto, Laravel mostrará un mensaje confirmando la creación.

2. **2**. Comprueba en tu herramienta de gestión de base de datos DBeaver, Adminer o similar) que la tabla `employees` se ha creado correctamente.

## 2. Crear una segunda migración para añadir un nuevo campo

En este paso crearás una nueva migración destinada a **modificar la tabla existente** y añadir un campo adicional llamado `salary`.

### Pasos

1. **1**. Genera una nueva migración utilizando el comando de Artisan que permite alterar una tabla existente. LLama a la migración `add_salary_to_employees_table`.

2. **2**. Edita el archivo de migración para añadir el campo `salary` después de la fecha de contratación. Este campo representará el salario del empleado.

`salar` `float` Campo numérico que

```text
y
```

puede contener decimales. No nulo.

1. **1**. Ejecuta la migración para aplicar el cambio. Verifica que el nuevo campo aparece correctamente en la tabla `employees`.

2. **2**. Comprueba en tu base de datos que el campo `salary` ha sido añadido en la posición adecuada.

## * 3. Crear el modelo

`Employee`

Una vez definida la estructura de la tabla, deberás crear un **modelo Eloquent** que represente la entidad `Employee`. El modelo permitirá interactuar con la base de datos utilizando las herramientas de Eloquent en lugar de escribir consultas SQL manuales.

### Pasos

1. **1**. Utiliza el comando de Artisan para generar un modelo llamado `Employee`.

2. **2**. Abre el archivo del modelo, ubicado en `app/Models/Employee.php`.

3. **3**. Configura las siguientes propiedades en el modelo:

`$table` Especifica el nombre de la tabla ( `employees`).

`$prima` Indica el campo usado como clave primaria ( `emp_id`).

```text
ryKey
```

`$incre` Establece si la clave es autoincremental ( `false` en

```text
mentin
```

este caso).

```text
g
```

`$keyTy` Define el tipo de la clave primaria ( `int`).

```text
pe
```

`$filla` Lista los campos que podrán asignarse de forma masiva

```text
ble
```

( `emp_id`, `emp_firstname`, `emp_lastname`, `emp_birth_date`, `emp_hire_date`, `salary`).

1. **1**. Guarda los cambios y asegúrate de que el modelo está correctamente configurado según la tabla creada.

## 4. Verificar el funcionamiento del modelo

Para comprobar que el modelo funciona correctamente, vas a crear una ruta temporal que permita insertar un nuevo registro en la tabla `employees`.

### Pasos

1. **1**. Abre el archivo `routes/web.php` del proyecto.

2. **2**. Crea una nueva ruta `crear-` `empleado` que apunte a una función anónima. Dentro de esta función, utiliza el modelo `Employee` para crear un registro de prueba en la tabla.

3. **3**. Introduce datos ficticios para un empleado (por ejemplo, nombre, apellidos, fecha de nacimiento, fecha de contratación y salario).

4. **4**. Guarda los cambios y accede a la URL que hayas definido para la ruta desde el navegador.

5. **5**. Comprueba en la base de datos que se ha insertado correctamente un nuevo registro.

## 5. Comprobaciones finales

Para completar la práctica, realiza las siguientes verificaciones:

- Confirma que existen **dos** **migraciones** en tu carpeta `database/migrations` y que ambas se han ejecutado correctamente. Puedes usar el comando que muestra el estado de las migraciones.

- Comprueba en la base de datos que la tabla `employees` contiene todos los campos definidos.

- Verifica que el modelo `Employee` está correctamente asociado con la tabla y que permite insertar datos sin errores.

- Si lo deseas, añade capturas de pantalla de los resultados para documentar tu práctica.

## Entrega

Sube a la plataforma de Moodle los siguientes elementos:

- Capturas de pantalla mostrando las migraciones creadas y ejecutadas.

- Evidencia de que el campo `salary` se añadió correctamente.

- Código del modelo `Employee` con sus propiedades configuradas.

- Captura de la base de datos mostrando el registro de prueba insertado.

## Rúbrica de evaluación

**Migración** La migración 25 %

```text
create_e
mployees
```

`_table` se ha generado correctamente y contiene los campos indicados.

**Migración de** Se ha creado y 25 % **modificacion** ejecutado correctamente la migración que añade el campo `salary`.

**Modelo** El modelo está 25 % **Employe** correctamente **e** vinculado a la **e** tabla

```text
employee
```

`s` y usa

```text
emp_id
```

como clave primaria.

**Comprension** El alumno 15 % **del modelo** demuestra entender el uso de propiedades como

```text
$fillabl
```

`e`,

```text
$primary
```

`Key` o

```text
$increme
```

`nting`.

**Prueba** Se demuestra 10 % **funcional** que la inserción de un registro en la tabla se realiza correctamente.

