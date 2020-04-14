# Importación de datos

## Introducción

Trabajar con datos incluidos en paquetes de R es una muy buena forma de empezar a conocer las herramientas de la ciencia de datos. Sin embargo, en algún punto deberás parar de aprender y comenzar a trabajar con tus propios datos. En este capítulo aprenderás cómo leer en R archivos rectangulares de texto plano. Si bien solo tocaremos superficialmente el tema de importación, muchos de los principios que veremos son aplicables al trabajo con otras formas de datos. Finalizaremos sugiriendo algunos paquetes que son útiles para otros formatos.


### Prerrequisitos

En este capítulo aprenderás cómo cargar archivos planos en R con __readr__, uno de los paquetes principales de **tidyverse**.


```r
library(tidyverse)
```

## Comenzando

La mayoría de las funciones de **readr** se enfocan en transformar archivos planos en *data frames*:

* `read_csv()` lee archivos delimitados por coma, `read_csv2()` lee archivos separados por punto y coma 
  (comunes en países donde ',' es utilizada para separar decimales),
  `read_tsv()` lee archivos delimitados por tabulaciones y `read_delim()` archivos con cualquier delimitador. 
  
* `read_fwf()` lee archivos de ancho fijo. Puedes especificar los campos ya sea por su ancho, con `fwf_widths()`, o por su ubicación, con `fwf_positions()`. `read_table()` lee una variación común de estos archivos de ancho fijo en los que las columnas se encuentran separadas por espacios.

