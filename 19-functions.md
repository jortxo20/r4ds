# Funciones

## Introducción

Una de las mejores maneras de lograr tener mayor alcance haciendo ciencia de datos es escribir funciones. Las funciones te permitirán automatizar algunas tareas comunes de una forma más poderosa y general que copiar-y-pegar. Escribir funciones tiene tres grandes ventajas sobre copiar-y-pegar:

1. Puedes dar a la función un nombre evocador que hará tu código más fácil de entender.

1. A medida que cambien los requerimientos, solo necesitarás cambiar tu código en un solo lugar, en vez de en varios lugares.

1. Eliminas las probabilidades  de errores accidentales cuando copias y pegas (por ej., al actualizar el nombre de una variable en un lugar, pero no en otro).

Escribir funciones es un viaje de toda una vida. Incluso después de usar R por varios años, seguimos aprendiendo nuevas técnicas y mejores formas de abordar viejos problemas. El objetivo de este capítulo no es enseñarte cada detalle esotérico de las funciones, sino introducirte en este tema con consejos pragmáticos que puedas aplicar inmediatamente.

Además de consejos prácticos para escribir funciones, este capítulo también te entregará consejos de estilo para tu código. Escribir código con buen estilo es como utilizar la puntuación correcta. Puedesmanejartesinella, pero utilizarla hace las cosas más fáciles de leer. Al igual que con los estilos de puntuación, hay muchas posibles variaciones. Si bien aquí presentamos el estilo que nosotros usamos en nuestro código, lo más importante es que seas consistente.

### Prerrequisitos

El foco de este capítulo es escribir funciones en R base, por lo que no necesitarás ningún paquete extra.

## ¿Cuándo deberías escribir una función?

Deberías considerar escribir una función cuando has copiado y pegado un bloque de código más de dos veces (es decir, ahora tienes tres copias del mismo). Mira, por ejemplo, el siguiente código. ¿Qué es lo que hace?


```r
df <- tibble::tibble(
 a = rnorm(10),
 b = rnorm(10),
 c = rnorm(10),
 d = rnorm(10)
)

df$a <- (df$a - min(df$a, na.rm = TRUE)) /
 (max(df$a, na.rm = TRUE) - min(df$a, na.rm = TRUE))
df$b <- (df$b - min(df$b, na.rm = TRUE)) /
 (max(df$b, na.rm = TRUE) - min(df$a, na.rm = TRUE))
df$c <- (df$c - min(df$c, na.rm = TRUE)) /
 (max(df$c, na.rm = TRUE) - min(df$c, na.rm = TRUE))
df$d <- (df$d - min(df$d, na.rm = TRUE)) /
 (max(df$d, na.rm = TRUE) - min(df$d, na.rm = TRUE))
```

Es posible que hayas podido descifrar que lo que hace es reescalar cada columna para que tenga un rango de 0 a 1. Pero ¿has encontrado el error? Ocurrió copiando-y-pegando el código para `df$b`: Hemos olvidado de cambiar `a` a `b`. Extraer el código repetido en una función es una buena idea, ya que previene que cometas errores como este.

Para escribir una función, lo primero que necesitas hacer es analizar el código. ¿Cuantos inputs tiene?



```r
(df$a - min(df$a, na.rm = TRUE)) /
 (max(df$a, na.rm = TRUE) - min(df$a, na.rm = TRUE))
```

Este código tiene un solo input `df$a`. (Si te sorprende que `TRUE` no es un input, puedes explorar el ejercicio de abajo). Para hacer los inputs más claros, es buena idea reescribir el código usando variables temporales con nombres generales. Acá el código requiere un solo vector numérico, por lo que lo llamaremos `x`:


```r
x <- df$a
(x - min(x, na.rm = TRUE)) / (max(x, na.rm = TRUE) - min(x, na.rm = TRUE))
#>  [1] 0.289 0.751 0.000 0.678 0.853 1.000 0.172 0.611 0.612 0.601
```

Hay algo de duplicación en este código. Estamos computando el rango de datos tres veces, así que tiene sentido hacerlo en un solo paso:


```r
rng <- range(x, na.rm = TRUE)
(x - rng[1]) / (rng[2] - rng[1])
#>  [1] 0.289 0.751 0.000 0.678 0.853 1.000 0.172 0.611 0.612 0.601
```

Sacar cálculos intermedios en variables nombradas es una buena práctica porque deja más claro qué es lo que está haciendo el código. Ahora que hemos simplificado el código y chequeado de que aún funciona, podemos convertirlo en una función:


```r
rescale01 <- function(x) {
 rng <- range(x, na.rm = TRUE)
 (x - rng[1]) / (rng[2] - rng[1])
}
rescale01(c(0, 5, 10))
#> [1] 0.0 0.5 1.0
```

