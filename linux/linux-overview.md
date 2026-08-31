# Descripción general de Linux

Chloros 1.2.0 ofrece compatibilidad nativa con Linux para **CLI**y**Python SDK** — procesamiento de imágenes multiespectrales sin interfaz gráfica, además de control en tiempo real de cámaras LATTICE y sensores de luz DAQ — en estaciones de trabajo, servidores y dispositivos periféricos NVIDIA Jetson.

{% hint style="info" %}
**No hay interfaz gráfica de usuario de escritorio en Linux.**La interfaz gráfica de usuario de escritorio de Chloros es exclusiva de Windows. Los usuarios de Linux interactúan con Chloros a través de [CLI](../CLI.md) y [Python SDK](../api-python-sdk.md). El `.deb` añade una entrada**Chloros CLI** al menú de tu aplicación; simplemente abre un emulador de terminal en el que se ejecuta `chloros-cli`.
{% endhint %}

***

## Matriz de compatibilidad con plataformas

| Característica | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Interfaz gráfica de usuario de escritorio** | Sí | N/A | No | No |
| **CLI** (`chloros-cli`) | Sí | Sí | Sí | Sí |
| **Python SDK** (`chloros-sdk`) | Sí | Sí | Sí | Sí |
| **Proceso de procesamiento de imágenes** | Sí | Sí | Sí | Sí |
| **Control de cámaras LATTICE (en directo)** | Sí (pestaña «Cámaras») | Sí (`chloros-cli lattice`, SDK) | Sí | Sí |
| **Sensores de luz DAQ (en directo)** | Sí (pestaña «Sensores de luz») | Sí (`chloros-cli daq pool-*`, SDK) | Sí | Sí |
| **Sincronización de tiempo PTP (el host es el grandmaster)** | Sí | Sí (`chloros-cli time-sync`) | Sí | Sí |
| **Aceleración por GPU (CUDA)** | Sí | Sí | Sí | Sí (JetPack 6) |
| **Desentrelazado con reconocimiento de texturas** | Sí (Chloros+) | Sí (Chloros+) | Sí (Chloros+) | Sí (Chloros+) |
| **Adaptación dinámica de cálculo** | Sí | Sí | Sí | Sí |
| **Backend como servicio del sistema** (`chloros-backend.service`) | No | No | Sí (opcional) | Sí (opcional) |
| **Actualizador in situ** (`chloros-cli update`) | No (ejecutar el instalador) | No (ejecutar el instalador) | Sí | Sí |***

## Arquitecturas compatibles

