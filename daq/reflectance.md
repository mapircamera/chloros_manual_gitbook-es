# Flujos de trabajo de reflectancia

Un sensor de luz DAQ convierte las imágenes radiométricas en reflectancia. Existen dos flujos de trabajo distintos:

1. **Sensor único**: un DAQ mide la irradiancia descendente mientras una cámara captura la imagen, y Chloros divide la radiancia de la cámara por esa referencia.
2. **Sensor doble**: dos sensores DAQ, uno orientado hacia el cielo y otro hacia un objeto, generan una curva de reflectancia espectral en tiempo real sin necesidad de utilizar una cámara.

## Sensor único + cámara (referencia descendente)

El DAQ actúa como sensor de luz descendente (DLS): la cámara mide la radiancia ascendente **L**(W/m²/sr/nm), el DAQ mide la irradiancia descendente**E**(W/m²/nm) y**Chloros** calcula la reflectancia por banda según la fórmula:

> ρ = π · L / E

La lectura del DAQ siempre **coincide en la marca de tiempo con la exposición**; por eso el DAQ y las cámaras comparten un reloj sincronizado mediante PTP (véase [Redes y sincronización horaria del DAQ-E](ethernet-ptp.md)). Ajusta la tapa coseno Sunshine para trabajos al aire libre y declárala correctamente; la declaración de la tapa escala directamente E (véase [Perfiles de tapa y rango calibrado](caps-and-range.md)). Para trabajos cuantitativos, recuerde la característica del instrumento: la irradiancia cuantitativa se obtiene a partir de una media de al menos 15 s de lecturas.

### Captura en directo

Vincula el DAQ a una cámara en la pestaña «Cámaras»: el panel de configuración de cada cámara cuenta con un menú desplegable **Sensor de luz** que muestra todos los DAQ conectados (DAQ-U/M/E) desde la pestaña «Sensores de luz»; en el caso de una matriz sincronizada, la selección de un sensor de luz para toda la matriz se propaga a todos los miembros (aunque las cámaras individuales pueden seguir anular esta configuración). Una vez vinculado, los espectros del sensor se envían a la ranura DLS de la cámara y las exportaciones de reflectancia se dividen por la lectura correspondiente. En una matriz, un miembro con su propio sensor utiliza su propia lectura —para sus bandas en la composición en directo, sus exportaciones y su exposición automática— y los demás miembros utilizan la de la matriz.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->


Dos comportamientos que conviene conocer:

* **Sin DAQ vinculado → la reflectancia se rechaza, no se simula.** Chloros rechaza el producto de reflectancia y registra el motivo del salto, en lugar de devolver silenciosamente un producto inferior.
* **La lectura utilizada se conserva.** Para cada fotograma de reflectancia, la lectura del DAQ realmente aplicada se guarda como un archivo `.daq` en la carpeta `daq/` de las capturas, de modo que la captura se pueda volver a procesar más adelante ([Grabación y el formato .daq](recording.md)). Un archivo `.daq.cameras.json` situado junto a él identifica las cámaras a las que se refiere, y el reprocesamiento lo asocia únicamente a las imágenes de dichas cámaras.

### Procesamiento de las imágenes grabadas

Para el procesamiento posterior al vuelo, graba un `.daq` durante la sesión y guárdalo junto con las imágenes: el proceso resuelve automáticamente la radiación descendente coincidente con la marca de tiempo, obteniendo cualquier calibración de fábrica que falte de la nube de MAPIR en el primer uso. Las grabaciones de la interfaz gráfica de usuario se añaden automáticamente al proyecto abierto cuando se detienen.

La referencia de reflectancia se puede seleccionar en el momento del procesamiento: `--reflectance-source` sobre `chloros-cli process`, o bien la configuración de la fuente de reflectancia en los «Ajustes del proyecto» de la interfaz gráfica de usuario:

| Valor | Comportamiento |
| --- | --- |
| `auto` (por defecto) | Un objetivo de calibración dentro del fotograma que haya superado el control de calidad (QA) es la referencia absoluta; la calibración descendente del DAQ (ρ = π·L/E) es la alternativa. Un objetivo que supera el control de calidad con un indicador de error (un panel descartado o una banda fuera de servicio) no se prefiere automáticamente: se prueban las cadenas «objetivo», «objetivo+DAQ» y «solo DAQ» en orden de su puntuación medida en los paneles no utilizados. |
| `daq` | DAQ con autoridad |
| `target` | Objetivo estricto dentro del marco; sin sustitución por DAQ |

