# Descripción general de Linux

Chloros 1.3.0 ofrece compatibilidad nativa con Linux para la **CLI**y el**Python SDK** —procesamiento de imágenes multiespectrales sin interfaz gráfica, además de control en tiempo real de cámaras LATTICE y sensores de luz DAQ— en estaciones de trabajo, servidores y dispositivos periféricos NVIDIA Jetson con Linux.

{% hint style="info" %}
**No hay interfaz gráfica de usuario (GUI) de escritorio en Linux.**La interfaz gráfica de usuario de Chloros para escritorio es exclusiva de Windows. Los usuarios de Linux interactúan con Chloros a través de la [CLI](../CLI.md) y el [Python SDK](../api-python-sdk.md). El `.deb` añade una entrada**Chloros CLI** al menú de aplicaciones; simplemente abre un emulador de terminal que ejecuta `chloros-cli`.
{% endhint %}

***

## Matriz de compatibilidad con plataformas

| Característica | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Interfaz gráfica de escritorio** | Sí | N/A | No | No |
| **Interfaz de línea de comandos (CLI)** (`chloros-cli`) | Sí | Sí | Sí | Sí |
| **SDK de Python** (`chloros-sdk`) | Sí | Sí | Sí | Sí |
| **Flujo de trabajo de procesamiento de imágenes** | Sí | Sí | Sí | Sí |
| **Control de cámaras LATTICE (en directo)** | Sí (pestaña «Cámaras») | Sí (`chloros-cli lattice`, SDK) | Sí | Sí |
| **Sensores de luz DAQ (en directo)** | Sí (pestaña «Sensores de luz») | Sí (`chloros-cli daq pool-*`, SDK) | Sí | Sí |
| **Sincronización temporal PTP (el host es el «grandmaster»)** | Sí | Sí (`chloros-cli time-sync`) | Sí | Sí |
| **Aceleración por GPU (CUDA)** | Sí | Sí | Sí | Sí (JetPack 6) |
| **Desentrelazado con reconocimiento de texturas** (requiere una GPU con CUDA) | Sí (Chloros+) | Sí (Chloros+) | Sí (Chloros+) | Sí (Chloros+) |
| **Adaptación dinámica de computación** | Sí | Sí | Sí | Sí |
| **Backend como servicio del sistema** (`chloros-backend@<user>.service`) | No | No | Sí (opcional) | Sí (opcional) |
| **Actualizador in situ** (`chloros-cli update`) | No (ejecutar el instalador) | No (ejecutar el instalador) | Sí | Sí |

***

## Arquitecturas compatibles

