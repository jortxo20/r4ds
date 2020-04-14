# Cadenas de caracteres

## Introducción

Este capítulo te introduce en la manipulación de cadenas de caracteres en R. Si bien aprenderás los aspectos básicos acerca de cómo funcionan y cómo crearlas a mano, el foco del capítulo estará puesto en las expresiones regulares (o _regex_). Como las cadenas de caracteres suelen contener datos no estructurados o semiestructurados, las expresiones regulares resultan útiles porque permiten describir patrones en ellas a través de un lenguaje conciso. Cuando mires por primera vez una expresión regular te parecerá que un gato caminó sobre tu teclado, pero a medida que vayas ampliando tu conocimiento pronto te empezarán a hacer sentido. 

### Prerequisitos

Este capítulo se enfocará en el paquete para manipulación de cadenas llamado __stringr__ (del inglés _string_, cadena), que es parte del Tidyverse.


```r
library(tidyverse)
library(datos)
```

## Cadenas: elementos básicos

Puedes crear una cadena utilizando comillas simples o dobles. A diferencia de otros lenguajes, no hay diferencias en su comportamiento. Nuestra recomendación es siempre utilizar `"`, a menos que quieras crear una cadena que contenga múltiples `"`.


```r
string1 <- "Esta es una cadena de caracteres"
string2 <- 'Si quiero incluir "comillas" dentro de la cadena, uso comillas simples'
```

Si olvidas cerrar las comillas, verás un `+` en la consola, que es el signo de continuación para indicar que el código no está completo:

```
> "Esta es una cadena de caracteres sin comillas de cierre
+ 
+ 
+ AYUDA, ESTOY ATASCADO
```

Si esto te ocurre, ¡presiona la tecla _Escape_ e inténtalo de nuevo!

Para incluir comillas simples o dobles de manera literal en una cadena puedes utilizar `\` para "escaparlas" ("escapar" viene de la tecla _escape_):


```r
comilla_doble <- "\"" # o '"'
comilla_simple <- '\'' # o "'"
```

Esto quiere decir que si quieres incluir una barra invertida, necesitas duplicarla: `"\\"`.

Ten cuidado con el hecho de que la representación impresa de una cadena no es equivalente a la cadena misma, ya que la representación muestra las barras utilizadas para "escapar" caracteres, es decir, para sean interpretados en su sentido literal, no como caracteres especiales. Para ver el contenido en bruto de una cadena utiliza `writeLines()`:


```r
x <- c("\"", "\\")
x
#> [1] "\"" "\\"
writeLines(x)
#> "
#> \
```

Existe una serie de otros caracteres especiales. Los más comunes son `"\n"`, para salto de línea, y `"\t"`, para tabulación. Puedes ver la lista completa pidiendo ayuda acerca de `"`: `?'"'` o `?"'"`. A veces también verás cadenas del tipo `"\u00b5"`, que es la manera de escribir caracteres que no están en inglés para que funcionen en todas las plataformas:


```r
x <- "\u00b5"
x
#> [1] "µ"
```

Usualmente se guardan múltiples cadenas en un vector de caracteres. Puedes crearlo usando `c()`:


```r
c("uno", "dos", "tres")
#> [1] "uno"  "dos"  "tres"
```

### Largo de cadena

R base tiene muchas funciones para trabajar con cadenas de caracteres, pero las evitaremos porque pueden ser incosistentes, lo que hace que sean difíciles de recordar. En su lugar, utilizaremos funciones del paquete __stringr__. Estas tienen nombres más intuitivos y todas empienzan con `str_`. Por ejemplo, `str_length()` te dice el número de caracteres de una cadena (_length_ en inglés es _largo_): 


```r
str_length(c("a", "R para Ciencia de Datos", NA))
#> [1]  1 23 NA
```

El prefijo común `str_` es particularmente útil si utilizas RStudio, ya que al escribir `str_` se activa el autocompletado, lo que te permite ver todas las funciones de __stringr__:

<img src="screenshots/stringr-autocomplete.png" width="70%" style="display: block; margin: auto;" />

### Combinar cadenas

Para combinar dos o más cadenas utiliza `str_c()`:


```r
str_c("x", "y")
#> [1] "xy"
str_c("x", "y", "z")
#> [1] "xyz"
```

Usa el argumento `sep` para controlar cómo separlas:


```r
str_c("x", "y", sep = ", ")
#> [1] "x, y"
```

Al igual que en muchas otras funciones de R, los valores faltantes son contagiosos. Si quieres que se impriman como `"NA"`, utiliza `str_replace_na()` (_replace_ = remplazar):


```r
x <- c("abc", NA)
str_c("|-", x, "-|")
#> [1] "|-abc-|" NA
str_c("|-", str_replace_na(x), "-|")
#> [1] "|-abc-|" "|-NA-|"
```

Como se observa arriba, `str_c()` es una función vectorizada que automáticamente recicla los vectores más cortos hasta alcanzar la extensión del más largo: 


```r
str_c("prefijo-", c("a", "b", "c"), "-sufijo")
#> [1] "prefijo-a-sufijo" "prefijo-b-sufijo" "prefijo-c-sufijo"
```

Los objetos de extensión 0 se descartan de manera silenciosa. Esto es particularmente útil en conjunto con `if` (_si_):


```r
nombre <- "Hadley"
hora_del_dia <- "mañana"
cumpleanios <- FALSE

str_c(
  "Que tengas una buena ", hora_del_dia, ", ", nombre,
  if (cumpleanios) " y ¡FELIZ CUMPLEAÑOS!",
  "."
)
#> [1] "Que tengas una buena mañana, Hadley."
```

Para colapsar un vector de cadenas en una sola, utiliza `collapse`:


```r
str_c(c("x", "y", "z"), collapse = ", ")
#> [1] "x, y, z"
```

### Dividir cadenas

Puedes extraer partes de una cadena utilizando `str_sub()`. Al igual que la cadena, `str_sub()` tiene como argumentos `start` (_inicio_) y `end` (_fin_), que indican la posición (inclusiva) del subconjunto que se quiere extraer:


```r
x <- c("Manzana", "Plátano", "Pera")
str_sub(x, 1, 3)
#> [1] "Man" "Plá" "Per"
# los números negativos cuentan de manera invertida desde el final
str_sub(x, -3, -1)
#> [1] "ana" "ano" "era"
```

Ten en cuenta que `str_sub()` no fallará si la cadena es muy corta; simplemente devolverá todo lo que sea posible: 


```r
str_sub("a", 1, 5)
#> [1] "a"
```

También puedes utilizar `str_sub()` en forma de asignación para modificar una cadena: 


```r
str_sub(x, 1, 1) <- str_to_lower(str_sub(x, 1, 1))
x
#> [1] "manzana" "plátano" "pera"
```

### _Locales_

Arriba utilizamos `str_to_lower()` para cambiar el texto a minúsculas. También puedes utilizar `str_to_upper()` o `str_to_title()`, si quieres modificar el texto a mayúsculas o formato título, respectivamente. Sin embargo, este tipo de cambios puede ser más complicado de lo parece a primera vista, ya que las reglas no son iguales en todos los idiomas. Puedes selecionar qué tipo de reglas aplicar especificando el entorno local o _locale_:


```r
# La lengua turca tiene dos i: una con punto y otra sin punto
# Tienen diferentes reglas para convertirlas en mayúsculas

str_to_upper(c("i", "ı"))
#> [1] "I" "I"
str_to_upper(c("i", "ı"), locale = "tr")
#> [1] "İ" "I"
```

El entorno local o _locale_ se especifica con un código de idioma ISO 639, que es una abreviación de dos letras. Si todavía no conoces el código para tu idioma, en [Wikipedia](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) puedes encontrar una buena lista. Si dejas el _locale_ en blanco, se aplicará el que estés utilizando actualmente, que es provisto por tu sistema operativo. 

