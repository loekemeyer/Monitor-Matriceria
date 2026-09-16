# Monitor Matricería

TV del taller: muestra las matrices que están en matricería con **número, día de ingreso,
días de demora, motivo y estado**. Entran 6 por pantalla en una TV de 32" a 1920×1080; si
hay más, rota solo.

> El repo se llama todavía `Monitor-Virgilio` por un error al crearlo. El nombre correcto
> es **`monitor-matriceria`** (Settings → Rename; GitHub redirige la URL vieja).

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

## Colores

| Estado | Color | Significa |
|---|---|---|
| Ingresada | azul | Entró, todavía no se tocó |
| En proceso | amarillo | Se está trabajando |
| Esperando | rojo | Frenada esperando repuesto o material |
| Terminada | — | Sale de la TV (y se cierra la tarea en Planify) |

La **demora** se pinta amarilla a los 3 días y roja a los 7.

## Si la TV muestra "SIN CONEXIÓN"

Sigue mostrando la última lista buena, con la hora del último dato. Revisar la red de esa
PC; no hay nada que reiniciar acá.
