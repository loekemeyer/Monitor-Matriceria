# Monitor Matricería

TV del taller: **un solo tablero fijo** con las matrices que están en matricería. Cada
tarjeta muestra **lo mismo que se carga en Planify → 🛠️ Matricería**: número, nombre,
estado, **problema**, **tarea a realizar**, **lo ya hecho**, **quién** la está haciendo,
**HS**, **salida estimada**, día de ingreso, días de demora y la **foto** si la subieron.
Entran 6 por pantalla en una TV de 32" a 1920×1080; si hay más de 6, rota entre las
pantallas del taller.

> El contador de **unidades sin accidente** por matriz **no va acá** (Thomas, 16/09/2026:
> *"en matricería solamente un monitor fijo de las matrices que hay en el taller"*). Ese
> dato vive sólo en **Gestión Productiva 2.0 → Producción → Unidades sin Accidente**.

> El repo se llamaba `Monitor-Virgilio` por un error al crearlo; el 16/09/2026 se renombró a
> **`Monitor-Matriceria`**. GitHub redirige la URL vieja, así que los clones existentes siguen
> andando.

## Es un visualizador, no carga nada

Las matrices las cargan **Martín Pregelj** y **Martín Cornejo** desde **Planify → 🛠️
Matricería**. Esto sólo lee la vista `planify.v_matriceria_monitor` de Supabase (y, para las
fotos, el bucket `planify_matriceria`). La clave que viaja en el HTML es la publishable y
**sólo tiene permiso de lectura**: ni la matriz ni la foto se pueden tocar desde la TV.

Lo que se escribe en la planilla de Planify aparece en la TV en la **próxima consulta**
(30 s por defecto): no hay que refrescar ni tocar nada en la TV.

## Cómo ponerlo en la TV

Sin build ni instalación: es un único `index.html`.

1. **Con GitHub Pages** (recomendado, se actualiza solo al pushear): Settings → Pages →
   Source: `Deploy from a branch`, rama `main`, carpeta `/ (root)`. Queda en
   `https://loekemeyer.github.io/<nombre-del-repo>/`. Repo privado → hace falta Pages
   privado (plan pago) o pasarlo a público: el archivo no tiene secretos, sólo la clave
   pública de lectura.
2. **Sin Pages**: copiar `index.html` a la PC de la TV y abrirlo con doble clic. Para que
   arranque solo, poner un acceso directo en la carpeta `Inicio` de Windows con
   `chrome.exe --kiosk "C:\ruta\index.html"`.

En los dos casos conviene dejar el navegador en **pantalla completa (F11)**.

### Ajustes por URL

`index.html?columnas=2&porPantalla=6&refresco=30&pagina=15`

| Parámetro | Default | Qué hace |
|---|---|---|
| `porPantalla` | 6 | Matrices por pantalla |
| `columnas` | 3 | Columnas de la grilla |
| `refresco` | 30 | Segundos entre consultas |
| `pagina` | 15 | Segundos por pantalla cuando hay más de las que entran |
| `fotos` | 1 | `fotos=0` apaga las fotos (TV con poco ancho de banda) |

## Colores

| Estado | Color | Significa |
|---|---|---|
| Ingresada | azul | Entró, todavía no se tocó |
| En proceso | amarillo | Se está trabajando |
| Esperando | rojo | Frenada esperando repuesto o material |
| Terminada | — | Sale de la TV (y se cierra la tarea en Planify) |

La **demora** se pinta amarilla a los 3 días y roja a los 7. La **salida estimada** se pinta
amarilla el día que vence y roja si ya pasó. Las dos cuentas usan el **día del servidor**
(la vista calcula la demora en hora Argentina y la TV deduce de ahí qué día es hoy): una PC
con la fecha mal puesta no puede mentir.

Si la matriz no tiene cargada la tarea, las HS, el quién o la salida estimada, esos renglones
**no se muestran** en vez de mostrarse vacíos: la tarjeta sólo dice lo que hay.

## Si la TV muestra "SIN CONEXIÓN"

Sigue mostrando la última lista buena, con la hora del último dato. Revisar la red de esa
PC; no hay nada que reiniciar acá.
