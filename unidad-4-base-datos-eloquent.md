# Unidad 4 — Base de datos en Laravel y Eloquent ORM

Hasta la Unidad 3 los datos vivían en arreglos dentro del controlador o en la sesión, y se perdían en cuanto terminaba la sesión. En esta unidad se completa la **M** de MVC: los datos se guardan de forma permanente en una **base de datos relacional** y se manipulan desde PHP con **Eloquent**, el ORM de Laravel. Se verá cómo conectar Laravel a la base de datos, cómo definir las tablas con código (migraciones), cómo consultar y relacionar modelos, cómo generar datos de prueba y cómo garantizar que una operación de varios pasos se complete entera o no se aplique.

---

## Índice

1. [Configuración de base de datos](#41-configuración-de-base-de-datos)
   - 4.1.1. [Conexión en .env](#411-conexión-en-env)
   - 4.1.2. [Creación de tablas con migraciones](#412-creación-de-tablas-con-migraciones)
2. [Modelos y consultas con Eloquent](#42-modelos-y-consultas-con-eloquent)
   - 4.2.1. [Creación de modelos](#421-creación-de-modelos)
   - 4.2.2. [Consultas con Eloquent (select, where, join)](#422-consultas-con-eloquent-select-where-join)
   - 4.2.3. [Relaciones entre modelos (uno a uno, uno a muchos, muchos a muchos)](#423-relaciones-entre-modelos-uno-a-uno-uno-a-muchos-muchos-a-muchos)
3. [Seeders y factories](#43-seeders-y-factories)
   - 4.3.1. [Generación de datos de prueba](#431-generación-de-datos-de-prueba)
4. [Transacciones y manejo de errores](#44-transacciones-y-manejo-de-errores)
   - 4.4.1. [Transacciones y control de errores con try-catch](#441-transacciones-y-control-de-errores-con-try-catch)

---

## 4.1. Configuración de base de datos

### 4.1.1. Conexión en .env

**Qué es:** la **conexión** es la información que Laravel necesita para comunicarse con el servidor de base de datos: qué motor es (MySQL, PostgreSQL, SQLite…), dónde está (host y puerto), qué base de datos usar y con qué usuario y contraseña. Esa información se escribe en el archivo **`.env`** (Unidad 2).

**Por qué en `.env`:** en tu laptop la base de datos es `tienda` con el usuario `root` sin contraseña; en el servidor de producción es otra base, otro usuario y una contraseña segura. El código debe ser **idéntico** en ambos lugares; solo cambia `.env`, que además no se sube a Git para no exponer contraseñas.

**Configuración para MySQL / MariaDB** (por ejemplo, con Laragon o XAMPP):

```ini
DB_CONNECTION=mysql        # motor: mysql, mariadb, pgsql, sqlite, sqlsrv
DB_HOST=127.0.0.1          # dirección del servidor de BD (esta misma máquina)
DB_PORT=3306               # puerto por defecto de MySQL
DB_DATABASE=tienda         # nombre de la base de datos (debe EXISTIR antes de migrar)
DB_USERNAME=root           # usuario
DB_PASSWORD=               # contraseña (vacía por defecto en XAMPP/Laragon)
```

**Configuración para SQLite** (la opción por defecto en Laravel 11+):

```ini
DB_CONNECTION=sqlite
# DB_DATABASE no es necesario: usa database/database.sqlite
```

**Cómo funciona por dentro — de `.env` a la conexión:**

```
.env                              config/database.php                         PDO (PHP)
DB_CONNECTION=mysql   ──▶ 'default' => env('DB_CONNECTION', 'sqlite')
DB_HOST=127.0.0.1     ──▶ 'connections' => [                               new PDO(
DB_DATABASE=tienda    ──▶     'mysql' => [                                   'mysql:host=127.0.0.1;
DB_USERNAME=root      ──▶         'host'     => env('DB_HOST', ...),  ──▶     port=3306;dbname=tienda',
                                  'database' => env('DB_DATABASE', ...),     'root', '')
                                  'username' => env('DB_USERNAME', ...),
                              ...
```

Laravel **no** abre la conexión al arrancar: la abre de forma **perezosa** (*lazy*), la primera vez que se ejecuta una consulta en la petición, y la reutiliza para todas las demás consultas de esa misma petición. Por eso un error de conexión no aparece al cargar una página que no consulta la base de datos.

**Comparación de motores:**

| Característica | SQLite | MySQL / MariaDB | PostgreSQL |
|---|---|---|---|
| Tipo | Un **archivo** en el proyecto | Servidor independiente | Servidor independiente |
| Instalación | Ninguna (viene con PHP) | Requiere servidor (XAMPP, Laragon) | Requiere servidor |
| Usuarios y permisos | No | Sí | Sí |
| Concurrencia de escrituras | Limitada (bloquea el archivo) | Alta | Alta |
| Uso típico | Aprender, prototipos, pruebas automatizadas | La mayoría de hostings PHP; el más común con Laravel | Proyectos con consultas complejas |

**Verificar la conexión:**

```bash
php artisan db:show        # muestra motor, versión, base de datos y tablas
php artisan migrate        # si conecta, crea las tablas (4.1.2)
```

**Errores y malentendidos típicos:**

| Mensaje | Causa | Solución |
|---|---|---|
| `SQLSTATE[HY000] [1049] Unknown database 'tienda'` | La base de datos no existe | Crearla primero (`CREATE DATABASE tienda;` en phpMyAdmin / HeidiSQL). Laravel crea **tablas**, no la base |
| `SQLSTATE[HY000] [2002] No connection could be made…` | El servidor MySQL no está encendido, o host/puerto incorrectos | Iniciar MySQL en XAMPP/Laragon; revisar `DB_HOST` y `DB_PORT` |
| `SQLSTATE[HY000] [1045] Access denied for user` | Usuario o contraseña incorrectos | Revisar `DB_USERNAME` y `DB_PASSWORD` |
| `could not find driver` | Falta la extensión de PHP (`pdo_mysql`) | Activarla en `php.ini` (`extension=pdo_mysql`) |
| Cambié `.env` y sigue usando los datos viejos | La configuración está en caché | `php artisan config:clear` |

> **Ojo con `DB_HOST=localhost` vs `127.0.0.1`:** en algunos sistemas `localhost` hace que el cliente de MySQL intente conectarse por un *socket* local en lugar de por TCP, y falla aunque el servidor esté funcionando. `127.0.0.1` fuerza la conexión TCP y evita esa ambigüedad.

### 4.1.2. Creación de tablas con migraciones

**Qué es:** una **migración** es un archivo de PHP que describe un **cambio en la estructura** de la base de datos (crear una tabla, agregar una columna, crear un índice) y cómo **deshacerlo**. Las migraciones funcionan como un **control de versiones de la base de datos**.

**Por qué existen:** sin migraciones, cada integrante del equipo crea las tablas a mano en phpMyAdmin. Cuando alguien agrega una columna, tiene que avisar a los demás para que la agreguen también, y el servidor de producción termina con una estructura distinta a la de desarrollo. Con migraciones, la estructura está **en el código**: quien clona el proyecto ejecuta `php artisan migrate` y obtiene exactamente las mismas tablas, en el mismo orden.

**Crear una migración:**

```bash
php artisan make:migration create_categorias_table
php artisan make:migration create_productos_table
# Genera: database/migrations/2026_10_17_100000_create_productos_table.php
#                              └── fecha y hora ───┘ → define el ORDEN de ejecución
```

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    // up(): aplica el cambio
    public function up(): void
    {
        Schema::create('productos', function (Blueprint $table) {
            $table->id();                                   // BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY "id"
            $table->string('nombre', 100);                  // VARCHAR(100) NOT NULL
            $table->text('descripcion')->nullable();        // TEXT, admite NULL
            $table->decimal('precio', 8, 2);                // DECIMAL(8,2): hasta 999999.99
            $table->unsignedInteger('stock')->default(0);   // INT UNSIGNED, valor 0 si no se indica
            $table->boolean('activo')->default(true);       // TINYINT(1) en MySQL
            $table->foreignId('categoria_id')               // BIGINT UNSIGNED "categoria_id"
                  ->nullable()
                  ->constrained()                           // FOREIGN KEY → categorias(id) (lo deduce del nombre)
                  ->nullOnDelete();                         // si se borra la categoría, queda en NULL
            $table->timestamps();                           // created_at y updated_at (TIMESTAMP NULL)
        });
    }

    // down(): deshace EXACTAMENTE lo que hizo up()
    public function down(): void
    {
        Schema::dropIfExists('productos');
    }
};
```

**Tipos de columna más usados:**

| Método | Tipo en MySQL | Uso |
|---|---|---|
| `id()` | `BIGINT UNSIGNED AUTO_INCREMENT` + llave primaria | Identificador |
| `string('x', n)` | `VARCHAR(n)` (255 por defecto) | Textos cortos: nombre, correo |
| `text('x')` | `TEXT` | Textos largos: descripción |
| `integer`, `unsignedInteger`, `bigInteger` | `INT`, `INT UNSIGNED`, `BIGINT` | Enteros |
| `decimal('x', total, decimales)` | `DECIMAL` | **Dinero** (exacto) |
| `float`, `double` | `FLOAT`, `DOUBLE` | Mediciones científicas (aproximado) |
| `boolean('x')` | `TINYINT(1)` | Verdadero/falso |
| `date`, `dateTime`, `timestamp` | `DATE`, `DATETIME`, `TIMESTAMP` | Fechas |
| `foreignId('x_id')->constrained()` | `BIGINT UNSIGNED` + llave foránea | Relaciones (4.2.3) |
| `timestamps()` | `created_at`, `updated_at` | Eloquent las llena automáticamente |
| `softDeletes()` | `deleted_at` | Borrado lógico ("papelera") |

> **Ojo con `float` para precios:** los tipos de punto flotante guardan aproximaciones binarias; `0.1 + 0.2` no es exactamente `0.3`. Para dinero se usa `decimal`, que guarda los dígitos exactos. En `decimal(8, 2)`, el 8 es el total de dígitos y el 2 cuántos van después del punto: el máximo es 999 999.99 (6 enteros + 2 decimales).

**Modificadores de columna:** `->nullable()`, `->default(valor)`, `->unique()`, `->index()`, `->comment('...')`, `->after('columna')` (solo MySQL).

**Cómo funciona por dentro — la tabla `migrations` y los lotes:** Laravel crea una tabla `migrations` donde registra cada migración ejecutada y en qué **lote** (*batch*). Así sabe cuáles faltan y cuáles deshacer.

**Traza — ejecución y reversión:**

| Comando | Qué hace | Tabla `migrations` después |
|---|---|---|
| (inicio) | — | vacía |
| `php artisan migrate` | Ejecuta `up()` de las 2 migraciones pendientes, en orden de fecha | `create_categorias_table` (lote 1), `create_productos_table` (lote 1) |
| Se crea `add_marca_to_productos_table` y se ejecuta `migrate` | Solo ejecuta la nueva | … + `add_marca_to_productos_table` (lote **2**) |
| `php artisan migrate:rollback` | Ejecuta `down()` del **último lote** (2) | Quedan solo las del lote 1 |
| `php artisan migrate:rollback` | Ejecuta `down()` del lote 1, en **orden inverso** (primero productos, luego categorías) | vacía |

> **Por qué el orden inverso:** `productos` tiene una llave foránea hacia `categorias`. Para crear `productos`, `categorias` debe existir antes; para borrarlas, primero hay que borrar `productos`, o MySQL rechaza borrar `categorias` porque otra tabla depende de ella. Por eso la fecha del nombre del archivo importa: la migración de la tabla "padre" debe tener una fecha anterior.

**Comandos de migración:**

| Comando | Efecto | ¿Destruye datos? |
|---|---|---|
| `migrate` | Ejecuta las pendientes | No |
| `migrate:status` | Muestra cuáles se ejecutaron y en qué lote | No |
| `migrate:rollback` | Deshace el último lote (`--step=1` deshace solo una migración) | Sí, de lo que deshace |
| `migrate:reset` | Deshace **todas** | Sí |
| `migrate:refresh` | `reset` + `migrate` (usa los `down()`) | Sí |
| `migrate:fresh` | **Borra todas las tablas** sin usar `down()` y vuelve a migrar | Sí, todo |
| `migrate:fresh --seed` | Lo anterior + ejecuta los *seeders* (4.3) | Sí, todo |

**Modificar una tabla existente:** se crea una migración **nueva** (nunca se edita la que ya se ejecutó):

```bash
php artisan make:migration add_marca_to_productos_table --table=productos
```

```php
public function up(): void
{
    Schema::table('productos', function (Blueprint $table) {   // table(), no create()
        $table->string('marca', 50)->nullable()->after('nombre');
    });
}

public function down(): void
{
    Schema::table('productos', function (Blueprint $table) {
        $table->dropColumn('marca');
    });
}
```

**Errores y malentendidos típicos:**

- **Editar una migración ya ejecutada y volver a correr `migrate`:** no pasa nada, porque Laravel ve que ya está registrada en la tabla `migrations` y la omite. En desarrollo, a solas, se puede usar `migrate:fresh`; en un proyecto compartido o en producción, siempre se crea una migración nueva.
- **Agregar una columna `NOT NULL` sin valor por defecto a una tabla con datos:** MySQL no sabe qué poner en las filas existentes. Se usa `->nullable()` o `->default(...)`.
- **"Foreign key constraint is incorrectly formed":** la tabla referenciada aún no existe (orden de las migraciones) o los tipos no coinciden (por ejemplo, `integer` contra el `BIGINT UNSIGNED` de `id()`). `foreignId()` crea el tipo correcto.
- **`down()` vacío o incompleto:** `rollback` "funciona" pero deja la base en un estado inconsistente. `down()` debe revertir exactamente lo que hizo `up()`.
- **Ejecutar `migrate:fresh` en producción:** borra **todos** los datos reales. Laravel pide confirmación en producción, pero no la saltes.

---

## 4.2. Modelos y consultas con Eloquent

**Qué es un ORM:** un **ORM** (*Object-Relational Mapping*, mapeo objeto-relacional) es una capa que traduce entre el mundo de la **base de datos** (tablas, filas, columnas) y el mundo de la **programación orientada a objetos** (clases, objetos, atributos). **Eloquent** es el ORM de Laravel.

```
        Base de datos                              PHP (Eloquent)
 tabla productos                         clase Producto
 ┌────┬─────────┬────────┐               ┌──────────────────────────┐
 │ id │ nombre  │ precio │   fila ──▶    │ $p->id      = 1          │
 ├────┼─────────┼────────┤   objeto      │ $p->nombre  = "Teclado"  │
 │ 1  │ Teclado │ 350.00 │               │ $p->precio  = "350.00"   │
 └────┴─────────┴────────┘               └──────────────────────────┘
 columna ◀──────────────────────────────▶ atributo
```

**Por qué existe:** sin un ORM se escribe SQL como texto dentro de PHP, se arman los valores a mano (con riesgo de **inyección SQL**) y se convierten los resultados en arreglos. Eloquent permite trabajar con objetos (`$producto->precio = 400; $producto->save();`) y genera el SQL correcto, con los valores siempre enviados de forma segura.

**Patrón Active Record:** Eloquent implementa el patrón **Active Record**: cada objeto representa **una fila** y sabe guardarse, actualizarse y borrarse a sí mismo. La clase (métodos estáticos como `Producto::where(...)`) representa **la tabla**.

### 4.2.1. Creación de modelos

**Crear un modelo:**

```bash
php artisan make:model Producto                # solo el modelo: app/Models/Producto.php
php artisan make:model Producto -mfs           # + migración, factory y seeder
php artisan make:model Producto --all          # + controlador de recurso, form requests, policy…
```

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Producto extends Model
{
    use HasFactory;                               // permite usar Producto::factory() (4.3)

    // Columnas que se pueden asignar en bloque con create() / update() / fill()
    protected $fillable = ['nombre', 'descripcion', 'precio', 'stock', 'activo', 'categoria_id'];

    // Conversión automática de tipos al leer/escribir
    protected function casts(): array
    {
        return [
            'precio' => 'decimal:2',
            'activo' => 'boolean',                // TINYINT 1/0 → true/false en PHP
        ];
    }
}
```

**Cómo funciona por dentro — convenciones que Eloquent asume:**

| Aspecto | Convención | Para `Producto` | Cómo cambiarlo |
|---|---|---|---|
| Nombre de la tabla | Plural en snake_case del nombre de la clase | `productos` | `protected $table = 'mi_tabla';` |
| Llave primaria | Columna `id`, entera, autoincremental | `id` | `$primaryKey`, `$incrementing`, `$keyType` |
| Marcas de tiempo | Existen `created_at` y `updated_at` | Se llenan solas | `public $timestamps = false;` |
| Conexión | La conexión por defecto de `.env` | `mysql` | `protected $connection = '...';` |

> **Ojo: el pluralizador de Laravel sigue reglas del inglés.** `Producto` → `productos` y `Categoria` → `categorias` funcionan por casualidad (basta con agregar "s"), pero `Profesor` → `profesors`, `Perfil` → `perfils` y `Rol` → `rols`. Si tu tabla se llama `profesores`, debes indicarlo con `protected $table = 'profesores';`, o Eloquent lanzará *"Table 'tienda.profesors' doesn't exist"*. Puedes comprobar qué tabla deduce con `php artisan tinker` → `(new App\Models\Profesor)->getTable()`.

**Asignación masiva (*mass assignment*) — por qué existe `$fillable`:**

```php
// En el controlador, con datos que llegan del formulario
Producto::create($request->all());
```

Si el formulario tiene `nombre` y `precio`, pero un usuario malicioso agrega a mano un campo `es_destacado=1` o `user_id=1`, `create($request->all())` lo guardaría. `$fillable` es una **lista blanca**: solo esas columnas se asignan en bloque.

| Configuración | Comportamiento |
|---|---|
| Sin `$fillable` ni `$guarded` (por defecto `$guarded = ['*']`) | `create([...])` lanza `MassAssignmentException`: nada es asignable en bloque |
| `$fillable = ['nombre', 'precio']` | Se asignan esas columnas; las demás se **descartan en silencio** |
| `$guarded = ['id']` | Lista negra: todo es asignable excepto `id` (más cómodo, menos seguro) |
| `$guarded = []` | Todo es asignable: solo si **siempre** pasas datos ya validados (`$request->validated()`) |

**Operaciones básicas (CRUD) con un modelo:**

```php
// CREATE — dos formas
$p = new Producto();
$p->nombre = 'Teclado';
$p->precio = 350;
$p->save();                                       // INSERT; ahora $p->id tiene el valor generado (p. ej. 1)

$p = Producto::create(['nombre' => 'Mouse', 'precio' => 180]);   // INSERT con asignación masiva

// READ
$p = Producto::find(1);                           // objeto Producto, o null si no existe
$p = Producto::findOrFail(1);                     // objeto, o error 404 si no existe
$todos = Producto::all();                         // Collection con todos los registros

// UPDATE
$p->precio = 400;
$p->save();                                       // UPDATE solo de las columnas que cambiaron
$p->update(['stock' => 15]);                      // asignación masiva + save()

// DELETE
$p->delete();                                     // DELETE de esa fila
Producto::destroy([2, 3]);                        // borra por id
```

**Traza — el estado del objeto con `save()`:**

| Paso | Código | `$p->exists` | `$p->isDirty()` | SQL ejecutado |
|---|---|---|---|---|
| 1 | `$p = new Producto()` | `false` | `false` | — |
| 2 | `$p->nombre = 'Teclado'; $p->precio = 350;` | `false` | `true` | — |
| 3 | `$p->save()` | `true` | `false` | `insert into productos (nombre, precio, updated_at, created_at) values (?, ?, ?, ?)` |
| 4 | `$p->precio = 400` | `true` | `true` (`precio`) | — |
| 5 | `$p->save()` | `true` | `false` | `update productos set precio = ?, updated_at = ? where id = ?` |
| 6 | `$p->save()` (sin cambios) | `true` | `false` | **Ninguno**: no hay nada que actualizar |

`save()` decide entre `INSERT` y `UPDATE` según `exists` (si el objeto ya vino de la base de datos o ya se guardó), y en el `UPDATE` solo incluye las columnas "sucias" (*dirty*).

**Errores y malentendidos típicos:**

- **`Add [nombre] to fillable property to allow mass assignment`:** se usó `create()`/`update()` sin declarar `$fillable`.
- **"Mis campos no se guardan, pero no hay error":** están fuera de `$fillable` y se descartaron en silencio.
- **Crear el modelo con nombre en plural** (`Productos`): Eloquent buscaría la tabla `productoss`.
- **Olvidar `use HasFactory`:** `Producto::factory()` lanza *"Call to undefined method"*.

### 4.2.2. Consultas con Eloquent (select, where, join)

**Qué es:** cada modelo Eloquent funciona también como un **constructor de consultas** (*query builder*): se encadenan métodos que describen la consulta (`where`, `orderBy`, `select`…) y al final un método que la **ejecuta** (`get`, `first`, `count`…).

**Cómo funciona por dentro — construir no es ejecutar:** los métodos como `where` u `orderBy` **no consultan** la base de datos; solo agregan partes a un objeto `Builder`. La consulta SQL se ejecuta **una sola vez**, al llamar a un método final:

```php
$consulta = Producto::where('precio', '>', 100);    // Builder: aún NO se ejecutó nada
$consulta->orderBy('nombre');                       // sigue sin ejecutarse
$consulta->toSql();                                 // "select * from `productos` where `precio` > ? order by `nombre` asc"
$consulta->getBindings();                           // [100]
$productos = $consulta->get();                      // AHORA se ejecuta, devuelve una Collection
```

> **Por qué aparece `?` en lugar del valor:** Eloquent usa **consultas preparadas** (*prepared statements*). El SQL y los valores viajan **por separado** al servidor de base de datos, que nunca interpreta los valores como código SQL. Por eso, aunque un usuario escriba `' OR 1=1 --` en un buscador, se busca literalmente ese texto: es la defensa contra la **inyección SQL**.

**Datos de ejemplo para todas las trazas de esta sección:**

Tabla `categorias`:

| id | nombre |
|---|---|
| 1 | Periféricos |
| 2 | Pantallas |

Tabla `productos`:

| id | nombre | precio | stock | categoria_id |
|---|---|---|---|---|
| 1 | Teclado | 350.00 | 10 | 1 |
| 2 | Mouse | 180.00 | 0 | 1 |
| 3 | Monitor | 3200.00 | 4 | 2 |
| 4 | Cable HDMI | 90.00 | 25 | NULL |

**Filtros con `where`:**

```php
Producto::where('precio', '>', 100)->get();            // Teclado, Mouse, Monitor
Producto::where('stock', 0)->get();                    // Mouse        ← sin operador significa "="
Producto::where('precio', '>', 100)
        ->where('stock', '>', 0)->get();               // Teclado, Monitor   ← encadenar = AND
Producto::where('stock', 0)
        ->orWhere('precio', '<', 100)->get();          // Mouse, Cable HDMI  ← OR
Producto::whereIn('id', [1, 3])->get();                // Teclado, Monitor
Producto::whereBetween('precio', [100, 400])->get();   // Teclado, Mouse    ← límites INCLUIDOS
Producto::whereNull('categoria_id')->get();            // Cable HDMI
Producto::where('nombre', 'like', '%o%')->get();       // Teclado, Mouse, Monitor ← contienen "o"
```

**Traza — evaluación fila por fila de `where('precio', '>', 100)->where('stock', '>', 0)`:**

| Fila | `precio > 100` | `stock > 0` | AND | ¿En el resultado? |
|---|---|---|---|---|
| Teclado (350, 10) | Sí | Sí | Sí | **Sí** |
| Mouse (180, 0) | Sí | No | No | No |
| Monitor (3200, 4) | Sí | Sí | Sí | **Sí** |
| Cable HDMI (90, 25) | No | Sí | No | No |

> **Ojo con mezclar AND y OR:** `where('activo', true)->where('stock', 0)->orWhere('precio', '<', 100)` genera `activo = 1 AND stock = 0 OR precio < 100`, y como AND se evalúa antes que OR, eso significa `(activo AND stock = 0) OR precio < 100`: un producto **inactivo** barato también aparece. Para agrupar se pasa una función: `where('activo', true)->where(fn ($q) => $q->where('stock', 0)->orWhere('precio', '<', 100))`, que genera `activo = 1 AND (stock = 0 OR precio < 100)`.

**Seleccionar columnas, ordenar, limitar y agregar:**

```php
Producto::select('nombre', 'precio')->get();             // solo esas columnas
Producto::orderBy('precio', 'desc')->first();            // Monitor (el más caro)
Producto::orderBy('precio')->take(2)->get();             // Cable HDMI, Mouse (los 2 más baratos)
Producto::pluck('nombre');                               // ["Teclado", "Mouse", "Monitor", "Cable HDMI"]
Producto::count();                                       // 4
Producto::where('stock', '>', 0)->count();               // 3
Producto::sum('stock');                                  // 39   (10 + 0 + 4 + 25)
Producto::avg('precio');                                 // 955  ((350 + 180 + 3200 + 90) / 4 = 3820 / 4)
Producto::max('precio');                                 // 3200
Producto::paginate(10);                                  // página actual de 10 en 10, con enlaces en Blade: {{ $productos->links() }}
```

**Métodos que ejecutan la consulta — qué devuelve cada uno:**

| Método | Devuelve | Si no hay resultados |
|---|---|---|
| `get()` / `all()` | `Collection` de modelos | `Collection` **vacía** (no `null`) |
| `first()` | Un modelo | `null` |
| `firstOrFail()` / `findOrFail()` | Un modelo | Error 404 (`ModelNotFoundException`) |
| `find($id)` | Un modelo | `null` |
| `count()`, `sum()`, `avg()`, `max()` | Un número | `0` (o `null` en `avg`/`max`) |
| `pluck('col')` | `Collection` de valores sueltos | `Collection` vacía |
| `exists()` | `true` / `false` | `false` |

> **Ojo con `Collection` vacía:** `if ($productos)` es **siempre** verdadero, incluso sin resultados, porque un objeto `Collection` vacío sigue siendo un objeto. Se pregunta con `$productos->isEmpty()` o `$productos->count() === 0` (o en Blade con `@forelse`).

**Consulta en la base de datos vs en la colección:**

```php
Producto::where('stock', '>', 0)->get();          // filtra MySQL: viajan solo 3 filas
Producto::all()->where('stock', '>', 0);          // trae las 4 filas a PHP y filtra en memoria
```

Ambas dan el mismo resultado con 4 filas, pero con 100 000 filas la segunda carga todo en memoria. Regla: **filtra en la base de datos** (antes de `get()`), y usa los métodos de colección solo para transformar resultados ya pequeños.

**Joins — combinar tablas:** un **join** combina filas de dos tablas según una condición (normalmente, llave foránea = llave primaria):

```php
// INNER JOIN: solo productos que tienen categoría
$filas = Producto::join('categorias', 'productos.categoria_id', '=', 'categorias.id')
    ->select('productos.nombre', 'categorias.nombre as categoria')
    ->get();

// LEFT JOIN: todos los productos, tengan o no categoría
$filas = Producto::leftJoin('categorias', 'productos.categoria_id', '=', 'categorias.id')
    ->select('productos.nombre', 'categorias.nombre as categoria')
    ->get();
```

**Traza — resultado de cada join:**

| `productos.nombre` | `categoria_id` | INNER JOIN → `categoria` | LEFT JOIN → `categoria` |
|---|---|---|---|
| Teclado | 1 | Periféricos | Periféricos |
| Mouse | 1 | Periféricos | Periféricos |
| Monitor | 2 | Pantallas | Pantallas |
| Cable HDMI | NULL | *(no aparece)* | `null` |
| **Total de filas** | | **3** | **4** |

```
INNER JOIN: solo la intersección         LEFT JOIN: toda la tabla izquierda
  productos   categorias                  productos   categorias
   ┌─────┐     ┌─────┐                    ┌─────┐     ┌─────┐
   │   ┌─┼─────┼─┐   │                    │█████┼─────┼─┐   │
   │   │█│█████│█│   │                    │█████│█████│█│   │
   │   └─┼─────┼─┘   │                    │█████┼─────┼─┘   │
   └─────┘     └─────┘                    └─────┘     └─────┘
```

> **Ojo con columnas del mismo nombre:** ambas tablas tienen `id` y `nombre`. Sin `select` con alias (`categorias.nombre as categoria`), el valor de una columna **sobrescribe** al de la otra en el resultado, y obtendrías el nombre de la categoría donde esperabas el del producto. En consultas con join, siempre califica las columnas con el nombre de la tabla.

**Join vs relaciones de Eloquent:** en Eloquent, la forma habitual de combinar datos son las **relaciones** (4.2.3), que devuelven objetos completos y anidados (`$producto->categoria->nombre`). El `join` se usa cuando necesitas **filtrar u ordenar** por una columna de otra tabla en una sola consulta, o generar reportes con columnas de varias tablas.

**Errores y malentendidos típicos:**

- **Olvidar `get()`:** `$p = Producto::where('stock', 0);` es un `Builder`, no resultados; en un `@foreach` no muestra lo esperado.
- **Usar `find()` con algo que no es la llave primaria:** `Producto::find('Teclado')` busca `id = 'Teclado'`. Para otra columna: `Producto::where('nombre', 'Teclado')->first()`.
- **Concatenar valores en `whereRaw`:** `whereRaw("nombre = '$texto'")` reintroduce la inyección SQL. Si necesitas SQL crudo, usa marcadores: `whereRaw('nombre = ?', [$texto])`.
- **Comparar con `null` usando `where('x', null)`:** Eloquent lo convierte en `IS NULL`, pero `where('x', '!=', null)` no hace lo esperado; usa `whereNotNull('x')`.

### 4.2.3. Relaciones entre modelos (uno a uno, uno a muchos, muchos a muchos)

**Qué es:** una **relación** describe cómo se conectan los registros de dos tablas mediante **llaves foráneas**. En Eloquent, cada relación se declara como un **método** del modelo, y luego se usa como si fuera un atributo (`$categoria->productos`).

**Por qué existen:** en una base de datos relacional los datos se **normalizan**: el nombre de la categoría "Periféricos" se guarda **una sola vez** en `categorias`, y cada producto solo guarda el número `categoria_id`. Así, cambiar el nombre de la categoría es actualizar una fila, no cientos. Las relaciones de Eloquent permiten recorrer esas conexiones sin escribir los joins a mano.

**Resumen de los tres tipos:**

| Relación | Ejemplo | Dónde está la llave foránea | Método en el modelo A | Inverso en el modelo B |
|---|---|---|---|---|
| **Uno a uno** | Un usuario tiene **un** perfil | En la tabla del "hijo": `perfiles.user_id` | `hasOne(Perfil::class)` | `belongsTo(User::class)` |
| **Uno a muchos** | Una categoría tiene **muchos** productos | En la tabla del lado "muchos": `productos.categoria_id` | `hasMany(Producto::class)` | `belongsTo(Categoria::class)` |
| **Muchos a muchos** | Un estudiante cursa muchos cursos y un curso tiene muchos estudiantes | En una **tabla intermedia** (*pivote*): `curso_estudiante` | `belongsToMany(Curso::class)` | `belongsToMany(Estudiante::class)` |

> **Regla para no confundir `hasOne`/`hasMany` con `belongsTo`:** el modelo cuya tabla **tiene la columna `xxx_id`** es el que "pertenece a" (`belongsTo`). El otro lado "tiene" (`hasOne` / `hasMany`). Un producto tiene la columna `categoria_id` → `Producto belongsTo Categoria`.

**Uno a uno:**

```
users                     perfiles
┌────┬──────┐             ┌────┬─────────┬────────────┐
│ id │ name │ 1 ──── 1    │ id │ user_id │ telefono   │
├────┼──────┤             ├────┼─────────┼────────────┤
│ 7  │ Ana  │ ◀────────── │ 1  │ 7       │ 5512345678 │
└────┴──────┘             └────┴─────────┴────────────┘
```

```php
// app/Models/User.php
public function perfil()
{
    return $this->hasOne(Perfil::class);       // busca perfiles.user_id = $this->id
}

// app/Models/Perfil.php
protected $table = 'perfiles';                 // el pluralizador en inglés deduciría "perfils" (4.2.1)

public function user()
{
    return $this->belongsTo(User::class);      // usa perfiles.user_id → users.id
}

// Uso
User::find(7)->perfil->telefono;               // "5512345678"
Perfil::find(1)->user->name;                   // "Ana"
```

```php
// Migración de perfiles
$table->foreignId('user_id')->unique()->constrained()->cascadeOnDelete();
//                           └ unique: garantiza en la BD que sea UNO a uno
```

**Uno a muchos:**

```php
// app/Models/Categoria.php
public function productos()
{
    return $this->hasMany(Producto::class);    // productos.categoria_id = $this->id
}

// app/Models/Producto.php
public function categoria()
{
    return $this->belongsTo(Categoria::class); // productos.categoria_id → categorias.id
}

// Uso (con los datos de 4.2.2)
Categoria::find(1)->productos->pluck('nombre');      // ["Teclado", "Mouse"]
Categoria::find(1)->productos()->where('stock', '>', 0)->get();  // [Teclado]
Producto::find(3)->categoria->nombre;                // "Pantallas"
Producto::find(4)->categoria;                        // null  (categoria_id es NULL)
Producto::find(4)->categoria?->nombre;               // null  ← "?->" evita el error si es null

// Crear un producto ya relacionado
Categoria::find(2)->productos()->create(['nombre' => 'Proyector', 'precio' => 5400]);
// INSERT con categoria_id = 2 asignado automáticamente
```

**Cómo deduce Eloquent las columnas:**

| Relación | Llave foránea que asume | Se deduce de… | Si es distinta |
|---|---|---|---|
| `hasMany(Producto::class)` en `Categoria` | `productos.categoria_id` | Nombre del modelo **padre** (`Categoria` → `categoria_id`) | `hasMany(Producto::class, 'cat_id')` |
| `belongsTo(Categoria::class)` en `Producto` | `productos.categoria_id` | Nombre del **método** (`categoria()` → `categoria_id`) | `belongsTo(Categoria::class, 'cat_id')` |

**Propiedad vs método — una diferencia clave:**

| Expresión | Qué es | ¿Ejecuta la consulta? |
|---|---|---|
| `$categoria->productos` (sin paréntesis) | Los **resultados**: una `Collection` | Sí, la primera vez; después usa la copia ya cargada |
| `$categoria->productos()` (con paréntesis) | La **relación** como constructor de consultas | No, hasta que llames a `get()`, `count()`, `create()`… |

Por eso `$categoria->productos()->where(...)->get()` filtra en la base de datos, mientras que `$categoria->productos->where(...)` trae **todos** los productos de la categoría y filtra en PHP.

**Muchos a muchos:**

```
estudiantes               curso_estudiante (pivote)           cursos
┌────┬────────┐           ┌──────────┬───────────────┐        ┌────┬──────────┐
│ id │ nombre │           │ curso_id │ estudiante_id │        │ id │ nombre   │
├────┼────────┤           ├──────────┼───────────────┤        ├────┼──────────┤
│ 1  │ Ana    │ ◀──────── │    1     │       1       │ ─────▶ │ 1  │ HTML     │
│ 2  │ Luis   │ ◀──┐      │    2     │       1       │ ─────▶ │ 2  │ Laravel  │
└────┴────────┘    └───── │    2     │       2       │        └────┴──────────┘
                          └──────────┴───────────────┘
 Ana cursa HTML y Laravel · Luis cursa Laravel · Laravel tiene a Ana y a Luis
```

**Por qué hace falta una tabla intermedia:** una columna `curso_id` en `estudiantes` solo permitiría **un** curso por estudiante, y una columna `estudiante_id` en `cursos` solo **un** estudiante por curso. La tabla pivote guarda una fila por cada **par** (estudiante, curso).

```php
// Migración de la tabla pivote
// Nombre por convención: los dos modelos en SINGULAR, snake_case, en ORDEN ALFABÉTICO → curso_estudiante
Schema::create('curso_estudiante', function (Blueprint $table) {
    $table->foreignId('curso_id')->constrained()->cascadeOnDelete();
    $table->foreignId('estudiante_id')->constrained()->cascadeOnDelete();
    $table->decimal('calificacion', 4, 2)->nullable();   // dato propio de la relación
    $table->primary(['curso_id', 'estudiante_id']);      // evita inscribir dos veces el mismo par
});

// app/Models/Estudiante.php
public function cursos()
{
    return $this->belongsToMany(Curso::class)->withPivot('calificacion');
}

// app/Models/Curso.php
public function estudiantes()
{
    return $this->belongsToMany(Estudiante::class)->withPivot('calificacion');
}
```

**Manipular la tabla pivote — traza con `attach`, `detach` y `sync`:**

| Paso | Código (sobre Ana, id 1) | Filas de Ana en `curso_estudiante` (cursos) |
|---|---|---|
| 0 | — | `[]` |
| 1 | `$ana->cursos()->attach(1)` | `[1]` |
| 2 | `$ana->cursos()->attach([2, 3])` | `[1, 2, 3]` |
| 3 | `$ana->cursos()->detach(3)` | `[1, 2]` |
| 4 | `$ana->cursos()->sync([2, 4])` | `[2, 4]` ← quita el 1, conserva el 2, agrega el 4 |
| 5 | `$ana->cursos()->attach(2)` | **Error** de llave duplicada (si existe la llave primaria compuesta) |
| 6 | `$ana->cursos()->syncWithoutDetaching([2, 5])` | `[2, 4, 5]` ← agrega lo que falte sin quitar nada |

```php
$ana->cursos()->attach(1, ['calificacion' => 9.5]);   // con dato de la pivote
foreach ($ana->cursos as $curso) {
    echo $curso->nombre . ': ' . $curso->pivot->calificacion;   // "HTML: 9.50"
}
```

| Método | Cuándo usarlo |
|---|---|
| `attach` | Agregar uno o varios vínculos (inscribir) |
| `detach` | Quitar vínculos (`detach()` sin argumentos quita **todos**) |
| `sync` | Dejar **exactamente** la lista indicada (típico al guardar un formulario con *checkboxes*) |
| `syncWithoutDetaching` | Agregar sin borrar los existentes ni duplicar |

**El problema N+1 y la carga anticipada (*eager loading*):**

```php
// Vista que lista productos con su categoría
$productos = Producto::all();                 // 1 consulta
foreach ($productos as $p) {
    echo $p->categoria?->nombre;              // 1 consulta POR CADA producto
}
```

**Traza del número de consultas:**

| Productos en la tabla | Sin `with` (carga perezosa) | Con `with('categoria')` (carga anticipada) |
|---|---|---|
| 4 | 1 + 4 = **5** | **2** |
| 50 | 1 + 50 = **51** | **2** |
| 1000 | 1 + 1000 = **1001** | **2** |

```php
$productos = Producto::with('categoria')->get();
// 1) select * from productos
// 2) select * from categorias where categorias.id in (1, 2)
// Eloquent asocia en memoria cada producto con su categoría
```

El nombre "N+1" viene de **1** consulta para la lista principal más **N** consultas, una por cada elemento. Es el problema de rendimiento más común con cualquier ORM, porque el código "se ve" correcto y con pocos datos de prueba no se nota la lentitud.

```php
// Varias relaciones y relaciones anidadas
Estudiante::with(['cursos', 'perfil'])->get();
Categoria::with('productos')->withCount('productos')->get();   // agrega $categoria->productos_count
```

**Errores y malentendidos típicos:**

- **"Attempt to read property 'nombre' on null":** el registro no tiene relacionado (por ejemplo, un producto sin categoría). Usa `?->` o `optional()`, o garantiza la relación con `NOT NULL` en la migración.
- **Invertir `hasMany` y `belongsTo`:** declarar `belongsTo` en `Categoria` haría que Eloquent buscara una columna `producto_id` en `categorias`, que no existe.
- **Nombre de tabla pivote fuera de convención** (`estudiante_curso`, `inscripciones`): Eloquent buscaría `curso_estudiante`. Si se usa otro nombre, se indica: `belongsToMany(Curso::class, 'inscripciones')`.
- **No usar `with` en listados:** revisa con la barra de depuración (paquetes como Laravel Debugbar) o con `DB::listen` cuántas consultas genera una página.

---

## 4.3. Seeders y factories

### 4.3.1. Generación de datos de prueba

**Qué es:**

- Un **seeder** es una clase que **inserta datos** en la base de datos al ejecutar un comando: datos iniciales necesarios (roles, categorías, un usuario administrador) o datos de prueba.
- Una **factory** es una "receta" que describe cómo **generar** un registro falso pero realista de un modelo (nombre, correo, precio aleatorios). Se usa desde los seeders y desde las pruebas automatizadas.

**Por qué existen:** para desarrollar y probar una aplicación se necesitan datos: ver cómo se ve la paginación con 200 productos, probar búsquedas, comprobar el rendimiento con 10 000 registros. Insertarlos a mano en phpMyAdmin es lento y se pierde todo con cada `migrate:fresh`. Con seeders y factories, cualquier persona del equipo obtiene una base de datos poblada y **reproducible** con un solo comando.

| Aspecto | Seeder | Factory |
|---|---|---|
| Qué es | Un **programa** que inserta datos | Una **plantilla** para fabricar modelos |
| Datos | Fijos (lo que tú escribes) o generados con factories | Aleatorios y realistas |
| Se ejecuta con | `php artisan db:seed` | Desde código: `Producto::factory()->create()` |
| Ubicación | `database/seeders/` | `database/factories/` |
| Ejemplo | "Crear las categorías Periféricos y Pantallas, y 50 productos" | "Un producto tiene un nombre de 2 palabras y un precio entre 50 y 5000" |

**Crear una factory:**

```bash
php artisan make:factory ProductoFactory --model=Producto
```

```php
<?php

namespace Database\Factories;

use App\Models\Categoria;
use Illuminate\Database\Eloquent\Factories\Factory;

class ProductoFactory extends Factory
{
    public function definition(): array
    {
        return [
            'nombre'       => ucfirst(fake()->words(2, true)),           // "Mesa azul" (palabras aleatorias)
            'descripcion'  => fake()->sentence(),                        // una oración aleatoria
            'precio'       => fake()->randomFloat(2, 50, 5000),          // p. ej. 1834.27
            'stock'        => fake()->numberBetween(0, 100),             // p. ej. 42
            'activo'       => fake()->boolean(80),                       // true con 80 % de probabilidad
            'categoria_id' => Categoria::factory(),                      // crea una categoría si no se indica otra
        ];
    }

    // Estado: una variación con nombre propio
    public function agotado(): static
    {
        return $this->state(fn (array $atributos) => ['stock' => 0]);
    }
}
```

> **Idioma de los datos falsos:** `fake()` usa la librería **Faker**. Con `APP_FAKER_LOCALE=es_MX` en `.env`, nombres, direcciones y teléfonos se generan con formatos de México ("María Hernández", "Calle Juárez 123"). Algunos métodos, como `words()` o `sentence()`, generan texto *lorem ipsum* en cualquier idioma.

**Usar una factory:**

```php
Producto::factory()->create();                              // 1 producto (y 1 categoría nueva) guardados en la BD
Producto::factory()->count(10)->create();                   // 10 productos
Producto::factory()->count(3)->agotado()->create();         // 3 con stock 0
Producto::factory()->create(['nombre' => 'Teclado']);       // sobrescribe un campo
$p = Producto::factory()->make();                           // lo construye SIN guardarlo (útil en pruebas)

// Relaciones
$cat = Categoria::factory()->create();
Producto::factory()->count(5)->for($cat)->create();         // 5 productos de ESA categoría
Categoria::factory()->has(Producto::factory()->count(5))->create();   // 1 categoría con 5 productos
```

> **Ojo con `'categoria_id' => Categoria::factory()` dentro de `definition()`:** `Producto::factory()->count(10)->create()` crea **10 categorías distintas**, una por producto. Para repartir productos entre categorías ya existentes, usa `for($categoria)` o pasa el valor: `->create(['categoria_id' => $cat->id])`, o en la definición `'categoria_id' => Categoria::inRandomOrder()->value('id')`.

**`create()` vs `make()`:**

| Método | ¿Guarda en la BD? | ¿Tiene `id`? | Uso |
|---|---|---|---|
| `make()` | No | No | Probar lógica sin tocar la base de datos |
| `create()` | Sí | Sí | Poblar la base de datos |

**Crear un seeder:**

```bash
php artisan make:seeder CategoriaSeeder
```

```php
// database/seeders/CategoriaSeeder.php — datos FIJOS
class CategoriaSeeder extends Seeder
{
    public function run(): void
    {
        foreach (['Periféricos', 'Pantallas', 'Almacenamiento'] as $nombre) {
            Categoria::create(['nombre' => $nombre]);
        }
    }
}

// database/seeders/DatabaseSeeder.php — el seeder PRINCIPAL, que llama a los demás
class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call([
            CategoriaSeeder::class,     // primero las categorías (los productos dependen de ellas)
            ProductoSeeder::class,
        ]);

        User::factory()->create([
            'name'  => 'Administrador',
            'email' => 'admin@tienda.test',   // la UserFactory de Laravel pone la contraseña "password" (hasheada)
        ]);
    }
}

// database/seeders/ProductoSeeder.php — datos GENERADOS, repartidos entre las categorías existentes
class ProductoSeeder extends Seeder
{
    public function run(): void
    {
        Categoria::all()->each(function (Categoria $categoria) {
            Producto::factory()->count(5)->for($categoria)->create();
        });
    }
}
```

**Comandos:**

| Comando | Efecto |
|---|---|
| `php artisan db:seed` | Ejecuta `DatabaseSeeder` |
| `php artisan db:seed --class=CategoriaSeeder` | Ejecuta solo ese seeder |
| `php artisan migrate:fresh --seed` | Borra todo, recrea las tablas y siembra: base de datos limpia y poblada desde cero |

**Traza — `php artisan migrate:fresh --seed` con los seeders anteriores:**

| Paso | Acción | `categorias` | `productos` | `users` |
|---|---|---|---|---|
| 1 | `migrate:fresh` borra y recrea las tablas | 0 | 0 | 0 |
| 2 | `CategoriaSeeder` | 3 | 0 | 0 |
| 3 | `ProductoSeeder`: 5 por cada una de las 3 categorías | 3 | **15** | 0 |
| 4 | `User::factory()->create(...)` | 3 | 15 | 1 |

**Errores y malentendidos típicos:**

- **Ejecutar `db:seed` dos veces sin `fresh`:** los seeders **no** comprueban si los datos ya existen; se insertan duplicados (o fallan las columnas `unique`). Para datos fijos que no deben duplicarse se usa `Categoria::firstOrCreate(['nombre' => $nombre])`.
- **Orden incorrecto en `$this->call`:** si `ProductoSeeder` se ejecuta antes que `CategoriaSeeder`, `Categoria::all()` está vacío y no se crea ningún producto (o fallan las llaves foráneas).
- **Correos duplicados con datos aleatorios:** al generar muchos usuarios, `fake()->email()` puede repetir valores y violar `unique`. Se usa `fake()->unique()->safeEmail()`.
- **Usar factories para datos que la aplicación necesita en producción** (roles, catálogo de estados): esos van en seeders con valores **fijos**, no aleatorios.

---

## 4.4. Transacciones y manejo de errores

### 4.4.1. Transacciones y control de errores con try-catch

**Qué es:**

- Una **transacción** es un grupo de operaciones sobre la base de datos que se trata como **una sola unidad**: o se aplican **todas** (*commit*), o no se aplica **ninguna** (*rollback*).
- **`try-catch`** es la estructura de PHP para **capturar excepciones**: el código que puede fallar va en `try`; si lanza una excepción, la ejecución salta al bloque `catch`, donde se decide cómo responder, en lugar de que el programa termine con un error.

**Por qué existen las transacciones — el ejemplo clásico:** transferir $300 de la cuenta A a la cuenta B requiere **dos** operaciones: restar a A y sumar a B. Si el servidor falla entre ambas, el dinero desaparece:

**Traza — transferencia de $300 con un fallo entre los dos pasos:**

| Paso | Sin transacción: saldo A | Sin transacción: saldo B | Con transacción: saldo A | Con transacción: saldo B |
|---|---|---|---|---|
| Inicio | 1000 | 500 | 1000 | 500 |
| 1. `A -= 300` | **700** (ya guardado) | 500 | 700 (pendiente, no confirmado) | 500 |
| 2. **Falla** (error, corte de luz, excepción) | 700 | 500 | — | — |
| Resultado | 700 | 500 → **se perdieron $300** | **Rollback**: 1000 | 500 → **nada cambió** |
| Si no hubiera fallado | 700 | 800 | Commit: 700 | 800 |

**Propiedades ACID — lo que garantiza una transacción:**

| Propiedad | Significado | En el ejemplo |
|---|---|---|
| **Atomicidad** | Todo o nada | No puede quedar A descontada sin que B reciba |
| **Consistencia** | La base pasa de un estado válido a otro estado válido, respetando las restricciones | La suma total (1500) se mantiene |
| **Aislamiento** (*Isolation*) | Las transacciones simultáneas no ven los cambios a medias de las otras | Otro proceso que consulte el saldo durante la transferencia no ve el estado intermedio (A = 700, B = 500) |
| **Durabilidad** | Una vez confirmada, el cambio sobrevive aunque el servidor se apague | Tras el *commit*, A = 700 y B = 800 aunque se reinicie MySQL |

**Cómo funciona por dentro:** entre `BEGIN` y `COMMIT`, el motor de base de datos (InnoDB en MySQL) registra los cambios en un **registro de deshacer** (*undo log*) y no los hace definitivos. `COMMIT` los vuelve permanentes; `ROLLBACK` usa el registro para restaurar los valores anteriores.

```
BEGIN ──▶ UPDATE A ──▶ UPDATE B ──▶ ¿todo bien?
                                      ├─ sí ──▶ COMMIT   ──▶ cambios permanentes
                                      └─ no ──▶ ROLLBACK ──▶ como si nada hubiera pasado
```

**Forma 1 — `DB::transaction` (automática, recomendada):**

```php
use Illuminate\Support\Facades\DB;

$pedido = DB::transaction(function () use ($carrito, $usuario) {
    $pedido = Pedido::create(['user_id' => $usuario->id, 'total' => $carrito->total()]);

    foreach ($carrito->items as $item) {
        $pedido->detalles()->create([
            'producto_id' => $item->producto_id,
            'cantidad'    => $item->cantidad,
        ]);
        $item->producto->decrement('stock', $item->cantidad);   // UPDATE productos SET stock = stock - ?
    }

    return $pedido;     // lo que devuelva la función es lo que devuelve DB::transaction
});
// Si CUALQUIER línea lanza una excepción: rollback automático y la excepción se vuelve a lanzar.
// Si termina sin excepciones: commit automático.
```

**Forma 2 — manual con `try-catch`:**

```php
DB::beginTransaction();

try {
    $origen->decrement('saldo', 300);
    $destino->increment('saldo', 300);

    DB::commit();                                            // todo salió bien: confirmar
    return back()->with('exito', 'Transferencia realizada');
} catch (\Throwable $e) {
    DB::rollBack();                                          // algo falló: deshacer TODO
    report($e);                                              // registrar el error en storage/logs/laravel.log
    return back()->withErrors(['transferencia' => 'No se pudo completar la transferencia.']);
}
```

**Las dos formas:**

| Aspecto | `DB::transaction(fn)` | `beginTransaction` + `try-catch` |
|---|---|---|
| Commit / rollback | Automáticos | Los escribes tú |
| Riesgo de olvidar el rollback | Ninguno | Existe |
| La excepción después del rollback | Se **vuelve a lanzar** (hay que capturarla afuera si quieres un mensaje amigable) | Tú decides qué hacer en el `catch` |
| Reintentos ante bloqueos (*deadlocks*) | `DB::transaction(fn, 3)` reintenta hasta 3 veces | Manual |
| Cuándo usarla | Por defecto | Cuando necesitas una respuesta distinta según el error dentro del mismo método |

**Combinarlas — mensaje amigable con `DB::transaction`:**

```php
try {
    DB::transaction(function () use ($origen, $destino) {
        if ($origen->saldo < 300) {
            throw new \RuntimeException('Saldo insuficiente');   // provoca el rollback
        }
        $origen->decrement('saldo', 300);
        $destino->increment('saldo', 300);
    });
    return back()->with('exito', 'Transferencia realizada');
} catch (\RuntimeException $e) {
    return back()->withErrors(['saldo' => $e->getMessage()]);   // "Saldo insuficiente"
}
```

> **Ojo con capturar la excepción DENTRO de la función de `DB::transaction`:** si dentro de la función escribes `try { ... } catch (\Exception $e) { }` y no vuelves a lanzar la excepción, para `DB::transaction` "todo salió bien" y hace **commit** de lo que alcanzó a ejecutarse antes del error: exactamente el estado a medias que se quería evitar. Captura afuera de la transacción, o vuelve a lanzar con `throw $e;`.

**Cómo funciona `try-catch` por dentro — el flujo de ejecución:**

```php
try {
    echo "1 ";
    throw new \RuntimeException("falló");
    echo "2 ";                          // nunca se ejecuta
} catch (\RuntimeException $e) {
    echo "3 ";
} finally {
    echo "4";                           // se ejecuta SIEMPRE, haya o no excepción
}
// Salida: 1 3 4
```

| Escenario | Bloques que se ejecutan |
|---|---|
| El `try` termina sin errores | `try` completo → `finally` |
| Excepción que coincide con un `catch` | `try` hasta el error → ese `catch` → `finally` |
| Excepción que **no** coincide con ningún `catch` | `try` hasta el error → `finally` → la excepción sigue subiendo (si nadie la captura, Laravel muestra la página de error 500) |

**Jerarquía de errores en PHP — qué captura cada `catch`:**

```
Throwable
├── Exception
│   ├── RuntimeException
│   ├── InvalidArgumentException
│   ├── PDOException
│   │   └── Illuminate\Database\QueryException      ← errores de SQL en Laravel
│   ├── Illuminate\Validation\ValidationException
│   └── Illuminate\Database\Eloquent\ModelNotFoundException
└── Error                                           ← errores del lenguaje
    ├── TypeError
    └── DivisionByZeroError
```

> **Ojo con `catch (\Exception $e)`:** no captura los `Error` (por ejemplo, un `TypeError` por pasar un tipo incorrecto a una función). En un bloque cuyo objetivo es "hacer rollback pase lo que pase", usa `catch (\Throwable $e)`, que captura ambos. Y si en un archivo con `namespace` escribes `catch (Exception $e)` sin la barra inicial ni un `use`, PHP busca `App\Http\Controllers\Exception`, que no existe, y el `catch` **nunca** coincide.

**Capturar errores de base de datos concretos:**

```php
use Illuminate\Database\QueryException;

try {
    Cliente::create($datos);
} catch (QueryException $e) {
    if ($e->errorInfo[1] === 1062) {                       // código de MySQL: entrada duplicada
        return back()->withErrors(['email' => 'Ese correo ya está registrado.']);
    }
    throw $e;                                              // otros errores: no los ocultes
}
```

**Árbol de decisión — ¿transacción, try-catch, ambos o ninguno?**

```
¿La operación hace MÁS DE UNA escritura que debe ser todo-o-nada?
│
├─ Sí ──▶ TRANSACCIÓN (DB::transaction)
│         └─ ¿Necesitas mostrar un mensaje propio si falla?
│               ├─ Sí ──▶ envuelve DB::transaction en try-catch
│               └─ No ──▶ deja que Laravel muestre el error (y lo registre)
│
└─ No (una sola escritura, o solo lecturas)
      └─ ¿Hay un error ESPERADO que sepas manejar (duplicado, servicio externo caído)?
            ├─ Sí ──▶ try-catch solo para ese tipo de excepción
            └─ No ──▶ ninguno: valida antes (Unidad 3) y deja que el manejador
                      global de Laravel registre los errores inesperados
```

**Errores y malentendidos típicos:**

- **`catch` vacío** (`catch (\Throwable $e) {}`): el error desaparece sin dejar rastro; el usuario cree que la operación se completó y nadie sabe por qué faltan datos. Como mínimo, llama a `report($e)` y muestra un mensaje.
- **Mostrar `$e->getMessage()` al usuario en errores de base de datos:** puede revelar nombres de tablas, columnas o fragmentos de SQL. Muestra un mensaje genérico y registra el detalle en el log.
- **Pensar que la transacción deshace todo:** solo deshace operaciones **de la base de datos**. Si dentro de la transacción guardaste un archivo con `Storage` o enviaste un correo, eso **no** se revierte. Haz esas acciones **después** del commit, o elimina el archivo en el `catch`.
- **Tablas MyISAM:** este motor antiguo de MySQL **no soporta transacciones**; el rollback no hace nada. Laravel crea las tablas con InnoDB por defecto.
- **Crear o modificar tablas dentro de una transacción en MySQL:** las sentencias como `CREATE TABLE` o `ALTER TABLE` provocan un **commit implícito** de lo anterior, así que no se pueden deshacer con rollback.
- **Transacciones muy largas** (por ejemplo, llamar a un servicio externo lento dentro de ellas): mantienen bloqueadas las filas modificadas y hacen esperar a otros usuarios. Dentro de la transacción solo deben ir las operaciones de base de datos.

---

*Universidad Politécnica de Tecamac — Ingeniería en Software*
