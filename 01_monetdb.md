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

```console
sudo nano /etc/apt/sources.list.d/monetdb.list
```

Y poner el contenido del 2o codeblockde abajo. 👀OJO👀 deben reemplazar la palabra `suite` con el **nombre del release de ubuntu**. Eso lo pueden saber con la salida del comando

```console
lsb_release -cs
```

Para los que tienen Ubuntu 20.04, la salida será `focal`. 

El contenido del archivo debe ser:

```console
deb https://dev.monetdb.org/downloads/deb/ [suite] monetdb
deb-src https://dev.monetdb.org/downloads/deb/ [suite] monetdb
```

El siguiente comando instalará la llave pública del monetdb. Esto es para comprobar que la instalación es válida.

```console
sudo wget --output-document=/etc/apt/trusted.gpg.d/monetdb.gpg https://www.monetdb.org/downloads/MonetDB-GPG-KEY.gpg
```

Ahora vamos a validar la instalación de la llave con:

```console
sudo apt-key finger
```

La llave para el entry `/etc/apt/trusted.gpg.d/monetdb.gpg` deberá ser `8289 A5F5 75C4 9F50 22F8 EE20 F654 63E2 DF0E 54F3`.

Después de esto ya se puede instalar el MonetDB:

```console
sudo apt update

sudo apt install monetdb5-sql monetdb-client
```

Contrario a PostgreSQL, los Ubuntu que corren dentro de Windows no soportan que los usuarios definan servicios, osea, que haya programas que arranquen junto con el sistema operativo, porque pues este Ubuntu no "arranca" en realidad.

Por eso tenemos que iniciar el MonetDB a pata 🐾.

## Cómo funcionan las BDs columnares?

Funcionan en cluster, básicamente.

Idóneamente debemos tener 1 nodo maestro y 1 o más nodos `worker`.

