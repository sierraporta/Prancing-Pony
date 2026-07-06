# **What if...** toast could predict the weather? Tostadas, correlaciones espurias y el arte de engañarnos con datos

## Introducción

Con esta entrada inauguro una serie nueva en el blog. La idea es simple: cada entrega comienza con una pregunta aparentemente absurda —un **What if...**— y termina, espero, revelando algo serio sobre cómo razona la ciencia con datos. La gracia del formato es que la pregunta invita a la especulación, pero obliga a responder con evidencia, modelos y simulación. Un equilibrio entre imaginación y rigor.

Empecemos con una pregunta digna de ese espíritu:

> **¿Y si el tostado de mi pan pudiera predecir el clima?**

Suena ridículo, y lo es. Pero les propongo tomarla en serio durante unos minutos, porque el camino hacia el "no" es mucho más instructivo que el "no" mismo. De hecho, como veremos, los datos van a decir que **sí**. Y ahí está precisamente el problema.

## El experimento: un año de tostadas

Imaginemos que durante un año entero mido, cada mañana, el nivel de tostado de mi pan con un índice entre 0 (pan crudo) y 100 (carbón). El tostador es viejo, la resistencia se degrada, a veces alguien mueve la perilla, o una serie de apagones no declarados en la ciudad va dañando cada vez el circuito simple que la hace funcionar: el índice fluctúa lentamente día a día. En paralelo, registro la temperatura máxima de la ciudad.

Al final del año calculo la correlación de Pearson entre ambas series,
\begin{equation}
r = \frac{\sum_i (x_i - \bar{x})(y_i - \bar{y})}{\sqrt{\sum_i (x_i - \bar{x})^2}\,\sqrt{\sum_i (y_i - \bar{y})^2}},
\end{equation}
y obtengo un resultado sorprendente: $r = 0.52$, con un p-valor menor que $10^{-27}$. Cualquier revista aceptaría esa significancia sin pestañear. Las tostadas <<Mis tostadas>>, al parecer, saben algo del clima.

<p style="text-align:center;">
  <img src="{{ '/Figures/ToastTemp.png' | relative_url }}"
       alt="Índice de tostado y temperatura máxima durante un año"
       style="width:70%; height:auto;">
</p>

Aclaro, por si hace falta: **todo es simulado**. El índice de tostado es un proceso autorregresivo sin ninguna conexión con la temperatura, que a su vez es un ciclo estacional con su propio ruido. Son series generadas de forma completamente independiente. La correlación es real en el sentido aritmético, pero no significa absolutamente nada. La pregunta interesante es: ¿cómo puede el azar producir algo tan convincente?

## Buscar hasta encontrar

Hagamos algo peor. Supongamos que no solo mido el tostado: mido mil variables de mi cocina. El tiempo que tarda la cafetera, la inclinación de la mantequilla al untarla, cuántas migas caen al plato. Mil series aleatorias, generadas sin ninguna relación con el clima, y todas correlacionadas contra la misma temperatura.

El resultado debería incomodarnos: el **77%** de esas variables resulta "estadísticamente significativa" al nivel usual de $p < 0.05$, y casi el 40% supera $|r| > 0.3$. La mejor de todas alcanza $r = -0.80$ con un p-valor del orden de $10^{-82}$. Si yo publicara solamente esa variable ganadora —y olvidara convenientemente las otras 999—, tendría un "descubrimiento" espectacular.

<p style="text-align:center;">
  <img src="{{ '/Figures/DredgingHist.png' | relative_url }}"
       alt="Histograma de correlaciones de 1000 variables aleatorias con la temperatura"
       style="width:70%; height:auto;">
</p>

Esto tiene nombre: **data dredging** (o *p-hacking*, cuando se hace con conocimiento de causa). Y aunque el ejemplo de la cocina es caricaturesco, el mecanismo es idéntico al de quien prueba cientos de indicadores hasta encontrar el que "predice" el mercado, o cientos de alimentos hasta encontrar el que "causa" cáncer. El p-valor individual no sabe cuántas preguntas hicimos antes de quedarnos con la respuesta que nos gustó. Por eso existen las correcciones por comparaciones múltiples, como la de Bonferroni: con mil pruebas, el umbral honesto no es $0.05$, sino $0.05/1000$.

