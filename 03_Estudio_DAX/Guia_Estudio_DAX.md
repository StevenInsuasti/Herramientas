# Guía de estudio DAX - Grupo 7, Proyecto 7

## Para qué sirve esta guía

Este material reúne las diez consultas tabulares y las medidas que calculan sus
resultados. Está escrito para estudiar y explicar el proyecto paso a paso con
el dataset de restaurantes de Yelp.

Una **medida** es una fórmula guardada en el modelo; responde al contexto de un
visual o de los filtros. Una **consulta DAX** usa `EVALUATE` para devolver una
tabla de resultados. Las diez consultas se ejecutan por separado en la Vista de
consultas DAX.

Para exponer cada análisis, expliquen: **qué pregunta responde, qué calcula la
medida, qué columnas organizan el resultado y qué cautela tiene**.

## 1. Medidas principales

### 1. Total Reseñas

**En palabras:** cuenta las reseñas que tienen una calificación. En este modelo
cada fila de `fact_review` representa una reseña.

```DAX
Total Reseñas =
COUNT(fact_review[stars])
```

### 2. Calificación Promedio

**En palabras:** calcula el promedio de estrellas en el contexto actual. Si un
visual muestra una ciudad, por ejemplo, calcula el promedio de esa ciudad.

```DAX
Calificación Promedio =
AVERAGE(fact_review[stars])
```

### 3. Votos Útiles Totales

**En palabras:** suma los votos marcados como útiles. Es una medida de
interacción, no una calificación de satisfacción.

```DAX
Votos Útiles Totales =
SUM(fact_review[useful])
```

### 4. % Reseñas Positivas

**En palabras:** divide las reseñas con etiqueta `Positive` entre el total de
reseñas del contexto. Se debe presentar con formato de porcentaje.

```DAX
% Reseñas Positivas =
DIVIDE(
    CALCULATE(
        [Total Reseñas],
        fact_review[sentiment_label] = "Positive"
    ),
    [Total Reseñas],
    0
)
```

### 5. Calificación MTD

**En palabras:** calcula el promedio de estrellas desde el comienzo del mes
hasta la fecha evaluada. Necesita la relación activa entre `dim_date[date]` y
la fecha de la reseña.

```DAX
Calificación MTD =
CALCULATE(
    [Calificación Promedio],
    DATESMTD(dim_date[date])
)
```

### 6. Calificación YTD

**En palabras:** calcula el promedio de estrellas desde el comienzo del año
hasta la fecha evaluada. Es un promedio acumulado, no una suma.

```DAX
Calificación YTD =
CALCULATE(
    [Calificación Promedio],
    DATESYTD(dim_date[date])
)
```

### 7. Cambio Interanual de Calificación

**En palabras:** resta el promedio del mismo período del año anterior al
promedio actual. Un resultado positivo indica aumento en estrellas; uno
negativo, disminución. Si no existe período comparable, devuelve vacío.

```DAX
Cambio Interanual de Calificación =
VAR CalificaciónActual = [Calificación Promedio]
VAR CalificaciónAnterior =
    CALCULATE(
        [Calificación Promedio],
        SAMEPERIODLASTYEAR(dim_date[date])
    )
RETURN
    IF(
    ISBLANK(CalificaciónActual) || ISBLANK(CalificaciónAnterior),
        BLANK(),
    CalificaciónActual - CalificaciónAnterior
    )
```

### 8. Ranking por Reseñas

**En palabras:** ordena los negocios por volumen de reseñas. `DENSE` conserva
rangos consecutivos cuando hay empates. No es un ranking de calidad.

```DAX
Ranking por Reseñas =
RANKX(
    ALL(dim_business),
    [Total Reseñas],
    [Total Reseñas],
    DESC,
    DENSE
)
```

### 9. Reseñas con Quejas

**En palabras:** cuenta una vez cada reseña cuyo campo de términos incluya al
menos uno de estos términos: `bad`, `slow`, `cold`, `rude` o `dirty`.

```DAX
Reseñas con Quejas =
COUNTROWS(
    FILTER(
        fact_review,
        SEARCH("bad", LOWER(fact_review[complaint_terms]), 1, 0) > 0
            || SEARCH("slow", LOWER(fact_review[complaint_terms]), 1, 0) > 0
            || SEARCH("cold", LOWER(fact_review[complaint_terms]), 1, 0) > 0
            || SEARCH("rude", LOWER(fact_review[complaint_terms]), 1, 0) > 0
            || SEARCH("dirty", LOWER(fact_review[complaint_terms]), 1, 0) > 0
    )
)
```

