# Monitor Matricería

TV del taller. Rota entre dos tableros:

1. **Matricería** — las matrices que están en el taller: **número, nombre, día de ingreso,
   días de demora, motivo y estado**.
2. **Unidades sin accidente** — la analogía del cartel de *días sin accidentes*, pero en
   piezas: cuántas lleva fabricadas cada matriz **desde su último accidente** (una rotura
   RM, un pare de matriz PM, o el ingreso a matricería cargado en Planify), con el récord
   histórico y 🏆 en la que está en su mejor racha.

Entran 6 por pantalla en una TV de 32" a 1920×1080; si hay más, rota solo.

> El repo se llamaba `Monitor-Virgilio` por un error al crearlo; el 16/09/2026 se renombró a
> **`Monitor-Matriceria`**. GitHub redirige la URL vieja, así que los clones existentes siguen
> andando.

## Es un visualizador, no carga nada

Las matrices las cargan **Martín Pregelj** y **Martín Cornejo** desde **Planify → Ingreso
matrices**. Esto sólo lee la vista `planify.v_matriceria_monitor` de Supabase. La clave que
viaja en el HTML es la publishable y **sólo tiene permiso de lectura** sobre esa tabla.

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
| `rachas` | 12 | Cuántas matrices entran al tablero de unidades sin accidente (`0` lo apaga) |

## Colores

| Estado | Color | Significa |
|---|---|---|
| Ingresada | azul | Entró, todavía no se tocó |
| En proceso | amarillo | Se está trabajando |
| Esperando | rojo | Frenada esperando repuesto o material |
| Terminada | — | Sale de la TV (y se cierra la tarea en Planify) |

La **demora** se pinta amarilla a los 3 días y roja a los 7.

En el tablero de unidades sin accidente, el marco verde y el 🏆 marcan la matriz que está en
la mejor racha de su historia. El contador lo calcula la base (`GP2.matriz_racha`), no la TV:
lo actualizan solos el registro de producción y los ingresos que cargan los matriceros.

## Si la TV muestra "SIN CONEXIÓN"

Sigue mostrando la última lista buena, con la hora del último dato. Revisar la red de esa
PC; no hay nada que reiniciar acá.
