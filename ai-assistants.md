# Cómo utilizar Chloros con asistentes de IA

Este manual está dirigido a dos públicos: las personas y los asistentes de IA con los que las personas trabajan cada vez más. Cada página incluye valores exactos, valores predeterminados y comandos que se pueden copiar y pegar, de modo que un asistente (Claude, ChatGPT, Copilot, un agente de programación, etc.) pueda escribir una automatización de Chloros que funcione a la primera.

Versión de Chloros: **

1.3.0**. Plataformas CLI/SDK: Windows 10/11 x64 y Linux (x86_64 / Jetson aarch64).

## Qué proporcionar a tu asistente

| Recurso | URL | Para qué sirve |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Índice legible por máquina de todas las páginas de este manual. |
| **Referencia de la CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | La superficie de comandos completa de `chloros-cli`: todos los comandos, indicadores, valores predeterminados, códigos de salida y reglas de la carpeta de salida. Escrita para su uso por parte de los LLM. |
| **Referencia del SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | La API de Python completa de `chloros_sdk`: clases, firmas, excepciones y ejemplos prácticos. Redactada para su uso por modelos de lenguaje grande (LLM). |
| **Cualquier página en formato Markdown sin formato** | añade `.md` a la URL de la página | p. ej., `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` devuelve la página en formato Markdown sin formato, ideal para pegarla en una ventana de contexto o recuperarla desde un agente. |

Enlaces dentro del manual: [Referencia de la CLI](reference/cli-reference.md) · [Referencia del SDK](reference/sdk-reference.md).

{% hint style="info" %}
Las dos páginas de referencia son autónomas: un asistente que haya leído una de ellas no necesita el resto del manual para escribir un script correcto.
{% endhint %}

## Deja que tu asistente ejecute Chloros directamente: `chloros-mcp`

Todo lo anterior hace que tu asistente *escriba* un script o un comando. Con `chloros-mcp`, un asistente que utilice el [Protocolo de Contexto de Modelo](https://modelcontextprotocol.io) (MCP), como Claude Desktop, Claude Code o Cursor, controla Chloros directamente. Le pides en lenguaje natural y él invoca las herramientas del servidor:

* **Procesar imágenes:** crear un proyecto, importar una carpeta de capturas, elegir la configuración, procesar e informar de lo que ha exportado la ejecución.
* **Manejo del hardware:** conecta cámaras LATTICE, matrices y sensores de luz DAQ, ofrece una vista previa, ajusta la exposición y el color, captura, graba y actualiza el firmware.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

Las herramientas del servidor abarcan lo que hace el [Python SDK](api-python-sdk.md). Se instala junto con el SDK, por lo que normalmente no hay que instalar nada más: en Windows, el instalador de Chloros coloca `chloros-mcp.exe` en la carpeta `Scripts` de Python, y el de Linux, `.deb`, coloca `chloros-mcp` en `/usr/bin`.

**Código de Claude:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**añádelo a `claude_desktop_config.json` (se abre en**Configuración > Desarrollador > Editar configuración**) y, a continuación, reinicia Claude Desktop.**Cursor** utiliza la misma entrada en `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Si el cliente indica que el servidor no se ha podido iniciar, es probable que `chloros-mcp` no figure en el `PATH` que detecta: indica la ruta completa en su lugar (`where chloros-mcp` en Windows, `which chloros-mcp` en Linux).

Es necesario tener Chloros instalado en el mismo ordenador y haber iniciado sesión allí, en la aplicación o mediante `chloros-cli login`, en cualquier plan de pago de Chloros+ (Copper o superior). El servidor no puede iniciar sesión por ti.

{% hint style="info" %}
Configuración del cliente para cada aplicación, opciones, todas las herramientas y cómo se comporta una ejecución: [Servidor MCP (asistentes de IA)](reference/sdk-reference.md#mcp-server-ai-assistants) en la Referencia del SDK.
{% endhint %}

## Recetas de comandos

Copia, rellena el campo `<placeholders>` y pégalo en tu asistente.

### 1. Procesar una carpeta de vuelo en NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Supervisar por lotes un directorio de capturas

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Conectar una matriz LATTICE y realizar una captura

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Registrar espectros de sensores de luz DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
La programación de scripts de DAQ desde la línea de comandos siempre se realiza a través de la familia `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Otros subcomandos de tipo `daq` que tu asistente pueda inventar no están disponibles en las versiones distribuidas y provocan un error al ejecutarse.
{% endhint %}

## Por qué los scripts escritos por IA funcionan bien con Chloros

Cada uno de estos es un comportamiento real y verificado de Chloros 1.3.0: eliminan los modos de fallo clásicos de la automatización escrita por máquinas:

* **Sin complicaciones de configuración.**Los ayudantes de conexión inteligente del SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) y los puntos de entrada de procesamiento (`ChlorosLocal`, `process_folder`)**inician automáticamente el backend local**. Un script generado no necesita que la interfaz gráfica esté abierta ni que se inicie manualmente un servidor; solo requiere que esté instalado el paquete de escritorio o CLI.
* **Todo el proceso se realiza en una sola llamada.** `chloros_sdk.process_folder("path", indices=["NDVI"])` ejecuta de principio a fin: importación → calibración → reflectancia → exportación del índice. Menos superficie de intervención, menos puntos en los que un script generado pueda fallar.
* **Las ejecuciones sin resultados se autodiagnostican.** Tras `process()`, el resumen de la ejecución se adjunta al resultado, y cada indicación sobre el procesamiento (p. ej., *por qué* una ejecución no ha generado ningún resultado) se vuelve a emitir como un `UserWarning` de Python, de modo que incluso un script que nunca inspeccione el diccionario de resultados muestra el diagnóstico.
* **La CLI falla de forma evidente.**Una ejecución `chloros-cli process` que solicitó productos pero no escribió ninguno muestra el mensaje `Processing finished but wrote no image products.` y**termina con un valor distinto de cero**, por lo que los scripts de shell y la integración continua (CI) lo detectan con una simple comprobación del código de salida. Las ejecuciones correctas informan de `Image products written: N`, además de `(plus M clamp masks)` cuando se han escrito máscaras. Los comandos de captura, del mismo modo, nombran todos los archivos que no han podido escribir y salen con el código `1`.
* **Las rutas relativas se interpretan tal y como espera el script.** Con un backend local, una ruta de salida relativa (`-o out/` en la CLI, `output_dir="out/"` en el SDK) se resuelve en relación con la carpeta desde la que se ejecuta el comando o el script.

