## Bases de Datos No-Relacionales para Ciencia de Datos

Este website contiene el material para la materia Bases de Datos No Relacionales (o NoSQL para brevedad) para Ciencia de Datos.

## Qué vamos a ver en esta materia?

Las BDs relacionales no son suficientes para contar toda la historia de un evento o de un objeto de negocio, de una orden, de un cliente, o de un envío. Frecuentemente la huella está dispersa en transacciones de una base de datos, visitas a unas páginas web, audios de atención a cliente, geolocalizaciones a lo largo de una ruta, texto libre en comentarios, grafos de redes sociales, y datos en memoria de los sistemas involucrados. Necesitamos saber interactuar con dichas bases de datos, y sobre todo, necesitamos poder consolidarlas en un repositorio central para lograr hacer analítica de todas las partes de la historia, sin dejar fuera ninguna fuente con la que corramos peligro de "quedar ciegos". En esta materia veremos como interactuar con dichas BDs no-relacionales (NoSQL = Not only SQL), y como construir dicho repositorio central.

En una frase, esta materia trata de **consolidación de datos** para contar una historia completa.

##   Rules of the game

### Qué texto vamos a usar?

1. [Seven databases in Seven Weeks](http://barbra-coco.dyndns.org/yuri/seven/seven2.pdf)
2. [Learning Spark](https://pages.databricks.com/rs/094-YMS-629/images/LearningSpark2.0.pdf)
3. [Apuntes de miguelito](https://drive.google.com/file/d/1yS4RXly7kGCKBjklfX11dwK-fhvMpm-s/view?usp=sharing)


### Cómo vamos a calificar?

1. 1er y único parcial individual: 60%
2. 1er avance proyecto final: 20%
3. 2o y último avance proyecto final: 20%

### Cómo serán los exámenes?
Tendrán un componente teórico y/o un componente práctico.

El componente práctico consistirá generalmente en uno de a) crear o alterar una BD, b) diseñar una BD, c) generar datos en una BD con una cierta forma, o d)  generar un reporte analítico.