Hay tres pasos claves para crear una función nueva:

1. Necesitas elegir un __nombre__ para la función. Aquí hemos usado `rescale01`, ya que esta función reescala (_rescale_, en inglés) un vector para que se ubique entre 0 y 1.

1. Listar los inputs, o __argumentos__, de la función dentro de `function`.
Aquí solo tenemos un argumento. Si tenemos más, la llamada se vería como `function(x, y, z)`.

1. Situar el código que has creado en el __cuerpo__ de una función, un
bloque de `{`  que sigue inmediatamente a `function(...)`.

Ten en cuenta el proceso general: solo hemos creado la función después de darnos cuenta cómo funciona con un input simple. Es más fácil empezar con código que funciona y luego convertirlo en una función; es más difícil crear la función y luego tratar que funcione.

En este punto es una buena idea chequear tu función con algunos inputs diferentes:


```r
rescale01(c(-10, 0, 10))
#> [1] 0.0 0.5 1.0
rescale01(c(1, 2, 3, NA, 5))
#> [1] 0.00 0.25 0.50   NA 1.00
```

A medida que escribas más y más funciones eventualmente querrás convertir estos tests interactivos informales en tests formales y automatizados. Este proceso se llama _pruebas unitarias_ (_unit testing_). Desafortunadamente, este tema está más allá del alcance de este libro, pero puedes aprender sobre él en <https://r-pkgs.org/tests.html>.

Podemos simplificar el ejemplo original ahora que tenemos una función:


```r
df$a <- rescale01(df$a)
df$b <- rescale01(df$b)
df$c <- rescale01(df$c)
df$d <- rescale01(df$d)
```

Comparado al original, este código es fácil de entender y hemos eliminado erorres del tipo copiar-y-pegar. Existe aún un poco de duplicación, ya que estamos relizando lo mismo en diferentes columnas. Aprenderemos cómo eliminar esta duplicación en el capítulo sobre [iteración], una vez que hayas aprendido más sobre las estructuras de R en el capítulo sobre [vectores].

Otra ventaja de las funcioens es que si nuestros requerimientos cambian, solo necesitamos hacer modificaciones en un solo lugar. Por ejemplo, podríamos descubrir que algunas de nuestras variables incluyen valores infinitos, lo que hará que `rescale01()` falle:



```r
x <- c(1:10, Inf)
rescale01(x)
#>  [1]   0   0   0   0   0   0   0   0   0   0 NaN
```

Debido a que hemos extraído el código en una función, solo necesitamos corregirlo en un lugar:


```r
rescale01 <- function(x) {
 rng <- range(x, na.rm = TRUE, finite = TRUE)
 (x - rng[1]) / (rng[2] - rng[1])
}
rescale01(x)
#>  [1] 0.000 0.111 0.222 0.333 0.444 0.556 0.667 0.778 0.889 1.000   Inf
```

Esta es una importante parte del principio de "no repetirse a uno mismo" (conocido como DRY: del inglés "**D**o not **R**epeat **Y**ourself"). Cuanta más repetición tengas en tu código, más lugares tendrás que recordar de actualizar cuando las cosas cambien (¡y eso siempre sucede!), y es más probable que crees errores (_bugs_) a lo largo del tiempo.

### Ejercicios

1. ¿Por qué `TRUE` no es un parámetro para `rescale01()`? ¿Qué pasaría si `x` está contenido en un valor único perdido y `na.rm` fuese `FALSE`?

1. En la segunda variante de `rescale01()`, los valores infinitos se dejan sin cambio. Reescribe `rescale01()` para que `-Inf` sea convertido a 0, e `Inf` a 1.

1. Practica convertir los siguientes fragmentos de código en funciones. Piensa en lo que hace cada función. ¿Cómo la llamarías? ¿Cuántos argumentos necesita? ¿Puedes reescribirla para ser más expresiva o con menos duplicación de código?

         
         ```r
         mean(is.na(x))
                 
         x / sum(x, na.rm = TRUE)
                 
         sd(x, na.rm = TRUE) / mean(x, na.rm = TRUE)
         ```

4. Escribe tus propias funciones para computar la varianza y la inclinación de un vector numérico. La varianza se define como
    $$
    \mathrm{Var}(x) = \frac{1}{n - 1} \sum_{i=1}^n (x_i - \bar{x}) ^2 \text{,}
    $$
    donde $\bar{x} = (\sum_i^n x_i) / n$ es la media de la muestra.
    La inclinación se define como
    $$
    \mathrm{Skew}(x) = \frac{\frac{1}{n-2}\left(\sum_{i=1}^n(x_i - \bar x)^3\right)}{\mathrm{Var}(x)^{3/2}} \text{.}
    $$


