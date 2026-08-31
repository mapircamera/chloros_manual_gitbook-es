# Área de pruebas de índices/LUT

El área de pruebas de índices/LUT es el espacio de trabajo interactivo situado en la barra lateral del visor de imágenes Chloros. Eliges una fórmula, le asignas los canales de tu cámara, le aplicas un degradado de color y ajustas el rango de valores; la imagen se actualiza en tiempo real mientras lo haces. Desde la versión 1.2.0 también puedes **guardar lo que has creado**, ya sea para una sola imagen o para todo el proyecto, sin necesidad de volver a procesarla.

## Para qué sirve el «Sandbox»

| «Index/LUT Sandbox» (interactivo)        | Procesamiento de proyectos (por lotes)       |
| -------------------------------------- | -------------------------------- |
| Una imagen cada vez, respuesta instantánea  | Todo el conjunto de datos de una sola vez     |
| Experimental e iterativo             | Ajustes preconfigurados          |
| Se renderiza en tiempo real; solo se guarda cuando lo solicitas  | Siempre escribe archivos de producto      |
| Perfecto para encontrar los ajustes adecuados | Ideal una vez que los ajustes son definitivos |

{% hint style="success" %}
**El flujo de trabajo habitual**: ajusta la configuración en el Sandbox hasta que la visualización se vea como deseas; a continuación, exporta directamente desde el Sandbox o copia los mismos ajustes de índice y LUT en [Configuración del proyecto](../project-settings/project-settings.md) para que la siguiente ejecución del procesamiento las incorpore a todas las imágenes.
{% endhint %}

***

## Abrir el Sandbox

1. Haz clic en una imagen de la cuadrícula: se abrirá a pantalla completa en la pestaña **Visor de imágenes** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Haz clic en el icono **Visor de imágenes**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> para desplegar la barra lateral izquierda si aún no está abierta
3. Elige una capa multibanda en el menú desplegable de capas situado en la parte superior derecha:**RAW (Reflectancia)** suele ser la opción habitual, ya que los valores de índice calculados a partir de la reflectancia calibrada son comparables entre imágenes

La barra lateral muestra, de arriba abajo:

* el nombre de la imagen y el modelo de su cámara
* el botón **Exportar/Guardar imagen(es)** —aparece una vez que se marca la casilla Índice o LUT—
* las casillas de selección **Índice**y**LUT**
* el panel de configuración del índice
* el panel **Valores del cursor** con la lectura, el histograma y el control de GSD

{% hint style="warning" %}
**No disponible para cámaras monocromáticas.** En una imagen LATTICE M3M de una sola banda, ambas casillas de selección están desactivadas y aparece la información sobre herramientas _«No disponible para sensores monocromáticos (M3M)»_; un índice multibanda no está definido en una sola banda. Para calcular índices a partir de cámaras M3M, combina dos o más en una pila multibanda alineada y utiliza el motor de índices de LATTICE.
{% endhint %}

***

## Aplicar un índice

1. Marca la casilla **Índice** en la parte superior de la barra lateral
2. Selecciona el filtro de tu cámara en el menú desplegable de la izquierda (`RGN`, `OCN`, `NGB`, `RGB`, `RE`, `NIR`)
3. Elige una fórmula de índice en el menú desplegable de la derecha: 27 fórmulas integradas, además de cualquier fórmula personalizada que hayas guardado
4. La fórmula se muestra a continuación en forma de expresión matemática, con un círculo vacío en cada ranura de banda. **Arrastra un círculo de canal de color a una ranura** para vincularlo
5. Una vez vinculadas todas las ranuras que utiliza la fórmula, la imagen se actualiza y muestra los valores del índice
6. Pasa el cursor por encima de la imagen para leer los valores; el panel **Valores del cursor** añade una fila de índice con el valor situado debajo del cursor

Haz doble clic en una casilla vinculada para borrarla. Una fórmula incompleta es un estado normal durante el arrastre, no un error: la imagen simplemente no se actualiza hasta que la fórmula esté completa.

Los círculos de los canales están codificados por colores: rojo = Red, verde = Green, azul = Blue, naranja = Orange, cian = Cyan, morado = NIR, magenta = RE. Se utilizan los mismos colores para los puntos de los canales y las curvas del histograma en el panel «Valores del cursor».

### Ejemplo de NDVI

