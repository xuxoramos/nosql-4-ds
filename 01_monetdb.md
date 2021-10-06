# MonetDB

MonetDB es una base de datos columnar.

Recordemos la arquitectura completa de una solución de data lake:

![image](https://user-images.githubusercontent.com/1316464/135890170-a36f5f83-61df-44d6-9012-ccb5c2762920.png)

Ya hemos visto que MongoDB está generalmente entre una API y nuestro lake, e hicimos nuestro 1er ETL donde extrajimos datos del APILegislativo y los metimos en nuestro MongoDB de nuestra infraestructura (de momento, local).

Ahora veremos MonetDB. MonetDB es una BD columnar.

Las BDs columnares son buenas para leer, malonas para escribir.

Las BDs columnares generalmente son el destino final de "cubos de datos"

Los cubos de datos son como las vistas.

Recuerdan la BD de Sakila del semestre pasado?

![image](https://user-images.githubusercontent.com/1316464/135899346-9bc04e14-b07f-4924-98a7-59e4c01a8896.png)

Recuerdan que cuando tenemos queries analíticos complejos, o que generan mucha carga en la BD, recurríamos a las "vistas"?

Bueno...

➡️**Cuando las BDs relacionales no son suficientes (ni con índices), las vistas las llevamos a BDs columnares.**

## Cómo funcionan?

Recordemos el siguiente apunte:

![image](https://user-images.githubusercontent.com/1316464/135899287-45d16a89-ea83-487f-b1fb-311da1397579.png)

Vamos a utilizar JSON para representar el 1er registro de la tabla `film` y `category` de la BD Sakila:

```javascript
film: {
  film_id:1,
  title:"ACADEMY DINOSAUR",
  description:"A Epic Drama of a Feminist And a Mad Scientist who must Battle a Teacher in The Canadian Rockies",
  release_year:2006,
  language_id:1,
  original_language_id:null,
  rental_duration:6,
  rental_rate:0.99,
  length:86,
  replacement_cost:20.99,
  rating:{"PG"},
  last_update:"2006-02-15 05:03:42",
  special_features:{"Deleted Scenes","Behind the Scenes"},
}

category:[
  {category_id:1,  name:"Action"
  {category_id:2,  name:"Animation"},
  {category_id:3,  name:"Children"},
  {category_id:4,  name:"Classics"},
  {category_id:5,  name:"Comedy"}
]
```

Si quisiéramos obtener el promedio de `length` de todo nuestro catálogo de películas, tendríamos que recorrer renglón por renglón, registro por registro, acumulando y al final dividiendo para obtener el promedio.

Aunque no estamos viajando a otra tabla debido a un `join` o algo así, de todos modos recorrer toda la tabla amerita un **full table scan**, que es de las operaciones más costosas en BD, con una running time de _O(N)_.

Si agregamos un `join` al problema, por ejemplo, si queremos sacar el num de actores por categoría de película, tendremos que hacer lo siguiente:

```sql
select c."name" , count(a.actor_id)
from actor a join film_actor fa using (actor_id)
join film f using (film_id)
join film_category fc using (film_id)
join category c using (category_id)
group by c."name";
```

Si representáramos en JSON estos viajes, terminaríamos con un documento bastante complejo, y dado que no tenemos `where`, tendríamos que recorrer tabla por tabla, renglón por renglón, _matcheando_ tanto la condición del `join` como las otras condiciones que tuviéramos, alrededor de _O(N)*M tablas_.

Ahora imaginemos en JSON, **en formato columnar**, las tablas 

```javascript
film_ids:[
  1,2,3,4,5,6,7,8,9,10
]
film_titles:[
  "ACADEMY DINOSAUR",
  "ACE GOLDFINGER",
  "ADAPTATION HOLES",
  "AFFAIR PREJUDICE",
  "AFRICAN EGG",
  "AGENT TRUMAN",
  "AIRPLANE SIERRA",
  "AIRPORT POLLOCK",
  "ALABAMA DEVIL",
  "ALADDIN CALENDAR"
]
film_lengths:[
  86,
  48,
  50,
  117,
  130,
  169,
  62,
  54,
  114,
  63
]
film_rental_rate:[
  0.99,
  4.99,
  2.99,
  2.99,
  2.99,
  2.99,
  4.99,
  4.99,
  2.99,
  4.99
]
```

PERO luego las BDs columnares implementan unos **algoritmos de compresión** para reducir estas estructuras a algo como esto:

```javascript
film_lengths_values:[
  86,
  48,
  50,
  117,
  130,
  169,
  62,
  54,
  114,
  63
]
film_lengths_keys:[
  1,2,3,4,5,6,7,8,9,10
]
film_rental_keys:[
  1,
  2,
  3,
  3,
  3,
  3,
  2,
  2,
  3,
  2
]
film_rental_values:[
  0.99,
  4.99,
  2.99
]
```

Teniendo este tipo de estructuras, entonces los queries analíticos como el de abajo, aunque deben hacer igual un _full table scan_, el running time es _O(N)*M_, donde la _N_ es un num de registros muchísimo más reducido que en una BD relacional, y el num de tablas involucradas _M_ también podemos mantenerla reducida.

```sql
select avg(f.rental_rate) from film f;
```

## Cómo se construyen las BDs columnares?

Por dentro, las BDs columnares son **archivos** que se construyen de la siguiente forma:

![image](https://user-images.githubusercontent.com/1316464/135909033-c8418a9e-3db1-4972-9335-158bf34f7c70.png)

Cada par de `ID` y valor es 1 solo archivo.

Posterior a la creación de estos archivos, viene la **compresión**.

Todos los algoritmos de compresión computacionales consiste en hacer una tabla de pares **llave-valor**, y los valores repetidos reemplazarlos por las llaves.

Idóneamente, las llaves serán enteros pequeñísimos, usualmente 2 a 4 bits, por lo que su búsqueda terminará siendo eficiente, y ocuparán mucho menos espacio que el dato real legible para los humanos (8 bits mínimo).

Por ejemplo, el algoritmo _zip_:

![image](https://user-images.githubusercontent.com/1316464/135909796-1323dacd-fd89-47a2-9965-a41b5e0d76ed.png)


En general, entre más secuencias de caracteres repetidos tengamos en un archivo de texto, más eficiente será la compresión.

Por eso comprimir archivos binarios no sale tan eficiente.

Igual con las BDs columnares.

Comprimir una sola tabla con columnas con poca varianza resultará en eficiencias modestas en los queries analíticos.

Por eso en lugar de meter tabla por tabla a una BD columnar, mejor insertamos una _Big Table_!

⚠️**Y es por esta misma razón que en las BDs columnares las operaciones de INSERT, DELETE o UPDATE o no son soportadas, o son parcialmente soportadas, o tardan muchísimo más que en una BD relacional**⚠️

## Qué puedo poner en una columnar?

Estas son las formas de BDs que podemos meter en una BD columnar. Todos estos son "esquemas de base de datos para data warehousing", es decir, formas de acomodar los datos resultado de un querisote analítico para su almacenamiento y consulta constante.

**Esquema de estrella**

![image](https://user-images.githubusercontent.com/1316464/135910788-f9247909-3925-4d33-8c0e-977429ced497.png)


**Esquema de snowflake**

![image](https://user-images.githubusercontent.com/1316464/135910871-e1c5bf95-dea0-40c8-be48-a90a47017e37.png)


**Esquema de Big Table**

![image](https://user-images.githubusercontent.com/1316464/135910938-f36c9c5d-81f9-434d-811e-dd9b24442e62.png)

Dado el abaratamiento del storage y el poder de cómputo, el esquema preferido para hacer un datawarehouse con una BD columnar es el esquema de Big Table.

Cómo podemos construir una Big Table desde una BD relacional?

```sql
select *
from actor a join film_actor fa using (actor_id)
join film f using (film_id)
join film_category fc using (film_id)
join category c using (category_id)
join inventory i using (film_id)
join rental r using (inventory_id)
join payment p using (rental_id);
```

Este query está haciendo join en 7 tablas, y forzosamente, al hacer join, dadas las cardinalidades de las relaciones, tendremos datos repetidos, y por tanto se vuelve candidato perfecto para formar un esquema _Big Table_ y por tanto, se volverá bastante eficiente guardarlo en una BD columnar.

```
rental_id|inventory_id|film_id|category_id|actor_id|first_name |last_name   |last_update        |last_update        |title                 |description                                                                                                         |release_year|language_id|original_language_id|rental_duration|rental_rate|length|replacement_cost|rating|last_update        |special_features                                        |fulltext                                                                                                                                                                            |last_update        |name       |last_update        |store_id|last_update        |rental_date        |customer_id|return_date        |staff_id|last_update        |payment_id|customer_id|staff_id|amount|payment_date       |
---------+------------+-------+-----------+--------+-----------+------------+-------------------+-------------------+----------------------+--------------------------------------------------------------------------------------------------------------------+------------+-----------+--------------------+---------------+-----------+------+----------------+------+-------------------+--------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------+-----------+-------------------+--------+-------------------+-------------------+-----------+-------------------+--------+-------------------+----------+-----------+--------+------+-------------------+
       76|        3021|    663|          4|      90|SEAN       |GUINESS     |2006-02-15 04:34:33|2006-02-15 05:05:03|PATIENT SISTER        |A Emotional Epistle of a Squirrel And a Robot who must Confront a Lumberjack in Soviet Georgia                      |        2006|          1|                    |              7|       0.99|    99|           29.99|NC-17 |2006-02-15 05:03:42|{Trailers,Commentaries}                                 |'confront':14 'emot':4 'epistl':5 'georgia':19 'lumberjack':16 'must':13 'patient':1 'robot':11 'sister':2 'soviet':18 'squirrel':8                                                 |2006-02-15 05:07:09|Classics   |2006-02-15 04:46:27|       2|2006-02-15 05:09:17|2005-05-25 11:30:37|          1|2005-06-03 12:00:37|       2|2006-02-15 21:30:53|         1|          1|       1|  2.99|2005-05-25 11:30:37|
       76|        3021|    663|          4|      74|MILLA      |KEITEL      |2006-02-15 04:34:33|2006-02-15 05:05:03|PATIENT SISTER        |A Emotional Epistle of a Squirrel And a Robot who must Confront a Lumberjack in Soviet Georgia                      |        2006|          1|                    |              7|       0.99|    99|           29.99|NC-17 |2006-02-15 05:03:42|{Trailers,Commentaries}                                 |'confront':14 'emot':4 'epistl':5 'georgia':19 'lumberjack':16 'must':13 'patient':1 'robot':11 'sister':2 'soviet':18 'squirrel':8                                                 |2006-02-15 05:07:09|Classics   |2006-02-15 04:46:27|       2|2006-02-15 05:09:17|2005-05-25 11:30:37|          1|2005-06-03 12:00:37|       2|2006-02-15 21:30:53|         1|          1|       1|  2.99|2005-05-25 11:30:37|
       76|        3021|    663|          4|      37|VAL        |BOLGER      |2006-02-15 04:34:33|2006-02-15 05:05:03|PATIENT SISTER        |A Emotional Epistle of a Squirrel And a Robot who must Confront a Lumberjack in Soviet Georgia                      |        2006|          1|                    |              7|       0.99|    99|           29.99|NC-17 |2006-02-15 05:03:42|{Trailers,Commentaries}                                 |'confront':14 'emot':4 'epistl':5 'georgia':19 'lumberjack':16 'must':13 'patient':1 'robot':11 'sister':2 'soviet':18 'squirrel':8                                                 |2006-02-15 05:07:09|Classics   |2006-02-15 04:46:27|       2|2006-02-15 05:09:17|2005-05-25 11:30:37|          1|2005-06-03 12:00:37|       2|2006-02-15 21:30:53|         1|          1|       1|  2.99|2005-05-25 11:30:37|
       76|        3021|    663|          4|      20|LUCILLE    |TRACY       |2006-02-15 04:34:33|2006-02-15 05:05:03|PATIENT SISTER        |A Emotional Epistle of a Squirrel And a Robot who must Confront a Lumberjack in Soviet Georgia                      |        2006|          1|                    |              7|       0.99|    99|           29.99|NC-17 |2006-02-15 05:03:42|{Trailers,Commentaries}                                 |'confront':14 'emot':4 'epistl':5 'georgia':19 'lumberjack':16 'must':13 'patient':1 'robot':11 'sister':2 'soviet':18 'squirrel':8                                                 |2006-02-15 05:07:09|Classics   |2006-02-15 04:46:27|       2|2006-02-15 05:09:17|2005-05-25 11:30:37|          1|2005-06-03 12:00:37|       2|2006-02-15 21:30:53|         1|          1|       1|  2.99|2005-05-25 11:30:37|
      573|        4020|    875|         15|     142|JADA       |RYDER       |2006-02-15 04:34:33|2006-02-15 05:05:03|TALENTED HOMICIDE     |A Lacklusture Panorama of a Dentist And a Forensic Psychologist who must Outrace a Pioneer in A U-Boat              |        2006|          1|                    |              6|       0.99|   173|            9.99|PG    |2006-02-15 05:03:42|{Commentaries,Deleted Scenes,Behind the Scenes}         |'boat':22 'dentist':8 'forens':11 'homicid':2 'lacklustur':4 'must':14 'outrac':15 'panorama':5 'pioneer':17 'psychologist':12 'talent':1 'u':21 'u-boat':20                        |2006-02-15 05:07:09|Sports     |2006-02-15 04:46:27|       2|2006-02-15 05:09:17|2005-05-28 10:35:23|          1|2005-06-03 06:32:23|       1|2006-02-15 21:30:53|         2|          1|       1|  0.99|2005-05-28 10:35:23|
      573|        4020|    875|         15|     131|JANE       |JACKMAN     |2006-02-15 04:34:33|2006-02-15 05:05:03|TALENTED HOMICIDE     |A Lacklusture Panorama of a Dentist And a Forensic Psychologist who must Outrace a Pioneer in A U-Boat              |        2006|          1|                    |              6|       0.99|   173|            9.99|PG    |2006-02-15 05:03:42|{Commentaries,Deleted Scenes,Behind the Scenes}         |'boat':22 'dentist':8 'forens':11 'homicid':2 'lacklustur':4 'must':14 'outrac':15 'panorama':5 'pioneer':17 'psychologist':12 'talent':1 'u':21 'u-boat':20                        |2006-02-15 05:07:09|Sports     |2006-02-15 04:46:27|       2|2006-02-15 05:09:17|2005-05-28 10:35:23|          1|2005-06-03 06:32:23|       1|2006-02-15 21:30:53|         2|          1|       1|  0.99|2005-05-28 10:35:23|
      573|        4020|    875|         15|      85|MINNIE     |ZELLWEGER   |2006-02-15 04:34:33|2006-02-15 05:05:03|TALENTED HOMICIDE     |A Lacklusture Panorama of a Dentist And a Forensic Psychologist who must Outrace a Pioneer in A U-Boat              |        2006|          1|                    |              6|       0.99|   173|            9.99|PG    |2006-02-15 05:03:42|{Commentaries,Deleted Scenes,Behind the Scenes}         |'boat':22 'dentist':8 'forens':11 'homicid':2 'lacklustur':4 'must':14 'outrac':15 'panorama':5 'pioneer':17 'psychologist':12 'talent':1 'u':21 'u-boat':20                        |2006-02-15 05:07:09|Sports     |2006-02-15 04:46:27|       2|2006-02-15 05:09:17|2005-05-28 10:35:23|          1|2005-06-03 06:32:23|       1|2006-02-15 21:30:53|         2|          1|       1|  0.99|2005-05-28 10:35:23|
      573|        4020|    875|         15|      44|NICK       |STALLONE    |2006-02-15 04:34:33|2006-02-15 05:05:03|TALENTED HOMICIDE     |A Lacklusture Panorama of a Dentist And a Forensic Psychologist who must Outrace a Pioneer in A U-Boat              |        2006|          1|                    |              6|       0.99|   173|            9.99|PG    |2006-02-15 05:03:42|{Commentaries,Deleted Scenes,Behind the Scenes}         |'boat':22 'dentist':8 'forens':11 'homicid':2 'lacklustur':4 'must':14 'outrac':15 'panorama':5 'pioneer':17 'psychologist':12 'talent':1 'u':21 'u-boat':20                        |2006-02-15 05:07:09|Sports     |2006-02-15 04:46:27|       2|2006-02-15 05:09:17|2005-05-28 10:35:23|          1|2005-06-03 06:32:23|       1|2006-02-15 21:30:53|         2|          1|       1|  0.99|2005-05-28 10:35:23|
      573|        4020|    875|         15|      36|BURT       |DUKAKIS     |2006-02-15 04:34:33|2006-02-15 05:05:03|TALENTED HOMICIDE     |A Lacklusture Panorama of a Dentist And a Forensic Psychologist who must Outrace a Pioneer in A U-Boat              |        2006|          1|                    |              6|       0.99|   173|            9.99|PG    |2006-02-15 05:03:42|{Commentaries,Deleted Scenes,Behind the Scenes}         |'boat':22 'dentist':8 'forens':11 'homicid':2 'lacklustur':4 'must':14 'outrac':15 'panorama':5 'pioneer':17 'psychologist':12 'talent':1 'u':21 'u-boat':20                        |2006-02-15 05:07:09|Sports     |2006-02-15 04:46:27|       2|2006-02-15 05:09:17|2005-05-28 10:35:23|          1|2005-06-03 06:32:23|       1|2006-02-15 21:30:53|         2|          1|       1|  0.99|2005-05-28 10:35:23|
     1185|        2785|    611|          4|     152|BEN        |HARRIS      |2006-02-15 04:34:33|2006-02-15 05:05:03|MUSKETEERS WAIT       |A Touching Yarn of a Student And a Moose who must Fight a Mad Cow in Australia                                      |        2006|          1|                    |              7|       4.99|    73|           17.99|PG    |2006-02-15 05:03:42|{Deleted Scenes,Behind the Scenes}                      |'australia':19 'cow':17 'fight':14 'mad':16 'moos':11 'musket':1 'must':13 'student':8 'touch':4 'wait':2 'yarn':5                                                                  |2006-02-15 05:07:09|Classics   |2006-02-15 04:46:27|       1|2006-02-15 05:09:17|2005-06-15 00:54:12|          1|2005-06-23 02:42:12|       2|2006-02-15 21:30:53|         3|          1|       1|  5.99|2005-06-15 00:54:12|
     1422|        1021|    228|          4|     186|JULIA      |ZELLWEGER   |2006-02-15 04:34:33|2006-02-15 05:05:03|DETECTIVE VISION      |A Fanciful Documentary of a Pioneer And a Woman who must Redeem a Hunter in Ancient Japan                           |        2006|          1|                    |              4|       0.99|   143|           16.99|PG-13 |2006-02-15 05:03:42|{Trailers,Commentaries,Behind the Scenes}               |'ancient':18 'detect':1 'documentari':5 'fanci':4 'hunter':16 'japan':19 'must':13 'pioneer':8 'redeem':14 'vision':2 'woman':11                                                    |2006-02-15 05:07:09|Classics   |2006-02-15 04:46:27|       2|2006-02-15 05:09:17|2005-06-15 18:02:53|          1|2005-06-19 15:54:53|       2|2006-02-15 21:30:53|         4|          1|       2|  0.99|2005-06-15 18:02:53|
     1422|        1021|    228|          4|     144|ANGELA     |WITHERSPOON |2006-02-15 04:34:33|2006-02-15 05:05:03|DETECTIVE VISION      |A Fanciful Documentary of a Pioneer And a Woman who must Redeem a Hunter in Ancient Japan                           |        2006|          1|                    |              4|       0.99|   143|           16.99|PG-13 |2006-02-15 05:03:42|{Trailers,Commentaries,Behind the Scenes}               |'ancient':18 'detect':1 'documentari':5 'fanci':4 'hunter':16 'japan':19 'must':13 'pioneer':8 'redeem':14 'vision':2 'woman':11                                                    |2006-02-15 05:07:09|Classics   |2006-02-15 04:46:27|       2|2006-02-15 05:09:17|2005-06-15 18:02:53|          1|2005-06-19 15:54:53|       2|2006-02-15 21:30:53|         4|          1|       2|  0.99|2005-06-15 18:02:53|
     1422|        1021|    228|          4|      94|KENNETH    |TORN        |2006-02-15 04:34:33|2006-02-15 05:05:03|DETECTIVE VISION      |A Fanciful Documentary of a Pioneer And a Woman who must Redeem a Hunter in Ancient Japan                           |        2006|          1|                    |              4|       0.99|   143|           16.99|PG-13 |2006-02-15 05:03:42|{Trailers,Commentaries,Behind the Scenes}               |'ancient':18 'detect':1 'documentari':5 'fanci':4 'hunter':16 'japan':19 'must':13 'pioneer':8 'redeem':14 'vision':2 'woman':11                                                    |2006-02-15 05:07:09|Classics   |2006-02-15 04:46:27|       2|2006-02-15 05:09:17|2005-06-15 18:02:53|          1|2005-06-19 15:54:53|       2|2006-02-15 21:30:53|         4|          1|       2|  0.99|2005-06-15 18:02:53|
     1476|        1407|    308|          5|      31|SISSY      |SOBIESKI    |2006-02-15 04:34:33|2006-02-15 05:05:03|FERRIS MOTHER         |A Touching Display of a Frisbee And a Frisbee who must Kill a Girl in The Gulf of Mexico                            |        2006|          1|                    |              3|       2.99|   142|           13.99|PG    |2006-02-15 05:03:42|{Trailers,Deleted Scenes,Behind the Scenes}             |'display':5 'ferri':1 'frisbe':8,11 'girl':16 'gulf':19 'kill':14 'mexico':21 'mother':2 'must':13 'touch':4                                                                        |2006-02-15 05:07:09|Comedy     |2006-02-15 04:46:27|       1|2006-02-15 05:09:17|2005-06-15 21:08:46|          1|2005-06-25 02:26:46|       1|2006-02-15 21:30:53|         5|          1|       2|  9.99|2005-06-15 21:08:46|
     1725|         726|    159|          5|     199|JULIA      |FAWCETT     |2006-02-15 04:34:33|2006-02-15 05:05:03|CLOSER BANG           |A Unbelieveable Panorama of a Frisbee And a Hunter who must Vanquish a Monkey in Ancient India                      |        2006|          1|                    |              5|       4.99|    58|           12.99|R     |2006-02-15 05:03:42|{Trailers,Behind the Scenes}                            |'ancient':18 'bang':2 'closer':1 'frisbe':8 'hunter':11 'india':19 'monkey':16 'must':13 'panorama':5 'unbeliev':4 'vanquish':14                                                    |2006-02-15 05:07:09|Comedy     |2006-02-15 04:46:27|       1|2006-02-15 05:09:17|2005-06-16 15:18:57|          1|2005-06-17 21:05:57|       1|2006-02-15 21:30:53|         6|          1|       1|  4.99|2005-06-16 15:18:57|
     1725|         726|    159|          5|     179|ED         |GUINESS     |2006-02-15 04:34:33|2006-02-15 05:05:03|CLOSER BANG           |A Unbelieveable Panorama of a Frisbee And a Hunter who must Vanquish a Monkey in Ancient India                      |        2006|          1|                    |              5|       4.99|    58|           12.99|R     |2006-02-15 05:03:42|{Trailers,Behind the Scenes}                            |'ancient':18 'bang':2 'closer':1 'frisbe':8 'hunter':11 'india':19 'monkey':16 'must':13 'panorama':5 'unbeliev':4 'vanquish':14                                                    |2006-02-15 05:07:09|Comedy     |2006-02-15 04:46:27|       1|2006-02-15 05:09:17|2005-06-16 15:18:57|          1|2005-06-17 21:05:57|       1|2006-02-15 21:30:53|         6|          1|       1|  4.99|2005-06-16 15:18:57|
     1725|         726|    159|          5|     157|GRETA      |MALDEN      |2006-02-15 04:34:33|2006-02-15 05:05:03|CLOSER BANG           |A Unbelieveable Panorama of a Frisbee And a Hunter who must Vanquish a Monkey in Ancient India                      |        2006|          1|                    |              5|       4.99|    58|           12.99|R     |2006-02-15 05:03:42|{Trailers,Behind the Scenes}                            |'ancient':18 'bang':2 'closer':1 'frisbe':8 'hunter':11 'india':19 'monkey':16 'must':13 'panorama':5 'unbeliev':4 'vanquish':14                                                    |2006-02-15 05:07:09|Comedy     |2006-02-15 04:46:27|       1|2006-02-15 05:09:17|2005-06-16 15:18:57|          1|2005-06-17 21:05:57|       1|2006-02-15 21:30:53|         6|          1|       1|  4.99|2005-06-16 15:18:57|
     1725|         726|    159|          5|     149|RUSSELL    |TEMPLE      |2006-02-15 04:34:33|2006-02-15 05:05:03|CLOSER BANG           |A Unbelieveable Panorama of a Frisbee And a Hunter who must Vanquish a Monkey in Ancient India                      |        2006|          1|                    |              5|       4.99|    58|           12.99|R     |2006-02-15 05:03:42|{Trailers,Behind the Scenes}                            |'ancient':18 'bang':2 'closer':1 'frisbe':8 'hunter':11 'india':19 'monkey':16 'must':13 'panorama':5 'unbeliev':4 'vanquish':14                                                    |2006-02-15 05:07:09|Comedy     |2006-02-15 04:46:27|       1|2006-02-15 05:09:17|2005-06-16 15:18:57|          1|2005-06-17 21:05:57|       1|2006-02-15 21:30:53|         6|          1|       1|  4.99|2005-06-16 15:18:57|
     1725|         726|    159|          5|      41|JODIE      |DEGENERES   |2006-02-15 04:34:33|2006-02-15 05:05:03|CLOSER BANG           |A Unbelieveable Panorama of a Frisbee And a Hunter who must Vanquish a Monkey in Ancient India                      |        2006|          1|                    |              5|       4.99|    58|           12.99|R     |2006-02-15 05:03:42|{Trailers,Behind the Scenes}                            |'ancient':18 'bang':2 'closer':1 'frisbe':8 'hunter':11 'india':19 'monkey':16 'must':13 'panorama':5 'unbeliev':4 'vanquish':14                                                    |2006-02-15 05:07:09|Comedy     |2006-02-15 04:46:27|       1|2006-02-15 05:09:17|2005-06-16 15:18:57|          1|2005-06-17 21:05:57|       1|2006-02-15 21:30:53|         6|          1|       1|  4.99|2005-06-16 15:18:57|
     1725|         726|    159|          5|      21|KIRSTEN    |PALTROW     |2006-02-15 04:34:33|2006-02-15 05:05:03|CLOSER BANG           |A Unbelieveable Panorama of a Frisbee And a Hunter who must Vanquish a Monkey in Ancient India                      |        2006|          1|                    |              5|       4.99|    58|           12.99|R     |2006-02-15 05:03:42|{Trailers,Behind the Scenes}                            |'ancient':18 'bang':2 'closer':1 'frisbe':8 'hunter':11 'india':19 'monkey':16 'must':13 'panorama':5 'unbeliev':4 'vanquish':14                                                    |2006-02-15 05:07:09|Comedy     |2006-02-15 04:46:27|       1|2006-02-15 05:09:17|2005-06-16 15:18:57|          1|2005-06-17 21:05:57|       1|2006-02-15 21:30:53|         6|          1|       1|  4.99|2005-06-16 15:18:57|
     2308|         197|     44|         14|     193|BURT       |TEMPLE      |2006-02-15 04:34:33|2006-02-15 05:05:03|ATTACKS HATE          |A Fast-Paced Panorama of a Technical Writer And a Mad Scientist who must Find a Feminist in An Abandoned Mine Shaft |        2006|          1|                    |              5|       4.99|   113|           21.99|PG-13 |2006-02-15 05:03:42|{Trailers,Behind the Scenes}                            |'abandon':23 'attack':1 'fast':5 'fast-pac':4 'feminist':20 'find':18 'hate':2 'mad':14 'mine':24 'must':17 'pace':6 'panorama':7 'scientist':15 'shaft':25 'technic':10 'writer':11|2006-02-15 05:07:09|Sci-Fi     |2006-02-15 04:46:27|       1|2006-02-15 05:09:17|2005-06-18 08:41:48|          1|2005-06-22 03:36:48|       2|2006-02-15 21:30:53|         7|          1|       1|  4.99|2005-06-18 08:41:48|
     2308|         197|     44|         14|     106|GROUCHO    |DUNST       |2006-02-15 04:34:33|2006-02-15 05:05:03|ATTACKS HATE          |A Fast-Paced Panorama of a Technical Writer And a Mad Scientist who must Find a Feminist in An Abandoned Mine Shaft |        2006|          1|                    |              5|       4.99|   113|           21.99|PG-13 |2006-02-15 05:03:42|{Trailers,Behind the Scenes}                            |'abandon':23 'attack':1 'fast':5 'fast-pac':4 'feminist':20 'find':18 'hate':2 'mad':14 'mine':24 'must':17 'pace':6 'panorama':7 'scientist':15 'shaft':25 'technic':10 'writer':11|2006-02-15 05:07:09|Sci-Fi     |2006-02-15 04:46:27|       1|2006-02-15 05:09:17|2005-06-18 08:41:48|          1|2005-06-22 03:36:48|       2|2006-02-15 21:30:53|         7|          1|       1|  4.99|2005-06-18 08:41:48|
     2308|         197|     44|         14|      74|MILLA      |KEITEL      |2006-02-15 04:34:33|2006-02-15 05:05:03|ATTACKS HATE          |A Fast-Paced Panorama of a Technical Writer And a Mad Scientist who must Find a Feminist in An Abandoned Mine Shaft |        2006|          1|                    |              5|       4.99|   113|           21.99|PG-13 |2006-02-15 05:03:42|{Trailers,Behind the Scenes}                            |'abandon':23 'attack':1 'fast':5 'fast-pac':4 'feminist':20 'find':18 'hate':2 'mad':14 'mine':24 'must':17 'pace':6 'panorama':7 'scientist':15 'shaft':25 'technic':10 'writer':11|2006-02-15 05:07:09|Sci-Fi     |2006-02-15 04:46:27|       1|2006-02-15 05:09:17|2005-06-18 08:41:48|          1|2005-06-22 03:36:48|       2|2006-02-15 21:30:53|         7|          1|       1|  4.99|2005-06-18 08:41:48|
     2308|         197|     44|         14|      18|DAN        |TORN        |2006-02-15 04:34:33|2006-02-15 05:05:03|ATTACKS HATE          |A Fast-Paced Panorama of a Technical Writer And a Mad Scientist who must Find a Feminist in An Abandoned Mine Shaft |        2006|          1|                    |              5|       4.99|   113|           21.99|PG-13 |2006-02-15 05:03:42|{Trailers,Behind the Scenes}                            |'abandon':23 'attack':1 'fast':5 'fast-pac':4 'feminist':20 'find':18 'hate':2 'mad':14 'mine':24 'must':17 'pace':6 'panorama':7 'scientist':15 'shaft':25 'technic':10 'writer':11|2006-02-15 05:07:09|Sci-Fi     |2006-02-15 04:46:27|       1|2006-02-15 05:09:17|2005-06-18 08:41:48|          1|2005-06-22 03:36:48|       2|2006-02-15 21:30:53|         7|          1|       1|  4.99|2005-06-18 08:41:48|
     2363|        3497|    766|          7|     153|MINNIE     |KILMER      |2006-02-15 04:34:33|2006-02-15 05:05:03|SAVANNAH TOWN         |A Awe-Inspiring Tale of a Astronaut And a Database Administrator who must Chase a Secret Agent in The Gulf of Mexico|        2006|          1|                    |              5|       0.99|    84|           25.99|PG-13 |2006-02-15 05:03:42|{Commentaries,Deleted Scenes,Behind the Scenes}         |'administr':14 'agent':20 'astronaut':10 'awe':5 'awe-inspir':4 'chase':17 'databas':13 'gulf':23 'inspir':6 'mexico':25 'must':16 'savannah':1 'secret':19 'tale':7 'town':2       |2006-02-15 05:07:09|Drama      |2006-02-15 04:46:27|       1|2006-02-15 05:09:17|2005-06-18 13:33:59|          1|2005-06-19 17:40:59|       1|2006-02-15 21:30:53|         8|          1|       2|  0.99|2005-06-18 13:33:59|
     2363|        3497|    766|          7|      67|JESSICA    |BAILEY      |2006-02-15 04:34:33|2006-02-15 05:05:03|SAVANNAH TOWN         |A Awe-Inspiring Tale of a Astronaut And a Database Administrator who must Chase a Secret Agent in The Gulf of Mexico|        2006|          1|                    |              5|       0.99|    84|           25.99|PG-13 |2006-02-15 05:03:42|{Commentaries,Deleted Scenes,Behind the Scenes}         |'administr':14 'agent':20 'astronaut':10 'awe':5 'awe-inspir':4 'chase':17 'databas':13 'gulf':23 'inspir':6 'mexico':25 'must':16 'savannah':1 'secret':19 'tale':7 'town':2       |2006-02-15 05:07:09|Drama      |2006-02-15 04:46:27|       1|2006-02-15 05:09:17|2005-06-18 13:33:59|          1|2005-06-19 17:40:59|       1|2006-02-15 21:30:53|         8|          1|       2|  0.99|2005-06-18 13:33:59|
```

## Instalando MonetDB

Estas instrucciones de instalación son para Ubuntu 18.04 o superior, o Debian.

Deben tener instalado el Ubuntu 18.04 o 20.04 desde la Microsoft Store, o bien una máquina virtual con VirtualBox.

Crear el siguiente archivo con el comando `nano`:

```bash
sudo nano /etc/apt/sources.list.d/monetdb.list
```

Y poner el contenido del 2o codeblockde abajo. 👀OJO👀 deben reemplazar la palabra `suite` con el **nombre del release de ubuntu**. Eso lo pueden saber con la salida del comando

```bash
lsb_release -cs
```

Para los que tienen Ubuntu 20.04, la salida será `focal`. 

El contenido del archivo debe ser:

```bash
deb https://dev.monetdb.org/downloads/deb/ [suite] monetdb
deb-src https://dev.monetdb.org/downloads/deb/ [suite] monetdb
```

El siguiente comando instalará la llave pública del monetdb. Esto es para comprobar que la instalación es válida.

```bash
sudo wget --output-document=/etc/apt/trusted.gpg.d/monetdb.gpg https://www.monetdb.org/downloads/MonetDB-GPG-KEY.gpg
```

Ahora vamos a validar la instalación de la llave con:

```bash
sudo apt-key finger
```

La llave para el entry `/etc/apt/trusted.gpg.d/monetdb.gpg` deberá ser `8289 A5F5 75C4 9F50 22F8 EE20 F654 63E2 DF0E 54F3`.

Después de esto ya se puede instalar el MonetDB:

```bash
sudo apt update

sudo apt install monetdb5-sql monetdb-client
```

Contrario a PostgreSQL, los Ubuntu que corren dentro de Windows no soportan que los usuarios definan servicios, osea, que haya programas que arranquen junto con el sistema operativo, porque pues este Ubuntu no "arranca" en realidad.

Por eso tenemos que iniciar el MonetDB a pata 🐾.

