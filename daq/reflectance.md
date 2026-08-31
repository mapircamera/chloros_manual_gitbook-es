# Flujos de trabajo de reflectancia

Un sensor de luz DAQ convierte las imágenes radiométricas en datos de reflectancia. Existen dos flujos de trabajo distintos:

1. **Sensor único**: un DAQ mide la irradiancia descendente mientras una cámara captura la imagen, y el Chloros divide la radiancia de la cámara por esa referencia.
2. **Dos sensores**: dos sensores DAQ, uno orientado hacia el cielo y otro hacia un objeto, generan una curva de reflectancia espectral en tiempo real sin necesidad de una cámara.

## Un sensor + cámara (referencia descendente)

El DAQ actúa como sensor de luz descendente (DLS): la cámara mide la radiancia ascendente **L**(W/m²/sr/nm), el DAQ mide la irradiancia descendente**E** (W/m²/nm) y Chloros calcula la reflectancia por banda de la siguiente forma:

> ρ = π · L / E

La lectura del DAQ siempre **coincide en la marca de tiempo con la exposición**; por eso el DAQ y las cámaras comparten un reloj sincronizado por PTP (véase [Redes y sincronización horaria del DAQ-E](ethernet-ptp.md)). Colócate la gorra Sunshine Cosine para trabajar al aire libre y declárala correctamente; la declaración de la gorra escala directamente E (véase [Perfiles de gorras y rango calibrado](caps-and-range.md)). Para trabajos cuantitativos, recuerde la característica del instrumento: la irradiancia cuantitativa se obtiene a partir de una media de lecturas de al menos 15 s.

### Captura en directo

Asigna el DAQ a una cámara en la pestaña «Cámaras»: el panel de configuración de cada cámara cuenta con un menú desplegable **Sensor de luz** que muestra todos los DAQ conectados (DAQ-U/M/E) desde la pestaña «Sensores de luz»; en el caso de una matriz sincronizada, la selección de un sensor de luz para toda la matriz se aplica a todos los miembros (aunque las cámaras individuales pueden anularla). Una vez vinculado, los espectros del sensor se introducen en la ranura DLS de la cámara y los valores de reflectancia exportados se dividen por la lectura correspondiente.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->

Dos comportamientos que conviene conocer:

* **Si no hay ningún DAQ vinculado → la reflectancia se rechaza, no se simula.** Chloros rechaza el producto de reflectancia y registra el motivo del salto, en lugar de devolver silenciosamente un producto inferior.
* **La lectura utilizada se conserva.** Para cada fotograma de reflectancia, la lectura del DAQ realmente aplicada se escribe como un archivo adjunto `.daq` junto a las imágenes, de modo que la captura se pueda volver a procesar más adelante ([Grabación y el formato .daq](recording.md)).

### Procesamiento de imágenes grabadas

Para el procesamiento posterior al vuelo, graba un `.daq` durante la sesión y guárdalo junto con las imágenes: el proceso resuelve automáticamente la radiación descendente que coincide con la marca de tiempo, y recupera cualquier calibración de fábrica que falte de la nube de MAPIR la primera vez que se utiliza. Las grabaciones de la interfaz gráfica de usuario se añaden automáticamente al proyecto abierto cuando finalizan.

La referencia de reflectancia se puede seleccionar en el momento del procesamiento: `--reflectance-source` en `chloros-cli process`, o la configuración de la fuente de reflectancia en los Ajustes del proyecto de la interfaz gráfica de usuario:

| Valor | Comportamiento |
| --- | --- |
| `auto` (por defecto) | Un objetivo de calibración dentro del fotograma que haya superado el control de calidad es la referencia absoluta; la radiación descendente del DAQ (ρ = π·L/E) es la opción alternativa |
| `daq` | Autoridad del DAQ |
| `target` | Objetivo estricto dentro del encuadre; sin sustitución por el DAQ |