Otra operación importante que es afectada por el _locale_ es ordenar. Las funciones `order()` y `sort()` de R base ordenan las cadenas usando el _locale_ actual. Si quieres un comportamiento consistente a través de diferentes computadoras, sería preferible usar `str_sort()` y `str_order()`, que aceptan un argumento adicional para definir el `locale`:


```r
x <- c("arándano", "espinaca", "banana")

str_sort(x, locale = "es")  # Español
#> [1] "arándano" "banana"   "espinaca"

str_sort(x, locale = "haw") # Hawaiano
#> [1] "arándano" "espinaca" "banana"
```

### Ejercicios

1.  En ejemplos de código en los que no se utiliza __stringr__, verás usualmente `paste()` y `paste0()` (_paste_ = pegar).
    ¿Cuál es la diferencia entre estas dos funciones? ¿A qué función de __stringr__ son equivalentes? ¿Cómo difieren estas dos funciones respecto de su manejo de los
    `NA`?
    
1.  Describe con tus propias palabras la diferencia entre los argumentos `sep` y `collapse`
    de la función `str_c()`.

1.  Utiliza `str_length()` y `str_sub()` para extraer el caracter del medio de una cadena. ¿Qué harías si el número de caracteres es par?

1.  ¿Qué hace `str_wrap()`? (_wrap_ = envolver) ¿Cuándo podrías querer utilizarla?

1.  ¿Qué hace `str_trim()`? (_trim_ = recortar) ¿Cuál es el opuesto de `str_trim()`?

1.  Escribe una función que convierta, por ejemplo, el vector `c("a", "b", "c")` en
    la cadena `a, b y c`. Piensa con detención qué debería hacer 
    dado un vector de largo 0, 1 o 2.

### Buscar coincidencia de patrones con expresiones regulares

Las expresiones regulares son un lenguaje conciso que te permite describir patrones en cadenas de caracteres. Toma un tiempo entenderlas, pero una vez que lo hagas te darás cuenta que son extremadamente útiles. 

Para aprender sobre expresiones regulares usaremos `str_view()` y `str_view_all()` (_view_ = ver). Estas funciones toman un vector de caracteres y una expresión regular y te muestran cómo coinciden. Partiremos con expresiones regulares simples que gradualmente se irán volviendo más y más complejas. Una vez que domines la coincidencia de patrones, aprenderás cómo aplicar estas ideas con otras funciones de __stringr__. 

### Coincidencias básicas

Los patrones más simples buscan coincidencias con cadenas exactas:


```r
x <- c("manzana", "banana", "pera")
str_view(x, "an")
```

<!--html_preserve--><div id="htmlwidget-ac96cb3ee4656e2e9ec3" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-ac96cb3ee4656e2e9ec3">{"x":{"html":"<ul>\n  <li>m<span class='match'>an<\/span>zana<\/li>\n  <li>b<span class='match'>an<\/span>ana<\/li>\n  <li>pera<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

El siguiente paso en complejidad es `.`, que coincide con cualquier caracter (excepto un salto de línea):


```r
str_view(x, ".a.")
```

<!--html_preserve--><div id="htmlwidget-e5c8c404fe174e4c81bd" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-e5c8c404fe174e4c81bd">{"x":{"html":"<ul>\n  <li><span class='match'>man<\/span>zana<\/li>\n  <li><span class='match'>ban<\/span>ana<\/li>\n  <li>pera<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

Pero si "`.`" coincide con cualquier caracter, ¿cómo buscar una coincidencia con el caracter "`.`"? Necesitas utilizar un "escape" para decirle a la expresión regular que quieres hacerla coincidir de manera exacta, no usar su comportamiento especial. Al igual que en las cadenas, las expresiones regulares usan la barra invertida, `\`, para "escapar" los comportamientos especiales. Por lo tanto, para hacer coincidir un `.`, necesitas la expresión regular `\.`. Lamentablemente, esto crea una problema. Estamos usando cadenas para representar una expresión regular y en ellas  `\` también se usa como símbolo de "escape". Por lo tanto, para crear la expresión regular `\.` necesitamos la cadena `"\\."`. 


```r
# Para crear una expresión regular necesitamos \\
punto <- "\\."

# Pero la expresión en sí misma solo contiene una \
writeLines(punto)
#> \.

# Esto le dice a R que busque el . de manera explícita
str_view(c("abc", "a.c", "bef"), "a\\.c")
```

<!--html_preserve--><div id="htmlwidget-36aa3d2a04d42bbc2145" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-36aa3d2a04d42bbc2145">{"x":{"html":"<ul>\n  <li>abc<\/li>\n  <li><span class='match'>a.c<\/span><\/li>\n  <li>bef<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

Si `\` se utiliza para escapar un caracter en una expresión regular, ¿cómo coincidir de manera literal una `\`? Bueno, necesitarías escaparla creando la expresión regular `\\`. Para crear esa expresión regular necesitas usar una cadena, que requiere también escapar la `\`. Esto quiere decir que para coincidir literalmente `\` necesitas escribir `"\\\\"` --- ¡necesitas cuatro barras invertidas para coincidir una! 


```r
x <- "a\\b"
writeLines(x)
#> a\b

str_view(x, "\\\\")
```

<!--html_preserve--><div id="htmlwidget-febe03efa1a2d8d52a86" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-febe03efa1a2d8d52a86">{"x":{"html":"<ul>\n  <li>a<span class='match'>\\<\/span>b<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

En este libro escribiremos las expresiones regulares como `\.` y las cadenas que representan a las expresiones regulares como `"\\."`.

#### Ejercicios

1.  Explica por qué cada una de estas cadenas no coincide con `\`: `"\"`, `"\\"`, `"\\\"`.