![image](https://user-images.githubusercontent.com/1316464/136245765-818307c7-746f-4c41-987a-bacf6663877f.png)

Cada nodo worker puede ser de **réplica**, o de _**sharding**_.

En la réplica, 2 nodos tienen la mismita información, en uno se escribe y en otro se lee.

Esta arquitectura en bases de datos relacionales sirve para distribuir la carga entre los sistemas transaccionales y los sistemas de información, es decir, en la fuente de la réplica conectamos nuestros puntos de venta, sistemas de inventario, de marketing, etc, y las herramientas de BI las conectamos a la réplica para que un query mal planeado de cientos de miles de registros no roben recursos a la BD transaccional y nos detengan la operación.

![image](https://user-images.githubusercontent.com/1316464/136246389-4590c16c-da88-4889-957b-cdbad617f82d.png)

Por otro lado, en el _sharding_ tenemos cierto cacho de la BD en un nodo, y otros cacho en otro, de modo que si un nodo falla, seguimos teniendo disponibilidad de cierta cantidad de registros o datos:

![image](https://user-images.githubusercontent.com/1316464/136246648-1a65159e-8c0a-478c-a92e-926c30b46271.png)


▶️ Pero como todo lo vamos a correr en local, entonces tanto el proceso `master` y los procesos `worker` en la misma máquina.


En MonetDB, los nodos `worker` los llamamos _farm_.

Vamos a arrancar el MonetDB con los siguientes comandos:

```console
cd ~
mkdir monetdb
mkdir monetdb/farm
monetdbd create monetdb/farm
monetdbd start monetdb/farm
```
Con estos comandos creamos un directorio monetdb/myfarm donde va a vivir nuestra BD.

Luego usamos el comando `monetdbd` para crear y arrancar el servidor de MonetDB con 1 nodo en su farm.

Cuando vean comandos de Unix que terminen con una `d`, seguro son _daemon_. Si, viene de la palabra _demon_, pero con este significado:

> The term was coined by the programmers at MIT's Project MAC. According to Fernando J. Corbató, who worked on Project MAC in 1963, his team was the first to use the term daemon, inspired by Maxwell's demon, an imaginary agent in physics and thermodynamics that helped to sort molecules, stating, "We fancifully began to use the word daemon to describe background processes that worked tirelessly to perform system chores". Unix systems inherited this terminology. Maxwell's demon is consistent with Greek mythology's interpretation of a daemon as a supernatural being working in the background. However, BSD and some of its derivatives have adopted a Christian demon as their mascot rather than a Greek daemon.

Todos los _daemon_ ayudan a correr procesos background en sistemas Unix.

Por ende, TODOS los servidores son ejecutados por _daemons_.

Una vez arrancado el _daemon_ o el server, podemos crear bases de datos.

Vamos a crear la base de datos VOC. Esta base de datos es el registro naviero de la _Vereenigde geoctrooieerde Oostindische Compagnie_, o _Dutch East India Company_ para los siglos XVII y XVIII. Si, la misma de "Piratas del Caribe".

![image](https://user-images.githubusercontent.com/1316464/136251974-0e608ce1-8079-4dfb-aed1-6a727219cbb2.png)


```console
monetdb create voc
monetdb release voc
```

El comando `monetdb` sirve para interactuar administrativamente con la BD, para crear objetos y estructuras. **⚠️NO CONFUNDIR CON EL COMANDO `monetdbd`⚠️**, que es el `daemon` que vimos arriba.

El operador `create` crea una BD en modo _maintenance_, es decir, en un área de staging que no es producción, y por tanto no está disponible para conexiones desde fuera, por ejemplo, desde DBeaver.

El operador `release` libera a producción la BD y ahora si podemos interactuar con ella desde afuera.

Vamos a [descargar el ZIP](https://www.monetdb.org/sites/default/files/voc_dump.zip) con la BD, crear usuarios y cargar datos a nuestra BD `voc`. Vamos a hacer esto con el cliente `mclient`, que nos permite interactuar con MonetDB desde la command line:

```console
mclient -u monetdb -d voc
    password: "monetdb"
CREATE USER "voc" WITH PASSWORD 'voc' NAME 'VOC Explorer' SCHEMA "sys";
CREATE SCHEMA "voc" AUTHORIZATION "voc";
ALTER USER "voc" SET SCHEMA "voc";
\q
```

Lo que estamos haciendo aquí es:
1. conectándonos con `mclient` con el usuario `monetdb` a la BD `voc`. El password es igual `monetdb`.
2. creando un usuario con clave `voc` con passwd `voc` con nombre "VOC Explorer" en el esquema `sys`, que es el de default, como el `public` en PostgreSQL. En MonetDB es necesario especificar el esquema donde queremos que se alojen los objetos que estamos creando.
3. creando el esquema `voc` y dando autorización al usuario `voc` para conectarse.
4. cambiando el usuario del esquema de default `sys` al esquema `voc`.
5. saliendo del `mclient` con el meta-comando `\q`.

Ahora vamos a reconectarnos con la BD `voc` con el usuario `voc` en lugar del usuario administrador `monetdb` y crear unas tablas de muestra:

```console
mclient -u voc -d voc
    password: "voc"
start transaction;

CREATE TABLE test (id int, data varchar(30));

\d

\d test

rollback;

\d
``` 

Qué estamos haciendo aquí?

1. Nos estamos conectando a la BD `voc` con el usuario `voc`.
2. Estamos iniciando una transacción. 👀OJO👀, en MonetDB y en la mayoría de las BDs columnares, los comandos para la creación de objetos **SI FORMAN PARTE DE LAS TRANSACCIONES Y POR TANTO PUEDEN REVERSARSE**.
3. Estamos creando la tabla `test` que tiene solo los campos `id` y `data`.
4. El _metacommand_  `\d` significa _describe_, y si no recibe argumentos, nos describe la BD e, términos de las tablas que contiene porque ahí estamos parados en este momento.
5. `\d test` nos describe la tabla `test` que acabamos de crear, y lo que nos muestra es el DDL que usamos para crear la tabla.
6. Reversamos la transacción con `rollback;`.
7. Vemos con `\d` que la tabla desapareció. En PostgreSQL y la mayoría de las relacionales, un rollback no afecta los comandos estructurales con los que creamos tablas, índices, secuencias, etc.

Vamos ahora a importar la BD que descargamos:

```console
mclient -u voc -d voc < voc_dump.sql
```





