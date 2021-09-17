# Document Databases - MongoDB

Las Document Databases guardan documentos JSON.

Los documentos JSON, como ya vimos, tienen la sig forma:

![image](https://user-images.githubusercontent.com/1316464/129584321-118512a6-0d63-4fe4-b508-991e1597fa7b.png)

## Descarga e instalación

1. Vamos a descargar la Community Edition [de esta liga](https://www.mongodb.com/try/download/community), dando click en la siguiente ventanita con los settings que aparecen según su sistema operativo:

![image](https://user-images.githubusercontent.com/1316464/129503203-6f73804f-922d-44a7-a2fe-d5cb8f0d7a73.png)

2. Abrimos el instalador y aceptamos los defaults dando click en este botón de esta ventanita. ACEPTEN TODOS LOS DEFAULTS. Es posible que les pida reiniciar:

![image](https://user-images.githubusercontent.com/1316464/129503596-c00274a9-75ad-4dd1-b523-6e75bccff0b5.png)

3. La instalación por default va a instalar y abrir MongoDB Compass, que es como el DBeaver de MongoDB. **No lo pelen**, vamos a instalar un cliente más pro: el RoboMongo. [Descárguenlo de aquí](https://robomongo.org/). Les va a pedir datos inútiles. Capturen lo que sea para que les permita bajarlo:

![image](https://user-images.githubusercontent.com/1316464/129585366-ec4adfc9-d6e2-4a21-bc82-1423af855a8c.png)

4. Esto va a bajar un ZIP, y en este ZIP vienen 2 archivos. Hay que instalar solo el Robo 3T, que lo tengo marcado aquí:

![image](https://user-images.githubusercontent.com/1316464/129586302-9fa1c80e-2c8b-417b-b85a-2f83ba544af4.png)

5. Nos va a salir este choro mareador. Obvio no hay que leerlo y darle **I accept**:

![image](https://user-images.githubusercontent.com/1316464/129586684-1652b439-4a12-4ba4-b718-c42446bfdbb9.png)

6. Todas estas instalaciones se pueden hacer en paralelo, así que para este momento ya debe estar instalado el MongoDB Community Server. Va a aparecer esta pantalla toda sosa. Den click en **Create** en la parte de arriba de esa ventana:

![image](https://user-images.githubusercontent.com/1316464/129588057-43dc7459-88c0-4345-ab01-c341dfc6545c.png)

7. El string de conexión a nuestra BD local es **`mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000`**. Vamos a capturarlo en la pantalla siguiente y darle "Save":

![image](https://user-images.githubusercontent.com/1316464/129588325-112afcf5-a69b-478b-bd72-562740ee92ae.png)

8. Vamos ahora a instalar el comando `mongosh`, que es el shell de MongoDB y que será una 2a forma de interactuar con esa DB. [Aquí los pasos para la instalación](https://docs.mongodb.com/mongodb-shell/install/#std-label-mdb-shell-install). Igual acepten todos los defaults.

9. Esto nos va a instalar el shell de MongoDB, de modo que al arrancar...
   - una terminal de Linux
   - una ventana de CMD de Windows
   - una terminal de MacOS

DEBEMOS poder tener acceso al comando `mongosh`. Pruébenlo. Mi salida de Windows debe ser igual a la suya:

![image](https://user-images.githubusercontent.com/1316464/129590933-45096309-4e3f-453b-9c6b-4d2cbcc49543.png)

Todo esto termina la instalación de mugres que vamos a necesitar para interactuar con MongoDB.

## Uso de MongoDB

### Creando una DB e insertando documentos

Vamos a usar `mongosh` para tener una línea de comandos. Entraremos el comando:

```javascript 
use book
```

De acuerdo a la documentación, este comando "crea una base de datos", pero esto no es enteramente cierto. Esto solo aparta espacio en el MongoDB para comenzar a agregar documentos (ojo: no son registros). No tenemos una BD formalmente creada hasta no agregar documentos a esa BD.

Para saber qué BD estamos usando:

```javascript
db
```

Y para ver todas las DBs que tenemos disponibles:

```javascript
show dbs
```

Vamos ahora a agregar un documento:

```javascript 
db.towns.insertOne({
   name: "New York", 
   population: 22200000, 
   lastCensus: ISODate("2016-07-01"), 
   famousFor: ["the MOMA", "food", "The Met"], 
   mayor: { 
      name: "Bill de Blasio", 
      party: "D"
   }
})
```

Analicemos línea por línea:
1. **db** es el objeto con el que `mongosh` se refiere a la BD que estamos usando, en este caso `book`. ⚠️**IMPORTANTE**:warning: - después del elemento db, tenemos el elemento **towns**, esto es una **COLECCIÓN**. Recuerden la jerarquía de los JSON:

![image](https://user-images.githubusercontent.com/1316464/129602030-b2d9e0e3-43e3-440c-97c4-ecbdc6008d4c.png)

Esto significa que estamos creando una colección de documentos llamada `towns`.

Igual, pongan atención a uno de los features más relevantes de las Document Databases: :**¡No necesitamos predefinir estructura/esquema para crear colecciones ni documentos!** 🤓

Por fin! Libres de la tiranía de tener que definir, pensar, diseñar antes de tirar código!

![image](https://user-images.githubusercontent.com/1316464/129604124-35f006ca-592d-4639-861e-9499fab807b1.png)

El `insertOne` sirve para insertar solo 1 documento en la colección.

El paréntesis que abre `(` indica el inicio de los argumentos de la función `insertOne`.

La llave que abre `{` indica que viene un documento JSON.

2. Inicia el documento con atributos `name` (string), `population` (number), `lastCensus` (Date), `famousFor` (array de strings), y `mayor` de tipo DOCUMENTO, que es otro objeto anidado con sus propios atributos, ⚠️TODO SEPARADO POR COMAS:warning:.

Qué pasa si se nos para una coma❓

Un error como estos:

![image](https://user-images.githubusercontent.com/1316464/129617183-b60c21f4-b3d2-4120-be57-c75d745daaf6.png)

Fíjense igual que `mongosh` nos ayuda a identar la función principal, y los documentos anidados.

3. Al cerrar llaves y paréntesis, debemos tener esta salida:

![image](https://user-images.githubusercontent.com/1316464/129617366-03acc5a7-cdc1-4e57-866f-f1d037a9eeb7.png)

Qué pasa si volvemos a ejecutar la misma inserción❓

Las Document Databases no tienen "llaves" como las BDs relacionales, entonces **al ejecutar una inserción 2 veces, para MongoDB son objetos enteramente diferentes**, y de hecho cada inserción se forma un ID autoasignado diferente (similar a las secuencias de las BDs relacionales). Adicionalmente, MongoDB crea un atributo llamado `_id` EN AUTOMÁGICO, sin preguntarnos, que es donde se guarda esta llave autogenerada. Este atributo se encuentra en TODOS los documentos de 1er nivel (es decir, no está en los documentos _anidados_).

Estos IDs autogenerados son de 12 bytes y tienen la siguiente estructura:

![image](https://user-images.githubusercontent.com/1316464/129835091-5da5e77a-55ae-4f2c-b4b5-cb54107f0458.png)

- **`time`**: timestamp del sistema operativo
- **`machine id`**: ID de la máquina
- **`process id`**: ID del proceso (un concepto de Unix)
- **`increment`**: contador autoincrementado de 3 bytes

Este tipo de IDs autogenerados es que podemos tener varias instancias de MongoDB corriendo en la misma máquina y no tendremos riesgos de colisiones. YAY!

### Inertando múltiples documentos

Intentemos ahora:

```javascript 
db.towns.insertMany([
   {name: "New York", 
   population: 22200000, 
   lastCensus: ISODate("2016-07-01"), 
   famousFor: ["the MOMA", "food", "The Met"], 
   mayor: { 
      name: "Bill de Blasio", 
      party: "D"
      }
   },
   {name: "London", 
   population: 15000000, 
   lastCensus: ISODate("2018-01-01"), 
   famousFor: ["The British Museum", "Fish & Chips", "The Tate Modern"], 
   mayor: { 
      name: "Sadiq Khan", 
      party: "L"
      }
   },
   {name: "Mexicalpan de las Tunas", 
   population: 20000000, 
   lastCensus: ISODate("2019-01-01"), 
   famousFor: ["Museo Nacional de Antropología", "Tacos de Canasta", "Tlacoyos"], 
   mayor: { 
      name: "Claudia Sheinbaum", 
      party: "MORENA"
      }
   }
])
```

Debemos tener esta salida:

![image](https://user-images.githubusercontent.com/1316464/129618720-de6017ca-4bde-4919-81e6-b5807567be2e.png)

## SQL es a BDs relacionales como JavaScript es a MongoDB

El lenguaje base de MongoDB es JavaScript. JavaScript tiene mala fama entre la comunidad de ingeniería de software, pero es ampliamente gustado por la comunidad de desarrollo web. Principalmente por su inconsistencia...

![image](https://user-images.githubusercontent.com/1316464/129836528-7d962188-c6ba-4733-90d9-cf18c97394a0.png)

...por su abundancia de frameworks inútiles...

![image](https://user-images.githubusercontent.com/1316464/129836673-14b6ec20-2775-4c5c-8348-33f7f445c765.png)

...aunque es el primero que nos ofrece productividad expedita.

Usaremos JavaScript para todo con MongoDB, hasta pedor ayuda:

```javascript
db.help()
db.towns.help()
```

Igual podemos identificar el tipo de un objeto, justo como en JavaScript:

```javascript
typeof db
typeof db.towns
typeof db.towns.insertOne
```

Examinemos el código fuente de la función `insertOne`:

```javascript
db.towns.insertOne //sin paréntesis
> [Function: insertOne] AsyncFunction {
  apiVersions: [ 1, Infinity ],
  serverVersions: [ '3.2.0', '999.999.999' ],
  returnsPromise: true,
  topologies: [ 0, 2, 3, 1 ],
  returnType: { type: 'unknown', attributes: {} },
  deprecated: false,
  platforms: [ 0, 1, 2 ],
  isDirectShellCommand: false,
  shellCommandCompleter: undefined,
  help: [Function (anonymous)] Help
}
```

Esto sería como ver qué hay dentro del comando `INSERT` en una BD relacional, cosa que no podemos hacer!

Vamos a crear nuestra propia función para insertar ciudades en la colección `db.towns`:

```javascript
function insertCity(name, population, lastCensus, famousFor, mayorInfo) {
   db.towns.insertOne({
      name: name,
      population: population,
      lastCensus: ISODate(lastCensus),
      famousFor: famousFor,
      mayor : mayorInfo
   });
}
```

Esto es como un `create function insertcity (string, numeric, date, string, string) AS 'insert into table city values ($1,$2,$3,$4,$5)'` para PostgreSQL.

Podemos llamar esta función ahora sin el `db.towns.insertOne`. No es mucho ahorro, pero con _user-defined functions_ podemos hacer cosas más elaboradas:

```javascript
insertCity("Punxsutawney", 6200, '2016-01-31', ["Punxsutawney Phil"], { name : "Richard Alexander" })
insertCity("Portland", 582000, '2016-09-20', ["beer", "food", "Portlandia"], { name : "Ted Wheeler", party : "D" })
```

## Leyendo datos: SELECT en SQL, find() en MongoDB

Para ensayar las funciones de consulta, debemos importar algunas BDs de prueba.

1. Vamos a clonar este repo en nuestro directorio preferido. Opcionalmente podemos bajar el archivo ZIP de ese URL

```sh
git clone https://github.com/neelabalan/mongodb-sample-dataset
```

2. Vamos a necesitar el comando `mongoimport`, que no se instala por default. Lo descargaremos [de aquí](https://www.mongodb.com/try/download/database-tools?tck=docs_databasetools).

3. En Windows muy probablemente el comando se instaló en `C:\Program Files\MongoDB\Tools\100\bin`, mientras que el Linux y en Mac ya está instalado.

4. Vamos a utilizar el comando import de esa localidad para insertar uno de los JSONs del repo que descargamos:

```sh
mongoimport --db trainingsessions --drop --file C:\Users\ramos\mongodb-sample-dataset\sample_training\tweets.json
```

5. Validamos que haya sido insertada esa colección correctamente:

```javascript
use trainingsessions
db.getCollectionNames()
db.tweets.find()
```

Ahora si vamos a leer estos datos. Para leer datos en MongoDB la función base es `find()`:

- `db.towns.find()` trae todos los _documentos_ de la _colección_ `towns`.
- `db.towns.find({ "_id" : ObjectId("611ce2e73afe7ee944574e51") })` va a traer el documento con ID `611ce2e73afe7ee944574e51`. Recordemos que los ID son autogenerados y el atributo `_id` es creado automáticamente
- `db.towns.find( {"_id" : ObjectId("611ce2e73afe7ee944574e51")}, {population : 1} )` va a traer el documento con ID `611ce2e73afe7ee944574e51` pero solo su atributo `population` similar a un `select population from towns where id = 611ce2e73afe7ee944574e51`
- `db.towns.find( {"_id" : ObjectId("611ce2e73afe7ee944574e51")}, {population : 0} )` va a traer el mismo documento, pero ahora con todos sus atributos **EXCEPTO** `population`
- `db.towns.find( {population : 6200})` va a traer el documento con `population` igual a _6200_
- `db.towns.find( {name : "London"})` va a traer el documento con `name` igual a _"London"_

En general, podemos decir que la función `find()` frecuentemente es llamada con 2 _documentos_ como argumento:

- 1 para filtrado, similar al `WHERE` de SQL. Esto se le llama **FILTER** en bases de datos de documentos.
- 1 para _selección_ de atributos, similar al `SELECT` de SQL. Esto se le llama **PROJECT** en bases de datos de documentos.

Vamos a establecer algunas equivalencias entre SQL y MongoDB con la siguiente tabla y la colección `tweets` que acabamos de importar. Para ejecutar los ejemplos primero debemos entrar `use trainingsessions`.

| Operación              | Sintaxis                    | E.g.                                                    | Equivalencia RDBMS                                                                    |
|------------------------|---------------------------|----------------------------------------------------------------|------------------------------------------------------------------------------------|
| Igual a X              | `{"key":[value]}`     | `db.tweets.find({"source":"web"})`                | where source = 'web'                                                       |
| AND en el WHERE              | `{"key1":[value1],"key2":[value2]}`     | `db.tweets.find({"source":"web","favorited":false})`                | where source = 'web' **and** favorited = false                                                       |
| Menor que              | `{"key":{$lt:[value]}}`     | `db.tweets.find({"user.friends_count":{$lt:50}})` | where user.friends_count < 50 (aquí estamos "viajando" del documento principal al documento anidado `user` y de ahí a su atributo `friends_count`                                                                   |
| Menor o igual a       | `{"key":{$lte:[value]}}`    | `db.tweets.find({"user.friends_count":{$lte:50}})` | where user.friends_count <= 50                                                             |
| Mayor que           | `{"key":{$gt:[value]}}`     | `db.tweets.find({"user.friends_count":{$gt:50}})`  | where user.friends_count > 50                                                                   |
| Mayor o igual a    | `{"key":{$gte:[value]}}`    | `db.tweets.find({"user.friends_count":{$gte:50}})` | where user.friends_count >= 50                                                                  |
| Diferente a             | `{"key":{$ne:[value]}}`     | `db.tweets.find({"user.friends_count":{$ne:50}})`  | where user.friends_count != 50                                                                  |
| Valores presentes en array     | `{"key":{$in:[value1,value2...valueN]}}` | `db.tweets.find({"entities.urls.indices":{$in:[54,74]}})` | where entities.urls.indices **in** (54,74)                    |
| Valores ausentes en array | `{"key":{$nin:[value]}}`                 | `db.tweets.find({"entities.urls.indices":{$nin:[54,74]}})`     | where entities.urls.indices **not in** (54,74) |



### Uso de expresiones regulares en `find()`

Para lograr emular el `LIKE` de SQL en MongoDB, debemos usar forzosamente expresiones regulares. Por ejemplo:

```javascript
db.tweets.find({"user.url":/^http(s|):\/\/(www\.|)facebook\.com/})
```

Esto es similar a la sentencia SQL:

```sql
...where user.url like 'http?://facebook.com%'
```

Esto va a encontrar todos los tuits cuyo URL del perfil de usuario sean ligas a perfiles de FB.

Para encontrar todos los tuits con el hashtag que comience on `#polit`:

```javascript
db.tweets.find({"entities.hashtags.text":/^polit/})
```

En este caso, el caracter `^` indica que el match debe darse desde el principio, porque si no lo ponemos, vamos a hacer match con este tuit que anda por ahí:

```json
{
   "_id":{
      "$oid":"5c8eccb1caa187d17ca64de8"
   },
   "text":"Balmoral, booze and the rest of Blair's book digested  http://bit.ly/9KwcSP  #Blair #AJourney #UKpolitics #Labour #Bush",
   "in_reply_to_status_id":null,
   "retweet_count":null,
   "contributors":null,
   "created_at":"Thu Sep 02 18:34:32 +0000 2010",
   "geo":null,
   "source":"<a href=\"http://www.tweetdeck.com\" rel=\"nofollow\">TweetDeck</a>",
   "coordinates":null,
   "in_reply_to_screen_name":null,
   "truncated":false,
   "entities":{
      "user_mentions":[
         
      ],
      "urls":[
         {
            "indices":[
               55,
               75
            ],
            "url":"http://bit.ly/9KwcSP",
            "expanded_url":null
         }
      ],
      "hashtags":[
         {
            "text":"Blair",
            "indices":[
               77,
               83
            ]
         },
         {
            "text":"AJourney",
            "indices":[
               84,
               93
            ]
         },
         {
            "text":"UKpolitics",
            "indices":[
               94,
               105
            ]
         },
         {
            "text":"Labour",
            "indices":[
               106,
               113
            ]
         },
         {
            "text":"Bush",
            "indices":[
               114,
               119
            ]
         }
      ]
   },
   "retweeted":false,
   "place":null,
   "user":{
      "friends_count":556,
      "profile_sidebar_fill_color":"DDEEF6",
      "location":"",
      "verified":false,
      "follow_request_sent":null,
      "favourites_count":0,
      "profile_sidebar_border_color":"C0DEED",
      "profile_image_url":"http://a2.twimg.com/profile_images/1026348478/US-UK-blend_normal.png",
      "geo_enabled":false,
      "created_at":"Sat Jun 26 14:58:34 +0000 2010",
      "description":"Promoting and discussing the special relatonship between the United States and the United Kingdom.",
      "time_zone":null,
      "url":null,
      "screen_name":"USUKrelations",
      "notifications":null,
      "profile_background_color":"C0DEED",
      "listed_count":4,
      "lang":"en",
      "profile_background_image_url":"http://a3.twimg.com/profile_background_images/116769793/specialrelations.jpg",
      "statuses_count":647,
      "following":null,
      "profile_text_color":"333333",
      "protected":false,
      "show_all_inline_media":false,
      "profile_background_tile":true,
      "name":"Special Relationship",
      "contributors_enabled":false,
      "profile_link_color":"0084B4",
      "followers_count":264,
      "id":159870717,
      "profile_use_background_image":true,
      "utc_offset":null
   },
   "favorited":false,
   "in_reply_to_user_id":null,
   "id":{
      "$numberLong":"22820800600"
   }
}
```

En esta materia no veremos a fondo expresiones regulares, pero aquí 2 ligas útiles:

1. https://regexone.com/ es un crash course rápido para aprender las bases de las expresiones regulares
2. https://regexr.com/ es una plataformita para probar sus regexp contra ejemplos suyos o de terceros

**⚠️IMPORTANTE:⚠️** Las expresiones regulares que deben ir en estos queries son [Perl-compatible Regular Expressions (PCRE)](https://en.wikipedia.org/wiki/Perl_Compatible_Regular_Expressions)

### Queries a arrays

A diferencia de las RDBMS, las Document Databases aceptan en sus atributos arrays de valores.

Recuerden que las reglas de diseño de las relacionales nos obligan a que **un atributo tenga solo 1 valor**, mientras que en las de documentos un atributo puede ser un string, un número, o un arreglo de cualquiera de ambos.

Este query va a regresar el documento que tenga **ÚNICA Y EXACTA Y ORDENADAMENTE** los elementos **54 y 74**.

```javascript
db.tweets.find({"entities.urls.indices":[54,74]})
```
Osea, si hay un elemento que tiene el orden **74 y 54**, no no lo va a encontrar.

Para buscarlos a todos, **sin importar orden**, usamos el operador `$all`:

```javascript
db.tweets.find({"entities.urls.indices":{$all:[54,74]}})
```

Para buscar todos los documentos que **AL MENOS** tengan uno de los elementos:

```javascript
db.tweets.find({"entities.urls.indices":54})
```

O usar el operador `$in` que vimos arriba.

Para buscar un rango en un array numérico, en este caso, entre 50 y 90, inclusive:

```javascript
db.tweets.find({"entities.urls.indices":{$lte:50, $gte:90}})
```

Y para buscar documentos cuyo N-avo elemento sea igual a X:

```javascript
db.tweets.find({"entities.urls.indices.1":59})
```

Recordemos  que los arrays en MongoDB **están indexados desde 0 y no desde 1**.

Para buscar un documento por el tamaño de uno de sus atributos de tipo array:

```javascript
db.tweets.find({"entities.hashtags":{$size:7}})
```

Y para buscar documentos cuyos atributos tipo array tengan más de 7 elementos:

```javascript
db.tweets.find({"entities.hashtags.7":{$exists:true}})
```

Podemos combinar operadores `$exists`, `$gte` y `$lte` para buscar documentos que tengan un array entre N y M elementos. El siguiente query regresa los tuits que tengan **EXACTAMENTE** un hashtag, aprovechando la _dot notation (.)_ para viajar de `entities->hashtags->[elemento del array con índice 0]` y verificar su existencia con `{$exists:true}`, y hacer elk mismo viaje al `[elemento del array con índice 1]` y asegurarnos que no existe con `{$exists:false}`. 

```javascript
db.tweets.find({"entities.hashtags.1":{$exists:false},"entities.hashtags.0":{$exists:true}},{"entities":1})
```

El racional de esta forma de `find()` es que si buscamos arrays con num de elementos mayores a 7, entonces tendremos arrays cuyo elemento en la posición 7 (que realmente es la posición 8 porque **comenzamos desde 0**) debe tener un elemento presente.

### Queries a documentos anidados y arrays de documentos

Para los siguientes ejemplos vamos a insertar estos documentos con la función `insertMany()`:

```json
[
   {
      "item":"journal",
      "instock":[
         {
            "warehouse":"A",
            "qty":5
         },
         {
            "warehouse":"C",
            "qty":15
         }
      ]
   },
   {
      "item":"notebook",
      "instock":[
         {
            "warehouse":"C",
            "qty":5
         }
      ]
   },
   {
      "item":"paper",
      "instock":[
         {
            "warehouse":"A",
            "qty":60
         },
         {
            "warehouse":"B",
            "qty":15
         }
      ]
   },
   {
      "item":"planner",
      "instock":[
         {
            "warehouse":"A",
            "qty":40
         },
         {
            "warehouse":"B",
            "qty":5
         }
      ]
   },
   {
      "item":"postcard",
      "instock":[
         {
            "warehouse":"B",
            "qty":15
         },
         {
            "warehouse":"C",
            "qty":35
         }
      ]
   }
]
```

1. Creen una nueva BD llamada `warehouse`
2. Creen una colección llamada `inventory`
3. Inserten estos documentos de arriba

El siguiente query va a regresar todos los artículos que estén en en warehouse A y de los que tengamos 5 en inventario:

```javascript
db.inventory.find( { "instock": { warehouse: "A", qty: 5 } } )
```

El valor de retorno es:

```json
[
  {
    _id: ObjectId("612339842cd2fe46682acd32"),
    item: 'journal',
    instock: [ { warehouse: 'A', qty: 5 }, { warehouse: 'C', qty: 15 } ]
  }
]
```

El query no nos está regresando 2 documentos, sino el documento en el array `instock` que hace match con las condiciones que le dimos.

**👀OJO:👀** esta sintaxis es parecida a la búsqueda de documentos de 1er nivel (`find("key1":value1,"key2":value2`), pero como estamos buscando documentos **ANIDADOS O EN ARRAY**, entonces debemos de especificar el nombre del array `instock` antes de los params de búsqueda.

Una gran diferencia es en el orden de los atributos que estemos buscando en el array de documentos. Por ejemplo, si ejecutamos esto:

```javascript
db.inventory.fnd( { "instock": { qty: 5, warehouse: "A" } } )
```

Va a regresar **NADA**, porque ningún documento dentro del array tiene primero el atributo `qty`.

El siguiente query va a regresar todos los documentos de `instock` que tengan un `qty` menor o igual a 20, junto con los documentos que acompañen a ese que hace match:

```javascript
db.inventory.find( { "instock.qty": { $lte: 20 } } )
```

Este query también es similar a los que vimos para consultar documentos de 1er nivel, con la diferencia de que `instock` es un array de documentos y no un atributo o un array de elementos individuales.

Si deseamos limitar la búsqueda a un índice del array, como para evitar tener un documento que no cumpla con las condiciones, podemos especificarlo así:

```javascript
db.inventory.find( { 'instock.0.qty': { $lte: 20 } } )
```

Este query nos regresará del arreglo `instock` los **PRIMEROS** documentos (índice 0) cuyo atributo `qty` sea igual o menor a 20.

### El operador `$elemMatch`

Hay estructuras de documentos de varios niveles y con arreglos anidados donde al lanzar queries a estos arreglos puede regresarnos documentos que no necesariamente cumplen el criterio.

1. Vamos a crear otra BD llamada "store"
2. Con una colección llamada "articles"
3. Insertamos este array de documentos con `insertMany`

```javascript
db.articles.insert([
{
	"_id" : 1,
	"description" : "DESCRIPTION ARTICLE AB",
	"article_code" : "AB",
	"purchase" : [
		{
			"company" : 1,
			"cost" : NumberDecimal("80.010000")
		},
		{
			"company" : 2,
			"cost" : NumberDecimal("85.820000")
		},
		{
			"company" : 3,
			"cost" : NumberDecimal("79.910000")
		}
	],
	"stock" : [
	    {
	        "country" : "01",
	        "warehouse" : {
	            "code" : "02",
	            "units" : 10
	        }
	    },
	    {
	        "country" : "02",
	        "warehouse" : {
	            "code" : "02",
	            "units" : 8
	        }
	    }
	]
},
{
	"_id" : 2,
	"description" : "DESCRIPTION ARTICLE AC",
	"article_code" : "AC",
	"purchase" : [
		{
			"company" : 1,
			"cost" : NumberDecimal("90.010000")
		},
		{
			"company" : 2,
			"cost" : NumberDecimal("95.820000")
		},
		{
			"company" : 3,
			"cost" : NumberDecimal("89.910000")
		}
	],
	"stock" : [
	    {
	        "country" : "01",
	        "warehouse" : {
	            "code" : "01",
	            "units" : 20
	        }
	    },
	    {
	        "country" : "02",
	        "warehouse" : {
	            "code" : "02",
	            "units" : 28
	        }
	    }
	]
}
]);
```

Qué función `find()` necesitamos para obtener los "artículos" con `stock` en el `warehouse` 02 en el `country` 01?

```javascript
db.articles.find({"stock.country":"01","stock.warehouse.code":"02"})
```

Ese query nos va a regresar los 2 documentos que insertamos:

```json
{
	"_id" : 1,
	"description" : "DESCRIPTION ARTICLE AB",
	"article_code" : "AB",
	"purchase" : [
		{
			"company" : 1,
			"cost" : NumberDecimal("80.010000")
		},
		{
			"company" : 2,
			"cost" : NumberDecimal("85.820000")
		},
		{
			"company" : 3,
			"cost" : NumberDecimal("79.910000")
		}
	],
	"stock" : [
		{
			"country" : "01",
			"warehouse" : {
				"code" : "02",
				"units" : 10
			}
		},
		{
			"country" : "02",
			"warehouse" : {
				"code" : "02",
				"units" : 8
			}
		}
	]
}
{
	"_id" : 2,
	"description" : "DESCRIPTION ARTICLE AC",
	"article_code" : "AC",
	"purchase" : [
		{
			"company" : 1,
			"cost" : NumberDecimal("90.010000")
		},
		{
			"company" : 2,
			"cost" : NumberDecimal("95.820000")
		},
		{
			"company" : 3,
			"cost" : NumberDecimal("89.910000")
		}
	],
	"stock" : [
		{
			"country" : "01",
			"warehouse" : {
				"code" : "01",
				"units" : 20
			}
		},
		{
			"country" : "02",
			"warehouse" : {
				"code" : "02",
				"units" : 28
			}
		}
	]
}
```

Como podemos ver, el array `stock` del documento de 1er nivel con `_id` 2 cumple con las condiciones **POR SEPARADO**, por lo tanto este query nos puede regresar resultados espurios _si es que estamos buscando solamente el documento cuyo array `stock` tenga un elemento que cumpla **CON AMBOS CRITERIOS**.

Para tener el comportamiento esperado, debemos usar el operador `$elemMatch`:

```javascript
db.articles.find({ stock : { $elemMatch : { country : "01", "warehouse.code" : "02" } } })
```

Esto nos debe dar el documento correcto:

```json
{
	"_id" : 1,
	"description" : "DESCRIPTION ARTICLE AB",
	"article_code" : "AB",
	"purchase" : [
		{
			"company" : 1,
			"cost" : NumberDecimal("80.010000")
		},
		{
			"company" : 2,
			"cost" : NumberDecimal("85.820000")
		},
		{
			"company" : 3,
			"cost" : NumberDecimal("79.910000")
		}
	],
	"stock" : [
		{
			"country" : "01",
			"warehouse" : {
				"code" : "02",
				"units" : 10
			}
		},
		{
			"country" : "02",
			"warehouse" : {
				"code" : "02",
				"units" : 8
			}
		}
	]
}
```

El operador `$elemMatch` sirve para encontrar elementos individuales **que cumplan con múltiples criterios _TODOS JUNTOS_ (a manera de `and`)**, al contrario del funcionamiento normal sobre arrays, donde nos regresa los arreglos que cumplan con **_AL MENOS_** uno de los criterios **_POR SEPARADO_**.

### El operador `$slice`

El operador `$slice`, por su parte, "rebana" un arreglo de un documento para regresarnos solamente N elementos:

```javascript
db.articles.find({},{"purchase":{$slice:1}})
```

Este query nos regresará todos los documentos, pero su array `purchase` solo tendrá el 1er elemento. `$slice` acepta **números positivos** para "rebanar" el array de izq a derecha, y **números negativos** para "rebanarlo" de derecha a izq:}

```javascript
db.articles.find({},{"purchase":{$slice:-2}})
```

Del mismo modo, podemos usar el operador `$slice` para obtener un elemento en específico del array usando la forma `find({},{atributo:{$slice:[indice_inicio, numero_de_elementos]}}`. El siguiente comando traerá solamente el 2o elemento de los arrays `purchase`.

```javascript
db.articles.find({},{purchase:{$slice:[1,1]}})
```

Aquí nos posicionamos en el índice 1 (el 2o elemento), y a partir de ahí, traemos 1 elemento.

### Ejercicios

Usaremos la BD `restaurants.json` para estos ejercicios.

Primero debemos [descargar el archivo `restaurants.json` de este repo](https://github.com/xuxoramos/nosql-4-ds/blob/gh-pages/restaurants.zip).

Luego lo debemos cargar con `mongoimport`:

```
mongoimport --db=reviews --collection=restaurants --file=restaurants.json
```

Debemos tener esta salida:

![image](https://user-images.githubusercontent.com/1316464/131386416-2cb8e561-4ef0-462d-9d49-a78a736142e8.png)

La estructura de esta colección de documentos es la siguiente (aunque recuerden que no nos debemos fiar, porque MongoDB no tiene estructura predefinida).

```json
{
  "address": {
     "building": "1007",
     "coord": [ -73.856077, 40.848447 ],
     "street": "Morris Park Ave",
     "zipcode": "10462"
  },
  "borough": "Bronx",
  "cuisine": "Bakery",
  "grades": [
     { "date": { "$date": 1393804800000 }, "grade": "A", "score": 2 },
     { "date": { "$date": 1378857600000 }, "grade": "A", "score": 6 },
     { "date": { "$date": 1358985600000 }, "grade": "A", "score": 10 },
     { "date": { "$date": 1322006400000 }, "grade": "A", "score": 9 },
     { "date": { "$date": 1299715200000 }, "grade": "B", "score": 14 }
  ],
  "name": "Morris Park Bake Shop",
  "restaurant_id": "30075445"
}
```

Vamos a responder las siguientes preguntas:

1. Escribe una función find() para mostrar todos los documentos de la colección de restaurantes.

```javascript
db.restaurants.find()
```


2. Escribe una función find() para mostrar los campos restaurant_id, nombre, municipio y cocina para todos los documentos en el restaurante de la colección.

```javascript
db.restaurants.find({},{restaurant_id:1,name:1,borough:1,cuisine:1})
```


3. Escribe una función find() para mostrar los campos restaurant_id, nombre, municipio y cocina, pero excluya el campo \_id para todos los documentos de la colección restaurant.

```javascript
db.restaurants.find({},{restaurant_id:1,name:1,borough:1,cuisine:1,_id:0})
```


4. Escribe una función find() para mostrar los campos restaurant_id, nombre, municipio y código postal, pero excluya el campo \_id para todos los documentos de la colección restaurant.

```javascript
db.restaurants.find({},{restaurant_id:1,name:1,borough:1,"address.zipcode":1,_id:0})
```


5. Escribe una función find() para mostrar todos los restaurantes que se encuentran en el distrito del Bronx.

```javascript
db.restaurants.find({borough:"Bronx"})
```


6. Escribe una función find() para mostrar los primeros 5 restaurantes que se encuentran en el condado del Bronx.

```javascript
db.restaurants.find({borough:"Bronx"}).limit(5)
```


7. Escribe una función find() para mostrar los siguientes 5 restaurantes después de omitir los primeros 5 que se encuentran en el condado del Bronx.

```javascript
db.restaurants.find({borough:"Bronx"}).skip(5).limit(5)
```


8. Escribe una función find() para encontrar los restaurantes que obtuvieron una puntuación de más de 90.

```javascript
db.restaurants.find({"grades.score":{$gt:90}},{"grades.score":1})
```

Como podemos ver aquí, se cumple la regla de MongoDB donde en un query a un array, si todas las condiciones por separado son cumplidas por algunos elementos del array, se regresa todo el array.


9. Escribe una función find() para encontrar los restaurantes que obtuvieron una puntuación, más de 80 pero menos de 100.

El siguiente query cumple con la regla que mencionamos arriba.

```javascript
db.restaurants.find({"grades.score":{$gt:80,$lt:100}},{"grades.score":1})
```

Y por ello tenemos elementos del array de `score` como `131`, el cual es mayor a 80, y `11`, que es menor a 100.

Para buscar los elementos que cumplan **exactamente** con ambos criterios debemos usar el operador `$elemMatch`:

```javascript
db.restaurants.find({"grades":{$elemMatch:{"score":{$gt:80,$lt:100}}}},{"grades.score":1})
```

Y de ese modo obtenemos arreglos que tengan al menos 1 elemento que cumpla con ambos criterios.

10. Escribe una función find() para encontrar los restaurantes que se ubican en un valor de latitud menor que -95.754168.

Suponiendo que el atributo tipo array `coord` tiene la latitud en el índice 0:

```javascript
db.restaurants.find({"address.coord.0":{$lte:-95.754168}},{"address.coord":1})
```


11. Escribe una función find() para encontrar los restaurantes que no preparan ningún tipo de cocina de 'estadounidense' y su puntuación de calificación es superior a 70 y latitud inferior a -65.754168.

Tenemos 2 opciones. Sin expresiones regulares, usando el oeprador _not equals_ (`$ne`) y atendiendo que por alguna razón el tipo de cocina `"American "` tiene un espacio al final:

```javascript
db.restaurants.find({"cuisine":{$ne:"American "},"grades.score":{$gt:70},"address.coord.0":{$lt:-65.754168}},{"cuisine":1,"grades":1,"address.coord":1})
```

O con expresiones regulares y ayudándonos del operador booleano `$not`. Usamos el `^` para indicar "inicio de línea", y así evitar sacar del query a los restaurantes de cocina "Latin American/Caribbean":

```javascript
db.restaurants.find({"cuisine":{$not:/^American/},"grades.score":{$gt:70},"address.coord.0":{$lt:-65.754168}},{"cuisine":1,"grades":1,"address.coord":1})
```


12. Escribe una función find() para encontrar los restaurantes que no preparan ninguna cocina del continente americano y lograron una puntuación superior a 70 y se ubicaron en la longitud inferior a -65.754168.

```javascript
db.restaurants.find(
                           {
                             "cuisine" : {$ne : "American "},
                             "grades.score" :{$gt: 70},
                             "address.coord" : {$lt : -65.754168}
                            }
                     );
```


13. Escribe una función find() para encontrar los restaurantes que no preparan ninguna cocina del continente americano y obtuvieron una calificación de 'A' que no pertenece al distrito de Brooklyn. El documento debe mostrarse según la cocina en orden descendente.

```javascript
db.restaurants.find( {
                             "cuisine" : {$ne : "American "},
                             "grades.grade" :"A",
                             "borough": {$ne : "Brooklyn"}
                       } 
                    ).sort({"cuisine":-1});
```


14. Escribe una función find() para encontrar el ID del restaurante, el nombre, el municipio y la cocina de aquellos restaurantes que contienen 'Wil' como las primeras tres letras de su nombre.

```javascript
db.restaurants.find({name: /^Wil/}, {"restaurant_id":1, "name":1, "borough":1, "cuisine":1});
```


15. Escribe una función find() para encontrar el ID del restaurante, el nombre, el municipio y la cocina de aquellos restaurantes que contienen "ces" como las últimas tres letras de su nombre.

```javascript
db.restaurants.find({name: /ces$/},{"restaurant_id" : 1,"name":1,"borough":1,"cuisine" :1});
```


16. Escribe una función find() para encontrar el ID del restaurante, el nombre, el municipio y la cocina de aquellos restaurantes que contienen 'Reg' como tres letras en algún lugar de su nombre.

```javascript
db.restaurants.find( { "name": /Reg/ }, { "restaurant_id": 1, "name": 1, "borough": 1, "cuisine": 1 });
```

O alternativamente:

```javascript
db.restaurants.find( { "name": /.*Reg.*/ }, { "restaurant_id": 1, "name": 1, "borough": 1, "cuisine": 1 });
```


17. Escribe una función find() para encontrar los restaurantes que pertenecen al municipio del Bronx y que prepararon platos estadounidenses o chinos.

```javascript
db.restaurants.find(
	{ 
		"borough": "Bronx" , 
		$or : [
			{ "cuisine" : "American " },
			{ "cuisine" : "Chinese" }
		] 
	} 
);
```

18. Escribe una función find() para encontrar la identificación del restaurante, el nombre, el municipio y la cocina de los restaurantes que pertenecen al municipio de Staten Island o Queens o Bronxor Brooklyn.

```javascript
db.restaurants.find(
	{"borough" :
		{$in :["Staten Island","Queens","Bronx","Brooklyn"]}
	},
	{
		"restaurant_id" : 1,
		"name":1,
		"borough":1,
		"cuisine" :1
	}
);
```

19. Escribe una función find() para encontrar el ID del restaurante, el nombre, el municipio y la cocina de aquellos restaurantes que no pertenecen al municipio de Staten Island o Queens o Bronxor Brooklyn.

```javascript
db.restaurants.find(
	{"borough" :
		{$nin :["Staten Island","Queens","Bronx","Brooklyn"]}
	},
	{
		"restaurant_id" : 1,
		"name":1,
		"borough":1,
		"cuisine" :1
	}
);
```

20. Escribe una función find() para encontrar el ID del restaurante, el nombre, el municipio y la cocina de aquellos restaurantes que obtuvieron una puntuación que no sea superior a 10.

```javascript
db.restaurants.find(
	{"grades.score" : 
		{ $not: {$gt : 10}}
	},
	{
		"restaurant_id" : 1,
		"name":1,
		"borough":1,
		"cuisine" :1
	}
);
```

Alternativamente...

```javascript
db.restaurants.find(
	{"grades.score" : 
		{$lte : 10}
	},
	{
		"restaurant_id" : 1,
		"name":1,
		"borough":1,
		"cuisine" :1
	}
);
```


21. Escribe una función find() para encontrar el ID del restaurante, el nombre, el municipio y la cocina de aquellos restaurantes que prepararon platos excepto 'Americano' y 'Chinese' o el nombre del restaurante comienza con la letra 'Wil'.

```javascript
db.restaurants.find(
	{$or: [
		{name: /^Wil/}, 
		{"$and": [
			{"cuisine" : {$ne :"American "}}, 
			{"cuisine" : {$ne :"Chinese"}}	]
		}]
	}
	,{
		"restaurant_id" : 1,
		"name":1,
		"borough":1,
		"cuisine" :1
	}
);
```

22. Escribe una función find() para encontrar el ID del restaurante, el nombre y las calificaciones de los restaurantes que obtuvieron una calificación de "A" y obtuvieron una puntuación de 11 en un ISODate "2014-08-11T00: 00: 00Z" entre muchas de las fechas de la encuesta. .

```javascript
db.restaurants.find( 
	{
		"grades.date": ISODate("2014-08-11T00:00:00Z"), 
		"grades.grade":"A" , 
		"grades.score" : 11
	}, 
	{
		"restaurant_id" : 1,
		"name":1,
		"grades":1
	}
);
```

**👀OJO👀**: Aquí la palabra clave es _"entre muchas de las fechas de la encuesta"_, porque implica el comportamiento esperado de los queries sobre los arrays, en donde todos sus elementos deben de ayudar a cumplir todas las condiciones. En este caso, entre todos los grades deben ayudar a cumplir el criterio de 1) fecha del 11 de Agosto de 2014, 2) grade = A, y 3) score = 11.

23. Escribe una función find() para encontrar el ID del restaurante, el nombre y las calificaciones de aquellos restaurantes donde el segundo elemento de la matriz de calificaciones contiene una calificación de "A" y una puntuación de 9 en un ISODate "2014-08-11T00: 00: 00Z".

```javascript
db.restaurants.find( 
	{
		"grades.1.date":ISODate("2014-08-11T00:00:00Z"),
		"grades.1.grade":"A", 
		"grades.1.score" : 9
	},
	{
		restaurant_id" : 1,
		"name":1,
		"grades":1
	}
);
```

Si intentamos buscar estos criterios y que los satisfaga 1 y solo 1 elemento del array con `$elemMatch`:

```javascript
db.restaurants.find( {"grades.1": {$elemMatch:{"date": ISODate("2014-08-11T00:00:00Z"), "grade": "A", "score": 9 }}}, { "restaurant_id": 1, "name": 1, "grades": 1 });
```

No vamos a encontrar nada.

Esto es porque `$elemMatch` espera como entrada un array, y al apuntar la búsqueda a `grades.1` estamos pasando solo 1 elemento.

Si en lugar de `grades.1` pasamos todo el arreglo de `grades` a `$elemMatch`:

```javascript
db.restaurants.find( {"grades": {$elemMatch:{"date": ISODate("2014-08-11T00:00:00Z"), "grade": "A", "score": 9 }}}, { "restaurant_id": 1, "name": 1, "grades": 1 });
```

Nos regresa los 2 restaurantes cuyos `grades` tienen elementos que cumplen con los 3 criterios.

24. Escribe una función find() para encontrar el ID del restaurante, el nombre, la dirección y la ubicación geográfica para aquellos restaurantes donde el segundo elemento de la matriz de coordenadas contiene un valor que sea más de 42 y hasta 52.

```javascript
db.restaurants.find( 
	{ 
		"address.coord.1": {$gt : 42, $lte : 52}
	},
	{
		"restaurant_id" : 1,
		"name":1,
		"address":1,
		"coord":1
	}
);
```

25. Escribe una función find() para organizar el nombre de los restaurantes en orden ascendente junto con todas las columnas.

```javascript
db.restaurants.find().sort({"name":1});
```

26. Escribe una función find() para organizar el nombre de los restaurantes en orden descendente junto con todas las columnas.

```javascript
db.restaurants.find().sort({"name":-1});
```

27. Escribe una función find() para organizar el nombre de la cocina en orden ascendente y para ese mismo distrito de cocina debe estar en orden descendente.

```javascript
db.restaurants.find().sort({"cuisine":1,"borough" : -1,});
```

28. Escribe una función find() para saber si todas las direcciones contienen la calle o no.

```javascript
db.restaurants.find({"address.street" : { $exists : true } } );
```

Otras formas de checar existencia (o nulidad) son:

- usando la condición `{"address.street" : {$type : 10}}`, que checa que el tipo sea `null` (ver ejercicio 29)
- usando `{"address.street" : null}`

29. Escribe una función find() que seleccionará todos los documentos de la colección de restaurantes donde el valor del campo coord es Double.

```javascript
db.restaurants.find({"address.coord" : {$type : 1} } );
```

El operador `$type` nos permite explorar el tipo de dato que tiene un atributo. Recordemos que javascript es _weakly-typed_ y las variables no tienen tipo hasta que tienen un dato. A continuación los valores `$type` comunes:

| Type | Number | Alias | Notes |
|---|---|---|---|
| Double | 1 | "double" |  |
| String | 2 | "string" |  |
| Object | 3 | "object" |  |
| Array | 4 | "array" |  |
| Binary data | 5 | "binData" |  |
| Undefined | 6 | "undefined" | Deprecated. |
| ObjectId | 7 | "objectId" |  |
| Boolean | 8 | "bool" |  |
| Date | 9 | "date" |  |
| Null | 10 | "null" |  |
| Regular Expression | 11 | "regex" |  |
| 32-bit integer | 16 | "int" |  |
| Timestamp | 17 | "timestamp" |  |
| 64-bit integer | 18 | "long" |  |
| Decimal128 | 19 | "decimal" | New in version 3.4. |

30. Escribe una función find() que seleccionará el ID del restaurante, el nombre y las calificaciones para esos restaurantes que devuelve 0 como resto después de dividir la puntuación por 7.

```javascript
db.restaurants.find({"grades.score" : {$mod : [7,0]} }, {"restaurant_id" : 1,"name":1,"grades":1});
```


31. Escribe una función find() para encontrar el nombre del restaurante, el municipio, la longitud y la actitud y la cocina de aquellos restaurantes que contienen "mon" como tres letras en algún lugar de su nombre.

```javascript
db.restaurants.find(
	{
		name : {
			$regex : "mon.*", $options: "i"
		} 
	},
	{
		"name":1,
		"borough":1,
		"address.coord":1,
		"cuisine" :1
	}
);
```

El operador `$options` modifica como se comportará la expresión regular. En este caso, `$options:"i"` realiza una búsqueda _case insensitive_, por lo que va a hacer match con "Mon", "mon", "MON", "MoN", "moN", etc.

32. Escribe una función find() para encontrar el nombre del restaurante, el distrito, la longitud y la latitud y la cocina de aquellos restaurantes que contienen 'Mad' como las primeras tres letras de su nombre.

```javascript

db.restaurants.find(
	{
		name : {
			$regex : /^Mad/i
		} 
	},
	{
		"name":1,
		"borough":1,
		"address.coord":1,
		"cuisine" :1
	}
);
```

Al igual que el caso anterior, pero la ubicación de las opciones modificadoras de la expresión regular es dentro de la expresión misma mediante la sintaxis `/patrón_1/opción`, similar al comando `sed` de Unix.

### Agregaciones

Las agregaciones son queries que colapsan registros individuales en un solo resultado al mismo tiempo que aplican algún operador sobre ellos como conteo, sumar, promedios, etc.

Estas operaciones **destruyen información**, es decir, el promedio, suma o conteo del grupo colapsado pierde información de cada miembro individual.

Como analistas de datos pocas veces haremos agregaciones directo en las bases de datos fuente, y probablemente primero las movamos a nuestro data lake y ahí hacerlas. Pero si no tuvieramos uno, esta es la forma de hacerlas:

Para esta parte de la sesión vamos a crear la BD `3tdb` y laa colecciones `universities` y `courses`

```javascript
use 3tdb;
db.universities.insertMany([
{
  country : 'Spain',
  city : 'Salamanca',
  name : 'USAL',
  location : {
    type : 'Point',
    coordinates : [ -5.6722512,17, 40.9607792 ]
  },
  students : [
    { year : 2014, number : 24774 },
    { year : 2015, number : 23166 },
    { year : 2016, number : 21913 },
    { year : 2017, number : 21715 }
  ]
},
{
  country : 'Spain',
  city : 'Salamanca',
  name : 'UPSA',
  location : {
    type : 'Point',
    coordinates : [ -5.6691191,17, 40.9631732 ]
  },
  students : [
    { year : 2014, number : 4788 },
    { year : 2015, number : 4821 },
    { year : 2016, number : 6550 },
    { year : 2017, number : 6125 }
  ]
}
]);
db.courses.insertMany([
{
  university : 'USAL',
  name : 'Computer Science',
  level : 'Excellent'
},
{
  university : 'USAL',
  name : 'Electronics',
  level : 'Intermediate'
},
{
  university : 'USAL',
  name : 'Communication',
  level : 'Excellent'
}
]);
```

Las agregaciones en MongoDB se hacen a través de **PIPELINES**, esto tiene la siguiente forma:

```
funcion_f(w).funcion_g(x).funcion_h(y).funcion_i(z)
```

Estos pipelines son idénticos a hacer esto:

```javascript
funcion_f(funcion_g(funcion_h(funcion_i(w,x,y,z))))
```

Con la diferencia que esto es mucho menos legible.

Un pipeline primero debe ser definido:

```javascript
var pipeline = [
  {"$match": {
    "level": "Excellent",
  }},
    
  {"$sort": {
    "name": -1,
  }},      
    
  {"$limit": 1},  

  {"$unset": [
    "_id",
    "name",
    "level",
  ]},    
];
```

Si se fijan, estamos definiendo un arreglo de condiciones y operadores.

Para ejecutar este pipeline:

```javascript
db.courses.aggregate(pipeline);
```

Y para que MongoDB nos expliquen el execution plan:

```javascript
db.courses.explain("executionStats").aggregate(pipeline);
```

El resultado esperado es:

```javascript
[ { university: 'USAL' } ]
```

NOTAS:

1. Podemos acelerar el query con un índice. Qué campos incluirían en dicho índice?
2. Estamos usando `$unset` en lugar de `$project`, que es lo mismo que usar `{"atributo":[1|0]}`.
3. Oiga, pero esto podemos hacerlo con `find` solito!

Si, de este modo:

```javascript
db.courses.find(
    {"level": "Excellent"},
    {"_id": 0, "name": 0, "level": 0},
  ).sort(
    {"name": -1}
  ).limit(1);
```

Pero tenemos menos legibilidad y no podemos encadenar operaciones de agregación, como las que siguen.

En general, un pipeline de agregación en MongoDB tiene la siguiente forma:

![image](https://user-images.githubusercontent.com/1316464/133106415-6386846b-aac4-47cd-8ba3-e6d5a1f814f3.png)

Es una generalización de una secuencia de funciones:

1. `$match`: filtrado de todos los documentos que nos interesan para el query (como el `WHERE` en SQL). Se puede conjuntar con `$project`.
2. `$group`: agrega los renglones seleccionados previo a aplicar algun operador
3. `$sort`:  ordena los resultados de acuerdo a un criterio

El input de la agregación puede ser 1 o más documentos en array.

No hay límites en cuanto al num de elementos de cada tipo para el pipeline (les llamamos _stages_), es decir, podemos combinar cualquier número de operadores. **SIN EMBARGO** el límite por pipeline en cuanto a su memory footprint es de **100MB**.

### Stage `$match`

El primer stage en los pipelines de agregación es similar al `find()` para filtrar documentos en los que estamos interesados:

```javascript
db.universities.aggregate([
  { $match : { country : 'Spain', city : 'Salamanca' } }
])
```

Y al igual que el `find()`, podemos hacer `$project`:

```javascript
 db.universities.aggregate([ 
    { $match:{country: 'Spain', city: 'Salamanca'} },
    { $project:{_id : 0, country : 1, city : 1, name : 1} }
 ])
```

### Stage `$group`

El `group by` de MongoDB y el corazón de operaciones como count, sum, avg, etc.

```javascript
 db.universities.aggregate([ 
    { $match:{country: 'Spain', city: 'Salamanca'} },
    { $project:{_id : 0, country : 1, city : 1, name : 1} },
    { $group:{_id: "$name", conteo:{$sum:1}} },
    { $project:{_id : 0, "uni" : "$_id"} }
 ])
 ```
 
 👀OJO!👀 En el `$group` hay algunos elementos de sintaxis **mandatorios**:
 
 1. el atributo de agrupación se debe llamar **`_id`**.
    - Podemos renombrarlo agregando otro stage de `$project` así:
    ```javascript
    db.universities.aggregate([ 
	    { $match:{country: 'Spain', city: 'Salamanca'} },
	    { $project:{_id : 0, country : 1, city : 1, name : 1} },
	    { $group:{_id: "$name", conteo:{$sum:1}} },
	    { $project:{_id : 0, "uni" : "$_id"} }
    ])
    ```
 2. el atributo por el cual vamos a agregar debe ir con la notación `$` como si se tratara de una variable (porque lo es) y entrecomillado.
 3. el atributo en el cual guardaremos el resultado de la función de agregación puede llamarse como nosotros deseemos
 4. `{$sum:1}` es similar al `COUNT(*)` de SQL en el sentido de que va sumando 1 por cada documento que encuentra de acuerdo al stage de `$match`

#### Caso especial: agregación total (sin grupos)

En caso de que deseemos hacer una agregación de todos los documentos, sin armar grupos:

```javascript
db.universities.aggregate([ 
    { $match:{country: 'Spain', city: 'Salamanca'} },
    { $project:{_id : 0, country : 1, city : 1, name : 1} },
    { $group: { _id: null, conteo: { $count: {} } } }
    { $project: { _id: 0, conteo:1 } }
])
```

Resultado:

```javascript
[ { conteo: 2 } ]
```

### Stage `$out`

Toma la ejecución de toda la salida del pipeline y lo guarda en otra colección.

```javascript
db.universities.aggregate([ 
    { $match:{country: 'Spain', city: 'Salamanca'} },
    { $project:{_id : 0, country : 1, city : 1, name : 1} },
    { $group:{_id: "$name", conteo:{$sum:1}} },
    { $project:{_id : 0, "uni" : "$_id", conteo:1} },
    { $out:"miranomas" }
])
```

### Stage `$unwind`

Si nuestros documentos tienen arrays, el stage `$group` no nos permite llegar a ellos para agregarlos.

El stage `$unwind` nos permite un hack para darle la vuelta a esta limitante.

Lo que hace es explotar el array de un documento, tomar cada uno de los N elementos, y clavárselos a N copias del atributo que lo contiene.

En efecto, lo "desenrolla" 🤣

Por ejemplo:

```javascript
db.universities.aggregate([
  { $match : { name : 'USAL' } }
])
```

Esto obviamente nos regresa 1 documento:

```javascript
{
  country : 'Spain',
  city : 'Salamanca',
  name : 'USAL',
  location : {
    type : 'Point',
    coordinates : [ -5.6722512,17, 40.9607792 ]
  },
  students : [
    { year : 2014, number : 24774 },
    { year : 2015, number : 23166 },
    { year : 2016, number : 21913 },
    { year : 2017, number : 21715 }
  ]
}
```

Pero si corremos la siguiente agregación:

```javascript
db.universities.aggregate([
  { $match : { name : 'USAL' } },
  { $unwind : '$students' }
])
```

Entonces tenemos el siguiente resultado:

```javascript
{
	"_id" : ObjectId("5b7d9d9efbc9884f689cdba9"),
	"country" : "Spain",
	"city" : "Salamanca",
	"name" : "USAL",
	"location" : {
		"type" : "Point",
		"coordinates" : [
			-5.6722512,
			17,
			40.9607792
		]
	},
	"students" : {
		"year" : 2014,
		"number" : 24774
	}
}
{
	"_id" : ObjectId("5b7d9d9efbc9884f689cdba9"),
	"country" : "Spain",
	"city" : "Salamanca",
	"name" : "USAL",
	"location" : {
		"type" : "Point",
		"coordinates" : [
			-5.6722512,
			17,
			40.9607792
		]
	},
	"students" : {
		"year" : 2015,
		"number" : 23166
	}
}
{
	"_id" : ObjectId("5b7d9d9efbc9884f689cdba9"),
	"country" : "Spain",
	"city" : "Salamanca",
	"name" : "USAL",
	"location" : {
		"type" : "Point",
		"coordinates" : [
			-5.6722512,
			17,
			40.9607792
		]
	},
	"students" : {
		"year" : 2016,
		"number" : 21913
	}
}
{
	"_id" : ObjectId("5b7d9d9efbc9884f689cdba9"),
	"country" : "Spain",
	"city" : "Salamanca",
	"name" : "USAL",
	"location" : {
		"type" : "Point",
		"coordinates" : [
			-5.6722512,
			17,
			40.9607792
		]
	},
	"students" : {
		"year" : 2017,
		"number" : 21715
	}
}
```

👀OJO!👀 Fíjense en el `_id` que **ES EL MISMO** en todos los casos, esto es, es el mismo objeto `university` pero con el array `students` _descompuesto_ e insertado en copias de cada elemento.

#### Casos especiales

1. `$unwind` de un array vacío no regresará nada
2. `$unwind` de un atributo simple regresará el mismo _enclosing document_
3. `$unwind` de un array de un diccionario que tiene un 2o o 3er array, solo va a "desenrollar" el diccionario que solicitamos en ese operador, por lo que los otros arrays estarán repetidos

#### Para qué sirve esto?

Para hacer cosas como contar los registros de alumnos de 2017:

```javascript
db.universities.aggregate([
  { $unwind : '$students' },
  { $project : { _id : 0, 'students.year' : 1, 'students.number' : 1 } },
  { $match: {'students.year':2017}},
  { $group:{_id: "$students.year", conteo:{$count:{}}} },
])
```

O acumular los alumnos de cada año:

```javascript
db.universities.aggregate([
	{ $unwind: '$students' },
	{ $project: { _id: 0, "name": 1, 'students.year': 1, 'students.number': 1 } },
	{ $group: { _id: "$students.year", totalAlumnos: { $sum: "$students.number" } } },
	{$project:{_id:0,"ano":"$_id",totalAlumnos:1}}
])
```

O el promedio de alumnos de 2014 a 2017

```javascript
db.universities.aggregate([
	{ $unwind: '$students' },
	{ $project: { _id: 0, "name": 1, 'students.year': 1, 'students.number': 1 } },
	{ $group: { _id: "$name", promedioAlumnos: { $avg: "$students.number" } } },
	{$project:{_id:0,"uni":"$_id",promedioAlumnos:1}}
])
```

O cualquiera de estas funciones:

|Función|Descrip|
|---------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| $addToSet     | Después de agrupar, agrega elementos individuales a un array|
| $avg          | Promedio|
| $count        | Conteo (igual a `{$sum:1}`|
| $first        | Regresa el 1er documento o diccionario de cada grupo. ⚠️No confundir con el operador `$first` aplicable a arrays. Este operador no se ocupa del orden, eso se garantiza desde el stage `$sort` del pipeline |
| $last         | Regresa el último documento o diccionario de cada grupo. Mismas reglas y observaciones que `$first`|
| $max          | Regresa el máximo de cada grupo|
| $mergeObjects | Después de armar los grupos, combinar los objetos/diccionarios/documentos que correspondan al grupo en uno solo|
| $min          | Regresa el mínimo de cada grupo|
| $stdDevPop    | Regresa la [desviación estándar de la población](https://statistics.laerd.com/statistical-guides/measures-of-spread-standard-deviation.php) (entre _n_) de cada grupo|
| $stdDevSamp   | Regresa la [desviación estándar de la muestra](https://statistics.laerd.com/statistical-guides/measures-of-spread-standard-deviation.php) (entre _n-1_) de cada grupo|
| $sum          | Suma acumulativa de cada grupo|

#### Ejemplo `$addToSet`

Vamos a crear la sig colección en la BD que sea:

```javascript
db.sales.insertMany([
	{ "_id" : 1, "item" : "abc", "price" : 10, "quantity" : 2, "date" : ISODate("2014-01-01T08:00:00Z") },
	{ "_id" : 2, "item" : "jkl", "price" : 20, "quantity" : 1, "date" : ISODate("2014-02-03T09:00:00Z") },
	{ "_id" : 3, "item" : "xyz", "price" : 5, "quantity" : 5, "date" : ISODate("2014-02-03T09:05:00Z") },
	{ "_id" : 4, "item" : "abc", "price" : 10, "quantity" : 10, "date" : ISODate("2014-02-15T08:00:00Z") },
	{ "_id" : 5, "item" : "xyz", "price" : 5, "quantity" : 10, "date" : ISODate("2014-02-15T09:12:00Z") }
]);
```

Vemos que solo hay 2 fechas. Si queremos agrupar por esa fecha y aglutinar los `item` en un solo array, podemos hacer:

```javascript
db.sales.aggregate([
	{$group: 
		{_id: { day: { $dayOfYear: "$date"}, year: { $year: "$date" } }, itemsSold: { $addToSet: "$item" } }
	}
]);
```

👀OJO!👀 Estamos utilizando 2 operadores para objetos `ISODate`:

1. `$dayOfYear`: extrae de un objeto `ISODate` un dato numérico entre 1 y 365 (o 366 si es año bisiesto) representando el día del año.
2. `$year`: extrae de un objeto `ISODate` el año en numérico.

A continuación los operadores más comunes sobre `ISODate`:

| Función | Descripción y Ejemplo|
|-----------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| $dateAdd        | `{ $dateAdd: {startDate: ISODate("2020-10-31T12:10:05Z"), unit: "month", amount: 1} }` - Agrega `amount` al campo `unit` de la fecha `startDate`                                                                                                                            |
| $dateDiff       | `{ $dateDiff: { startDate: ISODate("2014-01-01T08:00:00Z"), endDate: ISODate("2014-02-03T09:00:00Z"), unit: "day"} }` - Regresa la diferencia en `unit` entre `startDate` y `endDate`  |
| $dateFromString | `{ $dateFromString: {dateString: "15-06-2018", format: "%d-%m-%Y"} }` - Parsea el string `dateString` representando una fecha en formato `format` para convertirlo en un objeto `ISODate` que contenga esa misma fecha.                                                                                                                            |
| $dateSubtract   | `{ $dateSubtract: {startDate: ISODate("2020-10-31T12:10:05Z"), unit: "month", amount: 1} }` - Susbtrae `amount` al campo `unit` de la fecha `startDate`                                                                                                                     |
| $dateToParts    | `$dateToParts: { date: ISODate("2017-01-01T01:29:09.123Z") }` - Descompone el `date` en sus partes. Retorna `"date" : {"year" : 2017, "month" : 1, "day" : 1, "hour" : 1, "minute" : 29, "second" : 9, "millisecond" : 123}`                                                                                                          |
| $dateToString   | `{ $dateToString: { format: "%Y-%m-%d %H:%M:%S", date: ISODate("2014-01-01T08:15:39.736Z") } }` - Convierte un `ISODate` en un string con una fecha formateada por `format`. Retorna `"2014-01-01 03:15:39"`. Ver [opciones de formato](https://docs.mongodb.com/manual/reference/operator/aggregation/dateToString/).                                                                                                                           |
| $dayOfMonth     | Los siguientes operadores tienen la sintaxis `{ $[operador]: [objeto ISODate] }`. Regresa un numérico entre 1 y 31 del objeto `ISODate`.                                                                                                    |
| $dayOfWeek      | Regresa un numérico entre 1 (Domingo) y 7 (Sábado) del objeto `ISODate`. |
| $dayOfYear      | Regresa un numérico entre 1 y 366 (bisiesto) del objeto `ISODate`. |
| $hour           | Regresa un numérico entre 0 y 23 del objeto `ISODate`. |
| $isoDayOfWeek   | Regresa un numérico entre 1 (Lunes) y 7 (Domingo) del objeto `ISODate`. No confundir con `$dayOfWeek` |
| $isoWeek        | Regresa un numérico entre 1 y 53 del objeto `ISODate`.  |
| $millisecond    | Regresa un numérico entre 0 y 999 del objeto `ISODate`. |
| $minute         | Reegresa un numérico entre 0 y 59 del objeto `ISODate`. |
| $month          | Regresa un numérico entre 1 (Enero) y 12 (Diciembre) del objeto `ISODate`. |
| $second         | Regresa un numérico entre 0 y 60 (cuando es _leap second_) del objeto `ISODate`. |
| $year           | Regresa el valor del año del objeto `ISODate`|

Posterior a armar los grupos con esas 2 únicas fechas, cada `item` será agregado a un array:

```javascript
{ "_id" : { "day" : 46, "year" : 2014 }, "itemsSold" : [ "xyz", "abc" ] }
{ "_id" : { "day" : 34, "year" : 2014 }, "itemsSold" : [ "xyz", "jkl" ] }
{ "_id" : { "day" : 1, "year" : 2014 }, "itemsSold" : [ "abc" ] }
```


#### Ejemplo `$mergeObjects`

Crearemos la sig colección en cualquier BD:

```javascript
db.sales.insert( [
   { _id: 1, year: 2017, item: "A", quantity: { "2017Q1": 500, "2017Q2": 500 } },
   { _id: 2, year: 2016, item: "A", quantity: { "2016Q1": 400, "2016Q2": 300, "2016Q3": 0, "2016Q4": 0 } } ,
   { _id: 3, year: 2017, item: "B", quantity: { "2017Q1": 300 } },
   { _id: 4, year: 2016, item: "B", quantity: { "2016Q3": 100, "2016Q4": 250 } }
])
```

Vamos a agrupar por `item` y vamos a crear un diccionario con todos los `quantity` en un atributo llamado `mergedSales`:

```javascript
db.sales.aggregate( [
   { $group: { _id: "$item", mergedSales: { $mergeObjects: "$quantity" } } }
])
```

El resultado debe ser:

```javascript
{ "_id" : "B", "mergedSales" : { "2017Q1" : 300, "2016Q3" : 100, "2016Q4" : 250 } }
{ "_id" : "A", "mergedSales" : { "2017Q1" : 500, "2017Q2" : 500, "2016Q1" : 400, "2016Q2" : 300, "2016Q3" : 0, "2016Q4" : 0 } }
```

### Stages `$sort` y `$limit` 

El sort y el limit puede usarse como stage de un pipeline de agregación, o puede usarse _standalone_ como lo hemos hecho antes para ordenar resulsets individuales.

### Stage `$addFields`

Crea campos nuevos basados en las agregaciones, como una suma concentrada final, o un promedio concentrado final.

⚠️No confundir con el `$group`, el `$addFields` NO AGREGA NI AGRUPA.⚠️

Regresemos a nuestra BD de reviews de restaurantes con `use reviews`

La estructura de cada review es:

```javscript
{
  _id: ObjectId("612d222983a7f8a60c193d14"),
  address: {
    building: '351',
    coord: [ -73.98513559999999, 40.7676919 ],
    street: 'West   57 Street',
    zipcode: '10019'
  },
  borough: 'Manhattan',
  cuisine: 'Irish',
  grades: [
    { date: ISODate("2014-09-06T00:00:00.000Z"), grade: 'A', score: 2 },
    {
      date: ISODate("2013-07-22T00:00:00.000Z"),
      grade: 'A',
      score: 11
    },
    {
      date: ISODate("2012-07-31T00:00:00.000Z"),
      grade: 'A',
      score: 12
    },
    {
      date: ISODate("2011-12-29T00:00:00.000Z"),
      grade: 'A',
      score: 12
    }
  ],
  name: 'Dj Reynolds Pub And Restaurant',
  restaurant_id: '30191841'
}
```

Cómo podemos agregar un atributo a cada restaurante para tener su score total agregado de todos sus reviews y su promedio?

```javascript
db.restaurants.aggregate([
	{$unwind:"$grades"},
	{$project:{"grades.score":1, "name":1}},
	{$group:{_id:"$name", "gradeArray":{$push:"$grades.score"}}},
	{$project:{"name":"$_id",_id:0,"gradeArray":1}},
	{$addFields:{"totalScore":{$sum:"$gradeArray"},"avgScore":{$avg:"$gradeArray"}}}
])
```

Desmenucemos este query para entenderlo:

1. "desenrollo" el array `grades` y le clavo cada elemento a una copia del _enclosing object_.
2. quito toda la paja y me quedo con los scores y el nombre del restaurante
3. agrupo por nombre de restaurante - esto en SQL es una mala práctica, PERO en MongoDB y en general en bases de datos de documentos, se vale. Esto nos sirve para poder ejecutar el operador `$push`, que clava un array a un objeto. En esta línea lo que estamos haciendo es efectivamente **CONVERTIR** el diccionario que tiene los scores en un arreglo normalito.
4. Ya con el arreglo, renombramos el `_id` del grupo
5. Y sumamos horizontalmente los scores del array, así como su promedio.

### Stage `$sortByCount`

Es un operador que funge como si tuviéramos:

```javascript
db.collection.aggregate([
	{ $group: { _id: <expression>, count: { $sum: 1 } } },
	{ $sort: { count: -1 } }
])
```

Insertemos esta base de datos de obras de arte:

```javascript
db.artwork.insertMany([
	{ "_id" : 1, "title" : "The Pillars of Society", "artist" : "Grosz", "year" : 1926, "tags" : [ "painting", "satire", "Expressionism", "caricature" ] },
	{ "_id" : 2, "title" : "Melancholy III", "artist" : "Munch", "year" : 1902, "tags" : [ "woodcut", "Expressionism" ] },
	{ "_id" : 3, "title" : "Dancer", "artist" : "Miro", "year" : 1925, "tags" : [ "oil", "Surrealism", "painting" ] },
	{ "_id" : 4, "title" : "The Great Wave off Kanagawa", "artist" : "Hokusai", "tags" : [ "woodblock", "ukiyo-e" ] },
	{ "_id" : 5, "title" : "The Persistence of Memory", "artist" : "Dali", "year" : 1931, "tags" : [ "Surrealism", "painting", "oil" ] },
	{ "_id" : 6, "title" : "Composition VII", "artist" : "Kandinsky", "year" : 1913, "tags" : [ "oil", "painting", "abstract" ] },
	{ "_id" : 7, "title" : "The Scream", "artist" : "Munch", "year" : 1893, "tags" : [ "Expressionism", "painting", "oil" ] },
	{ "_id" : 8, "title" : "Blue Flower", "artist" : "O'Keefe", "year" : 1918, "tags" : [ "abstract", "painting" ] },
])
```

Si ejecutamos la siguiente agregación:

```javascript
db.exhibits.aggregate( [ { $unwind: "$tags" },  { $sortByCount: "$tags" } ] )
```

Tendremos la salida:

```javascript
{ "_id" : "painting", "count" : 6 }
{ "_id" : "oil", "count" : 4 }
{ "_id" : "Expressionism", "count" : 3 }
{ "_id" : "Surrealism", "count" : 2 }
{ "_id" : "abstract", "count" : 2 }
{ "_id" : "woodblock", "count" : 1 }
{ "_id" : "woodcut", "count" : 1 }
{ "_id" : "ukiyo-e", "count" : 1 }
{ "_id" : "satire", "count" : 1 }
{ "_id" : "caricature", "count" : 1 }
```

Esto es, cuenta los elementos comunes y los ordena por el num de ocurrencias.

### Queries analíticos avanzados

1. Cuál es el promedio de `score` por `type` de evaluación y por `class_id` en la BD `sample_training` en la colección `grades`?

Para esto debemos descargar [esta BD de calificaciones](https://github.com/xuxoramos/nosql-4-ds/blob/gh-pages/grades.zip) e insertarla con `mongoimport`:

```
mongoimport --db=sample_training --collection=grades
```

Primero debemos enterarnos de qué va la BD. Vamos a sacar los primeros 3 registros para ver de qué tratan:

```javascript
use sample_training
db.grades.find().limit(3)

[
  {
    _id: ObjectId("56d5f7eb604eb380b0d8d8ce"),
    student_id: 0,
    scores: [
      { type: 'exam', score: 78.40446309504266 },
      { type: 'quiz', score: 73.36224783231339 },
      { type: 'homework', score: 46.980982486720535 },
      { type: 'homework', score: 76.67556138656222 }
    ],
    class_id: 339
  },
  {
    _id: ObjectId("56d5f7eb604eb380b0d8d8d6"),
    student_id: 0,
    scores: [
      { type: 'exam', score: 25.926204502143857 },
      { type: 'quiz', score: 37.23668404170315 },
      { type: 'homework', score: 19.609679551976487 },
      { type: 'homework', score: 98.7923690220697 }
    ],
    class_id: 108
  },
  {
    _id: ObjectId("56d5f7eb604eb380b0d8d8da"),
    student_id: 1,
    scores: [
      { type: 'exam', score: 95.4702770345805 },
      { type: 'quiz', score: 59.14125426136129 },
      { type: 'homework', score: 34.32836889016887 },
      { type: 'homework', score: 84.07534235774499 }
    ],
    class_id: 237
  }
]
```

Parece que son calificaciones de un alumno, de una clase, para diferentes mecanismos de evaluación: examen, quiz, y tareas.

Qué tipo de relación hay entre `student_id` y `class_id`? Cuál es el punto de vista de esta estructura? "Una clase tiene N alumnos?", o "un alumno tiene N clases?".

Primero, veamos cuantos registros tenemos:

```javascript
db.grades.find().count()

1000000
```

Si la perspectiva está anclada en `class_id`, entonces deberíamos tener 1000000 clases, o 1000000 estudiantes si la perspectiva está en `student_id`.

```javascript
db.grades.distinct("class_id")

[
   0,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11,
  12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23,
  24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35,
  36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47,
  48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59,
  60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71,
  72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83,
  84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95,
  96, 97, 98, 99,
  ... 401 more items
]
```

De acuerdo a esto, el universo de clases es mucho menor, por lo que probablemente esta colección esté armada desde la perspectiva del estudiante.

```javascript
db.grades.distinct("student_id")

[
   0,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11,
  12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23,
  24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35,
  36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47,
  48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59,
  60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71,
  72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83,
  84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95,
  96, 97, 98, 99,
  ... 9900 more items
]
```

Ahora vamos a tratar de armar el query para dar respuesta a la pregunta inicial:

```javascript
db.grades.aggregate([
	{$unwind:"$scores"},{$project:{"student_id":0}},
	{$group:{_id:{"clase":"$class_id","eval":"$scores.type"}, "promedio":{$avg:"$scores.score"}}},
	{$sort:{"_id.clase":1,"_id.eval":1}}])
])
```

Nuestro resultado es:

```javascript
[
  { _id: { clase: 0, eval: 'exam' }, promedio: 46.224870203904395 },
  { _id: { clase: 0, eval: 'homework' }, promedio: 49.6592370300883 },
  { _id: { clase: 0, eval: 'quiz' }, promedio: 49.38124259163944 },
  { _id: { clase: 1, eval: 'exam' }, promedio: 50.78357850094616 },
  { _id: { clase: 1, eval: 'homework' }, promedio: 49.18339520790678 },
  { _id: { clase: 1, eval: 'quiz' }, promedio: 51.68365158823541 },
  { _id: { clase: 2, eval: 'exam' }, promedio: 51.212269415215715 },
  { _id: { clase: 2, eval: 'homework' }, promedio: 48.635517471345494 },
  { _id: { clase: 2, eval: 'quiz' }, promedio: 49.22183768413837 },
  { _id: { clase: 3, eval: 'exam' }, promedio: 49.24088016851434 },
  { _id: { clase: 3, eval: 'homework' }, promedio: 49.32958980280401 },
  { _id: { clase: 3, eval: 'quiz' }, promedio: 49.70705542324686 },
  { _id: { clase: 4, eval: 'exam' }, promedio: 48.45214274611575 },
  { _id: { clase: 4, eval: 'homework' }, promedio: 51.336198599567986 },
  { _id: { clase: 4, eval: 'quiz' }, promedio: 52.186677392814204 },
  { _id: { clase: 5, eval: 'exam' }, promedio: 51.91626171544547 },
  { _id: { clase: 5, eval: 'homework' }, promedio: 49.71133512774075 },
  { _id: { clase: 5, eval: 'quiz' }, promedio: 48.17571458478485 },
  { _id: { clase: 6, eval: 'exam' }, promedio: 54.20236080762028 },
  { _id: { clase: 6, eval: 'homework' }, promedio: 49.441178234623834 },
  ...,
  ...,
  ...,
]
```

### Stage `$lookup`

Este stage nos permite hacer un **join** entre la colección sobre la que estamos operando y una colección de **lookup**.

Se recomienda que ambas colecciones estén **en la misma BD**.

Al igual que las operaciones join en SQL, necesitamos que ambas colecciones tengan al menos 1 atributo idéntico cada uno **los cuales podamos asociar con una condición de igualdad**. Recordemos que en MongoDB los `ObjectID` no siguen (ni tienen por qué seguir) las mejores prácticas de identificadores que en SQL.

Para este ejercicio vamos a importar 2 colecciones a la BD `lookup`:

```javascript
use lookup

db.orders.insert([
   { "_id" : 1, "item" : "almonds", "price" : 12, "quantity" : 2 },
   { "_id" : 2, "item" : "pecans", "price" : 20, "quantity" : 1 },
   { "_id" : 3  }
])

db.inventory.insert([
   { "_id" : 1, "sku" : "almonds", "description": "product 1", "instock" : 120 },
   { "_id" : 2, "sku" : "bread", "description": "product 2", "instock" : 80 },
   { "_id" : 3, "sku" : "cashews", "description": "product 3", "instock" : 60 },
   { "_id" : 4, "sku" : "pecans", "description": "product 4", "instock" : 70 },
   { "_id" : 5, "sku": null, "description": "Incomplete" },
   { "_id" : 6 }
])
```

Y luego corremos el operador `$lookup` como parte de un pipeline de la función `.aggregate()`

```javascript
db.orders.aggregate([
   {
     $lookup:
       {
         from: "inventory",
         localField: "item",
         foreignField: "sku",
         as: "inventory_docs"
       }
  }
])
``` 

El resultado es:

```javascript
{
   "_id" : 1,
   "item" : "almonds",
   "price" : 12,
   "quantity" : 2,
   "inventory_docs" : [
      { "_id" : 1, "sku" : "almonds", "description" : "product 1", "instock" : 120 }
   ]
}
{
   "_id" : 2,
   "item" : "pecans",
   "price" : 20,
   "quantity" : 1,
   "inventory_docs" : [
      { "_id" : 4, "sku" : "pecans", "description" : "product 4", "instock" : 70 }
   ]
}
{
   "_id" : 3,
   "inventory_docs" : [
      { "_id" : 5, "sku" : null, "description" : "Incomplete" },
      { "_id" : 6 }
   ]
}
```

Posterior a esto podríamos continuar el pipeline, por ejemplo, para contar los `inventory_docs` por diccionario:

```javascript
db.orders.aggregate([
   {
     $lookup:
       {
         from: "inventory",
         localField: "item",
         foreignField: "sku",
         as: "inventory_docs"
       }
  },
  {$unwind:"$inventory_docs"},
  {$group: {_id:"$_id", numDocs:{$count:{}}}}
])
```