1.  ¿Cómo harías coincidir la secuencia `"'\`?

1.  ¿Con qué patrones coincidiría la expresión regular`\..\..\..`? 
    ¿Cómo la representarías en una cadena?

### Anclas

Por defecto, las expresiones regulares buscarán una coincidencia en cualquier parte de una cadena. Suele ser útil _anclar_ una expresión regular para que solo busque coincidencias al inicio o al final. Puedes utilizar: 

* `^` para buscar la coincidencia al inicio de la cadena.
* `$` para buscar la coincidencia al final de la cadena.


```r
x <- c("arándano", "banana", "pera")
str_view(x, "^a")
```

<!--html_preserve--><div id="htmlwidget-1fb4450895fe099f74a1" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-1fb4450895fe099f74a1">{"x":{"html":"<ul>\n  <li><span class='match'>a<\/span>rándano<\/li>\n  <li>banana<\/li>\n  <li>pera<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

```r
str_view(x, "a$")
```

<!--html_preserve--><div id="htmlwidget-10b3b7155e8045a1b2ad" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-10b3b7155e8045a1b2ad">{"x":{"html":"<ul>\n  <li>arándano<\/li>\n  <li>banan<span class='match'>a<\/span><\/li>\n  <li>per<span class='match'>a<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

Para recordar cuál es cuál, puedes intentar este recurso mnemotécnico que aprendimos de [Evan Misshula](https://twitter.com/emisshula/status/323863393167613953): si empiezas con potencia (`^`), terminarás con dinero (`$`).

Para forzar que una expresión regular coincida con una cadena completa, ánclala usando tanto `^` como `$`:


```r
x <- c("pie de manzana", "manzana", "queque de manzana")
str_view(x, "manzana")
```

<!--html_preserve--><div id="htmlwidget-4018eef1a407a0df6b52" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-4018eef1a407a0df6b52">{"x":{"html":"<ul>\n  <li>pie de <span class='match'>manzana<\/span><\/li>\n  <li><span class='match'>manzana<\/span><\/li>\n  <li>queque de <span class='match'>manzana<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

```r
str_view(x, "^manzana$")
```

<!--html_preserve--><div id="htmlwidget-5b1b2f4ad92281566982" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-5b1b2f4ad92281566982">{"x":{"html":"<ul>\n  <li>pie de manzana<\/li>\n  <li><span class='match'>manzana<\/span><\/li>\n  <li>queque de manzana<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

También puedes coincidir el límite entre palabras con `\b`. No utilizamos frecuentemente esta forma en R, pero sí a veces cuando buscamos en RStudio el nombre de una función que también compone el nombre de otras funciones. Por ejemplo, buscaríamos `\bsum\b` para evitar la coincidencia con `summarise`, `summary`, `rowsum` y otras.

#### Ejercicios

1.  ¿Cómo harías coincidir la cadena `"$^$"` de manera literal?

1.  Dado el corpus de palabras comunes en `datos::palabras`, crea una expresión
    regular que busque palabras que:
    
    1. Empiecen con "y".
    1. Terminen con "z"
    1. Tengan una extensión de exactamente tres letras. (¡No hagas trampa usando `str_length()`!)
    1. Tengan ocho letras o más. 

    Dado que esta será una lista larga, podrías querer usar el argumento `match` en
    `str_view()` para mostrar solo las palabras que coincidan o no coincidan. 

### Clases de caracteres y alternativas

Existe una serie de patrones especiales que coinciden con más de un caracter. Ya has visto `.`, que coincide con cualquier caracter excepto un salto de línea. Hay otras cuatro herramientas que son de utilidad: 

* `\d`: coincide con cualquier dígito.
* `\s`: coincide con cualquier espacio en blanco (por ejemplo, espacio simple, tabulador, salto de línea).
* `[abc]`: coincide con a, b o c.
* `[^abc]`: coincide con todo menos con a, b o c.

Recuerda que para crear una expresión regular que contenga `\d` o `\s` necesitas escapar la `\` en la cadena, por lo que debes escribir `"\\d"` o `"\\s"`.

Utilizar una clase de caracter que contenga en su interior un solo caracter puede ser una buena alternativa a la barra invertida cuando quieres incluir un solo metacaracter en la expresión regular. Muchas personas encuentran que así es más fácil de leer. 


```r
# Buscar de forma literal un caracter que usualmente tiene un significado especial en una expresión regular
str_view(c("abc", "a.c", "a*c", "a c"), "a[.]c")
```

<!--html_preserve--><div id="htmlwidget-25c3e940e6859592f801" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-25c3e940e6859592f801">{"x":{"html":"<ul>\n  <li>abc<\/li>\n  <li><span class='match'>a.c<\/span><\/li>\n  <li>a*c<\/li>\n  <li>a c<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

```r
str_view(c("abc", "a.c", "a*c", "a c"), ".[*]c")
```

<!--html_preserve--><div id="htmlwidget-3f27c09be0c60bb52829" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-3f27c09be0c60bb52829">{"x":{"html":"<ul>\n  <li>abc<\/li>\n  <li>a.c<\/li>\n  <li><span class='match'>a*c<\/span><\/li>\n  <li>a c<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

```r
str_view(c("abc", "a.c", "a*c", "a c"), "a[ ]")
```

<!--html_preserve--><div id="htmlwidget-416566eb193bf50d04e6" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-416566eb193bf50d04e6">{"x":{"html":"<ul>\n  <li>abc<\/li>\n  <li>a.c<\/li>\n  <li>a*c<\/li>\n  <li><span class='match'>a <\/span>c<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

Esto funciona para la mayoría (pero no para todos) los metacaracteres de las expresiones regulares: `$` `.` `|` `?` `*` `+` `(` `)` `[` `{`. Desafortunadamente, existen unos pocos caracteres que tienen un significado especial incluso dentro de una clase de caracteres y deben manejarse con barras invertidas para escaparlos: `]` `\` `^` y `-`.

Puedes utiizar una _disyunción_ para elegir entre uno más patrones alternativos. Por ejemplo, `abc|d..a` concidirá tanto con '"abc"', como con `"duna"`. Ten en cuenta que la precedencia de `|` es baja, por lo que `abc|xyz` coincidirá con `abc` o `xyz`, no con `abcyz` o `abxyz`. Al igual que en expresiones matemáticas, si el valor de `|` se vuelve confuso, utiliza paréntesis para dejar claro qué es lo que quieres: 


```r
str_view(c("cómo", "como"), "c(ó|o)mo")
```

<!--html_preserve--><div id="htmlwidget-72cbf064100ce560a04c" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-72cbf064100ce560a04c">{"x":{"html":"<ul>\n  <li><span class='match'>cómo<\/span><\/li>\n  <li><span class='match'>como<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

#### Ejercicios

1.  Crea una expresión regular que encuentre todas las palabras que:

    1. Empiecen con una vocal.

    1. Solo contengan consonantes. (Pista: piensa en cómo buscar coincidencias para 
       "no"-vocales.)

    1. Terminen en `ón`, pero no en `ión`.
    
    1. Terminen con `ndo` o `ado`.
    

1.  ¿Siempre a una "q" la sigue una "u"?

1.  Escribe una expresión regular que permita buscar un verbo que haya sido escrito usando voseo en segunda persona plural  
    (por ejemplo, _queréis_ en vez de _quieren_).

1.  Crea una expresión regular que coincida con la forma en que habitualmente 
    se escriben los números de teléfono en tu país.
    
1. En inglés existe una regla que dice que la letra i va siempre antes de la e, excepto cuando está después de una c". Verifica empíricamente esta regla utilizando las palabras contenidas en `stringr::words`. 


### Repetición

El siguiente paso en términos de poder implica controlar cuántas veces queremos que se encuentre un patrón:

* `?`: 0 o 1
* `+`: 1 o más
* `*`: 0 o más


```r
x <- "1888 es el año más largo en números romanos: MDCCCLXXXVIII"
str_view(x, "CC?")
```

<!--html_preserve--><div id="htmlwidget-d11fc4360aa0230696d7" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-d11fc4360aa0230696d7">{"x":{"html":"<ul>\n  <li>1888 es el año más largo en números romanos: MD<span class='match'>CC<\/span>CLXXXVIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

```r
str_view(x, "CC+")
```

<!--html_preserve--><div id="htmlwidget-21c7483268bafca56cec" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-21c7483268bafca56cec">{"x":{"html":"<ul>\n  <li>1888 es el año más largo en números romanos: MD<span class='match'>CCC<\/span>LXXXVIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

```r
str_view(x, 'C[LX]+')
```

<!--html_preserve--><div id="htmlwidget-1834a22cd196f3aa03a1" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-1834a22cd196f3aa03a1">{"x":{"html":"<ul>\n  <li>1888 es el año más largo en números romanos: MDCC<span class='match'>CLXXX<\/span>VIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

Ten en cuenta que la precedencia de este operador es alta, por lo que puedes escribir: `cantái?s` para encontrar tanto voseo americano como de la variante peninsular del español (es decir, _cantás_ y _cantáis_). Esto quiere decir que en la mayor parte de los usos se necesitarán paréntesis, como `bana(na)+`.

También puedes especificar el número de coincidencias que quieres encontrar de manera precisa:

* `{n}`: exactamente n
* `{n,}`: n o más
* `{,m}`: no más de m
* `{n,m}`: entre n y m


```r
str_view(x, "C{2}")
```

<!--html_preserve--><div id="htmlwidget-28515d92cb327f90c9eb" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-28515d92cb327f90c9eb">{"x":{"html":"<ul>\n  <li>1888 es el año más largo en números romanos: MD<span class='match'>CC<\/span>CLXXXVIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

```r
str_view(x, "C{2,}")
```

<!--html_preserve--><div id="htmlwidget-0caf26d4e3c00206b0c5" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-0caf26d4e3c00206b0c5">{"x":{"html":"<ul>\n  <li>1888 es el año más largo en números romanos: MD<span class='match'>CCC<\/span>LXXXVIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

```r
str_view(x, "C{2,3}")
```

<!--html_preserve--><div id="htmlwidget-da0b268a2927f570ebf3" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-da0b268a2927f570ebf3">{"x":{"html":"<ul>\n  <li>1888 es el año más largo en números romanos: MD<span class='match'>CCC<\/span>LXXXVIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

Por defecto, este tipo de coincidencias son "avaras" (_greedy_): tratarán de coincidir con la cadena más larga posible. También puedes hacerlas "perezosas" (_lazy_) para que coincidan con la cadena más corta posible, poniendo un `?` después de ellas. Esta es una característica avanzada de las expresiones regulares, pero es útil saber que existe:


```r
str_view(x, 'C{2,3}?')
```

<!--html_preserve--><div id="htmlwidget-0ed12bb554391c49c2e3" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-0ed12bb554391c49c2e3">{"x":{"html":"<ul>\n  <li>1888 es el año más largo en números romanos: MD<span class='match'>CC<\/span>CLXXXVIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

```r
str_view(x, 'C[LX]+?')
```

<!--html_preserve--><div id="htmlwidget-ec658d41f8c4f2d124e9" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-ec658d41f8c4f2d124e9">{"x":{"html":"<ul>\n  <li>1888 es el año más largo en números romanos: MDCC<span class='match'>CL<\/span>XXXVIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

#### Ejercicios

1.  Describe los equivalentes de `?`, `+`, `*` en el formato `{m,n}`.

1.  Describe en palabras con qué coincidiría cada una de estas expresiones regulares:
    (lee con atención para ver si estamos utilizando una expresión regular o una cadena
    que define una expresión regular.)

    1. `^.*$`
    1. `"\\{.+\\}"`
    1. `\d{4}-\d{2}-\d{2}`
    1. `"\\\\{4}"`

1.  Crea expresiones regulares para buscar todas las palabras que:

    1. Empiecen con dos consonantes.
    1. Tengan tres o más vocales seguidas.
    1. Tengan tres o más pares de vocal-consonante seguidos. 

### Agrupamiento y referencias previas

Anteriormente aprendiste sobre el uso de paréntesis para desambiguar expresiones complejas. Los paréntesis también sirven para crear un grupo de captura _numerado_ (número 1, 2, etc.). Un grupo de captura guarda _la parte de la cadena_ que coincide con la parte de la expresión regular entre paréntesis. Puedes referirte al mismo texto tal como fue guardado en un grupo de captura utilizando _referencias previas_, como `\1`, `\2` etc. Por ejemplo, la siguiente expresión regular busca todas las frutas que tengan un par de letras repetido. 


```r
str_view(frutas, "(..)\\1", match = TRUE)
```

<!--html_preserve--><div id="htmlwidget-6b83523733b890d61edc" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-6b83523733b890d61edc">{"x":{"html":"<ul>\n  <li>b<span class='match'>anan<\/span>a<\/li>\n  <li><span class='match'>papa<\/span>ya<\/li>\n  <li><span class='match'>anan<\/span>á<\/li>\n  <li><span class='match'>coco<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

(En breve, también verás cómo esto es útil en conjunto con `str_match()`.)

#### Ejercicios

1.  Describe en palabras con qué coinciden estas expresiones: 

    1. `(.)\1\1`
    1. `"(.)(.)\\2\\1"`
    1. `(..)\1`
    1. `"(.).\\1.\\1"`
    1. `"(.)(.)(.).*\\3\\2\\1"`

1.  Construye una expresión regular que coincida con palabras que: 

    1. Empiecen y terminen con el mismo caracter. 
    
    1. Contengan un par de letras repetido
       (p. ej. "nacional" tiene "na" repetido dos veces.)
    
    1. Contengan una letra repetida en al menos tres lugares
       (p. ej. "característica" tiene tres "a".)

## Herramientas

Ahora que has aprendido los elementos básicos de las expresiones regulares, es tiempo de aprender cómo aplicarlos en problemas reales. En esta sección aprenderás una amplia variedad de funciones de __stringr__ que te permitirán:

* Determinar qué cadenas coinciden con un patrón.
* Encontrar la posición de una coincidencia.
* Extraer el contenido de las coincidencias.
* Remplazar coincidencias con nuevos valores.
* Dividir una cadena de acuerdo a una coincidencia.

Una advertencia antes de continuar: debido a que las expresiones regulares son tan poderosas, es fácil intentar resolver todos los problemas con una sola expresión regular. En palabras de Jamie Zawinski:

> Cuando se enfrentan a un problema, algunas personas piensan “Lo sé, usaré expresiones
> regulares.” Ahora tienen dos problemas. 

Como advertencia, revisa esta expresión regular que chequea si una dirección de correo electrónico es válida:

```
(?:(?:\r\n)?[ \t])*(?:(?:(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t]
)+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:
\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(
?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ 
\t]))*"(?:(?:\r\n)?[ \t])*))*@(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\0
31]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\
](?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+
(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:
(?:\r\n)?[ \t])*))*|(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z
|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)
?[ \t])*)*\<(?:(?:\r\n)?[ \t])*(?:@(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\
r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[
 \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)
?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t]
)*))*(?:,@(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[
 \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*
)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t]
)+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*)
*:(?:(?:\r\n)?[ \t])*)?(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+
|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r
\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:
\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t
]))*"(?:(?:\r\n)?[ \t])*))*@(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031
]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](
?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?
:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?
:\r\n)?[ \t])*))*\>(?:(?:\r\n)?[ \t])*)|(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?
:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?
[ \t]))*"(?:(?:\r\n)?[ \t])*)*:(?:(?:\r\n)?[ \t])*(?:(?:(?:[^()<>@,;:\\".\[\] 
\000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|
\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>
@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"
(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*))*@(?:(?:\r\n)?[ \t]
)*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\
".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?
:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[
\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*|(?:[^()<>@,;:\\".\[\] \000-
\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(
?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*)*\<(?:(?:\r\n)?[ \t])*(?:@(?:[^()<>@,;
:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([
^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\"
.\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\
]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*(?:,@(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\
[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\
r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] 
\000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]
|\\.)*\](?:(?:\r\n)?[ \t])*))*)*:(?:(?:\r\n)?[ \t])*)?(?:[^()<>@,;:\\".\[\] \0
00-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\
.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,
;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?
:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*))*@(?:(?:\r\n)?[ \t])*
(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".
\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[
^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]
]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*\>(?:(?:\r\n)?[ \t])*)(?:,\s*(
?:(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\
".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*)(?:\.(?:(
?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[
\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t
])*))*@(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t
])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?
:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|
\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*|(?:
[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\
]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*)*\<(?:(?:\r\n)
?[ \t])*(?:@(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["
()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)
?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>
@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*(?:,@(?:(?:\r\n)?[
 \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,
;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t]
)*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\
".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*)*:(?:(?:\r\n)?[ \t])*)?
(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".
\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*)(?:\.(?:(?:
\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\[
"()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])
*))*@(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])
+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?:\
.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z
|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*\>(?:(
?:\r\n)?[ \t])*))*)?;\s*)
```

En cierto sentido, este es un ejemplo "patológico" (porque las direcciones de correo electrónico son de verdad sorpresivamente complejas), pero se usa en código real. Mira esta discusión (en inglés) en stackoverflow <http://stackoverflow.com/a/201378> para más detalles. 

No olvides que estás trabajando en un lenguaje de programación y que tienes otras herramientas a tu disposición. En vez de crear una sola expresión regular compleja, usualmente es más fácil crear una serie de expresiones regulares más simples. Si te atascaste tratando de crear una sola expresión regular que resuelva tu problema, da un paso atrás y piensa cómo podrías dividir el problema en partes más pequeñas. Esto te permitirá ir resolviendo cada desafío antes de moverte al siguiente. 

### Detectar coincidencias

Para determinar si un vector de caracteres coincide con un patrón de búsqueda, puedes utilizar `str_detect()`. Este devuelve un vector lógico del mismo largo que el _input_:


```r
x <- c("manzana", "plátano", "pera")
str_detect(x, "e")
#> [1] FALSE FALSE  TRUE
```

Recuerda que cuando usas vectores lógicos en un contexto numérico, `FALSE` (_falso_) se convierte en 0 y `TRUE` (_verdadero_) se convierte en 1. Eso hace que `sum()` (_suma_) y `mean()` (_media_) sean funciones útiles si quieres responder preguntas sobre coincidencias a lo largo de un vector más extenso: 


```r
# ¿Cuántas palabras comunes empiezan con m?
sum(str_detect(palabras, "^m"))
#> [1] 81
# ¿Qué proporción de palabras comunes terminan con una vocal?
mean(str_detect(palabras, "[aáeéiéoéuú]$"))
#> [1] 0.546
```


Cuando tienes condiciones lógicas complejas, (p. ej., encontrar _a_ o _b_ pero no _c_, salvo que _d_) suele ser más fácil combinar múltiples llamadas a `str_detect()` con operadores lógicos, que tratar de crear una sola expresión regular. Por ejemplo, hay dos maneras de buscar todas las palabras que no contengan ninguna vocal: 


```r
# Encuentra todas las palabras que contengan al menos una vocal, y luego niégalo
sin_vocales_1 <- !str_detect(palabras, "[aáeéiíoóuúúü]")
# Encuentra todas las palabras consistentes solo en consonantes (no vocales)
sin_vocales_2 <- str_detect(palabras, "^[^aáeéiíoóuúúü]+$")
identical(sin_vocales_1, sin_vocales_2)
#> [1] TRUE
```

Los resultados son idénticos; sin embargo, creemos que la primera aproximación es significativamente más fácil de entender. Si tu expresión regular se vuelve extremadamente compleja, trata de dividirla en partes más pequeñas, dale un nombre a cada parte y luego combínalas en operaciones lógicas. 

Un uso común de `str_detect()` es para seleccionar elementos que coincidan con un patrón. Puedes hacer eso con subdivisiones lógicas o utilizando `str_subset()`, que es un "envoltorio" (_wrapper_) de esas operaciones. 


```r
palabras[str_detect(palabras, "x$")]
#> [1] "ex"
str_subset(palabras, "x$")
#> [1] "ex"
```

En todo caso, lo más habitual es que tus cadenas de caracteres sean una columna de un _data frame_ y que prefieras utilizar la función `filter()` (_filtrar_):


```r
df <- tibble(
  palabra = palabras, 
  i = seq_along(palabra)
)
df %>% 
  filter(str_detect(palabras, "x$"))