5. Escribe `both_na()` (*ambos_na()*), una función que toma dos vectores de la misma longitud y retorna el número de posiciones que tienen `NA` en ambos vectores.

6. ¿Qué hacen las siguientes funciones? ¿Por qué son tan útiles pese a ser tan cortas?

         
         ```r
         is_directory <- function(x) file.info(x)$isdir
         is_readable <- function(x) file.access(x, 4) == 0
         ```

7. Lee la letra completa de "Pequeño Conejito Foo Foo". Como ves, hay mucha duplicación en la letra de la canción. 
 Extiende el ejemplo inicial de pipes para recrear la canción completa usando funciones para reducir la duplicación.


## Las funciones son para los seres humanos y para las computadoras

Es importante recordar que las funciones no son solo para las computadoras, sino también para los seres humanos. A R no le importa el nombre de tu función ni los comentarios que tiene, pero estos sí serán importantes para los seres humanos que la lean. En esta sección se discutirán algunas cosas que debes tener en mente a la hora de escribir funciones entendibles para otras personas.

El nombre de una función es importante. Idealmente, debería ser corto, pero que evoque claramente lo que la función hace. ¡Eso es difícil! Es mejor que sea claro a que sea corto, considerando que la función de autocompletar de RStudio hace más fácil tipear nombres largos. 

Generalmente, los nombres de las funciones deberían ser verbos y los argumentos sustantivos. Hay algunas excepciones: usar un sustantivo está bien si la función computa el valor de un sustantivo muy conocido (por ejemplo, `mean()` — (del inglés _media_) es mejor que `compute_mean()`— (del inglés _computar media_)), o accede a alguna propiedad del objeto (por ejemplo,  `coef()`— (abreviatura en inglés de _coeficientes_) es mejor que `get_coefficients()`— (en inglés, _obtener coeficientes_)). Una buena señal de que un sustantivo puede ser una mejor elección es analizar si estás usando un verbo muy amplio como “obtener”, “computar”, “calcular” o “determinar”. Utiliza tu criterio y no tengas miedo de renombrar tu función si encuentras un nombre mejor más tarde.


```r
# Muy corto
f()

# No es un verbo y es poco descriptivo
my_awesome_function()

# Largos, pero descriptivos
imputar_faltantes()
colapsar_anios()
```

Si el nombre de tu función está compuesto por múltiples palabras, te recomendamos usar el formato _serpiente_, o "snake\_case”, en el que cada palabra en minúscula está separada por un guión bajo. Otra alternativa popular es es camelCase, o formato camello. No importa realmente cuál elijas, lo importante es que seas consistente: elije uno o el otro y quédate con él. R mismo no es muy consistente, pero no hay nada que puedas hacer al respecto. Asegúrate de no caer en la misma trampa haciendo tu código lo más consistente posible.


```r
# ¡Nunca hagas esto!
col_mins <- function(x, y) {}
rowMaxes <- function(y, x) {}
```

Si tienes una familia de funciones que hacen cosas similares, asegúrate de que tengan nombres y argumentos consistentes. Utiliza un prefijo común para indicar que están conectadas. Eso es mejor que usar un sufijo común, ya que el autocompletado te permite escribir el prefijo y ver todos los otros miembros de la familia.


```r
# Bien
input_select()
input_checkbox()
input_text()

# No tan bien
select_input()
checkbox_input()
text_input()
```

Un buen ejemplo de este diseño es el paquete **stringr**: si no recuerdas exactamente qué función necesitas, puedes escribir `str_`y el autocompletado te ayudará a refrescar tu memoria.
Siempre que sea posible, evita sobrescribir funciones y variables ya existentes. No siempre es posible hacer esto, ya que hay un montón de nombres buenos que ya han sido utilizados por otros paquetes. De todas maneras, evitar el uso de los nombres más comunes de R base ahorrará confusiones.


```r
# ¡No hagas esto!
T <- FALSE
c <- 10
mean <- function(x) sum(x)
```

Usa comentarios, esto es, líneas que comienzan con `#`, para explicar el “porqué” de tu código. En general deberías evitar comentarios que expliquen el “qué” y el “cómo”. Si no se entiende qué es lo que hace el código leyéndolo, deberías pensar cómo reescribirlo de manera que sea más claro. ¿Necesitas agregar algunas variables intermedias con nombres útiles? ¿Deberías dividir una función larga en subcomponentes para que pueda ser nombrada? Sin embargo, tu código nunca podrá capturar la razón detrás de tus decisiones: ¿Por qué elegiste este enfoque frente a otras alternativas? ¿Qué otra cosa probaste que no funcionó? Es una gran idea capturar este tipo de pensamientos en un comentario.

Otro uso importante de los comentarios es para dividir tu archivo en partes, de modo que resulte más fácil de leer. Utiliza líneas largas de`-` y `=` para que resulte más fácil detectar los fragmentos.



