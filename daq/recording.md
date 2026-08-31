# Grabación y el formato .daq

Un archivo `.daq` es el formato de grabación del sensor de luz de Chloros: una **base de datos SQLite** que contiene fotogramas espectrales calibrados procedentes de un sensor DAQ. Si se graba uno durante una sesión de captura, el proceso de reflectancia podrá dividir posteriormente cada imagen por la irradiancia descendente medida en ese momento exacto.

## Qué contiene un archivo .daq

| Propiedad | Valor |
| --- | --- |
| Contenedor | Base de datos SQLite, un archivo por sensor y por grabación |
| Nombre de archivo | Incluye el **ID del sensor**y una**marca de tiempo**, p. ej., `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Espectro por fotograma | 135 puntos, de 340 a 1010 nm en intervalos de 5 nm, más el triestímulo CIE XYZ |
| Unidades | Irradiancia espectral calibrada, **W/m²/nm** (paquete de calibración de fábrica + perfil de la tapa aplicado) |
| Metadatos grabados | ID del sensor (la clave para recuperar la calibración de fábrica de esa unidad) y el perfil de tapa vigente — véase [Perfiles de tapa y rango calibrado](caps-and-range.md) |

El formato es idéntico en DAQ-U, DAQ-M y DAQ-E, por lo que al procesamiento posterior no le importa qué dispositivo de transporte lo haya registrado.

El registro calibrado requiere el paquete de calibración de fábrica del sensor. En el caso de DAQ-U y DAQ-M, el backend recupera el paquete de la nube de MAPIR mediante el ID del sensor (si no puede hacerlo, se rechaza la grabación); las unidades DAQ-E están exentas, ya que almacenan su calibración en el propio dispositivo.

## Grabación desde la interfaz gráfica de usuario

Para grabar desde la interfaz gráfica de usuario (GUI) es necesario tener un **proyecto abierto** (de lo contrario, los botones de grabación estarán desactivados):

* **Grabar todo / Detener todo** — en la parte superior de la barra lateral de «Sensores de luz»; inicia o detiene una grabación de `.daq` en todos los sensores conectados a la vez.
* **Grabar / Detener grabación** — por sensor, en la ventana modal de configuración (icono de engranaje). Durante la grabación, aparece un indicador rojo «REC» en las filas de información en tiempo real del sensor.

Los archivos se guardan en `<project>/light_sensor/` y, cuando se detiene una grabación —ya sea mediante «Detener», «Detener todo» o al desconectar un sensor de grabación—, el archivo `.daq` resultante se **añade automáticamente al proyecto abierto**. Aparece en la lista de archivos del proyecto sin necesidad de añadirlo manualmente, ya listo para el procesamiento de la reflectancia.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->

## Grabación desde el CLI

El CLI graba a través del conjunto de sensores del backend (el backend debe estar en ejecución; estos comandos son clientes ligeros de HTTP):

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

Obtener el valor `--sensor-id` de `chloros-cli daq pool-list`. Dos valores predeterminados que conviene conocer:

| Opción | Valor predeterminado |
| --- | --- |
| `--duration` | `0` — grabar hasta `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` en el sistema de archivos del **backend**, no en el de CLI |

La distinción entre directorios de salida es importante cuando el CLI apunta a un backend en otra máquina: el archivo se guarda donde se ejecuta el backend.

## Grabación desde Python

`DAQSensorSession` (devuela por `chloros_sdk.connect_daq_sensor()`) muestra la misma grabación agrupada: `record_start(output_dir=None, device_name=None)` devuelve la ruta del archivo, mientras que `record_stop()` devuelve `{path, rows}`. Consulte la [Referencia de SDK](../reference/sdk-reference.md) para ver la sesión completa API. Las clases de hardware directo de SDK (solo para instalaciones de escritorio) escriben las grabaciones en `~/Documents/DAQ/` de forma predeterminada; para las versiones publicadas, la ruta agrupada anterior es la recomendada.

## Uso de un archivo .daq en el momento del procesamiento

Para calcular la reflectancia a partir de las imágenes, Chloros necesita la irradiancia descendente correspondiente a cada exposición:

* **Conserva el archivo `.daq` junto con las imágenes.**En el momento del procesamiento, el flujo de trabajo resuelve automáticamente la**irradiación descendente con marca de tiempo coincidente** a partir de un `.daq` grabado (cualquier modelo de DAQ) —o de un `.csv` nativo de DAQ-M — que se encuentre junto a las imágenes. Las grabaciones de la interfaz gráfica de usuario cumplen este requisito automáticamente, ya que se añaden al proyecto en el momento en que se detienen.
* **La calibración se obtiene bajo demanda.** Si aún no se ha almacenado en caché localmente un paquete de calibración de fábrica por cámara o por DAQ, Chloros lo obtiene automáticamente de la nube de MAPIR al utilizarlo por primera vez (se requiere conexión a Internet una vez; se almacena en caché en `~/.chloros/`).
* **Las capturas en directo generan su propio archivo complementario.** Para cualquier fotograma de reflectancia capturado en directo, la lectura del DAQ realmente utilizada se guarda como un archivo complementario `.daq` junto a las imágenes, de modo que la captura se pueda volver a procesar más tarde sin necesidad de la grabación original.

## Recuperación de la irradiancia

Al procesar un proyecto, también se exportan todos los registros de sensores de luz que contiene a una
carpeta `Light Sensor/` situada junto a los productos de imagen. Para ello **no** se necesitan imágenes: un
sensor de luz que haya volado por sí solo constituye una captura completa, y una carpeta que contenga únicamente archivos `.daq`
es una entrada válida. La ejecución informa de cuántos productos del sensor de luz ha generado.

| Producto | Qué es |
| --- | --- |
| `<name>_calibrated.daq` | Un archivo reprocesable con el mismo esquema que una grabación en directo, que ahora declara el paquete de calibración que lo generó. Al volver a importarlo, **no** se calibra por segunda vez. |
| `<name>_calibrated.csv` | Irradiancia espectral en W/m²/nm en la propia rejilla de longitudes de onda del sensor, una fila por lectura, más columnas fotométricas: potencia total, lux fotópico y escotópico, PPFD con su división en azul/verde/rojo y longitud de onda máxima. |

Un DAQ-U o DAQ-M cuyo paquete de calibración no se pueda recuperar —porque estás sin conexión o
porque ese sensor no tiene ninguna calibración registrada— se **omite indicando el motivo**, y nunca se guarda
como un archivo «calibrado» que contenga recuentos sin procesar. Conéctate a Internet y vuelve a ejecutar el proceso. Un DAQ-E
lleva su propia calibración, por lo que solo la necesita cuando la unidad no está conectada y
no hay nada almacenado en la caché local.

### DAQ-A: recuentos sin procesar, y por qué esa es la respuesta correcta

El **DAQ-A** es anterior al sistema de paquetes de calibración por serie y no tiene ningún paquete que
recoger. No se trata de un descuido: un DAQ-A se calibra sobre el terreno frente a un
blanco de reflectancia, y la calibración basada en blancos solo necesita la respuesta *relativa*
del sensor, que es exactamente lo que representan sus recuentos sin procesar. Chloros se calibra con ellos en la actualidad.

Por lo tanto, un registro del DAQ-A se exporta, pero con un nombre diferente:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, no `_calibrated` —un nombre de archivo diferente en lugar de un indicador dentro del archivo,
porque la información debe conservarse aunque el archivo se reenvíe por correo electrónico como un simple nombre. El encabezado de `.csv`
indica `raw spectral sensor counts (NOT irradiance)` y advierte de que los valores son
comparables **dentro** del archivo y no entre sensores. Las columnas que solo tienen
significado para la irradiancia real —potencia total, lux, PPFD— se dejan vacías en lugar de
calcularse a partir de los recuentos.

Las grabaciones DAQ-A-SD más antiguas (esquema v1.01 / v1.02) solo registran la hora de escritura del archivo, no una
marca de tiempo por lectura. Chloros no hará coincidir las imágenes con esos datos —emparejar un fotograma con una
hora de escritura sería erróneo, aunque a simple vista no pareciera así—, pero la exportación los lee correctamente y
el CSV indica en qué reloj se encuentra.

Para obtener información completa sobre la reflectancia —sensor único con cámara y sensor doble (ambiente/objeto)—, consulta [Flujos de trabajo de reflectancia](reflectance.md).