* `read_log()` lee archivos de registro estilo Apache. (Revisa también [webreadr](https://github.com/Ironholds/webreadr), que está construido sobre `read_log()` y proporciona muchas otras herramientas útiles).

Todas estas funciones tienen una sintaxis similar, por lo que una vez que dominas una, puedes utilizar todas las demás con facilidad. En el resto del capítulo nos enfocaremos en `read_csv()`. Los archivos csv no solo son una de las formas de almacenamiento más comunes, sino que una vez que comprendas `read_csv()` podrás aplicar fácilmente tus conocimientos a todas las otras funciones de __readr__.

El primer argumento de `read_csv()` es el más importante: es la ruta al archivo a leer.


```r
alturas <- read_csv("data/alturas.csv")
#> Parsed with column specification:
#> cols(
#>   earn = col_double(),
#>   height = col_double(),
#>   sex = col_character(),
#>   ed = col_double(),
#>   age = col_double(),
#>   race = col_character()
#> )
```

Cuando ejecutas `read_csv()`, la función devuelve el nombre y tipo de datos con que se importó cada columna. Esta es una parte importante de **readr**, sobre la cual volveremos luego en [segmentar un archivo]. 

Puedes también definir un archivo CSV "en línea" (_inline_). Esto es útil para experimentar con **readr** y para crear ejemplos reproducibles para ser compartidos.


```r
read_csv("a,b,c
1,2,3
4,5,6")
#> # A tibble: 2 x 3
#>       a     b     c
#>   <dbl> <dbl> <dbl>
#> 1     1     2     3
#> 2     4     5     6
```

En ambos casos `read_csv()` emplea la primera línea de los datos para los nombres de columna, lo que es una convención muy común. Hay dos casos en los que podrías querer ajustar este comportamiento:

1. A veces hay unas pocas líneas de metadatos al comienzo del archivo. Puedes usar `skip = n` para omitir las           primeras `n` líneas, o bien, o usar `comment = "#"` para quitar todas las líneas que comienzan con, por ejemplo, `#`.


    
    ```r
    read_csv("La primera línea de metadata 
      La segunda línea de metadata
      x,y,z
      1,2,3", skip = 2)
    #> # A tibble: 1 x 3
    #>       x     y     z
    #>   <dbl> <dbl> <dbl>
    #> 1     1     2     3
    
    read_csv("# Un comentario que quiero ignorar
      x,y,z
      1,2,3", comment = "#")
    #> # A tibble: 1 x 3
    #>       x     y     z
    #>   <dbl> <dbl> <dbl>
    #> 1     1     2     3
    ```
    
2.  Los datos pueden no tener nombres de columna. En ese caso, puedes utilizar `col_names = FALSE` para decirle a `read_csv()` que no trate la primera fila como encabezados y que, en lugar de eso, los etiquete secuencialmente desde `X1` a `Xn`:

    
    ```r
    read_csv("1,2,3\n4,5,6", col_names = FALSE)
    #> # A tibble: 2 x 3
    #>      X1    X2    X3
    #>   <dbl> <dbl> <dbl>
    #> 1     1     2     3
    #> 2     4     5     6
    ```
    
    (`"\n"` es un atajo conveniente para agregar una línea nueva. Aprenderás más acerca de él y otros modos de evitar texto en la sección [Cadenas: elementos básicos]).
    
    Alternativamente puedes utilizar `col_names` con el vector de caracteres que será utilizado como nombres de columna:
    
    
    ```r
    read_csv("1,2,3\n4,5,6", col_names = c("x", "y", "z"))
    #> # A tibble: 2 x 3
    #>       x     y     z
    #>   <dbl> <dbl> <dbl>
    #> 1     1     2     3
    #> 2     4     5     6
    ```

Otra opción que comúnmente necesita ajustes es `na` (del inglés, *"not available"*: Esto especifica el valor (o valores) que se utilizan para representar los valores faltantes en tu archivo: 


```r
read_csv("a,b,c\n1,2,.", na = ".")
#> # A tibble: 1 x 3
#>       a     b c    
#>   <dbl> <dbl> <lgl>
#> 1     1     2 NA
```

Esto es todo lo que necesitas saber para leer el ~75% de los archivos csv con los que te encontrarás en la práctica. También puedes adaptar fácilmente lo que has aprendido para leer archivos separados por tabuladores con `read_tsv()` y archivos de ancho fijo con `read_fwf()`. Para leer archivos más desafiantes, necesitas aprender un poco más sobre cómo **readr** segmenta cada columna y las transforma en vectores de R. 

### Comparación con R base

Si has utilizado R anteriormente, tal vez te preguntas por qué no usamos `read.csv()`. Hay unas pocas buenas razones para preferir las funciones de **readr** sobre las equivalentes de R base:

* Generalmente son mucho más rápidas (~10x) que sus equivalentes. Los trabajos que tienen un tiempo de ejecución prolongado poseen una barra de progreso para que puedas ver qué está ocurriendo. Si solo te interesa la velocidad, prueba `data.table::fread()`. No se ajusta tan bien con el **tidyverse**, pero puede ser bastante más rápido.

* Producen tibbles, no convierten los vectores de caracteres a factores, no usan nombres de filas ni distorsionan los nombres de columnas. Estas son fuentes comunes de frustración al utilizar las funciones de R base.

* Son más reproducibles. Las funciones de R base heredan ciertos comportamientos de tu sistema operativo y de las variables del ambiente, de modo que importar código que funciona bien en tu computadora puede no funcionar en la de otros. 

### Ejercicios

1. ¿Qué función utilizarías para leer un archivo donde los campos están separados con "|"?

1. Además de `file`, `skip` y `comment`, ¿qué otros argumentos tienen en común `read_csv()` y `read_tsv()`?

1. ¿Cuáles son los argumentos más importantes de `read_fwf()`?

1. Algunas veces las cadenas de caracteres en un archivo csv contienen comas. Para evitar que causen problemas, deben estar rodeadas por comillas, como `"` o `'`. Por convención, `read_csv()` asume que el caracter de separación será `"`.¿Qué argumentos debes especificar para leer el siguiente texto en un _data frame_? 
  
    
    
    ```r
    "x,y\n1,'a,b'"
    ```

5. Identifica qué está mal en cada una de los siguientes archivos csv en línea (_inline_). ¿Qué pasa cuando corres el código?    

    
    ```r
    read_csv("a,b\n1,2,3\n4,5,6")
    read_csv("a,b,c\n1,2\n1,2,3,4")
    read_csv("a,b\n\"1")
    read_csv("a,b\n1,2\na,b")
    read_csv("a;b\n1;3")
    ```

## Segmentar un vector

Antes de entrar en detalles sobre cómo **readr** lee archivos del disco, necesitamos desviarnos un poco para hablar sobre las funciones `parse_*()` (del inglés _analizar_, _segmentar_). Estas funciones toman un vector de caracteres y devuelven un vector más especializado, como un vector lógico, numérico o una fecha:



```r
str(parse_logical(c("TRUE", "FALSE", "NA")))
#>  logi [1:3] TRUE FALSE NA
str(parse_integer(c("1", "2", "3")))
#>  int [1:3] 1 2 3
str(parse_date(c("2010-01-01", "1979-10-14")))
#>  Date[1:2], format: "2010-01-01" "1979-10-14"
```

Estas funciones son útiles por sí mismas, pero también son un bloque estructural importante para **readr**. Una vez que aprendas en esta sección cómo funcionan los segmentadores individuales, en la próxima volveremos atrás y veremos cómo se combinan entre ellos para analizar un archivo completo.

Como todas las funciones dentro del **tidyverse**, las funciones `parse_*()` son uniformes: el primer argumento es un vector de caracteres a analizar y el argumento `na` especifica qué cadenas deberían ser tratadas como faltantes: 



```r
parse_integer(c("1", "231", ".", "456"), na = ".")
#> [1]   1 231  NA 456
```

Si la segmentación falla, obtendrás una advertencia:


```r
x <- parse_integer(c("123", "345", "abc", "123.45"))
#> Warning: 2 parsing failures.
#> row col               expected actual
#>   3  -- an integer                abc
#>   4  -- no trailing characters    .45
```

Y las fallas aparecerán como faltantes en el output: 


```r
x
#> [1] 123 345  NA  NA
#> attr(,"problems")
#> # A tibble: 2 x 4
#>     row   col expected               actual
#>   <int> <int> <chr>                  <chr> 
#> 1     3    NA an integer             abc   
#> 2     4    NA no trailing characters .45
```

Si hay muchas fallas de segmentación, necesitarás utilizar `problems()` (del inglés _problemas_) para obtener la totalidad de ellas. Esto devuelve un tibble que puedes luego manipular con **dplyr**.



```r
problems(x)
#> # A tibble: 2 x 4
#>     row   col expected               actual
#>   <int> <int> <chr>                  <chr> 
#> 1     3    NA an integer             abc   
#> 2     4    NA no trailing characters .45
```

Utilizar segmentadores es más que nada una cuestión de entender qué está disponible y cómo enfrentar diferentes tipos de input. Hay ocho segmentadores particularmente importantes: 

1.  `parse_logical()` y `parse_integer()` analizan valores lógicos y números enteros respectivamente. No hay prácticamente nada que pueda salir mal con estos segmentadores, así que no los describiremos con detalle aquí.

1. `parse_double()` es un segmentador numérico estricto, y `parse_number()` es un segmentador numérico flexible. Son más complicados de lo que podrías esperar debido a que los números se escriben de diferentes formas en distintas partes del mundo.

1. `parse_character()` parece tan simple que no debiera ser necesario. Pero una complicación lo hace bastante importante: la codificación de caracteres (el _encoding_).

1.  `parse_factor()` crea factores, la estructura de datos que R usa para representar variables categóricas con valores fijos y conocidos.

1.  `parse_datetime()`, `parse_date()` y `parse_time()` te permiten analizar diversas especificaciones de fechas y horas. Estos son los más complicados, ya que hay muchas formas diferentes de escribir las fechas. 

Las secciones siguientes describen estos analizadores en mayor detalle.

### Números

Pareciera que analizar un número debiese ser algo sencillo, pero hay tres problemas que pueden complicar el proceso: 

1. Las personas escriben los números de forma distinta en diferentes partes del mundo. Por ejemplo, algunos países utilizan `.` entre el entero y la fracción de un número real, mientras que otros utilizan `,`.


2. A menudo los números están rodeados por otros caracteres que proporcionan algún contexto, como "$1000" o "10%".

3. Los números frecuentemente contienen caracteres de "agrupación" para hacerlos más fáciles de leer, como "1,000,000". Estos caracteres de agrupación varían alrededor del mundo. 

Para enfrentar al primer problema, **readr** tiene el concepto de "*locale*", un objeto que especifica las opciones de segmentación que difieren de un lugar a otro. Cuando segmentamos números, la opción más importante es el caracter que utilizas como símbolo decimal. Puedes sobreescribir el valor por defecto `.` creando un nuevo locale y estableciendo el argumento `decimal_mark` (del inglés _marca decimal_):




```r
parse_double("1.23")
#> [1] 1.23
parse_double("1,23", locale = locale(decimal_mark = ","))
#> [1] 1.23
```
El locale por defecto de **readr** es EEUU-céntrico, porque generalmente R es EEUU-céntrico (por ejemplo, la documentación de R base está escrita en inglés norteamericano). Una aproximación alternativa podría ser probar y adivinar las opciones por defecto de tu sistema operativo. Esto es difícil de hacer y, lo que es más importante, hace que tu código sea frágil. Incluso si funciona en tu computadora, puede fallar cuando lo envíes a un/a colega en otro país.

`parse_number()` responde al segundo problema: ignora los caracteres no-numéricos antes y después del número. Esto es particularmente útil para monedas y porcentajes, pero también sirve para extraer números insertos en texto.


```r
parse_number("$100")
#> [1] 100
parse_number("20%")
#> [1] 20
parse_number("It cost $123.45")
#> [1] 123
```

El problema final se puede enfrentar combinando `parse_number()` y el locale, ya que `parse_number()` ignorará el "símbolo decimal":


```r
# Utilizado en América
parse_number("$123,456,789")
#> [1] 1.23e+08
# Utilizado en muchas regiones de Europa
parse_number("123.456.789", locale = locale(grouping_mark = "."))
#> [1] 1.23e+08
# Utilizado en Suiza
parse_number("123'456'789", locale = locale(grouping_mark = "'"))
#> [1] 1.23e+08
```

### Cadenas de texto (_strings_) {#readr-strings}

En apariencia, `parse_character()` debería ser realmente simple --- podría tan solo devolver su input. Desafortunadamente, la vida no es tan simple, dado que existen múltiples formas de representar la misma cadena de texto. Para entender qué está pasando, necesitamos profundizar en los detalles de cómo las computadoras representan las cadenas de texto. En R, podemos acceder a su representación subyacente empleando `charToRaw()`:



```r
charToRaw("Hadley")
#> [1] 48 61 64 6c 65 79
```
Cada número hexadecimal representa un byte de información: `48` es _H_, `61` es _a_, y así. El mapeo desde un número hexadecimal a caracteres se denomina codificación o _encoding_ y, en este caso, la codificación utilizada se llama ASCII. ASCII hace un muy buen trabajo representando caracteres del inglés, ya que es el ***American** Standard Code for Information Interchange* (del inglés _Código Americano estandarizado para el intercambio de información_).

Las cosas se complican un poco más para lenguas distintas al inglés. En los comienzos de la computación existían muchos estándares de codificación para caracteres no-ingleses compitiendo.  Para poder interpretar correctamente una cadena de texto se necesita conocer tanto los valores como la codificación. Por ejemplo, dos codificaciones comunes son Latin1 (conocida también como ISO-8859-1 y utilizada para las lenguas del oeste de Europa) y Latin2 (o ISO-8859-2, utilizada para las lenguas de Europa del este). En Latin1, el byte 'b1' es "Â±", pero en Latin2, ¡es "ą"! Afortunadamente, en la actualidad hay un estándar que tiene soporte casi en todos lados: UTF-8. UTF-8 puede codificar casi cualquier caracter utilizado por humanos, así como muchos símbolos adicionales (¡como los emoji!).

**readr** utiliza UTF-8 en todas partes: asume que tus datos están codificados en UTF-8 cuando los lee y lo emplea siempre cuando los escribe. Esta es una buena opción por defecto, pero fallará con datos producidos por sistemas más viejos que no entienden UTF-8. Si te sucede esto, tus cadenas de texto se verán extrañas cuando las imprimas en la consola. Algunas veces solo uno o dos caracteres estarán errados. Otras veces obtendrás un total jeroglífico. Por ejemplo: 

```r
x1 <- "El Ni\xf1o was particularly bad this year"
x2 <- "\x82\xb1\x82\xf1\x82\xc9\x82\xbf\x82\xcd"
x1
#> [1] "El Ni\xf1o was particularly bad this year"
x2
#> [1] "\x82\xb1\x82\xf1\x82ɂ\xbf\x82\xcd"
```
Para corregir el problema necesitas especificar la codificación en `parse_character()`:

```r
parse_character(x1, locale = locale(encoding = "Latin1"))
#> [1] "El Niño was particularly bad this year"
parse_character(x2, locale = locale(encoding = "Shift-JIS"))
#> [1] "こんにちは"
```
¿Cómo encontrar la codificación correcta? Si tienes suerte, estará incluida en alguna parte de la documentación de los datos. Desafortunadamente raras veces es ese el caso, así que **readr** provee la función `guess_encoding()` para ayudarte a adivinarla. No es a prueba de tontos y funciona mejor cuando tienes mucho texto (a diferencia de aquí), pero es un punto de inicio razonable. Es esperable hacer varias pruebas con diferentes codificaciones antes de encontrar la correcta.

```r
guess_encoding(charToRaw(x1))
#> # A tibble: 2 x 2
#>   encoding   confidence
#>   <chr>           <dbl>
#> 1 ISO-8859-1       0.46
#> 2 ISO-8859-9       0.23
guess_encoding(charToRaw(x2))
#> # A tibble: 1 x 2
#>   encoding confidence
#>   <chr>         <dbl>
#> 1 KOI8-R         0.42
```
El primer argumento para `guess_encoding()` puede ser la ruta a un archivo o, como en este caso, un vector en bruto (útil si el texto ya se encuentra en R).
Las codificaciones son un tema rico y complejo y solo te hemos mostrado la superficie acá. Si quieres aprender más al respecto, te recomendamos que leas la explicación detallada en <http://kunststube.net/encoding/>.

### Factores {#readr-factors}
R utiliza factores para representar las variables categóricas que tienen un conjunto conocido de valores posibles. Puedes darle a `parse_factor()` un vector de niveles conocidos (`levels`) para generar una advertencia cada vez que haya un valor inesperado:


```r
fruta <- c("manzana", "banana")
parse_factor(c("manzana", "banana", "bananana"), levels = fruta)
#> Warning: 1 parsing failure.
#> row col           expected   actual
#>   3  -- value in level set bananana
#> [1] manzana banana  <NA>   
#> attr(,"problems")
#> # A tibble: 1 x 4
#>     row   col expected           actual  
#>   <int> <int> <chr>              <chr>   
#> 1     3    NA value in level set bananana
#> Levels: manzana banana
```
Si tienes muchas entradas problemáticas, a menudo es más fácil dejarlas como vectores de caracteres y luego utilizar las herramientas sobre las que aprenderás en los capítulos [Cadenas de caracteres] y [Factores] para limpiarlas.

### Fechas, fechas-horas, y horas {#readr-datetimes}

Debes elegir entre tres segmentadores dependiendo de si quieres una fecha (el número de los días desde el 01-01-1970), una fecha-hora (el número de segundos desde la medianoche del 01-01-1970) o una hora (el número de segundos desde la medianoche). Cuando se llaman sin argumentos adicionales: 

*   `parse_datetime()` asume una fecha-hora ISO8601. ISO8601 es un estándar internacional en el que los componentes de una fecha están organizados de mayor a menor: año, mes, día, hora, minuto, segundo.

  
  ```r
  parse_datetime("2010-10-01T2010")
  #> [1] "2010-10-01 20:10:00 UTC"
  # Si se omite la hora, será determinada como medianoche.
  parse_datetime("20101010")
  #> [1] "2010-10-10 UTC"
  ```
    
  Esta es la estandarización de fecha/hora más  importante. Si trabajas con fechas y horas   frecuentemente, te recomendamos que leas <https://en.wikipedia.org/wiki/ISO_8601>
    
*   `parse_date()` asume un año de cuatro dígitos, un guión `-` o `/`, el mes, un guión `-` o `/` y luego el día.
     
     ```r
         parse_date("2010-10-01")
     #> [1] "2010-10-01"
     ```
*   `parse_time()` espera la hora, `:`, minutos, opcionalmente `:` y segundos, y un especificador opcional am/pm:
     
     ```r
         library(hms)
         parse_time("01:10 am")
     #> 01:10:00
         parse_time("20:10:01")
     #> 20:10:01
     ```
    
    R base no tiene incorporada una muy buena clase para datos temporales, por lo que usamos la provista en el paquete **hms**.
    
Si esos valores por defecto no funcionan con tus datos, puedes proporcionar tu propio formato fecha-hora construido con las siguientes piezas:

_**Año**_

  `%Y` (4 dígitos). 
  
  `%y` (2 dígitos); 00-69 -> 2000-2069, 70-99 -> 1970-1999.
  
_**Mes**_

  `%m` (2 dígitos).
  
  `%b` (nombre abreviado, como "ene").
  
  `%B` (nombre completo, "enero").
  
_**Día**_

  `%d` (2 dígitos).
  
  `%e` (espacio opcional destacado).
  
_**Hora**_

  `%H` 0-23 horas.
  
  `%I` 0-12, debe utilizarse con `%p`.
  
  `%p` indicador AM/PM.
  
  `%M` minutos.
  
  `%S` segundos enteros.
  
  `%OS` segundos reales. 
  
  `%Z` Zona horaria (como nombre, por ejemplo, `America/Chicago`). Advertencia sobre abreviaturas: 
  si eres de EEUU, ten en cuenta que "EST" es una zona horaria canadiense que no tiene cambios de horario.¡**No** es la hora Estandar del Este! Retomaremos esto más adelante en la sección [Husos horarios].
  
  `%z` (como complemento para las UTC, por ejemplo, `+0800`). 
  
_**No-dígitos**_

  `%.` se salta un caracter no-dígito.
  
  `%*` se salta cualquier número de caracteres no-dígitos.

La mejor manera de deducir el formato correcto es crear unos pocos ejemplos en un vector de caracteres y probarlos con una de las funciones de segmentación. Por ejemplo: 


```r
parse_date("01/02/15", "%m/%d/%y")
#> [1] "2015-01-02"
parse_date("01/02/15", "%d/%m/%y")
#> [1] "2015-02-01"
parse_date("01/02/15", "%y/%m/%d")
#> [1] "2001-02-15"
```
Si estás utilizando `%b` o `%B` con nombres de meses no ingleses, necesitarás ajustar el argumento `lang` para `locale()`. Mira la lista de lenguas incorporados en `date_names_langs()`.  Si tu lengua no está incluida, puedes crearla con `date_names()`.


```r
parse_date("1 janvier 2015", "%d %B %Y", locale = locale("fr"))
#> [1] "2015-01-01"
```

### Ejercicios
1.  ¿Cuáles son los argumentos más importantes para `locale()`?
1.  ¿Qué pasa si intentas establecer `decimal_mark` y `grouping_mark` como el mismo caracter? ¿Qué pasa con el valor por defecto de `grouping_mark` cuando estableces `decimal_mark` como `,`? ¿Qué pasa con el valor por defecto de `decimal_mark` cuando estableces `grouping_mark` como `.`?
1.  No discutimos las opciones de `date_format` y `time_format` para `locale()`. ¿Qué hacen? Construye un ejemplo que muestre cuándo podrían ser útiles.
1.  Si vives fuera de EEUU, crea un nuevo objeto locale que contenga las opciones para los tipos de archivo que lees más comúnmente.
1.  ¿Cuál es la diferencia entre `read_csv()` y `read_csv2()`?
1.  ¿Cuáles son las codificaciones más comunes empleadas en Europa? ¿Cuáles son las codificaciones más comunes utilizadas en Asia? ¿Y en América Latina? Googlea un poco para descubrirlo. 
1.  Genera el formato correcto de texto para segmentar cada una de las siguientes fechas y horas:
    
    ```r
    d1 <- "Enero 1, 2010"
    d2 <- "2015-Ene-07"
    d3 <- "06-Jun-2017"
    d4 <- c("Augosto 19 (2015)", "Julio 1 (2015)")
    d5 <- "12/30/14" # Dec 30, 2014
    t1 <- "1705"
    t2 <- "11:15:10.12 PM"
    ```

## Segmentar un archivo

Ahora que aprendiste cómo analizar un vector individual, es tiempo de volver al comienzo y explorar cómo **readr** analiza un archivo. Hay dos cosas nuevas que aprenderás al respecto en esta sección: 

1.  Cómo **readr** deduce automáticamente el tipo de cada columna.
1.  Cómo sobreescribir las especificaciones por defecto.

### Estrategia
**readr** utiliza una heurística para deducir el tipo de cada columna: lee las primeras 1000 filas y utiliza una heurística (moderadamente conservadora) para deducir el formato de las columnas. Puedes simular este proceso con un vector de caracteres utilizando `guess_parser()`, que devuelve la mejor deducción de **readr**, y `parse_guess()` que utiliza esa deducción para analizar la columna:


```r
guess_parser("2010-10-01")
#> [1] "date"
guess_parser("15:01")
#> [1] "time"
guess_parser(c("TRUE", "FALSE"))
#> [1] "logical"
guess_parser(c("1", "5", "9"))
#> [1] "double"
guess_parser(c("12,352,561"))
#> [1] "number"
str(parse_guess("2010-10-10"))
#>  Date[1:1], format: "2010-10-10"
```

La heurística prueba cada uno de los siguientes tipos y se detiene cuando encuentra una coincidencia:

* lógico: contiene solo "F", "T", "FALSE", o "TRUE".
* entero: contiene solo caracteres numéricos (y '-').
* doble: contiene solo dobles válidos (incluyendo números como '4.5e-5').
* número: contiene dobles válidos con la marca de agrupamiento en su interior.
* hora: coincide con el formato horario por defecto (`time_format`).
* fecha: coincide con el formato fecha por defecto (`date_format`).
* fecha-hora: cualquier fecha ISO8601.

Si ninguna de esas reglas se aplica, entonces la columna quedará como un vector de cadenas de caracteres.

### Problemas

Esos valores por defecto no siempre funcionan para archivos de gran tamaño. Hay dos problemas básicos:

1.  Las primeras mil filas podrían ser un caso especial y **readr** estaría deduciendo un formato que no es suficientemente general. Por ejemplo, podrías tener una columna de dobles que solo contiene enteros en las primeras 1000 filas.
1.  La columna podría contener muchos valores faltantes. Si las primeras 1000 filas contienen solo `NA`, **readr** deducirá que es un vector lógico, mientras que tú probablemente quieras analizarlo como algo más específico.

**readr** contiene un archivo csv desafiante que ilustra ambos problemas:


```r
desafio <- read_csv(readr_example("challenge.csv"))
#> Parsed with column specification:
#> cols(
#>   x = col_double(),
#>   y = col_logical()
#> )
#> Warning: 1000 parsing failures.
#>  row col           expected     actual                                                 file
#> 1001   y 1/0/T/F/TRUE/FALSE 2015-01-16 '/home/travis/R/Library/readr/extdata/challenge.csv'
#> 1002   y 1/0/T/F/TRUE/FALSE 2018-05-18 '/home/travis/R/Library/readr/extdata/challenge.csv'
#> 1003   y 1/0/T/F/TRUE/FALSE 2015-09-05 '/home/travis/R/Library/readr/extdata/challenge.csv'
#> 1004   y 1/0/T/F/TRUE/FALSE 2012-11-28 '/home/travis/R/Library/readr/extdata/challenge.csv'
#> 1005   y 1/0/T/F/TRUE/FALSE 2020-01-13 '/home/travis/R/Library/readr/extdata/challenge.csv'
#> .... ... .................. .......... ....................................................
#> See problems(...) for more details.
```
(Fíjate en el uso de `readr_example()`, que encuentra la ruta a uno de los archivos incluidos en el paquete.)

Hay dos outputs impresos en la consola: la especificación de columna generada al mirar las primeras 1000 filas y las primeras cinco fallas de segmentación. Siempre es una buena idea extraer explícitamente los problemas con `problems()`, así puedes explorarlos en mayor profundidad:

```r
problems(desafio)
#> # A tibble: 1,000 x 5
#>     row col   expected         actual    file                                   
#>   <int> <chr> <chr>            <chr>     <chr>                                  
#> 1  1001 y     1/0/T/F/TRUE/FA… 2015-01-… '/home/travis/R/Library/readr/extdata/…
#> 2  1002 y     1/0/T/F/TRUE/FA… 2018-05-… '/home/travis/R/Library/readr/extdata/…
#> 3  1003 y     1/0/T/F/TRUE/FA… 2015-09-… '/home/travis/R/Library/readr/extdata/…
#> 4  1004 y     1/0/T/F/TRUE/FA… 2012-11-… '/home/travis/R/Library/readr/extdata/…
#> 5  1005 y     1/0/T/F/TRUE/FA… 2020-01-… '/home/travis/R/Library/readr/extdata/…
#> 6  1006 y     1/0/T/F/TRUE/FA… 2016-04-… '/home/travis/R/Library/readr/extdata/…
#> # … with 994 more rows
```
Una buena estrategia es trabajar columna por columna hasta que no queden problemas. Aquí podemos ver que hubo muchos problemas de análisis con la columna `y`. Si miramos la últimas líneas, verás que hay fechas almacenadas en un vector de caracteres. 


```r
tail(desafio)
#> # A tibble: 6 x 2
#>       x y    
#>   <dbl> <lgl>
#> 1 0.805 NA   
#> 2 0.164 NA   
#> 3 0.472 NA   
#> 4 0.718 NA   
#> 5 0.270 NA   
#> 6 0.608 NA
```

Esto sugiere que mejor sería utilizar un segmentador de fechas. Para arreglar este problema, copia y pega la especificación de las columnas que habías obtenido inicialmente y agrégalas a tu código:


```r
desafio <- read_csv(
  readr_example("challenge.csv"), 
  col_types = cols(
    x = col_double(),
    y = col_logical()
  )
)
```

Y luego ajusta el tipo de la columna `y` especificando que se trata de una fecha:


```r
desafio <- read_csv(
  readr_example("challenge.csv"), 
  col_types = cols(
    x = col_double(),
    y = col_date()
  )
)
tail(desafio)
#> # A tibble: 6 x 2
#>       x y         
#>   <dbl> <date>    
#> 1 0.805 2019-11-21
#> 2 0.164 2018-03-29
#> 3 0.472 2014-08-04
#> 4 0.718 2015-08-16
#> 5 0.270 2020-02-04
#> 6 0.608 2019-01-06
```

Cada función `parse_*()` tiene su correspondiente función `col_*()`. Se utiliza `parse_*()` cuando los datos se encuentran en un vector de caracteres que ya está disponible en R; `col_*` para cuando quieres decirle a **readr** cómo cargar los datos.

Te recomendamos proporcionar la estructura para `col_types` a partir de la impresión en consola provista por **readr**. Esto asegura que tienes un script para importar datos consistente y reproducible. Si confías en las deducciones por defecto y tus datos cambian, **readr** continuará leyéndolos. Si quieres ser realmente estricto/a, emplea `stop_for_problems()` (_detenerse en problemas_): esto devolverá un mensaje de error y detendrá tu script si hay cualquier problema con la segmentación.

### Otras estrategias

Existen algunas estrategias generales más para ayudarte a segmentar archivos:

*   En el ejemplo previo simplemente tuvimos mala suerta: si miramos solo una fila más que el número por defecto, podemos segmentar correctamente en un solo intento:
   
    
    ```r
    desafio2 <- read_csv(readr_example("challenge.csv"), guess_max = 1001)
    #> Parsed with column specification:
    #> cols(
    #>   x = col_double(),
    #>   y = col_date(format = "")
    #> )
    desafio2
    #> # A tibble: 2,000 x 2
    #>       x y         
    #>   <dbl> <date>    
    #> 1   404 NA        
    #> 2  4172 NA        
    #> 3  3004 NA        
    #> 4   787 NA        
    #> 5    37 NA        
    #> 6  2332 NA        
    #> # … with 1,994 more rows
    ```

*   Algunas veces es más fácil diagnosticar problemas si lees todas las columnas como vectores de caracteres:

    
    ```r
    desafio <- read_csv(readr_example("challenge.csv"), 
      col_types = cols(.default = col_character())
    )
    ```
    
    Esto es particularmente útil en combinación con 'type_convert()', que aplica la heurística de segmentación a las columnas de caracteres en un data frame.
    
    
    ```r
      df <- tribble(
        ~x,  ~y,
        "1", "1.21",
        "2", "2.32",
        "3", "4.56"
      )
      df
    #> # A tibble: 3 x 2
    #>   x     y    
    #>   <chr> <chr>
    #> 1 1     1.21 
    #> 2 2     2.32 
    #> 3 3     4.56
      
    # Fíjate en los tipos de columna
    type_convert(df)
    #> Parsed with column specification:
    #> cols(
    #>   x = col_double(),
    #>   y = col_double()
    #> )
    #> # A tibble: 3 x 2
    #>       x     y
    #>   <dbl> <dbl>
    #> 1     1  1.21
    #> 2     2  2.32
    #> 3     3  4.56
    ```
    
*   Si estás leyendo un archivo muy largo, podrías querer seleccionar `n_max` a un número pequeño como 10000 o 100000. Esto acelerará las iteraciones a la vez que eliminarás problemas comunes.

*   Si tienes problemas de segmentación importantes, a veces es más fácil leer un vector de caracteres de líneas con `read_lines()`, o incluso un vector de caracteres de largo 1 con `read_file()`. Luego puedes utilizar las habilidades sobre segmentación de cadenas de caracteres que aprenderás más adelante para segmentar formatos más exóticos.

## Escribir a un archivo

**readr** también incluye dos funciones muy útiles para escribir datos de vuelta al disco: `write_csv()` y `write_tsv()`. Ambas funciones incrementan las posibilidades de que el archivo resultante sea leído correctamente al: 

* codificar siempre las cadenas de caracteres en UTF-8.
* guardar fechas y fechas-horas en formato ISO8601, por lo que son fácilmente segmentadas en cualquier sitio.

Si quieres exportar un archivo csv a Excel, utiliza `write_excel_csv()` ---esto escribe un caracter especial (una marca de orden de bytes) al comienzo del archivo que le dice a Excel que estás utilizando codificación UTF-8.
Los argumentos más importantes son `x` (el data frame a guardar) y `path` (la ubicación donde lo guardarás). También puedes especificar cómo se escriben los valores ausentes con `na` y si quieres `append` (agregarlo) a un archivo existente.


```r
write_csv(desafio, "desafio.csv")
```

Fíjate que la información sobre el tipo de datos se pierde cuando guardas en csv:


```r
desafio
#> # A tibble: 2,000 x 2
#>   x     y    
#>   <chr> <chr>
#> 1 404   <NA> 
#> 2 4172  <NA> 
#> 3 3004  <NA> 
#> 4 787   <NA> 
#> 5 37    <NA> 
#> 6 2332  <NA> 
#> # … with 1,994 more rows
write_csv(desafio, "desafio-2.csv")
read_csv("desafio-2.csv")
#> Parsed with column specification:
#> cols(
#>   x = col_double(),
#>   y = col_logical()
#> )
#> # A tibble: 2,000 x 2
#>       x y    
#>   <dbl> <lgl>
#> 1   404 NA   
#> 2  4172 NA   
#> 3  3004 NA   
#> 4   787 NA   
#> 5    37 NA   
#> 6  2332 NA   
#> # … with 1,994 more rows
```

Esto hace a los CSV poco confiables para almacenar en caché los resultados provisorios --- necesitas recrear la especificación de las columnas cada vez que los cargas. Hay dos alternativas:

1.  `write_rds()` and `read_rds()` son funciones "envoltorio" (_wrappers_) uniformes sobre las funciones base `readRDS()` y `saveRDS()`. Estas almacenan datos en un formato binario propio de R llamado RDS:

    
    ```r
    write_rds(desafio, "desafio.rds")
    read_rds("desafio.rds")
    #> # A tibble: 2,000 x 2
    #>   x     y    
    #>   <chr> <chr>
    #> 1 404   <NA> 
    #> 2 4172  <NA> 
    #> 3 3004  <NA> 
    #> 4 787   <NA> 
    #> 5 37    <NA> 
    #> 6 2332  <NA> 
    #> # … with 1,994 more rows
    ```
  
1. El paquete **feather** implementa un formato rápido de archivos binarios que puede compartirse a través de lenguajes de programación:
    
    
    ```r
    library(feather)
    write_feather(desafio, "desafio.feather")
    read_feather("desafio.feather")
    #> # A tibble: 2,000 x 2
    #>       x      y
    #>   <dbl> <date>
    #> 1   404   <NA>
    #> 2  4172   <NA>
    #> 3  3004   <NA>
    #> 4   787   <NA>
    #> 5    37   <NA>
    #> 6  2332   <NA>
    #> # ... with 1,994 more rows
    ```

**Feather** tiende a ser más rápido que RDS y es utilizable fuera de R. RDS permite columnas-listas (sobre las que aprenderás en el capítulo [Muchos modelos]), algo que **feather** no permite actualmente.



## Otros tipos de datos
Para acceder a otros tipos de datos en R te recomendamos comenzar con los paquetes de **tidyverse** listados abajo. Ciertamente no son perfectos, pero son un buen lugar para comenzar. Para datos rectangulares: 

* __haven__ lee archivos SPSS, Stata y SAS.
* __readxl__ lee archivos excel (tanto `.xls` como `.xlsx`).
* __DBI__, junto con un _backend_ de base de datos específico (e.g. __RMySQL__, __RSQLite__, __RPostgreSQL__, etc.) te permite correr consultas SQL contra una base de datos y devolver un data frame.
  
Para datos jerárquicos: utiliza __jsonlite__ (de Jeroen Ooms) para json y __xml2__ para XML. Jenny Bryan tiene algunos ejemplos muy bien trabajados en <https://jennybc.github.io/purrr-tutorial/>.

Para otros tipos de archivos, prueba el [manual de importación/exportación de datos de R](https://cran.r-project.org/doc/manuals/r-release/R-data.html) y el paquete [__rio__](https://github.com/leeper/rio).
