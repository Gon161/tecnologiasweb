# Unidad 2 — Introducción a Laravel y el patrón MVC

En la Unidad 1 se vio que una página dinámica es HTML **generado en el servidor** en el momento de cada petición. Esta unidad presenta la herramienta con la que se hará eso durante el resto del curso: **Laravel**, un *framework* de PHP organizado según el patrón **MVC**. Primero se explica el patrón (que es independiente de Laravel y aparece en muchos otros frameworks), después cómo instalar Laravel y cómo está organizado, y finalmente las tres piezas con las que se construye cualquier página: **rutas**, **controladores** y **vistas**.

> Los ejemplos de esta unidad corresponden a **Laravel 11 en adelante** (requiere **PHP 8.2 o superior**). Las versiones anteriores tienen algunos archivos de configuración distintos (por ejemplo, `app/Http/Kernel.php`), que aquí se mencionan cuando la diferencia importa.

---

## Índice

1. [Patrón MVC (Modelo-Vista-Controlador)](#21-patrón-mvc-modelo-vista-controlador)
   - 2.1.1. [Definición, estructura y flujo de datos](#211-definición-estructura-y-flujo-de-datos)
2. [Instalación y estructura de Laravel](#22-instalación-y-estructura-de-laravel)
   - 2.2.1. [Instalación mediante Composer y configuración inicial](#221-instalación-mediante-composer-y-configuración-inicial)
   - 2.2.2. [Directorios principales y archivos de configuración](#222-directorios-principales-y-archivos-de-configuración)
   - 2.2.3. [Uso de Artisan para generar componentes](#223-uso-de-artisan-para-generar-componentes)
3. [Rutas y controladores](#23-rutas-y-controladores)
   - 2.3.1. [Definición de rutas](#231-definición-de-rutas)
   - 2.3.2. [Creación de controladores](#232-creación-de-controladores)
4. [Vistas y Blade](#24-vistas-y-blade)
   - 2.4.1. [Plantillas con Blade](#241-plantillas-con-blade)
   - 2.4.2. [Paso de datos a las vistas](#242-paso-de-datos-a-las-vistas)

---

## 2.1. Patrón MVC (Modelo-Vista-Controlador)

### 2.1.1. Definición, estructura y flujo de datos

**Qué es:** **MVC** (*Model-View-Controller*) es un **patrón de arquitectura** de software que divide una aplicación en tres componentes, cada uno con una única responsabilidad:

- **Modelo (*Model*):** representa los **datos** y las **reglas del negocio**. Sabe cómo leer y guardar información (por ejemplo, en una base de datos) y qué es válido para el dominio ("un producto no puede tener precio negativo").
- **Vista (*View*):** se encarga de la **presentación**. Recibe datos ya preparados y los convierte en algo que el usuario ve (en la web, HTML).
- **Controlador (*Controller*):** es el **intermediario**. Recibe la petición del usuario, decide qué hacer, le pide datos al modelo y elige qué vista mostrar con esos datos.

**Por qué existe:** imagina una página de productos escrita en un único archivo PHP, al estilo de los primeros sitios dinámicos:

```php
<?php // productos.php — TODO mezclado en un solo archivo
$conexion = new mysqli('localhost', 'root', '', 'tienda');           // acceso a datos
$categoria = $_GET['categoria'];
$resultado = $conexion->query("SELECT * FROM productos WHERE categoria = '$categoria'"); // inyección SQL
?>
<h1>Productos</h1>                                                   <!-- presentación -->
<?php while ($p = $resultado->fetch_assoc()): ?>
    <?php if ($p['stock'] > 0): ?>                                    <!-- regla de negocio -->
        <p><?= $p['nombre'] ?> — $<?= $p['precio'] ?></p>             <!-- sin escapar: XSS -->
    <?php endif; ?>
<?php endwhile; ?>
```

Este código funciona, pero tiene problemas que crecen con el proyecto:

| Problema | Consecuencia |
|---|---|
| La consulta SQL está dentro del HTML | Si otra página necesita los mismos productos, se **copia** la consulta; si cambia la tabla, hay que buscar y corregir cada copia |
| El diseñador y el programador editan el mismo archivo | Un cambio de diseño puede romper la lógica, y viceversa |
| No hay un punto central que reciba las peticiones | Cada archivo debe repetir la verificación de sesión, la validación, el manejo de errores |
| No se puede probar la lógica por separado | Para comprobar la regla "solo con stock" hay que generar el HTML completo |

MVC resuelve esto **separando responsabilidades**: cada cambio afecta a un solo componente. Si cambia el diseño, solo se toca la vista; si cambia la base de datos, solo el modelo.

**Características:**

| Característica | Descripción |
|---|---|
| **Separación de responsabilidades** | Cada capa tiene un trabajo y solo uno |
| **Bajo acoplamiento** | La vista no sabe de dónde vienen los datos (podrían venir de MySQL, de un archivo o de una API); el modelo no sabe cómo se mostrarán |
| **Reutilización** | Un mismo modelo sirve para varias vistas (página web, PDF, respuesta JSON de una API) |
| **Trabajo en paralelo** | Un equipo puede trabajar en las vistas mientras otro trabaja en los modelos |
| **Facilidad de pruebas** | Los modelos y controladores se pueden probar sin generar HTML |

**Estructura — quién habla con quién:**

```
                         ┌────────────────────┐
         1. petición     │                    │   3. pide/guarda datos   ┌────────────┐
 Usuario ──────────────▶ │    CONTROLADOR     │ ───────────────────────▶ │   MODELO   │ ◀──▶ Base de datos
 (navegador)             │ (coordina, decide) │ ◀─────────────────────── │  (datos +  │
     ▲                   │                    │   4. devuelve los datos  │   reglas)  │
     │                   └─────────┬──────────┘                          └────────────┘
     │                             │ 5. entrega datos a la vista
     │                             ▼
     │                   ┌────────────────────┐
     └────────────────── │       VISTA        │
       6. HTML resultante│  (presentación)    │
                         └────────────────────┘
```

> **Ojo con una regla que se rompe con frecuencia:** en el MVC web, la **vista no consulta la base de datos** y el **modelo no genera HTML**. Si en una plantilla aparece una consulta, o si un modelo devuelve una cadena con etiquetas `<p>`, la separación se perdió.

**Cómo funciona por dentro — el flujo completo en Laravel**, con el ejemplo de un usuario que visita `/productos`:

```
Navegador: GET /productos
   │
   ▼
public/index.php                ← único punto de entrada de TODAS las peticiones
   │
   ▼
Middleware                      ← filtros previos: sesión, cookies, CSRF, autenticación (Unidad 3)
   │
   ▼
Router  (routes/web.php)        ← "GET /productos le corresponde a ProductoController@index"
   │
   ▼
Controlador ProductoController::index()
   │   $productos = Producto::where('stock', '>', 0)->get();   ── Modelo ──▶ SQL ──▶ MySQL
   │   return view('productos.index', compact('productos'));
   ▼
Vista resources/views/productos/index.blade.php
   │   @foreach ($productos as $p) <p>{{ $p->nombre }}</p> @endforeach
   ▼
Respuesta HTTP 200 con el HTML ──▶ Navegador
```

**Traza — qué tiene cada capa en cada momento** (suponiendo que en la tabla hay 3 productos y uno no tiene stock):

| Paso | Componente | Qué recibe | Qué produce |
|---|---|---|---|
| 1 | Router | `GET /productos` | Decide llamar a `ProductoController@index` |
| 2 | Controlador | La petición (`Request`) | Le pide al modelo los productos con stock |
| 3 | Modelo (Eloquent) | La condición `stock > 0` | Ejecuta `select * from productos where stock > 0` y devuelve una colección con **2** objetos `Producto` |
| 4 | Controlador | La colección de 2 productos | Llama a la vista `productos.index` pasándole `$productos` |
| 5 | Vista (Blade) | `$productos` (2 elementos) | HTML con 2 párrafos `<p>` |
| 6 | Laravel | El HTML | Respuesta `200 OK` con `Content-Type: text/html` |

**Comparación de responsabilidades:**

| Pregunta | Modelo | Vista | Controlador |
|---|---|---|---|
| ¿Accede a la base de datos? | **Sí** | No | No directamente (lo hace a través del modelo) |
| ¿Contiene HTML? | No | **Sí** | No |
| ¿Lee datos de la petición (`$request`)? | No | No | **Sí** |
| ¿Contiene reglas del negocio? | **Sí** | No | Mínimas (coordinar, no decidir el negocio) |
| ¿Archivo típico en Laravel? | `app/Models/Producto.php` | `resources/views/productos/index.blade.php` | `app/Http/Controllers/ProductoController.php` |

**Errores y malentendidos típicos:**

- **"Controladores gordos" (*fat controllers*):** poner toda la lógica (cálculos, reglas de descuento, validaciones complejas) en el controlador. Funciona, pero el controlador se vuelve enorme y esa lógica no se puede reutilizar desde otro lugar. La regla general es "**modelos gordos, controladores delgados**": el controlador coordina, el modelo (o clases de servicio) contiene las reglas.
- **Pensar que el "modelo" es solo la tabla:** el modelo representa la entidad del negocio completa, incluidas sus reglas y relaciones (por ejemplo, un método `$pedido->calcularTotal()`), no solo las columnas.
- **Creer que MVC es exclusivo de Laravel:** es un patrón de 1979 (originalmente para interfaces de escritorio en Smalltalk). Lo usan Ruby on Rails, Django (que lo llama MTV), ASP.NET MVC y Spring MVC, con la misma idea de fondo.

**MVC frente a otras organizaciones:**

| Organización | Descripción | Cuándo se usa |
|---|---|---|
| Script único (sin patrón) | Todo en un archivo | Prototipos de minutos o páginas triviales |
| **MVC** | Controlador recibe la petición, modelo da datos, vista genera HTML en el servidor | Aplicaciones web tradicionales: Laravel, Rails, Django |
| API + *frontend* separado | El servidor solo devuelve JSON (sin vistas); otra aplicación en JavaScript (React, Vue) construye el HTML en el navegador | Aplicaciones muy interactivas o con app móvil que comparte el mismo servidor. Laravel también sirve para esto: los controladores devuelven JSON en vez de vistas |

---

## 2.2. Instalación y estructura de Laravel

**Qué es:** **Laravel** es un *framework* web de código abierto para **PHP**, creado por Taylor Otwell en 2011. Un **framework** es un conjunto de código ya escrito y organizado que resuelve las tareas comunes de cualquier aplicación (rutas, base de datos, sesiones, validación, seguridad), de modo que el programador solo escribe lo específico de su proyecto.

**Por qué usar un framework y no PHP "puro":** todas las aplicaciones web necesitan resolver los mismos problemas: interpretar URLs, proteger formularios contra CSRF, escapar HTML contra XSS, conectarse a la base de datos de forma segura, manejar sesiones. Programarlos desde cero en cada proyecto es lento y es fácil cometer errores de seguridad. Laravel trae soluciones probadas por miles de proyectos para todos ellos.

**Diferencia entre librería y framework:**

| Aspecto | Librería | Framework |
|---|---|---|
| Quién controla el flujo | **Tu código** llama a la librería cuando la necesita | **El framework** llama a tu código cuando corresponde (*inversión de control*) |
| Ejemplo | Una librería para generar PDFs | Laravel: decide cuándo ejecutar tu controlador según la ruta |
| Estructura del proyecto | Libre | Impuesta por convenciones (carpetas, nombres) |

### 2.2.1. Instalación mediante Composer y configuración inicial

**Qué es Composer:** **Composer** es el **gestor de dependencias** de PHP. Una **dependencia** es un paquete de código escrito por otros que tu proyecto necesita para funcionar. Laravel en sí está formado por decenas de paquetes, y Composer se encarga de descargarlos, en la versión correcta, a la carpeta `vendor/`.

**Por qué existe:** sin un gestor de dependencias habría que descargar cada librería a mano, copiarla al proyecto, verificar que sus versiones sean compatibles entre sí (el paquete A necesita la versión 2 de C, el paquete B la versión 3 de C…) y repetir todo cada vez que alguien clona el proyecto. Composer resuelve automáticamente ese "árbol de dependencias".

**Cómo funciona por dentro — `composer.json` y `composer.lock`:**

| Archivo | Qué contiene | Ejemplo | ¿Se sube a Git? |
|---|---|---|---|
| `composer.json` | Las dependencias que **pediste** y el **rango** de versiones aceptable | `"laravel/framework": "^11.0"` (cualquier 11.x) | Sí |
| `composer.lock` | La versión **exacta** que se instaló de cada paquete (y de sus subdependencias) | `laravel/framework` → `v11.9.2` | **Sí** (garantiza que todo el equipo use exactamente lo mismo) |
| `vendor/` | El código descargado | Miles de archivos | **No** (se regenera con `composer install`) |

```
composer install   ──▶ lee composer.lock ──▶ instala EXACTAMENTE esas versiones   (uso diario, al clonar)
composer update    ──▶ lee composer.json ──▶ busca las versiones más nuevas dentro del rango
                                            y REESCRIBE composer.lock               (actualizar a propósito)
```

> **Ojo con `^11.0`:** el acento circunflejo significa "versiones compatibles": acepta `11.0`, `11.5`, `11.31`, pero **no** `12.0`, porque un cambio en el primer número (versión mayor) puede romper la compatibilidad. Es la convención de **versionado semántico** (MAYOR.MENOR.PARCHE).

**Requisitos previos:**

| Herramienta | Para qué | Cómo comprobarlo |
|---|---|---|
| **PHP 8.2+** con extensiones comunes (`mbstring`, `openssl`, `pdo`, `tokenizer`, `xml`, `ctype`, `fileinfo`) | Ejecutar Laravel | `php -v` |
| **Composer** | Descargar Laravel y sus dependencias | `composer -V` |
| Base de datos (MySQL/MariaDB, o SQLite incluido en PHP) | Guardar datos (Unidad 4) | — |
| Node.js y npm (opcional al principio) | Compilar CSS/JS con Vite | `node -v` |

> En Windows, la forma más sencilla de tener todo junto es instalar un paquete como **Laragon** o **XAMPP** (PHP + MySQL + Apache), o usar el instalador oficial **Laravel Herd**. En cualquiera de los casos, verifica que `php -v` funcione desde la terminal antes de continuar.

**Instalación — dos opciones equivalentes:**

```bash
# Opción 1: directamente con Composer
composer create-project laravel/laravel mi-tienda

# Opción 2: con el instalador de Laravel (se instala una sola vez)
composer global require laravel/installer
laravel new mi-tienda          # hace preguntas: starter kit, base de datos, pruebas…

# Entrar al proyecto y levantar el servidor de desarrollo
cd mi-tienda
php artisan serve              # INFO  Server running on [http://127.0.0.1:8000].
```

**Qué hace `create-project` paso a paso:**

| Paso | Acción | Resultado |
|---|---|---|
| 1 | Descarga el "esqueleto" del proyecto (`laravel/laravel`) | Carpetas `app/`, `routes/`, `resources/`… |
| 2 | Ejecuta `composer install` | Descarga el framework y sus dependencias a `vendor/` |
| 3 | Copia `.env.example` a `.env` | Archivo de configuración local |
| 4 | Ejecuta `php artisan key:generate` | Escribe un valor aleatorio en `APP_KEY` dentro de `.env` |
| 5 | (Laravel 11+) Crea `database/database.sqlite` y ejecuta las migraciones iniciales | Base de datos lista con tablas de usuarios, sesiones, caché |

**Configuración inicial — el archivo `.env`:** **`.env`** (*environment*, entorno) guarda la configuración que **cambia de una computadora a otra**: datos de la base de datos, contraseñas, si se muestran errores detallados, etc.

```ini
APP_NAME="Mi Tienda"
APP_ENV=local                 # local, staging, production
APP_KEY=base64:Q2x...=        # clave de cifrado; la genera key:generate
APP_DEBUG=true                # muestra errores detallados (NUNCA true en producción)
APP_URL=http://localhost:8000
APP_LOCALE=es                 # idioma de los mensajes (validación, etc.)

DB_CONNECTION=sqlite          # en Laravel 11+ el valor por defecto es sqlite (Unidad 4 cambia a mysql)

SESSION_DRIVER=database       # dónde se guardan las sesiones (Unidad 3)
```

**Por qué existe `.env` separado del código:** el código es el mismo en tu laptop, en la de tu compañero y en el servidor de producción, pero la contraseña de la base de datos no. Si las credenciales estuvieran dentro del código, al subirlo a GitHub cualquiera podría verlas. Por eso `.env` está en `.gitignore` y se comparte `.env.example` (con las claves pero sin valores secretos) como plantilla.

> **Ojo con `APP_KEY`:** Laravel la usa para **cifrar** las cookies y la sesión. Si falta, aparece el error *"No application encryption key has been specified"*; se soluciona con `php artisan key:generate`. Si la cambias en un sitio en producción, todos los usuarios pierden su sesión, porque las cookies cifradas con la clave anterior ya no se pueden descifrar.

> **Ojo con `APP_DEBUG=true` en producción:** la página de error detallada muestra fragmentos de código, rutas de archivos y variables de entorno (incluidas contraseñas). En un servidor público debe ser siempre `false`.

**Errores y malentendidos típicos:**

- **Clonar un proyecto de Laravel y que no funcione:** al clonar no vienen `vendor/` ni `.env` (están en `.gitignore`). Pasos: `composer install`, copiar `.env.example` a `.env`, `php artisan key:generate`, configurar la base de datos y `php artisan migrate`.
- **"`php` no se reconoce como un comando":** PHP no está en la variable de entorno `PATH`. Hay que agregar la carpeta donde está `php.exe` al PATH de Windows, o usar la terminal que trae Laragon/XAMPP.
- **Editar `.env` y no ver el cambio:** si se ejecutó `php artisan config:cache`, Laravel lee la configuración desde la caché, no desde `.env`. Se soluciona con `php artisan config:clear` (ver 2.2.2).
- **Abrir el proyecto desde `http://localhost/mi-tienda`** en XAMPP y ver una lista de carpetas o un error: el servidor web debe apuntar a la carpeta **`public/`**, no a la raíz del proyecto. Con `php artisan serve` esto ya está resuelto.

### 2.2.2. Directorios principales y archivos de configuración

**Estructura de un proyecto Laravel:**

```
mi-tienda/
├── app/                        ← TU código de la aplicación
│   ├── Http/
│   │   └── Controllers/        ← controladores (C de MVC)
│   ├── Models/                 ← modelos Eloquent (M de MVC), p. ej. User.php
│   └── Providers/              ← configuración de servicios al arrancar
├── bootstrap/
│   ├── app.php                 ← arranque: registra rutas, middleware y manejo de excepciones
│   └── cache/                  ← cachés generadas por el framework
├── config/                     ← archivos de configuración (app.php, database.php, session.php…)
├── database/
│   ├── migrations/             ← definición de tablas (Unidad 4)
│   ├── factories/              ← generadores de datos de prueba (Unidad 4)
│   └── seeders/                ← inserción de datos iniciales (Unidad 4)
├── public/                     ← ÚNICA carpeta accesible desde el navegador
│   └── index.php               ← punto de entrada de todas las peticiones
├── resources/
│   ├── views/                  ← plantillas Blade (V de MVC)
│   ├── css/  y  js/            ← archivos fuente de estilos y scripts (los compila Vite)
├── routes/
│   ├── web.php                 ← rutas de las páginas web
│   └── console.php             ← comandos de Artisan personalizados
├── storage/
│   ├── app/                    ← archivos subidos por usuarios (Unidad 3)
│   ├── framework/              ← vistas compiladas, sesiones en archivo, caché
│   └── logs/laravel.log        ← registro de errores
├── tests/                      ← pruebas automatizadas
├── vendor/                     ← dependencias de Composer (no se edita, no se sube a Git)
├── .env                        ← configuración local y secretos (no se sube a Git)
├── artisan                     ← herramienta de línea de comandos
└── composer.json               ← dependencias del proyecto
```

**Características de cada directorio:**

| Directorio | Qué contiene | ¿Lo editas normalmente? |
|---|---|---|
| `app/` | Controladores, modelos, middleware, clases propias | **Sí, constantemente** |
| `routes/` | Mapa de URLs a controladores | **Sí** |
| `resources/views/` | Plantillas Blade | **Sí** |
| `database/` | Migraciones, *seeders*, *factories* | **Sí** |
| `config/` | Configuración por áreas | A veces |
| `public/` | `index.php` y archivos públicos (imágenes, CSS compilado, `favicon.ico`) | Rara vez |
| `storage/` | Archivos generados en tiempo de ejecución | No a mano (revisa `logs/` al depurar) |
| `bootstrap/` | Arranque del framework | Solo `app.php` para registrar middleware |
| `vendor/` | Código del framework y paquetes | **Nunca** (se sobrescribe con cada `composer install`) |

**Cómo funciona por dentro — por qué solo `public/` es accesible:** el servidor web se configura para que la raíz del sitio sea `public/`. Así, una petición a `http://sitio.com/.env` busca el archivo `public/.env`, que no existe, en vez de entregar tu archivo de contraseñas. Todo lo demás (`app/`, `.env`, `vendor/`) queda **fuera del alcance** del navegador y solo es accesible para PHP.

```
Navegador ve:                           En disco:
http://sitio.com/               ──▶    mi-tienda/public/index.php
http://sitio.com/favicon.ico    ──▶    mi-tienda/public/favicon.ico
http://sitio.com/.env           ──▶    mi-tienda/public/.env   → no existe → 404
                                        mi-tienda/.env          ← inalcanzable desde la web
```

**Archivos de configuración — la relación entre `config/` y `.env`:** los archivos de `config/` son arreglos de PHP que **leen** valores de `.env` con la función `env()`, y proporcionan un valor por defecto si no existe:

```php
// config/app.php (fragmento)
return [
    'name'     => env('APP_NAME', 'Laravel'),   // si APP_NAME no está en .env, usa 'Laravel'
    'debug'    => (bool) env('APP_DEBUG', false),
    'timezone' => 'UTC',                        // valor fijo, igual en todos los entornos
    'locale'   => env('APP_LOCALE', 'en'),
];
```

En el resto de la aplicación (controladores, vistas) se lee la configuración con `config()`, **nunca** con `env()`:

```php
config('app.name');                 // "Mi Tienda"   → archivo app.php, clave 'name'
config('database.default');         // "sqlite"
config('app.inexistente', 'x');     // "x"           → valor por defecto si no existe
```

> **Ojo con usar `env()` fuera de `config/`:** en producción se ejecuta `php artisan config:cache`, que junta todos los archivos de configuración en uno solo ya calculado (más rápido). A partir de ese momento Laravel **deja de leer `.env`**, y cualquier `env('ALGO')` escrito en un controlador devuelve `null`. Si siempre usas `config()`, tu código funciona igual con o sin caché.

| Archivo de `config/` | Qué configura |
|---|---|
| `app.php` | Nombre, entorno, depuración, zona horaria, idioma |
| `database.php` | Conexiones a bases de datos (Unidad 4) |
| `session.php` | Driver y duración de las sesiones (Unidad 3) |
| `filesystems.php` | Discos de almacenamiento de archivos (Unidad 3) |
| `auth.php` | Autenticación: *guards* y proveedores de usuarios (Unidad 3) |

**Errores y malentendidos típicos:**

- **Modificar archivos dentro de `vendor/`:** el cambio se pierde en el próximo `composer install` o `update`. Para cambiar el comportamiento del framework se usan los mecanismos de extensión de Laravel (configuración, *service providers*, clases propias).
- **Buscar `app/Http/Kernel.php` en Laravel 11+:** desde la versión 11 ese archivo ya no existe; el middleware se registra en `bootstrap/app.php`. Muchos tutoriales en Internet aún corresponden a versiones anteriores.
- **Guardar imágenes subidas por usuarios directamente en `public/`:** se mezclan con los archivos del proyecto y se pierden si se vuelve a desplegar. Para eso existe `storage/app` (Unidad 3).

### 2.2.3. Uso de Artisan para generar componentes

**Qué es:** **Artisan** es la **interfaz de línea de comandos** incluida en Laravel. Es el archivo `artisan` de la raíz del proyecto, y se ejecuta con `php artisan <comando>`.

**Por qué existe:** muchas tareas son repetitivas y siguen siempre la misma estructura: crear un controlador implica crear un archivo en la carpeta correcta, con el *namespace* correcto, la clase con el nombre correcto y los métodos esperados. Artisan genera ese código base (*scaffolding*) sin errores de escritura, y además ejecuta tareas de mantenimiento (migrar la base de datos, limpiar cachés, listar rutas).

**Cómo funciona por dentro:** los comandos `make:*` usan **plantillas** (*stubs*) internas: toman el nombre que escribes, lo sustituyen en la plantilla y guardan el archivo resultante en la carpeta que corresponde por convención.

```
php artisan make:controller ProductoController
        │
        ├─ toma el stub "controller.plain.stub"
        ├─ reemplaza {{ class }} por ProductoController y {{ namespace }} por App\Http\Controllers
        └─ guarda en app/Http/Controllers/ProductoController.php
```

**Comandos de generación (`make:`):**

| Comando | Genera | Ubicación |
|---|---|---|
| `make:controller ProductoController` | Controlador vacío | `app/Http/Controllers/` |
| `make:controller ProductoController --resource` | Controlador con los 7 métodos CRUD (2.3.2) | `app/Http/Controllers/` |
| `make:model Producto` | Modelo Eloquent | `app/Models/` |
| `make:model Producto -mfsc` | Modelo + **m**igración + **f**actory + **s**eeder + **c**ontrolador | Varias carpetas |
| `make:migration create_productos_table` | Migración (Unidad 4) | `database/migrations/` |
| `make:middleware VerificarEdad` | Middleware (Unidad 3) | `app/Http/Middleware/` |
| `make:request GuardarProductoRequest` | Clase de validación (Unidad 3) | `app/Http/Requests/` |
| `make:seeder` / `make:factory` | Datos de prueba (Unidad 4) | `database/seeders/`, `database/factories/` |
| `make:view productos.index` | Vista Blade vacía (Laravel 11+) | `resources/views/productos/index.blade.php` |

**Comandos de uso diario:**

| Comando | Para qué |
|---|---|
| `php artisan serve` | Servidor de desarrollo en `http://127.0.0.1:8000` |
| `php artisan route:list` | Lista todas las rutas registradas: método, URI, nombre y controlador |
| `php artisan migrate` | Ejecuta las migraciones pendientes (Unidad 4) |
| `php artisan tinker` | Consola interactiva de PHP con Laravel cargado, para probar código y consultas |
| `php artisan optimize:clear` | Borra todas las cachés (configuración, rutas, vistas) |
| `php artisan list` | Muestra todos los comandos disponibles |
| `php artisan help make:controller` | Muestra las opciones de un comando |

**Ejemplo — sesión de `tinker`:**

```
> 2 + 3
= 5
> config('app.name')
= "Mi Tienda"
> App\Models\User::count()
= 0
> now()->format('d/m/Y')
= "17/10/2026"
```

**Convenciones de nombres que Artisan espera:**

| Componente | Convención | Ejemplo correcto | Ejemplo incorrecto |
|---|---|---|---|
| Modelo | Singular, PascalCase | `Producto` | `productos`, `Productos` |
| Tabla (la asume Laravel) | Plural, snake_case | `productos` | `Producto` |
| Controlador | Singular + `Controller` | `ProductoController` | `ProductosCtrl` |
| Migración | Descriptiva en snake_case | `create_productos_table` | `tabla1` |

> **Por qué importan las convenciones:** Laravel aplica el principio de **convención sobre configuración**: si sigues los nombres esperados, no necesitas configurar nada (por ejemplo, el modelo `Producto` sabe sin que se lo digas que su tabla es `productos`). Si no las sigues, tendrás que indicarlo manualmente en cada caso (Unidad 4).

**Errores y malentendidos típicos:**

- **Ejecutar `php artisan` fuera de la carpeta del proyecto:** aparece *"Could not open input file: artisan"*. El comando debe ejecutarse donde está el archivo `artisan`.
- **Crear archivos a mano con el *namespace* equivocado:** el autoload de Composer no los encuentra y aparece *"Class not found"*. Artisan evita este error.
- **Usar `make:model` con nombre plural:** `make:model Productos` crearía la clase `Productos` que buscaría la tabla `productos`, pero todo el código se leería raro (`Productos::find(1)` para un solo producto).

---

## 2.3. Rutas y controladores

### 2.3.1. Definición de rutas

**Qué es:** una **ruta** es una regla que asocia una **URL** y un **método HTTP** con el **código** que debe ejecutarse cuando llega una petición que coincide. Las rutas web se definen en `routes/web.php`.

**Por qué existen:** en PHP tradicional, la URL corresponde a un archivo en disco (`/contacto.php` ejecuta `contacto.php`). Eso expone la estructura interna del proyecto y genera URLs poco legibles (`/producto.php?id=5`). Con un **enrutador** (*router*), todas las peticiones entran por `public/index.php` y es la tabla de rutas la que decide qué código atiende cada URL. Así las URLs son limpias (`/productos/5`), independientes de los archivos, y hay un lugar central donde aplicar seguridad.

**Sintaxis básica:**

```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\ProductoController;

// Ruta con una función anónima (closure): útil para páginas simples o pruebas
Route::get('/', function () {
    return view('welcome');                     // devuelve una vista
});

Route::get('/hola', fn () => 'Hola mundo');     // devuelve texto plano → "Hola mundo"

// Ruta que apunta a un controlador (la forma habitual)
Route::get('/productos', [ProductoController::class, 'index']);
//        └── URI ────┘  └────── clase ──────────┘  └ método ┘

// Una ruta por cada método HTTP
Route::post('/productos', [ProductoController::class, 'store']);
Route::put('/productos/{id}', [ProductoController::class, 'update']);
Route::delete('/productos/{id}', [ProductoController::class, 'destroy']);

// Ruta que solo muestra una vista (sin lógica)
Route::view('/acerca', 'acerca');
```

**Parámetros de ruta:**

```php
// Parámetro obligatorio: /productos/5 → $id = "5"
Route::get('/productos/{id}', function (string $id) {
    return "Producto número $id";               // /productos/5 → "Producto número 5"
});

// Parámetro opcional: lleva ? y necesita un valor por defecto
Route::get('/saludo/{nombre?}', function (string $nombre = 'invitado') {
    return "Hola, $nombre";                     // /saludo → "Hola, invitado"   /saludo/Ana → "Hola, Ana"
});

// Restricción con expresión regular: solo acepta dígitos
Route::get('/usuarios/{id}', fn (string $id) => "Usuario $id")
    ->where('id', '[0-9]+');                    // /usuarios/abc → 404
// atajo equivalente: ->whereNumber('id')
```

**Rutas con nombre:** se asigna un nombre con `->name()` y se generan las URLs con `route()`:

```php
Route::get('/productos/{id}', [ProductoController::class, 'show'])->name('productos.show');

route('productos.show', ['id' => 5]);   // "http://127.0.0.1:8000/productos/5"
```

> **Por qué usar nombres en lugar de escribir la URL:** si en las vistas escribes `href="/productos/5"` en 30 lugares y luego decides cambiar la URL a `/catalogo/5`, tendrías que modificar 30 archivos. Con `route('productos.show', 5)` solo cambias la definición de la ruta, y todos los enlaces se actualizan solos.

**Grupos de rutas:** agrupan rutas que comparten prefijo, nombre o middleware para no repetirlo:

```php
Route::prefix('admin')->name('admin.')->middleware('auth')->group(function () {
    Route::get('/usuarios', [UsuarioController::class, 'index'])->name('usuarios');
    // URI: /admin/usuarios     Nombre: admin.usuarios     Requiere sesión iniciada
    Route::get('/reportes', [ReporteController::class, 'index'])->name('reportes');
    // URI: /admin/reportes     Nombre: admin.reportes
});
```

**Rutas de recurso — las 7 rutas CRUD en una línea:**

```php
Route::resource('productos', ProductoController::class);
```

| Método HTTP | URI | Acción del controlador | Nombre de ruta | Para qué |
|---|---|---|---|---|
| GET | `/productos` | `index` | `productos.index` | Listar todos |
| GET | `/productos/create` | `create` | `productos.create` | Mostrar formulario de alta |
| POST | `/productos` | `store` | `productos.store` | Guardar el nuevo registro |
| GET | `/productos/{producto}` | `show` | `productos.show` | Mostrar uno |
| GET | `/productos/{producto}/edit` | `edit` | `productos.edit` | Mostrar formulario de edición |
| PUT/PATCH | `/productos/{producto}` | `update` | `productos.update` | Guardar los cambios |
| DELETE | `/productos/{producto}` | `destroy` | `productos.destroy` | Eliminar |

**Cómo funciona por dentro — cómo el router elige la ruta:** al llegar una petición, Laravel recorre las rutas registradas **en el orden en que se definieron** y usa la **primera** cuya URI y método coincidan.

**Traza — por qué el orden importa:**

```php
Route::get('/productos/{id}', ...);      // (1)
Route::get('/productos/nuevo', ...);     // (2)
```

| Petición | ¿Coincide (1)? | ¿Coincide (2)? | Se ejecuta |
|---|---|---|---|
| `GET /productos/5` | Sí, `id = 5` | No | (1) |
| `GET /productos/nuevo` | **Sí**, `id = "nuevo"` | Sí | **(1)** ← la ruta (2) nunca se alcanza |
| `POST /productos/5` | No (método distinto) | No | Ninguna → `405 Method Not Allowed` |
| `GET /clientes` | No | No | Ninguna → `404 Not Found` |

Soluciones: definir primero las rutas **fijas** (`/productos/nuevo`) y después las que tienen parámetros, o restringir el parámetro con `->whereNumber('id')`. `Route::resource` ya registra `create` antes de `{producto}` para evitar este problema.

**Errores y malentendidos típicos:**

- **"Route [x] not defined":** se usó `route('x')` con un nombre que no existe o que está mal escrito. `php artisan route:list` muestra todos los nombres.
- **Error 405 al enviar un formulario:** la ruta existe, pero para otro método (por ejemplo, se definió `Route::get` y el formulario usa `method="POST"`).
- **Usar un *closure* para toda la aplicación:** funciona, pero convierte `web.php` en un archivo enorme que mezcla el mapa de URLs con la lógica, y esa lógica no se puede reutilizar ni probar por separado. Los *closures* sirven para pruebas rápidas; las rutas reales deben apuntar a controladores (2.3.2).
- **Olvidar el valor por defecto del parámetro opcional:** `{nombre?}` sin `$nombre = '...'` produce un error cuando la URL no lo incluye.

### 2.3.2. Creación de controladores

**Qué es:** un **controlador** es una clase de PHP que agrupa la lógica para atender las peticiones relacionadas con un mismo recurso. Cada método público del controlador que se asocia a una ruta se llama **acción**.

**Por qué existen:** si toda la lógica estuviera en *closures* dentro de `routes/web.php`, el archivo crecería a miles de líneas imposibles de mantener. El controlador ordena el código por recurso (`ProductoController` para todo lo relacionado con productos) y deja `web.php` como un **mapa** limpio de la aplicación.

**Crear un controlador:**

```bash
php artisan make:controller ProductoController             # vacío
php artisan make:controller ProductoController --resource  # con index, create, store, show, edit, update, destroy
php artisan make:controller ContactoController --invokable # una sola acción (__invoke)
```

**Ejemplo de controlador:**

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class ProductoController extends Controller
{
    // GET /productos
    public function index()
    {
        $productos = ['Teclado', 'Mouse', 'Monitor'];   // en la Unidad 4 vendrá de la base de datos
        return view('productos.index', ['productos' => $productos]);
    }

    // GET /productos/{id}
    public function show(string $id)
    {
        return "Detalle del producto $id";              // /productos/7 → "Detalle del producto 7"
    }

    // POST /productos
    public function store(Request $request)
    {
        $nombre = $request->input('nombre');            // lee el campo "nombre" del formulario
        // ... guardar (Unidad 4)
        return redirect()->route('productos.index');    // respuesta 302 hacia /productos
    }
}
```

**Cómo funciona por dentro — inyección de dependencias:** observa que `store(Request $request)` recibe un objeto `Request` aunque nunca lo creaste. Laravel tiene un **contenedor de servicios** que, al llamar a la acción, revisa los tipos de los parámetros mediante **reflexión** (lee la firma del método) y construye automáticamente lo que haga falta:

```
Router: "hay que llamar a ProductoController@store para la petición actual"
   │
   ├─ ¿qué pide store()?   → un parámetro de tipo Request
   ├─ el contenedor entrega el objeto Request de la petición en curso
   └─ ProductoController->store($request)
```

Los parámetros de ruta se asignan **por nombre**: `{id}` en la ruta llega al parámetro `$id` del método, sin importar el orden respecto a `Request`.

**Qué puede devolver una acción:**

| Retorno | Resultado HTTP |
|---|---|
| `return view('x', [...])` | `200` con HTML |
| `return 'texto'` | `200` con texto |
| `return ['a' => 1]` o un modelo/colección | `200` con JSON: `{"a":1}` |
| `return redirect('/ruta')` / `redirect()->route('nombre')` | `302` con cabecera `Location` |
| `return back()` | `302` hacia la página anterior |
| `abort(404)` | Página de error 404 |
| `return response('Creado', 201)` | Respuesta con código personalizado |

**Route model binding — el modelo llega solo:** cuando el parámetro de la ruta tiene el mismo nombre que una variable con tipo de modelo, Laravel busca el registro automáticamente:

```php
Route::get('/productos/{producto}', [ProductoController::class, 'show']);

public function show(Producto $producto)   // Laravel ejecuta Producto::findOrFail($valorDeLaUrl)
{
    return view('productos.show', compact('producto'));
}
// /productos/3   → $producto es el registro con id 3
// /productos/999 → si no existe, responde 404 sin ejecutar show()
```

> **Ojo con el nombre:** el *binding* solo funciona si el parámetro de la ruta (`{producto}`) y la variable del método (`$producto`) se llaman **igual**. Si la ruta dice `{id}` y el método `Producto $producto`, Laravel crea un `Producto` **vacío** en lugar de buscarlo.

**Tipos de controladores:**

| Tipo | Cómo se crea | Cuándo usarlo |
|---|---|---|
| Básico | `make:controller X` | Acciones variadas que no siguen el patrón CRUD |
| De recurso | `make:controller X --resource` + `Route::resource` | CRUD completo de una entidad (lo más común) |
| De una sola acción | `make:controller X --invokable` + `Route::post('/contacto', X::class)` | Una tarea concreta y aislada (enviar contacto, generar un reporte) |

**Errores y malentendidos típicos:**

- **"Target class [ProductoController] does not exist":** falta el `use App\Http\Controllers\ProductoController;` al inicio de `web.php`, o el nombre de la clase no coincide con el del archivo.
- **Poner la consulta y el HTML dentro del controlador:** el controlador debe **delegar**: consultar mediante el modelo y presentar mediante la vista (2.1.1).
- **Olvidar `return`:** una acción que no devuelve nada genera una respuesta vacía (página en blanco con `200`), lo que es muy confuso de depurar.

---

## 2.4. Vistas y Blade

### 2.4.1. Plantillas con Blade

**Qué es:** **Blade** es el **motor de plantillas** de Laravel. Una plantilla Blade es un archivo HTML con extensión `.blade.php` (en `resources/views/`) en el que se pueden insertar datos y estructuras de control con una sintaxis breve: `{{ }}` para imprimir y directivas con `@` para la lógica de presentación.

**Por qué existe:** PHP puede mezclarse directamente con HTML (`<?php echo $nombre; ?>`), pero esa sintaxis es larga, fácil de romper y, lo más grave, **no escapa** el contenido: si `$nombre` contiene `<script>`, el script se ejecuta en el navegador del visitante (ataque **XSS**). Blade ofrece una sintaxis más limpia, **escapa automáticamente** todo lo que imprime y permite **herencia de plantillas** para no repetir el encabezado y el pie en cada página.

**Cómo funciona por dentro — compilación a PHP:** el navegador nunca ve código Blade. La primera vez que se usa una vista, Laravel la **compila** a PHP normal y guarda el resultado en `storage/framework/views/`. En las siguientes peticiones usa directamente el archivo compilado, mientras la plantilla original no cambie.

```
resources/views/saludo.blade.php            storage/framework/views/a1b2c3….php
┌────────────────────────────────┐          ┌───────────────────────────────────────────────┐
│ <h1>Hola, {{ $nombre }}</h1>   │ ──────▶  │ <h1>Hola, <?php echo e($nombre); ?></h1>      │
│ @if ($admin)                   │ compila  │ <?php if($admin): ?>                          │
│   <p>Eres admin</p>            │ (1 vez)  │   <p>Eres admin</p>                           │
│ @endif                         │          │ <?php endif; ?>                               │
└────────────────────────────────┘          └───────────────────────────────────────────────┘
                                                       │ se ejecuta en cada petición
                                                       ▼
                                             <h1>Hola, Ana</h1><p>Eres admin</p>  → navegador
```

La función `e()` aplica `htmlspecialchars`, que convierte los caracteres especiales de HTML en entidades.

**Imprimir datos — escapado vs sin escapar:**

```blade
@php $comentario = '<script>alert("hackeado")</script>'; @endphp

{{ $comentario }}    {{-- se muestra como texto: &lt;script&gt;alert(&quot;hackeado&quot;)&lt;/script&gt; --}}
{!! $comentario !!}  {{-- se inserta como HTML: el script SE EJECUTA en el navegador --}}
{{-- esto es un comentario de Blade: no aparece en el HTML final --}}
```

| Sintaxis | Escapa HTML | Cuándo usarla |
|---|---|---|
| `{{ $variable }}` | **Sí** | **Siempre por defecto**, sobre todo con datos ingresados por usuarios |
| `{!! $variable !!}` | No | Solo con HTML que **tú** generaste y es de confianza (por ejemplo, contenido de un editor ya filtrado) |
| `{{-- comentario --}}` | — | Notas que no deben llegar al navegador (a diferencia de `<!-- -->`, que sí se envía) |

**Estructuras de control:**

```blade
@if ($productos->isEmpty())
    <p>No hay productos.</p>
@elseif ($productos->count() === 1)
    <p>Hay un producto.</p>
@else
    <p>Hay {{ $productos->count() }} productos.</p>
@endif

@foreach ($productos as $producto)
    <li class="{{ $loop->first ? 'destacado' : '' }}">
        {{ $loop->iteration }}. {{ $producto }}
    </li>
@endforeach

@forelse ($pedidos as $pedido)           {{-- foreach + caso vacío en una sola estructura --}}
    <p>Pedido #{{ $pedido->id }}</p>
@empty
    <p>Aún no tienes pedidos.</p>
@endforelse

@isset($mensaje) <p>{{ $mensaje }}</p> @endisset
@auth  <a href="/panel">Mi panel</a> @endauth      {{-- solo si hay sesión iniciada (Unidad 3) --}}
@guest <a href="/login">Entrar</a>    @endguest
```

**La variable `$loop`** (disponible dentro de cualquier `@foreach`):

| Propiedad | Valor | Ejemplo con `['Teclado', 'Mouse', 'Monitor']` en la 2.ª vuelta |
|---|---|---|
| `$loop->index` | Índice desde 0 | `1` |
| `$loop->iteration` | Número de vuelta desde 1 | `2` |
| `$loop->count` | Total de elementos | `3` |
| `$loop->first` | ¿Es la primera vuelta? | `false` |
| `$loop->last` | ¿Es la última vuelta? | `false` |
| `$loop->remaining` | Vueltas que faltan | `1` |

**Herencia de plantillas — layouts:** casi todas las páginas de un sitio comparten encabezado, menú y pie. En vez de copiarlos en cada vista, se define un **layout** con "huecos" (`@yield`) que cada página **hija** rellena (`@section`):

```blade
{{-- resources/views/layouts/app.blade.php  (PADRE) --}}
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@yield('titulo', 'Mi Tienda')</title>   {{-- 2.º argumento: valor si la hija no lo define --}}
</head>
<body>
    <nav>Inicio | Productos | Contacto</nav>
    <main>
        @yield('contenido')
    </main>
    <footer>© 2026 Mi Tienda</footer>
</body>
</html>
```

```blade
{{-- resources/views/productos/index.blade.php  (HIJA) --}}
@extends('layouts.app')                {{-- debe ser lo primero del archivo --}}

@section('titulo', 'Productos')        {{-- forma corta para un valor simple --}}

@section('contenido')
    <h1>Catálogo</h1>
    <ul>
        @foreach ($productos as $producto)
            <li>{{ $producto }}</li>
        @endforeach
    </ul>
@endsection
```

```
        layouts/app.blade.php                    productos/index.blade.php
 ┌────────────────────────────────┐         ┌──────────────────────────────┐
 │ <title>@yield('titulo')</title>│ ◀────── │ @section('titulo','Productos')│
 │ <nav>…</nav>                   │         │                              │
 │ <main>@yield('contenido')</main>│ ◀───── │ @section('contenido') …       │
 │ <footer>…</footer>             │         │ @endsection                  │
 └────────────────────────────────┘         └──────────────────────────────┘
                     resultado: una sola página con el menú y pie del padre
                                y el título y contenido de la hija
```

> **Ojo con la notación de puntos:** `'layouts.app'` significa el archivo `resources/views/layouts/app.blade.php`. El punto sustituye a la barra de carpeta y **no** se escribe la extensión `.blade.php`.

**Reutilizar fragmentos — `@include` y componentes:**

```blade
{{-- @include: inserta otra vista; recibe las variables de la vista actual --}}
@include('partials.alerta', ['tipo' => 'exito', 'texto' => 'Guardado'])

{{-- Componente anónimo: resources/views/components/alerta.blade.php --}}
@props(['tipo' => 'info'])
<div class="alerta alerta-{{ $tipo }}">
    {{ $slot }}                         {{-- lo que se escriba entre las etiquetas del componente --}}
</div>

{{-- Uso del componente, con sintaxis tipo HTML --}}
<x-alerta tipo="error">El correo ya está registrado.</x-alerta>
{{-- Resultado: <div class="alerta alerta-error">El correo ya está registrado.</div> --}}
```

**Herencia vs inclusión vs componentes:**

| Mecanismo | Relación | Uso típico |
|---|---|---|
| `@extends` + `@section`/`@yield` | La hija **rellena** huecos de un esqueleto | Estructura general de la página (layout) |
| `@include` | Inserta una vista dentro de otra | Fragmentos simples: menú, pie, mensajes |
| Componentes `<x-...>` | Pieza reutilizable con **propiedades** (`@props`) y **contenido** (`$slot`) | Botones, tarjetas, alertas, campos de formulario; también se pueden usar como layout |

**Errores y malentendidos típicos:**

- **Escribir contenido fuera de `@section` en una vista hija:** todo lo que quede fuera de una sección en una vista que usa `@extends` se imprime **antes** del layout, rompiendo la página.
- **Usar `{!! !!}` "para que se vea bien el HTML" con datos del usuario:** abre la puerta a XSS. Si el contenido viene de un usuario, se imprime con `{{ }}`.
- **Hacer consultas a la base de datos dentro de la vista** (`{{ App\Models\Producto::count() }}`): funciona, pero rompe MVC (2.1.1) y suele causar consultas repetidas dentro de un `@foreach`.
- **Ver la vista "vieja" después de editarla:** normalmente Laravel detecta el cambio; si no, `php artisan view:clear` borra las vistas compiladas.

### 2.4.2. Paso de datos a las vistas

**Qué es:** pasar datos a una vista significa entregarle, desde el controlador (o la ruta), las **variables** que la plantilla va a mostrar. La vista solo puede usar lo que se le pasó explícitamente (más algunas variables globales como `$errors`).

**Por qué funciona así:** la vista no tiene acceso a las variables locales del controlador. Esto es intencional: obliga a que la comunicación entre controlador y vista sea **explícita**, y así se puede saber, leyendo solo el `return view(...)`, de qué depende la plantilla.

**Formas de pasar datos (todas producen el mismo resultado):**

```php
public function show()
{
    $nombre = 'Ana';
    $cursos = ['HTML', 'CSS', 'Laravel'];

    // 1. Arreglo asociativo: la CLAVE se convierte en el nombre de la variable
    return view('perfil', ['nombre' => $nombre, 'cursos' => $cursos]);

    // 2. compact(): crea el arreglo a partir de los nombres de las variables
    return view('perfil', compact('nombre', 'cursos'));   // equivale a ['nombre' => $nombre, 'cursos' => $cursos]

    // 3. with(): encadenado
    return view('perfil')->with('nombre', $nombre)->with('cursos', $cursos);
}
```

```blade
{{-- resources/views/perfil.blade.php --}}
<h1>{{ $nombre }}</h1>                         {{-- Ana --}}
<p>Inscrita en {{ count($cursos) }} cursos</p> {{-- Inscrita en 3 cursos --}}
```

**Cómo funciona por dentro — de arreglo a variables:** Laravel toma el arreglo asociativo y, antes de ejecutar la plantilla compilada, convierte cada clave en una variable local (equivalente a la función `extract()` de PHP):

| Lo que pasa el controlador | Variable en la vista | Valor |
|---|---|---|
| `'nombre' => $nombre` | `$nombre` | `"Ana"` |
| `'cursos' => $cursos` | `$cursos` | `["HTML", "CSS", "Laravel"]` |
| `'titulo' => 'Mi perfil'` | `$titulo` | `"Mi perfil"` |
| `'Nombre Completo' => '...'` | — | No es un nombre de variable válido: no se puede usar como `$Nombre Completo` |

> **Ojo con el nombre que importa:** lo que determina el nombre de la variable en la vista es la **clave** del arreglo, no el nombre de la variable en el controlador. `view('perfil', ['usuario' => $nombre])` hace que en la vista se use `$usuario`, y `$nombre` **no existe** allí.

**Datos compartidos con todas las vistas:** cuando un dato debe estar en todas las páginas (el nombre de la tienda, el número de artículos del carrito), repetirlo en cada `return view` sería tedioso:

```php
// app/Providers/AppServiceProvider.php
use Illuminate\Support\Facades\View;

public function boot(): void
{
    View::share('tienda', 'Mi Tienda');   // $tienda disponible en TODAS las vistas
}
```

**Datos que llegan a la vista sin pasarlos explícitamente:**

| Variable / función | Qué contiene | Detalle |
|---|---|---|
| `$errors` | Errores de validación (Unidad 3) | Siempre existe (vacía si no hay errores) |
| `session('clave')` | Un valor de la sesión, como mensajes *flash* (Unidad 3) | — |
| `auth()->user()` | El usuario con sesión iniciada (Unidad 3) | `null` si no hay sesión |
| `old('campo')` | El valor que el usuario escribió antes de que fallara la validación | Unidad 3 |
| Variables de `@include` | Las mismas que tenga la vista que lo incluye | Más las que se pasen en su segundo argumento |

**Paso de datos entre layout e hija:** el layout ve las mismas variables que la vista hija, porque ambos se renderizan con los datos que el controlador pasó a la vista hija.

**Traza completa — de la URL a la pantalla:**

```php
// routes/web.php
Route::get('/cursos/{alumno}', [CursoController::class, 'index']);

// CursoController
public function index(string $alumno)
{
    $cursos = ['HTML', 'CSS', 'Laravel'];
    return view('cursos', compact('alumno', 'cursos'));
}
```

```blade
{{-- cursos.blade.php --}}
<h1>Cursos de {{ ucfirst($alumno) }}</h1>
@foreach ($cursos as $curso)
    <p>{{ $loop->iteration }} de {{ $loop->count }}: {{ $curso }}</p>
@endforeach
```

| Paso | Dónde | Estado |
|---|---|---|
| 1 | Petición | `GET /cursos/ana` |
| 2 | Router | `{alumno}` = `"ana"` → llama a `CursoController@index("ana")` |
| 3 | Controlador | `$alumno = "ana"`, `$cursos = ["HTML","CSS","Laravel"]` |
| 4 | `compact` | `['alumno' => "ana", 'cursos' => [...]]` |
| 5 | Vista, `<h1>` | `ucfirst("ana")` → `<h1>Cursos de Ana</h1>` |
| 6 | Vista, vuelta 1 | `<p>1 de 3: HTML</p>` |
| 7 | Vista, vuelta 2 | `<p>2 de 3: CSS</p>` |
| 8 | Vista, vuelta 3 | `<p>3 de 3: Laravel</p>` |

**Errores y malentendidos típicos:**

- **"Undefined variable $productos":** la vista usa una variable que el controlador no le pasó, o se pasó con otra clave. Revisa el `return view(...)`.
- **Escribir `compact('$nombre')`:** `compact` recibe el **nombre** de la variable como texto y **sin** `$`. Con `$` no encuentra la variable.
- **Pasar demasiados datos "por si acaso"** o pasar objetos completos con información sensible: la vista puede terminar mostrando algo que no debía (por ejemplo, imprimir un modelo `User` entero como JSON incluye todos sus campos visibles). Pasa solo lo necesario.
- **Usar `View::share` para datos que dependen del usuario** en `boot()`: en ese momento puede no haberse cargado aún la sesión. Para datos dinámicos se usan **View Composers** o se pasan desde el controlador.

**Cuándo usar cada forma:**

| Forma | Ventaja | Úsala cuando… |
|---|---|---|
| Arreglo `['clave' => $valor]` | Permite renombrar | La variable de la vista debe llamarse distinto que en el controlador |
| `compact('a', 'b')` | Breve | Los nombres coinciden (lo más habitual) |
| `->with('a', $a)` | Legible al encadenar | Agregas datos de forma condicional |
| `View::share` | Global | El dato es igual en todas las páginas |

---

*Universidad Politécnica de Tecamac — Ingeniería en Software*
