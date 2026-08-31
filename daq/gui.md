# La pestaña «DAQ» en Chloros

La pestaña DAQ —denominada **Sensores de luz** en la barra lateral de Chloros— es la interfaz de control en tiempo real para [los sensores de luz DAQ-U, DAQ-M y DAQ-E](README.md): conecta los sensores a través de cualquier protocolo de transporte, observa los espectros calibrados en tiempo real, calcula la reflectancia en tiempo real a partir de un par de sensores y graba archivos `.daq` directamente en tu proyecto.

La pestaña estará disponible una vez que el backend Chloros haya finalizado su inicio. Los gráficos de la pestaña se alimentan del servicio DAQ de Chloros a través de una conexión en directo que se vuelve a establecer automáticamente (tiempo de espera de 2 a 10 s) si se interrumpe; mientras no se pueda acceder al servicio, la fila «Estado» de un sensor mostrará **«Sin servidor»**.

El diseño consiste en una **barra lateral de sensores**(una fila por cada sensor conectado) más un**área de gráficos** (un mosaico de gráfico por sensor o grupo).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Conexión de un sensor

Haz clic en **Conectar sensor** en la parte superior de la barra lateral. Se abre el cuadro de diálogo de conexión en el área principal (o como una ventana superpuesta al añadir otro sensor; en ese caso, aparece un botón «Cancelar»).