```r
# Cargar los datos --------------------------------------

# Graficar los datos --------------------------------------
```

RStudio proporciona un método abreviado de teclado para crear estos encabezados (Cmd/Ctrl + Shift + R), y los mostrará en el menú desplegable de navegación de código en la parte inferior izquierda del editor:

<img src="screenshots/rstudio-nav.png" width="125" style="display: block; margin: auto;" />

### Ejercicios

1. Lee el código fuente para cada una de las siguientes tres funciones, interpreta qué hacen y luego propone nombres mejores.

         
         ```r
         f1 <- function(string, prefix) {
         substr(string, 1, nchar(prefix)) == prefix
         }
         f2 <- function(x) {
         if (length(x) <= 1) return(NULL)
         x[-length(x)]
         }
         f3 <- function(x, y) {
         rep(y, length.out = length(x))
         }
         ```

2. Toma una función que hayas escrito recientemente y tómate 5 minutos para pensar un mejor nombre para la función y para sus argumentos.

3. Compara y contrasta `rnorm()` y `MASS::mvrnorm()`. ¿Cómo podrías hacerlas más consistentes?

4. Argumenta por qué `norm_r()`,`norm_d()`, etc. sería una mejor opción que `rnorm()`, `dnorm()`. Argumenta lo contrario.

## Ejecución condicional

Una sentencia `if` (_si_) te permite ejecutar un código condicional. Por ejemplo:


```r
if (condition) {
 # el código que se ejecuta cuando la condición es verdadera (TRUE)
} else {
 # el código que se ejecuta cuando la condición es falsa (FALSE)
}
```

Para obtener ayuda acerca de `if` necesitas ponerlo entre acentos graves: `` ?`if` ``. La ayuda no es especialmente útil si aún no tienens tanta experiencia programando. ¡Pero al menos puedes saber cómo llegar a ella!

Aquí se presenta una función simple que utiliza una sentencia `if`. El objetivo de esta función es devolver un vector lógico que describa si cada elemento de un vector tiene nombre (_name_).


```r
tiene_nombre <- function(x) {
 nms <- names(x)
 if (is.null(nms)) {
 rep(FALSE, length(x))
 } else {
 !is.na(nms) & nms != ""
 }
}
```

Esta función aprovecha la regla de retorno estándar: una función devuelve el último valor que calculó. Este es uno de los dos usos de la declaración `if`.

### Condiciones

La condición debe evaluar como `TRUE` o `FALSE`. Si es un vector, recibirás un mensaje de advertencia; si es una `NA`, obtendrás un error. Ten cuidado con estos mensajes en tu propio código:


```r
if (c(TRUE, FALSE)) {}
#> Warning in if (c(TRUE, FALSE)) {: the condition has length > 1 and only the
#> first element will be used
#> NULL

if (NA) {}
#> Error in if (NA) {: missing value where TRUE/FALSE needed
```

Puedes usar `||` (o) y `&&`(y) para combinar múltiples expresiones lógicas. Estos operadores hacen "cortocircuito": tan pronto como `||` vea el primer `TRUE` devolverá `TRUE` sin calcular nada más. Tan pronto como && vea el primer `FALSE`, devolverá `FALSE`. Nunca debes usar`|`o `&` en una sentencia `if`: estas son operaciones vectorizadas que se aplican a valores múltiples (es por eso que las usas en `filter()`). Si tienes un vector lógico, puedes utilizar `any()` (_cualquier_) o `all()` (_todo_) para juntarlo en un único valor.

Ten cuidado al comprobar igualdad.`==` está vectorizado, lo que significa que es fácil obtener más de un output. Comprueba si la longitud ya es 1 y junta con `all()` o `any()`, o usa la función no vectorizada `identical()` (_indéntico_). `identical()` es una función muy estricta: siempre devuelve un solo `TRUE` o un solo `FALSE`, y no fuerza tipos de estructuras de datos. Esto significa que debes tener cuidado al comparar enteros y dobles:


```r
identical(0L, 0)
#> [1] FALSE
```

También hay que tener cuidado con los números de punto flotante:


```r
x <- sqrt(2) ^ 2
x
#> [1] 2
x == 2
#> [1] FALSE
x - 2
#> [1] 4.44e-16
```

En su lugar, utiliza `dplyr::near()` para comparaciones, como se describe en la sección sobre [comparaciones].

Y recuerda, ¡`x == NA` no hace nada útil!

### Condiciones múltiples

Puedes encadenar múltiples sentencias _if_ juntas:


```r
if (this) {
 # haz aquello
} else if (that) {
 # haz otra cosa
} else {
 #
}
```

Pero si terminas con una larga serie de sentencias `if` encadenadas, deberías considerar reescribir el código. Una técnica útil es la función `switch()` . Esta te permite evaluar el código seleccionado según la posición o el nombre.


