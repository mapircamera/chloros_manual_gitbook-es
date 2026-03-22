# Descripción general de Linux

Chloros 1.1.0 ofrece compatibilidad nativa con Linux para **CLI**y**Python SDK**, lo que permite el procesamiento de imágenes multiespectrales sin interfaz gráfica en estaciones de trabajo, servidores y dispositivos periféricos NVIDIA Jetson.

{% hint style="info" %}
**No hay interfaz gráfica de usuario en Linux.** La interfaz gráfica de usuario de escritorio Chloros solo está disponible en Windows. Los usuarios de Linux interactúan con Chloros a través de [CLI](../CLI.md) y [Python SDK](../api-python-sdk.md).
{% endhint %}

***

## Matriz de compatibilidad de plataformas

| Característica | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Interfaz gráfica de usuario de escritorio** | Sí | N/A | No | No |
| **CLI** | Sí | Sí | Sí | Sí |
| **Python SDK** | Sí | Sí | Sí | Sí |
| **Aceleración por GPU (CUDA)** | Sí | Sí | Sí | Sí (JetPack 6) |
| **Desbayering sensible a texturas** | Sí (Chloros+) | Sí (Chloros+) | Sí (Chloros+) | Sí (Chloros+) |
| **Adaptación dinámica de computación** | Sí | Sí | Sí | Sí |***

## Arquitecturas compatibles

| Arquitectura | Descripción | Método de instalación |
| --- | --- | --- |
| **amd64 (x86_64)** | Procesadores estándar de escritorio/servidor (Intel, AMD) | Paquete `.deb` |
| **arm64 (aarch64)** | Procesadores basados en ARM, principalmente NVIDIA Jetson | Paquete `.deb` (JetPack 6) |

## Distribuciones compatibles con Linux

* **Ubuntu 20.04+** (amd64)
* **Debian 11+** (amd64)
* **NVIDIA JetPack 6** (arm64 — plataformas Jetson)***

## Qué obtienen los usuarios de Linux

* **Chloros CLI** — Interfaz de línea de comandos completa para procesamiento por lotes, automatización y creación de scripts
* **Chloros Python SDK** — Interfaz programática Python (`pip install chloros-sdk`) para la integración en flujos de trabajo de investigación y herramientas personalizadas
* **Aceleración por GPU** — Procesamiento acelerado por CUDA en GPU NVIDIA (escritorio y Jetson)
* **Adaptación dinámica del cálculo** — Detección automática del hardware y optimización de la estrategia de procesamiento
* **Todas las funciones de procesamiento** — El mismo flujo de trabajo de procesamiento multiespectral que Windows (calibración, corrección de viñeteado, índices de vegetación, todos los formatos de exportación)
* **Funciones de Chloros+** — Procesamiento multihilo, debayer con reconocimiento de texturas, índices personalizados (con licencia Chloros+)

## Lo que los usuarios de Linux no obtienen

* **Interfaz gráfica de usuario de escritorio** — Sin interfaz gráfica; toda la interacción se realiza a través de CLI o Python SDK
* **Visor de imágenes** — No hay visor de imágenes interactivo, vista en cuadrícula ni marcadores de mapa
* **Gestión visual de proyectos** — Los proyectos se gestionan mediante comandos CLI y llamadas SDK***

## Primeros pasos con Linux

1. **Instalar Chloros** — Consulte [Instalación de Linux](linux-installation.md) para la instalación del paquete `.deb`
2. **Instala Python SDK** (opcional) — `pip install chloros-sdk`
3. **Activa tu licencia** — `chloros-cli login your@email.com 'password'`
4. **Procesa tu primer conjunto de datos** — `chloros-cli process ~/datasets/flight001`

Para los usuarios de NVIDIA Jetson, consulta la [Guía de NVIDIA Jetson](nvidia-jetson-guide.md) específica para la configuración y optimización de la plataforma.

***

## Próximos pasos

* [Instalación de XPROTX](linux-installation.md) — Instrucciones detalladas de instalación para amd64 y arm64
* [Guía de NVIDIA Jetson](nvidia-jetson-guide.md): configuración específica para Jetson, gestión térmica e implementación en campo
* [CLI: Línea de comandos](../CLI.md): referencia completa de CLI
* [API : Python SDK](../api-python-sdk.md) — Referencia completa de SDK
* [Adaptación dinámica de computación](../processing-architecture/dynamic-compute-adaptation.md) — Cómo se adapta Chloros a su hardware