Consulte [Objetivos de calibración](../calibration-targets.md) para conocer los flujos de trabajo de los objetivos y el [capítulo LATTICE](../lattice/README.md) y la [Referencia CLI](../reference/cli-reference.md) para conocer el proceso completo de procesamiento. Al leer los píxeles de reflectancia exportados, utilice la escala indicada (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535); véase [Formatos de imagen de salida](../output-image-formats.md).

### Bandas fuera del rango calibrado del DAQ

El rango radiométricamente calibrado del DAQ es de ~374–974 nm. Chloros rechaza la reflectancia basada en el DAQ para cualquier banda de la cámara que tenga menos de la mitad de su peso espectral dentro de ese intervalo, indicando como motivo de omisión `dls-uncalibrated-band-<nm>`. Entre los modelos disponibles, esto solo afecta al F988: la reflectancia del F988 se calibra utilizando un panel de reflectancia integrado en la escena; dado que la banda se encuentra fuera del rango calibrado del sensor de luz del DAQ, Chloros aplica la captura más reciente del panel y la mantiene entre observaciones del panel. Si una cámara F988 funciona solo en modo DAQ, el código Chloros rechaza la reflectancia basada en DAQ para esa banda con el motivo de omisión `dls-uncalibrated-band-988`; el flujo de trabajo con el panel es la opción recomendada.

## Sensor dual (ambiental + objeto)

Dos sensores DAQ —cualquier par, en cualquier sistema de transporte— proporcionan un espectro de reflectancia en tiempo real sin necesidad de cámara: un sensor apunta al cielo (**Fuente de luz ambiental**) y otro al sujeto (**Escáner de objeto**), y Chloros calcula por longitud de onda:

> R(λ) = objeto(λ) / ambiental(λ)

(cero cuando ambiental ≤ 0).

### En la interfaz gráfica de usuario

Con ambos sensores conectados en la pestaña «Sensores de luz», abre la ventana emergente para añadir un sensor (el botón «+» en un mosaico del gráfico en la vista de cuadrícula) y selecciona **Combinar luz ambiental + objeto**. Selecciona los dos sensores en los menús desplegables «Fuente de luz ambiental» y «Escáner de objetos» y haz clic en «Crear». El grupo aparece como un gráfico independiente y como una fila en la barra lateral con una insignia verde**REF**.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->

Debajo del gráfico de reflectancia (vista de lista), una **tabla de índices de vegetación** en tiempo real calcula los índices a partir de la curva utilizando los centros de banda en azul 450 / verde 550 / rojo 670 / NIR 800 nm. Índices basados en relaciones que anulan la escala absoluta (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) se muestran siempre; los índices que requieren reflectancia absoluta (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) solo aparecen cuando ambos sensores son modelos calibrados por potencia.

### Aparente frente a relativo: la regla de etiquetado

Chloros etiqueta la salida del sensor doble según lo que el par de sensores puede realmente indicar:

| Par de sensores | Etiqueta |
| --- | --- |
| Ambos sensores calibrados — paquete de fábrica cargado | **Reflectancia aparente** |
| Cualquiera de los sensores sin calibrar | **Reflectancia relativa** |

Los tres modelos son radiométricos: una vez cargado el paquete de calibración de fábrica de un sensor, sus espectros son absolutos en W/m²/nm, por lo que un par de sensores calibrados da lugar a una reflectancia aparente absoluta; el transporte no la determina. Un sensor que siga transmitiendo recuentos sin procesar (sin paquete accesible) rebaja el resultado a una curva relativa (la forma espectral sigue siendo válida). Ambos sensores deben llevar límites máximos correctamente declarados ([Perfiles de límites máximos y rango calibrado](caps-and-range.md)).

### De Python

No existe una llamada específica para dos sensores en la superficie agrupada SDK: abre dos sesiones con `chloros_sdk.connect_daq_sensor()` y calcula tú mismo la relación entre sus espectros `latest()`, aplicando la misma convención de etiquetado. (También existe una herramienta de grabación con doble sensor en la superficie interna de hardware directo de MAPIR, que aparece en la [Referencia de CLI](../reference/cli-reference.md) para completar la información; no forma parte del CLI suministrado; el flujo de trabajo de la interfaz gráfica de usuario anterior es la ruta activa compatible.)
