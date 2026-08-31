---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Descargar

Descarga la última versión de Chloros para empezar a trabajar con el procesamiento de imágenes multiespectrales.

### Requisitos del sistema

#### Windows

| Requisito          | Mínimo                                              | Recomendado                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Sistema operativo** | Windows 10 (64 bits)                                  | Windows 11 (64 bits)                                  |
| **Procesador**        | Intel Core i5 o equivalente                          | Intel Core i7 o superior                              |
| **Memoria (RAM)**     | 8 GB                                                  | 16 GB o más                                         |
| **Tarjeta gráfica**    | Compatible con DirectX 11                                | GPU NVIDIA con 4 GB o más de VRAM                            |
| **Almacenamiento**          | 6 GB de espacio libre                                       | SSD con 10 GB o más de espacio libre                            |
| **Pantalla**          | 1920x1080                                            | 2560x1440 o superior                                  |
| **Internet**         | Necesario para la activación de la licencia [opcional] Chloros+ | Necesario para la activación de la licencia [opcional] Chloros+ |

#### Linux amd64 (x86_64)

| Requisitos       | Mínimos                    | Recomendados               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribución**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Procesador**     | x86\_64 (Intel/AMD)        | Intel Core i7 o superior   |
| **Memoria (RAM)**  | 8 GB                        | 16 GB o más              |
| **Tarjeta gráfica** | Ninguna (procesamiento por CPU)      | GPU NVIDIA con 4 GB o más de VRAM |
| **Almacenamiento**       | 2 GB de espacio libre             | SSD con 10 GB o más de espacio libre       |
| **Python**        | Python 3.7+ (para SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Requisito      | Mínimo                      | Recomendado                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Plataforma**     | NVIDIA Jetson con JetPack 6 | Jetson Orin NX de 16 GB o AGX Orin |
| **Memoria (RAM)** | 8 GB (compartida entre GPU y CPU)     | 16 GB o más compartidos                    |
| **Almacenamiento**      | 2 GB de espacio libre               | SSD NVMe con 10 GB o más de espacio libre        |
| **Python**       | Python 3.7+ (para SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Aceleración por GPU**: los usuarios de Chloros+ con GPUs de NVIDIA pueden utilizar la aceleración CUDA para un procesamiento significativamente más rápido. Esto funciona tanto en Windows (GPU de sobremesa) como en Linux (GPU de sobremesa y NVIDIA Jetson). Los usuarios de Chloros+ también se benefician del procesamiento multihilo para obtener la máxima velocidad.
{% endhint %}

***

## Descargar Chloros

### Última versión estable: Versión 1.2.0

<!-- NOLAN: replace installer links + release date for 1.2.0 — the three download buttons below still point at the 1.1.0 Google Drive files, and the release date needs to be added to the heading above. -->



### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Descargar Chloros para Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Descargar Chloros para Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Descargar Chloros para Linux arm64 / Jetson (.deb)</a>

#### Instalador de Windows (GUI + CLI + Backend)

* **Tipo de archivo**: .exe (instalador de Windows)**Pasos de instalación:**

1. Descarga el archivo .exe anterior
2. Haz doble clic en el instalador para iniciar la instalación
3. Sigue las instrucciones del asistente de instalación
4. Elige el directorio de instalación (por defecto: `C:\Program Files\MAPIR\Chloros\`)
5. Finaliza la instalación y ejecuta Chloros o Chloros CLI
6. Inicia sesión con tu [cuenta de MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (o continúa con la versión gratuita)

{% hint style="success" %}
El instalador añade automáticamente `chloros-cli` a la variable PATH de tu sistema para permitir el acceso desde la línea de comandos.
{% endhint %}

#### Linux amd64 (paquete .deb — CLI + backend)

* **Tipo de archivo**: .deb (paquete de Debian/Ubuntu)
* **Arquitectura**: x86_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (paquete .deb — CLI + backend)

* **Tipo de archivo**: .deb (JetPack 6)
* **Arquitectura**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Consulta la [Instalación de Linux](linux/linux-installation.md) para obtener instrucciones detalladas de configuración y la [Guía de NVIDIA Jetson](linux/nvidia-jetson-guide.md) para obtener orientación específica sobre Jetson.

#### Python SDK (todas las plataformas)

Cada instalador incluye un paquete «wheel» `chloros_sdk` correspondiente, por lo que la versión SDK siempre coincide con la interfaz gráfica de usuario (GUI)/CLI/backend instalado. En Windows, el instalador lo instala automáticamente en el sistema Python; en Linux, el `.deb` coloca el wheel en `/usr/lib/chloros/sdk/` y muestra el comando de instalación:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Para los hosts que solo utilizan pip (sin el paquete Chloros instalado), el SDK también está disponible en PyPI:

```bash
pip install chloros-sdk
```

Véase [API : Python SDK](api-python-sdk.md) y la [Referencia de SDK](reference/sdk-reference.md) para consultar la documentación.

{% hint style="info" %}
**Usuarios de Linux**: El paquete `.deb` instala CLI y el backend. Linux no dispone de interfaz gráfica de usuario (GUI); toda la interacción se realiza a través de CLI o SDK.
{% endhint %}

***

## Recursos adicionales

### Python SDK

Para desarrolladores y flujos de trabajo de automatización, instale el Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Documentación**: [API: Python SDK](api-python-sdk.md)**Requisitos**: Debe estar instalado Chloros (instalador de Windows o paquete de Linux `.deb`), se requiere iniciar sesión con la licencia de Chloros+***

## Qué incluye

### Instalador de Windows

* ✅ **Chloros GUI**: interfaz gráfica con todas las funciones
* ✅ **Chloros CLI** - Interfaz de línea de comandos (requiere una licencia Chloros+)
* ✅ **Chloros Backend** - Motor de procesamiento
* ✅ **Perfiles de cámara** - Plantillas de cámara MAPIR preconfiguradas

### Paquete .deb de Linux

* ✅ **Chloros CLI** - Interfaz de línea de comandos (requiere licencia Chloros+)
* ✅ **Chloros Backend** - Motor de procesamiento
* ✅ **Perfiles de cámara** - Plantillas de cámara MAPIR preconfiguradas
* ❌ Sin interfaz gráfica de usuario: Linux es una versión sin interfaz gráfica, solo CLI/SDK

### Python SDK (pip, todas las plataformas)

* ✅ **Chloros SDK** - Python API (requiere licencia Chloros+)***

## Actualiza a Chloros+

Desbloquea funciones avanzadas con una suscripción a Chloros+:

* 🚀 **Procesamiento multihilo**: procesa imágenes en paralelo
* ⚡ **Aceleración por GPU (CUDA)**: aprovecha la potencia de las GPU de NVIDIA
* 💻 **Acceso a CLI** - Automatiza con herramientas de línea de comandos
* 🐍 **Python SDK** - Acceso programático a API
* 📱 **Múltiples dispositivos**: utilízalo en entre 2 y más de 10 dispositivos (dependiendo del plan)
* **🐻 Método avanzado de eliminación de ruido con reconocimiento de texturas**: un método de eliminación de ruido de alta calidad que tiene en cuenta los bordes, combinado con un modelo de eliminación de ruido basado en IA/ML que elimina casi todo el ruido del proceso.
* 🧮 **Fórmulas personalizadas**: crea índices multiespectrales personalizados

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Ver planes y precios de Chloros+</a></p>***

## Ayuda para la instalación

### Solución de problemas

**La instalación falla con el siguiente mensaje de error:**

* Asegúrate de tener derechos de administrador
* Desactiva temporalmente el software antivirus
* Comprueba que cumples los requisitos mínimos del sistema

**La aplicación no se inicia (Windows):**

* Comprueba que Windows 10/11 (64 bits) esté instalado
* Actualiza los controladores gráficos
* Consulte el Visor de eventos de Windows para obtener detalles sobre el error
* Póngase en contacto con el servicio de asistencia técnica y facilite los registros de errores

**CLI no se inicia (Linux):**

* Comprueba que el paquete `.deb` esté instalado correctamente: `dpkg -l | grep chloros`
* Comprueba los permisos: `sudo chmod +x /usr/bin/chloros-cli`
* Ejecuta el diagnóstico: `chloros-cli selftest`
* Comprueba si faltan bibliotecas: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problemas con la activación de la licencia:**

* Asegúrate de que la conexión a Internet esté activa
* Verifica las credenciales en [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Comprueba que el cortafuegos no esté bloqueando Chloros
* Consulta [Chloros+ Inicio de sesión](chloros+-login.md) para obtener instrucciones detalladas

### Asistencia técnica

¿Necesitas ayuda con la instalación o la configuración?

* 📧 **Correo electrónico**: info@mapir.camera
* 🌐 **Página web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentación**: [Primeros pasos](./)
* ❓ **Preguntas frecuentes**: [Preguntas frecuentes](faq.md)***

## Actualizaciones de software

Chloros comprueba si hay actualizaciones, avisa cuando hay una nueva versión disponible y ofrece un enlace a esta página de descarga; para actualizar, basta con ejecutar el nuevo instalador firmado. Tu configuración y tus proyectos se conservan tras las actualizaciones. En Linux y Jetson, `chloros-cli update` comprueba si hay una versión más reciente y ofrece la posibilidad de descargar e instalar el `.deb` correspondiente (este comando solo está disponible en Linux).

***

## Registro de cambios**Versión 1.2.0 (última)**— consulta**Novedades de Chloros 1.2.0** en la página [Introducción](./) para ver la lista completa de características.

<details>

<summary>Versión 1.0.5</summary>

**Fecha de lanzamiento: 10 de febrero de 2026**

**Nuevas características*** **Método de despixelar con reconocimiento de texturas \[Solo Chloros+] —** El método «Texture Aware» utiliza un despixelar de alta calidad sensible a los bordes, combinado con un modelo de eliminación de ruido basado en IA/ML que elimina casi todo el ruido del proceso de despixelar.
* **Compatibilidad con los objetivos de calibración T4P*** **Procesamiento más rápido en la GPU de Chloros+, mejor gestión de la memoria**

**Corrección de errores*** Interfaz de usuario (GUI) completamente nueva; ahora debería funcionar en todos los ordenadores Windows.

</details>

<details>

<summary>Versión 1.0.4</summary>

**Fecha de lanzamiento: 5 de enero de 2026**

**Nuevas funciones*** **Alternancia entre imagen y metadatos**: Se ha añadido una opción en el explorador de archivos para ver los metadatos de la imagen seleccionada en una tabla en lugar de en la cuadrícula de imágenes
* **Control deslizante de zoom de la cuadrícula de imágenes**: Nuevo control deslizante en la interfaz de usuario para ajustar el tamaño de las miniaturas (también es compatible con CTRL + rueda del ratón)
* **Botones de exportación de la cuadrícula de imágenes**: Botones en la fila superior para cambiar las miniaturas de JPG a exportaciones procesadas (Objetivos, Reflectancia, Índice, LUT)
* **Pestaña «Mapa»**: Nuevo mapa 2D interactivo que muestra marcadores de ubicación GPS de las imágenes
  * Compatible con Google Maps y los mosaicos de mapas de ESRI (selecciona automáticamente el mejor servicio de mosaicos en función de la disponibilidad del nivel de zoom)
  * Vista previa de las miniaturas al pasar el ratón por encima de los marcadores del mapa

**Correcciones de errores*** Se ha mejorado la compatibilidad para instalar Chloros en ordenadores que no estén configurados en inglés

</details>

<details>

<summary>Versión 1.0.3</summary>

**Fecha de lanzamiento: 20 de diciembre de 2025**

**Nuevas funciones*** Lanzamiento inicial

**Mejoras*** Lanzamiento inicial

**Corrección de errores*** Lanzamiento inicial

**Problemas conocidos*** Lanzamiento inicial

</details>***

## Acuerdo de licencia**Software propietario** - Copyright (c) 2026 MAPIR Inc.

Queda prohibido el uso, la distribución o la modificación no autorizados.

**Versión gratuita**: Disponible para uso personal y comercial con limitaciones de funciones**Chloros+**: Licencia por suscripción para funciones avanzadas e implementaciones comerciales
