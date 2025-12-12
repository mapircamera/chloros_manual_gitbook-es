---
description: Preguntas Frecuentes
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Preguntas Frecuentes

<details>

<summary>¿Puedo procesar imágenes de cámaras que no son de marca MAPIR con Chloros?</summary>

No, Chloros solo admite el procesamiento de imágenes de cámaras MAPIR. Consulte la lista de [modelos de cámara compatibles](supported-cameras.md) para obtener más información. Ofrecemos procesamiento de otras cámaras en MAPIR Cloud, vea la lista completa [aquí](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>¿Puedo calibrar mis imágenes para reflectancia sin un objetivo de calibración?</summary>

No. Sin una imagen del objetivo de calibración capturada alrededor del momento en que se capturan las imágenes sin objetivo, no podrá relacionar los valores de píxeles de la imagen con un porcentaje de reflectancia conocido. Si tampoco incluye el registro de un sensor de luz MAPIR, entonces no se medirá el espectro de luz ambiental, y los resultados de reflectancia no serán precisos.

</details>

<details>

<summary>¿Puedo editar mis imágenes antes de procesarlas en Chloros?</summary>

No. Chloros asume que los datos de entrada no han sido modificados. No cambie los nombres de archivo.

</details>

<details>

<summary>¿Puedo configurar mis cámaras MAPIR Survey3 en exposición automática y procesar las imágenes en Chloros?</summary>

No. Los conjuntos de imágenes de Survey3 deben tener una exposición fija/bloqueada, por lo que no debe haber velocidad de obturación automática o ISO automático. Todas las imágenes del mismo modelo de cámara deben tener velocidad de obturación e ISO (exposición) idénticos.

</details>

<details>

<summary>¿Puede Chloros procesar o analizar imágenes de ortomosaicos?</summary>

No. Solo se admiten imágenes individuales de cámaras MAPIR, no imágenes unidas como un mapa de ortomosaico.

</details>

<details>

<summary>¿Cómo puedo acelerar el paso de detección de objetivo de Chloros?</summary>

En la tabla del explorador de archivos, preseleccionar las imágenes objetivo en la columna derecha le indicará a Chloros que solo busque en esas imágenes los objetivos de calibración, acelerando enormemente el procesamiento.

</details>

<details>

<summary>Si voy a subir mis imágenes a <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud</a>, ¿debo procesarlas en Chloros antes de subirlas?</summary>

Si planea subir a nuestra plataforma de procesamiento en línea [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), no edite las imágenes antes de subirlas. Cloud realizará todo el mismo procesamiento y más.

</details>

<details>

<summary>¿Alguna vez MAPIR admitirá la función X? Realmente desearía que MAPIR ofreciera X.</summary>

Siempre estamos interesados en recibir comentarios sobre nuestros productos. Si encuentra un problema con nuestros productos, o tiene una sugerencia sobre cómo podemos mejorar nuestros productos, [CONTÁCTENOS](https://www.mapir.camera/community/contact) para compartir sus pensamientos. La mayor parte de nuestra I+D se guía escuchando las mayores necesidades de nuestros clientes.

</details>
