# Uso de Chloros con asistentes de IA

Este manual está dirigido a dos públicos: las personas y los asistentes de IA con los que las personas trabajan cada vez más. Cada página incluye valores exactos, valores predeterminados y comandos que se pueden copiar y pegar, de modo que un asistente (Claude, ChatGPT, Copilot, un agente de programación, etc.) pueda escribir una automatización Chloros que funcione a la primera.

Versión de Chloros: **

1.2.0**. Plataformas CLI/SDK: Windows 10/11 x64 y Linux (x86_64 / Jetson aarch64).

## Qué entregar a tu asistente

| Recurso | URL | Para qué sirve |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Índice legible por máquina de todas las páginas de este manual. |
| **Referencia de CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | La superficie de comandos completa de `chloros-cli`: todos los comandos, indicadores, valores por defecto, códigos de salida y reglas de la carpeta de salida. Escrito para su uso con modelos de lenguaje grande (LLM). |
| **Referencia de SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | `chloros_sdk` Python API al completo: clases, firmas, excepciones y ejemplos prácticos. Escrito para su uso con LLM. |
| **Cualquier página como Markdown sin formato** | añade `.md` a la página URL | p. ej. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` devuelve la página en formato Markdown sin formato, ideal para pegarla en una ventana de contexto o recuperarla desde un agente. |

Enlaces dentro del manual: [Referencia de CLI](reference/cli-reference.md) · [Referencia de SDK](reference/sdk-reference.md).

{% hint style="info" %}
Las dos páginas de referencia son autónomas: un asistente que haya leído una de ellas no necesita el resto del manual para escribir un script correcto.
{% endhint %}

## Recetas de comandos

Copia, rellena el campo `<placeholders>` y pégalo en tu asistente.

### 1. Procesar una carpeta de vuelos en NDVI

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

### 3. Conectar una matriz LATTICE y capturar

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Grabar espectros del sensor de luz DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
La creación de scripts de DAQ desde la línea de comandos siempre se realiza a través de la familia `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Otros subcomandos de `daq` que tu asistente pueda inventar no están disponibles en las versiones distribuidas y provocan un error al ejecutarse.
{% endhint %}

## Por qué los scripts escritos por IA funcionan bien con Chloros

Cada uno de estos es un comportamiento real y verificado de Chloros 1.2.0: eliminan los modos de fallo clásicos de la automatización escrita por máquinas:

* **Sin complicaciones de configuración.**Los ayudantes de conexión inteligente de SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) y los puntos de entrada de procesamiento (`ChlorosLocal`, `process_folder`)**inician automáticamente el backend local**. Un script generado no necesita que la interfaz gráfica esté abierta ni que el servidor se haya iniciado manualmente; solo requiere que esté instalado el paquete desktop/CLI.
* **Todo el proceso se ejecuta en una sola llamada.** `chloros_sdk.process_folder("path", indices=["NDVI"])` ejecuta de principio a fin: importación → calibración → reflectancia → exportación del índice. Menos superficie de error, menos puntos en los que un script generado pueda fallar.
* **Las ejecuciones sin resultados se autodiagnostican.** Tras `process()`, el resumen de la ejecución se adjunta al resultado, y cada indicación sobre el procesamiento (p. ej., *por qué* una ejecución no ha generado resultados) se vuelve a emitir como un Python `UserWarning`, de modo que incluso un script que nunca inspeccione el diccionario de resultados muestra el diagnóstico.
* **El CLI falla de forma evidente.**Una ejecución `chloros-cli process` que solicitó productos pero no generó ninguno muestra `Processing finished but wrote no image products.` y**termina con un valor distinto de cero**, por lo que los scripts de shell y la integración continua (CI) lo detectan con una simple comprobación del código de salida. Las ejecuciones correctas informan de `Image products written: N`.

Una asimetría que un asistente debe conocer: el `process()` del SDK **no** genera deliberadamente un error en una ejecución sin productos; en su lugar, lo indica a través del resumen y las sugerencias. Si una cadena de procesamiento Python debe detenerse en una ejecución vacía, comprueba el resumen (la receta 2 lo hace).

## Advertencias

* **Se requiere inicio de sesión en Chloros+.** Los niveles CLI y SDK requieren un nivel de Chloros+, que se aplica del lado del servidor: las solicitudes fallan con el código `401 AUTH_REQUIRED` si no se ha iniciado sesión y con el código `403 PLAN_UPGRADE_REQUIRED` en el nivel gratuito. Ejecuta `chloros-cli login` una vez por máquina antes de ejecutar los scripts generados. Consulta [Chloros+ Inicio de sesión](chloros+-login.md).
* **Los comandos de captura controlan hardware real.** Los comandos `lattice` / `daq` / `project` y los objetos de sesión SDK se conectan, transmiten y activan cámaras y sensores físicos. Revisa el script generado antes de su primera ejecución y ejecútalo con el hardware supervisado.
* **Comprueba aleatoriamente los resultados.** Verifica las carpetas de productos y algunos valores de píxeles antes de publicar los resultados. En concreto, los archivos TIFF de reflectancia se escalan por fuente; consulta la etiqueta XMP de `Chloros:PixelScale` (LATTICE: 32768 = reflectancia 1,0; Survey3: 65535) en lugar de suponer un divisor. Ambas páginas de referencia documentan esto en el apartado «Lectura de píxeles de reflectancia».
* **Pequeños detalles que pueden causar problemas en el código generado:**`pool-record` escribe en el sistema de archivos del**host del backend** (por defecto, `~/Documents/DAQ Live View/`); en equipos con varias interfaces de red, es preferible utilizar `daq pool-connect --eth-host <ip-or-hostname>` en lugar de la detección automática; y utiliza `http://127.0.0.1:5000` (nunca `localhost`) siempre que aparezca un backend URL.
