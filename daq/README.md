# Sensores de luz DAQ

> **¿Buscas información sobre el hardware?**Los sensores en sí —modelos, montaje, tapas, puertos, alimentación y la aplicación SCANNER— se describen en el**[manual de usuario de DAQ](https://mapir.gitbook.io/daq)**. Este capítulo trata sobre su uso desde Chloros. Los sensores de luz**DAQ** de

MAPIR miden la luz ambiental en forma de espectros calibrados radiométricamente. En Chloros desempeñan dos funciones:

* **Un instrumento espectral autónomo**: gráficos de espectro en tiempo real, datos colorimétricos y grabaciones `.daq`, todo ello desde la [pestaña «Sensores de luz»](gui.md), la [CLI](cli-quick-start.md) o el SDK de Python.
* **Una fuente de irradiancia descendente para la reflectancia**: durante el procesamiento, Chloros interpolará tus lecturas `.daq` a la marca de tiempo de exposición de cada captura y utilizará la luz descendente medida para convertir la radiancia de la cámara en reflectancia (`--reflectance-source daq`); no se requiere un panel en la escena para las bandas calibradas.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Tres modelos, un formato de datos

| Modelo | Transporte | Detección |
| --- | --- | --- |
| **DAQ-U** | USB (serie) | escaneo de puerto serie |
| **DAQ-M** | Bluetooth de baja energía | escaneo BLE por nombre |
| **DAQ-E** | Ethernet (IPv4, alimentación PoE) | mDNS `_daq-e._tcp` (nombre de host `daq-e-<id>.local`) |**DAQ-E-S (novedad en la versión 1.3.0).** Un DAQ-E con un difusor permanente y un sensor de inclinación. Chloros lo incluye como `DAQ-E-S` y funciona exactamente igual que un DAQ-E, salvo que no admite corrección de tapa y cada lectura incluye la inclinación, el balanceo y el cabeceo de la unidad. Véase [DAQ-E-S: difusor fijo e inclinación](caps-and-range.md#daq-e-s-fixed-diffuser-and-tilt).

Los tres utilizan el mismo protocolo de comunicación y proporcionan datos idénticos:

* Un **espectro de 135 puntos de 340 a 1010 nm en intervalos de 5 nm**, además de los valores triestímulos CIE XYZ, en cada fotograma.
* **Irradiancia espectral calibrada radiométricamente en W/m²/nm**: el paquete de calibración de fábrica de cada unidad (además de su perfil de corrección de tapa activo) se aplica antes de que los datos lleguen a ti.
* El mismo **formato de grabación `.daq`** (un archivo SQLite). El procesamiento posterior es idéntico independientemente del medio de transporte que haya generado el archivo.

Las pilas de transporte (serie USB, BLE, mDNS/zeroconf) están integradas en el backend de Chloros: no hay que instalar nada para comunicarse con cualquiera de los tres modelos desde la interfaz gráfica de usuario o mediante los comandos `pool-*` de la CLI.

***

## Rango calibrado: 340–1010 nm declarados, ~374–974 nm calibrados

El sensor declara la cuadrícula completa de 340–1010 nm, pero la ganancia radiométrica trazable según el NIST abarca aproximadamente **374–974 nm**. Chloros rechaza la división de reflectancia absoluta para cualquier banda de la cámara que tenga menos de la mitad de su peso espectral dentro de ese intervalo calibrado; la banda omitida se indica con el motivo de omisión `dls-uncalibrated-band-<nm>`.

Entre las referencias de filtros LATTICE disponibles, solo se ve afectada la **F988**:

La reflectancia de la F988 se calibra utilizando un panel de reflectancia en escena: la banda se encuentra fuera del rango calibrado del sensor de luz DAQ, por lo que Chloros aplica la captura más reciente del panel y la mantiene entre las lecturas del panel.

Si se procesa una captura de F988 disponiendo únicamente de datos del DAQ, Chloros rechaza la reflectancia basada en el DAQ para esa banda con el motivo de omisión `dls-uncalibrated-band-988` — el [flujo de trabajo del panel de reflectancia](../calibration-targets.md) es la vía compatible para F988.

***

## ID de sensores

Cada DAQ comunica un identificador de sensor estable. Su formato varía según el modelo:

| Modelo | Formato del identificador | Ejemplo |
| --- | --- | --- |
| DAQ-U | 5 octetos con guiones | `CB-7C-A8-2E-5F` |
| DAQ-M | 5 octetos con guiones | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

El ID del sensor es:

* el que aparece en cada archivo `.daq` que registra,
* la clave que utiliza Chloros para recuperar el paquete de calibración de fábrica de esa unidad,
* el valor que se pasa a `--sensor-id` en los comandos CLI de `pool-*`, y
* en el caso del DAQ-E, también su nombre de host mDNS (`daq-e-def330.local`), que es el valor que acepta `--eth-host`.

***

## Calibración de fábrica y la nube

Cada unidad DAQ se calibra individualmente en fábrica mediante una cadena radiométrica trazable según el NIST, y Chloros carga el paquete de calibración de cada unidad, asociado a su ID de sensor. El informe de calibración de cada unidad (PDF) se puede descargar desde la configuración del sensor en la [pestaña «Sensores de luz»](gui.md).

{% hint style="warning" %}
**Los modelos DAQ-U y DAQ-M requieren acceso a la nube para su calibración.**Ninguno de los dos modelos almacena nada en su memoria interna: sus paquetes de calibración de fábrica se encuentran en la nube de MAPIR y se recuperan mediante el ID del sensor (para luego almacenarse en la caché local). Chloros necesita una conexión a Internet para enviar datos calibrados en W/m²/nm desde un DAQ-U o un DAQ-M.**El DAQ-E es la excepción**: lleva su calibración en el propio dispositivo.
{% endhint %}
<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

***

## Dónde se guardan los registros

| Superficie | Destino predeterminado de `.daq` |
| --- | --- |
| Interfaz gráfica de usuario — Pestaña «Sensores de luz» | `<project folder>/light_sensor/` (los registros finalizados se añaden al proyecto automáticamente) |
| CLI: `daq pool-record` | `~/Documents/DAQ Live View/` en el equipo que ejecuta el backend (una ruta relativa `-o` se resuelve en la carpeta en la que se ejecutó el comando cuando el backend se encuentra en el mismo ordenador) |

Cada nombre de archivo `.daq` incluye el ID del sensor y una marca de tiempo.

***

## En este capítulo

* [**La pestaña «DAQ» en Chloros**](gui.md): guía completa de la interfaz gráfica de usuario: conexión de cada modelo, ajustes por sensor, gráficos de espectro, datos colorimétricos en tiempo real, reflectancia de doble sensor y grabación.
* [**Guía rápida de CLI (pool-\*)**](cli-quick-start.md): control de los sensores DAQ desde `chloros-cli daq pool-*`, la ruta de línea de comandos compatible.
* [**Perfiles de límites y rango calibrado**](caps-and-range.md): qué límites existen por modelo, cómo declararlos y el rango espectral calibrado en detalle.
* [**Grabación y el formato .daq**](recording.md): el formato SQLite de `.daq` y los flujos de trabajo de grabación.
* [**Redes DAQ-E y sincronización horaria**](ethernet-ptp.md): modos de transporte DAQ-E y sincronización horaria PTP.
* [**Flujos de trabajo de reflectancia**](reflectance.md): uso de datos de DAQ de radiación descendente para calcular la reflectancia.
* Para consultar la documentación completa a nivel de indicadores, véase la [Referencia de la CLI](../reference/cli-reference.md) (sección `chloros-cli daq`) y la [Referencia del SDK](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), ambas redactadas para que puedan ser utilizadas directamente por asistentes de IA.