### 10. Desviación de Calificaciones

**En palabras:** mide cuánto se dispersan las estrellas respecto al promedio.
Una desviación alta indica más variabilidad; por sí sola no indica mayor ni
menor satisfacción.

```DAX
Desviación de Calificaciones =
STDEV.P(fact_review[stars])
```

## 2. Medidas auxiliares del modelo

### KPI Satisfacción

Esta medida adicional clasifica el promedio de estrellas. Los umbrales de 4,5
y 3,5 son una decisión del equipo, no valores definidos por la guía del
docente. El KPI porcentual principal sigue siendo `% Reseñas Positivas`.

```DAX
KPI Satisfacción =
SWITCH(
    TRUE(),
    [Calificación Promedio] >= 4.5, "Excelente",
    [Calificación Promedio] >= 3.5, "Aceptable",
    "Por mejorar"
)
```

### Top 10 mejor calificado

Esta medida es un filtro auxiliar del visual; no reemplaza ninguna de las diez
consultas principales. Considera solo negocios con al menos 100 reseñas, ordena
por promedio de estrellas y desempata por total de reseñas e ID.

**Importante:** esta fórmula ya se aplicó en el PBIX. La tabla quedó limitada a
negocios con al menos 100 reseñas. El Top 10 por volumen de la consulta 8 es
distinto.

```DAX
Es Top 10 Restaurante =
VAR NegociosElegibles =
    FILTER(
        ALL(dim_business),
        [Total Reseñas] >= 100
    )
VAR TopNegocios =
    TOPN(
        10,
        NegociosElegibles,
        [Calificación Promedio], DESC,
        [Total Reseñas], DESC,
        dim_business[business_id], ASC
    )
VAR NegocioActual = SELECTEDVALUE(dim_business[business_id])
RETURN
    IF(
        ISBLANK(NegocioActual),
        0,
        IF(
            CONTAINS(
                TopNegocios,
                dim_business[business_id], NegocioActual
            ),
            1,
            0
        )
    )
```

## 3. Diez consultas tabulares, en orden

### Consulta 1. ¿Cuántas reseñas tiene cada ciudad?

**Qué devuelve:** una fila por ciudad, ordenada desde la que tiene más
reseñas. La salida observada contiene 848 ciudades; Philadelphia aparece con
687.553 reseñas. Los nombres de ciudad se compararon normalizando espacios y
mayúsculas.

```DAX
EVALUATE
SUMMARIZECOLUMNS(
    dim_business[city],
    "TotalResenas", [Total Reseñas]
)
ORDER BY [TotalResenas] DESC
```

### Consulta 2. ¿Cómo cambia la calificación según el rango de precio?

**Qué devuelve:** el promedio de estrellas y el número de reseñas para cada
rango, incluido el grupo sin dato de precio. El rango 4 mostró el promedio más
alto observado, 4,15; esto describe una asociación, no una causa.

```DAX
EVALUATE
SUMMARIZECOLUMNS(
    dim_business[price_range],
    "CalificacionPromedio", [Calificación Promedio],
    "TotalResenas", [Total Reseñas]
)
ORDER BY dim_business[price_range]
```

### Consulta 3. ¿Cuántos votos útiles hay por etiqueta Yelp?

**Qué devuelve:** una fila por etiqueta con la suma de votos útiles, de mayor a
menor. American (New) mostró 265.793 votos útiles. Las etiquetas Yelp no
siempre representan tipos de cocina.

```DAX
EVALUATE
SUMMARIZECOLUMNS(
    fact_review[category],
    "VotosUtiles", [Votos Útiles Totales]
)
ORDER BY [VotosUtiles] DESC
```

### Consulta 4. ¿Qué proporción de reseñas es positiva por etiqueta?

**Qué devuelve:** el porcentaje de reseñas positivas y el total de reseñas
para cada etiqueta. Revisen el tamaño de muestra: un porcentaje alto con pocas
reseñas no prueba que un grupo sea el mejor.

