# Monitor Matricería — Instrucciones para Claude Code

> **Nombre del repo:** nació como `loekemeyer/Monitor-Virgilio` por error del dueño y el
> 16/09/2026 se renombró a **`Monitor-Matriceria`**. GitHub redirige la URL vieja, así que
> los `git remote` que todavía apunten al nombre viejo siguen funcionando.

## Qué es

Un **visualizador** para la TV de 32" del taller: **un solo tablero fijo** con las matrices
que están en matricería. Muestra **toda la planilla que se carga en Planify → 🛠️
Matricería**: número, nombre, estado, problema (motivo + observaciones), tarea a realizar,
lo ya hecho, quién, HS, salida estimada, día de ingreso, días de demora y la foto.
Sale de `planify.v_matriceria_monitor` (+ el bucket `planify_matriceria` para las fotos).

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
- Entran **6 matrices por pantalla** (grilla 3×2, pensada para 1920×1080 a varios metros).
  Si hay más de 6, **rota** de pantalla cada 15 s con los puntitos abajo.
- Muestra sólo lo que está EN el taller (`estado != terminada`). Cuando la matriz se
  termina —o se cierra su tarea "Tratamiento matriz …" en Planify— desaparece sola.
- Los **días de demora los calcula el servidor** (la vista, en hora Argentina), no el
  navegador: una TV con la fecha mal puesta no puede mentir con la demora. La **salida
  estimada vencida** usa el mismo reloj sin pedir nada más: para una matriz que sigue en el
  taller, `fecha_ingreso + dias_demora` **es** el día de hoy del servidor (`hoyServidor()`).
- Lo que está **vacío no se dibuja**: sin tarea cargada, el problema se estira a 3 líneas;
  sin HS / salida / quién, esos renglones no ocupan lugar. Así la tarjeta no se llena de
  guiones.
- Las **fotos** viven en el bucket PRIVADO `planify_matriceria`, así que no se pueden poner
  como `src` directo: se bajan con la clave (`fetch` + `objectURL`) **una sola vez por
  archivo** y quedan cacheadas en memoria. Si una foto falla, la tarjeta se dibuja igual sin
  ella — en una TV jamás se rompe la pantalla por una imagen. Se apagan con `?fotos=0`.
- ⚠ **La TV del taller NO es una PC**: entra por un aparato tipo **Roku**, con control
  remoto, sin F11 y sin manera de sacar la barra del navegador. Informa **ventana 962 × 485**,
  pantalla 962 × 541 y escala 1,33 (panel real 1280 × 720). De ahí sale el
  **`@media (max-height: 600px)`** del CSS: con esa ventana el JS pasa solo a **4 tarjetas
  (2×2)** — `VENT_BAJA` — y la hoja de estilo sube toda la tipografía (texto 2,3 → 2,7vh).
  Se puede medir el 17% de aumento porque con 2 columnas cada tarjeta tiene el DOBLE de ancho
  y el texto entra en menos renglones; con 6 tarjetas y esa letra, "tarea a realizar" no
  entraba. `porPantalla` y `columnas` por URL siguen pisando el default.
- **El número grande lleva la etiqueta MATRIZ arriba.** Sin ella, un "2" solo se lee como
  una cantidad, no como el número de la matriz (pasó de verdad: Elías preguntó "2 qué?" el
  23/09/2026 mirando la TV). Cuesta casi nada (`line-height:1`) y se pagó bajando el número
  de 6,4 a 6vh en pantallas grandes.
- **Lector de resolución** al lado del reloj (`pintarRes()`): **apagado por defecto**, se
  prende con `?res=1`. Muestra ventana, pantalla y escala; sirve para medir una pantalla
  nueva antes de tocar el CSS. En la TV va apagado porque es ruido.
- **Nada cortado a la mitad** (`podarBloques()`): después de pintar, todo bloque de texto
  que no entre ENTERO en la tarjeta se saca del DOM. Recorre los bloques **de abajo hacia
  arriba y corta en el primero que entra**, así se cae siempre lo menos importante (el orden
  del HTML es problema > tarea a realizar > ya hecho); al revés se podía borrar la tarea y
  dejar el "ya hecho". El bloque del problema NO se saca nunca. Corre dos veces (la segunda
  en un `requestAnimationFrame`) porque en pantallas chicas el redondeo deja algún bloque
  asomando un píxel después de la primera medición. Por eso el diseño aguanta cualquier
  resolución sin retocar los `vh`.
- Si se cae la red, **deja lo último que se vio en pantalla** y avisa `SIN CONEXIÓN` abajo
  a la izquierda. Una TV en blanco no le sirve a nadie.

### Parámetros por URL (opcionales)

| Parámetro | Default | Qué hace |
|---|---|---|
| `porPantalla` | 6 | Cuántas matrices entran en una pantalla |
| `columnas` | 3 | Columnas de la grilla |
| `refresco` | 30 | Segundos entre consultas a Supabase |
| `pagina` | 15 | Segundos que dura cada pantalla cuando hay más de las que entran |
| `fotos` | 1 | `fotos=0` apaga las fotos |
| `res` | 0 | `res=1` muestra el lector de resolución (apagado por defecto) |

Ejemplo para una TV vertical: `index.html?columnas=2&porPantalla=6`.

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
