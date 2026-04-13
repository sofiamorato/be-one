## Activity 1

**Q1. What happens if you send a POST to `/products` with `price: -5`? Why?**
Si se envía un POST a `/products` con `price: -5`, la respuesta es un **400 Bad Request** porque el `ValidationPipe` valida el body usando el DTO antes de que llegue al service. Como `price` tiene reglas como `@IsPositive()`, un valor negativo no pasa la validación.

**Q2. What is the role of `ParseIntPipe` in `@Param('id', ParseIntPipe)`?**
Sirve para convertir el `id` que llega como string en la URL a un número y además validar que sea un entero válido. Si no se puede convertir, NestJS responde con un error 400.

**Q3. What would happen without `@IsNotEmpty()` on `name`?**
Se podría enviar `name: ""` y pasaría la validación porque sigue siendo un string. El problema es que sería un valor vacío, por eso se usa `@IsNotEmpty()`.

**Q4. Why does the service throw `NotFoundException` instead of returning `null`?**
Porque así NestJS responde directamente con **404 Not Found**. Si devolviera `null`, habría que manejar ese caso manualmente en el controller u otros métodos.

**Q5. What is the difference between `@Get()` and `@Get(':id')`?**
`@Get()` maneja la ruta general como `/products` y se usa para listar todo. `@Get(':id')` maneja rutas con parámetro como `/products/1` y se usa para obtener un elemento específico.


## Validation Questions

**Q6. Dead route diagnosis**
Si `findAll()` no tiene decorador, `GET /tasks` devuelve **404 Not Found**. Esto pasa porque NestJS solo registra métodos como rutas si tienen decoradores como `@Get()`. Para arreglarlo hay que agregar `@Get()` encima del método.


**Q7. When `transform: true` is not enough**
Aunque `transform: true` convierte tipos automáticamente, no es lo mismo que `ParseIntPipe`. `transform` es global dentro del `ValidationPipe`, mientras que `ParseIntPipe` se aplica directamente al parámetro y valida explícitamente que sea un entero. Es más específico.


**Q8. Silent strip vs hard rejection**
Si solo usas `whitelist: true`, la petición con un campo extra como `password` devuelve **201 Created** y ese campo se elimina antes de llegar al service. Esto puede ser un problema porque el cliente cree que el dato fue procesado cuando no fue así. Por eso `forbidNonWhitelisted: true` es más seguro, porque devuelve 400.


**Q9. Mutation side-effect**
Sí, si alguien modifica lo que devuelve `findAll()`, también cambia los datos del service. Esto pasa porque se devuelve la misma referencia del arreglo. Para evitarlo, se deberían devolver copias de los objetos.


**Q10. The optional field trap**
Si envías `{"price": -50}`, la validación falla porque el campo está presente y debe cumplir las reglas. Si envías `{}`, pasa porque `@IsOptional()` permite que el campo no venga. No significa que acepte cualquier valor.


**Q11. ID reuse after deletion**
Si borras una tarea y creas otra, la nueva usa `nextId`, no reutiliza IDs. Esto evita duplicados. Si se usara `length + 1`, se podrían repetir IDs después de borrar elementos.


**Q12. Module forgotten**
Si no agregas `UsersModule` en `AppModule`, el servidor arranca pero `POST /users` devuelve **404 Not Found** porque la ruta no fue registrada. Es un error en runtime.


**Q13. Missing 201**
Un `@Post()` devuelve **201 Created** por defecto en NestJS. No es obligatorio usar `@HttpCode(201)` a menos que quieras cambiar o dejar explícito el código.


**Q14. Service throws, not returns null**
Si el service devolviera `null`, el controller tendría que validar eso manualmente. Es mejor lanzar `NotFoundException` desde el service porque evita repetir lógica y mantiene el comportamiento consistente en toda la aplicación.