#> # A tibble: 1 x 2
#>   palabra     i
#>   <chr>   <int>
#> 1 ex        338
```


Una varación de `str_detect()` es `str_count()` (_count_ = contar): más que un simple sí o no, te indica cuántas coincidencias hay en una cadena:


```r
x <- c("manzana", "plátano", "pera")
str_count(x, "a")
#> [1] 3 1 1

# En promedio, ¿cuántas vocales hay por palabra?
mean(str_count(palabras, "[aáeéiíoóuúü]"))
#> [1] 2.72
```

Es natural usar `str_count()` junto con `mutate()`:


```r
df %>% 
  mutate(
    vocales = str_count(palabra, "[aáeéiíoóuúü]"),
    consonantes = str_count(palabra, "[^aáeéiíoóuúü]")
  )
#> # A tibble: 1,000 x 4
#>   palabra      i vocales consonantes
#>   <chr>    <int>   <int>       <int>
#> 1 a            1       1           0
#> 2 abril        2       2           3
#> 3 acción       3       3           3
#> 4 acciones     4       4           4
#> 5 acerca       5       3           3
#> 6 actitud      6       3           4
#> # … with 994 more rows
```

Ten en cuenta que las coincidencias nunca se superponen. Por ejemplo, en `"abababa"`, ¿cuántas veces se encontrará una coincidencia con el patrón `"aba"`? Las expresiones regulares dicen que dos, no tres:


```r
str_count("abababa", "aba")
#> [1] 2
str_view_all("abababa", "aba")
```

<!--html_preserve--><div id="htmlwidget-b3f7c917b6c8ff580948" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-b3f7c917b6c8ff580948">{"x":{"html":"<ul>\n  <li><span class='match'>aba<\/span>b<span class='match'>aba<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

Toma nota sobre el uso de `str_view_all()`. Como aprenderás dentro de poco, muchas funciones de __stringr__ vienen en pares: una función trabaja con una sola coincidencia y la otra con todas. La segunda función tendrá el sufijo `_all` (_todas_).

### Ejercicios

1.  Para cada uno de los siguientes desafíos, intenta buscar una solución utilizando tanto una 
    expresión regular simple como una combinación de múltiples llamadas a `str_detect()`.
    
    1.  Encuentra todas las palabras que empiezan o terminan con `y`.
    
    1.  Encuentra todas las palabras que empiezan con una vocal y terminan con una consonante.
    
    1.  ¿Existen palabras que tengan todas las vocales?

1.  ¿Qué palabra tiene el mayor número de vocales? ¿Qué palabra tiene la mayor
    proporción de vocales? (Pista: ¿cuál es el denominador?)

### Extraer coincidencias

Para extraer el texto de una coincidencia utiliza `str_extract()`. Para mostrar cómo funciona, necesitaremos un ejemplo más complicado. Para ello, usaremos una selección y adaptación al español de las oraciones disponibles originalmente en `stringr::sentences` y que puedes encontrar en `datos::oraciones`:


```r
length(oraciones)
#> [1] 50
head(oraciones)
#> [1] "Las casas están construidas de ladrillos de arcilla roja."
#> [2] "La caja fue arrojada al lado del camión estacionado."     
#> [3] "El domingo es la mejor parte de la semana."               
#> [4] "Agrega a la cuenta de la tienda hasta el último centavo." 
#> [5] "Nueve hombres fueron contratados para excavar las ruinas."
#> [6] "Pega la hoja en el fondo azul oscuro."
```

Imagina que quieres encontrar todas las oraciones que tengan el nombre de un color. Primero, creamos un vector con nombres de colores y luego lo convertimos en una sola expresión regular:


```r
colores <- c("rojo", "amarillo", "verde", "azul", "marrón")
coincidencia_color <- str_c(colores, collapse = "|")
coincidencia_color
#> [1] "rojo|amarillo|verde|azul|marrón"
```

Ahora, podemos seleccionar las oraciones que contienen un color y extraer luego el color para saber de cuál se trata:


```r
tiene_color <- str_subset(oraciones, coincidencia_color)
coincidencia <- str_extract(tiene_color, coincidencia_color)
head(coincidencia)
#> [1] "azul"   "azul"   "rojo"   "azul"   "azul"   "marrón"
```

Ten en cuenta que `str_extract()` solo extrae la primera coincidencia. Podemos ver eso de manera sencilla seleccionando primero todas las oraciones que tengan más de una coincidencia:


```r
mas <- oraciones[str_count(oraciones, coincidencia_color) > 1]
str_view_all(mas, coincidencia_color)
```

<!--html_preserve--><div id="htmlwidget-d258b2ee1c304ebe1664" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-d258b2ee1c304ebe1664">{"x":{"html":"<ul>\n  <li>Instalaron <span class='match'>azul<\/span>ejos <span class='match'>verde<\/span>s en la cocina.<\/li>\n  <li>Si ar<span class='match'>rojo<\/span> la taza <span class='match'>azul<\/span> al suelo se romperá.<\/li>\n  <li>Las hojas se vuelven de color <span class='match'>marrón<\/span> y <span class='match'>amarillo<\/span> en el otoño.<\/li>\n  <li>La luz <span class='match'>verde<\/span> en la caja <span class='match'>marrón<\/span> parpadeaba.<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

```r

