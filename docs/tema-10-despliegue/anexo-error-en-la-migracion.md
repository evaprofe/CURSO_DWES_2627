# Error en la migración

En la última migración, la base de datos ha saltado con un error que impide que se complete el proceso. En este caso el error es el siguiente:

```text
Error ejecutando comando Artisan: SQLSTATE[42000]: Syntax error or access violation: 1071 Specified key was too long; max key length is 1000 bytes (Connection: mysql, Host: sql110.infinityfree.com, Port: 3306, Database: if0_42257514_laravel, SQL: create table password_reset_tokens (email varchar(255) not null, token varchar(255) not null, created_at timestamp null, primary key (email)) default character set utf8mb4 collate 'utf8mb4_unicode_ci')
```

Este error ocurre porque la longitud máxima de la clave primaria en MySQL es de 1000 bytes, y al usar el conjunto de caracteres `utf8mb4`, cada carácter puede ocupar hasta 4 bytes. Por lo tanto, una columna `varchar(255)` puede ocupar hasta 1020 bytes, lo que excede el límite permitido para la clave primaria.

Para solucionar este problema, sin tocar la configuración de la base de datos, se puede cambiar la longitud de la columna `email` en la tabla `password_reset_tokens` a un valor menor, como `varchar(191)`, que es seguro para el conjunto de caracteres `utf8mb4`.

Otra solución es cambiar el tamaño de los campos `string` que por defecto tienen un tamaño de 255 caracteres. Para que Laravel aplique un tamaño de 191 caracteres.

Tenemos que modificar el archivo `AppServiceProvider.php` que se encuentra en la carpeta `app/Providers`. Dentro del método `boot`, agregamos la siguiente línea de código:

```php
use Illuminate\Support\Facades\Schema;  

public function boot()
{
    Schema::defaultStringLength(191);
}
```

De esta manera, Laravel aplicará un tamaño de 191 caracteres a todas las columnas `string` por defecto, evitando así el error de longitud de clave en futuras migraciones.

Una vez modificado este archivo, podemos volver a ejecutar la migración y debería completarse sin problemas.

!!! note "Note"

    Recuerda que todos los campos creados como 'string' sin tamaño definido, tendrán un tamaño de 191 caracteres por defecto, lo que evitará problemas de longitud de clave en la base de datos. Si tenemos validaciones de longitud en los campos, debemos ajustarlas para que no excedan este límite.