```
#> function(x, y, op) {
#>  switch(op,
#>  plus = x + y,
#>  minus = x - y,
#>  times = x * y,
#>  divide = x / y,
#>  stop("¡operación desconocida!")
#>  )
#> }
```

Otra función útil que a menudo puede eliminar largas cadenas de sentencias `if` es `cut()` (_cortar_). Esta es utilizada para convertir en categóricas variables que son continuas.

### Estilo del código

Tanto `if` como `function` deberían ir (casi) siempre entre llaves (`{}`) y el contenido debería tener una sangría de dos espacios. Esto hace que sea más fácil distinguir la jerarquía dentro de tu código al mirar el margen izquierdo.

La llave de apertura nunca debe ir en su propia línea y siempre debe ir seguida de una línea nueva. Una llave de cierre siempre debe ir en su propia línea, a menos que sea seguida por `else`. Siempre ponle sangría al código que va dentro de las llaves.


```r
# Bien
if (y < 0 && debug) {
 message("Y es negativo")
}

if (y == 0) {
 log(x)
} else {
 y ^ x
}

# Mal
if (y < 0 && debug)
message("Y is negative")

if (y == 0) {
 log(x)
}
else {
 y ^ x
}
```

Está bien evitar las llaves si tienes una sentencia `if` muy corta que cabe en una sola línea:


```r
y <- 10
x <- if (y < 20) "Too low" else "Too high"
```

Esto se recomienda solo para sentencias `if` muy breves. De lo contrario, la sentencia completa es más fácil de leer:


```r
if (y < 20) {
 x <- "Muy bajo"
} else {
 x <- "Muy alto"
}
```

### Ejercicios

1. ¿Cuál es la diferencia entre `if` e `ifelse()`? Lee cuidadosamente la ayuda y construye tres ejemplos que ilustren las diferencias clave.


1. Escribe una función de saludo que diga "buenos días", "buenas tardes" o "buenas noches", según la hora del día. (Sugerencia: usa un argumento de tiempo que por defecto sea `lubridate::now()`; eso hará que sea más fácil testear tu función).

1. Implementa una función `fizzbuzz` que tenga un solo número como input. Si el número es divisible por tres, devuelve "fizz". Si es divisible por cinco, devuelve "buzz". Si es divisible por tres y cinco, devuelve "fizzbuzz". De lo contrario, devuelve el número. Asegúrate de escribir primero código que funcione antes de crear la función.

1. ¿Cómo podrías usar `cut()` (_cortar()_) para simplificar este conjunto de sentencias if-else anidadas?

 
 ```r
 if (temp <= 0) {
 "congelado"
 } else if (temp <= 10) {
 "helado"
 } else if (temp <= 20) {
 "fresco"
 } else if (temp <= 30) {
 "tibio"
 } else {
 "caluroso"
 }
 ```

        ¿Cómo cambiarías la llamada a `cut()` si hubieras usado `<`en lugar de `<=`? ¿Cuál es la otra ventaja principal de `cut()` para este problema? (Pista: ¿qué sucede si tienes muchos valores en `temp`?)

5. ¿Qué sucede si usas `switch()` con un valor numérico?

6. ¿Qué hace la llamada a `switch()`? ¿Qué sucede si `x` fuera “e”?

         
         ```r
         switch(x,
         a = ,
         b = "ab",
         c = ,
         d = "cd"
         )
         ```

        Experimenta, luego lee cuidadosamente la documentación.


## Argumentos de funciones

Los argumentos de las funciones normalmente están dentro de dos conjuntos amplios: un conjunto provee los __datos__ a computar y el otro los argumentos que controlan los __detalles__ de la computación. Por ejemplo:


* En `log()`, los datos son `x`, y los detalles son la `base` del algoritmo.

* En `mean()`, los datos son `x`, y los detalles son la cantidad de datos para recortar de los extremos
 (`trim`) y cómo lidiar con los valores faltantes (`na.rm`).

* En `t.test()`, los datos son `x` e `y`, y los detalles del test son
 `alternative`, `mu`, `paired`, `var.equal`, y `conf.level`.

* En `str_c()` puedes suministrar cualquier número de caracteres a `...`, y los detalles de la concatenación son contralos por `sep` y `collapse`.

Generalmente, argumentos relativos a los datos deben ir primero. El detalle de los mismos podría estar al final y con valores por defecto. Se especifica un valor por defecto de la misma manera en la que se llama a una función con un argumento nombrado:


```r
# Computar intervalo de confianza alrededor de la media usando la aproximación normal 
mean_ci <- function(x, conf = 0.95) {
 se <- sd(x) / sqrt(length(x))
 alpha <- 1 - conf
 mean(x) + se * qnorm(c(alpha / 2, 1 - alpha / 2))
}

x <- runif(100)
mean_ci(x)
#> [1] 0.498 0.610
mean_ci(x, conf = 0.99)
#> [1] 0.480 0.628
```

