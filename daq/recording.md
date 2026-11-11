# Grabación y el formato .daq

Un archivo `.daq` es el formato de grabación del sensor de luz de Chloros: una **base de datos SQLite** que contiene fotogramas espectrales calibrados procedentes de un sensor DAQ. Si se graba uno durante una sesión de captura, el proceso de reflectancia podrá dividir posteriormente cada imagen por la irradiancia descendente medida en ese momento exacto.

## Qué contiene un archivo .daq

| Propiedad | Valor |
| --- | --- |
| Contenedor | Base de datos SQLite, un archivo por sensor y por grabación |
| Nombre de archivo | Incluye el **ID del sensor**y una**marca de tiempo**, p. ej., `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Espectro por fotograma | 135 puntos, de 340 a 1010 nm en intervalos de 5 nm, más el triestímulo CIE XYZ |
| Unidades | Irradiancia espectral calibrada, **W/m²/nm** (paquete de calibración de fábrica + perfil de la tapa aplicado) |
| Metadatos grabados | ID del sensor (la clave para recuperar la calibración de fábrica de esa unidad) y el perfil de tapa vigente; véase [Perfiles de tapa y rango calibrado](caps-and-range.md) |

El formato es idéntico en DAQ-U, DAQ-M, DAQ-E y DAQ-E-S, por lo que al procesamiento posterior no le importa qué dispositivo de adquisición lo haya registrado. Un registro de DAQ-E-S también almacena la inclinación, el balanceo y el cabeceo de la unidad junto a cada espectro.

**Las grabaciones se conservan tras un corte de corriente (1.3.0).**Una grabación en directo se escribe con un registro y una sincronización completa del disco en cada confirmación, por lo que un corte de corriente o la interrupción de un proceso solo supone la pérdida de los últimos 3 segundos, no del archivo. Una grabación interrumpida a mitad de la escritura puede dejar un archivo `<name>.daq-journal` junto al `.daq`:**conserva y copia ambos archivos juntos.** Chloros revierte el registro la primera vez que lee el archivo. Un disco ocupado retrasa el archivo, pero nunca las lecturas del sensor.

**Un DAQ-E que deja de enviar datos se vuelve a conectar (1.3.0).** Cuando el flujo de un DAQ-E se interrumpe (por un reinicio del conmutador o una pérdida de conexión), el backend lo vuelve a conectar con la configuración que tenía, y la grabación en él continúa en el mismo archivo `.daq` con el intervalo marcado en el archivo.

La grabación calibrada requiere el paquete de calibración de fábrica del sensor. Para los DAQ-U y DAQ-M, el backend recupera el paquete de la nube de MAPIR mediante el ID del sensor (si no puede hacerlo, se rechaza la grabación); las unidades DAQ-E están exentas porque almacenan su calibración en el propio dispositivo.

## Grabación desde la interfaz gráfica de usuario

Para grabar desde la interfaz gráfica de usuario es necesario tener un **proyecto abierto** (de lo contrario, los botones de grabación estarán desactivados):

* **Grabar todo / Detener todo** — en la parte superior de la barra lateral de «Sensores de luz»; inicia o detiene una grabación `.daq` en todos los sensores conectados a la vez.
* **Grabar / Detener grabación** — por sensor, en la ventana modal de configuración (icono de engranaje). Durante la grabación, aparece un indicador rojo «REC» en las filas de información en tiempo real del sensor.

Los archivos se guardan en `<project>/light_sensor/`, y cuando se detiene una grabación —ya sea mediante «Detener», «Detener todo» o al desconectar un sensor de grabación—, el archivo `.daq` resultante se **añade automáticamente al proyecto abierto**. Aparece en la lista de archivos del proyecto sin necesidad de añadirlo manualmente, ya listo para el procesamiento de reflectancia.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->



## Grabación desde la CLI



La CLI graba a través del grupo de sensores del backend (el backend debe estar en ejecución; estos comandos son clientes HTTP ligeros):

```bash
# Connect the sensor into the backend pool
chloros-cli daq pool-connect --eth-host daq-e-def330.local

# Record for 150 seconds, with a human-friendly device label
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
    -o ./out --device-name "rooftop-A"

# Or run open-ended and stop explicitly
chloros-cli daq pool-record --sensor-id daq-e-def330            # --duration defaults to 0 = run until --stop
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

Obtiene el valor de `--sensor-id` a partir de `chloros-cli daq pool-list`. Dos valores predeterminados que conviene conocer:

| Opción | Valor predeterminado |
| --- | --- |
| `--duration` | `0` — grabar hasta `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` en el sistema de archivos del **backend**, no en el de la CLI |

La distinción entre directorios de salida es importante cuando la CLI apunta a un backend en otra máquina: el archivo se guarda donde se ejecuta el backend. Si el backend se encuentra en el mismo ordenador, una ruta relativa `-o` (como `./out` arriba) se resuelve en relación con la carpeta en la que se ejecutó el comando.

## Grabación desde Python

: `DAQSensorSession` (devuelto por `chloros_sdk.connect_daq_sensor()`) muestra la misma grabación agrupada: `record_start(output_dir=None, device_name=None)` devuelve la ruta del archivo, `record_stop()` devuelve `{path, rows}`. Consulte la [Referencia del SDK](../reference/sdk-reference.md) para ver la API completa de la sesión. Las clases de acceso directo al hardware del SDK (solo en instalaciones de escritorio) escriben las grabaciones en `~/Documents/DAQ/` de forma predeterminada; para las versiones publicadas, la ruta agrupada anterior es la admitida.

## Uso de un archivo .daq en el momento del procesamiento

Para calcular la reflectancia a partir de las imágenes, Chloros necesita la irradiancia descendente correspondiente a cada exposición:

* **Conserva el archivo `.daq` junto con las imágenes.**En el momento del procesamiento, el flujo de trabajo determina automáticamente la**irradiación descendente con marca de tiempo coincidente** a partir de un `.daq` grabado (cualquier modelo de DAQ) —o de un `.csv` nativo de DAQ-M — que se encuentre junto a las imágenes. Las grabaciones de la interfaz gráfica de usuario cumplen este requisito automáticamente, ya que se añaden al proyecto en el momento en que se detienen.
* **La calibración se obtiene bajo demanda.** Si aún no se ha almacenado en caché localmente un paquete de calibración de fábrica por cámara o por DAQ, Chloros lo recupera automáticamente de la nube de MAPIR la primera vez que se utiliza (se requiere conexión a Internet una vez; se almacena en caché como `~/.chloros/`).
* **Las capturas en directo escriben su propio archivo complementario.** Para cualquier fotograma de reflectancia capturado en directo, la lectura del DAQ realmente utilizada se guarda como un archivo `.daq` en la carpeta `daq/` de las capturas, de modo que la captura se pueda volver a procesar más tarde sin necesidad de la grabación original.
* **Grabaciones de los grabadores `chloros_scripts` (1.3.0).**Un archivo `.daq` generado por los grabadores públicos [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) aparece como el modelo DAQ que lo ha creado, y su límite máximo se puede configurar en**Configuración del proyecto → Sensor de luz DAQ**, al igual que en cualquier otra grabación. Estas grabadoras registran los recuentos brutos del sensor; Chloros recupera la calibración de fábrica del sensor a través del puerto serie y la aplica.
* **A quién corresponde la lectura (1.3.0).**Junto a cada captura `.daq`, un `.daq.cameras.json` identifica la(s) cámara(s) cuyos fotogramas describe esa lectura, y el procesamiento la asocia**únicamente a las imágenes de esas cámaras**. Por lo tanto, una cámara independiente que no disponga de sensor de luz propio ya no toma prestada la lectura de otra cámara: hay que asignarle un sensor o grabar una. Una grabación realizada con el propio DAQ (GUI Record, `daq pool-record`), y cualquier `.daq` que no contenga ese archivo (incluidas las capturas más antiguas), sigue siendo válida para todas las cámaras.

## Recuperar la irradiancia

Al procesar un proyecto, también se exportan todas las grabaciones de sensores de luz que contiene a una
carpeta `Light Sensor/` junto a los productos de imagen. Para ello **no** se necesitan imágenes: un
sensor de luz grabado por sí solo constituye una captura completa, y una carpeta que contenga únicamente archivos `.daq`
es una entrada válida. Un proyecto que solo contenga grabaciones de sensores de luz las muestra en una lista y
se procesa tanto desde la aplicación como desde la CLI, y permanecen en la lista tras la ejecución. La ejecución
informa de cuántos productos de sensores de luz se han generado.

| Producto | Qué es |
| --- | --- |
| `<name>_calibrated.daq` | Un archivo reprocesable con el mismo esquema que un registro en tiempo real, que ahora declara el paquete de calibración que lo generó. Al volver a importarlo, **no** se vuelve a calibrar por segunda vez. |
| `<name>_calibrated.csv` | Irradiancia espectral en W/m²/nm en la propia rejilla de longitudes de onda del sensor, una fila por lectura, más columnas fotométricas: potencia total, lux fotópicos y escotópicos, PPFD con su desglose en azul/verde/rojo, y longitud de onda máxima. |

Un DAQ-U o DAQ-M cuyo paquete de calibración no se pueda recuperar —porque estás sin conexión o
porque ese sensor no tiene ninguna calibración archivada— se **omite indicando el motivo**, y nunca se guarda
como un archivo «calibrado» que contenga recuentos sin procesar. Conéctate a Internet y vuelve a ejecutar el proceso. Un DAQ-E
lleva su propia calibración, por lo que solo la necesita cuando la unidad no está conectada y
no hay nada almacenado en la caché local.

### DAQ-A: recuentos sin procesar, y por qué esa es la respuesta correcta

El **DAQ-A** es anterior al sistema de paquetes de calibración por serie y no tiene ningún paquete que
recoger. No se trata de un descuido: un DAQ-A se calibra sobre el terreno frente a un
objetivo de reflectancia, y la calibración basada en objetivos solo necesita la respuesta *relativa*
del sensor, que es exactamente lo que son sus recuentos sin procesar. Chloros se calibra con ellos en la actualidad.

Por lo tanto, un registro del DAQ-A se exporta, pero con un nombre diferente:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, no `_calibrated` —un nombre de archivo diferente en lugar de un indicador dentro del archivo,
porque la información debe conservarse aunque el archivo se reenvíe por correo electrónico solo con su nombre. El encabezado `.csv`
indica `raw spectral sensor counts (NOT irradiance)` y advierte de que los valores son
comparables **dentro** del archivo y no entre sensores. Las columnas que solo tienen
significado para la irradiancia real —potencia total, lux, PPFD— se dejan vacías en lugar de
calcularse a partir de los recuentos.

Las grabaciones DAQ-A-SD más antiguas (esquema v1.01 / v1.02) solo registran la hora de escritura del archivo, no una
marca de tiempo por lectura. Chloros no cotejará las imágenes con esos datos —emparejar un fotograma con una
hora de escritura sería erróneo, aunque a simple vista no lo pareciera—, pero la exportación los lee correctamente y
el archivo CSV indica en qué reloj se encuentra.

Para obtener información completa sobre la reflectancia —sensor único con cámara y sensor doble (ambiente/objeto)—, consulta [Flujos de trabajo de reflectancia](reflectance.md).
