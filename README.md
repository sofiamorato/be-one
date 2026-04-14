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

**Q1. Dead route diagnosis**
Si `findAll()` no tiene decorador, `GET /tasks` devuelve **404 Not Found**. Esto pasa porque NestJS solo registra métodos como rutas si tienen decoradores como `@Get()`. NestJS no lanza ningún error al iniciar, simplemente ignora el método. Para arreglarlo hay que agregar `@Get()` encima del método.

**Q2. When `transform: true` is not enough**
Aunque `transform: true` convierte tipos automáticamente, no es lo mismo que `ParseIntPipe`. `transform` es global dentro del `ValidationPipe`, mientras que `ParseIntPipe` se aplica directamente al parámetro y valida explícitamente que sea un entero. Es más específico. Además, si el valor no es un número válido como `"abc"`, `ParseIntPipe` lanza un 400 inmediatamente, mientras que `transform: true` puede fallar silenciosamente.

**Q3. Silent strip vs hard rejection**
Si solo usas `whitelist: true`, la petición con un campo extra como `password` devuelve **201 Created** y ese campo es eliminado antes de llegar al service, por lo que llega como `undefined`. Esto puede ser un problema porque el cliente cree que el dato fue procesado cuando no fue así. Por eso `forbidNonWhitelisted: true` es más seguro, porque devuelve 400.

**Q4. Mutation side-effect**
Sí, si alguien modifica lo que devuelve `findAll()`, también cambia los datos del service. Esto pasa porque se devuelven las mismas referencias a los objetos dentro del arreglo, no copias. Para evitarlo, se deberían devolver copias de cada objeto, por ejemplo con `return this.products.map(p => ({ ...p }))`.

**Q5. The optional field trap**
Si envías `{"price": -50}`, la validación falla porque el campo está presente y debe cumplir las reglas. Si envías `{}`, pasa porque `@IsOptional()` permite que el campo no venga. La regla exacta es que `@IsOptional()` le dice a class-validator que omita todas las demás validaciones del campo si el valor es `null` o `undefined`. Si el campo viene con cualquier otro valor, las demás validaciones sí se aplican. No significa que acepte cualquier valor.

**Q6. ID reuse after deletion**
Si borras una tarea y creas otra, la nueva usa `nextId`, no reutiliza IDs. Con `nextId`: borras #1, creas nueva → id es 4. `findOne(1)` devuelve 404, nunca el elemento equivocado. Si se usara `length + 1`: tienes [1,2,3], borras #1 → quedan 2 elementos, creas nueva → id sería `2 + 1 = 3`, que ya existe. Habría dos tareas con id 3, lo que rompería `findOne`.

**Q7. Module forgotten**
Si no agregas `UsersModule` en `AppModule` el servidor arranca normalmente, no crashea y `POST /users` devuelve **404 Not Found** porque las rutas nunca fueron registradas. Es un error de **runtime**, no de compilación ni de startup, porque TypeScript no puede detectar imports faltantes en el módulo.

**Q8. Missing 201**
Por defecto `@Post()` devuelve **200 OK** en NestJS, no 201. Para retornar 201 hay que agregar `@HttpCode(HttpStatus.CREATED)` explícitamente. Un cliente estricto que espere 201 para confirmar la creación del recurso podría fallar sin este decorador.

**Q9. Service throws, not returns null**
Si el service devolviera `null`, el controller tendría que validar eso manualmente. Por ejemplo:


// Service retornando null:
findOne(id: number): Product | null {
  return this.products.find(p => p.id === id) ?? null;
}

// Controller tendría que hacer:
findOne(@Param('id', ParseIntPipe) id: number) {
  const product = this.productsService.findOne(id);
  if (!product) throw new NotFoundException(`Product #${id} not found`);
  return product;
}

Es mejor lanzar `NotFoundException` desde el service porque evita repetir lógica y mantiene el comportamiento consistente en toda la aplicación, especialmente cuando `findOne` es llamado desde múltiples lugares como `update` y `remove`.