## La memoria de las series

Pero hay algo más profundo aquí, y es mi parte favorita. ¿Por qué la correlación tostada–temperatura dio $0.52$ y no algo cercano a cero, si las series son independientes?

La respuesta está en la **memoria**. Tanto el índice de tostado como la temperatura son series persistentes: el valor de hoy se parece al de ayer. Y cuando dos series tienen memoria larga, la correlación de Pearson entre ellas deja de comportarse como esperamos. Los grados de libertad efectivos son muchos menos que los 365 aparentes, y el azar produce correlaciones enormes con total facilidad.

El caso extremo es la caminata aleatoria. Si genero dos caminatas completamente independientes —puro azar acumulado— y las correlaciono, la mediana de $|r|$ que obtengo no es cero: es **0.43**. Y más del 40% de los pares supera $|r| > 0.5$. No es difícil encontrar pares con $r = 0.90$ que, puestos en una gráfica, parecen series gemelas.

<p style="text-align:center;">
  <img src="{{ '/Figures/RandomWalks.png' | relative_url }}"
       alt="Dos caminatas aleatorias independientes con correlación 0.90 y distribución de correlaciones entre pares independientes"
       style="width:85%; height:auto;">
</p>

Este fenómeno no es nuevo. Udny Yule lo describió en 1926 en un artículo con un título maravilloso: *Why do we sometimes get nonsense-correlations between time-series?* (Yule, G. U. (1926). Why do we sometimes get nonsense-correlations between Time-Series?--a study in sampling and the nature of time-series. Journal of the royal statistical society, 89(1), 1-63. https://doi.org/10.2307/2341482). Un siglo después seguimos tropezando con la misma piedra, ahora con más datos y mejores gráficas. Quien quiera una colección moderna y deliciosa de estos disparates puede visitar el proyecto [Spurious Correlations](https://www.tylervigen.com/spurious-correlations) de Tyler Vigen, donde el consumo de queso per cápita correlaciona con las muertes por enredarse en las sábanas.

## La prueba de fuego: predecir el futuro

Hay una manera honesta de desenmascarar a las tostadas, y es la más importante de todas: **pedirle al modelo que prediga datos que nunca vio**.

Entrené modelos polinomiales de complejidad creciente para predecir la temperatura a partir del índice de tostado, usando los primeros nueve meses del año, y los evalué sobre los últimos tres. El resultado es demoledor: mientras el $R^2$ de entrenamiento se mantiene estable alrededor de $0.27$, el $R^2$ de prueba es **negativo** para todos los modelos —llegando a $-5$—, lo que significa que predicen peor que simplemente usar el promedio histórico. La "señal" que el modelo aprendió no era señal: era la coincidencia particular de ese año, de esas tostadas.

<p style="text-align:center;">
  <img src="{{ '/Figures/Overfit.png' | relative_url }}"
       alt="R2 de entrenamiento vs prueba para modelos de complejidad creciente"
       style="width:70%; height:auto;">
</p>

Esta asimetría —ajustar bien el pasado, fracasar en el futuro— es la firma inconfundible de la correlación espuria. Y la validación fuera de muestra es, en mi opinión, el detector de mentiras más barato y efectivo que tenemos en ciencia de datos.

## Cierre

La tostada no predice el clima, pero nos deja tres lecciones que valen para cualquier análisis serio. Primero: **una correlación fuerte y significativa no es evidencia de nada** si no controlamos cuántas preguntas hicimos antes de encontrarla. Segundo: **las series con memoria conspiran contra nuestra intuición estadística**; dos procesos persistentes e independientes pueden parecer gemelos durante un año entero. Y tercero: la prueba definitiva nunca es qué tan bien explicamos el pasado, sino qué tan bien anticipamos lo que aún no hemos visto.

Quizás esa sea la moraleja de este primer *What if*: la pregunta absurda no era si las tostadas predicen el clima, sino cuántos "descubrimientos" que circulan por ahí son, en el fondo, tostadas con buen p-valor.

Para quienes deseen reproducir las simulaciones, modificar parámetros o buscar sus propias correlaciones absurdas, dejo disponible el notebook en Python en: [notebook_toast_weather.ipynb](https://github.com/sierraporta/prancing-pony/blob/main/Codes/notebook_toast_weather.ipynb).
