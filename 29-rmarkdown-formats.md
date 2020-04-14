# Formatos de R Markdown

## Introducción

Hasta ahora has visto usar R Markdown para producir documentos HTML. Este capítulo muestra una breve descripción de algunos de los muchos otros tipos de documentos que puedes generar con R Markdown. Hay dos maneras de definir la salida de un documento:

1.  De forma permanente, modificando el encabezado YAML: 
    
    ```yaml
    title: "Demo Viridis"
    output: html_document
    ```
    
2.  De forma transitoria, llamando `rmarkdown::render()` directamente:
    
    
    ```r
    rmarkdown::render("diamond-sizes.Rmd", output_format = "word_document")
    ```
    
    Esto es útil si quieres producir múltiples tipos de salida programáticamente.

El botón knit de RStudio genera un archivo con el primer tipo de formato listado en su campo de salida `output`. Puedes generar archivos en formatos adicionales haciendo clic en el menú de selección al lado del botón knit.


<img src="screenshots/rmarkdown-knit.png" width="206" style="display: block; margin: auto;" />

## Opciones de salida

Cada formato de salida está asociado con una función de R. Puedes escribir `foo` o `pkg::foo`. Si omites `pkg`, por defecto se asume que es rmarkdown. Es importante conocer el nombre de la función que genera el documento de salida, porque así es como obtienes ayuda. Por ejemplo, para saber qué parámetros puedes definir con `html_document`, busca en `?rmarkdown::html_document`.

Para sobrescribir los parámetros predeterminados necesitas usar un campo de `output` extendido. Por ejemplo, si quisieras generar un `html_document` con una tabla de contenido flotante, usarías:

```yaml
output:
  html_document:
    toc: true
    toc_float: true
```

Incluso puedes generar múltiples salidas suministrando una lista de formatos:

```yaml
output:
  html_document:
    toc: true
    toc_float: true
  pdf_document: default
```

Nota la sintaxis especial si no quieres modificar ninguna de las opciones por defecto, en inglés default.

## Documentos

El capítulo anterior se enfocó en la salida predeterminada del documento html `html_document`. Sin embargo hay un número de variaciones básicas para generar diferentes tipos de documentos:

*   `pdf_document` crea un PDF con LaTeX (un sistema de código abierto 
    de composición de textos), que necesitarás instalar. RStudio te notificará
    si no lo tienes.
  
*   `word_document` para documentos de Microsoft Word (`.docx`).
  
*   `odt_document` para documentos de texto OpenDocument (`.odt`).
  
*   `rtf_document` para documentos de Formato de Texto Enriquecido (`.rtf`).
  
*   `md_document` para documentos Markdown. Típicamente no es muy útil en sí mismo, 
    pero puedes usarlo si, por ejemplo, tu CMS corporativo o 
    tu wiki de laboratorio usa markdown.
       
*   `github_document`: esta es una versión de `md_document` específicamente
    diseñada para compartir en GitHub.

Recuerda que cuando generes un documento para compartirlo con los responsables de la toma de decisiones, puedes desactivar la visualización predeterminada de código, definiendo las opciones globales en el fragmento de configuración:


```r
knitr::opts_chunk$set(echo = FALSE)
```

Para los `html_document` otra opción es hacer que los fragmentos de código estén escondidos por defecto, pero visibles con un clic: 

```yaml
output:
  html_document:
    code_folding: hide
```

## Notebooks

Un notebook, `html_notebook` (cuaderno en español), es una variación de un `html_document`. Las salidas de los dos documentos son muy similares, pero tienen propósitos distintos. Un `html_document` está enfocado en la comunicación con los encargados de la toma de decisiones, mientras que un notebook está enfocado en colaborar con otros científicos de datos. Estos propósitos diferentes llevan a usar la salida HTML de diferentes maneras. Ambas salidas HTML contendrán la salida renderizada, pero el notebook también contendrá el código fuente completo. Esto significa que puedes usar el archivo `.nb.html` generado por el notebook de dos maneras:

1. Puedes verlo en un navegador web, y ver la salida generada. A diferencia del
   `html_document`, esta renderización siempre incluye una copia incrustada del
   código fuente que lo generó.   

