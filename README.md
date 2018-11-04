# Práctico de Clustering - David González  
- - -

Preproceso:
-- 

##### **Corpus elegido:**
 - El corpus elegido fue el de La Voz 

##### **Normalizado de palabras:**
- La herramienta utilizada mayormente fué **nltk**
- Filtrado de palabras:
Para armar los tokens primero se procesa el texto línea por línea  y luego palabra por palabra, con sent_tokenize y word_tokenize respectivamente. Con esto obtenemos una lista con todas las palabras/números/símbolos del corpus.  
Luego, con la libreria **re** filtramos todo lo que no sea palabra o número y finalmente hacemos el pos tagging con pos_tag.
- Diccionario de features:
Al procesar las tuplas de (palabra,tag) obtenidas, si la palabra es un número, ésta es reemplazada por el string "NUMERO" para que todos los números estén representados de igual forma. Si la palabra es una stopword entonces se descarta.  
Las features para cada palabra en el diccionario son: 
	- is_number: si la palabra es un número (por ej. "dos")
	- is_upper: para saber si la palabra está en mayúsculas.
	- tag: el tag de la palabra.
	- frequency: cantidad de veces que la palabra aparece en el corpus.
	- stem: el stem de la palabra, obtenido con SnowballStemmer.
	- prevword: la palabra anterior.
	- prevtag: tag de la palabra anterior.
	- nextword: palabra siguiente.
	- nexttag: tag de la palabra siguiente.
	- START: si es incio de oración.
	- END: si es final de oración.  

Finalmente sacamos del diccionario las palabras del diccionario que tengan poca frecuencia.

- Vectores de palabras:
Para vectorizar el diccionario primero se guardan los índices de las palabras para luego poder recuperarlas y luego se aplica Dictvectorizer. Se normalizan los vectores obtenidos y luego, a través de TruncatedSVD, se aplica SVD para reducir la dimensionalidad.

**Clustering:**
--
#### **Herramienta utilizada:**
La herramienta elegida fue **K-means**, de sklearn.  
K-means es un método de clustering de complejidad lineal (uno de los algoritmos de clustering más rápidos), que tiene como objetivo la partición de un conjunto de n observaciones en k grupos en el que cada observación pertenece al grupo cuyo valor medio es más cercano. K-means utiliza la distancia euclídea (los centroides no se normalizan con su longitud).  
Cada clúster se define por su centroide ("causa latente") y se trata de minimizar la distancia de los elementos con su centroide iterando con dos pasos:  

-  Reasignación: Se asigna cada vector a su centroide más cercano.
-  Recomputar: Se recomputa cada centroide como el promedio de vectores que fueron asignados a tal centroide en la etapa de reasignación.

Que el algoritmo converja no implica que sea el resultado óptimo, por lo tanto tiene que probarse con distintas semillas. También se puede definir un criterio de parada para no esperar que converja totalmente (ya que podría demasiadas iteraciones).  
El algoritmo se ejecuta un número de veces (con diferentes semillas) y luego se define cuál fué el mejor conjunto de clusters obtenido, usando la medida de compasidad de los clusters.  
Particularmente el algoritmo de sklearn tiene varios parámetros para definir cosas como las mencionadas anteriormente y algunas más, como la cantidad de clusters, forma de elegir los centroides iniciales, etc.  
Para medir la efectividad/calidad de los clusters obtenidos, en este caso, se va a evaluar el resultado "a ojo", ya que se sabe (aproximadamente) que tipo de clusters debemos encontrar.

#### **Parámetros utilizados:**
- Tamaño de corpus: 
	- Cantidad de líneas tomadas: Desde 6000 líneas a corpus completo.
- Palabras poco frecuentes:
	- Umbral elegido: Depende directamente de la cantidad de líneas procesadas: Para 6000 líneas: 30, para 10000: 50, para 25000: 100, corpus completo: 150
- Reducción de dimensionalidad:
	- n_components: Se varió entre 100 y 600
- K-means: 
	- n_clusters: Entre 20 y 60.
	- n_init: Entre 10 y 100.
	- max_iter: Entre 300 y 500
	- random_state: Entre 0 y 50.
#### **Resultados obtenidos:**

Variando los parámetros nombrados arriba, en algunos casos no se observaron grandes cambios en los clusters obtenidos, mientras que en otros casos si.  

- El uso de stemming mostró cosas positivas como tener realizará, realizar, realizó, realizado, realiza, realizan y realizada en el mismo cluster, y también mostró cosas negativas como tener sábado, saben, sabe, sabemos y saber en el mismo clúster, aunque son casos aislados y en general fué más lo bueno que lo malo.
- Con el umbral de frecuencia de las palabras en general se obtuvo diccionarios de 3500 a 4500 palabras, lo que hizo cómodo poder diferenciar bien los clusters ya que éstos no tenían demasiadas palabras.
- Normalizar la matriz obtenida de Dictvectorizer definitivamente arregló el problema de obtener singletons.
- Para 10000 líneas aprox, y eligiendo **n_components** = 100 se obtuvieron clusters con mucha mayor cantidad de palabras que el resto; con 600 dimensiones se obtuvieron clusters con muy pocas palabras (de 2 a 5 aprox); finalmente 300 pareció balancear los problemas mencionados.
- En **k-means** no se vió una diferencia significante modificando los parametros n_init, max_iter y random_state. La cantidad de clusters que aparentemente fue la más adecuada fue de 30 clusters, los cuales oscilaron entre 50 y 200 palabras cada uno.

El resultado final en términos generales (sobre todo considerando que se usó mayoritariamente nltk) fue bueno:  

- Clusters de sólo nombres propios.
- Clusters de sólo verbos.
- Un cluster que contiene nombres de países y provincias.
- Un cluster de palabras que se escriben típicamente en mayúscula (UNC, USA, ADN, IVA, ONU, etc).
- Un cluster con mayoría de establecimientos como colegios, ipem, edificios, departamento, museo, hogares, comercios, inmuebles, hospitales, boliches, instituto, club, cabildo, palacio, aeropuerto, vaticano, teatro, parque, monte, valle, etc
- Un cluster con mayoría de palabras basados en el ámbito educativo, como  estudiantes, escolares, adolescentes, docentes, directores, militantes, profesores, etc; Aunque este mismo cluster tuvo palabras que no tendría que tener, como lugares, habitantes, pobres, muertes, donantes, entre otros...
- Un cluster con sólo números (incluyendo la palabra NUMERO).
- Un cluster con los meses del año (si bien el cluster tenía también otras palabras no relacionadas).
- En general los clusters contienen palabras que deberían estar juntas (en vez de estar en clusters distintos) como kircherismo, cristina, gobierno, macri, etc...