El valor por defecto debería ser casi siempre el valor más común. Las pocas excepciones que existen a esta regla deben realizarse con cuidado. Por ejemplo, tiene sentido que `na.rm` por defecto sea `FALSE` porque los valores faltantes son importantes. Aunque `na.rm = TRUE` es lo que usualmente pones en tu codigo, es una mala idea que el comportamiento por defecto sea ignorar silenciosamente los valores faltantes.

Cuando llamas una función, generalmente omites los nombres de los argumentos de datos justamente porque son los más comúnmente usados. Si quieres usar un valor distinto al por defecto en  de un argumento de detalle, debes usar el nombre completo:


```r
# Bien
mean(1:10, na.rm = TRUE)

# Mal
mean(x = 1:10, , FALSE)
mean(, TRUE, x = c(1:10, NA))
```

Puedes referirte a un argumento por su prefijo único (ej. `mean(x, n = TRUE)`), pero generalmente es mejor evitarlo dadas las posibilidades de confusión.

Ten en cuenta que cuando llamas a una función, debes colocar un espacio alrededor de `=` y siempre poner un espacio después de la coma, no antes (como cuando escribes en español). El uso del espacio en blanco hace más fácil echar un vistazo a la función para identificar los componentes importantes.


```r
# Bien
promedio <- mean(pies / 12 + pulgadas, na.rm = TRUE)

# Mal
promedio<-mean(pies/12+pulgadas,na.rm=TRUE)
```

### Elección de nombres

Los nombres de los argumentos también son importantes. A R no le importa, pero sí a quienes leen tu código (¡incluyéndo a tu futuro-yo!). En general, deberías preferir nombres largos y más descriptivos, aunque hay un puñado de nombres muy comunes y muy cortos. Vale la pena memorizar estos:

* `x`, `y`, `z`: vectores.
* `w`: un vector de pesos.
* `df`: un data frame.
* `i`, `j`: índices numéricos (usualmente filas y columnas).
* `n`: longitud, o número de filas.
* `p`: número de columnas.

En caso contrario, deberías considerar hacerlos coincidir con nombres de argumentos de funciones de R que ya existen. Por ejemplo, usa `na.rm` para determinar si los valores faltantes deberían ser eliminados.

### Chequear valores

A medida que vayas escribiendo más funciones,  eventualmente llegarás al punto en el que no recordarás cómo opera una determinada función. En este punto es común que llames a la función con inputs inválidos. Para evitar este problema, a menudo es útil hacer las restricciones explícitas. Por ejemplo, imagina que has escrito algunas funciones para calcular estadísticos de resumen ponderados:


```r
wt_mean <- function(x, w) {
 sum(x * w) / sum(w)
}
wt_var <- function(x, w) {
 mu <- wt_mean(x, w)
 sum(w * (x - mu) ^ 2) / sum(w)
}
wt_sd <- function(x, w) {
 sqrt(wt_var(x, w))
}
```

¿Qué pasa si `x` y `w` no son de la misma longitud?


```r
wt_mean(1:6, 1:3)
#> [1] 7.67
```

En este caso, debido a las reglas de reciclado de vectores de R, no obtenemos un error.

Es una buena práctica verificar las condiciones previas importantes y arrojar un error (con `stop()`, _parar_), si estas no son verdaderas:


```r
wt_mean <- function(x, w) {
 if (length(x) != length(w)) {
 stop("`x` y `w` deben tener la misma extensión", call. = FALSE)
 }
 sum(w * x) / sum(w)
}
```

Ten cuidado de no llevar esto demasiado lejos. Debe haber un equilibrio entre la cantidad de tiempo que inviertes en hacer que tu función sea sólida y la cantidad de tiempo que pasas escribiéndola. Por ejemplo, si además agregas a la función un argumento `na.rm`, probablemente no lo verificaste con cuidado:


```r
wt_mean <- function(x, w, na.rm = FALSE) {
 if (!is.logical(na.rm)) {
 stop("`na.rm` debe ser lógico")
 }
 if (length(na.rm) != 1) {
 stop("`na.rm` debe tener extensión 1")
 }
 if (length(x) != length(w)) {
 stop("`x` y `w` deben tener la misma extensión", call. = FALSE)
 }

 if (na.rm) {
 miss <- is.na(x) | is.na(w)
 x <- x[!miss]
 w <- w[!miss]
 }
 sum(w * x) / sum(w)
}
```

Esto es mucho trabajo con poca ganancia adicional. Una opción útil es incorporar
`stopifnot()`: esto comprueba que cada argumento sea `TRUE`. En caso contrario genera un mensaje de error.