```

Formula: (NIR - Red) / (NIR + Red)

For a Survey3W RGN camera:
  NIR = 850 nm band
  Red = 661 nm band

Result range:          -1.0 to +1.0
Typical vegetation:     0.4 to 0.9
Stressed vegetation:    0.2 to 0.4
Bare soil:              0.0 to 0.2
Water:                 -0.1 to 0.1
```

Para consultar la referencia completa de fórmulas —las tres listas de preajustes y qué nombres funcionan en cada caso—, véase [Fórmulas de índices multiespectrales](../project-settings/multispectral-index-formulas.md).

### Con la opción «Índice» marcada pero sin LUT

La imagen se representa en **escala de grises**, estirada entre los dos valores umbral. Esto es intencionado: la imagen de índice es data escalar, y la escala de grises es su representación fiel. Añade una LUT cuando desees color.***

## Trabajar con LUT (tablas de consulta)

Una **tabla de consulta** asigna valores de índice a colores: si se introduce NDVI 0,65, se obtiene un verde concreto. No modifica los datos, sino la forma en que se interpretan.

### Añadir una LUT

1. Haz clic en el botón **«+ Añadir LUT»** de <img src="../.gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> situado debajo de la fórmula
2. Elige un degradado de color
3. Establece el mínimo y el máximo de recorte
4. Elige un modo de recorte
5. Marca la casilla **LUT** en la barra lateral para aplicarla

La casilla de LUT permanece desactivada hasta que se haya configurado realmente una LUT en el índice.

### Seleccionar un degradado de color

Pasa el cursor por encima de la **barra de degradado**para abrir la lista de ajustes preestablecidos: Chloros incluye**siete** ajustes preestablecidos de degradado:

| # | Degradado                            | Forma                                                               |
| - | ----------------------------------- | ------------------------------------------------------------------- |
| 1 | Red → Amarillo → Green (**predeterminado**)  | Divergente: se ajusta a la percepción habitual de la vegetación, verde = sano |
| 2 | Morado → Amarillo → Green             | Divergente, con un extremo inferior bien definido                                  |
| 3 | Marrón → Blanco → Blue                | Divergente en torno a un punto medio claro                                   |
| 4 | Negro → Morado → Rosa → Amarillo pálido | Secuencial, de oscuro a claro                                           |
| 5 | Red → Amarillo → Blue                 | Divergente en torno a un punto medio claro                                   |
| 6 | Morado → Blue → Green → Amarillo      | Secuencial, de oscuro a claro                                           |
| 7 | Orange → Blanco → Morado             | Divergente en torno a un punto medio claro                                   |

Un degradado **divergente**coloca un color neutro en el centro de la ventana, lo que resulta adecuado cuando el punto medio tiene un significado concreto (un umbral, una fecha de referencia). Un degradado**secuencial** va de oscuro a claro de forma monótona, lo que resulta adecuado para una cantidad que solo tiene «más» y «menos».

Cada preajuste tiene siete paradas de color. Haz clic en un preajuste y la imagen se actualizará inmediatamente (cuando la casilla LUT esté marcada).

### Edición de los puntos de color

Debajo de la barra de degradado hay una fila de muestras de color, una por cada punto:

* **Cambiar un color**: haz clic en una muestra para abrir el selector de color (rueda de colores, controles deslizantes RGB/HSV o un código hexadecimal como `#FF0000`)
* **Añadir un punto**: haz clic en el botón**+** al final de la fila; se añadirá un punto blanco
* **Eliminar un punto**:**haz doble clic** en la muestra
* **Guardar un degradado editado**: haz clic en el icono de guardar junto a la barra de degradado para añadir tu degradado editado a la lista de preajustes y poder volver a seleccionarlo

El degradado que hayas configurado en un índice se almacena junto con dicho índice en la configuración del proyecto, por lo que se conserva al cerrar y volver a abrir el proyecto.

**Un número menor de paradas**produce zonas diferenciadas que se interpretan como una clasificación;**un número mayor de paradas** produce transiciones suaves, casi fotográficas. De tres a cinco paradas son adecuadas para diapositivas de presentación y mapas de clasificación; de seis a diez, para análisis generales; quince o más, para inspecciones detalladas y figuras de publicación.

### Configuración del rango de valores