| Arquitectura | Descripción | Paquete |
| --- | --- | --- |
| **amd64 (x86_64)** | Procesadores estándar de escritorio/servidor (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | Procesadores ARM — Familia NVIDIA Jetson Orin (JetPack 6) | `chloros_<version>_arm64_jp6.deb` |

## Distribuciones de Linux compatibles

* **Ubuntu 22.04 LTS o posterior** (amd64)
* **Debian 12 o posterior** (amd64)
* **NVIDIA JetPack 6** (arm64 — plataformas Jetson Orin)

***

## Qué obtienen los usuarios de Linux

* **Chloros CLI** — la interfaz de línea de comandos completa para el procesamiento por lotes, la automatización y la creación de scripts
* **Chloros Python SDK**: interfaz de programación en Python para flujos de trabajo de investigación y herramientas personalizadas, instalada por el paquete `.deb` para el sistema `python3` (en `/usr/lib/chloros/sdk/` hay un «wheel» con la versión compatible para entornos virtuales; PyPI es para máquinas sin el paquete Chloros)
* **Control de cámaras LATTICE**: detecta, conecta, configura y captura imágenes desde cámaras LATTICE y conjuntos sincronizados de varias cámaras a través de `chloros-cli lattice` y el SDK; `.deb` incluye el entorno de ejecución del SDK de Arena que requieren las cámaras
* **Control de sensores de luz DAQ**: conecta sensores DAQ-U/M/E, transmite espectros calibrados y graba archivos `.daq` a través de `chloros-cli daq pool-*` y el SDK
* **Sincronización de tiempo PTP**: el backend de Chloros ejecuta el «grandmaster» PTP al que se sincronizan las cámaras LATTICE y los sensores DAQ-E; compruébalo con `chloros-cli time-sync` y manténlo en funcionamiento en modo sin interfaz gráfica con la unidad systemd `chloros-backend@<user>.service` (consulta [Instalación en Linux](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Automatización de proyectos**: ejecuta proyectos guardados en modo sin interfaz gráfica con `chloros-cli project` y el `open_project` del SDK
* **Aceleración por GPU**: procesamiento acelerado por CUDA en GPUs de NVIDIA (GPUs de sobremesa en amd64 y la GPU del Jetson con JetPack 6)
* **Adaptación dinámica del cálculo**: detección automática del hardware y selección de la estrategia de procesamiento, con la opción `CHLOROS_STRATEGY` como vía de escape para expertos
* **Todas las funciones de procesamiento**: el mismo flujo de trabajo que en Windows: calibración, corrección de viñeteado, índices de vegetación y todos los formatos de exportación
* **Funcionalidades de Chloros+**: procesamiento multihilo (en pipeline), debayer con reconocimiento de texturas (en una GPU CUDA) e índices personalizados, con un plan de pago de Chloros+

## Lo que los usuarios de Linux no tienen

* **Interfaz gráfica de usuario (GUI) de escritorio**: no hay interfaz gráfica; toda la interacción se realiza a través de la CLI o el Python SDK
* **Visor de imágenes**: no hay visor de imágenes interactivo, vista en cuadrícula ni marcadores de mapa
* **Gestión visual de proyectos**: los proyectos se crean y gestionan mediante comandos de lCLI y llamadas al SDK (el propio hardware —cámaras, sensores, captura— sigue siendo totalmente controlable desde la terminal)

***

## Requisitos de licencia

El acceso a la CLI y al SDK requiere un **nivel de pago de Chloros+ — Copper o superior**(Copper, Bronze, Silver, Gold). El nivel gratuito**Iron** no tiene acceso a la CLI ni al SDK. Esta restricción se aplica desde el backend, no solo desde la CLI:

| Situación | Respuesta del backend |
| --- | --- |
| Sin iniciar sesión | `401` con `error_code: AUTH_REQUIRED` |
| Iniciado sesión en el nivel gratuito «Iron» | `403` con `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` funciona en cualquier nivel —es la única ruta exenta del filtro—, por lo que el motivo del rechazo siempre es visible.

***

## Primeros pasos en Linux

1. **Instala Chloros**: consulta [Instalación en Linux](linux-installation.md) para la instalación de `.deb`
2. **Verifica**: `chloros-cli --version` muestra `Chloros CLI 1.3.0`; `chloros-cli selftest` ejecuta el diagnóstico de 7 pasos
3. **Comprueba el SDK de Python** — `.deb` lo instala: `python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"` (en un virtualenv, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`)
4. **Inicia sesión** — `chloros-cli login your@email.com 'your-password'` (una vez por equipo; las actualizaciones del paquete conservan los datos de inicio de sesión; al eliminar el paquete, estos se borran)
5. **Procesa tu primer conjunto de datos** — `chloros-cli process ~/datasets/flight001`

Para NVIDIA Jetson, consulta la [Guía de NVIDIA Jetson](nvidia-jetson-guide.md) específica, donde encontrarás información sobre la configuración propia de la plataforma, el comportamiento térmico y la implementación sobre el terreno.

***

## Próximos pasos

* [Instalación en Linux](linux-installation.md): instalación detallada, ubicaciones de los archivos y resolución de problemas para amd64 y arm64
* [Guía de NVIDIA Jetson](nvidia-jetson-guide.md) — configuración específica para Jetson, comportamiento térmico y de la memoria, y despliegue en campo
* [CLI: línea de comandos](../CLI.md) — la guía de la CLI
* [API: Python SDK](../api-python-sdk.md) — la guía del SDK
* [Referencia de CLI](../reference/cli-reference.md) y [Referencia de SDK](../reference/sdk-reference.md): listados exhaustivos de comandos y API para la versión 1.3.0
* [Adaptación dinámica de la computación](../processing-architecture/dynamic-compute-adaptation.md) — cómo se adapta Chloros a tu hardware

{% hint style="info" %}
**Lectura de este manual mediante programación.** Cada página también se ofrece en formato Markdown sin formato en su propia URL más `.md` (por ejemplo, `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), y el índice de todo el manual se publica en [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