| Control | Comportamiento |
| --- | --- |
| **Tipo de dispositivo** | `DAQ-U (USB)` (por defecto), `DAQ-M (Bluetooth)` o `DAQ-E (Ethernet)`. Al cambiar de opción, se reinicia la búsqueda del nuevo transporte seleccionado. |
| **Puerto / Dispositivo BLE / Nombre de host / IP** | Muestra una lista de los dispositivos detectados como `device - description`; se selecciona automáticamente la primera entrada reconocida como sensor. Durante la búsqueda, muestra `Scanning...` (USB), `Scanning (N)...` con una cuenta atrás de 8 segundos (BLE) o `Discovering ethernet sensors (N)...` con una cuenta atrás de 5 segundos (Ethernet). Los resultados vacíos se muestran como `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Actualizar** | Vuelve a escanear inmediatamente el transporte seleccionado (desactivado durante el escaneo BLE/Ethernet). |
| **Conectar** | Se activa una vez seleccionado un dispositivo; cambia la etiqueta a `Connecting...` mientras se establece la conexión. |

La búsqueda solo se ejecuta **mientras el cuadro de diálogo de conexión está en pantalla** y se repite cada 15 segundos únicamente para el transporte seleccionado; el simple hecho de abrir la pestaña no inicia el escaneo. En caso de fallo, el cuadro de diálogo muestra: *«Error de conexión. Intenta desconectar y volver a conectar el sensor; a continuación, haz clic en Conectar de nuevo».*

La barra lateral se abre automáticamente cuando se conecta el primer sensor.

{% hint style="info" %}
**¿No aparece el DAQ-E?** El DAQ-E no tiene LED de estado: comprueba el indicador de PoE/enlace del conmutador o del puerto del inyector al que está conectado, y espera unos segundos tras el encendido para que se inicie. El dispositivo Chloros debe estar en el mismo dominio de difusión (el mDNS no atraviesa los routers). En el Windows, acepta la solicitud del cortafuegos Defender la primera vez que el Chloros establezca la conexión de sus sockets de multidifusión (mDNS UDP 5353, datos del DAQ-E UDP 5002, PTP UDP 319/320). Las dos unidades DAQ-E conectadas a una misma LAN se detectan por separado, cada una con su propio nombre de host `daq-e-<id>.local`.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>El tipo de dispositivo ofrece DAQ-U (USB), DAQ-M (Bluetooth) y DAQ-E (Ethernet)</figcaption></figure>***

## La barra lateral de sensores

Cada sensor conectado ocupa una fila (más una fila por cada grupo «Ambiente+Objeto»). Las filas se pueden reordenar arrastrándolas, y su orden también reordena los mosaicos del gráfico. Haz clic en una fila para que ese sensor o grupo se convierta en el gráfico activo en la vista de lista.

| Elemento | Significado |
| --- | --- |
| Borde izquierdo de color | El color del gráfico del sensor. |
| Insignia de transporte | `DAQ-U` / `DAQ-M` / `DAQ-E`, o una insignia verde `REF` para un grupo de reflectancia «Ambient+Object». |
| Nombre del dispositivo | Por defecto, es el número de serie del sensor (su identidad estable para la calibración, los nombres de archivo `.daq` y la coincidencia en la importación); los nombres personalizados se conservan por proyecto. |
| Indicador **Calibrado** (verde) | Aparece cuando se carga el paquete de calibración de fábrica del sensor, es decir, cuando los espectros están expresados en W/m²/nm. |
| Indicador **Actualización disponible** (ámbar, solo DAQ-E) | El firmware en ejecución es anterior a la imagen incluida en esta versión Chloros. Durante una actualización, muestra el progreso en tiempo real (`Flashing… N%`, `Restarting sensor…`, luego `Updated X → Y` o `Failed`). |
| Ojo | Activa o desactiva la visibilidad de este sensor en su gráfico. |
| Engranaje | Abre la ventana modal de configuración de cada sensor (más abajo). |
| ✕ (rojo) | Desconecta el sensor o elimina un grupo «Ambiente+Objeto». |

Por encima de las filas hay dos botones:

* **Conectar sensor**: abre el cuadro de diálogo de conexión (cambia el nombre a `Connecting...` mientras está ocupado).
* **Grabar todo / Detener todo**: inicia o detiene una grabación `.daq` en**todos**los sensores conectados. Requiere al menos un sensor**y un proyecto abierto** (información sobre herramientas: «Abre un proyecto para grabar»); se vuelve rojo mientras se está realizando alguna grabación.

Cuando está vacío, muestra el mensaje «No hay sensores conectados».

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Ajustes por sensor (ventana modal del engranaje)

Se abre con el icono del engranaje en la fila de un sensor. Contenido por orden:

* **Filas de información** — Tipo de dispositivo (DAQ-U/M/E), Conexión (`Serial (USB)` / `Bluetooth` / `Ethernet`), puerto (puerto COM, dirección BLE o host) y número de serie.
* **Informe de calibración: Descargar** — descarga el certificado de calibración de esta unidad, trazable según el NIST (PDF), y lo abre en tu visor de PDF. Disponible una vez que se conoce el número de serie; el certificado se almacena en caché en la primera conexión.
* **Nombre del dispositivo** — haz clic en el lápiz para cambiarlo; se mantiene por proyecto.
* **Color de la línea del gráfico** — muestra de color; se mantiene para cada proyecto.
* **Tiempo de integración (ms)**— control deslizante + número,**1–500 ms**, valor por defecto**32 ms**. Desactivado mientras la exposición automática (AE) está activada.
* **Promedio de fotogramas**: control deslizante + número,**1–50 fotogramas**, valor por defecto**20**.
* **AE: activado/desactivado**: interruptor de exposición automática;**activado por defecto** al conectarse. Desactívalo para ajustar el tiempo de integración manualmente.
* **Detener transmisión / Iniciar transmisión** — pausa o reanuda la transmisión en directo.
* **Grabar / Detener grabación** — grabación `.daq` por sensor (requiere un proyecto abierto).
* **Cap** — el perfil de corrección de cap (siguiente sección).
* **Líneas de información en directo** — Tiempo de integración (ms), FPS, Muestras, Grabación (rojo `REC` o `Off`) y Estado (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Solo DAQ-E: filas de red, firmware y PTP

* **Nombre de host / IP**: la dirección actual de la unidad.
* **Firmware**: versión actual del firmware, además de una celda de acción:<version\>

aparece</version\>

un<version\>

botón</version\>

**Actualizar a \<version\>** cuando esta versión Chloros incluye una imagen de firmware más reciente para el DAQ-E. La actualización se realiza a través de la red en unos 30 segundos; el sensor se reinicia y se vuelve a conectar automáticamente, y si la transferencia se interrumpe, el firmware actual permanece intacto. El progreso se muestra en tiempo real (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), y la celda indica `Up to date` cuando está actualizado.
* **Sincronización PTP**: el estado PTP en tiempo real (vuelve a `unknown`). El firmware v1.2.0+ del DAQ-E participa en el estándar IEEE 1588 PTPv2 como reloj exclusivamente esclavo; el backend del host Chloros es el «grandmaster» de PTP, y todas las cámaras DAQ-E y LATTICE de la LAN se sincronizan con él en el dominio 0, manteniendo las marcas de tiempo con una precisión de aproximadamente 1 ms.

En el caso de un grupo «Ambient+Object», el menú modal «Gear» muestra únicamente los sensores de origen del grupo, el nombre del dispositivo y el color de la línea del gráfico.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->

### Selección de la tapa

El menú desplegable **Tapa** indica a Chloros qué tapa física está colocada sobre el difusor del sensor y aplica el perfil de corrección medido de fábrica de esa tapa a cada espectro. Las opciones dependen del modelo:

| Modelo | Opciones de tapa |
| --- | --- |
| DAQ-U | Ninguna (sensor sin cubierta), FOV 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunshine (corrector coseno) |
| DAQ-M | Ninguna (sensor sin tapa), Sunshine (corrector de coseno) |
| DAQ-E | Ninguna (sensor sin tapa), campo de visión (FOV) de 15°, campo de visión (FOV) de 45°, campo de visión (FOV) de 90°, Sunshine (corrector de coseno) |

**La configuración predeterminada para todos los modelos es «Sunshine» (corrector de coseno)** — MAPIR suministra todos los DAQ con la tapa «Sunshine» instalada, y esta es la configuración estándar para exteriores: una visión hemisférica de 180° con un error de coseno ≤ ±4 % hasta los 60° y ≤ ±4,5 % hasta los 70° (no recomendado por debajo de una elevación solar de ~15°), con una atenuación por diseño de ~12×. La selección elegida se mantiene en el proyecto.

{% hint style="warning" %}
**La selección de la tapa debe coincidir con la tapa física.**Ni el sensor ni el software pueden detectar qué tapa está colocada. La selección determina tanto la corrección en tiempo real como la marca que se escribe en cada archivo `.daq`; dada la atenuación de ~12× de la tapa Sunshine, un cambio de tapa no declarado corrige erróneamente los espectros aproximadamente en ese factor. (Al retirar y volver a montar la misma tapa, la repetibilidad es de aproximadamente el 1,5 %.) Selecciona**Ninguna (sensor sin tapa)** únicamente cuando la tapa se haya retirado físicamente; en un DAQ-E, la opción «Ninguna» sigue aplicando un perfil geométrico de fábrica para su difusor de cristal empotrado —no es una operación nula— y un DAQ-E sin tapa es una configuración de laboratorio, no una configuración de campo compatible.
{% endhint %}

{% hint style="info" %}
Actualización desde una versión anterior del manual: el botón de alternancia «Difusor Sunshine instalado» del navegador, presente en la versión 1.1.0, ha desaparecido. La gestión de la tapa se realiza ahora mediante este perfil de tapa por sensor, que se aplica del lado del servidor.
{% endhint %}

***

## El área de gráficos

Una barra superior fija contiene un **botón para alternar entre vista de lista y vista de cuadrícula**y un control deslizante de**zoom del gráfico** (tamaño de los mosaicos: 200–2000 píxeles). La vista cambia automáticamente a cuadrícula cuando hay más de un grupo de gráficos, y vuelve a la vista de lista cuando hay uno o menos. El modo de visualización y el tamaño del gráfico se conservan por proyecto.

El **gráfico de espectro** de cada sensor muestra:

* **Eje X** — Longitud de onda (nm). La cuadrícula del sensor abarca de 340 a 1010 nm con un paso de 5 nm (135 puntos), interpolada a 1 nm para su visualización.
* **Eje Y**: potencia (W/m²), con un prefijo del Sistema Internacional (SI) automático (m/µ/n) elegido a partir del pico. Los espectros son irradiancias espectrales (W/m²/nm) calibradas radiométricamente en los tres transportes.
* Un relleno espectral en forma de arco iris bajo una única traza; los sensores múltiples en un mismo gráfico se superponen como líneas de colores con rellenos atenuados.
* **Pasar el cursor**: un cursor vertical con la longitud de onda y el valor por sensor;**arrastrar** para ampliar (aparece un botón para alejar la imagen mientras se está ampliada).
* Un botón **+** (solo en vista de cuadrícula) para añadir un sensor a este gráfico o crear un grupo (más abajo).
* El nombre del dispositivo centrado en la parte superior y un indicador giratorio hasta que llega el primer fotograma.

**La saturación** no se indica en el propio gráfico: un sensor saturado muestra el texto de estado `SATURATED` en rojo y una fila `Saturated: Yes` en rojo en la tabla de datos en tiempo real. Reduce el tiempo de integración o vuelve a activar AE para borrarlo.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->

***

## Tabla de datos en tiempo real (vista de lista)

Debajo del gráfico, en la vista de lista, que se actualiza cada 500 ms:

* **Todos los modelos**: Muestra de color de la luz (sRGB a partir de CIE XYZ), Saturado (Sí/No), CIE 1931 X/Y/Z, Cromaticidad x/y, CIE u′/v′, CCT (K), CRI (Ra), longitud de onda dominante (nm), longitud de onda máxima (nm), pureza de excitación, Duv, CIE L\*/a\*/b\* y Munsell H/V/C.
* **Solo sensores calibrados**(cualquiera de los modelos DAQ-U / DAQ-M / DAQ-E una vez cargado su paquete de calibración de fábrica; la insignia verde**Calibrado** en la fila del sensor lo indica): Potencia total (W/m²), lux fotópico (lx), lux escotópico (lx), relación S/P, PPFD más PPFD Red/Green/Blue (µmol/m²/s), y las irradiancias opicas: cono S, melanópica, rodópica, cono M, cono L (todas en W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Grupos de reflectancia (Ambiente + Objeto)

Se pueden combinar dos sensores conectados para obtener una visualización de la reflectancia en tiempo real, sin necesidad de cámara:

1. En la vista de cuadrícula, haz clic en **+**en un mosaico del gráfico y selecciona**Combinar ambiental + objeto**.
2. Selecciona un sensor de **fuente de luz ambiental**y un sensor de**escáner de objeto**(dos sensores distintos) y, a continuación,**Crear**.

Chloros calcula R(λ) = objeto(λ) / ambiental(λ) por longitud de onda a partir de las dos señales en tiempo real (0 cuando ambiental ≤ 0). La etiqueta del grupo sigue la clase de calibración de los sensores:

* Ambos sensores calibrados (paquete cargado) → **«Reflectancia aparente»**.
* Cualquiera de los sensores sin calibrar → **«Reflectancia relativa»**.

El grupo aparece como una fila verde `REF` en la barra lateral y con su propio gráfico (relleno arcoíris, valores al pasar el cursor con 4 decimales, zoom al arrastrar).

El menú **+**también ofrece la opción**Añadir nuevo sensor** con tres ubicaciones: *Combinar nuevo sensor* (añadirlo a este gráfico), *Mover sensor existente aquí* o *Ver nuevo sensor* (en su propio gráfico).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->

### Tabla de índices de vegetación

En la vista de lista, debajo del gráfico de un grupo de reflectancia aparece una tabla de índices de vegetación, calculada a partir de la reflectancia en tiempo real en los centros de banda **azul 450 / verde 550 / rojo 670 / NIR 800 nm** (valores con 4 decimales; `---` cuando no se puede calcular; pasa el cursor por encima del nombre de un índice para ver su nombre completo):

* **Siempre se muestran** (invariantes a la escala, cualquier combinación de sensores): NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Solo cuando ambos sensores están calibrados en potencia** (ambos paquetes cargados): EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

.***

## Grabación de archivos `.daq`

* Para grabar es necesario tener un **proyecto abierto**; de lo contrario, tanto la opción «Grabar todo» (barra lateral) como el botón «Grabar» de cada sensor estarán desactivados.
* Los archivos se guardan con el formato **`<project folder>/light_sensor/`**; los nombres de los archivos incluyen el ID del sensor y una marca de tiempo, y el nombre del dispositivo se almacena junto con la grabación.
* Cuando se detiene una grabación (con «Detener», «Detener todo» o una desconexión durante la grabación), el archivo `.daq` finalizado **se añade automáticamente al proyecto abierto** — y aparece en la lista de archivos del proyecto sin necesidad de añadirlo manualmente, listo para servir como datos de irradiación descendente para el [procesamiento de la reflectancia](README.md).
* Durante la grabación, aparece un indicador rojo `REC` en las filas en tiempo real de la ventana modal de configuración.

Para obtener valores cuantitativos de irradiancia, se debe calcular la media de al menos 15 segundos de datos; se trata de una característica del instrumento, no de un defecto.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->

***

## Disposiciones multisensor y persistencia del proyecto

* Combina varios sensores en un mismo gráfico (ejes compartidos), mantén gráficos separados (disposición automática en cuadrícula), mueve sensores entre gráficos, reordena filas o mosaicos arrastrándolos y oculta sensores individuales con el botón de ojo.
* Por proyecto, Chloros conserva: los nombres de los dispositivos, los colores de los gráficos, el tamaño de los gráficos, el modo de visualización y la configuración de cada sensor (tiempo de integración, promediado de fotogramas, estado AE, selección de límite).
* **Al volver a abrir un proyecto, los sensores se vuelven a conectar automáticamente** por dirección —puerto COM para DAQ-U, dispositivo BLE para DAQ-M, nombre de host mDNS para DAQ-E (se resuelve incluso si ha cambiado la IP de la unidad)— y vuelve a aplicar el perfil de cap guardado de cada sensor, el promedio de fotogramas, el estado AE y el tiempo de integración manual.***

## Emparejamiento de la cámara (DLS)

No hay nada que emparejar. A diferencia de los flujos de trabajo DLS con drones, que vinculan un sensor de luz a una cámara desde el principio, Chloros empareja los datos DAQ con las imágenes posteriormente: en el momento de la importación o el procesamiento, las lecturas de `.daq` se interpolan a la marca de tiempo de exposición de cada captura. Graba con cualquier sensor conectado (el `.daq` se incluye automáticamente en el proyecto) y el procesamiento de la reflectancia encuentra las lecturas correctas según el tiempo; consulta [Sensores de luz DAQ](README.md) para saber cómo se utilizan los datos de luz descendente.</version\>