El control de umbral es un **control deslizante de doble manejador**que va de −1 a +1, con un cuadro de texto editable en cada extremo para introducir valores exactos y un botón**AUTO**.

* Arrastra cualquiera de los controles deslizantes o escribe un número en su cuadro y pulsa Intro
* **AUTO**establece el rango entre el**percentil 2 y el percentil 98** de los valores de índice válidos de la imagen, lo que constituye un buen punto de partida que ignora los valores atípicos. Chloros redondea el resultado de forma adaptativa: a 4 decimales para un rango muy estrecho, a 3 para uno estrecho y a 2 en los demás casos
* Cualquier ajuste manual tiene prioridad sobre AUTO hasta que se vuelva a pulsar AUTO

Ejemplo de ventanas de NDVI:

| Objetivo                                    | Mín.  | Máx. |
| --------------------------------------- | ---- | --- |
| Mostrar todo                            | −1,0 | 1,0 |
| Solo vegetación, excluir suelo y agua | 0,2  | 0,9 |
| Solo vegetación sana                    | 0,5  | 0,9 |
| Resaltar el estrés                        | 0,2  | 0,5 |

Al reducir la ventana se aumenta el contraste dentro de la zona de interés y se deja todo lo demás fuera del rango, donde el **modo de recorte** decide qué ocurre con ello.***

## Modos de recorte

Cuando el valor del índice de un píxel queda fuera de la ventana mín./máx., el modo de recorte decide cómo se dibuja.

| Etiqueta del menú desplegable                  | Valor almacenado      | Los píxeles fuera de rango se dibujan como                                                                                                |
| ------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Mínimo y máximo** (predeterminado) | `clip`            | El color más cercano de cada extremo del degradado: los valores por debajo del mínimo adoptan el primer color, y los valores por encima del máximo, el último |
| **Fondo transparente**      | `transparent`     | Totalmente transparente (alfa real)                                                                                                  |
| **Fondo indexado**| `indexColor`      | Escala de grises, extendida a lo largo de**todo** el rango de índices de la imagen, de modo que la estructura fuera de rango sigue siendo visible en gris                |
| **Fondo original**         | `backgroundColor` | La propia imagen subyacente, de modo que la superposición de color se sitúa sobre la escena real                                                |

| Modo                       | Ideal para                               | Aspecto                                      |
| -------------------------- | -------------------------------------- | ----------------------------------------- |
| **Mínimo y máximo**      | Visualización completa de datos, análisis científico | Cada píxel coloreado                      |
| **Fondo transparente** | Superposiciones SIG, aislamiento de un intervalo de valores   | Color dentro de la ventana, nada fuera |
| **Fondo indexado**       | Énfasis manteniendo el contexto de los datos    | Color en el interior, gris en el exterior               |
| **Fondo original**    | Informes y presentaciones              | Color en el interior, fotografía en el exterior         |

{% hint style="info" %}
**Los píxeles sin datos son siempre transparentes, en todos los modos.** Un píxel cuyo índice no sea finito (una división por 0) o sea exactamente −1,0 o +1,0 (sentínelas de saturación, cuando una banda da cero mientras que la otra no) se trata como «sin datos» en lugar de como un valor extremo. Esto evita que las luces sobreexpuestas y las sombras perdidas se incluyan en la escala de color, en lugar de representarlas como el valor más extremo del fotograma. La misma regla define qué píxeles alimentan los umbrales AUTO y el histograma de índices, de modo que los tres coincidan.
{% endhint %}

La transparencia se conserva cuando la exportación se guarda como PNG. No se puede representar en JPG.

***

## Lectura de valores mientras se ajusta

El panel **Valores del cursor**, situado debajo del panel de configuración, es el instrumento de medición de Sandbox:

* Mueve el cursor sobre la imagen y lee los valores de origen por canal, además del valor del índice en su propia fila
* Activa el botón **ÍNDICE** situado encima del histograma para ver la distribución de los valores de índice en el fotograma, con tus dos umbrales de recorte representados como líneas discontinuas naranjas y el valor del cursor como una línea blanca; esta es la forma más rápida de elegir una ventana que realmente contenga tus datos
* Activa **CURSOR** para ver líneas marcadoras en los valores situados bajo el puntero
* Amplía más allá de 60× (menos si se ha establecido un tamaño de bloque GSD) para resaltar los píxeles individuales mostrados con un valor flotante