```r
wt_mean <- function(x, w, na.rm = FALSE) {
 stopifnot(is.logical(na.rm), length(na.rm) == 1)
 stopifnot(length(x) == length(w))

 if (na.rm) {
 miss <- is.na(x) | is.na(w)
 x <- x[!miss]
 w <- w[!miss]
 }
 sum(w * x) / sum(w)
}
wt_mean(1:6, 6:1, na.rm = "foo")
#> Error in wt_mean(1:6, 6:1, na.rm = "foo"): is.logical(na.rm) is not TRUE
```

Ten en cuenta que al usar `stopifnot()` afirmas lo que debería ser cierto en lugar de verificar lo que podría estar mal.

### Punto-punto-punto (...)

Muchas funciones en R tienen un número arbitrario de inputs:


```r
sum(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
#> [1] 55
stringr::str_c("a", "b", "c", "d", "e", "f")
#> [1] "abcdef"
```

¿Cómo operan estas funciones? Estas se sostienen en un argumento especial: `...` (llamado punto-punto-punto). Este argumento especial captura cualquier número de argumentos que no estén contemplados de otra forma.

Es práctico porque puedes enviar estos `...` a otra función. Este es un argumento multipropósito útil si tu función principalmente envuelve (_wraps_) a otra función. Por ejemplo, usualmente creamos estas funciones de ayuda alrededor de `str_c()`:


```r
commas <- function(...) stringr::str_c(..., collapse = ", ")
commas(letters[1:10])
#> [1] "a, b, c, d, e, f, g, h, i, j"

rule <- function(..., pad = "-") {
 title <- paste0(...)
 width <- getOption("width") - nchar(title) - 5
 cat(title, " ", stringr::str_dup(pad, width), "\n", sep = "")
}
rule("Important output")
#> Important output -----------------------------------------------------------
```

Aquí `...` nos permite  enviar cualquier argumento con el que no queramos lidiar hacia `str_c()`. Esto es muy conveniente, pero tiene un costo asociado: cualquier argumento mal escrito no generará un error. Esto hace que sea más fácil que los errores de tipeo pasen inadvertidos:


```r
x <- c(1, 2)
sum(x, na.mr = TRUE)
#> [1] 4
```

Si solo quieres capturar los valores de `...`, entonces utiliza `list(...)`.

### Evaluación diferida

Los argumentos en R se evalúan de forma "perezosa": no se computan hasta que se los necesita. Esto significa que si nunca se los usa, nunca son llamados. Esta es una propiedad importante de R como lenguaje de programación, pero generalmente no es fundamental cuando escribes tus propias funciones para el análisis de datos. Puedes leer más acerca de la evaluación diferida en
<https://adv-r.hadley.nz/functions.html#lazy-evaluation>.

### Ejercicios

1. ¿Qué realiza `commas(letters, collapse = "-")`? ¿Por qué?

1. Sería bueno si se pudiera suministrar múltiples caracteres al argumento `pad`, por ejemplo, `rule("Title", pad = "-+")`. ¿Por qué esto actualmente no funciona? ¿Cómo podrías solucionarlo?

1. ¿Qué realiza el argumento `trim` a la función `mean()`? ¿Cuándo podrías utilizarlo?

1. El valor de defecto del argumento `method` para `cor()` es
 `c("pearson", "kendall", "spearman")`. ¿Qué significa esto? ¿Qué valor se utiliza por defecto?

## Valores de retorno

Darse cuenta qué es lo que tu función debería devolver suele ser bastante directo: ¡es el porqué de crear la función en primer lugar! Hay dos cosas que debes considerar al retornar un valor:

1. ¿Devolver un valor antes hace que tu función sea más fácil de leer?

1. ¿Puedes hacer tu función apta para utilizarla con pipes (`%>%`)?

### Sentencias de retorno explícitas

El valor devuelto por una función suele ser la última sentencia que esta evalúa; sin embargo, puedes optar por devolver algo anticipadamente haciendo uso de la función `return()` (_retornar_ o _devolver_ en inglés). Creemos que es mejor reservar el uso de la función `return()` para los casos en los que es posible devolver anticipadamente una solución más simple. Una razón común para hacer esto es que los argumentos estén vacíos:


```r
complicated_function <- function(x, y, z) {
 if (length(x) == 0 || length(y) == 0) {
 return(0)
 }
 # Código complicado aquí
}
```

Otra razón puede ser porque tienes una sentencia `if` con un bloque complicado y uno sencillo. Por ejemplo, podrías escribir una sentencia _if_ de esta manera:


```r
f <- function() {
 if (x) {
 # Haz
 # algo
 # que
 # tome
 # muchas
 # líneas
 # para
 # ser
 # expresado
 } else {
 # retorna algo corto
 }
}
```

