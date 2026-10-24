# Unidad 1 — Introducción a las Tecnologías Web

Esta unidad construye la base conceptual de todo el curso: de dónde viene la web, qué pasa realmente entre que escribes una dirección en el navegador y ves la página, y cuáles son las tres tecnologías con las que se construye cualquier página (HTML, CSS y JavaScript). Todo lo que se hará después con Laravel (unidades 2–4) sigue funcionando sobre estas mismas piezas: Laravel genera HTML en el servidor, y el navegador lo recibe por HTTP exactamente igual que una página estática.

---

## Índice

1. [Historia y evolución de la web](#11-historia-y-evolución-de-la-web)
   - 1.1.1. [Orígenes y desarrollo de la web](#111-orígenes-y-desarrollo-de-la-web)
   - 1.1.2. [Web 1.0, 2.0 y 3.0](#112-web-10-20-y-30)
2. [Cómo funciona la web](#12-cómo-funciona-la-web)
   - 1.2.1. [Modelo cliente-servidor](#121-modelo-cliente-servidor)
   - 1.2.2. [Protocolo HTTP y HTTPS](#122-protocolo-http-y-https)
3. [Estructura de una página web](#13-estructura-de-una-página-web)
   - 1.3.1. [HTML: estructura y etiquetas](#131-html-estructura-y-etiquetas)
   - 1.3.2. [CSS: diseño y estilo](#132-css-diseño-y-estilo)
   - 1.3.3. [JavaScript: interactividad](#133-javascript-interactividad)
4. [Diseño responsivo](#14-diseño-responsivo)
   - 1.4.1. [Media queries y buenas prácticas](#141-media-queries-y-buenas-prácticas)

---

## 1.1. Historia y evolución de la web

**Qué es:** la **World Wide Web** (o simplemente "la web") es un sistema de documentos y recursos enlazados entre sí mediante **hipervínculos**, a los que se accede a través de Internet usando un navegador. Cada recurso se identifica con una dirección única (**URL**) y se transfiere con un protocolo común (**HTTP**).

> **Ojo con confundir "Internet" con "la web".** **Internet** es la red física y lógica que conecta computadoras en todo el mundo (cables, routers, el protocolo TCP/IP). **La web** es solo *uno* de los servicios que funcionan sobre Internet, igual que el correo electrónico (SMTP), la transferencia de archivos (FTP) o los videojuegos en línea. Internet existe desde 1969 (ARPANET); la web apareció 20 años después.

```
┌────────────────────────── Internet (TCP/IP) ──────────────────────────┐
│                                                                        │
│   ┌─────────┐    ┌──────────┐    ┌─────────┐    ┌──────────────────┐   │
│   │   Web   │    │  Correo  │    │   FTP   │    │ Juegos, VoIP,    │   │
│   │ (HTTP)  │    │  (SMTP)  │    │         │    │ streaming, etc.  │   │
│   └─────────┘    └──────────┘    └─────────┘    └──────────────────┘   │
└────────────────────────────────────────────────────────────────────────┘
```

### 1.1.1. Orígenes y desarrollo de la web

**Por qué existe:** a finales de los años 80, en el **CERN** (el laboratorio europeo de física de partículas, en Suiza) trabajaban miles de científicos de distintos países, cada uno con computadoras, sistemas operativos y formatos de documentos diferentes. Compartir información era un problema serio: para leer un documento había que saber en qué máquina estaba, conectarse a ella y usar el programa correcto. **Tim Berners-Lee** propuso en **marzo de 1989** un sistema de "gestión de información" basado en **hipertexto**: documentos que contienen enlaces a otros documentos, sin importar en qué computadora estén guardados.

**Cómo funciona por dentro — las tres piezas que inventó Berners-Lee:** la web no fue un solo invento, sino la combinación de tres estándares que siguen siendo la base actual:

| Pieza | Qué resuelve | Ejemplo |
|---|---|---|
| **HTML** (*HyperText Markup Language*) | Un formato común para escribir documentos con enlaces, independiente del sistema operativo | `<a href="otra.html">Ver más</a>` |
| **URL** (*Uniform Resource Locator*) | Una forma única de nombrar cualquier recurso en cualquier servidor | `https://info.cern.ch/hypertext/WWW/TheProject.html` |
| **HTTP** (*HyperText Transfer Protocol*) | Un protocolo sencillo para pedir y entregar esos documentos | `GET /index.html` |

Sin cualquiera de las tres, la web no funciona: sin HTML no hay formato común, sin URL no hay forma de decir *qué* documento quieres, y sin HTTP no hay forma acordada de pedirlo.

**Línea del tiempo — momentos clave:**

| Año | Hecho | Por qué importa |
|---|---|---|
| 1969 | ARPANET conecta sus primeros nodos | Nace la red que evolucionará en Internet |
| 1983 | ARPANET adopta TCP/IP | Se establece el protocolo base de Internet que aún usamos |
| 1989 | Berners-Lee presenta su propuesta en el CERN | Origen conceptual de la web |
| 1990 | Primer servidor web y primer navegador (*WorldWideWeb*), en una computadora NeXT | La idea se vuelve software funcionando |
| 1991 | La web se abre al público fuera del CERN | Cualquiera puede crear y visitar páginas |
| 1993 | El CERN libera la web como dominio público; aparece el navegador **Mosaic** | Sin patentes ni licencias, la web puede crecer sin pagar a nadie; Mosaic muestra imágenes junto al texto y populariza la web |
| 1994 | Nace **Netscape Navigator**; se funda el **W3C** | El W3C (*World Wide Web Consortium*) se encarga de estandarizar las tecnologías web |
| 1995 | Netscape crea **JavaScript**; Microsoft lanza Internet Explorer | Comienza la "guerra de navegadores" |
| 1996 | Se publica **CSS1** como recomendación del W3C | Se separa el estilo visual del contenido |
| 2004–2005 | Se populariza el término **Web 2.0** y la técnica **AJAX** | Páginas que se actualizan sin recargar (Gmail, Google Maps) |
| 2008 | Lanzamiento de Google Chrome | Motores de JavaScript mucho más rápidos (V8) |
| 2014 | **HTML5** se vuelve recomendación del W3C | Video, audio, `<canvas>` y etiquetas semánticas nativas, sin plugins como Flash |
| 2019 en adelante | HTML se mantiene como **Living Standard** del WHATWG | HTML deja de tener "versiones" numeradas; se actualiza continuamente |

**La "guerra de navegadores" y por qué existen los estándares:** a mediados de los 90, Netscape e Internet Explorer competían agregando etiquetas y funciones propias que el otro navegador no entendía (por ejemplo, `<blink>` solo en Netscape y `<marquee>` solo en IE). El resultado era que los desarrolladores tenían que escribir **dos versiones** de cada página, o poner avisos de "esta página se ve mejor en…". Los estándares del W3C (y después del WHATWG) existen precisamente para evitar eso: una página escrita según el estándar debe verse igual en cualquier navegador que lo respete.

**Errores y malentendidos típicos:**

- **"Tim Berners-Lee inventó Internet":** no; inventó la **web**. Internet ya existía dos décadas antes.
- **"HTML5 es la última versión y después vendrá HTML6":** desde 2019 HTML es un **estándar vivo** (*Living Standard*): no hay versiones numeradas nuevas, se agregan funciones de forma continua. "HTML5" se sigue usando como nombre informal del HTML moderno.
- **Pensar que JavaScript tiene relación con Java:** el nombre fue una decisión de *marketing* de Netscape en 1995 para aprovechar la popularidad de Java. Son lenguajes completamente distintos en diseño, tipado y modelo de ejecución.

### 1.1.2. Web 1.0, 2.0 y 3.0

**Qué es:** "Web 1.0", "Web 2.0" y "Web 3.0" **no son versiones técnicas** de ningún software ni estándar; son **etiquetas históricas** que describen cómo cambió la forma en que la gente *usa* la web y quién produce el contenido.

**Por qué existen estas etiquetas:** sirven para describir cambios de fondo en el papel del usuario. La diferencia central entre las tres no es la tecnología sino **quién crea el contenido y quién lo controla**.

**Características de cada etapa:**

| Característica | Web 1.0 (≈1991–2004) | Web 2.0 (≈2004–actualidad) | Web 3.0 (en desarrollo) |
|---|---|---|---|
| Papel del usuario | **Lector** pasivo | **Lector y autor** (comenta, publica, comparte) | Usuario que además es **dueño** de sus datos (según la visión *Web3*) |
| Quién crea el contenido | El dueño del sitio (*webmaster*) | Los propios usuarios | Usuarios y aplicaciones que "entienden" los datos |
| Tipo de páginas | **Estáticas**: archivos HTML fijos | **Dinámicas**: generadas al momento desde una base de datos | Datos enlazados, aplicaciones descentralizadas |
| Interacción | Leer y navegar entre enlaces | Formularios, comentarios, "me gusta", actualizaciones sin recargar (AJAX) | Agentes y máquinas que interpretan significado |
| Tecnologías típicas | HTML básico, imágenes, tablas para maquetar | CSS, JavaScript, AJAX, PHP/Laravel, APIs, redes sociales | Web semántica (RDF), *blockchain*, IA |
| Ejemplos | Páginas personales, directorios como el primer Yahoo! | Wikipedia, YouTube, Facebook, blogs | Buscadores que responden preguntas, aplicaciones descentralizadas |

> **Ojo con "Web 3.0" porque tiene dos significados distintos que se mezclan con frecuencia:**
> - **Web semántica** (la visión original de Berners-Lee, ≈2001): que los datos de la web estén descritos de forma que las **máquinas** puedan entender su significado, no solo mostrarlos. Por ejemplo, que un buscador sepa que "Tecamac" es un municipio y no solo una palabra.
> - **Web3** (término de 2014, asociado a *blockchain*): una web **descentralizada**, donde los datos y la identidad no dependen de grandes empresas sino de redes distribuidas y criptomonedas.
>
> Ninguna de las dos está consolidada como "la" web actual. Cuando un examen o un texto hable de Web 3.0, conviene revisar a cuál de los dos sentidos se refiere.

**Cómo funciona por dentro — estática vs dinámica (la diferencia técnica clave entre 1.0 y 2.0):**

```
WEB 1.0 — página estática
Navegador ──GET /noticias.html──▶ Servidor
                                    │ busca el archivo noticias.html en disco
Navegador ◀──── el mismo archivo ───┘ (todos los visitantes reciben exactamente lo mismo)

WEB 2.0 — página dinámica
Navegador ──GET /noticias──▶ Servidor
                               │ 1. ejecuta un programa (PHP, Laravel…)
                               │ 2. consulta la base de datos (últimas noticias, usuario logueado)
                               │ 3. ARMA el HTML en ese momento
Navegador ◀── HTML generado ───┘ (cada usuario puede recibir algo distinto)
```

Esto es justo lo que hace Laravel: es una herramienta de la Web 2.0 para generar páginas dinámicas. Para actualizar una página 1.0 había que editar el archivo HTML a mano; en una página 2.0, basta con insertar un registro nuevo en la base de datos.

**Errores y malentendidos típicos:**

- **Creer que la Web 2.0 "reemplazó" a la 1.0 en una fecha concreta:** fue una transición gradual, y hoy siguen existiendo sitios estáticos (documentación técnica, *landing pages*) porque para ciertos casos son más rápidos, baratos y seguros.
- **Pensar que "estático" significa "sin JavaScript" o "sin diseño":** una página estática puede tener CSS y JavaScript muy elaborados. "Estático" se refiere a que el **servidor** entrega siempre el mismo archivo, no a que la página no se mueva.

---

## 1.2. Cómo funciona la web

### 1.2.1. Modelo cliente-servidor

**Qué es:** el **modelo cliente-servidor** es una forma de organizar un sistema en dos papeles: el **cliente**, que *pide* un servicio o recurso, y el **servidor**, que *espera* peticiones, las procesa y *responde*. En la web, el cliente casi siempre es un **navegador** (Chrome, Firefox, Edge) y el servidor es un programa (Apache, Nginx, o el servidor de Laravel) que corre en una computadora conectada a Internet.

**Por qué existe:** centralizar los datos y la lógica en un servidor resuelve varios problemas a la vez:

- **Un solo lugar de verdad:** si la información de las calificaciones está en el servidor de la universidad, todos los alumnos ven la misma versión actualizada. Si cada alumno tuviera su copia, se desincronizarían.
- **Seguridad:** la base de datos y las contraseñas nunca salen del servidor; el cliente solo recibe lo que se le permite ver.
- **Clientes ligeros:** un celular no necesita guardar todo YouTube; solo pide el video que va a reproducir.

**Características:**

| Característica | Descripción |
|---|---|
| **Petición–respuesta** | El servidor no envía nada por iniciativa propia: siempre **responde** a una petición del cliente (salvo tecnologías especiales como WebSockets) |
| **Roles asimétricos** | El cliente inicia la comunicación; el servidor espera ("escucha") en un **puerto** conocido (80 para HTTP, 443 para HTTPS) |
| **Muchos a uno** | Un servidor atiende a muchos clientes simultáneamente |
| **Independencia de plataforma** | Un servidor en Linux puede atender a clientes en Windows, Android o iOS, porque solo comparten el protocolo, no el sistema operativo |
| **Sin estado (en HTTP)** | Cada petición es independiente; el servidor no "recuerda" la anterior por sí solo (se profundiza en 1.2.2 y en la Unidad 3, sesiones) |

**Cómo funciona por dentro — qué pasa cuando escribes `https://www.ejemplo.com/productos` y presionas Enter:**

```
 Navegador (cliente)                 DNS                   Servidor web (93.184.215.14)
        │                             │                                 │
   1.   │── ¿IP de www.ejemplo.com? ─▶│                                 │
   2.   │◀──── 93.184.215.14 ─────────│                                 │
        │                                                               │
   3.   │════════ conexión TCP (y TLS, por ser https) al puerto 443 ═══▶│
        │                                                               │
   4.   │── GET /productos HTTP/1.1 ───────────────────────────────────▶│
        │   Host: www.ejemplo.com                                       │ 5. procesa: lee archivos,
        │                                                               │    ejecuta PHP, consulta BD
   6.   │◀── HTTP/1.1 200 OK + HTML ────────────────────────────────────│
        │                                                               │
   7.   │ lee el HTML, encuentra <link href="estilos.css">,             │
        │ <img src="logo.png">, <script src="app.js">                   │
   8.   │── GET /estilos.css, GET /logo.png, GET /app.js ──────────────▶│
        │◀── cada archivo en su propia respuesta ───────────────────────│
   9.   │ dibuja la página en pantalla                                  │
```

| Paso | Qué ocurre | Por qué es necesario |
|---|---|---|
| 1–2 | **Resolución DNS**: el navegador traduce el nombre de dominio a una dirección IP | Las computadoras se localizan por IP, no por nombres; el DNS es la "agenda telefónica" de Internet. El resultado se guarda en caché para no repetirlo en cada visita |
| 3 | Se abre una conexión **TCP** (y se negocia **TLS** si es HTTPS) | TCP garantiza que los datos lleguen completos y en orden; TLS los cifra |
| 4 | El navegador envía la **petición HTTP** | Indica qué recurso quiere (`/productos`) y con qué método (`GET`) |
| 5 | El servidor **procesa** la petición | En un sitio estático solo lee un archivo; en Laravel ejecuta rutas, controladores y consultas |
| 6 | El servidor envía la **respuesta HTTP** | Incluye un código de estado (`200`) y el contenido (HTML) |
| 7–8 | El navegador pide los **recursos secundarios** | Una página no es un solo archivo: cada CSS, imagen y script es **otra petición** |
| 9 | El navegador **renderiza** la página | Convierte HTML + CSS en píxeles y ejecuta el JavaScript (ver 1.3) |

> **Ojo con el paso 8:** una página "simple" puede generar decenas de peticiones (un HTML, 3 hojas de estilo, 20 imágenes, 5 scripts). Por eso optimizar la web consiste muchas veces en **reducir el número y el tamaño de las peticiones**, no solo en hacer más rápido el servidor.

**Arquitectura en capas — dónde queda Laravel:** en una aplicación real el "servidor" suele dividirse en varias piezas:

```
┌───────────┐  HTTP   ┌──────────────────┐        ┌──────────────────┐  SQL   ┌──────────────┐
│ Navegador │ ──────▶ │ Servidor web     │ ─────▶ │ Aplicación       │ ─────▶ │ Base de datos│
│ (cliente) │ ◀────── │ (Nginx / Apache) │ ◀───── │ (PHP + Laravel)  │ ◀───── │ (MySQL)      │
└───────────┘         └──────────────────┘        └──────────────────┘        └──────────────┘
  Capa de                 Recibe peticiones,          Lógica del negocio:        Guarda los datos
  presentación            entrega archivos            rutas, validación,         de forma
                          estáticos                   controladores              permanente
```

Nota que la aplicación Laravel es **servidor** respecto al navegador, pero es **cliente** respecto a la base de datos: los papeles de cliente y servidor dependen de quién pide y quién responde, no del tipo de máquina.

**Errores y malentendidos típicos:**

- **"El servidor es una computadora especial":** un servidor es un **programa** que escucha peticiones. Cuando ejecutes `php artisan serve` en la Unidad 2, tu propia laptop será el servidor y el navegador de esa misma laptop será el cliente (en la dirección `127.0.0.1`, que siempre significa "esta misma máquina").
- **Creer que el código del servidor llega al navegador:** el código PHP de Laravel **nunca** sale del servidor; el navegador solo recibe el HTML resultante. En cambio, el HTML, CSS y JavaScript del cliente sí son visibles para cualquiera (clic derecho → "Ver código fuente"). Por eso las validaciones de seguridad nunca deben hacerse solo en JavaScript.

**Cuándo hablar de cliente-servidor vs P2P:** en el modelo **P2P** (*peer-to-peer*, como BitTorrent), cada equipo es cliente y servidor a la vez y no hay un punto central. La web usa cliente-servidor porque necesita una fuente de verdad controlada (tu banco no puede depender de que otros usuarios tengan copia de tu saldo).

### 1.2.2. Protocolo HTTP y HTTPS

**Qué es:** **HTTP** (*HyperText Transfer Protocol*) es el conjunto de reglas que definen **cómo** un cliente pide un recurso y **cómo** el servidor responde. Es un protocolo de **texto** (en sus versiones 1.x), basado en **petición–respuesta** y **sin estado**. **HTTPS** es exactamente el mismo HTTP, pero viajando dentro de un canal **cifrado** con **TLS** (*Transport Layer Security*).

**Por qué existe:** cliente y servidor pueden estar escritos en lenguajes distintos, por empresas distintas y en sistemas operativos distintos. HTTP es el "idioma común" que ambos acuerdan para entenderse. HTTPS existe porque HTTP viaja en **texto plano**: cualquiera en la misma red (el Wi-Fi de una cafetería, el proveedor de Internet) puede leer y modificar lo que se envía, incluidas contraseñas.

**Cómo funciona por dentro — anatomía de una petición y una respuesta:**

```
PETICIÓN (cliente → servidor)
┌───────────────────────────────────────────────────────────┐
│ POST /login HTTP/1.1                    ← línea de petición: MÉTODO  RUTA  VERSIÓN
│ Host: www.ejemplo.com                   ← cabeceras (headers): metadatos
│ Content-Type: application/x-www-form-urlencoded
│ Cookie: laravel_session=eyJpdiI6...
│                                         ← línea en blanco: fin de cabeceras
│ email=ana%40mail.com&password=secreta   ← cuerpo (body): datos enviados
└───────────────────────────────────────────────────────────┘

RESPUESTA (servidor → cliente)
┌───────────────────────────────────────────────────────────┐
│ HTTP/1.1 302 Found                      ← línea de estado: VERSIÓN  CÓDIGO  TEXTO
│ Location: /panel                        ← cabeceras
│ Set-Cookie: laravel_session=nuevoValor; HttpOnly; Secure
│                                         ← línea en blanco
│ (cuerpo vacío en una redirección)       ← cuerpo: HTML, JSON, imagen…
└───────────────────────────────────────────────────────────┘
```

> **Ojo con el `%40` del cuerpo:** en los formularios, los caracteres especiales se **codifican** (`@` → `%40`, espacio → `+` o `%20`). El servidor los decodifica automáticamente; por eso en Laravel recibirás `ana@mail.com` sin hacer nada.

**Métodos HTTP principales** (se profundiza GET y POST en la Unidad 3):

| Método | Para qué se usa | ¿Lleva cuerpo? | **Seguro** (no modifica nada) | **Idempotente** (repetirlo da el mismo efecto) |
|---|---|---|---|---|
| `GET` | Obtener un recurso | No | Sí | Sí |
| `POST` | Crear un recurso / enviar datos para procesar | Sí | No | **No** (enviar dos veces puede crear dos registros) |
| `PUT` | Reemplazar un recurso completo | Sí | No | Sí |
| `PATCH` | Modificar parcialmente un recurso | Sí | No | No necesariamente |
| `DELETE` | Eliminar un recurso | Normalmente no | No | Sí (borrar algo ya borrado no cambia nada más) |

**Códigos de estado — se agrupan por su primer dígito:**

| Rango | Significado | Códigos más comunes |
|---|---|---|
| **1xx** | Informativo | `101 Switching Protocols` (al abrir un WebSocket) |
| **2xx** | Éxito | `200 OK`, `201 Created`, `204 No Content` |
| **3xx** | Redirección | `301 Moved Permanently`, `302 Found` (redirección temporal), `304 Not Modified` (usa tu copia en caché) |
| **4xx** | Error **del cliente** (la petición está mal) | `400 Bad Request`, `401 Unauthorized` (no autenticado), `403 Forbidden` (autenticado pero sin permiso), `404 Not Found`, `405 Method Not Allowed`, `422 Unprocessable Content` (datos inválidos), `429 Too Many Requests` |
| **5xx** | Error **del servidor** (la petición era válida, pero el servidor falló) | `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable` |

> **Ojo con 401 vs 403:** `401` significa "no sé quién eres" (no has iniciado sesión); `403` significa "sé quién eres, pero no tienes permiso". Laravel usa también un código **no estándar**, `419 Page Expired`, cuando falta o expiró el token CSRF de un formulario (Unidad 3).

**"Sin estado" — qué significa y qué problema causa:** **sin estado** (*stateless*) significa que el servidor trata cada petición como si fuera la primera; no guarda memoria de las anteriores. Esto hace a HTTP simple y escalable (cualquier servidor de un grupo puede atender cualquier petición), pero plantea un problema: ¿cómo sabe el servidor que la petición número 2 viene del mismo usuario que inició sesión en la petición número 1? La solución son las **cookies** y las **sesiones**: el servidor envía un identificador en una cabecera `Set-Cookie`, y el navegador lo devuelve automáticamente en la cabecera `Cookie` de cada petición siguiente.

```
Petición 1: POST /login (email + contraseña)
            ◀── Set-Cookie: laravel_session=abc123
Petición 2: GET /panel
            Cookie: laravel_session=abc123  ──▶ el servidor busca "abc123" en sus sesiones:
                                                 "ah, es Ana, que ya inició sesión"
```

**Cómo funciona HTTPS por dentro:** antes de enviar la primera petición HTTP, cliente y servidor realizan un **handshake TLS** (saludo inicial):

| Paso | Qué ocurre | Qué protege |
|---|---|---|
| 1 | El cliente saluda e indica qué algoritmos de cifrado soporta | — |
| 2 | El servidor responde con su **certificado digital** (contiene su clave pública y está firmado por una **Autoridad Certificadora**, como Let's Encrypt) | **Autenticidad**: el navegador verifica la firma y comprueba que realmente hablas con `ejemplo.com` y no con un impostor |
| 3 | Ambos calculan una **clave de sesión simétrica** compartida sin enviarla directamente por la red | **Confidencialidad**: solo ellos dos conocen la clave |
| 4 | A partir de aquí, todo el HTTP viaja cifrado con esa clave | **Integridad**: si alguien modifica un byte en el camino, la verificación falla y la conexión se rechaza |

> **Por qué una clave simétrica y no usar siempre la clave pública:** el cifrado asimétrico (clave pública/privada) es seguro para acordar claves, pero muy lento para cifrar grandes cantidades de datos. Por eso se usa solo al inicio para acordar una clave **simétrica**, que es mucho más rápida, y con esa se cifra el resto de la comunicación.

**HTTP vs HTTPS:**

| Aspecto | HTTP | HTTPS |
|---|---|---|
| Puerto por defecto | 80 | 443 |
| Cifrado | Ninguno (texto plano) | TLS |
| Verifica la identidad del servidor | No | Sí, mediante certificado |
| Indicador en el navegador | "No seguro" | Candado / sin advertencia |
| Requisito para funciones modernas | — | Necesario para geolocalización, cámara, *service workers*, HTTP/2 en navegadores |
| Impacto en SEO | Penalizado | Preferido por buscadores |

**Versiones de HTTP:**

| Versión | Año | Novedad principal |
|---|---|---|
| HTTP/0.9 | 1991 | Solo `GET`, solo HTML, sin cabeceras |
| HTTP/1.0 | 1996 | Cabeceras, códigos de estado, otros tipos de archivo |
| HTTP/1.1 | 1997 | Conexiones persistentes (reutilizar la conexión TCP), cabecera `Host` obligatoria (varios sitios en un mismo servidor) |
| HTTP/2 | 2015 | Protocolo **binario**, **multiplexación**: varias peticiones a la vez por la misma conexión |
| HTTP/3 | 2022 | Funciona sobre **QUIC** (basado en UDP) en lugar de TCP; menos latencia en redes inestables como la móvil |

La semántica (métodos, códigos, cabeceras) es la **misma** en todas las versiones modernas: un `GET` que devuelve `404` significa lo mismo en HTTP/1.1 que en HTTP/3. Lo que cambia es cómo se transportan los bytes. Por eso, como programador en Laravel, casi nunca necesitas preocuparte por la versión.

**Errores y malentendidos típicos:**

- **"HTTPS hace que un sitio sea seguro":** HTTPS protege los datos **mientras viajan**, pero no protege contra un sitio malicioso (una página de *phishing* puede tener un certificado válido) ni contra errores de programación del servidor (inyección SQL, contraseñas mal guardadas).
- **"Si uso POST, los datos están ocultos y son seguros":** con HTTP plano, el cuerpo de un POST viaja tan legible como una URL de GET. Lo único que oculta los datos es HTTPS. POST solo evita que aparezcan en la barra de direcciones y en el historial.
- **Confundir 4xx con 5xx al depurar:** si ves un `500`, el problema está en tu código del servidor (revisa `storage/logs/laravel.log` en Laravel); si ves un `4xx`, revisa qué está enviando el cliente (ruta, método, datos, sesión).

---

## 1.3. Estructura de una página web

**Qué es:** toda página web se construye con tres lenguajes, cada uno con una responsabilidad distinta. Esta división se llama **separación de responsabilidades** (*separation of concerns*):

| Capa | Lenguaje | Responsabilidad | Analogía con una casa |
|---|---|---|---|
| **Estructura / contenido** | HTML | *Qué* hay en la página y qué significa cada parte | Muros, cuartos, puertas |
| **Presentación** | CSS | *Cómo se ve*: colores, tamaños, posición | Pintura, decoración, muebles |
| **Comportamiento** | JavaScript | *Qué hace* cuando el usuario interactúa | Instalación eléctrica: interruptores, timbre |

**Por qué separarlos:** si el estilo viviera dentro del HTML (como se hacía en la Web 1.0 con etiquetas `<font color="red">`), cambiar el color de todos los títulos de un sitio de 200 páginas obligaría a editar las 200. Con CSS separado, se cambia **una línea** en un solo archivo. Además, el mismo HTML puede presentarse distinto en pantalla, en impresión o en un lector de pantalla para personas ciegas.

**Cómo funciona por dentro — cómo el navegador convierte el código en píxeles (*critical rendering path*):**

```
 HTML ──parsear──▶ DOM (árbol de elementos)
                       \
                        ──▶ Render tree ──▶ Layout ──▶ Paint ──▶ Composite ──▶ pantalla
                       /    (solo lo        (calcular   (pintar   (combinar
 CSS  ──parsear──▶ CSSOM     visible +       posición y  píxeles)  capas)
                    (árbol    estilos)        tamaño)
                    de estilos)
                                ▲
 JavaScript ───────────────────┘  puede leer y modificar el DOM y el CSSOM en cualquier momento,
                                  lo que obliga a repetir layout y paint
```

| Etapa | Qué hace | Detalle importante |
|---|---|---|
| **DOM** | Convierte el texto HTML en un árbol de objetos (*Document Object Model*) | Es lo que JavaScript manipula, no el archivo HTML original |
| **CSSOM** | Convierte el CSS en un árbol de reglas | Mientras no termina, el navegador **no pinta nada** (el CSS bloquea el renderizado) para no mostrar una página sin estilo y luego "saltar" |
| **Render tree** | Combina DOM + CSSOM, dejando solo lo visible | Un elemento con `display: none` **no** entra al render tree; uno con `visibility: hidden` sí (ocupa espacio aunque no se vea) |
| **Layout** (o *reflow*) | Calcula posición y tamaño exactos de cada caja | Es la etapa más costosa; cambiar un ancho puede obligar a recalcular toda la página |
| **Paint** | Rellena los píxeles: texto, colores, bordes, sombras | — |
| **Composite** | Combina capas en el orden correcto | Las animaciones con `transform` y `opacity` solo afectan esta etapa, por eso son las más fluidas |

### 1.3.1. HTML: estructura y etiquetas

**Qué es:** **HTML** (*HyperText Markup Language*) es un **lenguaje de marcado**, no un lenguaje de programación: no tiene variables, condiciones ni ciclos. Su trabajo es **etiquetar** el contenido para indicar qué es cada parte (un título, un párrafo, un enlace, una lista) mediante **etiquetas** (*tags*).

**Anatomía de un elemento:**

```
  etiqueta de apertura          contenido          etiqueta de cierre
 ┌────────────────────────────┐┌───────────┐┌────┐
 <a href="/contacto" class="btn">Escríbenos</a>
    │    └──── valor ───┘ │
    │                     └── otro atributo
    └── nombre del atributo
 └──────────────────── elemento completo ────────────────┘
```

- **Elemento:** la unidad completa (apertura + contenido + cierre).
- **Atributo:** información adicional sobre el elemento, siempre en la etiqueta de apertura (`href`, `class`, `id`, `src`, `alt`).
- **Elementos vacíos** (*void elements*): no tienen contenido ni cierre: `<img>`, `<br>`, `<input>`, `<meta>`, `<link>`, `<hr>`.

**Estructura mínima de un documento HTML:**

```html
<!DOCTYPE html>                  <!-- le dice al navegador: usa el modo estándar moderno -->
<html lang="es">                 <!-- idioma: lo usan buscadores y lectores de pantalla -->
<head>                           <!-- metadatos: NO se muestran en la página -->
    <meta charset="UTF-8">       <!-- codificación: sin esto, "ñ" y acentos pueden verse como "Ã±" -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0"> <!-- clave para el diseño responsivo (1.4) -->
    <title>Mi tienda</title>     <!-- texto de la pestaña y del resultado en buscadores -->
    <link rel="stylesheet" href="estilos.css">
</head>
<body>                           <!-- todo lo visible -->
    <h1>Bienvenido</h1>
    <p>Este es un párrafo.</p>
    <script src="app.js" defer></script>
</body>
</html>
```

> **Ojo con omitir `<!DOCTYPE html>`:** sin esa línea, el navegador entra en **modo *quirks*** (modo de compatibilidad con páginas de los 90) y calcula ciertos tamaños de forma distinta. El síntoma típico es que el CSS "no funciona igual" que en otro proyecto sin razón aparente.

**Cómo funciona por dentro — el documento es un árbol:** el navegador convierte el HTML anterior en el siguiente DOM. Cada etiqueta es un **nodo**; las etiquetas dentro de otras son sus **hijos**:

```
document
└── html (lang="es")
    ├── head
    │   ├── meta (charset)
    │   ├── meta (viewport)
    │   ├── title ── "Mi tienda"
    │   └── link (estilos.css)
    └── body
        ├── h1 ── "Bienvenido"
        ├── p ── "Este es un párrafo."
        └── script (app.js)
```

Esta estructura de árbol explica tres cosas que se verán después: por qué las etiquetas deben **anidarse correctamente** (un hijo no puede "salirse" de su padre), por qué CSS puede seleccionar "los `p` dentro de un `article`", y por qué JavaScript recorre la página con métodos como `parentElement` o `children`.

**Etiquetas más usadas:**

| Categoría | Etiquetas | Uso |
|---|---|---|
| Títulos | `<h1>` … `<h6>` | Jerarquía de encabezados; `<h1>` es el título principal (normalmente uno por página) |
| Texto | `<p>`, `<strong>`, `<em>`, `<span>`, `<br>` | Párrafo, importancia, énfasis, fragmento sin significado, salto de línea |
| Enlaces e imágenes | `<a href>`, `<img src alt>` | Hipervínculo; imagen con texto alternativo |
| Listas | `<ul>`, `<ol>`, `<li>` | Lista sin orden, ordenada, elemento de lista |
| Tablas | `<table>`, `<thead>`, `<tbody>`, `<tr>`, `<th>`, `<td>` | Datos tabulares (no para maquetar) |
| Formularios | `<form>`, `<input>`, `<label>`, `<select>`, `<textarea>`, `<button>` | Captura de datos (Unidad 3) |
| Contenedores genéricos | `<div>` (bloque), `<span>` (en línea) | Agrupar sin aportar significado |
| Semánticas | `<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<aside>`, `<footer>` | Agrupar **con** significado |

**Elementos de bloque vs en línea:**

| Característica | Bloque (`div`, `p`, `h1`, `section`, `ul`) | En línea (`span`, `a`, `strong`, `img`) |
|---|---|---|
| Empieza en una línea nueva | Sí | No, se acomoda junto al texto |
| Ancho por defecto | 100% del contenedor | Solo lo que ocupa su contenido |
| Respeta `width` y `height` | Sí | No (excepto elementos como `img`, que son *replaced*) |
| Margen vertical | Sí | No afecta la posición de las líneas |

**HTML semántico — por qué importa usar la etiqueta correcta:** el **HTML semántico** consiste en elegir cada etiqueta por lo que **significa**, no por cómo se ve. Visualmente, un `<div>` con estilo y un `<nav>` con el mismo estilo son idénticos; la diferencia es para las máquinas:

```
SIN semántica                          CON semántica
┌─────────────────────────┐            ┌─────────────────────────┐
│ <div class="arriba">    │            │ <header>                │
│   <div class="menu">    │            │   <nav>                 │
├─────────────────────────┤            ├─────────────────────────┤
│ <div class="contenido"> │            │ <main>                  │
│   <div class="nota">    │            │   <article>             │
├─────────────────────────┤            ├─────────────────────────┤
│ <div class="abajo">     │            │ <footer>                │
└─────────────────────────┘            └─────────────────────────┘
 Un lector de pantalla solo ve          Un lector de pantalla puede decir
 "grupo, grupo, grupo…"                 "navegación" y permite saltar
                                        directamente al contenido principal
```

Beneficios concretos: **accesibilidad** (lectores de pantalla), **SEO** (los buscadores entienden qué parte es el contenido principal) y **mantenimiento** (otro programador entiende la estructura sin leer el CSS).

**Errores y malentendidos típicos:**

- **Anidar mal las etiquetas:** `<p><strong>texto</p></strong>` es inválido. El navegador no muestra error: intenta "adivinar" y corrige el árbol a su manera, lo que puede producir estilos aplicados a lugares inesperados. Valida el HTML con el validador del W3C cuando algo se vea raro.
- **Usar `<br>` repetidos o `<table>` para acomodar elementos:** eso es trabajo de CSS. Las tablas son solo para **datos tabulares**.
- **Omitir `alt` en las imágenes:** además de afectar la accesibilidad, si la imagen no carga el usuario no sabe qué había. Si la imagen es puramente decorativa, se usa `alt=""` (vacío pero presente) para que el lector de pantalla la ignore.
- **Elegir `<h3>` porque "se ve del tamaño que quiero":** el nivel del encabezado indica jerarquía, no tamaño. El tamaño se ajusta con CSS.
- **Repetir un mismo `id` en varios elementos:** `id` debe ser **único** en la página; para varios elementos se usa `class`. Con ids repetidos, `document.getElementById` solo devuelve el primero.

### 1.3.2. CSS: diseño y estilo

**Qué es:** **CSS** (*Cascading Style Sheets*, hojas de estilo en cascada) es el lenguaje que define la **presentación** de un documento HTML: colores, tipografías, espacios, tamaños y la disposición de los elementos en la pantalla.

**Anatomía de una regla CSS:**

```css
  selector       declaración          declaración
 ┌──────────┐ ┌──────────────────┐ ┌─────────────────┐
 .tarjeta h2 { color: #1a5276;      font-size: 1.5rem; }
              └─┬─┘  └──┬───┘
            propiedad  valor
```

**Tres formas de incluir CSS:**

| Forma | Ejemplo | Cuándo usarla |
|---|---|---|
| **Externa** | `<link rel="stylesheet" href="estilos.css">` | Siempre que sea posible: se reutiliza en todas las páginas y el navegador la guarda en caché |
| **Interna** | `<style> p { color: gray; } </style>` en el `<head>` | Estilos exclusivos de una sola página |
| **En línea** | `<p style="color: red;">` | Casos excepcionales; mezcla estilo con contenido y tiene la especificidad más alta, lo que dificulta sobrescribirla |

**Selectores principales:**

| Selector | Ejemplo | Selecciona |
|---|---|---|
| Etiqueta | `p` | Todos los `<p>` |
| Clase | `.alerta` | Elementos con `class="alerta"` |
| Id | `#menu` | El elemento con `id="menu"` |
| Descendiente | `nav a` | Los `<a>` que estén **en cualquier nivel** dentro de un `<nav>` |
| Hijo directo | `ul > li` | Los `<li>` que son hijos **directos** de un `<ul>` |
| Atributo | `input[type="email"]` | Los `<input>` cuyo `type` sea `email` |
| Pseudoclase | `a:hover`, `li:first-child` | Un elemento en cierto **estado** o posición |
| Pseudoelemento | `p::first-line`, `.icono::before` | Una **parte** de un elemento, o contenido generado |

**Por qué "en cascada" — cómo decide el navegador qué regla gana:** es común que varias reglas apliquen al mismo elemento con valores contradictorios. El navegador resuelve el conflicto en este orden:

```
¿Hay varias reglas para la misma propiedad del mismo elemento?
│
├─ 1. ¿Alguna tiene !important?  ── sí ──▶ gana esa (evítalo: rompe la cascada)
│
├─ 2. Compara ESPECIFICIDAD ─────────────▶ gana la más específica
│
└─ 3. Empate en especificidad ───────────▶ gana la que aparece DESPUÉS en el código
```

**Cómo se calcula la especificidad:** se cuenta como una terna **(A, B, C)** y se compara de izquierda a derecha (como un número donde A pesa más que cualquier cantidad de B):

- **A:** número de selectores de **id**.
- **B:** número de **clases**, **atributos** y **pseudoclases**.
- **C:** número de **etiquetas** y **pseudoelementos**.
- Los estilos **en línea** (`style="..."`) le ganan a cualquier selector.

**Ejemplo resuelto — ¿de qué color queda el enlace?**

```html
<nav id="menu">
  <a class="item activo" href="#">Inicio</a>
</nav>
```

```css
a               { color: black; }   /* (0,0,1) */
.item           { color: blue;  }   /* (0,1,0) */
nav .item.activo{ color: green; }   /* (0,2,1) */
#menu a         { color: red;   }   /* (1,0,1) */
```

| Regla | Ids (A) | Clases (B) | Etiquetas (C) | Especificidad |
|---|---|---|---|---|
| `a` | 0 | 0 | 1 | (0,0,1) |
| `.item` | 0 | 1 | 0 | (0,1,0) |
| `nav .item.activo` | 0 | 2 | 1 | (0,2,1) |
| `#menu a` | **1** | 0 | 1 | **(1,0,1)** ← gana |

Resultado: el enlace es **rojo**. Aunque `nav .item.activo` tiene más selectores en total, un solo id (A = 1) supera a cualquier cantidad de clases. Por eso se recomienda estilizar con **clases** y no con ids: los ids crean reglas difíciles de sobrescribir después.

> **Ojo con la herencia, que es distinta de la cascada:** algunas propiedades se **heredan** de padres a hijos (`color`, `font-family`, `font-size`), y otras no (`border`, `margin`, `padding`, `background`). Por eso si pones `color: blue` en el `<body>`, todo el texto se vuelve azul, pero si pones un `border` en el `<body>`, los párrafos no reciben borde.

**El modelo de caja (*box model*):** para CSS, **todo elemento es una caja rectangular** formada por cuatro capas, de adentro hacia afuera:

```
┌─────────────────────────── margin (espacio EXTERIOR, transparente) ────────────┐
│   ┌──────────────────────── border (borde visible) ────────────────────────┐   │
│   │   ┌───────────────────── padding (relleno INTERIOR) ────────────────┐   │   │
│   │   │                                                                 │   │   │
│   │   │                    content (texto, imagen)                     │   │   │
│   │   │                                                                 │   │   │
│   │   └─────────────────────────────────────────────────────────────────┘   │   │
│   └─────────────────────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────────────────┘
```

**Ejemplo numérico — `content-box` vs `border-box`:**

```css
.caja { width: 200px; padding: 20px; border: 5px solid; margin: 10px; }
```

| `box-sizing` | Qué mide `width: 200px` | Ancho visible (contenido + padding + borde) | Ancho del contenido |
|---|---|---|---|
| `content-box` (por defecto) | Solo el **contenido** | 200 + 20·2 + 5·2 = **250 px** | 200 px |
| `border-box` | Contenido + padding + borde | **200 px** | 200 − 40 − 10 = **150 px** |

En ambos casos el `margin` (10 px por lado) se suma **afuera**, así que la caja ocupa 270 px o 220 px de espacio respectivamente. El comportamiento por defecto (`content-box`) es la causa número uno de que "un elemento de 50% + otro de 50% no quepan en la misma fila": cualquier padding o borde los hace sumar más de 100%. Por eso casi todos los proyectos empiezan con:

```css
*, *::before, *::after { box-sizing: border-box; }
```

> **Ojo con el colapso de márgenes:** cuando dos márgenes **verticales** se tocan (por ejemplo, un `<p>` con `margin-bottom: 20px` seguido de otro con `margin-top: 30px`), no se suman: el espacio resultante es el **mayor** de los dos (30 px, no 50 px). Esto no ocurre con márgenes horizontales ni dentro de contenedores flex o grid.

**Maquetación (*layout*) — Flexbox vs Grid:**

| Aspecto | **Flexbox** (`display: flex`) | **Grid** (`display: grid`) |
|---|---|---|
| Dimensiones | **Una** (fila *o* columna) | **Dos** (filas *y* columnas a la vez) |
| Enfoque | Desde el **contenido**: los elementos se reparten el espacio | Desde el **contenedor**: defines la cuadrícula y colocas elementos en ella |
| Ideal para | Barras de navegación, alinear botones, centrar un elemento | La estructura general de la página, galerías, tableros |
| Propiedades clave | `justify-content`, `align-items`, `flex-wrap`, `gap` | `grid-template-columns`, `grid-template-areas`, `gap` |

```css
/* Centrar un elemento vertical y horizontalmente (antes de Flexbox era sorprendentemente difícil) */
.contenedor { display: flex; justify-content: center; align-items: center; height: 100vh; }

/* Galería: tantas columnas de al menos 200px como quepan */
.galeria { display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 16px; }
```

**Unidades de medida:**

| Unidad | Relativa a | Uso recomendado |
|---|---|---|
| `px` | Fija (píxel de CSS) | Bordes, sombras, detalles finos |
| `%` | El tamaño del elemento **padre** | Anchos fluidos |
| `em` | El `font-size` del **propio elemento** (o del padre, si se usa en `font-size`) | Espaciados que deben crecer con el texto del componente |
| `rem` | El `font-size` del elemento **raíz** (`<html>`, normalmente 16 px) | Tipografía y espaciados generales: `1.5rem` = 24 px si la raíz mide 16 px |
| `vw` / `vh` | 1% del ancho / alto de la ventana | Secciones a pantalla completa |

> **Ojo con `em` anidado:** si un `<li>` tiene `font-size: 1.2em` y dentro hay otra lista con otro `<li>` de `1.2em`, el segundo mide 1.2 × 1.2 = 1.44 veces el tamaño base, porque cada `em` se calcula sobre su padre. `rem` evita ese efecto acumulativo porque siempre se mide respecto a la raíz.

**Errores y malentendidos típicos:**

- **Usar `!important` para "arreglar" un estilo que no se aplica:** casi siempre el problema es de especificidad u orden. `!important` resuelve el síntoma, pero obliga a usar otro `!important` más adelante y vuelve el CSS imposible de mantener.
- **Escribir `.menu.item` queriendo decir `.menu .item`:** sin espacio significa "un elemento que tiene **ambas** clases"; con espacio significa "un `.item` **dentro** de un `.menu`".
- **Asumir que el estilo no se aplica por un error de CSS, cuando en realidad es caché:** el navegador guarda los CSS externos en caché; usa recarga forzada (Ctrl + F5) o las herramientas de desarrollador (F12) para comprobar qué reglas se están aplicando y cuáles aparecen tachadas.

### 1.3.3. JavaScript: interactividad

**Qué es:** **JavaScript** es un lenguaje de **programación** (a diferencia de HTML y CSS) que se ejecuta **en el navegador** del usuario. Permite reaccionar a acciones (clics, teclas, envío de formularios), modificar la página sin recargarla y comunicarse con el servidor en segundo plano. Es **interpretado** (compilado al vuelo por el motor del navegador, como V8 en Chrome), de **tipado dinámico** y su estándar oficial se llama **ECMAScript**.

**Por qué existe:** sin JavaScript, cualquier cambio en la página requiere pedir una página **nueva** completa al servidor. Validar que un correo tenga `@` significaría enviar el formulario, esperar la respuesta y recargar todo solo para mostrar "correo inválido". JavaScript permite hacer esa comprobación **instantáneamente** en el cliente y actualizar solo la parte de la página que cambió.

**Cómo funciona por dentro — JavaScript y el DOM:** JavaScript **no** modifica el archivo HTML; modifica el **DOM**, el árbol de objetos que el navegador construyó a partir de él (ver 1.3). Cada cambio en el DOM se refleja inmediatamente en pantalla:

```html
<button id="btn">Contar</button>
<p id="salida">Clics: 0</p>

<script>
  let clics = 0;                                         // estado guardado en memoria del navegador
  const boton  = document.querySelector('#btn');         // busca el nodo en el DOM (selector CSS)
  const salida = document.querySelector('#salida');

  boton.addEventListener('click', () => {                // registra qué hacer cuando ocurra el evento
    clics++;
    salida.textContent = `Clics: ${clics}`;              // modifica el DOM → la pantalla se actualiza
  });
</script>
```

**Trace paso a paso — qué ocurre con tres clics:**

| Momento | Valor de `clics` | Texto de `#salida` en pantalla |
|---|---|---|
| Carga de la página | 0 | `Clics: 0` (viene del HTML) |
| 1.er clic | 1 | `Clics: 1` |
| 2.º clic | 2 | `Clics: 2` |
| 3.er clic | 3 | `Clics: 3` |
| Recargar la página (F5) | 0 | `Clics: 0` — el estado en memoria se pierde, el HTML original se vuelve a cargar |

La última fila es importante: lo que JavaScript guarda en variables **vive solo mientras la página está abierta**. Para que un dato persista hay que guardarlo en el servidor (base de datos, Unidad 4) o en el almacenamiento del navegador (`localStorage`, cookies).

**Tipado dinámico y comparaciones — una fuente clásica de errores:**

```javascript
let x = 5;           // x es number
x = "cinco";         // ahora es string: JavaScript lo permite

typeof 42            // "number"
typeof "hola"        // "string"
typeof null          // "object"  ← error histórico del lenguaje que nunca se corrigió

"5" == 5             // true   ← == convierte tipos antes de comparar (coerción)
"5" === 5            // false  ← === compara valor Y tipo, sin convertir
0 == ""              // true
0 === ""             // false
"3" + 2              // "32"   ← con un string, + concatena
"3" - 2              // 1      ← - solo tiene sentido numérico, convierte "3" a número
```

> **Ojo con `==`:** usa siempre `===` y `!==`. Las reglas de conversión de `==` son tan poco intuitivas que la mayoría de las guías de estilo lo prohíben.

**`var`, `let` y `const`:**

| Característica | `var` (antiguo) | `let` | `const` |
|---|---|---|---|
| Ámbito | De **función** | De **bloque** `{ }` | De **bloque** `{ }` |
| Se puede reasignar | Sí | Sí | **No** |
| Se puede redeclarar en el mismo ámbito | Sí (fuente de errores) | No | No |
| Uso recomendado | Evitarlo | Valores que cambian (contadores) | Por defecto, para todo lo demás |

> **Ojo con `const` y los objetos:** `const` impide **reasignar la variable**, no modificar el objeto. `const lista = [1, 2]; lista.push(3);` funciona (`lista` sigue apuntando al mismo arreglo, que ahora tiene 3 elementos), pero `lista = [9];` lanza `TypeError`. Es el mismo comportamiento de referencias que ocurre en Java con una variable `final` que apunta a un objeto.

**Asincronía — el *event loop*:** JavaScript en el navegador ejecuta tu código en un **solo hilo**: solo puede hacer una cosa a la vez. Si una operación lenta (pedir datos al servidor) bloqueara ese hilo, la página se congelaría. Para evitarlo, las operaciones lentas se delegan al navegador y su resultado se "encola" para ejecutarse **cuando el hilo quede libre**.

```javascript
console.log("A");
setTimeout(() => console.log("B"), 0);          // macrotarea: va a la cola de tareas
Promise.resolve().then(() => console.log("C")); // microtarea: va a la cola de microtareas
console.log("D");
// Salida: A, D, C, B
```

**Trace paso a paso:**

| Paso | Qué se ejecuta | Pila (ejecutando ahora) | Cola de microtareas | Cola de tareas | Salida acumulada |
|---|---|---|---|---|---|
| 1 | `console.log("A")` | código principal | — | — | A |
| 2 | `setTimeout(..., 0)` | código principal | — | `B` | A |
| 3 | `Promise...then(...)` | código principal | `C` | `B` | A |
| 4 | `console.log("D")` | código principal | `C` | `B` | A D |
| 5 | Termina el código principal; se vacían **todas** las microtareas | `C` | — | `B` | A D C |
| 6 | Se toma la siguiente tarea | `B` | — | — | A D C B |

Conclusión: `setTimeout(..., 0)` **no** significa "ejecutar ahora", sino "ejecutar en cuanto el hilo esté libre y no queden microtareas". Este es el mecanismo que usa `fetch` para pedir datos al servidor sin congelar la página:

```javascript
async function cargarProductos() {
  const respuesta = await fetch('/api/productos');  // la función se pausa aquí sin bloquear la página
  const datos = await respuesta.json();             // convierte el cuerpo JSON en objetos de JS
  console.log(datos.length);                        // p. ej. 12
}
```

**Dónde colocar el `<script>` — `defer` y `async`:** cuando el navegador encuentra un `<script>` normal, **detiene** la construcción del DOM, descarga el archivo, lo ejecuta y solo entonces continúa. Esto tiene dos efectos: la página tarda más en aparecer, y el script no puede encontrar elementos que estén **más abajo** en el HTML (todavía no existen en el DOM).

```
<script src="app.js">          HTML: ████████░░░░░░░░░░████████
(en el <head>)                 JS:           ▓▓descarga▓▓ejecuta
                                             ↑ el análisis del HTML se detiene

<script src="app.js" defer>    HTML: ██████████████████████████
                               JS:   ▓▓descarga▓▓        ejecuta (al terminar el HTML, en orden)

<script src="app.js" async>    HTML: ██████████░░░░░░██████████
                               JS:   ▓▓descarga▓▓ejecuta (en cuanto llega, sin orden garantizado)
```

| Opción | Bloquea el HTML | Orden entre scripts | Cuándo usarla |
|---|---|---|---|
| Sin atributo | Sí | Sí | Casi nunca; si se usa, al final del `<body>` |
| `defer` | No | Sí, se respeta | **Opción por defecto** para scripts que manipulan la página |
| `async` | Solo mientras se ejecuta | No | Scripts independientes (estadísticas, publicidad) |

**Errores y malentendidos típicos:**

- **`Cannot read properties of null` al usar `querySelector`:** el script se ejecutó antes de que el elemento existiera en el DOM. Solución: usar `defer` o colocar el script al final del `<body>`.
- **Validar solo con JavaScript:** el usuario puede desactivar JavaScript o enviar la petición directamente con otras herramientas. La validación en el cliente es para **comodidad** del usuario; la validación en el servidor (Laravel, Unidad 3) es la que garantiza **seguridad**.
- **Esperar que una función asíncrona devuelva el dato inmediatamente:** `const datos = fetch(url);` no contiene los datos, sino una **promesa**. Hay que esperarla con `await` (dentro de una función `async`) o con `.then()`.

---

## 1.4. Diseño responsivo

**Qué es:** el **diseño responsivo** (*responsive web design*) es la técnica de construir **una sola página** que adapta su disposición, tamaños e imágenes al dispositivo que la muestra: celular, tableta, laptop o pantalla grande.

**Por qué existe:** desde mediados de la década de 2010 la mayor parte del tráfico web proviene de dispositivos móviles. Antes, la solución era mantener **dos sitios separados** (`www.sitio.com` y `m.sitio.com`), lo que duplicaba el trabajo y producía versiones desincronizadas. Con el diseño responsivo, el mismo HTML se reorganiza con CSS. Además, los buscadores indexan primero la versión móvil de las páginas, así que un sitio que no funcione bien en celular queda en desventaja.

**Características:**

| Característica | Descripción |
|---|---|
| **Una sola fuente** | El mismo HTML y la misma URL para todos los dispositivos; solo cambia el CSS aplicado |
| **Rejillas fluidas** | Anchos en `%`, `fr`, `rem` en lugar de píxeles fijos, para que el contenido se estire o encoja |
| **Imágenes flexibles** | Las imágenes nunca superan el ancho de su contenedor (`max-width: 100%`) |
| **Media queries** | Reglas CSS que solo se aplican si el dispositivo cumple ciertas condiciones (ancho, orientación, etc.) |
| **Viewport configurado** | La etiqueta `<meta name="viewport">` hace que el celular use su ancho real |

### 1.4.1. Media queries y buenas prácticas

**Qué es:** una **media query** es un bloque de CSS condicional: sus reglas solo se aplican cuando el dispositivo o la ventana cumple la condición indicada.

```css
@media (min-width: 768px) {          /* condición: la ventana mide 768px de ancho o más */
    .menu { flex-direction: row; }   /* reglas que solo aplican si se cumple */
}
```

**Paso previo indispensable — la etiqueta viewport:** sin esta línea, los navegadores móviles **simulan** una pantalla de escritorio de unos 980 px de ancho y luego encogen todo para que quepa. El resultado es texto diminuto y, lo más importante, las media queries **no se activan**, porque el navegador "cree" que mide 980 px.

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

```
SIN viewport                         CON viewport
┌─────────┐                          ┌─────────┐
│▫▫▫▫▫▫▫▫▫│  la página de escritorio │ Título  │  la página usa el ancho
│▫▫▫▫▫▫▫▫▫│  completa, encogida;     │ Texto   │  real del celular (p. ej. 390px)
│▫▫▫▫▫▫▫▫▫│  hay que hacer zoom      │ legible │  y se aplican las media queries
└─────────┘                          └─────────┘
```

**Cómo funciona por dentro — *mobile first* con `min-width`:** la estrategia recomendada es escribir primero los estilos para **pantallas pequeñas** (sin media query) y luego **agregar** cambios para pantallas más grandes con `min-width`. Como las reglas posteriores con la misma especificidad ganan (cascada, 1.3.2), cada media query va sobrescribiendo lo anterior conforme la pantalla crece:

```css
/* 1. Base (celular): todo en una columna */
.productos { display: grid; grid-template-columns: 1fr; gap: 1rem; }

/* 2. Tableta: dos columnas */
@media (min-width: 768px) {
    .productos { grid-template-columns: repeat(2, 1fr); }
}

/* 3. Escritorio: cuatro columnas */
@media (min-width: 1024px) {
    .productos { grid-template-columns: repeat(4, 1fr); }
}
```

**Traza — qué columnas se ven según el ancho de la ventana:**

| Ancho de la ventana | ¿`min-width: 768px`? | ¿`min-width: 1024px`? | Regla que gana | Columnas |
|---|---|---|---|---|
| 375 px (celular) | No | No | Base | 1 |
| 767 px | No | No | Base | 1 |
| 768 px | **Sí** | No | Tableta | 2 |
| 1023 px | Sí | No | Tableta | 2 |
| 1024 px | Sí | **Sí** | Escritorio (aparece después en el código) | 4 |
| 1440 px | Sí | Sí | Escritorio | 4 |

> **Ojo con el orden:** con `min-width`, las media queries deben ir de **menor a mayor**. Si la de 768 px se escribiera después de la de 1024 px, en una ventana de 1440 px ambas se cumplirían y ganaría la de 768 px por aparecer al final: el escritorio mostraría 2 columnas en vez de 4.

**Mobile first (`min-width`) vs desktop first (`max-width`):**

| Aspecto | **Mobile first** (`min-width`) | **Desktop first** (`max-width`) |
|---|---|---|
| Estilos base pensados para | Celular | Escritorio |
| Las media queries… | **Agregan** complejidad para pantallas grandes | **Quitan** o simplifican para pantallas pequeñas |
| Qué estilos aplica un celular | Solo los estilos base; las media queries de pantallas grandes no se cumplen y se ignoran | Todo el diseño de escritorio, y encima las correcciones que lo deshacen |
| Obliga a priorizar contenido | Sí: en el celular solo cabe lo esencial | No |
| Recomendación | **Preferida** en proyectos nuevos | Útil al adaptar un sitio de escritorio ya existente |

**Puntos de quiebre (*breakpoints*) comunes:**

| Nombre habitual | Ancho mínimo | Dispositivos típicos |
|---|---|---|
| Base (sin media query) | 0 | Celulares en vertical |
| `sm` | 576 px / 640 px | Celulares grandes, en horizontal |
| `md` | 768 px | Tabletas |
| `lg` | 1024 px | Laptops, tabletas en horizontal |
| `xl` | 1280 px | Escritorio |

> **Por qué estos números no son reglas fijas:** existen cientos de tamaños de pantalla distintos. La buena práctica es colocar un *breakpoint* **donde el diseño se rompe** (cuando el texto queda demasiado ancho o las columnas demasiado apretadas), no donde está "el ancho del iPhone". Los valores de la tabla son convenciones de frameworks como Bootstrap y Tailwind.

**Otras condiciones útiles en media queries:**

```css
@media (orientation: landscape) { ... }              /* dispositivo en horizontal */
@media (min-width: 768px) and (max-width: 1023px) { ... } /* solo en un rango */
@media (prefers-color-scheme: dark) { ... }          /* el usuario activó el modo oscuro */
@media (prefers-reduced-motion: reduce) { ... }      /* el usuario pidió menos animaciones */
@media print { ... }                                 /* al imprimir la página */
@media (hover: hover) { ... }                        /* el dispositivo tiene puntero que puede "pasar encima" (mouse) */
```

**Buenas prácticas:**

| Práctica | Cómo se aplica | Por qué |
|---|---|---|
| Imágenes flexibles | `img { max-width: 100%; height: auto; }` | Evita que una imagen de 1200 px desborde una pantalla de 390 px; `height: auto` mantiene la proporción |
| Tamaños relativos | `rem` para texto, `%`/`fr` para anchos | Respetan la configuración de tamaño de letra del usuario y se adaptan al contenedor |
| Evitar anchos fijos grandes | `max-width: 1200px; width: 100%;` en vez de `width: 1200px;` | Un ancho fijo provoca *scroll* horizontal en pantallas más pequeñas |
| Áreas táctiles suficientes | Botones y enlaces de al menos ~44×44 px | El dedo es mucho menos preciso que el puntero del mouse |
| No depender de `:hover` | Toda acción importante debe funcionar con un toque | En pantallas táctiles no existe "pasar el mouse por encima" |
| Imágenes según el dispositivo | `<img srcset="foto-400.jpg 400w, foto-1200.jpg 1200w" sizes="100vw">` | El celular descarga la versión ligera; ahorra datos y tiempo |
| Probar en dispositivos reales y en el modo responsivo | F12 → icono de dispositivo en las herramientas de desarrollador | El simulador ayuda, pero no reproduce el rendimiento ni el tacto real |

**Árbol de decisión — ¿media query o diseño intrínsecamente flexible?**

```
¿Necesitas que el diseño cambie según el ancho?
│
├─ ¿Solo quieres que los elementos se acomoden en tantas columnas como quepan?
│     └─ Sí ──▶ Grid con repeat(auto-fill, minmax(200px, 1fr)) o Flexbox con flex-wrap
│               (no necesitas ninguna media query)
│
├─ ¿Cambia la ESTRUCTURA (menú hamburguesa vs barra, ocultar una barra lateral)?
│     └─ Sí ──▶ Media query
│
└─ ¿El cambio depende del tamaño del CONTENEDOR y no de la ventana
   (una tarjeta que puede estar en una barra lateral angosta o en el área principal)?
      └─ Sí ──▶ Container query (@container), soportada en los navegadores modernos
```

**Errores y malentendidos típicos:**

- **Olvidar la etiqueta viewport y pensar que las media queries "no funcionan":** es el error más frecuente; revisa el `<head>` antes que el CSS.
- **Diseñar por dispositivo concreto** ("la versión de iPhone", "la versión de iPad"): el próximo modelo tendrá otro tamaño. Diseña por **rangos de ancho** donde el contenido lo necesite.
- **Ocultar contenido importante en móvil con `display: none`:** el usuario móvil suele necesitar la misma información; además, el contenido oculto **se sigue descargando**.
- **Media queries que se solapan o dejan huecos:** usar `max-width: 768px` en una y `min-width: 768px` en otra hace que **ambas** se apliquen exactamente en 768 px. Con *mobile first* y solo `min-width`, ese problema no aparece.
- **Probar únicamente estrechando la ventana del navegador de escritorio:** no reproduce la densidad de píxeles, el teclado virtual que reduce la altura visible, ni la imposibilidad de usar `:hover`.

---

*Universidad Politécnica de Tecamac — Ingeniería en Software*
