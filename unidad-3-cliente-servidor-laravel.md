# Unidad 3 — Programación cliente-servidor con Laravel

En la Unidad 2 el flujo iba en una sola dirección: el navegador pide una página y el servidor la genera. En esta unidad el usuario empieza a **enviar información** al servidor (formularios, archivos) y el servidor necesita **recordar** quién es el usuario entre una petición y otra (sesiones, cookies, autenticación). Aquí se retoman directamente los conceptos de HTTP de la Unidad 1 (métodos, códigos de estado, "sin estado", cookies) y se ve cómo Laravel los resuelve.

---

## Índice

1. [Métodos HTTP (GET y POST)](#31-métodos-http-get-y-post)
   - 3.1.1. [Diferencias e implementación en Laravel](#311-diferencias-e-implementación-en-laravel)
2. [Formularios y validación de datos](#32-formularios-y-validación-de-datos)
   - 3.2.1. [Creación de formularios con Blade](#321-creación-de-formularios-con-blade)
   - 3.2.2. [Validación en controladores y mensajes de error](#322-validación-en-controladores-y-mensajes-de-error)
3. [Middleware y autenticación](#33-middleware-y-autenticación)
   - 3.3.1. [Creación de middleware](#331-creación-de-middleware)
   - 3.3.2. [Protección de rutas con autenticación](#332-protección-de-rutas-con-autenticación)
4. [Sesiones y cookies](#34-sesiones-y-cookies)
   - 3.4.1. [Manejo de sesiones](#341-manejo-de-sesiones)
   - 3.4.2. [Lectura y escritura de cookies](#342-lectura-y-escritura-de-cookies)
5. [Subida y descarga de archivos](#35-subida-y-descarga-de-archivos)
   - 3.5.1. [Validación, almacenamiento y descarga de archivos](#351-validación-almacenamiento-y-descarga-de-archivos)

---

## 3.1. Métodos HTTP (GET y POST)

### 3.1.1. Diferencias e implementación en Laravel

**Qué es:** el **método HTTP** es la primera palabra de una petición (`GET /productos HTTP/1.1`) e indica la **intención** del cliente. **GET** significa "dame este recurso"; **POST** significa "aquí te envío datos para que los proceses" (normalmente para crear algo o ejecutar una acción).

**Por qué existen dos métodos distintos:** el servidor, el navegador y los intermediarios (cachés, *proxies*, buscadores) necesitan saber si una petición es **inofensiva** o si **cambia algo**. Un buscador como Google sigue todos los enlaces (`GET`) que encuentra; si "eliminar producto" fuera un enlace GET, el robot del buscador borraría los productos al indexar el sitio. Por eso la regla es: **GET solo lee; todo lo que modifique datos va por POST** (o PUT/PATCH/DELETE).

**Cómo funciona por dentro — dónde viajan los datos:**

```
GET — los datos van en la URL (query string)
┌──────────────────────────────────────────────────────────────┐
│ GET /buscar?q=teclado&orden=precio HTTP/1.1                  │
│ Host: tienda.com                                             │
│                                         (sin cuerpo)         │
└──────────────────────────────────────────────────────────────┘

POST — los datos van en el CUERPO de la petición
┌──────────────────────────────────────────────────────────────┐
│ POST /productos HTTP/1.1                                     │
│ Host: tienda.com                                             │
│ Content-Type: application/x-www-form-urlencoded              │
│                                                              │
│ nombre=Teclado&precio=350&_token=Xy9k...                     │
└──────────────────────────────────────────────────────────────┘
```

**Comparación:**

| Característica | GET | POST |
|---|---|---|
| Intención | Obtener / consultar | Enviar datos / crear / ejecutar una acción |
| Dónde van los datos | En la URL (`?clave=valor&...`) | En el cuerpo de la petición |
| Visible en la barra de direcciones e historial | Sí | No |
| Se puede guardar como marcador o compartir | Sí | No |
| Lo guarda la caché del navegador | Sí, puede | No por defecto |
| Límite de tamaño | Limitado por la longitud máxima de URL (varios miles de caracteres, depende del navegador y el servidor) | Mucho mayor (en PHP lo limita `post_max_size`, 8 MB por defecto) |
| Puede enviar archivos | No | Sí (con `multipart/form-data`) |
| Seguro / idempotente (Unidad 1) | Sí / Sí | No / No |
| Al recargar (F5) | Se repite sin aviso | El navegador pregunta "¿reenviar formulario?" |
| En Laravel requiere token CSRF | No | **Sí** |

> **Ojo con "POST es seguro porque oculta los datos":** POST solo evita que los datos aparezcan en la URL. Sin HTTPS, el cuerpo viaja tan legible como la URL. Y cualquier usuario puede ver lo que envía su navegador en las herramientas de desarrollador (F12 → Red).

**Implementación en Laravel — rutas y lectura de datos:**

```php
// routes/web.php
Route::get('/buscar', [BusquedaController::class, 'index']);        // ?q=...
Route::post('/productos', [ProductoController::class, 'store']);    // formulario

// Atender varios métodos con una sola ruta (poco común)
Route::match(['get', 'post'], '/contacto', [ContactoController::class, 'manejar']);
```

```php
use Illuminate\Http\Request;

public function index(Request $request)
{
    // GET /buscar?q=teclado&orden=precio
    $q     = $request->query('q');            // "teclado"  ← solo de la query string
    $orden = $request->input('orden', 'id');  // "precio"   ← input() lee de query string Y del cuerpo
    $pag   = $request->input('pagina', 1);    // 1          ← no vino: se usa el valor por defecto
    $request->has('q');                       // true
    $request->filled('pagina');               // false (no existe o está vacío)
    $request->method();                       // "GET"
    $request->isMethod('post');               // false
}

public function store(Request $request)
{
    // POST con cuerpo: nombre=Teclado&precio=350
    $nombre = $request->input('nombre');               // "Teclado"
    $todo   = $request->all();                         // ['_token' => '...', 'nombre' => 'Teclado', 'precio' => '350']
    $algo   = $request->only(['nombre', 'precio']);    // ['nombre' => 'Teclado', 'precio' => '350']
    $sin    = $request->except(['_token']);            // todo menos el token
}
```

> **Ojo con los tipos:** todo lo que llega por HTTP es **texto**. `$request->input('precio')` devuelve el string `"350"`, no el número `350`. Laravel ofrece conversiones explícitas: `$request->integer('precio')` → `350`, `$request->boolean('activo')` → `true`/`false` (acepta `"1"`, `"true"`, `"on"`, `"yes"`), `$request->float('precio')`.

**Métodos PUT, PATCH y DELETE desde un formulario — *method spoofing*:** los formularios HTML **solo** admiten `GET` y `POST` en el atributo `method`. Para usar las rutas `update` (PUT/PATCH) y `destroy` (DELETE) de un controlador de recurso (Unidad 2), el formulario se envía como POST con un campo oculto `_method` que Laravel interpreta:

```blade
<form action="{{ route('productos.destroy', $producto) }}" method="POST">
    @csrf
    @method('DELETE')     {{-- genera: <input type="hidden" name="_method" value="DELETE"> --}}
    <button type="submit">Eliminar</button>
</form>
```

```
Navegador envía:  POST /productos/5   cuerpo: _token=...&_method=DELETE
Laravel ve:       DELETE /productos/5 → ProductoController@destroy
```

**Patrón PRG (Post/Redirect/Get) — por qué un POST nunca debe devolver una vista directamente:**

```
SIN PRG                                         CON PRG
POST /productos ──▶ 200 + HTML "Guardado"        POST /productos ──▶ 302 Location: /productos
Usuario presiona F5:                             GET  /productos ──▶ 200 + HTML con la lista
  el navegador REPITE el POST                    Usuario presiona F5:
  → se crea un segundo producto duplicado          se repite solo el GET → no se duplica nada
```

En Laravel, toda acción que procesa un POST termina con `return redirect()->route(...)` o `return back()`, nunca con `return view(...)`.

**Árbol de decisión — ¿qué método usar?**

```
¿La acción modifica datos en el servidor (crear, editar, borrar, enviar un correo, pagar)?
│
├─ No ──▶ GET   (búsquedas, filtros, paginación, ver un detalle)
│          └─ ventaja: el resultado se puede compartir con un enlace
│
└─ Sí ──▶ ¿Qué modifica?
          ├─ Crea algo nuevo o ejecuta una acción ──▶ POST
          ├─ Reemplaza / edita un recurso ─────────▶ PUT / PATCH  (POST + @method('PUT'))
          └─ Elimina un recurso ───────────────────▶ DELETE       (POST + @method('DELETE'))
```

**Errores y malentendidos típicos:**

- **Enlace `<a href="/productos/5/eliminar">` para borrar:** es un GET que modifica datos; cualquier robot, extensión que precargue enlaces o un simple clic accidental desde el historial lo ejecuta. Borrar siempre con un formulario POST + `@method('DELETE')`.
- **Enviar contraseñas por GET:** quedan en el historial del navegador, en los registros (*logs*) del servidor y en la cabecera `Referer` que se envía a otros sitios.
- **Error 405 "Method Not Allowed":** la URL existe, pero no para ese método (por ejemplo, el formulario hace POST y solo hay `Route::get`, o falta `@method('PUT')`).

---

## 3.2. Formularios y validación de datos

### 3.2.1. Creación de formularios con Blade

**Qué es:** un **formulario** HTML (`<form>`) es el mecanismo del navegador para reunir datos del usuario y enviarlos al servidor en una petición HTTP. En Laravel se escribe en una vista Blade, que agrega la protección CSRF, la recuperación de valores y la muestra de errores.

**Anatomía de un formulario completo:**

```blade
{{-- resources/views/productos/create.blade.php --}}
<form action="{{ route('productos.store') }}" method="POST">
    {{-- action: a qué URL se envía · method: GET o POST --}}
    @csrf
    {{-- genera: <input type="hidden" name="_token" value="Xy9k..."> --}}

    <label for="nombre">Nombre</label>
    <input type="text" id="nombre" name="nombre" value="{{ old('nombre') }}">
    {{-- name: la CLAVE con la que llega el dato al servidor · old(): valor anterior si falló la validación --}}
    @error('nombre')
        <p class="error">{{ $message }}</p>
    @enderror

    <label for="categoria">Categoría</label>
    <select id="categoria" name="categoria">
        <option value="perifericos" @selected(old('categoria') === 'perifericos')>Periféricos</option>
        <option value="monitores"   @selected(old('categoria') === 'monitores')>Monitores</option>
    </select>

    <label>
        <input type="checkbox" name="activo" value="1" @checked(old('activo'))> Activo
    </label>

    <button type="submit">Guardar</button>
</form>
```

**Cómo funciona por dentro — qué se envía exactamente:** al presionar el botón, el navegador recorre los controles del formulario y arma pares `name=valor`:

| Control | ¿Se envía? | Ejemplo en el cuerpo |
|---|---|---|
| `<input name="nombre">` con "Teclado" | Sí | `nombre=Teclado` |
| `<select name="categoria">` | Sí, el `value` de la opción elegida | `categoria=perifericos` |
| `<input type="checkbox" name="activo">` **marcado** | Sí, su `value` | `activo=1` |
| El mismo checkbox **sin marcar** | **No se envía nada** | — |
| Un control **sin** atributo `name` | **No** | — |
| Un control con atributo `disabled` | **No** | — |
| `<input type="hidden" name="_token">` | Sí | `_token=Xy9k...` |

> **Ojo con los checkbox desmarcados:** como no se envían, en el servidor el campo simplemente **no existe**. Por eso se lee con `$request->boolean('activo')`, que devuelve `false` si falta, en lugar de `$request->input('activo')`, que devuelve `null`.

**CSRF — por qué `@csrf` es obligatorio:** un ataque **CSRF** (*Cross-Site Request Forgery*, falsificación de petición entre sitios) aprovecha que el navegador envía **automáticamente** las cookies de un sitio en cada petición hacia él, aunque la petición la haya iniciado otra página:

```
1. Ana inicia sesión en tienda.com  → su navegador guarda la cookie de sesión
2. Ana visita sitio-malicioso.com, que contiene:
     <form action="https://tienda.com/cuenta/eliminar" method="POST"> … </form>
     <script>document.forms[0].submit()</script>
3. El navegador envía el POST a tienda.com JUNTO con la cookie de Ana
4. Sin protección: tienda.com cree que Ana pidió eliminar su cuenta
```

**La defensa:** Laravel guarda un **token aleatorio** en la sesión de cada usuario y exige que todo POST/PUT/PATCH/DELETE incluya ese mismo token en el campo `_token`. El sitio malicioso puede hacer que el navegador envíe la cookie, pero **no puede leer** el token (las políticas del navegador le impiden leer páginas de otro dominio), así que su petición llega sin token válido y Laravel responde **`419 Page Expired`**.

| Situación | Resultado |
|---|---|
| Formulario propio con `@csrf` | El token coincide con el de la sesión → se procesa |
| Formulario propio **sin** `@csrf` | Falta el token → `419` |
| Formulario abierto mucho tiempo (la sesión expiró) | El token ya no corresponde a ninguna sesión → `419` |
| Formulario desde otro sitio | No conoce el token → `419` |

**Directivas de formulario en Blade:**

| Directiva | Genera | Uso |
|---|---|---|
| `@csrf` | `<input type="hidden" name="_token" value="...">` | En **todo** formulario que no sea GET |
| `@method('PUT')` | `<input type="hidden" name="_method" value="PUT">` | Formularios de edición / eliminación |
| `old('campo', $valorPorDefecto)` | El valor enviado anteriormente | Para no obligar al usuario a reescribir todo tras un error |
| `@error('campo') … @enderror` | El bloque solo si hay error en ese campo, con `$message` | Mensaje junto al campo |
| `@checked(cond)`, `@selected(cond)`, `@disabled(cond)` | El atributo solo si la condición es verdadera | Mantener la selección del usuario |

**Formulario de edición — `old()` con valor por defecto:**

```blade
<form action="{{ route('productos.update', $producto) }}" method="POST">
    @csrf
    @method('PUT')
    <input name="nombre" value="{{ old('nombre', $producto->nombre) }}">
    {{-- 1.ª vez: muestra el valor guardado · tras un error: lo que escribió el usuario --}}
</form>
```

**Errores y malentendidos típicos:**

- **El campo llega como `null`:** casi siempre falta el atributo `name` o tiene otro nombre del que se lee en el controlador. `id` sirve para `<label for>` y para CSS/JS; el servidor solo ve `name`.
- **Error 419 al enviar:** falta `@csrf`, o la sesión expiró (el formulario estuvo abierto más tiempo que `SESSION_LIFETIME`, 120 minutos por defecto).
- **Confiar en la validación de HTML** (`required`, `type="email"`, `maxlength`): mejora la experiencia, pero se puede quitar desde las herramientas de desarrollador en dos segundos. El servidor **siempre** debe validar (3.2.2).
- **Usar `method="PUT"` en el `<form>`:** el navegador no lo reconoce y envía GET. Se usa `method="POST"` + `@method('PUT')`.

### 3.2.2. Validación en controladores y mensajes de error

**Qué es:** **validar** es comprobar en el servidor que los datos recibidos cumplen las reglas esperadas (que existan, que tengan el tipo, formato y tamaño correctos, que no estén duplicados) **antes** de usarlos o guardarlos.

**Por qué existe:** el servidor no puede confiar en nada de lo que llega del cliente. Cualquier persona puede construir una petición a mano con herramientas como `curl` o Postman, saltándose el formulario y cualquier JavaScript. Sin validación en el servidor llegan a la base de datos precios negativos, correos inválidos, textos de un millón de caracteres o campos vacíos que rompen la aplicación más adelante.

**Validación básica en el controlador:**

```php
public function store(Request $request)
{
    $datos = $request->validate([
        'nombre'    => 'required|string|max:100',
        'email'     => ['required', 'email', 'unique:clientes,email'],   // reglas también como arreglo
        'edad'      => 'nullable|integer|between:18,99',
        'precio'    => 'required|numeric|min:0',
        'password'  => 'required|string|min:8|confirmed',                // exige password_confirmation igual
        'categoria' => 'required|in:perifericos,monitores',
    ]);

    // Si llegamos aquí, TODO es válido. $datos contiene SOLO los campos validados.
    Cliente::create($datos);   // Unidad 4
    return redirect()->route('clientes.index')->with('exito', 'Cliente registrado');
}
```

**Cómo funciona por dentro — qué pasa si falla:**

```
POST /clientes con email="ana@" y edad="15"
        │
        ▼
$request->validate([...])
        │ evalúa cada campo con sus reglas
        │ ✗ email: no es un correo válido
        │ ✗ edad: debe estar entre 18 y 99
        ▼
lanza ValidationException  ── el resto del método store() NO se ejecuta
        │
        ├─ Petición normal de formulario:
        │     1. guarda los errores en la sesión (flash)
        │     2. guarda lo que escribió el usuario (old input), excepto contraseñas
        │     3. responde 302 → regresa a la página del formulario
        │     4. la vista lee $errors y old('campo')
        │
        └─ Petición que espera JSON (Accept: application/json, p. ej. fetch o API):
              responde 422 con {"message": "...", "errors": {"email": ["..."], "edad": ["..."]}}
```

**Traza — evaluación de reglas para `edad`:**

| Valor enviado | `nullable` | `integer` | `between:18,99` | Resultado |
|---|---|---|---|---|
| *(vacío)* | Permite vacío → no se evalúan las demás reglas | — | — | Válido; `$datos['edad'] = null` |
| `"25"` | — | Sí (texto numérico entero) | 18 ≤ 25 ≤ 99 | Válido |
| `"15"` | — | Sí | 15 < 18 | **Error** en `between` |
| `"veinte"` | — | **No** | Se evalúa también, pero ya hay error | **Error** en `integer` |
| `"25.5"` | — | **No** | — | **Error** en `integer` |

> **Ojo con `between`, `min`, `max` y `size`:** su significado **depende del tipo** del campo. Con `integer`/`numeric` comparan el **valor numérico**; con `string` comparan el **número de caracteres**; con arreglos, el **número de elementos**; con archivos, el **tamaño en kilobytes**. Por eso `'edad' => 'max:99'` sin `integer` validaría que el texto tenga como máximo 99 caracteres, no que la edad sea menor de 100. Para detener la evaluación en el primer error de un campo se agrega `bail` al inicio: `'edad' => 'bail|integer|between:18,99'`.

**Reglas más usadas:**

| Regla | Comprueba | Ejemplo |
|---|---|---|
| `required` | Que exista y no esté vacío | `'nombre' => 'required'` |
| `nullable` | Que pueda venir vacío (si viene, se aplican las demás reglas) | `'telefono' => 'nullable|digits:10'` |
| `string`, `integer`, `numeric`, `boolean`, `array` | Tipo | `'stock' => 'integer'` |
| `email` | Formato de correo | `'email' => 'email'` |
| `min:n`, `max:n`, `between:a,b`, `size:n` | Tamaño/valor (según el tipo) | `'nombre' => 'max:100'` |
| `in:a,b,c` | Que sea uno de los valores permitidos | `'rol' => 'in:admin,cliente'` |
| `confirmed` | Que exista `campo_confirmation` con el mismo valor | Contraseñas |
| `unique:tabla,columna` | Que no exista ya en la base de datos | `'email' => 'unique:users,email'` |
| `exists:tabla,columna` | Que **sí** exista en la base de datos | `'categoria_id' => 'exists:categorias,id'` |
| `date`, `after:today`, `before:fecha` | Fechas | `'entrega' => 'date|after:today'` |
| `regex:/patrón/` | Expresión regular | `'curp' => 'regex:/^[A-Z]{4}\d{6}/'` |

**Mostrar los errores en la vista:** la variable `$errors` existe **siempre** en todas las vistas (vacía si no hubo errores), gracias a un middleware que la comparte automáticamente.

```blade
{{-- Lista general arriba del formulario --}}
@if ($errors->any())
    <div class="alerta-error">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

{{-- Junto a cada campo --}}
<input name="email" value="{{ old('email') }}" class="@error('email') campo-invalido @enderror">
@error('email') <small>{{ $message }}</small> @enderror

{{-- Mensaje de éxito enviado con ->with('exito', ...) --}}
@if (session('exito')) <div class="alerta-ok">{{ session('exito') }}</div> @endif
```

**Mensajes de error personalizados:**

```php
$request->validate(
    [
        'nombre' => 'required|max:100',
        'email'  => 'required|email',
    ],
    [   // mensajes: 'campo.regla' => 'texto'
        'nombre.required' => 'Escribe el nombre del cliente.',
        'email.email'     => 'El correo no tiene un formato válido.',
        'max'             => 'El campo :attribute no puede superar :max caracteres.', // para cualquier campo
    ],
    [   // nombres "bonitos" de los campos para :attribute
        'email' => 'correo electrónico',
    ]
);
```

> **Por qué los mensajes salen en inglés:** Laravel trae los textos en inglés. Para tenerlos en español se configura `APP_LOCALE=es` y se agregan los archivos de traducción en `lang/es/validation.php` (se publican con `php artisan lang:publish` y se traducen, o se instala un paquete de traducciones de la comunidad).

**Form Requests — validación fuera del controlador:**

```bash
php artisan make:request GuardarClienteRequest     # app/Http/Requests/GuardarClienteRequest.php
```

```php
class GuardarClienteRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;   // ¿el usuario tiene permiso de hacer esto? false → 403
    }

    public function rules(): array
    {
        return [
            'nombre' => 'required|string|max:100',
            'email'  => 'required|email|unique:clientes,email',
        ];
    }

    public function messages(): array
    {
        return ['email.unique' => 'Ese correo ya está registrado.'];
    }
}

// En el controlador: basta con cambiar el tipo del parámetro
public function store(GuardarClienteRequest $request)
{
    $datos = $request->validated();   // la validación YA ocurrió antes de entrar al método
    // ...
}
```

**Validación en el controlador vs Form Request:**

| Aspecto | `$request->validate()` | Form Request |
|---|---|---|
| Dónde están las reglas | Dentro del método del controlador | En una clase propia |
| Reutilización (crear y editar con reglas parecidas) | Hay que copiar | Una clase que se reutiliza o extiende |
| Autorización incluida | No | Sí (`authorize()`) |
| Controlador | Más largo | Más delgado ("controladores delgados", Unidad 2) |
| Cuándo usarlo | Formularios pequeños y únicos | Formularios con muchas reglas o reglas compartidas |

**Errores y malentendidos típicos:**

- **Guardar `$request->all()` en lugar de los datos validados:** `all()` incluye cualquier campo extra que el usuario haya agregado a mano (por ejemplo, `es_admin=1`). Usa el arreglo que devuelve `validate()` o `validated()`.
- **Olvidar `nullable` en un campo opcional:** `'telefono' => 'digits:10'` hace que un teléfono vacío **falle**, porque una cadena vacía no tiene 10 dígitos.
- **`unique` al editar:** al actualizar un cliente sin cambiar su correo, `unique:clientes,email` falla porque el correo ya existe (es el suyo). Hay que excluir su propio registro: `Rule::unique('clientes', 'email')->ignore($cliente->id)`.
- **No ver los errores:** la vista no tiene `@error`/`$errors`, o el formulario redirige a otra página distinta de la que lo mostró.

---

## 3.3. Middleware y autenticación

### 3.3.1. Creación de middleware

**Qué es:** un **middleware** es una capa de código que se ejecuta **antes** (y/o **después**) de que la petición llegue al controlador. Cada middleware recibe la petición, decide si la deja **pasar** a la siguiente capa o la **detiene** (respondiendo él mismo, por ejemplo con una redirección o un error 403), y puede modificar la respuesta de regreso.

**Por qué existe:** muchas comprobaciones se repiten en decenas de rutas: "¿inició sesión?", "¿es administrador?", "¿el sitio está en mantenimiento?", "¿el token CSRF es válido?". Si cada controlador las hiciera por su cuenta, el código se duplicaría y bastaría con olvidarla en un solo método para abrir un hueco de seguridad. El middleware centraliza esa lógica y se **asigna** a rutas o grupos de rutas.

**Cómo funciona por dentro — el modelo de cebolla:** los middleware se encadenan como capas. La petición atraviesa cada capa hacia adentro hasta el controlador, y la respuesta las atraviesa en **orden inverso** hacia afuera:

```
 Petición ──▶ ┌──────────────── Middleware A (cookies cifradas) ─────────────────┐
              │  antes de A  ┌──────────── Middleware B (sesión) ───────────┐    │
              │              │  antes de B ┌──── Middleware C (auth) ───┐   │    │
              │              │             │                            │   │    │
              │              │             │       CONTROLADOR          │   │    │
              │              │             │                            │   │    │
              │              │  después B  └────────────────────────────┘   │    │
              │  después A   └──────────────────────────────────────────────┘    │
 Respuesta ◀──└──────────────────────────────────────────────────────────────────┘

 Si C decide detener la petición (no hay sesión), responde una redirección;
 el controlador NUNCA se ejecuta, pero la respuesta sigue saliendo a través de B y A.
```

**Crear un middleware:**

```bash
php artisan make:middleware VerificarEdad      # app/Http/Middleware/VerificarEdad.php
```

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class VerificarEdad
{
    public function handle(Request $request, Closure $next): Response
    {
        // --- ANTES: se ejecuta antes del controlador ---
        if ($request->integer('edad') < 18) {
            return redirect('/')->with('error', 'Debes ser mayor de edad.');  // se DETIENE aquí
        }

        $respuesta = $next($request);   // deja pasar a la siguiente capa (y al final, al controlador)

        // --- DESPUÉS: se ejecuta cuando el controlador ya generó la respuesta ---
        $respuesta->headers->set('X-Verificado', 'si');

        return $respuesta;
    }
}
```

**Traza — `GET /contenido?edad=X`:**

| `edad` | Condición `< 18` | ¿Se llama `$next`? | ¿Se ejecuta el controlador? | Respuesta |
|---|---|---|---|---|
| `20` | No | Sí | Sí | `200` con cabecera `X-Verificado: si` |
| `15` | Sí | **No** | **No** | `302` a `/` con mensaje de error |
| *(ausente)* | `integer()` devuelve `0` → Sí | No | No | `302` a `/` |

> **Ojo con olvidar `return $next($request)`:** si el método no devuelve el resultado de `$next`, la petición **nunca llega** al controlador y el usuario recibe una página en blanco o un error de tipo de retorno.

**Registrar y asignar el middleware (Laravel 11+):**

```php
// bootstrap/app.php
->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'edad' => \App\Http\Middleware\VerificarEdad::class,   // nombre corto
    ]);
    // $middleware->append(VerificarEdad::class);   // global: TODAS las peticiones
    // $middleware->web(append: [VerificarEdad::class]);  // todas las rutas del grupo web
})
```

```php
// routes/web.php
Route::get('/contenido', [ContenidoController::class, 'index'])->middleware('edad');

Route::middleware(['auth', 'edad'])->group(function () {   // varios, en orden
    Route::get('/peliculas', ...);
    Route::get('/series', ...);
});

// También se puede usar la clase directamente, sin alias
Route::get('/otro', ...)->middleware(VerificarEdad::class);
```

> En Laravel 10 y anteriores el registro se hacía en `app/Http/Kernel.php`, en los arreglos `$middleware`, `$middlewareGroups` y `$middlewareAliases` (`$routeMiddleware` en versiones más antiguas).

**Middleware con parámetros:**

```php
public function handle(Request $request, Closure $next, string $rol): Response
{
    if ($request->user()?->rol !== $rol) {
        abort(403);                              // 403 Forbidden
    }
    return $next($request);
}

// Uso: los parámetros van después de ":" separados por comas
Route::get('/admin', ...)->middleware('rol:admin');
```

**Middleware que Laravel ya trae en el grupo `web`:**

| Middleware | Qué hace |
|---|---|
| `EncryptCookies` | Cifra/descifra las cookies (3.4.2) |
| `StartSession` | Carga la sesión del usuario a partir de su cookie (3.4.1) |
| `ShareErrorsFromSession` | Pone `$errors` a disposición de todas las vistas (3.2.2) |
| `ValidateCsrfToken` | Rechaza con 419 los POST/PUT/PATCH/DELETE sin token válido (3.2.1) |
| `SubstituteBindings` | Realiza el *route model binding* (Unidad 2) |

**Tipos de asignación:**

| Tipo | Se aplica a | Ejemplo de uso |
|---|---|---|
| Global | Todas las peticiones | Modo mantenimiento, recortar espacios de los campos |
| De grupo (`web`, `api`) | Todas las rutas de ese grupo | Sesión, CSRF |
| De ruta (alias) | Solo las rutas que lo declaran | `auth`, `rol:admin` |

**Errores y malentendidos típicos:**

- **"Target class [edad] does not exist":** se usó el alias sin registrarlo en `bootstrap/app.php`.
- **El orden importa:** un middleware que lee `$request->user()` debe ejecutarse **después** de `auth`; si va antes, todavía no hay usuario identificado.
- **Poner lógica de negocio en un middleware:** el middleware filtra y prepara peticiones; calcular un descuento o guardar un pedido corresponde al controlador o al modelo.

### 3.3.2. Protección de rutas con autenticación

**Qué es:** la **autenticación** es el proceso de comprobar **quién es** el usuario (normalmente con correo y contraseña). La **autorización** es decidir **qué puede hacer** una vez identificado. Proteger una ruta con autenticación significa que solo los usuarios que iniciaron sesión pueden acceder a ella.

| Concepto | Pregunta que responde | Error si falla | En Laravel |
|---|---|---|---|
| **Autenticación** | ¿Quién eres? | `401` / redirección al login | `Auth::attempt()`, middleware `auth` |
| **Autorización** | ¿Tienes permiso para esto? | `403` | Middleware de rol, *Gates*, *Policies* |

**Por qué existe:** HTTP no tiene estado (Unidad 1): el servidor no sabe por sí mismo que la petición a `/panel` viene de la misma persona que escribió su contraseña hace un momento. La autenticación de Laravel resuelve esto combinando la **sesión** (3.4.1) con el **middleware `auth`**: tras verificar la contraseña, guarda el id del usuario en la sesión, y en cada petición posterior lo recupera de ahí.

**Cómo funciona por dentro — el ciclo completo:**

```
1. LOGIN
   POST /login  (email, password)
      │
      ├─ Auth::attempt(): busca el usuario por email en la tabla users
      ├─ Hash::check(password, $usuario->password)   ← compara contra el HASH guardado
      ├─ si coincide: guarda el id del usuario en la sesión
      ├─ session()->regenerate(): genera un id de sesión NUEVO
      └─ 302 → /panel   (Set-Cookie: laravel_session=<id nuevo>)

2. PETICIONES SIGUIENTES
   GET /panel   (Cookie: laravel_session=<id>)
      │
      ├─ StartSession: carga la sesión → contiene el id del usuario
      ├─ middleware auth: ¿hay usuario en la sesión?
      │     ├─ sí → continúa al controlador; auth()->user() devuelve el modelo User
      │     └─ no → 302 → /login
      └─ controlador

3. LOGOUT
   POST /logout → Auth::logout() + invalidar la sesión + regenerar el token CSRF
```

**Contraseñas — por qué se guarda un hash y no la contraseña:** un **hash** es el resultado de una función **de un solo sentido**: a partir de la contraseña se calcula fácilmente, pero a partir del hash no se puede recuperar la contraseña. Laravel usa **bcrypt** por defecto, que además agrega una **sal** (*salt*) aleatoria, de modo que dos usuarios con la misma contraseña tienen hashes distintos.

```php
use Illuminate\Support\Facades\Hash;

$hash = Hash::make('secreta123');   // "$2y$12$Kd8...(60 caracteres)" — distinto cada vez que se ejecuta
Hash::check('secreta123', $hash);   // true
Hash::check('otra', $hash);         // false
```

> **Ojo:** como el hash cambia en cada ejecución por la sal, **no** se puede comparar con `==` (`Hash::make('x') == Hash::make('x')` es `false`). Siempre se usa `Hash::check`. Si la base de datos se filtra, los atacantes obtienen hashes, no contraseñas, y bcrypt es deliberadamente lento para que probar millones de contraseñas sea muy costoso.

**Implementación manual de login y logout:**

```php
use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    public function mostrar()
    {
        return view('auth.login');
    }

    public function entrar(Request $request)
    {
        $credenciales = $request->validate([
            'email'    => 'required|email',
            'password' => 'required',
        ]);

        if (Auth::attempt($credenciales, $request->boolean('recordarme'))) {
            $request->session()->regenerate();          // previene la fijación de sesión
            return redirect()->intended('/panel');      // vuelve a la página que quería visitar
        }

        return back()
            ->withErrors(['email' => 'Las credenciales no coinciden.'])
            ->onlyInput('email');                       // conserva el email, NO la contraseña
    }

    public function salir(Request $request)
    {
        Auth::logout();
        $request->session()->invalidate();              // destruye los datos de la sesión
        $request->session()->regenerateToken();         // nuevo token CSRF
        return redirect('/');
    }
}
```

```php
// routes/web.php
Route::middleware('guest')->group(function () {        // solo para quien NO ha iniciado sesión
    Route::get('/login', [LoginController::class, 'mostrar'])->name('login');
    Route::post('/login', [LoginController::class, 'entrar']);
});

Route::middleware('auth')->group(function () {         // solo con sesión iniciada
    Route::get('/panel', [PanelController::class, 'index']);
    Route::post('/logout', [LoginController::class, 'salir'])->name('logout');
});
```

> **Por qué la ruta de login se llama `login`:** cuando el middleware `auth` encuentra a un visitante sin sesión, lo redirige a la ruta con nombre **`login`**. Si no existe ninguna ruta con ese nombre, aparece el error *"Route [login] not defined"*.

**Traza — `redirect()->intended()`:**

| Paso | Petición | Estado de la sesión | Respuesta |
|---|---|---|---|
| 1 | `GET /panel/pedidos` sin sesión | Laravel guarda `url.intended = /panel/pedidos` | `302` → `/login` |
| 2 | `GET /login` | — | Formulario |
| 3 | `POST /login` correcto | Se guarda el usuario; se regenera el id de sesión | `302` → `/panel/pedidos` (la URL guardada, no `/panel`) |
| 4 | `GET /panel/pedidos` | Usuario identificado | `200` |

**Acceso al usuario autenticado:**

```php
Auth::check();          // true si hay sesión iniciada
Auth::id();             // 7   (id del usuario)
Auth::user()->name;     // "Ana López"
auth()->user();         // equivalente con el helper
$request->user();       // equivalente desde la petición
```

```blade
@auth
    Hola, {{ auth()->user()->name }}
    <form action="{{ route('logout') }}" method="POST">@csrf <button>Salir</button></form>
@endauth
@guest
    <a href="{{ route('login') }}">Iniciar sesión</a>
@endguest
```

**Opciones para implementar autenticación:**

| Opción | Qué incluye | Cuándo usarla |
|---|---|---|
| Manual (`Auth::attempt`) | Lo que tú programes | Para aprender el mecanismo o requisitos muy específicos |
| *Starter kits* oficiales (se eligen al crear el proyecto con `laravel new`) | Login, registro, recuperación de contraseña, verificación de correo y vistas listas | Proyectos reales: código revisado y probado |
| **Sanctum** | Tokens para APIs y aplicaciones SPA/móviles | Cuando el cliente no es una vista Blade |

**Errores y malentendidos típicos:**

- **Guardar contraseñas en texto plano o con `md5`:** `Auth::attempt` compara con `Hash::check`; si la contraseña se guardó sin `Hash::make`, el login nunca funciona. Y `md5` es demasiado rápido para considerarse seguro.
- **Proteger solo el enlace y no la ruta:** ocultar el botón "Panel" con `@auth` no impide que alguien escriba `/panel` en la barra de direcciones. La protección real es el middleware `auth` en la ruta.
- **Olvidar `session()->regenerate()`:** permite un ataque de **fijación de sesión**: si el atacante logró que la víctima usara un id de sesión conocido, ese id quedaría autenticado tras el login.
- **Hacer logout con un enlace GET:** cualquier página externa podría cerrar la sesión del usuario con una simple imagen `<img src="/logout">`. El logout va por POST con `@csrf`.

---

## 3.4. Sesiones y cookies

**Cookie vs sesión — la diferencia de fondo:**

```
COOKIE: el dato vive en el NAVEGADOR             SESIÓN: el dato vive en el SERVIDOR
┌─────────────┐                                  ┌─────────────┐          ┌──────────────────────┐
│ Navegador   │  Cookie: tema=oscuro             │ Navegador   │ Cookie:  │ Servidor             │
│ tema=oscuro │ ───────────────────▶ Servidor    │ session=abc │ ───────▶ │ abc → { usuario: 7,  │
└─────────────┘  (viaja en CADA petición)        └─────────────┘ solo id  │   carrito: [3, 8] }  │
                                                                          └──────────────────────┘
```

| Característica | Cookie | Sesión |
|---|---|---|
| Dónde se guarda el dato | Navegador del usuario | Servidor (archivo, base de datos, Redis) |
| Qué viaja en cada petición | El dato completo | Solo un identificador (que viaja en una cookie) |
| Capacidad | Unos 4 KB por cookie | Lo que permita el almacenamiento del servidor |
| ¿El usuario puede verla o modificarla? | Sí (Laravel la cifra, así que puede verla pero no leer ni alterar su contenido sin invalidarla) | No: solo ve el identificador |
| Duración típica | La que se indique (días, meses) | Hasta que expira por inactividad o se cierra sesión |
| Uso típico | Preferencias (tema, idioma), "recordarme" | Usuario autenticado, carrito, mensajes *flash* |

### 3.4.1. Manejo de sesiones

**Qué es:** una **sesión** es un espacio de almacenamiento **en el servidor**, asociado a un visitante concreto, que persiste entre peticiones. El navegador solo guarda un **identificador de sesión** aleatorio en una cookie (en Laravel, `laravel_session`); con ese identificador el servidor encuentra los datos que le corresponden.

**Por qué existe:** es la solución al carácter **sin estado** de HTTP. Sin sesiones, el servidor no podría saber que la persona que agregó un producto al carrito hace un minuto es la misma que ahora quiere pagar.

**Cómo funciona por dentro:**

| Paso | Qué ocurre |
|---|---|
| 1 | Primera visita: no hay cookie → Laravel genera un id aleatorio (40 caracteres) y crea una sesión vacía |
| 2 | Al responder: `Set-Cookie: laravel_session=<id cifrado>; HttpOnly` |
| 3 | El controlador hace `session(['carrito' => [3]])` → al terminar la petición, Laravel **guarda** los datos de la sesión en el *driver* configurado |
| 4 | Siguiente petición: llega `Cookie: laravel_session=...` → el middleware `StartSession` descifra el id y **carga** los datos guardados |
| 5 | Si pasan más de `SESSION_LIFETIME` minutos sin actividad (120 por defecto), la sesión expira |

**Drivers de sesión (`SESSION_DRIVER` en `.env`):**

| Driver | Dónde se guarda | Ventaja | Desventaja |
|---|---|---|---|
| `file` | `storage/framework/sessions/` | Sin configuración | No sirve si hay varios servidores |
| `database` (por defecto en Laravel 11+) | Tabla `sessions` | Compartida entre servidores, fácil de inspeccionar | Una consulta por petición |
| `redis` | Memoria (Redis) | Muy rápida | Requiere instalar Redis |
| `cookie` | Cookie cifrada en el navegador | Sin almacenamiento en el servidor | Límite de ~4 KB; viaja completa en cada petición |
| `array` | Memoria de la petición (no persiste) | — | Solo para pruebas automatizadas |

**Operaciones con la sesión:**

```php
// Escribir
session(['tema' => 'oscuro']);                       // helper
$request->session()->put('idioma', 'es');            // desde la petición
$request->session()->push('carrito', 8);             // agrega al final de un arreglo

// Leer
session('tema');                                     // "oscuro"
session('moneda', 'MXN');                            // "MXN"  ← no existe: valor por defecto
$request->session()->get('carrito', []);             // [3, 8]
$request->session()->has('idioma');                  // true
$request->session()->all();                          // todos los datos

// Eliminar
$request->session()->forget('tema');                 // borra una clave
$cupon = $request->session()->pull('cupon');         // la lee Y la borra
$request->session()->flush();                        // borra todo
```

**Traza — un carrito de compras en la sesión:**

| Petición | Acción | `session('carrito')` después |
|---|---|---|
| 1. `POST /carrito` (producto 3) | `push('carrito', 3)` | `[3]` |
| 2. `POST /carrito` (producto 8) | `push('carrito', 8)` | `[3, 8]` |
| 3. `GET /carrito` | `get('carrito', [])` | `[3, 8]` (solo lectura) |
| 4. `POST /carrito/vaciar` | `forget('carrito')` | `null` |
| 5. Otro navegador / ventana privada | — | `null`: es **otra sesión**, con otro id |

**Datos *flash* — duran exactamente una petición:** un dato **flash** se guarda en la sesión y se elimina automáticamente **después de la siguiente petición**. Es lo que se usa para los mensajes del tipo "Producto guardado" tras una redirección (patrón PRG, 3.1.1):

```php
return redirect()->route('productos.index')->with('exito', 'Producto guardado');
// equivale a: session()->flash('exito', 'Producto guardado'); return redirect(...);
```

| Petición | `session('exito')` |
|---|---|
| `POST /productos` → se hace `with('exito', ...)` y se redirige | — (se guarda para la siguiente) |
| `GET /productos` (la redirección) | `"Producto guardado"` → se muestra en la vista |
| `GET /productos` (el usuario recarga) | `null` → el mensaje ya no aparece |

Por la misma razón, `$errors` y `old()` funcionan tras una validación fallida: son datos flash.

**Errores y malentendidos típicos:**

- **Guardar objetos enormes en la sesión** (toda la lista de productos): cada petición debe cargarla y guardarla. Guarda identificadores y consulta los datos cuando los necesites.
- **Usar `put` para mensajes de "guardado":** el mensaje aparecería en todas las páginas hasta que se borrara. Para avisos de una sola vez se usa `flash`/`with`.
- **"Mis datos de sesión se pierden al cambiar de navegador":** es lo esperado; cada navegador tiene su propia cookie y, por lo tanto, su propia sesión.
- **Confundir la sesión de PHP (`$_SESSION`) con la de Laravel:** Laravel no usa `$_SESSION`; tiene su propio sistema. Escribir en `$_SESSION` no aparecerá en `session()`.

### 3.4.2. Lectura y escritura de cookies

**Qué es:** una **cookie** es un pequeño par **nombre=valor** que el servidor pide al navegador guardar (cabecera `Set-Cookie`) y que el navegador **devuelve automáticamente** en la cabecera `Cookie` de cada petición posterior al mismo sitio, hasta que expire.

**Por qué existe:** permite guardar en el cliente información que debe sobrevivir incluso al cierre del navegador (preferencias, "recordarme") sin ocupar espacio en el servidor, y es el mecanismo que transporta el identificador de sesión.

**Cómo funciona por dentro — atributos de una cookie:**

```
Set-Cookie: tema=eyJpdiI6...; expires=Mon, 16 Nov 2026 10:00:00 GMT; path=/; secure; httponly; samesite=lax
            └ nombre=valor ┘  └──────── cuándo expira ────────────┘ └ ruta ┘ └────── seguridad ──────────┘
```

| Atributo | Qué controla | Por qué importa |
|---|---|---|
| `expires` / `max-age` | Cuándo se borra | Sin él, es una **cookie de sesión del navegador**: se borra al cerrarlo |
| `path`, `domain` | En qué URLs se envía | Limita a qué partes del sitio llega |
| `Secure` | Solo se envía por HTTPS | Evita que viaje en texto plano |
| `HttpOnly` | JavaScript **no** puede leerla (`document.cookie` no la muestra) | Si un ataque XSS inyecta un script, no puede robar la cookie de sesión |
| `SameSite` (`Lax`, `Strict`, `None`) | Si se envía en peticiones iniciadas desde otros sitios | Defensa adicional contra CSRF; Laravel usa `lax` por defecto |

**Escribir cookies en Laravel:**

```php
use Illuminate\Support\Facades\Cookie;

// Opción 1: encolar — se adjunta automáticamente a la respuesta que se envíe
Cookie::queue('tema', 'oscuro', 60 * 24 * 30);     // nombre, valor, minutos (30 días = 43200)

// Opción 2: adjuntar a una respuesta concreta
return response('Preferencia guardada')->cookie('idioma', 'es', 60 * 24 * 365);

// Cookie de larga duración (los navegadores modernos limitan cualquier cookie a unos 400 días)
Cookie::queue(Cookie::forever('aviso_aceptado', '1'));

// Borrar: se envía la misma cookie con fecha de expiración en el pasado
Cookie::queue(Cookie::forget('tema'));
```

**Leer cookies:**

```php
$tema = $request->cookie('tema');           // "oscuro" (ya descifrada)
$tema = $request->cookie('tema', 'claro');  // "claro" si no existe
```

> **Ojo: una cookie escrita en esta petición no se puede leer en esta misma petición.** `Cookie::queue('tema', 'oscuro')` solo agrega una cabecera `Set-Cookie` a la respuesta; el navegador la guardará al recibirla y la enviará **a partir de la siguiente** petición.

**Traza — ciclo de vida de la cookie `tema`:**

| Petición | Cabecera `Cookie` que envía el navegador | Acción del servidor | `$request->cookie('tema')` |
|---|---|---|---|
| 1. `GET /` | *(ninguna)* | — | `null` |
| 2. `POST /preferencias` | *(ninguna)* | `Cookie::queue('tema', 'oscuro', 43200)` | `null` (aún no llega) |
| 3. `GET /` | `tema=<cifrado>` | — | `"oscuro"` |
| 4. Cerrar y abrir el navegador; `GET /` | `tema=<cifrado>` (no ha expirado) | — | `"oscuro"` |
| 5. `POST /preferencias/borrar` | `tema=<cifrado>` | `Cookie::forget('tema')` | `"oscuro"` (todavía llega en esta) |
| 6. `GET /` | *(ninguna)* | — | `null` |

**Cifrado de cookies:** el middleware `EncryptCookies` cifra (con `APP_KEY`) **todas** las cookies que crea Laravel. Si el usuario edita el valor en su navegador, al descifrarlo falla la verificación y Laravel la trata como inexistente. Si una cookie debe ser legible por JavaScript o por otro sistema, se excluye del cifrado:

```php
// bootstrap/app.php
->withMiddleware(function (Middleware $middleware) {
    $middleware->encryptCookies(except: ['tema']);   // 'tema' se guarda en texto plano
})
```

**¿Cookie o sesión? — árbol de decisión:**

```
¿El dato es sensible o decide permisos (id del usuario, rol, saldo)?
│
├─ Sí ──▶ SESIÓN  (el usuario nunca debe poder modificarlo)
│
└─ No ──▶ ¿Debe sobrevivir al cierre del navegador o durar semanas?
          ├─ Sí ──▶ COOKIE con expiración  (tema, idioma, aviso de cookies aceptado)
          └─ No ──▶ ¿Es grande (más de unos KB) o cambia en cada petición?
                    ├─ Sí ──▶ SESIÓN
                    └─ No ──▶ cualquiera; la sesión suele ser más simple
```

**Errores y malentendidos típicos:**

- **Leer con `$_COOKIE['tema']`:** devuelve el valor **cifrado** (texto ilegible), porque `$_COOKIE` es de PHP y no pasa por el middleware de Laravel. Usa `$request->cookie()`.
- **Pasar segundos en lugar de minutos:** `Cookie::queue('x', 'y', 3600)` en Laravel dura **3600 minutos** (2.5 días), no una hora.
- **Guardar datos que deciden permisos en una cookie sin cifrar** (`es_admin=1`): el usuario puede cambiarla a mano en las herramientas de desarrollador.
- **Superar el límite de ~4 KB:** el navegador descarta la cookie sin avisar.

---

## 3.5. Subida y descarga de archivos

### 3.5.1. Validación, almacenamiento y descarga de archivos

**Qué es:** **subir** un archivo es enviarlo desde el navegador al servidor dentro de una petición POST; **descargar** es que el servidor lo envíe de vuelta al navegador indicando que debe guardarse (o mostrarse). Laravel abstrae el almacenamiento con el sistema de **discos** (*filesystems*): ubicaciones configuradas donde se guardan los archivos, ya sea en el disco local o en la nube (Amazon S3, etc.).

**Por qué requiere cuidados especiales:** un archivo es la entrada más peligrosa que puede enviar un usuario. Si no se valida, alguien puede subir un `virus.php` a una carpeta pública y luego **ejecutarlo** visitando su URL, tomando el control del servidor. También puede llenar el disco con archivos enormes o sobrescribir archivos de otros usuarios si se usa el nombre original.

**Paso 1 — el formulario:**

```blade
<form action="{{ route('documentos.store') }}" method="POST" enctype="multipart/form-data">
    @csrf
    <input type="file" name="archivo" accept=".pdf,.jpg,.png">
    @error('archivo') <small>{{ $message }}</small> @enderror
    <button type="submit">Subir</button>
</form>
```

> **Ojo con `enctype="multipart/form-data"`:** sin este atributo el navegador envía el formulario como `application/x-www-form-urlencoded`, que solo transporta **texto**: en el servidor llega únicamente el nombre del archivo y `$request->file('archivo')` devuelve `null`. Con `multipart`, el cuerpo se divide en partes separadas por un delimitador, y cada parte puede contener datos binarios:

```
POST /documentos
Content-Type: multipart/form-data; boundary=----X7b

------X7b
Content-Disposition: form-data; name="_token"

Xy9k...
------X7b
Content-Disposition: form-data; name="archivo"; filename="constancia.pdf"
Content-Type: application/pdf

%PDF-1.7 ...bytes del archivo...
------X7b--
```

**Paso 2 — validación:**

```php
$request->validate([
    'archivo' => 'required|file|mimes:pdf,jpg,png|max:2048',   // max en KILOBYTES: 2048 KB = 2 MB
    'foto'    => 'nullable|image|max:1024|dimensions:min_width=200,min_height=200',
]);
```

| Regla | Comprueba |
|---|---|
| `file` | Que sea un archivo subido correctamente |
| `image` | Que sea una imagen (jpg, jpeg, png, bmp, gif, webp) |
| `mimes:pdf,jpg` | Que el tipo **detectado por el contenido** corresponda a esas extensiones |
| `mimetypes:application/pdf` | Que el tipo MIME detectado sea ese |
| `max:2048` / `min:10` | Tamaño en **kilobytes** |
| `dimensions:max_width=1920` | Dimensiones de la imagen en píxeles |

> **Por qué se valida por contenido y no por extensión:** cambiar el nombre de `virus.php` a `virus.pdf` no cambia lo que contiene. `mimes` analiza los primeros bytes del archivo (la "firma" o *magic number*: un PDF empieza con `%PDF`, un PNG con los bytes `89 50 4E 47`) para determinar su tipo real.

**Límites de PHP — antes de que Laravel valide nada:** PHP rechaza los archivos grandes **antes** de que lleguen a tu código, según su configuración en `php.ini`:

| Directiva | Valor por defecto | Efecto si se supera |
|---|---|---|
| `upload_max_filesize` | 2M | El archivo no se recibe; la regla `file`/`uploaded` falla con *"The archivo failed to upload"* |
| `post_max_size` | 8M | PHP descarta **todo** el cuerpo: no llegan ni el archivo ni los demás campos. Laravel lo detecta antes de validar y responde `413 Content Too Large` (`PostTooLargeException`) |

Para aceptar archivos de 10 MB no basta con `max:10240` en Laravel: también hay que subir esos dos valores en `php.ini` (y `post_max_size` debe ser mayor que `upload_max_filesize`).

**Paso 3 — almacenamiento:**

```php
public function store(Request $request)
{
    $request->validate(['archivo' => 'required|file|mimes:pdf,jpg,png|max:2048']);

    $archivo = $request->file('archivo');            // objeto UploadedFile
    $archivo->getClientOriginalName();               // "constancia.pdf"  ← nombre que puso el usuario (no confiable)
    $archivo->getSize();                             // 183204  (bytes)
    $archivo->extension();                           // "pdf"   ← deducida del contenido

    // Guarda con un nombre ÚNICO aleatorio y devuelve la ruta relativa
    $ruta = $archivo->store('documentos');           // "documentos/kT3vX9...aQ.pdf"  (disco local, privado)
    $ruta = $archivo->store('fotos', 'public');      // "fotos/Lm2...Zp.jpg"          (disco public)

    // Con un nombre elegido por ti
    $ruta = $archivo->storeAs('documentos', 'cliente-' . auth()->id() . '.pdf');

    Documento::create(['ruta' => $ruta, 'nombre_original' => $archivo->getClientOriginalName()]);  // Unidad 4

    return back()->with('exito', 'Archivo subido');
}
```

> **Ojo con guardar con el nombre original:** si dos usuarios suben `foto.jpg`, el segundo **sobrescribe** al primero; además el nombre puede contener caracteres problemáticos (`../`, espacios, acentos). `store()` genera un nombre aleatorio de 40 caracteres; el nombre original se guarda **en la base de datos** para mostrarlo o usarlo al descargar.

**Cómo funciona por dentro — los discos `local` y `public`:**

```
storage/app/
├── private/        ← disco "local" (Laravel 11+): NO accesible desde el navegador
│   └── documentos/kT3vX9...aQ.pdf
└── public/         ← disco "public"
    └── fotos/Lm2...Zp.jpg
             ▲
             │  enlace simbólico creado con: php artisan storage:link
             │
public/storage ──┘   → URL: http://127.0.0.1:8000/storage/fotos/Lm2...Zp.jpg
```

| Disco | Carpeta | ¿Accesible por URL? | Uso |
|---|---|---|---|
| `local` | `storage/app/private` (en Laravel 10 y anteriores, `storage/app`) | **No** | Documentos privados: constancias, facturas, identificaciones. Solo se entregan a través de un controlador que verifica permisos |
| `public` | `storage/app/public` | Sí, tras `storage:link` | Archivos que cualquiera puede ver: fotos de productos, avatares |
| `s3` (y otros) | Servicio en la nube | Según su configuración | Producción con varios servidores o grandes volúmenes |

> **Por qué el enlace simbólico:** solo `public/` es accesible desde la web (Unidad 2), pero los archivos subidos deben vivir en `storage/` para no mezclarse con el código. `php artisan storage:link` crea un "acceso directo" `public/storage` que apunta a `storage/app/public`, así se cumplen las dos condiciones. Si las imágenes dan 404, lo primero que se revisa es si se ejecutó este comando.

**Mostrar un archivo público:**

```blade
<img src="{{ Storage::url($producto->foto) }}" alt="{{ $producto->nombre }}">
{{-- Storage::url('fotos/Lm2...Zp.jpg') → "/storage/fotos/Lm2...Zp.jpg" --}}
{{-- equivalente: asset('storage/' . $producto->foto) --}}
```

**Paso 4 — descarga:**

```php
use Illuminate\Support\Facades\Storage;

public function descargar(Documento $documento)
{
    // Autorización: solo el dueño puede descargarlo
    abort_if($documento->user_id !== auth()->id(), 403);

    abort_unless(Storage::exists($documento->ruta), 404);

    return Storage::download($documento->ruta, $documento->nombre_original);
    // Content-Disposition: attachment; filename="constancia.pdf" → el navegador lo guarda con ese nombre
}

// Mostrar en el navegador (por ejemplo, un PDF) en vez de descargarlo
return response()->file(Storage::path($documento->ruta));   // sin "attachment": el navegador lo abre

// Eliminar
Storage::delete($documento->ruta);
Storage::disk('public')->delete($producto->foto);
```

**Descarga vs visualización:**

| Método | Cabecera | Comportamiento del navegador |
|---|---|---|
| `Storage::download($ruta, $nombre)` / `response()->download($rutaAbsoluta, $nombre)` | `Content-Disposition: attachment` | Abre "Guardar como" / descarga |
| `response()->file($rutaAbsoluta)` | Sin `attachment` | Lo muestra si sabe cómo (PDF, imagen) |
| Enlace directo a `/storage/...` | — | Solo para archivos del disco `public`; **sin** control de permisos |

**Traza — el viaje completo de un archivo:**

| Paso | Dónde | Estado |
|---|---|---|
| 1 | Navegador | El usuario elige `constancia.pdf` (183 204 bytes) |
| 2 | Petición | `POST /documentos`, `multipart/form-data` |
| 3 | PHP | Lo guarda en una carpeta **temporal** del sistema (`C:\...\Temp\php1A2B.tmp`) y lo borra al terminar la petición |
| 4 | Validación | `mimes:pdf` → la firma `%PDF` coincide · `max:2048` → 183 204 B ≈ 179 KB ≤ 2048 KB → válido |
| 5 | `store('documentos')` | Mueve el temporal a `storage/app/private/documentos/kT3vX9...aQ.pdf` |
| 6 | Base de datos | Guarda `ruta = documentos/kT3vX9...aQ.pdf`, `nombre_original = constancia.pdf` |
| 7 | Descarga | `GET /documentos/12/descargar` → verifica dueño → `Storage::download` → el usuario recibe `constancia.pdf` |

> **Ojo con el paso 3:** el archivo temporal **desaparece** al terminar la petición. Si no llamas a `store()`/`storeAs()` dentro de la misma petición, el archivo se pierde.

**Errores y malentendidos típicos:**

- **`$request->file('archivo')` devuelve `null`:** falta `enctype="multipart/form-data"`, el `name` no coincide o el archivo superó `upload_max_filesize`.
- **Confundir `max:2` con 2 MB:** en archivos `max` se mide en **kilobytes**: `max:2` son 2 KB. 2 MB es `max:2048`.
- **Guardar documentos privados en el disco `public`:** cualquiera que adivine o reciba la URL puede verlos, aunque no haya iniciado sesión.
- **Confiar en `getClientOriginalExtension()` o `getClientMimeType()`:** ambos vienen de lo que **dice** el cliente y se pueden falsificar. Para decidir el tipo se usan `extension()` (deducida del contenido) y las reglas `mimes`/`mimetypes`.
- **Borrar el registro en la base de datos sin borrar el archivo** (o al revés): quedan archivos "huérfanos" ocupando espacio, o registros que apuntan a archivos inexistentes. Elimina ambos en la misma operación (Unidad 4, transacciones).

---

*Universidad Politécnica de Tecamac — Ingeniería en Software*
