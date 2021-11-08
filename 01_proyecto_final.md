# Proyecto Final BDs No Relacionales

Desarrollo de un análisis de lenguaje de propuestas de legislaciones.

## Narrativa

> SocialTIC y Article19 desean realizar un análisis de las propuestas legislativas de lo que va de este sexenio VS las del sexenio anterior para identificar cualquier patrón de uso de lenguaje.

## Fuentes de datos

Todo este proyecto final utilizará el [APILegislativo](https://backend.apilegislativo.com:5000/) para descargar iniciativas de ley y datos alrededor de ellas para su análisis.

## Producto final y estipulaciones

1. Comparación de conteos de propuestas de ley por por sexenio, por trimestre, por tópico o temática, por partido y sus combinaciones, con especial énfasis en comparativa durante el último año del sexenio de EPN y lo que va del sexenio de AMLO.
2. Análisis de [Term Frequency/Inverse Document Frequency](https://en.wikipedia.org/wiki/Tf%E2%80%93idf) de las propuestas de ley para detección de anomalías dentro de las propuestas.
3. En el bloque inicial debe haber 2 variables que deben automatizar el resto de la ejecución del notebook por parte de **CUALQUIER PERSONA QUE CLONE SU REPO**:
   - `id_propuesta_ley` de tipo entera en la que pondré cualquier ID de iniciativa dentro del APILegislativo para poder correr todo el análisis del punto 2 arriba.
   - `id_token` con el token que vamos a utilizar para interactuar con el APILegislativo.
4. Parte de **mi evaluación** de su trabajo va a ser que yo meta mi propio `id_token` y un `id_propuesta_ley` preseleccionados y todo el análisis en el punto 2 debe ejecutarse en automático. 
5. El resultado del TF/IDF deberá guardarse en AWS S3 **CADA VEZ QUE SE EJECUTE UN PROCESAMIENTO** de una propuesta de ley del punto 2.
6. El punto 1 será estático, sin necesidad de guardado o de procesamiento automático, y puede estar todo precalculado.

Debe entregarse un notebook en [Google Colab](https://colab.research.google.com/notebooks/intro.ipynb). Esto forzosamente es en Python.

## Deadlines y evaluación

### Presentación 1
- Fecha: 17 de Nov, 14:30
- Entregable: análisis preliminar estático de conteo de iniciativas por sexenio, por trimestre, por tópico o temática, por partido y sus combinaciones
- Valor: 10%
- No estará presente A19 ni SocialTIC

### Presentación 2
- Fecha: 1o de Dic, 14:30
- Entregable: el análisis preliminar TF/IDF de 3 iniciativas de ley de particular interés para el equipo y el guardado del resultado en AWS S3. No pueden ser las mismas de los 2 equipos.
- Valor: 20%
- No estará presente A19 ni SocialTIC

### Presentación final
- Fecha: 9 de Diciembre 7pm.
- Entregables:
   - Presentación ejecutiva de 40 min + sesión de preguntas de 20 min donde se comenten:
      - Análisis completo estático
      - Análisis TF/IDF completo
      - Guardado en AWS S3
   - Modificación por mi parte, en vivo, de las variables `id_token` y un `id_propuesta_ley` y ejecución de pipeline de adquisición de datos, análisis y guardado de resultados en AWS S3. 
- Valor: 50%
- En presencia de SocialTIC. A19 de forma remota.




