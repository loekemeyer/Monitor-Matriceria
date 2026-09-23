# Monitor Matricería

TV del taller: **un renglón por matriz**, con **Ingreso · Descripción · Estado**, en el
**formato de cuadro sinóptico** de la casa — columnas del ancho del dato, título más grande
que el contenido, centrado y ordenado por demora de mayor a menor. El color y el relleno
siguen la estética de la página (fondo del renglón, barra del estado y chip), que es la
excepción que pidió el dueño a ese formato.
Entran todas las matrices en una sola pantalla: no hay carrusel — cuantas más haya, más
chicos los renglones.

**El número de matriz no se muestra.** Si lo cargaron, se ve la descripción del maestro; si
escribieron texto libre (matriz experimental, muestra, recién hecha), ese texto es la
descripción. El número se sigue usando en Planify para que la misma matriz no termine con una
descripción distinta cada vez. En la descripción, la palabra **Corte** se abrevia **C/**.

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

`index.html?refresco=30`

| Parámetro | Default | Qué hace |
|---|---|---|
| `refresco` | 30 | Segundos entre consultas |
| `res` | 0 | `res=1` muestra el lector de resolución (apagado: en la TV es ruido) |
| `autorecarga` | 1 | `autorecarga=0` deja el cartel rojo pero no recarga sola |

## Estados

`Ingresado` · `Proceso` · `Ver Damián` (frenada hasta que la mire Damián) · `Esperando`
(repuesto o material) · `Terminada` (sale de la TV). Se cambian desde Planify, no desde acá.
Cada uno con su color, como el resto de la página: azul, ámbar, violeta, rojo y verde.

El orden lo decide la **demora**, que calcula el servidor en hora Argentina: la matriz que
lleva más días arriba de todo. Además la fecha se pinta **ámbar a los 3 días y roja a los 7**.
Una PC con la fecha mal puesta no puede mentir con eso.

## La TV del taller entra por un aparato tipo Roku

No es una PC con navegador: es un **dispositivo de TV** (Roku / Fire TV / smart TV) que se
maneja con **control remoto**. No tiene F11 ni forma de sacar la barra del navegador. Lo que
informa, medido el 23/09/2026:

| | |
|---|---|
| Ventana (lo que miden los `vh`/`vw`) | **962 × 485** |
| Pantalla que informa el aparato | 962 × 541 |
| Escala (`devicePixelRatio`) | 1,33 → panel real **1280 × 720** |

El diseño se acomoda solo a esa medida: la grilla y el tamaño de letra salen de cuántas
matrices hay y del espacio que queda, así que no hay nada atado a una resolución. Al probar
un cambio hay que mirarlo **a 962 × 485**, que es lo que de verdad tiene el taller.

Si alguna vez hay que averiguar con qué medida dibuja una pantalla nueva, se abre con
**`?res=1`** y el header muestra los tres números de la tabla de arriba.

## Si la TV quedó con una copia vieja

La página se publica sola al pushear, pero el navegador de la TV se queda con el
`index.html` que bajó la primera vez: **los datos se refrescan solos, el archivo no**. Por eso
cada 10 minutos se compara la versión que está corriendo contra la publicada. Si son
distintas sale un **cartel rojo arriba de todo** y la TV se recarga sola a los 4 segundos.
Si después de recargar sigue vieja (caché que no suelta), el cartel queda pidiendo recarga
a mano y no insiste más.

⚠ Al tocar `index.html` hay que **subir el `<meta name="monitor-version">`**. Si no, el
aviso no salta.

## Si la TV muestra "SIN CONEXIÓN"

Sigue mostrando la última lista buena, con la hora del último dato. Revisar la red de esa
PC; no hay nada que reiniciar acá.
