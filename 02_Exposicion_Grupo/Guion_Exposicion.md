# Guion de presentación - Grupo 7, Proyecto 7

**Integrantes**

- Héctor Alejandro Riascos Insuasty · ID 871752
- Sebastián Orlando Manchabajoy Rosero · ID 899013
- Steven Eraso Insuasti · ID 907337

**Universidad:** Universidad Cooperativa de Colombia  
**Curso:** Herramientas Computacionales para Interpretación de Resultados
(Analítica de Datasets), sexto semestre  
**Docente:** Cristian Camilo Ordoñez Quintero  
**Lugar y fecha:** Pasto, Nariño · 20 de noviembre de 2026

Este guion acompaña `deliverables/presentation.pptx`. Las cinco páginas del
PBIX fueron revisadas, las diez consultas DAX devolvieron «Correcto» y la
versión final del PBIX está publicada en la Release `v1.0.0` y en Power BI
Service. El informe interactivo ya tiene un enlace público de «Publicar en la
web»: [abrir el informe](https://app.powerbi.com/view?r=eyJrIjoiNTdhOWNjNzUtNWIwZS00NzQzLWJjNDktODkyY2EzMDEzZWU0IiwidCI6IjhkMzY4MzZlLTZiNzUtNGRlNi1iYWI5LTVmNGIxNzc1NDI3ZiIsImMiOjR9).
Cualquier persona que obtenga el enlace puede verlo; no es un acceso privado
para el docente.

## Diapositiva 1. Problema y objetivo

Analizamos cómo varían las opiniones y calificaciones de clientes de
restaurantes. El objetivo es procesar reseñas con ETL, estudiar satisfacción por
categoría, ciudad, precio, tiempo y perfil de cliente, y validar diferencias
entre categorías mediante ANOVA.

## Diapositiva 2. Dataset y alcance

Usamos Yelp Open Dataset, alternativa de datos abiertos compartida por el
docente. Elegimos Yelp por sus reseñas textuales, estrellas, fecha, negocios,
categorías, ciudades y datos de actividad de usuarios. El ETL filtra negocios
con categoría `Restaurants`; la muestra del ANOVA se limita a las cinco
categorías con más reseñas y al menos 30 observaciones. Las reseñas de Yelp no
son una muestra aleatoria de todos los clientes de restaurantes.

## Diapositiva 3. ETL y modelo

1. **Extracción:** lectura de archivos JSON Lines de negocios, reseñas y
   usuarios.
2. **Transformación:** selección de restaurantes, validación y limpieza,
   normalización de fechas, cálculo de sentimiento con VADER y preparación de
   categorías, quejas y palabras frecuentes.
3. **Carga:** tablas compactas para Power BI y esquema con hechos de reseñas y
   dimensiones de negocio, usuario y fecha.
4. **Control:** se registran muestras, conteos y resultados estadísticos para
   poder auditar el proceso.

Explicar que VADER clasifica texto en inglés con umbrales de compound: positivo
si `>= 0,05`, negativo si `<= -0,05` y neutral en el intervalo intermedio.

## Diapositiva 4. Diez análisis propios del Proyecto 7

Mostrar cómo el tablero responde a los diez puntos del ejercicio: calificación
por restaurante; sentimiento por categoría; palabras frecuentes; tendencia
mensual; calificación por ciudad; relación precio-satisfacción; Top 10 mejor
calificado entre negocios con al menos 100 reseñas; frecuencia de quejas;
reseñas positivas y negativas; y perfiles de cliente. Aclarar que la frecuencia
de palabras y el sentimiento se preparan en el ETL, no se presentan como
sustitutos de consultas DAX. El umbral reduce rankings dominados por muestras
pequeñas.

## Diapositiva 5. Diez consultas analíticas resueltas mediante DAX

Presentar cada elemento como **pregunta + expresión DAX + contexto del visual +
respuesta/interpretación**. No basta con enumerar nombres de medidas.

1. ¿Cuántas reseñas hay en el contexto seleccionado? `Total Reseñas` (`COUNT`).
2. ¿Cuál es la calificación media por restaurante, ciudad o rango de precio?
   `Calificación Promedio` (`AVERAGE`); el visual compara precio y satisfacción.
3. ¿Cuántos votos útiles hay? `Votos Útiles Totales` (`SUM`), como indicador de
   interacción.
4. ¿Qué proporción de reseñas es positiva? `% Reseñas Positivas`
   (`CALCULATE`), presentado como KPI y segmentable.
5. ¿Cómo cambia la calificación por año, trimestre, mes y día? `Calificación
   Promedio` en el contexto de los campos de fecha de `dim_date`.
6. ¿Cuál es el promedio acumulado del mes? `Calificación MTD` (`DATESMTD`).
7. ¿Cuál es el promedio acumulado del año y su cambio contra el mismo período
   anterior? `Calificación YTD` (`DATESYTD`) y `Cambio Interanual de
   Calificación` (`SAMEPERIODLASTYEAR`, `VAR`, `RETURN`).
8. ¿Qué negocios tienen más reseñas? `Ranking por Reseñas` (`RANKX`); mide
   volumen, no calidad.
9. ¿Cuántas reseñas incluyen términos de queja? `Reseñas con Quejas`
   (`SEARCH` y filtro de filas).
10. ¿Qué dispersión tienen las calificaciones? `Desviación de Calificaciones`
    (`STDEV.P`).

La pregunta es la consulta; la expresión DAX y el contexto producen la
respuesta. El enunciado enumera medidas y funciones, pero no exige
explícitamente sentencias tabulares `EVALUATE`. Como evidencia adicional, se
ejecutaron las diez consultas de `dax_queries.dax` en la Vista de consultas DAX
de Power BI; las diez devolvieron «Correcto». Conservar las capturas como
evidencia. La consulta 8 clasifica por volumen de reseñas, no por calificación.

Resultados para explicar: Q1 mostró 848 ciudades; Philadelphia quedó primero
con 687.553 reseñas. Q2 mostró promedios de 3,65, 3,81, 3,94 y 4,15 para los
rangos 1–4, respectivamente (con 176.385 reseñas sin rango). Q3 mostró más
votos útiles para American (New), con 265.793. Q5 devolvió 6.018 fechas, Q6
mostró 3,83 MTD el 19/01/2022, Q7 devolvió 204 filas año-mes, Q8 clasificó
10 negocios por volumen, Q9 devolvió 420 etiquetas y Q10 devolvió 445. Los
resultados completos y cautelas están en `report.md`.

**Agrupación de ciudades:** la consulta 1 del PBIX devuelve 848 ciudades y
687.553 reseñas en Philadelphia. Un recálculo local que normaliza espacios y
mayúsculas reproduce exactamente esos resultados; el total es 4.724.471
reseñas. La diferencia inicial se debía a comparar agrupaciones de texto con
criterios distintos, no a datos faltantes.

**Cuidado con la categoría:** `category` es la primera etiqueta Yelp no
genérica del negocio seleccionado por tener la etiqueta `Restaurants`; no
siempre es una cocina o tipo de restaurante. En las consultas aparecen
etiquetas como `Photographers`, `Pet Training` y `Roofing`. Describir estos
resultados como grupos de etiquetas Yelp, no como comparación de cocinas.
Además, en la consulta 4 algunos grupos con 7–39 reseñas muestran 100 % de
positividad; no concluir que son los más satisfechos sin considerar el tamaño
de muestra.

## Diapositiva 6. ANOVA: pregunta, hipótesis y supuestos

- **Variable dependiente:** estrellas (`stars`, escala de 1 a 5).
- **Factor:** etiqueta Yelp (`category`) asociada a negocios que incluyen
  `Restaurants`; no necesariamente una categoría de cocina.
- **Muestra:** cinco etiquetas Yelp con más observaciones; 1.210.971 reseñas.
- **H₀:** las cinco medias de calificación son iguales.
- **H₁:** al menos una media es diferente.
- **Normalidad:** Shapiro-Wilk fue significativo en los cinco grupos
  (`p < 0,001`); se aplicó a los primeros 5.000 valores de cada grupo.
- **Varianzas:** Levene `W = 531,05`, `p < 0,001`; no se cumple homogeneidad.

Las calificaciones son discretas y el tamaño de muestra es muy grande, por lo
que se informa el resultado de normalidad con cautela. La desigualdad de
varianzas motiva usar Welch como prueba principal.

## Diapositiva 7. ANOVA: resultado e interpretación

Welch ANOVA: `F(4; 594.272,81) = 976,19`, `p < 0,001`. Se rechaza H₀: hay
evidencia estadística de diferencias entre las medias de al menos dos
etiquetas Yelp. La prueba global no identifica qué pares difieren ni demuestra
causalidad.

El gráfico del PBIX muestra los promedios observados: Breakfast & Brunch 3,88;
American (New) 3,84; Bars 3,74; Mexican 3,73; American (Traditional) 3,66.
Son medias de las etiquetas seleccionadas, no una comparación de todos los
restaurantes ni una recomendación causal.

El ANOVA clásico y las comparaciones de Tukey se reportan solo como análisis
exploratorios, porque Levene detecta varianzas desiguales. No presentar Tukey
como prueba confirmatoria.

## Diapositiva 8. Conclusiones, límites y entrega

- La calificación, el sentimiento y las quejas describen aspectos distintos de
  la experiencia; ninguna medida aislada define toda la satisfacción.
- Las reseñas son autoseleccionadas; los resultados describen esta muestra.
- VADER y los términos de queja son reglas de procesamiento de texto en inglés,
  no una evaluación humana de cada comentario.
- Los cuartiles de actividad de usuario describen volumen de reseñas, no tipos
  oficiales de clientes.
- No afirmar causalidad a partir de las diferencias estadísticas.

El PBIX final fue guardado y sustituido en la Release `v1.0.0`. El Top 10
aplica el mínimo de 100 reseñas y la jerarquía temporal
año-trimestre-mes-fecha quedó comprobada. Las cinco páginas del informe
contienen los diez análisis propios del Proyecto 7. El dashboard HTML entregado
es estático y local; no es una publicación en Power BI Service.
