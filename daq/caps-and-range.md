# Perfiles de las tapas y rango calibrado

> Las propias tapas —qué tapa se suministra con cada sensor, cómo se montan y su comportamiento óptico— están documentadas en el **[manual de usuario del DAQ](https://mapir.gitbook.io/daq)**. Esta página trata sobre cómo *declarar* la tapa instalada en Chloros, que es lo que hace que la corrección sea correcta.

La calibración radiométrica de fábrica de cada sensor de luz DAQ describe el sensor *sin cubierta*. La cubierta física instalada sobre el difusor modifica la luz que capta el sensor, por lo que Chloros aplica un **perfil de corrección de cubierta** medido en fábrica además del paquete de calibración. Declarar la tapa correcta es parte del proceso para obtener datos calibrados; esta página explica qué tapas existen para cada modelo, cómo declararlas y cuál es realmente el rango espectral calibrado del sensor.

## Disponibilidad de tapas por modelo

| Perfil de tapa (`cap_id`) | Tapa física | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Tapa correctora de coseno para luz solar (**predeterminada en todos los modelos**) | Sí | Sí | Sí |
| `fov_15` / `fov_45` / `fov_90` | Conos de restricción del campo de visión (15° / 45° / 90°) | Sí | — | Sí |
| `fov_30` / `fov_60` | Conos limitadores del campo de visión (30° / 60°) | Sí | — | — |
| `none` | Sin tapa instalada | — | — | Sí |

Notas específicas del modelo:

* **El DAQ-M tiene un único perfil de tapa: `sunshine_cosine`.** «Bare-plus-Sunshine-cap» es su definición de producto, y un DAQ-M «bare» no necesita perfil geométrico.
* **Un DAQ-U «bare» es verdaderamente «bare»**: no necesita ningún perfil geométrico, por lo que no existe el perfil `none` para él.
* **El `none` en un DAQ-E NO es una operación nula.** El difusor empotrado y cubierto de cristal del DAQ-E tiene su propia corrección geométrica real, por lo que «sin tapa» es en sí mismo un perfil medido en este modelo.
* Un **DAQ-E sin cubierta no puede medir la luz solar directa a ninguna elevación**: la cubierta Sunshine es la configuración de campo. No planifiques trabajos al aire libre con un DAQ-E sin cubierta.

En la configuración por sensor de la interfaz gráfica de usuario (icono de engranaje en la pestaña «Sensores de luz»), el menú desplegable **Tapa** también ofrece la opción «Ninguna (sensor sin tapa)» en los modelos DAQ-U y DAQ-M; en estos dos modelos, «sin tapa» significa simplemente que no se aplica ninguna corrección de tapa, tal y como se indica en las notas anteriores. Selecciónala solo cuando se haya retirado físicamente la tapa.

## Declaración de la tapa: por qué es importante

**El código `cap_id` declarado debe coincidir con la tapa que se encuentra físicamente en el sensor.** Ni el sensor ni el software pueden detectar la tapa instalada. La declaración determina dos aspectos:

1. La **corrección en tiempo real** aplicada a cada espectro.
2. La **marca de la tapa que se incluye en cada registro `.daq`**, en la que se basa el procesamiento posterior de la reflectancia.

La tapa Sunshine atenúa aproximadamente **12 veces por diseño**, por lo que registrar con una tapa incorrecta declarada distorsiona la escala de los espectros en aproximadamente ese factor. Declara los cambios de tapa inmediatamente.

### Configuración de la tapa

Interfaz gráfica de usuario (GUI): pestaña «Sensores de luz» → icono de engranaje en la fila del sensor → menú desplegable **Tapa**. El valor predeterminado para todos los modelos es `sunshine_cosine` (todos los sensores DAQ se suministran con el corrector de coseno instalado), y la selección se mantiene en el proyecto.

<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->

CLI (el backend debe estar en ejecución):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

El CLI acepta sintácticamente la lista completa de `cap_id` (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); cada perfil se valida con el modelo del sensor en el momento de la conexión, por lo que un ID de condensador no disponible (por ejemplo, un ID «solo E» en un DAQ-U) da lugar a un error claro en lugar de una corrección errónea. El valor predeterminado del backend cuando no se pasa nada es `sunshine_cosine`.

Python SDK nota: `cap_id` **no** es un control SDK; `connect_daq_sensor()` / `DAQSensorSession` no exponen ningún parámetro de condensador. Selecciona el límite mediante los comandos CLI anteriores o el menú desplegable de la interfaz gráfica de usuario; consulta la [Referencia de SDK](../reference/sdk-reference.md).

Avanzado: los perfiles se incluyen en la instalación de Chloros en `daq/cap_profiles/<u|m|e>/<cap_id>.json` y pueden ser anulados por cada usuario en `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Al margen de los límites máximos, los sensores que nunca se han recalibrado reciben automáticamente un pequeño ajuste de la compensación de fondo oscuro derivado de la flota, sin que el usuario tenga que intervenir.

## Rendimiento del límite máximo de luz solar (configuración para exteriores)

Cifras en las que puede basar sus procedimientos:

| Propiedad | Valor |
| --- | --- |
| Campo de visión | Hemisférico de 180° |
| Error de respuesta coseno | ≤ ±4 % hasta una incidencia de 60°; ≤ ±4,5 % hasta una incidencia de 70° |
| Límite de sol bajo | No se recomienda por debajo de ~15° de elevación solar |
| Atenuación | ~12× (por diseño) |
| Repetibilidad al volver a colocar la gorra | ≈ 1,5 % |
| Irradiancia cuantitativa | Media de **≥ 15 s** de lecturas (característica del instrumento, no es un defecto) |

Para cualquier valor de irradiancia cuantitativa —incluidas las referencias de reflectancia—, utilice una media de al menos 15 segundos de lecturas en lugar de un único fotograma.

## Rango espectral calibrado

| Propiedad | Valor |
| --- | --- |
| Muestreo espectral | 340–1010 nm en intervalos de 5 nm (135 puntos) |
| Rango calibrado radiométricamente | **~374–974 nm** (impuesto por el software) |

El sensor proporciona la cuadrícula completa de 340–1010 nm, pero la ganancia radiométrica trazable según el NIST abarca ~374–974 nm. Chloros **rechaza la división de reflectancia absoluta** para cualquier banda de la cámara que tenga menos de la mitad de su peso espectral dentro de ese intervalo, indicando el motivo de omisión `dls-uncalibrated-band-<nm>` en lugar de generar un producto sin calibrar. Entre los modelos de cámara disponibles en el mercado, solo el filtro F988 queda fuera de este rango; en su lugar, utiliza el flujo de trabajo del panel de reflectancia —véase [Flujos de trabajo de reflectancia](reflectance.md).

Para conocer los modelos de sensores, los transportes y los identificadores de sensores, consulte la [descripción general de DAQ](README.md). Para saber cómo se consume el sello de límite máximo durante el procesamiento, consulte [Grabación y el formato .daq](recording.md).
