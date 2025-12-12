---
description: Paneles medidos en laboratorio utilizados para calibrar datos capturados en postprocesamiento
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Objetivos de Calibración

MAPIR ofrece varios objetivos de calibración para cubrir una gama de aplicaciones. El compacto T4-R50 que se muestra a continuación contiene 4 paneles que han sido medidos para la reflectancia de luz de 250 a 2,500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

Los objetivos de referencia difusa T4 tienen las siguientes curvas de reflectancia, [descarga de datos aquí](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Reflectancia :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Reflectancia :: 400-1000nm</p></figcaption></figure>

Al observar el gráfico de reflectancia, puede ver que los valores son longitud de onda (eje x) versus porcentaje de reflectancia (eje y). Cuando capturamos una imagen del objetivo de calibración, creamos una relación entre el valor de píxel y el porcentaje de reflectancia, dentro del espectro al que cada una de las bandas del sensor de la cámara es sensible.

Esto significa que con cada imagen que capture con nuestras cámaras, puede usar una foto de nuestros objetivos de reflectancia, como el [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) o [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) para calibrar las imágenes para reflectancia. Una vez calibrado, cada píxel en la imagen es igual al porcentaje de reflectancia.

Si exporta las imágenes calibradas en Chloros como el JPG o TIFF típico, entonces el porcentaje de reflectancia se calcula dividiendo el valor del píxel por la profundidad de bits del formato de imagen. Entonces, para JPG divida por 255, y para TIFF divida por 65,535. También puede elegir la salida de formato PERCENT en Chloros, y luego cada píxel tendrá un rango de valor porcentual de 0.0 a 1.0 (0% a 100% de reflectancia). Solo tenga en cuenta que algunas aplicaciones de imágenes no pueden aceptar las imágenes de porcentaje (punto flotante), y son grandes en tamaño de almacenamiento.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
