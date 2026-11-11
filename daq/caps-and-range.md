# Perfiles de las tapas y rango calibrado

> Las propias tapas —qué tapa se suministra con cada sensor, cómo se montan y su comportamiento óptico— se describen en el **[manual de usuario del DAQ](https://mapir.gitbook.io/daq)**. Esta página trata sobre cómo *declarar* la tapa instalada a Chloros, que es lo que garantiza que la corrección sea correcta.

La calibración radiométrica de fábrica de cada sensor de luz DAQ describe el sensor *sin cubierta*. La cubierta física instalada sobre el difusor modifica la luz que recoge el sensor, por lo que Chloros aplica un **perfil de corrección de cubierta** medido en fábrica sobre el conjunto de calibración. Declarar la tapa correcta es parte del proceso para obtener datos calibrados: esta página explica qué tapas existen para cada modelo, cómo declararlas y cuál es realmente el rango espectral calibrado del sensor.

## Disponibilidad de tapas por modelo

| Perfil de tapa (`cap_id`) | Tapa física | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Tapa correctora de coseno para luz solar (**predeterminada en todos los modelos**) | Sí | Sí | Sí |
| `fov_15` / `fov_45` / `fov_90` | Conos limitadores del campo de visión (15° / 45° / 90°) | Sí | — | Sí |
| `fov_30` / `fov_60` | Conos limitadores del campo de visión (30° / 60°) | Sí | — | — |
| `none` | Sin tapa | — | — | Sí |

Notas específicas del modelo:

* **El DAQ-M tiene un único perfil de tapa: `sunshine_cosine`.** «Bare-plus-Sunshine-cap» es su definición de producto, y un DAQ-M «bare» no necesita perfil geométrico.
* **Un DAQ-U «bare» es verdaderamente «bare»**: no necesita ningún perfil geométrico, por lo que no existe el perfil `none` para él.
* **El `none` en un DAQ-E NO es una operación nula.** El difusor empotrado y cubierto de cristal del DAQ-E tiene su propia corrección geométrica real, por lo que «sin tapa» es en sí mismo un perfil medido en este modelo.
* Un **DAQ-E sin cubierta no puede medir la luz solar directa a ninguna elevación**: la cubierta Sunshine es la configuración de campo. No planifiques trabajos al aire libre con un DAQ-E sin cubierta.

En la configuración por sensor de la interfaz gráfica de usuario (icono de engranaje en la pestaña «Sensores de luz»), el menú desplegable **Tapa** también ofrece la opción «Ninguna (sensor sin tapa)» en los modelos DAQ-U y DAQ-M; en estos dos modelos, «sin tapa» significa simplemente que no se aplica ninguna corrección de tapa, tal y como se indica en las notas anteriores. Selecciónala solo cuando la tapa se haya retirado físicamente.

## Declaración de la tapa: por qué es importante

**El código `cap_id` declarado debe coincidir con la tapa que se encuentra físicamente en el sensor.** Ni el sensor ni el software pueden detectar la tapa instalada. La declaración determina dos aspectos:

1. La **corrección en tiempo real** aplicada a cada espectro.
2. La **marca de la tapa que se incluye en cada registro `.daq`**, en la que se basa el procesamiento posterior de la reflectancia.

La tapa Sunshine atenúa aproximadamente **12 veces por diseño**, por lo que registrar con una tapa incorrecta declarada provoca un error de escala en los espectros de aproximadamente ese factor. Declara los cambios de tapa inmediatamente.

### Configuración de la tapa

Interfaz gráfica de usuario (GUI): pestaña «Sensores de luz» → icono de engranaje en la fila del sensor → menú desplegable **Tapa**. El valor predeterminado para todos los modelos es `sunshine_cosine`, una suposición (los modelos DAQ-U, DAQ-M y DAQ-E se suministran sin tapa, y un registro indica que se ha asumido la tapa hasta que se elija una), y la selección se mantiene en el proyecto.



CLI<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->
(el backend debe estar en ejecución):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

La CLI acepta sintácticamente la lista completa `cap_id` (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); cada perfil se valida con el modelo del sensor en el momento de la conexión, por lo que un ID de tapa no disponible (por ejemplo, un ID exclusivo de E en un DAQ-U) da un error claro en lugar de realizar una corrección errónea. El valor predeterminado del backend cuando no se pasa nada es `sunshine_cosine`.

Python SDK: asigna un nombre a la capa en el momento de la conexión con `connect_daq_sensor(cap_id="sunshine_cosine")`, o cámbialo en tiempo de ejecución con `DAQSensorSession.set_cap("fov_45")`; consulta la [Referencia del SDK](../reference/sdk-reference.md#daq-sensor-sessions).

Avanzado: los perfiles se incluyen en la instalación de Chloros en `daq/cap_profiles/<u|m|e>/<cap_id>.json` y pueden ser sobrescritos por cada usuario en `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Al margen de los límites, los sensores que nunca se han recalibrado reciben automáticamente un pequeño ajuste de el desplazamiento oscuro derivado de la flota, sin que el usuario tenga que intervenir.

## DAQ-E-S: difusor fijo e inclinación

El **DAQ-E-S** es un DAQ-E con dos diferencias importantes para Chloros. Todo lo demás (detección, conexión, PTP, grabación y configuración del dispositivo) funciona exactamente igual que en un DAQ-E.

* **Su difusor es fijo, por lo que no requiere corrección de tapa.** El difusor ya estaba instalado en la unidad cuando se realizó su calibración de fábrica, por lo que dicha calibración ya indica la irradiancia descendente real. Chloros no le aplica ningún perfil de límite, independientemente de lo que se solicite: si se indica un límite específico para él, este se ignora con un mensaje; sus grabaciones indican que el límite proviene del modelo, y la advertencia de límite supuesto nunca se activa en su caso. En Ajustes del proyecto → Sensor de luz DAQ, su fila no muestra ningún límite, y la anulación del límite a nivel de proyecto no le afecta. No apliques un límite a sus datos manualmente: la corrección de luz solar resultante sería aproximadamente 11 veces mayor de lo real.
* **Incorpora un acelerómetro de 3 ejes.** La respuesta coseno del difusor solo es válida mientras está nivelado, por lo que un sensor inclinado indica una irradiancia errónea sin ningún otro síntoma. Por lo tanto, cada lectura del DAQ-E-S incluye su inclinación, balanceo y cabeceo, y un `.daq` registrado a partir de una de ellas los almacena junto a cada espectro (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, …), de modo que se puede distinguir una nube de una montura desajustada.

Para obtener información sobre cada campo de actitud y sobre cómo se notifica un fallo en el flujo de datos de actitud, consulta [DAQ-E-S en la Referencia de la CLI](../reference/cli-reference.md#daq-e-s--fixed-diffuser-and-tilt-sensor).

## Rendimiento de la cubierta solar (configuración en exteriores)

Cifras en las que basar los procedimientos:

| Propiedad | Valor |
| --- | --- |
| Campo de visión | Hemisférico de 180° |
| Error de respuesta coseno | ≤ ±4 % hasta una incidencia de 60°; ≤ ±4,5 % hasta 70° |
| Límite de sol bajo | No se recomienda por debajo de ~15° de elevación solar |
| Atenuación | ~12× (por diseño) |
| Repetibilidad al volver a montar la tapa | ≈ 1,5 % |
| Irradiancia cuantitativa | Media de **≥ 15 s** de lecturas (característica del instrumento, no es un defecto) |

Para cualquier valor de irradiancia cuantitativa —incluidas las referencias de reflectancia—, utilice una media de al menos 15 segundos de lecturas en lugar de un único fotograma.

## Rango espectral calibrado

| Propiedad | Valor |
| --- | --- |
| Muestreo espectral | 340–1010 nm en intervalos de 5 nm (135 puntos) |
| Rango calibrado radiométricamente | **~374–974 nm** (impuesto por el software) |

El sensor proporciona la cuadrícula completa de 340–1010 nm, pero la ganancia radiométrica trazable según el NIST abarca ~374–974 nm. Chloros **rechaza la división de reflectancia absoluta** para cualquier banda de la cámara que tenga menos de la mitad de su peso espectral dentro de ese intervalo, indicando el motivo de omisión `dls-uncalibrated-band-<nm>` en lugar de generar un producto sin calibrar. Entre los modelos de cámara disponibles, solo el filtro F988 queda fuera de este rango; en su lugar, utiliza el flujo de trabajo con paneles de reflectancia — véase [Flujos de trabajo de reflectancia](reflectance.md).

Para conocer los modelos de sensores, los transportes y los ID de los sensores, consulte la [descripción general de DAQ](README.md). Para saber cómo se consume el sello de límite máximo durante el procesamiento, consulte [Grabación y el formato .daq](recording.md).