Una asimetría que un asistente debe conocer: el `process()` del SDK **no** genera deliberadamente un error en una ejecución sin resultados; en su lugar, lo indica a través del resumen y las sugerencias. Si un proceso de Python debe detenerse en una ejecución vacía, comprueba el resumen (la receta 2 lo hace).

## Advertencias

* **Se requiere inicio de sesión en Chloros+.**La CLI y el SDK requieren un nivel**de pago** de Chloros+, lo cual se aplica del lado del servidor: las solicitudes fallan con el error `401 AUTH_REQUIRED` si no se ha iniciado sesión y con el error `403 PLAN_UPGRADE_REQUIRED` en el nivel gratuito. Ejecuta `chloros-cli login` una vez por máquina antes de ejecutar los scripts generados. Consulta [Inicio de sesión en Chloros+](chloros+-login.md).
* **Los comandos de captura controlan el hardware real.** Los comandos `lattice` / `daq` / `project`, los objetos de sesión del SDK y las herramientas de `chloros-mcp` se conectan, transmiten y activan cámaras y sensores físicos. Revisa el script generado antes de su primera ejecución y ejecútalo con el hardware supervisado.
* **Comprueba aleatoriamente los resultados.** Verifica las carpetas del producto y algunos valores de píxeles antes de publicar los resultados. En concreto, los archivos TIFF de reflectancia se escalan por fuente: consulte la etiqueta XMP de `Chloros:PixelScale` (los archivos LATTICE la incluyen: 32768 = 1,0 de reflectancia; los archivos Survey3 no la incluyen y utilizan 65535 = 1,0) en lugar de suponer un divisor. Ambas páginas de referencia documentan esto en el apartado «Lectura de píxeles de reflectancia».
* **Pequeños detalles que pueden causar problemas en el código generado:**`pool-record` escribe en el sistema de archivos del**host del backend** (por defecto, `~/Documents/DAQ Live View/`); en equipos con varias interfaces de red, es preferible utilizar `daq pool-connect --eth-host <ip-or-hostname>` en lugar de la detección automática; utiliza `http://127.0.0.1:5000` (nunca `localhost`) siempre que aparezca una URL de backend; un script que recorra el árbol de salida de un proyecto debe omitir las carpetas `clamp_masks/` (sus archivos `*_clampmask.tif` son máscaras, no imágenes; véase [Formatos de imagen de salida](output-image-formats.md#clamp-masks)); y en Windows, con las rutas largas desactivadas, mantén las carpetas de proyecto cortas: se rechaza cualquier carpeta de proyecto con un nombre de más de unos 130 caracteres antes de que comience el procesamiento.
