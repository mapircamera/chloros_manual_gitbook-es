---
description: Laboratory-measured panels used to calibrate captured data in post-processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---
# Objetivos de calibración

MAPIR ofrece varios objetivos de calibración para cubrir una amplia gama de aplicaciones. El compacto T4-R50 que se muestra a continuación contiene 4 paneles que se han medido para determinar su reflectancia de la luz entre 250 y 2500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

Los objetivos de referencia difusa T4 tienen las siguientes curvas de reflectancia, [descarga de datos aquí](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Reflectancia :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Reflectancia :: 400-1000 nm</p></figcaption></figure>

Si observamos el gráfico de reflectancia, podemos ver que los valores son la longitud de onda (eje x) frente al porcentaje de reflectancia (eje y). Cuando capturamos una imagen del objetivo de calibración, creamos una relación entre el valor de los píxeles y el porcentaje de reflectancia, dentro del espectro al que son sensibles las bandas del sensor de la cámara.

Esto significa que con cada imagen que capture con nuestras cámaras, puede utilizar una foto de nuestros objetivos de reflectancia, como el [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) o el [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) para calibrar las imágenes en cuanto a reflectancia. Una vez calibrada, cada píxel de la imagen es igual al porcentaje de reflectancia.

Si se generan las imágenes calibradas en Chloros como JPG típico o TIFF, el porcentaje de reflectancia se calcula dividiendo el valor del píxel por la profundidad de bits del formato de imagen. Por lo tanto, para JPG se divide por 255 y para TIFF se divide por 65 535. También puede elegir el formato PERCENT en Chloros, y entonces cada píxel tendrá un valor porcentual entre 0,0 y 1,0 (0 % a 100 % de reflectancia). Solo tenga en cuenta que algunas aplicaciones de imágenes no aceptan imágenes porcentuales (de punto flotante) y que ocupan mucho espacio de almacenamiento.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