Si el primer bloque es muy largo, para cuando lleges al `else` ya te habrás olvidado de la condición. Una forma de reescribir esto es usar un retorno anticipado para el caso sencillo:


```r

f <- function() {
 if (!x) {
 return(algo_corto)
 }

 # Haz
 # algo
 # que
 # tome
 # muchas
 # líneas
 # para
 # ser
 # expresado
}
```

Esto tiende a hacer el código más fácil de entender, ya que no necesitas tanto contexto para interpretarlo.

### Escribir funciones aptas para un pipe

Si quieres escribir funciones que sean aptas para usarlas con un pipe (` %>% `), es importante que pienses en los valores de retorno. Conocer el tipo de objeto de tu valor de retorno significará que tu secuencia de pipes "simplemente funcionará". Por ejemplo, en **dplyr** y **tidyr** el tipo de objeto es un data frame.

Hay dos tipos básicos de funciones aptas para pipes: transformaciones y efectos secundarios. En las __transformaciones__, se ingresa un objeto como primer argumento y se retorna una versión modificada del mismo. En el caso de los __efectos_secundarios__, el objeto ingresado no es modificado, sino que la función realiza una acción sobre el objeto (como dibujar un gráfico o guardar un archivo). Las funciones de efectos secundarios deben retornar “invisiblemente” el primer argumento, de manera que aún cuando no se impriman, puedan ser utilizados en una secuencia de pipes. Por ejemplo, esta función imprime el número de valores faltantes en un data frame:


```r
mostrar_faltantes <- function(df) {
 n <- sum(is.na(df))
 cat("Valores faltantes: ", n, "\n", sep = "")

 invisible(df)
}
```

Si la llamamos de manera interactiva, `invisible()` implica que el `df` input no se imprime:



```r
mostrar_faltantes(mtautos)
#> Valores faltantes: 0
```

Pero sigue estando ahí, solamente que no se imprime por defecto:


```r
x <- mostrar_faltantes(mtautos)
#> Valores faltantes: 0
class(x)
#> [1] "data.frame"
dim(x)
#> [1] 32 11
```

Y todavía podemos usarlo en un pipe:



```r
mtautos %>%
 mostrar_faltantes() %>%
 mutate(millas = ifelse(millas < 20, NA, millas)) %>%
 mostrar_faltantes()
#> Valores faltantes: 0
#> Valores faltantes: 18
```

## Entorno

El último componente de una función es su entorno. Esto no es algo que debas entender con profundidad cuando recién empiezas a escribir funciones. Sin embargo, es importante saber un poco acerca de los entornos, ya que son cruciales para que algunas funciones trabajen. El entorno de una función controla cómo R encuentra el valor asociado a un nombre. Por ejemplo, toma la siguiente función:


```r
f <- function(x) {
 x + y
}
```

En muchos lenguajes de programación, esto sería un error, porque `y` no está definida dentro de la función. En R, esto es un código válido ya que R usa reglas llamadas de __ámbito léxico__ (_lexical scoping_) para encontrar el valor asociado a un nombre. Como `y` no está definida dentro de la función, R mirará dentro del __entorno__ donde la función fue definida:


```r
y <- 100
f(10)
#> [1] 110

y <- 1000
f(10)
#> [1] 1010
```

Este comportamiento parece una receta para errores (_bugs_) y, de hecho, debes evitar crear deliberadamente funciones como esta. Sin embargo, en líneas generales no causa demasiados problemas (especialmente si reinicias regularmente R para hacer borrón y cuenta nueva). 
La ventaja de este comportamiento es que, desde el punto de vista del lenguaje, permite que R sea muy consistente. Cada nombre es buscado usando el mismo conjunto de reglas. Para `f()` esto incluye el comportamiento de dos cosas que podrías no esperar: `{` y `+`. Esto te permite hacer cosas enrevesadas como la siguiente:


```r
`+` <- function(x, y) {
 if (runif(1) < 0.1) {
 sum(x, y)
 } else {
 sum(x, y) * 1.1
 }
}
table(replicate(1000, 1 + 2))
#> 
#>   3 3.3 
#> 100 900
rm(`+`)
```

Este es un fenómeno común en R. R pone pocos límites a tu poder.
Puedes hacer muchas cosas que no podrías hacer en otro lenguaje de programación.
Puedes hacer cosas que el 99% de las veces son extremadamente desaconsejables (¡como sobrescribir manualmente cómo funciona la adición!). Pero este poder y flexibilidad es lo que hace que herramientas como **ggplot2** y **dplyr** sean posibles. Aprender cómo hacer el mejor uso de esta flexibilidad está mas allá del alcance de este libro, pero puedes leer al respecto en [_Advanced R_](http://adv-r.hadley.nz).