```DAX
EVALUATE
SUMMARIZECOLUMNS(
    fact_review[category],
    "PorcentajePositivas", [% Reseñas Positivas],
    "TotalResenas", [Total Reseñas]
)
ORDER BY [PorcentajePositivas] DESC
```

### Consulta 5. ¿Cómo varía el promedio por año, trimestre, mes y día?

**Qué devuelve:** el promedio de estrellas agrupado por los cuatro niveles de
la dimensión de fecha. La jerarquía del visual temporal ya se verificó en el
PBIX.

```DAX
EVALUATE
SUMMARIZECOLUMNS(
    dim_date[year],
    dim_date[quarter],
    dim_date[month_number],
    dim_date[date],
    "CalificacionPromedio", [Calificación Promedio]
)
ORDER BY dim_date[year], dim_date[quarter], dim_date[month_number], dim_date[date]
```

### Consulta 6. ¿Cuál es la calificación MTD de las 31 fechas más recientes?

**Qué devuelve:** la medida MTD para las 31 fechas con datos más recientes,
ordenadas de la más reciente a la más antigua. El 19 de enero de 2022 mostró
3,83.

```DAX
EVALUATE
TOPN(
    31,
    SUMMARIZECOLUMNS(
        dim_date[date],
        "CalificacionMTD", [Calificación MTD]
    ),
    dim_date[date], DESC
)
ORDER BY dim_date[date] DESC
```

### Consulta 7. ¿Cuál es la calificación YTD y su cambio interanual?

**Qué devuelve:** una fila por combinación de año y mes, con el promedio YTD
y la diferencia frente al mismo período del año anterior. El cambio queda
vacío si no existe un período comparable.

```DAX
EVALUATE
SUMMARIZECOLUMNS(
    dim_date[year],
    dim_date[month_number],
    "CalificacionYTD", [Calificación YTD],
    "CambioInteranual", [Cambio Interanual de Calificación]
)
ORDER BY dim_date[year], dim_date[month_number]
```

### Consulta 8. ¿Qué negocios tienen más reseñas?

**Qué devuelve:** los diez negocios con mayor volumen de reseñas y su rango.
Los primeros observados fueron Acme Oyster House (7.673) y Oceana Grill
(7.516). Este es un ranking de volumen, no el Top 10 mejor calificado.

```DAX
EVALUATE
TOPN(
    10,
    SUMMARIZECOLUMNS(
        dim_business[business_id],
        dim_business[business_name],
        "TotalResenas", [Total Reseñas],
        "RankingResenas", [Ranking por Reseñas]
    ),
    [RankingResenas], ASC,
    [TotalResenas], DESC,
    dim_business[business_id], ASC
)
ORDER BY [RankingResenas], [TotalResenas] DESC
```

### Consulta 9. ¿Cuántas reseñas contienen términos de queja?

**Qué devuelve:** el conteo por etiqueta Yelp. Cada reseña cuenta una vez
dentro de cada etiqueta aunque contenga varios términos de queja; por eso los
conteos entre etiquetas no se deben sumar como reseñas únicas.

```DAX
EVALUATE
SUMMARIZECOLUMNS(
    fact_review[category],
    "ResenasConQuejas", [Reseñas con Quejas]
)
ORDER BY [ResenasConQuejas] DESC
```

### Consulta 10. ¿Cuánta dispersión tienen las calificaciones?

**Qué devuelve:** la desviación estándar poblacional y el promedio para cada
etiqueta. Una desviación mayor significa más variabilidad, no necesariamente
peor promedio.

```DAX
EVALUATE
SUMMARIZECOLUMNS(
    fact_review[category],
    "DesviacionCalificaciones", [Desviación de Calificaciones],
    "CalificacionPromedio", [Calificación Promedio]
)
ORDER BY [DesviacionCalificaciones] DESC
```

## 4. Cómo explicarlo en la exposición

1. Lean la pregunta de la consulta.
2. Nombren la medida y expliquen su función en una frase.
3. Digan qué columna organiza las filas (ciudad, categoría, precio o fecha).
4. Citen un resultado y su limitación.

No digan que la consulta 8 es el Top 10 de mejor calificación: ordena por
volumen de reseñas. El Top 10 mejor calificado usa la medida auxiliar y el
mínimo de 100 reseñas.