str_extract(mas, coincidencia_color)
#> [1] "azul"   "rojo"   "marrón" "verde"
```

Este es un patrón de coincidencia común para las funciones de __stringr__, ya que trabajar con una sola coincidencia te permite utilizar estructuras de datos más simples. Para obtener todas las coincidencias, utiliza `str_extract_all()`. Esta función devuelve una lista:


```r
str_extract_all(mas, coincidencia_color)
#> [[1]]
#> [1] "azul"  "verde"
#> 
#> [[2]]
#> [1] "rojo" "azul"
#> 
#> [[3]]
#> [1] "marrón"   "amarillo"
#> 
#> [[4]]
#> [1] "verde"  "marrón"
```

Aprenderás más sobre listas en el capítulo sobre [vectores] y en el sobre [iteración].

Si utilizas `simplify = TRUE` (es decir, _simplificar_ = VERDADERO), `str_extract_all()` devolverá una matriz con las coincidencias más cortas expandidas hasta el largo de las más extensas:


```r
str_extract_all(mas, coincidencia_color, simplify = TRUE)
#>      [,1]     [,2]      
#> [1,] "azul"   "verde"   
#> [2,] "rojo"   "azul"    
#> [3,] "marrón" "amarillo"
#> [4,] "verde"  "marrón"