Una rutina práctica:

1. Toma nota de los valores sobre la vegetación sana, la vegetación estresada, el suelo desnudo y el agua
2. Observa dónde se sitúan esos grupos en el histograma de índices
3. Establece los valores mínimo y máximo para enmarcar el grupo que te interesa
4. Elige un modo de recorte: _Original Background_ mantiene visible la escena a su alrededor

***

## Exportar desde el Sandbox

Todo lo anterior es una vista previa en tiempo real hasta que lo guardes. El botón **Exportar/Guardar imagen(es)** situado en la parte superior de la barra lateral abre un panel que se desliza sobre la barra lateral (en lugar de cubrir la imagen, por lo que sigues pudiendo ver sobre qué estás tomando una decisión).

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>### Opciones

| Opción                          | Efecto                                                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Aplicar a la imagen actual**      | Guarda exactamente la imagen mostrada, con esta configuración                                                                                                |
| **Aplicar a todas las imágenes del proyecto** | Vuelve a aplicar la misma configuración a todas las imágenes del proyecto. Las imágenes que no tengan las bandas que necesita este índice se omiten, no se consideran errores |
| **Barra de gradiente de índice/LUT**      | También genera una imagen de leyenda independiente por cada exportación, con el rango de valores etiquetado                                                                     |
| **Histograma de índice**             | También genera una imagen de histograma independiente por cada exportación, que muestra los valores mínimo y máximo de los datos y los umbrales de recorte                                               |

Si el **tamaño de bloque GSD** de la pestaña de la imagen es superior a 1, el panel lo indica antes de que confirmes: la exportación guarda lo que estás viendo, incluido el promedio por bloques. Vuelve a establecer el control GSD en 1 primero si deseas la resolución completa.

### Dónde se guardan los archivos

Cada vez que se hace clic en **Exportar**, se asigna una**carpeta nueva que nunca se reutiliza**:

```
<project folder>/Sandbox_Exports/<IndexName>_<Index|LUT>_<NNN>/
```

Ejemplos: `Sandbox_Exports/NDVI_LUT_001/`, y luego `Sandbox_Exports/NDVI_LUT_002/` para la siguiente ejecución. La numeración se genera analizando lo que ya hay en el disco, por lo que se mantiene tras los reinicios y aunque elimines carpetas manualmente. Nunca se sobrescribe nada: el objetivo de Sandbox es comparar un intento con el anterior.

Dentro de la carpeta, por imagen:

| Archivo                                                   | Contenido                                                   |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| `<source name>_<IndexName>_<Index\|LUT>.png`           | La imagen renderizada, píxel a píxel tal y como la mostraba el visor |
| `<source name>_<IndexName>_<Index\|LUT>_legend.png`    | El archivo complementario de la barra de degradado, si se solicita                     |
| `<source name>_<IndexName>_<Index\|LUT>_histogram.png` | El archivo complementario del histograma de índices, si se solicita                  |

Los dos archivos complementarios siempre se guardan a **resolución completa**, incluso cuando la imagen principal se promedia por bloques: el tamaño de bloque es la resolución de pantalla, y ambos archivos complementarios contienen los valores reales del índice por píxel. Además, muestran más información que las versiones en pantalla: ambos indican la ventana de estiramiento _y_ los valores mínimos y máximos reales de los datos, por lo que una leyenda guardada sigue siendo legible meses después sin necesidad de tener el proyecto abierto.

### Progreso y resultados

La exportación de todo el proyecto tarda unos minutos, por lo que la ejecución informa a través de un canal de progreso en tiempo real en lugar de bloquearse:

* Una barra de progreso muestra «`current / total`» y el archivo que se está escribiendo
* Cuando finaliza, el panel indica cuántas imágenes se han exportado, cuántas se han omitido y la ruta de la carpeta de salida
* Las imágenes omitidas se enumeran con el motivo (se muestran hasta cinco, seguidas de una línea que indica «+N más»). El motivo habitual es que una capa no tiene los canales que necesita este índice
* Si **ninguna** imagen del proyecto puede utilizar el índice, la ejecución informa de un error en lugar de dejarte una carpeta vacía

