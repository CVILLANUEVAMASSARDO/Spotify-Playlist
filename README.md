Spotify - Recomendación de Playlist
================

# 1. Introducción

### 1.1 Análisis de música

El objetivo de esta tarea es ver cómo abordan un problema de la vida
real, con incertidumbre, ya que en el mundo profesional se enfrentarán
con tareas similares.

### 1.2 Descripción del problema

Spotify recomienda nuevas canciones a sus usuarios basándose en las
reproducciones pasadas y en estilos musicales similares. Esto lo hace a
través de diversos algoritmos que relacionan las canciones a través de
diferentes atributos como la verbosidad o energía. Una lista con las
mediciones que se hace para cada canción esta disponible en la
documentación de la API de
Spotify(<https://developer.spotify.com/documentation/web-api/reference/>).

El objetivo principal de este encargo es crear un programa computacional
que permita crear una lista de reproducción de 3 horas de duración
basándose en alguna canción de referencia. La base de datos incluye
447.622 canciones, con 36 de las variables descritas en la documentación
de la API.

Como resultado de la prueba, genere un reporte en RMarkdown describiendo
las etapas de su proceso, los modelos de clustering utilizados, los
resultados obtenidos y el código empleado. Debe explicar cómo limpió los
datos, como se eligieron y generaron las variables, y como construyó su
lógica.

### 1.3 Metodología utilizada

Se estudiarán las características de audio de las distintas canciones
con el objetivo de realizar análisis de agrupaciones para ofrecer
recomendaciones que sean similares a la canción de referencia utilizada.
Se intentará comprobar si existe algún patrón para clusterizar y en qué
se diferencia cada cluster.

# 2. Paquetes requeridos

``` r
library(tidyverse)
library(corrplot)
```

### 2.1 Información de paquetes utilizados

**tidyverse** - Permite la manipulación, importación, exploración y
visualización de datos. Contiene los paquetes readr, dplyr, ggplot2,
tibble, tidyr, purr, stringr y forcats.

**corrplot** - Visualización de correlación entre variables.

# 3. Preparación de los datos

### 3.1 Fuente

Se utilizará el archivo “beats.RData” que contiene 447.622 canciones con
36 variables cada una.

Datos extraídos de la API de Spotify.

``` r
load('beats.RData')
beats <- beats
```

    ## El dataframe contiene 447622 filas y 36 columnas.

### 3.2 Información

Nombre de columna, tipo de dato y registro de la columna.

``` r
glimpse(beats)
```

    ## Rows: 447,622
    ## Columns: 36
    ## $ artist_name                  <chr> "2Pac", "2Pac", "2Pac", "2Pac", "2Pac", "…
    ## $ artist_id                    <chr> "1ZwdS5xdxEREPySFridCfh", "1ZwdS5xdxEREPy…
    ## $ album_id                     <chr> "1nGbXgS6toEOcFCDwEl5R3", "1nGbXgS6toEOcF…
    ## $ album_type                   <chr> "album", "album", "album", "album", "albu…
    ## $ album_release_date           <chr> "2019-08-01", "2019-08-01", "2019-08-01",…
    ## $ album_release_year           <dbl> 2019, 2019, 2019, 2019, 2019, 2019, 2019,…
    ## $ album_release_date_precision <chr> "day", "day", "day", "day", "day", "day",…
    ## $ danceability                 <dbl> 0.656, 0.810, 0.548, 0.839, 0.854, 0.697,…
    ## $ energy                       <dbl> 0.882, 0.642, 0.590, 0.657, 0.694, 0.598,…
    ## $ key                          <int> 0, 8, 4, 5, 0, 2, 1, 11, 11, 7, 5, 8, 11,…
    ## $ loudness                     <dbl> -3.011, -8.647, -9.301, -4.959, -4.258, -…
    ## $ mode                         <int> 1, 1, 0, 0, 0, 1, 0, 0, 1, 1, 0, 1, 0, 1,…
    ## $ speechiness                  <dbl> 0.0941, 0.2440, 0.4750, 0.2220, 0.1230, 0…
    ## $ acousticness                 <dbl> 0.03300, 0.04800, 0.11300, 0.05260, 0.009…
    ## $ instrumentalness             <dbl> 0.00e+00, 0.00e+00, 7.22e-04, 1.06e-04, 7…
    ## $ liveness                     <dbl> 0.6700, 0.2640, 0.2290, 0.3910, 0.0767, 0…
    ## $ valence                      <dbl> 0.782, 0.694, 0.267, 0.615, 0.776, 0.387,…
    ## $ tempo                        <dbl> 91.661, 90.956, 87.841, 85.111, 104.379, …
    ## $ track_id                     <chr> "6ayeqYtOtwVhqVB6k6MKoh", "1UDsnzBp8gUCFs…
    ## $ analysis_url                 <chr> "https://api.spotify.com/v1/audio-analysi…
    ## $ time_signature               <int> 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4,…
    ## $ disc_number                  <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,…
    ## $ duration_ms                  <int> 347973, 241026, 240013, 295026, 241000, 2…
    ## $ explicit                     <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE,…
    ## $ track_href                   <chr> "https://api.spotify.com/v1/tracks/6ayeqY…
    ## $ is_local                     <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE,…
    ## $ track_name                   <chr> "California Love", "Slippin' Into Darknes…
    ## $ track_preview_url            <chr> "https://p.scdn.co/mp3-preview/93e456ef0b…
    ## $ track_number                 <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13…
    ## $ type                         <chr> "track", "track", "track", "track", "trac…
    ## $ track_uri                    <chr> "spotify:track:6ayeqYtOtwVhqVB6k6MKoh", "…
    ## $ external_urls.spotify        <chr> "https://open.spotify.com/track/6ayeqYtOt…
    ## $ album_name                   <chr> "California Love", "California Love", "Ca…
    ## $ key_name                     <chr> "C", "G#", "E", "F", "C", "D", "C#", "B",…
    ## $ mode_name                    <chr> "major", "major", "minor", "minor", "mino…
    ## $ key_mode                     <chr> "C major", "G# major", "E minor", "F mino…

### 3.2 Limpieza

#### 3.2.1 Removiendo duplicados

Se remueven las canciones duplicadas.

``` r
beats <- beats[!duplicated(beats$track_id),]
```

#### 3.2.2 Creando variables

Se crea la variable que mide la duración de la canción en minutos.

``` r
beats$duration_min <- beats$duration_ms / 60000
```

#### 3.2.3 Removiendo variables

Para el análisis que se busca, se necesitan las características de
sonido de la canción, su duración y su nombre.

Solo se conservaran estas variables y se removerá el resto.

``` r
beats <- beats %>% select(c(track_name,duration_min,8:18))
```

#### 3.2.4 Removiendo NA’s

Se observa que las variables seleccionadas están limpias y no presentan
NA´s.

No es necesario realizar limpieza.

``` r
colSums(is.na(beats))
```

    ##       track_name     duration_min     danceability           energy 
    ##                0                0                0                0 
    ##              key         loudness             mode      speechiness 
    ##                0                0                0                0 
    ##     acousticness instrumentalness         liveness          valence 
    ##                0                0                0                0 
    ##            tempo 
    ##                0

### 3.3 Descripción de los atributos

Cada canción presente en la base de datos contiene los siguientes
atributos:

**track_name** - Nombre de la canción.

**duration_min** - Duración en minutos.

**danceability** - Qué tan bailable es la canción, donde 0,0 es el menor
y 1,0 es el mayor.

**energy** - Que tanta intensidad y actividad representa, donde 0,0 es
el menor y 1,0 es el mayor.

**key** - Clave de la pista. 0 es Do, 1 es Do#, 2 es Re y así
sucesivamente. Cuando no presenta tonalidad es -1.

**loudness** - Qué tanto decibelios (dB) tiene la canción. Oscila entre
-60 y 0 dB.

**mode** - Contenido melódico de la canción. Mayor es 1 y Menor es 0.

**speechiness** - Presencia de palabras habladas. 1 es el mayor y 0 es
el menor.

**acousticness** - Acústica de la canción. 1 es acústica y 0 es no
acústica.

**instrumentalness** - Presencia de voces. 1 es instrumento y 0 es voz.

**liveness** - Detecta público en la grabación. Mayor valor es alta
presencia de público.

**valence** - Positividad de la canción. 1 es alegre y 0 es triste.

**tempo** - Pulsaciones por minuto (BPM).

    ## El dataframe contiene 445097 filas y 13 columnas.

# 4 Análisis exploratorio de datos (EDA)

### 4.1 Mapa de correlación

``` r
beats_atributos <- select(beats, 3:13)
corrplot(cor(beats_atributos))
```

![](Spotify_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

### 4.2 Histograma

``` r
beats_grafico <- beats_atributos %>% tidyr::gather(key = "variable", value = "valor")

ggplot(beats_grafico, aes(x = valor, fill = variable)) +
  geom_histogram(binwidth=0.25) +
  facet_wrap(~ variable, scales = "free") +
  scale_fill_hue() +
  guides(fill = "none") +
  theme(legend.position = "none")
```

![](Spotify_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

### 4.3 Boxplot

``` r
ggplot(beats_grafico, aes(x = variable, y = valor)) +
  geom_boxplot(aes(fill = variable)) +
  facet_wrap(~ variable, scales = "free") +
  scale_fill_hue() +
  guides(fill = "none") +
  theme(legend.position = "none")
```

![](Spotify_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

# 5 Construcción del modelo

### 5.1 K-Means