x <- c("a", "a b", "a b c")
str_extract_all(x, "[a-z]", simplify = TRUE)
#>      [,1] [,2] [,3]
#> [1,] "a"  ""   ""  
#> [2,] "a"  "b"  ""  
#> [3,] "a"  "b"  "c"
```

#### Ejercicios

1.  Te habrás dado cuenta que en el ejemplo anterior la expresión regular
    que utilizamos también devolvió como resultado "arrojo" y "azulejos", 
    que no son nombres de colores. Modifica la expresión regular para 
    resolver ese problema.
    

1.  De `datos::oraciones` extrae:

    1. La primera palabra de cada oración.
    1. Todas las palabras que terminen en `ción`.
    1. Todos los plurales.

### Coincidencias agrupadas

Antes en este capítulo hablamos sobre el uso de paréntesis para aclarar la _precedencia_ y las _referencias previas_ al buscar coincidencias. También puedes utilizar los paréntesis para extraer una coincidencia compleja. Por ejemplo, imagina que quieres extraer los sustantivos de una oración. Como heurística, buscaremos cualquier palabra que venga después de un artículo (_el_, _la_, _un_, _una_, etc.). Definir qué es una palabra en una expresión regular es un poco complicado, así que aquí utilizaremos una aproximación simple: una secuencia de al menos un caracter que no sea un espacio. 


```r
sustantivo <- "(el|la|los|las|lo|un|una|unos|unas) ([^ ]+)"

tiene_sustantivo <- oraciones %>%
  str_subset(sustantivo) %>%
  head(10)
tiene_sustantivo %>% 
  str_extract(sustantivo)
#>  [1] "los de"      "el camión"   "la mejor"    "la cuenta"   "las ruinas."
#>  [6] "la hoja"     "la cocina."  "la taza"     "el tanque."  "el calor"
```

`str_extract()` nos devuelve la coincidencia completa; `str_match()` nos  entrega cada componente. En vez de un vector de caracteres, devuelve una matriz con una columna para la coincidencia completa y una columna para cada grupo:


```r
tiene_sustantivo %>% 
  str_match(sustantivo)
#>       [,1]          [,2]  [,3]     
#>  [1,] "los de"      "los" "de"     
#>  [2,] "el camión"   "el"  "camión" 
#>  [3,] "la mejor"    "la"  "mejor"  
#>  [4,] "la cuenta"   "la"  "cuenta" 
#>  [5,] "las ruinas." "las" "ruinas."
#>  [6,] "la hoja"     "la"  "hoja"   
#>  [7,] "la cocina."  "la"  "cocina."
#>  [8,] "la taza"     "la"  "taza"   
#>  [9,] "el tanque."  "el"  "tanque."
#> [10,] "el calor"    "el"  "calor"
```

(Como era de esperarse, nuestra heurística para detectar sustantivos es pobre, ya que también selecciona adjetivos como "mejor" y preposiciones como "de").

Si tus datos están en un tibble, suele ser más fácil utilizar `tidyr::extract()`. Funciona como `str_match()` pero requiere ponerle un nombre a las coincidencias, las que luego son puestas en columnas nuevas:


```r
tibble(oracion = oraciones) %>% 
  tidyr::extract(
    oracion, c("articulo", "sustantivo"), "(el|la|los|las|un|una|unos|unas) ([^ ]+)", 
    remove = FALSE
  )
#> # A tibble: 50 x 3
#>   oracion                                                   articulo sustantivo
#>   <chr>                                                     <chr>    <chr>     
#> 1 Las casas están construidas de ladrillos de arcilla roja. los      de        
#> 2 La caja fue arrojada al lado del camión estacionado.      el       camión    
#> 3 El domingo es la mejor parte de la semana.                la       mejor     
#> 4 Agrega a la cuenta de la tienda hasta el último centavo.  la       cuenta    
#> 5 Nueve hombres fueron contratados para excavar las ruinas. las      ruinas.   
#> 6 Pega la hoja en el fondo azul oscuro.                     la       hoja      
#> # … with 44 more rows
```

Al igual que con `str_extract()`, si quieres todas las coincidencias para cada cadena, tienes que utilizar `str_match_all()`.

#### Ejercicios

1. Busca en `datos::oraciones` todas las palabras que vengan después de un "número", como "un(o|a)", "dos", "tres", etc.
   Extrae tanto el número como la palabra.

1. En español a veces se utiliza el guión para unir adjetivos, establecer relaciones entre conceptos o para unir gentilicios (p. ej., _teórico-práctico_, _precio-calidad_, _franco-porteña_). ¿Cómo podrías encontrar esas palabras y separar lo que viene antes y después del guión?

### Remplazar coincidencias

`str_replace()` y `str_replace_all()` te permiten remplazar coincidencias en una nueva cadena. Su uso más simple es para remplazar un patrón con una cadena fija: 


```r
x <- c("manzana", "pera", "banana")
str_replace(x, "[aeiou]", "-")
#> [1] "m-nzana" "p-ra"    "b-nana"
str_replace_all(x, "[aeiou]", "-")
#> [1] "m-nz-n-" "p-r-"    "b-n-n-"
```

Con `str_replace_all()` puedes realizar múltiples remplazos a través de un vector cuyos elementos tiene nombre (_named vector_):


```r
x <- c("1 casa", "2 autos", "3 personas")
str_replace_all(x, c("1" = "una", "2" = "dos", "3" = "tres"))
#> [1] "una casa"      "dos autos"     "tres personas"
```

En vez de hacer remplazos con una cadena fija, puedes utilizar _referencias previas_ para insertar componentes de la coincidencia. En el siguiente código invertimos el orden de la segunda y la tercera palabra: 


```r
oraciones %>% 
  str_replace("([^ ]+) ([^ ]+) ([^ ]+)", "\\1 \\3 \\2") %>% 
  head(5)
