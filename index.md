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

1. 1er Parcial (teórico): 10%
2. 2o Parcial (avance proyecto final): 10%
3. 3er Parcial (avance proyecto final): 20%
4. Proyecto final: 50%
5. Tareas: 10%

### Cómo serán los exámenes?
Tendrán un componente teórico y/o un componente práctico.

El componente práctico consistirá generalmente en uno de a) crear o alterar una BD, b) diseñar una BD, c) generar datos en una BD con una cierta forma, o d)  generar un reporte analítico.

El componente teórico es un examen de opción múltiplemen la plataforma [Socrative](https://www.socrative.com/) en el cual podrás sacar apuntes o usar una o varias de las plataformas que configuraremos a lo largo del semestre (PostgreSQL, DBeaver, VSCode, Anaconda, etc).

### Y cómo será el examen final?
Será enteramente práctico y consistirá en el desarrollo de un proyecto integral con todo lo visto en el semestre. Daremos más detalles más adelante durante el curso.

### Cómo me contacto con ud, prof?
Usen el el correo institucional (jesus.ramos@itam.mx) o Slack, o escriban a ramos.cardona@gmail.com.

Pero preferentemente usen Slack.

### Cómo nos comunicaremos?
Por [Slack](https://slack.com), que es un chat orientado a equipos y trabajo. Comenzó como un juego tipo "Among Us", pero cuando hicieron el chat para el juego quedó tan chingón, que lo sacaron como producto separado. Abajo las ligas de descarga:
- [Windows](https://slack.com/downloads/windows)
- [Mac](https://slack.com/help/articles/207677868-Download-Slack-for-Mac)
- [Linux](https://slack.com/downloads/linux) - .rpm para Fedora o RedHat, .deb para Ubuntu

Una vez que descarguen Slack, hagan click [en esta liga](https://join.slack.com/t/nosql4ds-aug-dec-2021/shared_invite/zt-ty8xeun7-KGFTNAdK1D7ylLa5XnoZsg) para que sean automágicamente agregados a nuestro workspace.

### Por dónde serán las sesiones?

Lunes y Miércoles de 2:30pm a 4:00pm.

La sesión 0 será por [Teams de Microsoft](https://www.microsoft.com/es-mx/microsoft-365/microsoft-teams/download-app), y posteriormente nos moveremos a [Zoom](https://zoom.us/download), en [este link](https://itam.zoom.us/my/xuxoramos).

### Dónde estará el material?
Aquí en Github. Github es una plataforma de colaboración y control de versiones para no andar como este meme:
![](https://i.redd.it/05b6u19pseoz.png)

Es importante que si nunca has usado Github, o algún otro sistema de control de versiones, [leas esta guía](https://guides.github.com/activities/hello-world/) para que no te agarren en curva y tengas de menos los fundamentos de estas plataformas.

### Qué necesito para la clase?

1. Document - MongoDB - [Download](https://www.mongodb.com/try/download/community)
2. Graph - Neo4J - [Download](https://neo4j.com/download/)
3. Wide column - MonetDB - [Download](https://www.monetdb.org/Downloads)
3. Key-value store - Apache Ignite - [Download](https://ignite.apache.org/)
4. DAta lake - Spark - [Download](https://spark.apache.org/)
5. Cuenta de AWS - [Download](https://aws.amazon.com/free/?sc_icampaign=acq_aws_takeover-1st-visit-free-tier&sc_ichannel=ha&sc_icontent=awssm-evergreen-1st-visit&sc_iplace=hero&trk=ha_awssm-evergreen-1st-visit&all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all)

#### Solamente eso?

Dependiendo de como vengan de experiencia y las materias previas que hayan cursado, necesitarán:

1. VSCode - [Download](https://code.visualstudio.com/download)
2. Python (mediante miniconda) - [Download](https://docs.conda.io/en/latest/miniconda.html)
3. .NET SDK - [Download](https://dotnet.microsoft.com/download)

Ya veremos qué necesitaremos de todos estos.

### NEWS & UPDATES

Check back here often.

#### 2021-08-04
Bienvenidos!

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
