---
description: Preguntas frecuentes
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---
# Preguntas frecuentes

<details>

<summary>¿Puedo procesar imágenes de cámaras que no sean de la marca MAPIR con Chloros?</summary>

No, Chloros solo admite el procesamiento de imágenes de cámaras MAPIR. Consulte la lista de [modelos de cámaras compatibles](supported-cameras.md) para obtener más información. Ofrecemos el procesamiento de otras cámaras en MAPIR Cloud, consulte la lista completa [aquí](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>¿Puedo calibrar mis imágenes para la reflectancia sin un objetivo de calibración?</summary>

No. Sin una imagen del objetivo de calibración capturada aproximadamente al mismo tiempo que las imágenes sin objetivo, no podrá relacionar los valores de píxeles de la imagen con un porcentaje de reflectancia conocido. Si tampoco incluye el registro de un sensor de luz MAPIR, no se medirá el espectro de luz ambiental y los resultados de reflectancia no serán precisos.

</details>

<details>

<summary>¿Puedo editar mis imágenes antes de procesarlas en Chloros?</summary>

No. Chloros asume que los datos de entrada no han sido modificados. No cambie los nombres de los archivos.

</details>

<details>

<summary>¿Puedo configurar mis cámaras MAPIR Survey3 en exposición automática y procesar las imágenes en Chloros?</summary>

No. Los conjuntos de datos de imágenes Survey3 deben tener una exposición fija/bloqueada, por lo que no se permite la velocidad de obturación automática ni el ISO automático. Todas las imágenes del mismo modelo de cámara deben tener una velocidad de obturación y un ISO (exposición) idénticos.

</details>

<details>

<summary>¿Puede Chloros procesar o analizar imágenes ortomosaicas?</summary>

No. Solo se admiten imágenes individuales de la cámara MAPIR, no imágenes unidas como un mapa ortomosaico.

</details>

<details>

<summary>¿Cómo puedo acelerar el paso de detección de objetivos de Chloros?</summary>

En la tabla del explorador de archivos, si preselecciona las imágenes objetivo en la columna derecha, Chloros solo buscará objetivos de calibración en esas imágenes, lo que acelerará considerablemente el procesamiento.

</details>

<details>

<summary>Si voy a cargar mis imágenes en <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> ¿debo procesarlas en Chloros antes de cargarlas?</summary>

Si tiene previsto subirlas a nuestra plataforma de procesamiento en línea [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), no edite las imágenes antes de subirlas. Cloud realizará el mismo procesamiento y mucho más.

</details>

<details>

<summary>¿MAPIR admitirá alguna vez la función X? Me gustaría mucho que MAPIR ofreciera X.</summary>

Siempre nos interesa recibir comentarios sobre nuestros productos. Si encuentra algún problema con nuestros productos o tiene alguna sugerencia sobre cómo podemos mejorarlos, póngase en contacto con nosotros (https://www.mapir.camera/community/contact) para compartir sus opiniones. La mayor parte de nuestra I+D se guía por las necesidades más importantes de nuestros clientes.

</details>