#> [1] "Las están casas construidas de ladrillos de arcilla roja."
#> [2] "La fue caja arrojada al lado del camión estacionado."     
#> [3] "El es domingo la mejor parte de la semana."               
#> [4] "Agrega la a cuenta de la tienda hasta el último centavo." 
#> [5] "Nueve fueron hombres contratados para excavar las ruinas."
```

#### Ejercicios

1.   Remplaza en una cadena todas las barras por barras invertidas.

1.   Implementa una versón simple de `str_to_lower()` (_a minúsculas_) usando `replace_all()`.

1.   Cambia la primera y la última letra en `palabras`. ¿Cuáles de esas cadenas
     siguen siendo palabras?

### Divisiones

Usa `str_split()` para dividir una cadena en partes. Por ejemplo, podemos dividir `oraciones` en palabras:


```r
oraciones %>%
  head(5) %>% 
  str_split(" ")
#> [[1]]
#> [1] "Las"         "casas"       "están"       "construidas" "de"         
#> [6] "ladrillos"   "de"          "arcilla"     "roja."      
#> 
#> [[2]]
#> [1] "La"           "caja"         "fue"          "arrojada"     "al"          
#> [6] "lado"         "del"          "camión"       "estacionado."
#> 
#> [[3]]
#> [1] "El"      "domingo" "es"      "la"      "mejor"   "parte"   "de"     
#> [8] "la"      "semana."
#> 
#> [[4]]
#>  [1] "Agrega"   "a"        "la"       "cuenta"   "de"       "la"      
#>  [7] "tienda"   "hasta"    "el"       "último"   "centavo."
#> 
#> [[5]]
#> [1] "Nueve"       "hombres"     "fueron"      "contratados" "para"       
#> [6] "excavar"     "las"         "ruinas."
```

Como cada componente podría tener un número diferente de elementos, esto devuelve una lista. Si estás trabajando con vectores de extensión 1, lo más fácil es extraer el primer elemento de la lista:


```r
"a|b|c|d" %>% 
  str_split("\\|") %>% 
  .[[1]]
#> [1] "a" "b" "c" "d"
```

Otra opción es, al igual que con otras funciones de __stringr__ que devuelven una lista, utilizar `simplify = TRUE` para obtener una matriz:


```r
oraciones %>%
  head(5) %>% 
  str_split(" ", simplify = TRUE)
#>      [,1]     [,2]      [,3]     [,4]          [,5]    [,6]        [,7]    
#> [1,] "Las"    "casas"   "están"  "construidas" "de"    "ladrillos" "de"    
#> [2,] "La"     "caja"    "fue"    "arrojada"    "al"    "lado"      "del"   
#> [3,] "El"     "domingo" "es"     "la"          "mejor" "parte"     "de"    
#> [4,] "Agrega" "a"       "la"     "cuenta"      "de"    "la"        "tienda"
#> [5,] "Nueve"  "hombres" "fueron" "contratados" "para"  "excavar"   "las"   
#>      [,8]      [,9]           [,10]    [,11]     
#> [1,] "arcilla" "roja."        ""       ""        
#> [2,] "camión"  "estacionado." ""       ""        
#> [3,] "la"      "semana."      ""       ""        
#> [4,] "hasta"   "el"           "último" "centavo."
#> [5,] "ruinas." ""             ""       ""
```

También puedes indicar un número máximo de elementos:


```r
campos <- c("Nombre: Hadley", "País: NZ", "Edad: 35")
campos %>% str_split(": ", n = 2, simplify = TRUE)
#>      [,1]     [,2]    
#> [1,] "Nombre" "Hadley"
#> [2,] "País"   "NZ"    
#> [3,] "Edad"   "35"
```

En vez de dividir una cadena según patrones, puedes hacerlo según caracter, línea, oración o palabra. Para ello, puedes utilizar la función `boundary()` (_límite_). En el siguiente ejemplo la división se hace por palabra (_word_):


```r
x <- "Esta es una oración. Esta es otra oración."
str_view_all(x, boundary("word"))
```

<!--html_preserve--><div id="htmlwidget-b8f31ebacaee3527bb86" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-b8f31ebacaee3527bb86">{"x":{"html":"<ul>\n  <li><span class='match'>Esta<\/span> <span class='match'>es<\/span> <span class='match'>una<\/span> <span class='match'>oración<\/span>. <span class='match'>Esta<\/span> <span class='match'>es<\/span> <span class='match'>otra<\/span> <span class='match'>oración<\/span>.<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

```r

str_split(x, " ")[[1]]
#> [1] "Esta"     "es"       "una"      "oración." "Esta"     "es"       "otra"    
#> [8] "oración."
str_split(x, boundary("word"))[[1]]
#> [1] "Esta"    "es"      "una"     "oración" "Esta"    "es"      "otra"   
#> [8] "oración"
```

#### Ejercicios

1.  Divide una cadena como `"manzanas, peras y bananas"` en elementos
    individuales.
    
1.  ¿Por qué es mejor dividir utilizando `boundary("word")` en vez de `" "`?

1.  ¿Qué pasa si dividimos con una cadena vacía (`""`)? Experimenta y
    luego lee la documentación

### Buscar coincidencias

`str_locate()` y `str_locate_all()` te indican la posición inicial y final de una coincidencia. Son particularmente útiles cuando ninguna otra función hace exactamente lo que quieres. Puedes utilizar `str_locate()` para encontrar los patrones de coincidencia y `str_sub()` para extraerlos y/o modificarlos.

## Otro tipo de patrones

Cuando utilizas un patrón que es una cadena, este automáticamente es encapsulado en la función `regex()` (_regex_ es la forma abreviada de _regular expression_, es decir, expresión regular):


```r
# La manera regular en que escribimos el patrón
str_view(frutas, "nana")
# Es un atajo de
str_view(frutas, regex("nana"))
```

Puedes utilizar los otros argumentos de `regex()` para controlar los detalles de la coincidencia:

*   `ignore_case = TRUE` permite que la búsqueda coincida tanto con caracteres en mayúscula
    como en minúscula. Este argumento siempre utiliza los parámetros de tu _locale_.
    
    
    ```r
    bananas <- c("banana", "Banana", "BANANA")
    str_view(bananas, "banana")
    ```
    
    <!--html_preserve--><div id="htmlwidget-b25b670b028f478bf741" style="width:960px;height:100%;" class="str_view html-widget"></div>
    <script type="application/json" data-for="htmlwidget-b25b670b028f478bf741">{"x":{"html":"<ul>\n  <li><span class='match'>banana<\/span><\/li>\n  <li>Banana<\/li>\n  <li>BANANA<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->
    
    ```r
    str_view(bananas, regex("banana", ignore_case = TRUE))
    ```
    
    <!--html_preserve--><div id="htmlwidget-46d1193f7ba074d981c8" style="width:960px;height:100%;" class="str_view html-widget"></div>
    <script type="application/json" data-for="htmlwidget-46d1193f7ba074d981c8">{"x":{"html":"<ul>\n  <li><span class='match'>banana<\/span><\/li>\n  <li><span class='match'>Banana<\/span><\/li>\n  <li><span class='match'>BANANA<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->
    
*   `multiline = TRUE` permite que `^` y `$` coincidan con el inicio y fin de cada
    línea, en vez del inicio y fin de la cadena completa.
    
    
    ```r
    x <- "Línea 1\nLínea 2\nLínea 3"
    str_extract_all(x, "^Línea")[[1]]
    #> [1] "Línea"
    str_extract_all(x, regex("^Línea", multiline = TRUE))[[1]]
    #> [1] "Línea" "Línea" "Línea"
    ```
    
*   `comments = TRUE` te permite utilizar comentarios y espacios en blanco
    para hacer más entendibles las expresiones regulares complejas.
    Los espacios son ignorados, al igual que todo lo que está después de 
    `#`. Para coincidir un espacio de manera literal, tendrías que 
    "escaparlo: `"\\ "`.
    
    
    ```r
    telefono <- regex("
      \\(?     # paréntesis inicial opcional
      (\\d{3}) # código de área
      [) -]?   # paréntesis, espacio o guión inicial opcional
      (\\d{3}) # otros tres números
      [ -]?    # espacio o guión opcional
      (\\d{3}) # otros tres números
      ", comments = TRUE)
    
    str_match("514-791-8141", telefono)
    #>      [,1]          [,2]  [,3]  [,4] 
    #> [1,] "514-791-814" "514" "791" "814"
    ```