| Arquitectura | Descripción | Paquete |
| --- | --- | --- |
| **amd64 (x86_64)** | Procesadores estándar de sobremesa/servidor (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | Procesadores ARM — Familia NVIDIA Jetson Orin | `chloros_<version>_arm64_jp6.deb` (compilación de JetPack 6) |

## Distribuciones compatibles con Linux

* **Ubuntu 22.04 LTS o posterior** (amd64)
* **Debian 12 o posterior** (amd64)
* **NVIDIA JetPack 6** (arm64 — plataformas Jetson Orin)***

## Qué obtienen los usuarios de Linux

* **Chloros CLI**: la interfaz completa de línea de comandos para el procesamiento por lotes, la automatización y la creación de scripts
* **Chloros Python SDK** — interfaz programática Python para flujos de trabajo de investigación y herramientas personalizadas (instalable desde PyPI y también incluida en el paquete `.deb` como un archivo wheel con la versión correspondiente)
* **Control de cámaras LATTICE** — detecta, conecta, configura y captura imágenes desde cámaras LATTICE y conjuntos sincronizados de varias cámaras a través de `chloros-cli lattice` y SDK; el `.deb` incluye el entorno de ejecución Arena SDK que necesitan las cámaras
* **Control de sensores de luz DAQ**: conecta sensores DAQ-U/M/E, transmite espectros calibrados y graba archivos `.daq` a través de `chloros-cli daq pool-*` y SDK
* **Sincronización de tiempo PTP**: el backend Chloros ejecuta el «grandmaster» PTP al que se sincronizan las cámaras LATTICE y los sensores DAQ-E; compruébalo con `chloros-cli time-sync`, y manténlo en funcionamiento en modo sin interfaz gráfica con la unidad systemd `chloros-backend.service` (consulta [Instalación de Linux](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Automatización de proyectos**: ejecuta proyectos guardados en modo sin interfaz gráfica con `chloros-cli project` y `open_project` de SDK
* **Aceleración por GPU**: procesamiento acelerado por CUDA en GPUs de NVIDIA (de sobremesa y Jetson)
* **Adaptación dinámica del cálculo**: detección automática del hardware y selección de la estrategia de procesamiento, con la opción de anulación `CHLOROS_STRATEGY` como vía de escape para expertos
* **Todas las funciones de procesamiento**: el mismo flujo de trabajo que el Windows: calibración, corrección de viñeteado, índices de vegetación y todos los formatos de exportación
* **Funcionalidades de Chloros+**: procesamiento multihilo (en pipeline), debayer con reconocimiento de texturas e índices personalizados, con un plan de pago Chloros+

## Lo que los usuarios de Linux no obtienen

* **Interfaz gráfica de usuario de escritorio**: no hay interfaz gráfica; toda la interacción se realiza a través de CLI o Python SDK
* **Visor de imágenes**: no hay visor de imágenes interactivo, vista en cuadrícula ni marcadores en el mapa
* **Gestión visual de proyectos**: los proyectos se crean y se gestionan mediante comandos CLI y llamadas SDK (el hardware en sí —cámaras, sensores, captura— sigue siendo totalmente controlable desde el terminal)***

## Requisitos de licencia

El acceso a CLI y SDK requiere un **nivel de pago Chloros+ — Copper o superior**(Copper, Bronze, Silver, Gold). El nivel gratuito**Iron** no tiene acceso a CLI/SDK. El límite mínimo lo aplica el backend, no solo el CLI:

| Situación | Respuesta del backend |
| --- | --- |
| Sin iniciar sesión | `401` con `error_code: AUTH_REQUIRED` |
| Iniciado sesión en el nivel gratuito Iron | `403` con `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` funciona en cualquier nivel —es la única ruta exenta de la puerta de control—, por lo que el motivo del rechazo siempre es visible.

***

## Primeros pasos con Linux

1. **Instala Chloros**: consulta [Instalación de Linux](linux-installation.md) para la instalación de `.deb`
2. **Verifica**: `chloros-cli --version` imprime `Chloros CLI 1.2.0`; `chloros-cli selftest` ejecuta el diagnóstico de 7 pasos
3. **Instale el Python SDK** (opcional) — `pip install chloros-sdk`
4. **Inicia sesión** — `chloros-cli login your@email.com 'your-password'` (una vez por máquina y de nuevo tras cada actualización del paquete)
5. **Procesa tu primer conjunto de datos** — `chloros-cli process ~/datasets/flight001`

Para NVIDIA Jetson, consulta la [Guía de NVIDIA Jetson](nvidia-jetson-guide.md) específica para conocer la configuración propia de la plataforma, el comportamiento térmico y la implementación en campo.

***

## Próximos pasos

* [Instalación de Linux](linux-installation.md): instalación detallada, ubicaciones de los archivos y resolución de problemas para amd64 y arm64
* [Guía de NVIDIA Jetson](nvidia-jetson-guide.md): configuración específica de Jetson, comportamiento térmico y de la memoria, y despliegue en campo
* [CLI: Línea de comandos](../CLI.md) — la guía CLI
* [API: Python SDK](../api-python-sdk.md) — la guía SDK
* [Referencia de CLI](../reference/cli-reference.md) y [Referencia de SDK](../reference/sdk-reference.md) — Listas exhaustivas de comandos/API para la versión 1.2.0
* [Adaptación dinámica de la computación](../processing-architecture/dynamic-compute-adaptation.md): cómo Chloros se adapta a tu hardware

{% hint style="info" %}
**Lectura programática de este manual.** Cada página también se sirve como Markdown sin formato en su propia URL más `.md` (por ejemplo, `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), y el índice completo del manual se publica en [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
