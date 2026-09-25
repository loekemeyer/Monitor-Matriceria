# Monitor Matricería — Instrucciones para Claude Code

> **Nombre del repo:** nació como `loekemeyer/Monitor-Virgilio` por error del dueño y el
> 16/09/2026 se renombró a **`Monitor-Matriceria`**. GitHub redirige la URL vieja, así que
> los `git remote` que todavía apunten al nombre viejo siguen funcionando.

## Qué es

Un **visualizador** para la TV de 32" del taller: **un renglón por matriz**, con
**Ingreso · Descripción · Estado · Problema · Espera**, en el **formato de cuadro sinóptico** de la casa.
**La TV está colgada de costado**, así que la página se dibuja girada 90° y en modo lista.
Sale de `planify.v_matriceria_monitor`.

⚠ **El número de matriz NO se muestra** (Elías, 23/09/2026). Si lo cargaron, se muestra la
**descripción** que trae el maestro; si escribieron texto libre —matriz experimental, muestra,
recién hecha, que no está en `public."Matrices"`— ese texto **es** la descripción. El número
se sigue usando **en Planify**, para que la misma matriz no termine cargada con una
descripción distinta cada vez. Hoy 6 de las 10 matrices del taller no tienen número.

⚠ **Acá NO va el contador de "unidades sin accidente".** Estuvo como segundo tablero unas
horas del 16/09/2026 y el dueño lo sacó ese mismo día: *"eso no va acá… en matricería
solamente un monitor fijo de las matrices que hay en el taller"*. El dato existe y se
mantiene solo (`GP2.matriz_racha`), pero se mira **únicamente** en Gestión Productiva 2.0 →
Producción → *Unidades sin Accidente*. Si alguien pide "poner las unidades en la TV", es una
decisión que ya se tomó al revés: preguntar antes de volver a agregarlo.

**No carga NI modifica nada.** Los datos los cargan Martín Pregelj (employee_id 15) y
Martín Cornejo (34) desde Planify → módulo **"🛠️ Matricería"**, que escribe en
`planify.matrices_ingresos` del proyecto Supabase `hrxfctzncixxqmpfhskv`. Este repo sólo
LEE la vista `planify.v_matriceria_monitor`.

⚠ **El contrato es la VISTA, no la tabla.** La vista ya expone las columnas que sumó el
módulo unificado (`tarea`, `tarea_realizada`, `hs`, `quien`, `salida_estimada`, `foto_path`,
`nombre_matriz`) y el monitor pide `select=*`, así que **una columna nueva en la vista llega
sola**; lo que hay que tocar acá es sólo el dibujo de la tarjeta (`tarjeta()` en
`index.html`). Si alguien agrega un campo en Planify y NO lo agrega a la vista, la TV no lo
ve nunca.

Que sea de solo lectura no es una convención, está impuesto por la base: la clave
publishable que viaja en `index.html` tiene **únicamente SELECT** sobre esa tabla (los
writes de Planify van por RPC `SECURITY DEFINER` `planify_matriz_*`). Aunque alguien abra
la consola del navegador en la TV, no puede escribir ni borrar una matriz.

## Cómo funciona

- **Un solo archivo**: `index.html`. Sin build, sin npm, sin dependencias. Se abre con
  doble clic (`file://`) o publicado por GitHub Pages; anda igual.