2. Puedes editarlo en RStudio. Cuando abras un archivo `.nb.html`, RStudio 
   automáticamente recreará el archivo `.Rmd` que lo creó. En el futuro, también
   podrás incluir archivos de soporte (ej. archivos de datos `.csv`), que 
   serán extraídos automáticamente cuando sea necesario.

Enviar archivos `.nb.html` por correo electrónico es una manera simple de compartir los análisis con tus colegas. Pero las cosas se pondrán difíciles tan pronto como quieras hacer cambios. Si esto empieza a suceder, es un buen momento para aprender Git y GitHub. Aprender Git y Github definitivamente es doloroso al principio, pero la recompensa de la colaboración es enorme. Como se mencionó anteriormente, Git y GitHub están fuera del alcance de este libro, pero este es un consejo útil si ya los estás usando: usa las dos salidas, `html_notebook` y `github_document`: 

```yaml
output:
  html_notebook: default
  github_document: default
```

`html_notebook` te da una vista previa local, y un archivo que puedes compartir por correo electrónico. `github_document` crea un archivo md mínimo que puedes ingresar en git. Puedes revisar fácilmente como los resultados de tus análisis (no solo el código) cambian con el tiempo, y GitHub lo renderizará muy bien en línea.

## Presentaciones

También puedes usar R Markdown para crear presentaciones. Obtienes menos control visual que con herramientas como Keynote y PowerPoint, pero ahorrarás mucho tiempo insertando automáticamente los resultados de tu código R en una presentación. Las Presentaciones funcionan dividiendo tu contenido en diapositivas, con una nueva diapositiva que comienza en cada encabezado de primer nivel (`#`) o de segundo nivel (`##`). También puedes insertar una regla horizontal (`***`) para crear una nueva diapositiva sin encabezado. 

R Markdown viene con tres formatos de presentación integrados:

1.  `ioslides_presentation` - Presentación HTML con ioslides.

2.  `slidy_presentation` - Presentación HTML con W3C Slidy.

3.  `beamer_presentation` - Presentación PDF con LaTeX Beamer.

Otros dos formatos populares son proporcionados por paquetes:

1.  `revealjs::revealjs_presentation` - Presentación HTML con reveal.js. 
    Requiere el paquete __revealjs__.

2.  __rmdshower__, <https://github.com/MangoTheCat/rmdshower>, proporciona un 
    wrapper para el motor de presentaciones __shower__, <https://github.com/shower/shower> .

## Dashboards

Los dashboards (tableros de control en español) son una forma útil de comunicar grandes cantidades de información de forma visual y rápida. Flexdashboard hace que sea particularmente fácil crear dashboards usando R Markdown y proporciona una convención de como los encabezados afectan el diseño:

* Cada encabezado de Nivel 1 (`#`) comienza una nueva página en el dashboard.
* Cada encabezado de Nivel 2 (`##`) comienza una nueva columna.
* Cada encabezado de Nivel 3 (`###`) comienza una nueva fila.

Por ejemplo, puedes producir este dashboard:

<img src="screenshots/rmarkdown-flexdashboard.png" width="75%" style="display: block; margin: auto;" />

Usando este código:


````
---
title: "Dashboard de distribución de diamantes"
output: flexdashboard::flex_dashboard
---

```{r setup, include = FALSE}
library(datos)
library(ggplot2)
library(dplyr)
knitr::opts_chunk$set(fig.width = 5, fig.asp = 1 / 3)
```

## Columna 1

### Quilate

```{r}
ggplot(diamantes, aes(quilate)) + geom_histogram(binwidth = 0.1)
```

### Corte

```{r}
ggplot(diamantes, aes(corte)) + geom_bar()
```

### Color

```{r}
ggplot(diamantes, aes(color)) + geom_bar()
```

## Columna 2

### Diamantes más grandes

```{r}
diamantes %>%
  arrange(desc(quilate)) %>%
  head(100) %>%
  select(quilate, corte, color, precio) %>%
  DT::datatable()
```
````

Flexdashboard también proporciona herramientas simples para crear barras laterales, tabuladores, cuadros de valores y medidores. Para obtener más información (en inglés) acerca de Flexdashboard visita <http://rmarkdown.rstudio.com/flexdashboard/>.

## Interactividad

Cualquier formato HTML (documento, notebook, presentación, o dashboard) puede contener componentes interactivos.