El componente teórico es un examen de opción múltiplemen la plataforma [Socrative](https://www.socrative.com/) en el cual podrás sacar apuntes o usar una o varias de las plataformas que configuraremos a lo largo del semestre (PostgreSQL, DBeaver, VSCode, Anaconda, etc).

### Y cómo será el examen final?
Será enteramente práctico y consistirá en el desarrollo de un proyecto integral con todo lo visto en el semestre. Daremos más detalles más adelante durante el curso.

### Cómo me contacto con ud, prof?
Usen el el correo institucional (jesus.ramos@itam.mx) o Slack.

Pero preferentemente usen Slack.

### Cómo nos comunicaremos?
Por [Slack](https://slack.com). Abajo las ligas de descarga:
- [Windows](https://slack.com/downloads/windows)
- [Mac](https://slack.com/help/articles/207677868-Download-Slack-for-Mac)
- [Linux](https://slack.com/downloads/linux) - .rpm para Fedora o RedHat, .deb para Ubuntu

Una vez que descarguen Slack, hagan click [en esta liga](https://join.slack.com/t/slack-phh3413/shared_invite/zt-12jco12g3-XkoahXqMXkx4mdNiIi2VUg) para que sean automágicamente agregados a nuestro workspace.

### Por dónde serán las sesiones?

Viernes de 8AM a 11AM.

La sesión 1 será por [Zoom](https://zoom.us/download), en [este link](https://itam.zoom.us/my/xuxoramos), y solo durará de 8 a 10.

A partir de la 2a sesión estaremos presenciales en el salón **RHCC302** de 8 a 11.

### Dónde estará el material?
Aquí en Github. Es importante que si nunca has usado Github, o algún otro sistema de control de versiones, [leas esta guía](https://guides.github.com/activities/hello-world/) para que no te agarren en curva y tengas de menos los fundamentos de estas plataformas.

### Qué necesito para la clase?

1. Document - MongoDB - [Download](https://www.mongodb.com/try/download/community)
2. Graph - Neo4J - [Download](https://neo4j.com/download/)
3. Wide column - MonetDB - [Download](https://www.monetdb.org/Downloads)
4. Data lake - AWS Lake Formation - [Download](https://spark.apache.org/)
5. Cuenta de AWS - Suministrada por mi

#### Solamente eso?

Dependiendo de como vengan de experiencia y las materias previas que hayan cursado, necesitarán:

1. VSCode - [Download](https://code.visualstudio.com/download)
2. Python (mediante miniconda) - [Download](https://docs.conda.io/en/latest/miniconda.html)

### NEWS & UPDATES

Check back here often.

### Temario + Fechas = Plan de Materia
A continuación el temario, fecha por fecha:

| Num de sesión | Fecha de sesión | Temas |
|---------------|-----------------|-------|
| 1             | 2021/08/04      | Intro / 2-speed IT ([apunte](https://drive.google.com/file/d/1yS4RXly7kGCKBjklfX11dwK-fhvMpm-s/view?usp=sharing)/[video](https://drive.google.com/file/d/1ieoOT-bMrxuAFETPJRegX0bBlerJRWXd/view?usp=sharing))
| 2             | 2021/08/09      | 2-speed IT / Data Warehouse VS Data Lake ([apunte](https://drive.google.com/file/d/1yS4RXly7kGCKBjklfX11dwK-fhvMpm-s/view?usp=sharing)/[video](https://drive.google.com/file/d/1l6x1-kBVJ2J7OhMVgSCPrmdQn7ze_oQ_/view?usp=sharing))
| 3             | 2021/08/11      | BDs analíticas VS BDs transaccionales / BDs columnares VS BDs relacionales ([apunte](https://drive.google.com/file/d/1yS4RXly7kGCKBjklfX11dwK-fhvMpm-s/view?usp=sharing)/[video](https://drive.google.com/file/d/1uurA9_b-LqRqZOrSPzbCWqDMDmsxYhjq/view?usp=sharing))
| 4             | 2021/08/16      | Revisión de [tarea 1](https://docs.google.com/spreadsheets/d/1vAfKm4FhWRxNLE7i9uCAhLwyY-GBkY-Z/edit?usp=sharing&ouid=112731314315641508878&rtpof=true&sd=true), Showcase de [herramienta de desduplicación](https://dedupe.io/), Instalación de [MongoDB y accesorios](https://xuxoramos.github.io/nosql-4-ds/01_mongodb) / [video](https://drive.google.com/file/d/1PsGHNzYT7OPvNlR0zdvIM9RQX_tdr7bd/view?usp=sharing)
| 5             | 2021/08/18      | **CLASE A REPONER** |
| 6             | 2021/08/23      | Funciones `insert` y `find` en [MongoDB](https://xuxoramos.github.io/nosql-4-ds/01_mongodb) / [video](https://drive.google.com/file/d/163qpEwLeXLFY1_17gx-xCLfRjH3NA_0p/view?usp=sharing)
| 7             | 2021/08/25      | [Más de la función `find`](https://xuxoramos.github.io/nosql-4-ds/01_mongodb) / [video](https://drive.google.com/file/d/1uicMJ_S5rMgZWhBYajXHJAMwOkMvfLiy/view?usp=sharing)
| 8             | 2021/08/30      | [La función `find` sobre arrays de elementos y arrays de diccionarios/objetos](https://xuxoramos.github.io/nosql-4-ds/01_mongodb) / [video](https://drive.google.com/file/d/10vUhMDTksjWHVQzV2uljwfw8zyF2B97W/view?usp=sharing)
| 9             | 2021/09/01      | [`find()` sobre arrays de objetos/diccionarios, los operadores `$elemMatch` y `$slice`, y ejercicios](https://xuxoramos.github.io/nosql-4-ds/01_mongodb) / [video](https://drive.google.com/file/d/1Ue5E5tQjNdHr5KqTnUT1H_ayaDKpyGj7/view?usp=sharing)
| 10             | 2021/09/06      | [Revisión y solución de ejercicios](https://xuxoramos.github.io/nosql-4-ds/01_mongodb) / [video](https://drive.google.com/file/d/1ZegIut_kc4C1DxFBpxwkA9-yZvlaaZkv/view?usp=sharing)
| 11             | 2021/09/08      | **CLASE A REPONER**
| 12             | 2021/09/13      | [Aggregations, pipelines, grouping](https://xuxoramos.github.io/nosql-4-ds/01_mongodb) / [video](https://drive.google.com/file/d/1mpZTq96WXuJV47VJ8OPHKczbrzrUIddd/view?usp=sharing)
| 13             | 2021/09/15      | [Grouping, funciones de fecha, agregadores (count, avg, stdev, sum)](https://xuxoramos.github.io/nosql-4-ds/01_mongodb) / [video](https://drive.google.com/file/d/1wq3iDtXWOkXy1pDBP-ZV2Lyx65rQBjUe/view?usp=sharing)
| 5             | **Sesión de reposición del 2021/08/18**      | [API Legislativo,  MongoDB y nuestro 1er ETL](https://xuxoramos.github.io/nosql-4-ds/01_mongodb) / [video](https://drive.google.com/file/d/1gVCyjQOJ1mHX7_yykdLRI8mhRA_cC8Lx/view?usp=sharing)
| 15             |   2021/09/20    | [Stages `$mergeObjects` y `$lookup`. Ejercicios integradores. Tarea 2](https://xuxoramos.github.io/nosql-4-ds/01_mongodb) / [video](https://drive.google.com/file/d/1kIU8YQlX5SqxfdN2JP0d34q_e8ExIYel/view?usp=sharing)
| 16             |   2021/09/22    | [Presentación del APILegislativo](https://drive.google.com/file/d/19sXSr-qu9qqpIpM7Kr8DNoCvAUcn9GFP/view?usp=sharing) / [video](https://drive.google.com/file/d/1-syp_3A1cD-kOOAUhP1Bl_Cox6m9QxxU/view?usp=sharing)
| 17             |   2021/09/27    | **CLASE A REPONER**
| 18             |   2021/09/29    | [Ejercicios](https://xuxoramos.github.io/nosql-4-ds/01_mongodb) / [video](https://drive.google.com/file/d/16_L00-V1JQLs1T0o2wnbKHxxU4GPfwu9/view?usp=sharing)
| 19             |   2021/10/04    | [Intro a BDs Columnares](https://xuxoramos.github.io/nosql-4-ds/01_monetdb) / [video](https://drive.google.com/file/d/1KioyeEsxjtJybviuhYFSvxV5EfUsOoit/view?usp=sharing)
| 20             |   2021/10/06    | [Instalación de MonetDB](https://xuxoramos.github.io/nosql-4-ds/01_monetdb) / [video](https://drive.google.com/file/d/1TqoF-OfzUQpXSpKx3wS8AYHlKmIKI1rM/view?usp=sharing)
| 21             |   2021/10/11    | **CLASE A REPONER**
| 22             |   2021/10/13    | [Comparativa de desempeño EN VIVO entre PostgreSQL y MonetDB con base de datos de Ecobici](https://xuxoramos.github.io/nosql-4-ds/01_monetdb) / [video](https://drive.google.com/file/d/1nVO7-nZKtSkp6oz2TvW-OLV_XcUX1q5v/view?usp=sharing)
| 23             |   2021/10/18    | [Construcción de un Data Warehouse Histórico en MonetDB y migración de datos desde PostgreSQL](https://xuxoramos.github.io/nosql-4-ds/01_monetdb) / [video](https://drive.google.com/file/d/1k-c_QXwZloKJReCVhsQ7AOPKHWiu0cYc/view?usp=sharing)
| 24             |   2021/10/20    | [Intro a Cloud Computing y creación de una máquina virtual en AWS](https://xuxoramos.github.io/nosql-4-ds/01_neo4j) / [video](https://drive.google.com/file/d/19hobazsdMgyCyrg_9Y3Hk7c8VIVMa607/view?usp=sharing)
| 25             |   2021/10/25    | [Uso de AWS Academy. Creación de máquinas virtuales en EC2 dentro de AWS Academy. Instalación de Neo4j.](https://xuxoramos.github.io/nosql-4-ds/01_neo4j) / [video](https://drive.google.com/file/d/1lLO5h5GX32Z2c2ZzdiXC4vcejTl-jnUw/view?usp=sharing)
| 26             |   2021/10/27    | **CLASE A REPONER**
| 27             |   2021/11/01    | **ASUETO**
| 28             |   2021/11/03    | [Conexión a Neo4j. Carga de BD Northwind a Neo4j. Similitudes y diferencias entre Neo4j y SQL. Intro al lenguaje Cypher.](https://xuxoramos.github.io/nosql-4-ds/01_neo4j) / [video](https://drive.google.com/file/d/1sJkytduTJHxQbpFzmdSLi5EZNv5cI9hj/view?usp=sharing)
| 29             |   2021/11/08    | [Ejercicios de queries con Cypher a la BD Northwind. Intro a Proyecto Final.](https://xuxoramos.github.io/nosql-4-ds/01_neo4j) / [video](https://drive.google.com/file/d/15Qaoikj9A0xC4vBCotetb4JfvKvDopAJ/view?usp=sharing)
| 30             |   2021/11/10    | [Ejercicios de queries con Cypher a la BD Northwind II. Carga de los Pandora Papers. Análisis avanzado de grafos: centralidad y similitud. Grafos para law enforcement](https://xuxoramos.github.io/nosql-4-ds/01_neo4j) / [video](https://drive.google.com/file/d/1mCbechLS_-4aQc6J1Dd9s8Qgiu-aoKrL/view?usp=sharing)
| 31             |   2021/11/15    | **CLASE A REPONER**
| 32             |   2021/11/17    | Presentaciones del 1er avance del proyecto final / [video](https://zoom.us/rec/play/2izcLUDmBHU7YREltYmlqwZfkzN6vpOZsMBPC_mof-nJaWvlYWdFAwt6heXExBgYMhap2QMqdYLb45tC.UWNTuyMTOMmfBK24?autoplay=true&startTime=1637181266000)
| 33             |   2021/11/22    | **CLASE A REPONER**
| 34             |   2021/11/24    | Sesión de Paco Mekler (no fue posible grabarla porque la información presentada está protegida por NDAs)
| 35             |   2021/11/29    | [Creación y operación de un Data Lake con AWS](https://xuxoramos.github.io/nosql-4-ds/01_datalake) / [video](https://zoom.us/rec/play/4ntH6Nbc-799LaexFfWEHvLyw7-E4wSlWlORSpNnym2ilKAxPvRlISD5efcIvhQ-b5vajEYY9NujDOlC.QIBKLNMBFtBH-0df?autoplay=true&startTime=1638218040000)