- Pide los datos cada **30 s** (`fetch` a PostgREST con el header `Accept-Profile: planify`).
- **Todas las matrices entran en una sola pantalla. No hay carrusel** (Elías, 23/09/2026:
  *"no hacer un carrusel, achicar la pantalla en base a la cantidad de matrices activas para
  mostrar todas"*). El tamaño de letra sale de cuántos renglones hay: `fs = alto / (n × 1,85
  em + encabezado)`, y una segunda pasada lo achica más si la descripción más larga no entra
  a lo ancho. Con 10 matrices en 962 × 485 da 18,8 px — el doble de lo que se leía cuando
  eran tarjetas.
- **El formato es el del cuadro sinóptico**, y se aplica entero (Elías: *"no excluyas ni
  omitas ninguna regla"*):
  - **Ancho de columna según el DATO, no según el título**: `ajustarColumnas()` mide lo que
    ocupan de verdad las celdas y deja la columna del ancho del valor más largo. El título,
    si no entra, se parte en dos líneas (`.th span` va con `white-space:normal`).
  - **Título más grande que el contenido** (16 contra 14): `.th span` va a `1.14em`.
  - **Centrado horizontal y vertical**, en las celdas y en la tabla entera.
  - **Sin ancho fijo y sin columnas vacías**: la tabla es `width:auto`, queda del ancho de
    sus datos y el `<main>` la centra. Columnas próximas, sin ancho sobrante.
  - ⚠ **El "sin color / sin relleno" del formato NO se aplica acá**, por decisión de Elías
    (23/09/2026): *"en este caso por el color y el relleno seguí la estética de la página"*.
    Por eso el renglón conserva el fondo, la barra del estado a la izquierda y el chip de
    color. Es una excepción **pedida**, no un olvido: el resto del formato se cumple tal cual.
  - ⚠ **Todo el TEXTO del renglón va en blanco (`#e6edf3`) y sin negrita** — `#`, Ing.,
    Descripción, Problema y Espera (Elías, 23/09/2026: *"ese cambio de texto hacelo a desc ing
    # problema y espera, sin el bold"*). El único que conserva color y negrita es el **chip**
    de Estado. **Esto sacó la señal de demora**: hasta esa versión la fecha se pintaba ámbar a
    los 3 días y roja a los 7, y era lo ÚNICO que la mostraba (la barra de la izquierda es del
    estado, no de la demora). Hoy el orden lo dice la columna `#`, que ordenan Pregelj y
    Cornejo. Las clases `.aviso` y `.alerta` se siguen poniendo en `fila()` pero ya no pintan
    nada: ahí está el lugar si alguna vez hace falta la señal de vuelta.
  - **Ordenado por gravedad, de mayor a menor**: lo trae la consulta, por `dias_demora desc`.
  - **Tabla sólo con 3 filas o más**; con menos va como lista, sin encabezado (`conTitulos`).
- **Problema y "qué se espera" en pantalla** (Elías, 23/09/2026: *"que el problema y el texto
  de esperando se vean también en la TV"*). El `motivo` del módulo es la columna
  **Problema**, entre Descripción y Estado, en un gris más apagado: lo primero que se busca
  en la TV es QUÉ matriz es, y después por qué está. Si está vacío se dibuja un `—`, para que
  la columna no quede en blanco. El `espera` va al lado del chip **sólo** cuando el estado es
  `esperando`. Los dos ya viajaban en la vista y el monitor pide `select=*`: no hubo que tocar
  la base.
  ⚠ **El tope en em de cada texto es lo que protege el tamaño de letra.** Como la letra sale
  de dividir el alto por los renglones y después se achica si la tabla no entra a lo ancho, UN
  motivo largo le bajaba la letra a los 11 renglones. Por eso `.c-desc span` corta a 18 em,
  `.c-prob span` a 16 y `.c-estado em` a 12, con puntos suspensivos. Medido a 962×485 con las
  11 matrices: 835 px de tabla y **16,5 px de letra**, igual que sin la columna; con un motivo
  y una espera largos a propósito, 888 px y **los mismos 16,5 px**.
  **Todo va en UNA línea; el texto que no entra se parte en dos** (Elías, 23/09/2026:
  *"misma línea, que sea 2 líneas en caso de sobrepasar el límite de texto"*). Nada se corta
  con puntos suspensivos mientras quepa en dos renglones: `.c-desc span`, `.c-prob span` y
  `.c-prob em` van con `-webkit-line-clamp:2`. Recién al tercero corta, porque si no un texto
  enorme desarma la pantalla. Con los textos de hoy no se parte ninguno: la segunda línea es
  la salida de emergencia, no el caso normal.
  **El orden es `#` · Ing. · Descripción · ESTADO · Problema · Espera** (Elías, 23/09/2026:
  *"problema y espera van después de estado"* y *"problema y espera separados, 1ro
  problema"*). Cada una en su columna: así la de Problema mide sólo los problemas y no se
  ensancha por la espera más larga. Eso resuelve de paso el chip descolocado — la celda de
  Estado queda con el chip y nada más, así que mide lo que mide un chip y los once quedan
  alineados. El texto de la espera va en blanco como todo el resto (Elías: *"la letra en
  blanco no rojo"*): el rojo ya lo pone el chip de al lado y repetirlo no agregaba nada.
  ⚠ **La columna Espera sólo se dibuja si alguna matriz espera algo** (`conEspera` en
  `pintar()`, que se le pasa a `fila()`). Sin eso quedaría una columna entera vacía, que es
  justo lo que el formato prohíbe; y ése es el caso normal, porque casi nunca hay una
  esperando. Con cero esperas la tabla mide 682 px; con una, 865; con tres, 877.
  ⚠ `ajustarColumnas()` mide la **celda entera**, no sus hijos: la del problema lleva dos
  cosas al lado y midiendo hijo por hijo se tomaría el más ancho en vez de la suma.
  ⚠ Cuántos renglones se parten en dos **no se puede saber antes de dibujar**, así que la
  primera estimación de `pintar()` asume una línea por matriz y la **segunda pasada mide la
  caja real —ancho Y alto— y corrige**. Medido a 962×485 con las 11 matrices: **16,5 px** de
  letra sin esperas, con una y con tres; con un problema y una espera largos a propósito la
  tabla llega justo a los 924 px disponibles y la letra baja a **16,2 px**.
- ⚠ **PANTALLA VERTICAL: la página se dibuja GIRADA 90° y en modo LISTA** (Elías,
  23/09/2026: *"poneme el monitor de TV en 90°, dimos vuelta la TV para que sea una lista a
  lo largo"*). La TV se colgó de costado pero el aparato que la maneja **sigue mandando la
  imagen en horizontal**: la única forma de enderezarla es rotar la página. Todo el contenido
  vive dentro de `#rot`, que en modo giro se arma al revés (`width:100vh; height:100vw`) y se
  rota con un `translate` que lo trae de vuelta al área visible.
  - **`?giro=90`** es el default, **confirmado en la pantalla real** (Elías, 23/09/2026:
    *"estaba bien girado"*). Se probó el 270 y era el equivocado, así que **este número ya no
    se toca sin mirar la TV**: desde el código no hay forma de saber para qué lado la
    colgaron, el aparato no lo reporta. · **`?giro=270`** la gira para el otro lado ·
    **`?giro=0`** la deja horizontal, que es como conviene mirarla desde una PC.
  - **Girada NO entra la tabla de seis columnas.** Medido: el área pasa de 924×396 a
    **446,5×838,5 px** y la letra caía a **10,2 px**, ilegible desde el taller. Por eso en
    vertical cada matriz ocupa **dos renglones y no hay encabezado** — que es literalmente
    "una lista a lo largo": arriba puesto · descripción · chip de estado, abajo fecha ·
    problema · qué espera. El alto sobra, así que gastarlo en un segundo renglón sale gratis:
    **18,6 px de letra contra los 16,5 de la horizontal**.
  - **Para mirarlo desde una PC hay `?vista=1`** (Elías, 25/09/2026: *"¿hay forma de girarlo
    para verlo como se estaría viendo la TV?"*). Girada, el navegador muestra el texto de
    costado —que es correcto: así sale el video, y la TV está colgada de costado— pero es
    ilegible en el monitor de un escritorio. `?giro=90&vista=1` lo endereza. **Lo importante
    es lo que NO cambia**: `#rot` sigue siendo `100vh × 100vw` y todos los `vw`/`vh` se
    siguen calculando contra la misma ventana, así que el layout y el tamaño de letra salen
    **idénticos** a los de la TV (verificado: `#rot` 485×962, tabla 447×695, letra 18,59 px
    en los dos). Lo único que se reemplaza es el `transform`: en vez de rotar, achica para
    que la caja parada entre en la ventana acostada (a 962×485 la escala es **0,504**) y la
    centra. **Es una lupa para mirar, no un modo de la TV** — la TV no lo usa nunca.
    Con `?giro=0` se ignora, porque sin giro la página ya se lee derecha.
    ⚠ La regla CSS de `.vista` va **después** de las de `giro90`/`giro270`: tienen la misma
    especificidad, así que lo que decide es el orden. Si alguien la mueve arriba, deja de
    funcionar sin tirar ningún error.
  - ⚠ **`getBoundingClientRect()` MIENTE con la pantalla girada**: devuelve la caja alineada a
    la pantalla, o sea con el ancho y el alto **cambiados de lugar**. La segunda pasada de
    `pintar()` hacía la corrección con los números al revés y dejaba la letra en 13,2 px por
    un motivo inventado. Ahora usa **`offsetWidth`/`offsetHeight`**, que son medidas de
    maquetado y el `transform` del padre no las toca (`clientWidth`/`clientHeight`, que ya se
    usaban para el área, nunca tuvieron el problema por lo mismo). **Si mañana se agrega una
    medición acá, que no sea con `getBoundingClientRect()`.**
- **Abreviaturas de la descripción** (`ABREVIATURAS`): hoy sólo `corte` → `C/`, que es la
  palabra que más se repite en el maestro. Sumar otra es agregar un par a esa lista.
- Muestra sólo lo que está EN el taller (`estado != terminada`). Cuando la matriz se
  termina —o se cierra su tarea "Tratamiento matriz …" en Planify— desaparece sola.
- **Cinco estados**: `ingresada` → Ingresado · `en_proceso` → Proceso · `ver_damian` →
  Ver Damián · `esperando` → Esperando · `terminada` → Terminada (no se muestra). Agregar uno
  toca CUATRO lugares: el `check` de `planify.matrices_ingresos`, la RPC
  `planify_matriz_estado`, la lista `MAT_ESTADOS` de Planify y el objeto `ESTADOS` de acá.
- ⚠⚠ **EN ESTE ARCHIVO NO SE ESCRIBE `gap`. El navegador de la TV no lo soporta.**
  El aparato tipo Roku del taller **descarta el `gap` de flexbox en silencio** — sin error y
  sin fallback: las cosas quedan **pegadas**. En la foto del 25/09/2026 se leía
  `MATRICERIA12 matrices`, `7Bombilla nueva` y `23/09Reponer manoplas`, y en Chrome de una PC
  se veía perfecto, que es por lo que tardó en encontrarse. **Reproducido**: sirviendo el
  `index.html` real con el `gap` neutralizado, las tres separaciones dan **0 px**; con
  `margin` dan **9 / 8 / 13 px**, y en Chrome normal el resultado es **idéntico** (el total es
  el mismo, `.55em × (n−1)` de un lado o del otro, así que ningún ancho medido cambia).
  Para separar va **`> * + *` con `margin`**, nunca `gap`. Hoy el archivo tiene **cero**.
- ⚠ **El chip de estado no se achica nunca** (`flex:0 0 auto` en `.c-estado i, .lista .l1 i`)
  y la descripción de la lista tiene un tope (`max-width:calc(100% - 9em)` en `.lista .l1 .d`).
  **Honestidad sobre estos dos frenos**: el corte que se vio en la TV (`INGRESAI`, `PROCES`,
  `ESPERANI`) **NO se pudo reproducir en Chromium** — acá la descripción cede primero y el
  chip sale entero siempre, medido con los textos de hoy y con descripciones largas a
  propósito (**0 de 12 cortados en los cuatro casos**). O sea que **no son un arreglo
  comprobado**: son el único mecanismo capaz de producir ese corte (el chip era un item
  flexible más, y si el navegador no respeta el `min-width:0` de la descripción, ésta empuja
  al chip fuera del renglón y el `overflow:hidden` de `.fila` se lo come). **Hay que mirar la
  TV con la 2026-09-25.30 para saber si alcanzó.**
- ⚠ **La letra de la lista está topeada por el ANCHO, no por el alto.** Medido a 962×485 con
  las 12 matrices: área útil 447 × 838, tabla 447 × 695, letra **18,6 px**; sobra ×0,999 a lo
  ancho y ×1,206 a lo alto. O sea que **los ~143 px que sobran abajo NO se pueden convertir en
  letra más grande**: al primer píxel que crezca, no entra a lo ancho. Si alguna vez hace falta
  más letra, hay que acortar los textos (más abreviaturas) o mandar el problema a un tercer
  renglón — subir la constante `EM_ITEM` no sirve. Y la segunda pasada de `pintar()` **sólo
  achica** (`if (sobra < 1)`), a propósito: dejarla crecer puede partir un texto en dos líneas,
  volver a no entrar y terminar cortando el último renglón en una pantalla que no vemos.
- ⚠ **`<meta name="monitor-version">` HAY QUE SUBIRLA EN CADA COMMIT que toque
  `index.html`.** La página baja cada 10 minutos el `index.html` publicado en GitHub Pages
  (sin caché), le lee esa meta y la compara con la de la copia que está corriendo. Si no
  coinciden, saca un **cartel rojo arriba de todo** y —salvo `?autorecarga=0`— se recarga
  sola a los 4 s con `?_v=<version>` para pisar la caché del navegador de la TV. Si ya se
  recargó por esa versión y sigue vieja (caché que no suelta), NO insiste: deja el cartel
  pidiendo recarga a mano. Sin subir la meta, el aviso no salta nunca y la TV puede quedar
  semanas con una copia vieja — que es exactamente lo que pasó el 23/09/2026, dos veces en
  la misma tarde.
  **Chequea a los 15 s de abrir y después cada 2 minutos.** Arrancó en 60 s / 10 minutos y
  esa espera de hasta diez minutos es lo que hizo decir a Elías que *"no funcionó lo del
  mensaje en rojo"*: el mecanismo andaba —se reprodujo contra un servidor real, 6 de 6
  checks— pero entre publicar y que la TV se entere pasaba un rato largo. El costo de
  chequear es UN GET del propio `index.html` (~30 KB); a 2 minutos son 30 por hora, contra
  las 120 consultas a Supabase que la pantalla ya hace en ese mismo rato.
  **La versión que corre se ve en el pie**, abajo a la derecha (`#ver`), y al lado **qué pasó
  en el último chequeo** (`diag()`): `sin chequear todavía` · `al día 17:43` ·
  `hay 2026-09-23.99 17:43` · `HTTP 404 17:43` · `sin meta en lo publicado` ·
  `fallo en fetch: <motivo>`. **Existe porque este aviso ya falló dos veces sin dejar rastro**:
  el `catch` se comía el error y desde el taller no se podía distinguir *"el chequeo no corre"*
  de *"corre y dice que está al día"* o de *"la red lo rechaza"*. Ahora la TV lo cuenta sola y
  se lee de la misma pantalla — es el único camino, porque a esa TV no se le puede abrir la
  consola.
  ⚠ El fetch pide **`index.html?_v=…`** por su nombre explícito, no `location.pathname`: si la
  TV quedó en una URL con parámetros o sin la barra final, `pathname` no siempre apunta al
  html.
- ⚠ **La TV del taller NO es una PC**: entra por un aparato tipo **Roku**, con control
  remoto, sin F11 y sin manera de sacar la barra del navegador. Informa **ventana 962 × 485**,
  pantalla 962 × 541 y escala 1,33 (panel real 1280 × 720). **Al probar un cambio hay que
  mirarlo a 962 × 485**, que es la medida real del taller, no a 1920 × 1080.
- **Lector de resolución** al lado del reloj (`pintarRes()`): **apagado por defecto**, se
  prende con `?res=1`. Muestra ventana, pantalla y escala; sirve para medir una pantalla
  nueva antes de tocar el CSS.
- Si se cae la red, **deja lo último que se vio en pantalla** y avisa `SIN CONEXIÓN` abajo
  a la izquierda. Una TV en blanco no le sirve a nadie.
- **Lo que ya NO se muestra** (estuvo unas horas el 23/09/2026 y lo sacó el formato de una
  línea): número de matriz, tarea a realizar, ya hecho, quién, HS, salida estimada y la foto.
  El **problema** (`motivo`) volvió el 23/09 como columna propia, ver más arriba.
  El bucket `planify_matriceria` sigue existiendo y Planify sigue guardando fotos; la TV
  simplemente no las pide.

### Parámetros por URL (opcionales)

| Parámetro | Default | Qué hace |
|---|---|---|
| `refresco` | 30 | Segundos entre consultas a Supabase |
| `res` | 0 | `res=1` muestra el lector de resolución (apagado por defecto) |
| `autorecarga` | 1 | `autorecarga=0` deja el cartel rojo pero no recarga sola |
| `giro` | 90 | Giro de la pantalla: `90` (la TV del taller, confirmado), `270` (al revés), `0` (horizontal) |
| `vista` | 0 | `vista=1` endereza la pantalla girada para mirarla desde una PC (misma letra y mismo layout, achicado para entrar). No lo usa la TV |

La grilla ya no se elige a mano: el cuadro ocupa lo que necesita y se centra.

## Estados

`ingresada` (azul) · `en_proceso` (amarillo) · `esperando` repuesto/material (rojo) ·
`terminada` (verde, no se muestra en la TV). Se cambian desde Planify, no desde acá.
Para agregar un estado nuevo hay que tocar el `check` de `planify.matrices_ingresos`,
la lista `MAT_ESTADOS` de Planify (`src/index.html`) y el objeto `ESTADOS` de este repo.

## Dónde está la otra mitad

Repo **`loekemeyer/Planify`**: el módulo "Matricería" (`src/index.html`,
`supabase.js`, `main.js`, `preload.js`) y la migración
`supabase/20260916_matriceria_ingresos.sql`, que es la fuente de verdad del schema.
Repo **`loekemeyer/Gestion-Productiva-2.0`**: la pantalla *Unidades sin Accidente*
(`Produccion/UnidadesSinAccidente/`) y el detalle de `GP2.matriz_racha` — ese dato vive
allá, no acá.
Si cambia el nombre de la vista o de una columna allá, este monitor deja de ver las
matrices: son dos repos, pero un solo contrato.

## ⚠ REGLA: preguntar QUIÉN habla y dejar cada pedido como tarea en su Planify

**Vale para TODOS los repos** (LK, Gestión Virgilio, Planify y cualquiera nuevo: copiar este
bloque al `CLAUDE.md` del repo nuevo). Objetivo del dueño: que ninguna tarea quede a medio
hacer sin figurar en la agenda de alguien.

1. **Al empezar la sesión, preguntar quién está hablando** (antes de hacer nada):
   *"¿Quién sos? (Thomas, Marianela, Luis, Gastón, …)"*. Si el mensaje ya lo dice, no repreguntar.
2. **Cada pedido de trabajo se registra como tarea en el Planify de esa persona**, apenas se
   empieza, con nombre MUY resumido (≤ 60 caracteres). Queda `done=false` hasta que se cierre
   (punto 4). Si la sesión termina sin cerrar, la tarea queda en la agenda: ése es el objetivo.

   **La nota (comentario) lleva SIEMPRE estas tres cosas, en este orden y conciso** (dueño,
   2026-09-11: *"en comentarios tiene que explicar conciso qué es lo que falta y quién le creó
   la tarea y desde qué sesión de Claude"*):
   1. **Qué falta**: qué hay que hacer, concreto y accionable — no el historial de lo ya hecho.
      Si algo ya se hizo, va en una línea aparte al final ("Ya hecho: …").
   2. **Quién la pidió**: el nombre de la persona que lo pidió en el chat (Thomas, Marianela, …).
   3. **De qué sesión salió**: la URL de esta sesión de Claude, para poder ir a leer la charla.

   Formato:
   `Falta: <qué hay que hacer>. Pedido de <Nombre> · cargada por Claude, sesión <url>`

   Ejemplo real: `Falta: cargar el secreto KRIKOS_IMAP_PASS en el Vault de Supabase LK
   (kwkclwhmoygunqmlegrg); sin eso krikos-ingest no lee la casilla y la Bandeja de OC queda
   vacía. Pedido de Thomas · cargada por Claude, sesión https://claude.ai/code/session_XXXX`

   **Al cerrar o actualizar la tarea, la nota se reescribe con lo que quedó pendiente**, no se
   le agrega texto encima: quien la lee tiene que ver de un vistazo qué falta hoy.
3. **Excepción del dueño:** Thomas Loekemeyer NO usa Planify. Sus pedidos se cargan con el
   nombre antepuesto por **`Th `** (ej. `Th Fecha estimada de entrega por zona`) en el Planify
   de **quien corresponda según el área del pedido** — dueño, 2026-09-12: *"no mandes todo a
   Tomás, mandá al que correspondan"*. El `Th ` queda igual: sirve para saber que el pedido
   salió de él.

   Para resolver a quién va: mirar de qué área es el tema y buscar el responsable en
   `procesos.responsables` / `procesos.areas_de_empleado()` (ver el bloque de procesos más
   abajo). Ejemplos reales: importaciones y cuenta NTL → Viviana Gauna (Comercio Exterior);
   armado de tandas, PPP y carga de camión → Marianela Becker (Ventas); recepción de mercadería
   e insumos → Alan Gonzalez (Logística); temas técnicos de Supabase → Elías Irace (IT).

   **Sólo va al Planify de Tomás Beviglia (employee_id 20)** lo que es de él, lo transversal
   (no pertenece a ningún área) o lo que no tiene un responsable claro. Antes iba TODO ahí y
   por eso su agenda juntó 92 pedidos que no eran suyos.

**Dónde:** proyecto Supabase de Gestión Virgilio `hrxfctzncixxqmpfhskv`, schema `planify`.
Empleados activos con Planify (`planify.employees`): Marianela Becker **38**, Luis Rial Otero
**52**, Gastón Dalponte **61**, Tomás Beviglia **20**, Gonzalez Tomas 16, Elías Irace 1,
Nazareno Rodríguez 27, Angely Asuaje 22, Viviana Gauna 4, Alan Gonzalez 5, Diego Mollo 44,
Nora Heredia 33, Juan Cruz Karaygan 51, Pablo Martos 6, Martín Cornejo 34, Martín Pregelj 15,
Romina Maturano 55, Iván Meta 58, Jhonny Cartaya 46. Si el nombre no está, buscar:
`select id, nombre from planify.employees where activo and nombre ilike '%<apellido>%'`.

```sql
-- alta (al empezar el pedido)
insert into planify.tasks (name, type, prio, time, date, note, rec, done, assignment_type,
  employee_id, department_id, system_generated, broadcast, created_at, updated_at)
values ('<resumen ≤60>', 'tarea', 'normal', '09:00', to_char(now() at time zone
  'America/Argentina/Buenos_Aires', 'YYYY-MM-DD'),
  'Falta: <qué hay que hacer, concreto>. Pedido de <Nombre> · cargada por Claude, sesión
  <url de ESTA sesión>', 'none', false, 'employee', <employee_id>, null, false, false, now(), now())
returning id;
-- cierre (cuando la persona la da por terminada)
update planify.tasks set done = true, updated_at = now() where id = <id>;
```

Avisar en el chat el `id` al crearla y al cerrarla. No crear tareas para preguntas o consultas
que se responden en el momento; sólo para pedidos que implican hacer algo.

4. **Cierre por criterio propio y SIN preguntar** (dueño, 2026-09-11: *"las que ya están
   cerradas, cerradas"*). Claude evalúa **solo** si el objetivo del pedido se cumplió (lo
   entregado funciona, está commiteado/pusheado/aplicado, y no quedó ninguna parte del
   pedido sin hacer). Si se cumplió: `done=true` y lo avisa en el chat. **NO** se pregunta
   "¿falta algo más para dar por cerrada la tarea?" — esa pregunta queda prohibida. Lo que
   se pidió y quedó a medias NO se cierra: queda abierta con la nota actualizada ("queda
   pendiente: …") y en el chat se dice qué falta y por qué. Si después la persona pide algo
   más sobre esa tarea, se reabre (`done=false`) o se crea una nueva.

5. **Alerta de inactividad (1 hora).** Si hay tareas abiertas de esta sesión y pasa una
   hora sin mensajes, Claude escribe: *"Te estoy registrando estas tareas pendientes:
   … ¿Querés continuar alguna o damos por cerrada la charla?"* Cómo: al terminar un turno
   con tareas abiertas, si la sesión tiene `send_later` (Claude Code web/remoto) o
   `ScheduleWakeup`, armar UN recordatorio a 60 min (borrar el anterior si existía); al
   dispararse, si sigue habiendo tareas abiertas, mandar la alerta; si no, no decir nada.
   En una sesión local sin esas herramientas no hay forma de despertarse sola: en ese
   caso, al cerrar cada turno con tareas abiertas, dejar la lista escrita en el chat.

6. **Propagar la regla a todo repo nuevo.** Si en una charla se agrega o se toca por
   primera vez un repo que NO tiene este bloque en su `CLAUDE.md` (se lo trae de referencia,
   se lo crea, o se le hace un cambio), copiarle este bloque entero (creando el `CLAUDE.md`
   si no existe) y commitearlo en ese repo, avisando en el chat. Así el dueño no tiene que
   pedirlo cada vez. Fuente canónica del bloque: `CLAUDE.md` de `loekemeyer/pagina-LK-copia`.


## ⚠ REGLA: NO preguntar — razonar primero y resolver

**Dueño (2026-09-11): *"no me tenés que preguntar, tenés que razonar primero"*.** Vale para
TODOS los repos (LK, Chef, Gestión Virgilio, Planify y cualquiera nuevo: copiar este bloque
al `CLAUDE.md` del repo nuevo, igual que el de Planify).

Antes de escribirle una pregunta al dueño, **resolverla**: leer el código, consultar la base,
mirar la doc del repo (`GUIA-PROYECTO.md`, `docs/SUPABASE-GESTION-VIRGILIO.md`, los `CLAUDE.md`),
probar. Preguntar es el último recurso, no el primero.

- **Nunca** preguntar algo averiguable: qué tabla es, qué versión corre, si algo ya está hecho,
  qué significa un dato, si el cron lo pisa. Se averigua y se sigue.
- **Nunca** preguntar "¿lo hago?" / "¿querés que…?" sobre lo que ya pidió. Si el pedido se
  entiende, se hace completo.
- **Dos caminos razonables** → elegir el más seguro y reversible (con backup si toca datos),
  hacerlo, y avisar en UNA línea el criterio usado. No se frena la tarea esperando respuesta.
- **Un pedido ambiguo** se interpreta como lo haría alguien que conoce el negocio, mirando las
  reglas del dueño ya escritas en estos archivos. Si quedan dos lecturas con consecuencias muy
  distintas, se hace la reversible y se avisa cuál se tomó.
- **Sí se pregunta y se espera** sólo en tres casos: (a) la acción es destructiva o irreversible
  sobre datos reales (borrar, pisar, mandar algo afuera: mail, WhatsApp, ISIS); (b) dos reglas
  del dueño se contradicen y hay que elegir; (c) falta un dato que no existe en ningún lado
  porque es una decisión comercial suya (un precio, a quién se le vende, una fecha pactada).
- El cierre de tareas de Planify **no se pregunta**: punto 4 del bloque de arriba.

## REGLA: auditar en Supabase cada problema del repo y su solucion

**Vale para TODOS los repos** (igual que la regla de Planify: copiar este bloque al `CLAUDE.md`
de cualquier repo nuevo). Objetivo: que cada error que tuvo un repositorio quede con su causa,
su correccion y el/los commits donde se arreglo, para no volver a pisar el mismo pozo.

**Donde:** proyecto Supabase `hrxfctzncixxqmpfhskv`, schema `github_repo_problemas`.
Se escribe con el MCP de Supabase (`execute_sql`), no con la anon key.

### Que se audita y que NO

Regla corta: **si ya estaba pusheado y andaba mal, se audita.** Si es trabajo nuevo, no.

| Se registra | NO se registra |
|---|---|
| Bug en codigo ya pusheado que llego al usuario | Feature nueva o pedido de cambio |
| Dato corrupto o mal migrado en la base | Refactor pedido por el usuario |
| Config o credencial rota o filtrada | Bug que introducis y arreglas antes de pushear |
| Performance degradada, query que no escala | Duda o consulta que se responde en el momento |
| Tabla derivada desincronizada de su madre | Ajuste de estilo o texto |

### Cuando

1. **Al detectar el problema** (antes de tocar nada): `registrar_problema` devuelve el id.
2. **Al pushear el fix**: `cerrar_problema` con el sha del commit.
3. **Si el fix necesita mas commits**: `agregar_commit` por cada uno. Un problema puede tener N
   commits; NO abrir un problema nuevo por el segundo pase del mismo fix.
4. Una sesion de Claude puede abarcar **varios** problemas: `sesion_id` no es unico.

### SQL

```sql
-- 1) al detectar
select github_repo_problemas.registrar_problema(
  p_repo          => 'owner/repo',            -- en minuscula
  p_titulo        => '<sintoma en <=120 chars>',
  p_descripcion   => '<que se rompio y como se manifesto>',
  p_categoria     => 'bug',                   -- bug|datos|seguridad|performance|config|ux|deuda_tecnica|documentacion
  p_severidad     => 'alto',                  -- critico|alto|medio|bajo
  p_modulo        => 'Carpeta/Modulo',
  p_archivos      => array['ruta/relativa.html'],
  p_sesion_id     => '<id de la sesion de Claude>',
  p_detectado_por => '<usuario> (claude-remote)',
  p_detectado_en  => now()                    -- fecha REAL si es carga historica
);

-- 2) al pushear el fix
select github_repo_problemas.cerrar_problema(
  p_id            => <id>,
  p_correccion    => '<que se cambio>',
  p_commit_sha    => '<sha corto>',
  p_branch        => '<branch>',
  p_commit_url    => 'https://github.com/owner/repo/commit/<sha>',
  p_causa_raiz    => '<por que paso, no que paso>',
  p_corregido_por => '<usuario> (claude-remote)',
  p_mensaje       => '<subject del commit>'
);

-- 3) commits extra del mismo problema
select github_repo_problemas.agregar_commit(<id>, '<sha>', '<branch>', '<url>', '<mensaje>', '<autor>');

-- lectura
select * from github_repo_problemas.v_problemas order by detectado_en desc;
```

**Avisar en el chat el titulo del problema** al registrarlo y al cerrarlo, no el numero de id
(mismo criterio que Planify).

**Si el problema se detecta pero NO se arregla, queda en `estado='abierto'`.** Ese es el punto:
que quede anotado. Estados: `abierto` | `en_curso` | `corregido` | `no_corregible` | `descartado`.
Para pasar a `corregido` la base exige `correccion` y `corregido_en` cargados (constraint).

**La auditoria no se borra.** El rol `anon` tiene SELECT/INSERT/UPDATE pero NO DELETE ni
TRUNCATE en las tres tablas. Si una fila esta mal, se corrige o se pasa a `descartado`.

## REGLA: toda copia de respaldo nace sin RLS

**Vale para TODOS los repos** (igual que las reglas de Planify y de auditoria: copiar este bloque
al `CLAUDE.md` de cualquier repo nuevo).

**⚠️ `CREATE TABLE AS` y `SELECT INTO` NO heredan Row Level Security de la tabla de origen.** La
copia queda con `relrowsecurity = false` aunque la madre este protegida, y los `GRANT` del schema
le siguen aplicando, asi que `anon` hereda SELECT/INSERT/UPDATE/DELETE. Postgres no emite ninguna
advertencia. **Prender RLS en el MISMO paso en que se crea la copia**, no despues:

```sql
create table <schema>.<copia> as select * from <schema>.<madre>;
alter table <schema>.<copia> enable row level security;  -- sin politicas = deny-all para anon
```

Sin politicas, RLS habilitada deja la tabla accesible solo para `service_role`, que es exactamente
lo que se quiere en un respaldo.

**Caso real (2026-09-14):** `planify.bkp_items_mayo_20260914`, respaldo de la liquidacion de sueldos
de mayo hecho —bien— antes de tocarla, quedo con 56 sueldos completos (legajo, nombre,
`sueldo_bolsillo`, banco, aportes) legibles y borrables por cualquiera con la clave publishable,
durante 24 horas. El respaldo estuvo bien; lo que falto fue el `alter`.

Para barrer copias abiertas en un proyecto:

```sql
select n.nspname, c.relname
  from pg_class c join pg_namespace n on n.oid = c.relnamespace
 where c.relkind = 'r' and c.relrowsecurity = false
   and has_table_privilege('anon', c.oid, 'SELECT')
   and n.nspname not in ('pg_catalog','information_schema','pg_toast');
```