### htmlwidgets

HTML es un formato interactivo, y puedes aprovechar esa interactividad con __htmlwidgets__, funciones de R que producen visualizaciones HTML interactivas. Por ejemplo, fíjate en el mapa de __leaflet__ a continuación. Si estás viendo esta página en la web, puedes arrastrar el mapa, acercar y alejar, etc. Obviamente no puedes hacer esto en un libro, por lo que RMarkdown automáticamente inserta una captura de pantalla estática.


```r
library(leaflet)
leaflet() %>%
  setView(174.764, -36.877, zoom = 16) %>% 
  addTiles() %>%
  addMarkers(174.764, -36.877, popup = "Maungawhau") 
```

<!--html_preserve--><div id="htmlwidget-ac96cb3ee4656e2e9ec3" style="width:70%;height:415.296px;" class="leaflet html-widget"></div>
<script type="application/json" data-for="htmlwidget-ac96cb3ee4656e2e9ec3">{"x":{"options":{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}}},"setView":[[-36.877,174.764],16,[]],"calls":[{"method":"addTiles","args":["//{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",null,null,{"minZoom":0,"maxZoom":18,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":1,"detectRetina":false,"attribution":"&copy; <a href=\"http://openstreetmap.org\">OpenStreetMap<\/a> contributors, <a href=\"http://creativecommons.org/licenses/by-sa/2.0/\">CC-BY-SA<\/a>"}]},{"method":"addMarkers","args":[-36.877,174.764,null,null,null,{"interactive":true,"draggable":false,"keyboard":true,"title":"","alt":"","zIndexOffset":0,"opacity":1,"riseOnHover":false,"riseOffset":250},"Maungawhau",null,null,null,null,{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null]}],"limits":{"lat":[-36.877,-36.877],"lng":[174.764,174.764]}},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

Lo bueno de los htmlwidgets es que no necesitas saber nada sobre HTML o JavaScript para usarlos. Todos los detalles están incluidos en el paquete, por lo que no tienes que preocuparte por eso.

Hay muchos paquetes que proporcionan htmlwidgets, incluyendo:

* __dygraphs__, <http://rstudio.github.io/dygraphs/>, para visualizaciones
interactivas de series de tiempo.

* __DT__, <http://rstudio.github.io/DT/>, para tablas interactivas.

* __threejs__, <https://github.com/bwlewis/rthreejs>, para plots 3D interactivos.

* __DiagrammeR__, <http://rich-iannone.github.io/DiagrammeR/>, para diagramas
  (como diagramas de flujo y diagramas simples de nodos).

Para obtener más información sobre los htmlwidgets y ver una lista más completa de los paquetes que los proporcionan, visita <http://www.htmlwidgets.org/>.

### Shiny

Los htmlwidgets proporcionan interactividad del lado del cliente, es decir que toda la interactividad ocurre en el navegador, independientemente de R. Por un lado, esto es bueno porque puedes distribuir el archivo HTML sin ninguna conexión con R. Sin embargo, también limita fundamentalmente lo que puedes hacer a las cosas que han sido implementadas en HTML y JavaScript. Una alternativa es usar __shiny__, un paquete que te permite crear interactividad usando código R, no JavaScript.

Para llamar código Shiny desde un documento R Markdown, agrega `runtime: shiny` al encabezado:

```yaml
title: "Applicación Web Shiny"
output: html_document
runtime: shiny
```

Luego puedes usar las funciones de "input" (entrada en español) para agregar componentes interactivos al documento:


```r
library(shiny)
textInput("nombre", "Cuál es tu nombre?")
numericInput("edad", "Cuántos años tienes?", NA, min = 0, max = 150)
```
<img src="screenshots/rmarkdown-shiny.png" width="357" style="display: block; margin: auto;" />

Después puedes referirte a los valores con `input$nombre` e `input$edad`, y el código que los usa se volverá a ejecutar automáticamente cada vez que los valores cambien.

