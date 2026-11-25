# Práctica: Validaciones y Mensajes en el CRUD de Productos

!!! quote "Autoría"

    Contenido original de **Javier Beteta** en el curso [*Introducción a Laravel*](https://jbeteta-ies.github.io/2025_Laravel_32/) ([ver esta página en el original](https://jbeteta-ies.github.io/2025_Laravel_32/content/T07/actividad07/)). Adaptado para 2º DAW del IES Juan de Garay por Eva María Gómez Abad.

## Objetivo de la actividad

![Actividad en el aula virtual](../assets/beteta/comun/tarea.png)

El objetivo de esta práctica es **mejorar el CRUD de productos** incorporando validaciones, gestión de errores y mensajes de retroalimentación para el usuario.

El alumnado consolidará los conocimientos de validación de datos en Laravel, aprendiendo a:

- Usar **Form Requests** personalizados para centralizar las reglas de validación.
- Mostrar mensajes de error junto a los campos del formulario.
- Mantener los valores anteriores con `old()`.
- Mostrar **mensajes flash** (de éxito o error) tras las operaciones del CRUD.
- Reutilizar código mediante **partials** y layouts.

Al finalizar, la aplicación ofrecerá una experiencia de usuario más robusta y profesional.

---

## Instrucciones paso a paso

Sigue las indicaciones en orden. Trabaja sobre tu proyecto del CRUD de productos de la práctica anterior.

---

### 1. Crear el FormRequest personalizado

1. Genera una nueva clase de validación `StoreProductRequest`

2. Añade las siguientes reglas de validación:

    - 'name': obligatorio, cadena de texto, entre 3 y 255 caracteres.
    - 'description': obligatorio, cadena de texto, mínimo 10 caracteres.
    - 'price': obligatorio, numérico, mínimo 0.01.
    - 'stock': obligatorio, entero, mínimo 0.

3. (Opcional) Si tu formulario contiene checkboxes u otros campos que necesiten ser adaptados antes de validar, puedes usar el método `prepareForValidation()` para normalizarlos.

---

### 2. Aplicar el FormRequest en el controlador

Realiza los cambios necesarios en el controlador `ProductController` para usar el `StoreProductRequest`, para validar los datos en los métodos `store()` y `update()`.

Añade también mensajes `flash` para las operaciones que redirigen al listado de productos, para que el usuario reciba retroalimentación.

Mensajes:

```text
- Producto creado correctamente.
- Producto actualizado correctamente.
- Producto eliminado correctamente.
```

---

### 3. Mostrar errores en los formularios

1. En los formularios de creación y edición de productos muestra una lista con los errores de validación al inicio del formulario. Si no hay errores no mestrestes nada.

2. Los errores estarán en un `div` con la clase `alert alert-danger`. Puedes utilizar CSS para darle estilo.

3. Debajo de cada campo del formulario, muestra el error específico usando la directiva `@error`. Estos errores pueden mostrase en una etiqueta `<small>` o algún contenedor `<div>` como prefieras. Tendrán el estilo `alert alert-danger` o un estilo personalizado. Podemos usar CSS para darle estilo.

4. Recuerda usar `old()` para mantener los valores introducidos por el usuario en caso de error de validación.

---

### 4. Crear el partial para los mensajes del sistema

1. Crea un `partial` llamado `messages.blade.php` para mostrar los mensajes flash.

    - Si el mensaje es de éxito, usa la clase `flash alert alert-success`.
    - Si el mensaje es de error, usa la clase `flash alert alert-danger`.

2. Añade el `partial` en el layout principal para que se muestre en todas las vistas. De esta forma, cualquier mensaje flash aparecerá automáticamente en todas las vistas.

---

### 5. Comprobar las rutas del CRUD

Ejecuta en consola:

```bash
php artisan route:list --path=product
```

Comprueba que se listan todas las rutas REST del recurso `product`, incluyendo `index`, `create`, `store`, `edit`, `update`, `show` y `destroy`.

---

### 6 Probar las validaciones y los mensajes

1. Intenta crear un producto con menos de 3 caracteres en el nombre o sin descripción. → Debes ver mensajes de error en el formulario.

2. Crea un producto válido. → Debe redirigir al listado con el mensaje **“Producto creado correctamente.”**

3. Edita un producto y cambia los valores. → Debe mostrar **“Producto actualizado correctamente.”**

4. Elimina un producto. → Debe mostrar **“Producto eliminado correctamente.”**

---

## Entrega

Entregar un documento pdf con la siguiente información:

- Código del FormRequest `StoreProductRequest`, controlador modificado y vistas actualizadas.
- Capturas de pantalla de:

    - Errores de validación en el formulario.
    - Creación correcta con mensaje de éxito.
    - Edición correcta con mensaje de éxito.
    - Eliminación con mensaje de confirmación.
- Entregar un pdf con el código y las capturas en la plataforma del aula virtual.

---

## Rúbrica de evaluación

| Criterio | Descripción | Peso |
|---|---|---|
| **FormRequest y reglas** | Se ha creado y configurado correctamente la clase `StoreProductRequest` con todas las reglas solicitadas. | 25 % |
| **Controlador actualizado** | Los métodos `store`, `update` y `destroy` utilizan `$request->validated()` y devuelven mensajes flash. | 25 % |
| **Gestión de errores en formularios** | Se muestran correctamente los errores generales y por campo; los valores se mantienen con `old()`. | 25 % |
| **Mensajes flash y partial** | Se han implementado correctamente los mensajes de éxito y error mediante `session()` e inclusión del partial. | 15 % |
| **Verificación y prueba final** | El CRUD funciona completamente y las rutas están correctamente registradas. | 10 % |

**Total: 100 %**