Consulte [Objetivos de calibración](../calibration-targets.md) para conocer los flujos de trabajo de los objetivos y el [capítulo sobre LATTICE](../lattice/README.md), así como la [Referencia de CLI](../reference/cli-reference.md) para conocer el proceso completo de procesamiento. Al leer los píxeles de reflectancia exportados, utilice la escala indicada (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535); véase [Formatos de imagen de salida](../output-image-formats.md).

### Bandas fuera del rango calibrado del DAQ

El rango calibrado radiométricamente del DAQ es de ~374–974 nm. Chloros rechaza la reflectancia basada en el DAQ para cualquier banda de la cámara que tenga menos de la mitad de su peso espectral dentro de ese intervalo, indicando el motivo de omisión `dls-uncalibrated-band-<nm>`. Entre los modelos disponibles, esto solo afecta al F988: la reflectancia del F988 se calibra utilizando un panel de reflectancia en la escena; dado que la banda se encuentra fuera del rango calibrado del sensor de luz del DAQ, Chloros aplica la captura más reciente del panel y la mantiene entre las lecturas del panel. Si una cámara F988 funciona solo con DAQ, Chloros rechaza la reflectancia basada en DAQ para esa banda con el motivo de omisión `dls-uncalibrated-band-988`; el flujo de trabajo con el panel es la vía recomendada.

## Sensor dual (ambiental + objeto)

Dos sensores DAQ —cualquier par, en cualquier sistema de transporte— proporcionan un espectro de reflectancia en tiempo real sin necesidad de cámara: un sensor apunta al cielo (**Fuente de luz ambiental**), otro apunta al sujeto (**Escáner de objeto**), y Chloros calcula por longitud de onda:

> R(λ) = objeto(λ) / ambiental(λ)

(cero cuando ambiental ≤ 0).

### En la interfaz gráfica de usuario

Con ambos sensores conectados en la pestaña «Sensores de luz», abre la ventana emergente para añadir sensores (el botón «+» en un mosaico de gráfico en la vista de cuadrícula) y selecciona **Combinar luz ambiental + objeto**. Selecciona los dos sensores en los menús desplegables**Fuente de luz ambiental**y**Escáner de objetos**y haz clic en**Crear**. El grupo aparece como un gráfico independiente y como una fila de la barra lateral con una insignia verde**REF**.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->


Debajo del gráfico de reflectancia (vista de lista), una **tabla de índices de vegetación** en tiempo real calcula los índices a partir de la curva utilizando los centros de banda en azul 450 / verde 550 / rojo 670 / NIR 800 nm. Los índices basados en ratios que anulan la escala absoluta (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) se muestran siempre; los índices que requieren reflectancia absoluta (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) solo aparecen cuando ambos sensores son modelos calibrados en potencia.

### Aparente frente a relativa: la regla de etiquetado de

Chloros etiqueta la salida del sensor dual según lo que el par de sensores puede realmente afirmar:

| Par de sensores | Etiqueta |
| --- | --- |
| Ambos sensores calibrados — paquete de fábrica cargado | **Reflectancia aparente** |
| Cualquiera de los sensores sin calibrar | **Reflectancia relativa** |

Los tres modelos son radiométricos: una vez cargado el paquete de calibración de fábrica de un sensor, sus espectros son absolutos en W/m²/nm, por lo que un par de sensores calibrados da lugar a una reflectancia aparente absoluta —el transporte no la determina—. Un sensor que siga transmitiendo recuentos sin procesar (sin paquete accesible) degrada el resultado a una curva relativa (la forma espectral sigue siendo válida). Ambos sensores deben llevar límites máximos correctamente declarados ([Perfiles de límites máximos y rango calibrado](caps-and-range.md)).

### Desde Python


No existe una llamada específica para dos sensores en la interfaz del SDK agrupado: abre dos sesiones con `chloros_sdk.connect_daq_sensor()` y compara tú mismo sus espectros con `latest()`, aplicando la misma convención de etiquetado. (También existe una herramienta de registro para dos sensores en la interfaz interna de acceso directo al hardware de MAPIR, que figura en la [Referencia de la CLI](../reference/cli-reference.md) para completar la información; no forma parte de la CLI suministrada; el flujo de trabajo de la interfaz gráfica de usuario descrito anteriormente es la ruta activa compatible.)