No puedo mostrarte una aplicacion Shiny corriendo ahora porque las interacciones de Shiny ocurren en el lado del servidor. Esto quiere decir que puedes escribir aplicaciones interactivas sin saber JavaScript, pero necesitas un servidor para correrlas. Esto introduce un problema logístico: Las aplicaciones Shiny necesitan un servidor para correr en línea. Cuando corres una aplicación Shiny en tu propia computadora, Shiny automáticamente crea un servidor Shiny para que la puedas correr, pero necesitarás un servidor Shiny público si quieres proporcionar este tipo de interactividad en línea. Ésta es la limitación fundamental de Shiny: puedes crear en Shiny todo lo que puedes crear en R, pero necesitarás que R esté corriendo en algún lugar.

Aprende más sobre Shiny en: <http://shiny.rstudio.com/>.

## Sitios Web

Con un poco más de infraestructura puedes usar R Markdown para crear un sitio web completo:

*   Coloca todos tus archivos `.Rmd` en un mismo directorio. `index.Rmd` será tu 
    página de inicio.

*   Agrega un archivo YAML llamado `_site.yml` que proporcionará la navegación para el sitio.
    Por ejemplo:

    
    ```
    name: "mi-sitio"
    navbar:
      title: "Mi Sitio"
      left:
        - text: "Inicio"
          href: index.html
        - text: "Colores Viridis"
          href: 1-example.html
        - text: "Colores Terrain"
          href: 3-inline.html
    ```

Ejecuta `rmarkdown::render_site()` para crear `_site` (sitio en español), un directorio de archivos listo para ser implementado como un sitio web estático independiente, o si usas un proyecto de RStudio para el directorio de tu sitio web, RStudio agregará una pestaña de compilación que puedes usar para crear y obtener una vista previa de tu sitio.

Lee más acerca de los Sitios Web en: <http://rmarkdown.rstudio.com/rmarkdown_websites.html>.

## Otros Formatos

Otros paquetes proveen incluso más formatos de salida:

*   El paquete __bookdown__, <https://github.com/rstudio/bookdown>, 
    hace que crear libros, como este mismo, sea fácil. Para aprender más, lee
    "Creando libros con R Markdown" en inglés 
    [_Authoring Books with R Markdown_](https://bookdown.org/yihui/bookdown/),
    por Yihui Xie, el cual está, por supuesto, escrito en bookdown. Visita
    <http://www.bookdown.org> para ver otros libros bookdown escritos 
    por la comunidad R.

*   El paquete __prettydoc__, <https://github.com/yixuan/prettydoc/>, 
    proporciona formatos livianos de documentos con una gama de estéticas atractivas.

*   El paquete __rticles__, <https://github.com/rstudio/rticles>, compila una
    selección de formatos específicos para revistas científicas.

Consulta <http://rmarkdown.rstudio.com/formats.html> para ver una lista de más formatos. También puedes crear tu propio formato siguiendo las instrucciones en inglés de: <http://rmarkdown.rstudio.com/developer_custom_formats.html>.

## Aprende más

Para obtener más información sobre comunicación efectiva con estos diferentes formatos, recomiendo los siguientes recursos (en inglés):

* Para mejorar tus habilidades de presentación, recomiendo: Patrones de Presentación,
  [_Presentation Patterns_](https://amzn.com/0321820800), por Neal Ford,
  Matthew McCollough y Nathaniel Schutta. Proporciona un conjunto de patrones 
  efectivo (de alto y bajo nivel) que puedes usar para mejorar tus presentaciones.

* Si das charlas académicas, recomiendo que leas La guía para dar charlas del grupo Leek:
  [_Leek group guide to giving talks_](https://github.com/jtleek/talkguide).
  
* Yo no lo he tomado, pero he escuchado buenos comentarios sobre 
  el curso en línea Hablar en Público; Public Speaking de Matt McGarrity: 
  <https://www.coursera.org/learn/public-speaking>.

* Si estás creando muchos dashboards, asegúrate de leer: 
  Diseño de Dashboards de información: La comunicación de datos efectiva, de Stephen Few.
  [Information Dashboard Design: The Effective Visual Communication 
  of Data](https://amzn.com/0596100167). Te ayudará a crear dashboards realmente útiles
  y no solo bonitos a la vista.

* Comunicar tus ideas efectivamente a menudo se beneficia de un poco 
  de conocimiento en diseño gráfico. El libro de diseño para no diseñadores,
  [_The Non-Designer's Design Book_](http://amzn.com/0133966151) 
  es un buen lugar para empezar.
