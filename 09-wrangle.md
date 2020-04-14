# (PART) Manejar o domar datos {-}


# Introducción {#domardatos-intro}

En esta parte del libro, aprenderás cómo "domar" datos (_data wrangling_, en inglés): el arte de tener tus datos en R de una forma conveniente para su visualización y modelado. Si bien en español también se suele hacer referencia a esta etapa como _manejo_ o _manipulación_ de datos, hemos mantenido la traducción literal ya que, como se señala en la introducción, la noción de domar (_wrangling_) apunta a lo difícil que puede ser a veces este proceso. 
La doma de datos es muy importante: ¡sin ella no puedes trabajar con tus propios datos! Este proceso tiene tres partes principales:

<img src="diagrams_w_text_as_path/es/data-science-wrangle.svg" width="75%" style="display: block; margin: auto;" />

Esta parte del libro continúa de la siguiente forma:

* En [Tibbles] aprenderás sobre la variante de _data frame_ que usamos
 en este libro: el __tibble__. Conocerás qué los hace diferentes de los
 data frames comunes y cómo puedes construirlos "a mano".

* En [Importación de datos] aprenderás cómo traer tus datos del disco hacia R.
 Nos enfocaremos en los formatos rectangulares de texto plano, pero daremos referencias
 a paquetes que ayudan con otros tipos de datos.

* En [Datos ordenados] aprenderás una manera consistente de almacenar
 tus datos que facilita la transformación, la visualización y el modelado.
 Aprenderás los principios subyacentes y cómo poner tus datos en una forma ordenada.

La doma de datos también abarca la transformación de los mismos, algo sobre lo que ya has aprendido un poco. Ahora nos enfocaremos en nuevas habilidades para cuatro tipos de datos específicos que encontrarás frecuentemente en la práctica:

* La sección [Datos relacionales] te dará herramientas para trabajar con múltiples
 conjuntos de datos interrelacionados.

* [Cadenas de caracteres] te introducirá en las expresiones regulares (_regular expressions_), una herramienta
 poderosa para manipular cadenas de caracteres (_strings_).

* En [Factores] veremos cómo R almacena los datos categóricos. Los factores se utilizan cuando una variable
 tiene un conjunto fijo de posibles valores, o cuando quieres usar una cadena de caracteres en un
 orden distinto al alfabético.

* [Fechas y horas] te dará herramientas clave para trabajar con
 fechas y fecha-horas.