*   `dotall = TRUE` permite que `.` coincida con todo, incluidos los saltos de línea (`\n`).

Existen otras tres funciones que puedes utilizar en vez de `regex()`:

*   `fixed()`: busca una coincidencia exacta de la secuencia de bytes especificada.
    Ignora todas las expresiones regulares especiales y opera a un nivel muy bajo. 
    Esto te permite evitar formas de "escapado" complejas y puede ser mucho
    más rápida que las expresiones regulares. La comparación utilizando
    `microbenchmark` muestra que `fixed()` es casi dos veces más rápida.
  
    
    ```r
    #install.packages(microbenchmark)
    microbenchmark::microbenchmark(
      fixed = str_detect(oraciones, fixed("la")),
      regex = str_detect(oraciones, "la"),
      times = 20
    )
    #> Unit: microseconds
    #>   expr  min   lq mean median   uq   max neval
    #>  fixed 20.2 21.0 50.0   22.7 25.7 538.1    20
    #>  regex 31.2 33.6 42.5   36.3 45.1  77.3    20
    ```
    
    IMPORTANTE: ten precaución al utilizar `fixed()` con datos que no estén en inglés. Puede causar problemas porque
    muchas veces existen múltiples formas de representar un mismo caracter. Por 
    ejemplo, hay dos formas de difinir "á": como un solo caracter o como una
    "a" con un acento:
    
    
    ```r
    a1 <- "\u00e1"
    a2 <- "a\u0301"
    c(a1, a2)
    #> [1] "á" "á"
    a1 == a2
    #> [1] FALSE
    ```

    Ambas se _renderean_ de manera idéntica, pero como están definidas de manera distinta, 
    `fixed()` no encuentra una coincidencia. En su lugar, puedes utilizar `coll()`, que definiremos a continuación, ya que respeta las reglas humanas de comparación de caracteres:

    
    ```r
    str_detect(a1, fixed(a2))
    #> [1] FALSE
    str_detect(a1, coll(a2))
    #> [1] TRUE
    ```
    
*   `coll()`: compara cadenas usando reglas de secuenciación (_collation_) estándar. Esto es 
    útil para buscar coincidencias que sean insensibles a mayúsculas y minúsculas. Ten en cuenta que `coll()` incluye un parámetro para el _locale_, 
    Lamentablemente, se utilizan diferentes reglas en diferentes partes del mundo. 

    
    ```r
    # Esto quiere decir que también tienes que prestar atención a esas 
    # diferencias al buscar coincidencias insensibles a mayúsculas y
    # minúsculas
    i <- c("I", "İ", "i", "ı")
    i
    #> [1] "I" "İ" "i" "ı"
    
    str_subset(i, coll("i", ignore_case = TRUE))
    #> [1] "I" "i"
    str_subset(i, coll("i", ignore_case = TRUE, locale = "tr"))
    #> [1] "İ" "i"
    ```
    
    Tanto `fixed()` como `regex()` tienen argumentos para ignorar la
    diferencia entre mayúsculas y minúsculas (`ignore_case`); sin embargo, 
    no te permiten elegir tu _locale_: siempre utilizan el que está definido
    por defecto. Puedes ver cuál se está usando con el siguiente código. Pronto
    hablaremos más sobre el paquete __stringi__. 
    
    
    ```r
    stringi::stri_locale_info()
    #> $Language
    #> [1] "en"
    #> 
    #> $Country
    #> [1] "US"
    #> 
    #> $Variant
    #> [1] ""
    #> 
    #> $Name
    #> [1] "en_US"
    ```
    
    Una desventaja de `coll()` es la velocidad. Debido a que las reglas para
    reconocer qué caracteres son iguales suelen ser complicadas, `coll()` es
    relativamente más lenta al compararla con `regex()` y `fixed()`.

*   Como viste con `str_split()`, puedes utilizar `boundary()` para
coincidir límites. 
    También puedes utilizarla con otras funciones:
    
    
    ```r
    x <- "Esta es una oración."
    str_view_all(x, boundary("word"))
    ```
    
    <!--html_preserve--><div id="htmlwidget-382a200f56fb8e6a1fd3" style="width:960px;height:100%;" class="str_view html-widget"></div>
    <script type="application/json" data-for="htmlwidget-382a200f56fb8e6a1fd3">{"x":{"html":"<ul>\n  <li><span class='match'>Esta<\/span> <span class='match'>es<\/span> <span class='match'>una<\/span> <span class='match'>oración<\/span>.<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->
    
    ```r
    str_extract_all(x, boundary("word"))
    #> [[1]]
    #> [1] "Esta"    "es"      "una"     "oración"
    ```

### Ejercicios

1.  ¿Cómo buscarías todas las cadenas que contienen `\` con `regex()` vs.
    con `fixed()`?

1.  ¿Cuáles son las cinco palabras más comunes en `oraciones`?

## Otros usos de las expresiones regulares. 

Existen dos funciones útiles en R base que también utilizan expresiones regulares: 

*   `apropos()` busca todos los objetos disponibles en el ambiente global
(_global environment_). Esto
    es útil si no recuerdas bien el nombre de una función.
    
    
    ```r
    apropos("replace")
    #> [1] "%+replace%"       "replace"          "replace_na"       "setReplaceMethod"
    #> [5] "str_replace"      "str_replace_all"  "str_replace_na"   "theme_replace"
    ```
    
*   `dir()` entrega una lista con todos los archivos en un directorio. El argumento
    `pattern` recibe una expresión regular y retorna solo los 
    nombres de archivos que coinciden con ese patrón. 
    Por ejemplo, puedes encontrar todos los archivos de R Markdown en el 
    directorio actual con: 
    
    
    ```r
    head(dir(pattern = "\\.Rmd$"))
    #> [1] "01-intro.Rmd"            "02-explore.Rmd"         
    #> [3] "03-visualize.Rmd"        "04-workflow-basics.Rmd" 
    #> [5] "05-transform.Rmd"        "06-workflow-scripts.Rmd"
    ```
    
    (Si te resulta más cómodo trabajar con "globs", es decir, especificar 
    los nombres de archivo utilizando comodines, como en `*.Rmd`, puedes
    convertirlos a expresiones regulares con la función `glob2rx()`)

## stringi

__stringr__ está construido sobre la base del paquete __stringi__. __stringr__ es útil cuando estás aprendiendo, ya que presenta un set mínimo de funciones, que han sido elegidas cuidadosamente para manejar las funciones de manipulación de cadenas más comunes. __stringi__, por su parte, está diseñado para ser comprehensivo. Contiene casi todas las funciones que podrías necesitar: __stringi__ tiene 244 funciones, frente a las 49 de __stringr__.

Si en algún momento te encuentras en dificultades para hacer algo en __stringr__ , vale la pena darle una mirada a __stringi__. Ambos paquetes funcionan de manera muy similar, por lo que deberías poder traducir tu conocimiento sobre __stringr__ de manera natural. La principal diferencia es el prefijo: `str_` vs. `stri_`.

### Ejercicios

1.  Busca la función de __stringi__ que:

    1. Cuenta el número de palabras.
    1. Busca cadenas duplicadas.
    1. Genera texto aleatorio.

1.  ¿Cómo puedes controlar qué lengua usa `stri_sort()` para ordenar?
