---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Objetivos de calibración:

MAPIR ofrece diversos objetivos de calibración para cubrir una amplia gama de aplicaciones. El modelo compacto T4-R50 que se muestra a continuación contiene 4 paneles cuya reflectancia de la luz se ha medido en el rango de 250 a 2.500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Los objetivos de referencia difusa T4 presentan las siguientes curvas de reflectancia, [descargar datos aquí](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>Reflectancia MAPIR T4 :: 250-2.500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>Reflectancia MAPIR T4 :: 400-1.000 nm</p></figcaption></figure>Los objetivos de referencia difusos T4P presentan las siguientes curvas de reflectancia; [descargar datos aquí](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>Reflectancia MAPIR T4P :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>Reflectancia MAPIR T4P :: 400-1000 nm</p></figcaption></figure>Al observar el gráfico de reflectancia, se puede ver que los valores representan la longitud de onda (eje x) frente al porcentaje de reflectancia (eje y). Cuando capturamos una imagen del objetivo de calibración, establecemos una relación entre el valor del píxel y el porcentaje de reflectancia, dentro del espectro al que es sensible cada una de las bandas del sensor de la cámara.

Esto significa que, con cada imagen que captures con nuestras cámaras, puedes utilizar una fotografía de nuestras dianas de reflectancia, como la [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) o [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), para calibrar las imágenes en cuanto a la reflectancia. Una vez calibradas, cada píxel de la imagen equivale a un porcentaje de reflectancia.

En el caso de los resultados de **Survey3**, si exportas las imágenes calibradas en Chloros como archivos JPG o TIFF habituales, el porcentaje de reflectancia se calcula dividiendo el valor del píxel por la profundidad de bits del formato de imagen. Así, para JPG hay que dividir por 255, y para TIFF, por 65 535. También puede elegir el formato de salida «PERCENT» en Chloros; de este modo, cada píxel oscilará entre un valor porcentual de 0,0 y 1,0 (del 0 % al 100 % de reflectancia). Solo debe tener en cuenta que algunas aplicaciones de imágenes no admiten imágenes en formato porcentual (de coma flotante) y que estas ocupan mucho espacio de almacenamiento.

{% hint style="info" %}
**La reflectancia LATTICE utiliza una escala de píxeles diferente.** La reflectancia LATTICE se almacena con DN 32768 = 100 % de reflectancia (no 65 535), y cada archivo lleva una etiqueta XMP `Chloros:PixelScale` que indica su escala. Lee la etiqueta y divide por ese valor en lugar de suponer una constante; consulta [Formatos de imagen de salida](output-image-formats.md).
{% endhint %}

## Objetivos de calibración con cámaras LATTICE

Con las cámaras LATTICE, un objetivo de calibración es **opcional** para la reflectancia: en su lugar, Chloros puede referenciar la reflectancia a la irradiancia descendente medida por un sensor de luz DAQ (ρ = π·L/E). La referencia se selecciona mediante el ajuste de «fuente de reflectancia» del proyecto, que se configura con `chloros-cli process --reflectance-source` (la interfaz gráfica de usuario no dispone de ningún control para ello y utiliza el valor almacenado del proyecto, `auto` por defecto — véase [Configuración del proyecto](project-settings/project-settings.md)):

| Valor | Comportamiento |
| --- | --- |
| `auto` *(por defecto)* | Un objetivo dentro del encuadre que haya superado el control de calidad (QA) es la **referencia absoluta**; cuando no hay ningún objetivo presente o el control de calidad falla, Chloros recurre a la división descendente del DAQ. Un objetivo que supere el control de calidad con un indicador (un panel descartado o una banda fuera de servicio) no se prefiere automáticamente: se prueban las cadenas «objetivo», «objetivo+DAQ» y «solo DAQ» en orden de su puntuación medida en los paneles no utilizados. |
| `target` | Solo objetivo estricto: sin sustitución por DAQ. |
| `daq` | DAQ como referencia: la medición descendente es siempre la referencia. |

Comportamiento adicional de los objetivos para LATTICE:

* **Geometrías de los objetivos**: se admiten paneles marcados con ArUco, paneles con ROI fijos y objetivos en forma de tira; la geometría proviene de la configuración de objetivos del proyecto.
* **Datos de objetivos medidos por unidad** — `--target-reflectance-dir DIR` apunta a un directorio de escaneos de reflectancia de objetivos medidos por unidad (`<serial>.csv`, consultados mediante el número de serie o el código QR de la unidad del objetivo). En caso de fallo, Chloros recurre a los espectros nominales T3/T4P.
* **Anclaje temporal**: un objetivo detectado calibra los fotogramas a su alrededor y se mantiene entre avistamientos del objetivo.

La semántica completa de los indicadores y los ejemplos se encuentran en la [Referencia de la CLI](reference/cli-reference.md) (véase «Opciones de exportación por producto»).

### F988

«La reflectancia de F988 se calibra utilizando un panel de reflectancia en la escena: la banda se encuentra fuera del rango calibrado del sensor de luz del DAQ, por lo que Chloros aplica tu captura de panel más reciente y la mantiene entre los avistamientos del panel».

Si F988 se ejecuta con una calibración basada únicamente en el DAQ, Chloros rechaza la reflectancia basada en el DAQ para esa banda y explica el motivo (motivo de omisión `dls-uncalibrated-band-988`); el flujo de trabajo con el panel es la vía recomendada.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