Solo se ejecuta una exportación de sandbox a la vez. Si se intenta iniciar una segunda mientras otra está en curso, se rechaza con un mensaje claro, en lugar de permitir que dos ejecuciones compitan por el mismo archivo de proyecto.

### La cuadrícula recoge la ejecución

Cada ejecución completada aparece como un botón independiente en la [cuadrícula de imágenes](image-grid.md) de la barra de herramientas, etiquetado como `<IndexName> <Index|LUT> <NNN>`. Así es como se comparan las ejecuciones: exporta dos veces con diferentes gradientes o umbrales y, a continuación, alterna entre los dos botones de la cuadrícula.

***

## Fórmulas de índice personalizadas (Chloros+)

{% hint style="info" %}
**Dónde crearlas**: en la barra lateral de Sandbox o en**Configuración del proyecto** antes del procesamiento. Ambas se guardan en la misma lista a nivel de proyecto.
{% endhint %}

1. Abre la calculadora de fórmulas personalizadas desde el menú desplegable de fórmulas de índice (es necesario iniciar sesión con una suscripción válida a Chloros+)
2. Escribe la fórmula utilizando los **símbolos de franja de banda** `x`, `y`, `z`, `a`, `b`, `c` — no son nombres de bandas
3. Operadores disponibles: `+`, `-`, `*`, `/`, `^` y `()` para agrupar
4. Funciones disponibles: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
5. Asigna un nombre y guárdala: aparecerá en la parte inferior del menú desplegable de fórmulas y podrás asignar sus ranuras arrastrando los círculos de los canales, exactamente igual que con un preajuste integrado

```

Modified NDVI with an offset:   (y-x)/(y+x+0.5)
Simple ratio:                   y/x
Three-band difference:          (y-x)/(y+x-z)
Squared ratio:                  (y/x)^2
```

{% hint style="warning" %}
**Las fórmulas personalizadas solo están disponibles en la interfaz gráfica de usuario.** La opción CLI/SDK `--indices` amplía los 22 nombres de preajustes integrados y omite silenciosamente cualquier otra cosa, incluidas tus fórmulas personalizadas. Para procesar por lotes una fórmula personalizada, configúrala en «Ajustes del proyecto» y ejecuta el procesamiento, o utiliza la exportación «Aplicar a todas las imágenes del proyecto» del Sandbox.
{% endhint %}

***

## Solución de problemas

### «Esta capa no tiene los canales que necesita este índice»

La fórmula lee una posición de canal que la capa actual no tiene; por ejemplo, un índice de tres ranuras en un archivo de uno o dos canales. Cambia a una capa multibanda (reflectancia o sin bayering) o elige un índice que se adapte al filtro de tu cámara.

### «No se ha podido acceder al backend de procesamiento de imágenes»

El backend no responde. Comprueba la pestaña «Registro»; si el backend se está reiniciando, Sandbox se recuperará por sí solo una vez que vuelva a estar operativo.

### La imagen no ha cambiado al arrastrar un círculo

La fórmula aún no está completa. Una fórmula incompleta se trata como un estado normal de arrastre: no se renderiza nada y no se notifica ningún error. Rellena todos los campos que utiliza la fórmula.

### Toda la imagen es de un solo color

Probablemente tu ventana de recorte se encuentre muy fuera de los datos. Pulsa **AUTO**para ajustarla a los percentiles 2 y 98, o activa el histograma**ÍNDICE** para ver dónde se encuentran realmente los datos.

### Los colores exportados no coinciden con lo que veía

Deberían coincidir: la ruta de exportación es un reflejo deliberado de la vista previa en directo, incluido el alfa del modo de recorte, y el promedio por bloques se aplica _después_ de la coloración, exactamente igual que lo hace el visor. Si difieren, comprueba que el tamaño del bloque GSD no haya cambiado entre la visualización y la exportación.

***

## Próximos pasos

* [**Capas de imagen**](image-layers.md): en qué capa se debe aplicar un índice y qué significan sus valores
* [**Abrir una imagen a pantalla completa**](opening-an-image-full-screen.md): la lectura del cursor, el histograma y el control de GSD en detalle
* [**Fórmulas de índices multiespectrales**](../project-settings/multispectral-index-formulas.md): todos los ajustes preestablecidos, en todas las superficies
* [**Configuración del proyecto**](../project-settings/project-settings.md): cómo aplicar la configuración que has establecido a una sesión de procesamiento
