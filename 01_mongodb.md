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

Qué función `find()` necesitamos para obtener los "artículos" con `stock` en el `warehouse` 01 en el `country` 02?

```javascript
// TBD: respuesta de mis queridos alumnos
```

Ese query nos va a regresar esto:

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

Qué `_id` tiene el documento de 1er nivel en donde uno de sus subdocumentos cumple con nuestras condiciones del query❓

```javascript
/// TBD respuesta de mis queridos alumnos
```

Como podemos ver, el documento de 1er nivel con `_id` 2 solo cumple con 1 de las condiciones, por lo tanto este query nos puede regresar resultados espurios.

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


2. Escribe una función find() para mostrar los campos restaurant_id, nombre, municipio y cocina para todos los documentos en el restaurante de la colección.


3. Escribe una función find() para mostrar los campos restaurant_id, nombre, municipio y cocina, pero excluya el campo _id para todos los documentos de la colección restaurant.


4. Escribe una función find() para mostrar los campos restaurant_id, nombre, municipio y código postal, pero excluya el campo _id para todos los documentos de la colección restaurant.


5. Escribe una función find() para mostrar todos los restaurantes que se encuentran en el distrito del Bronx.


6. Escribe una función find() para mostrar los primeros 5 restaurantes que se encuentran en el condado del Bronx.


7. Escribe una función find() para mostrar los siguientes 5 restaurantes después de omitir los primeros 5 que se encuentran en el condado del Bronx.


8. Escribe una función find() para encontrar los restaurantes que obtuvieron una puntuación de más de 90.


9. Escribe una función find() para encontrar los restaurantes que obtuvieron una puntuación, más de 80 pero menos de 100.


10. Escribe una función find() para encontrar los restaurantes que se ubican en un valor de latitud menor que -95.754168.


11. Escribe una función find() para encontrar los restaurantes que no preparan ningún tipo de cocina de 'estadounidense' y su puntuación de calificación es superior a 70 y latitud inferior a -65.754168.


12. Escribe una función find() para encontrar los restaurantes que no preparan ninguna cocina de 'estadounidense' y lograron una puntuación superior a 70 y se ubicaron en la longitud inferior a -65.754168.


13. Escribe una función find() para encontrar los restaurantes que no preparan ninguna cocina de 'estadounidense' y obtuvieron una calificación de 'A' que no pertenece al distrito de Brooklyn. El documento debe mostrarse según la cocina en orden descendente.


14. Escribe una función find() para encontrar el ID del restaurante, el nombre, el municipio y la cocina de aquellos restaurantes que contienen 'Wil' como las primeras tres letras de su nombre.


15. Escribe una función find() para encontrar el ID del restaurante, el nombre, el municipio y la cocina de aquellos restaurantes que contienen "ces" como las últimas tres letras de su nombre.


16. Escribe una función find() para encontrar el ID del restaurante, el nombre, el municipio y la cocina de aquellos restaurantes que contienen 'Reg' como tres letras en algún lugar de su nombre.


17. Escribe una función find() para encontrar los restaurantes que pertenecen al municipio del Bronx y que prepararon platos estadounidenses o chinos.


18. Escribe una función find() para encontrar la identificación del restaurante, el nombre, el municipio y la cocina de los restaurantes que pertenecen al municipio de Staten Island o Queens o Bronxor Brooklyn.


19. Escribe una función find() para encontrar el ID del restaurante, el nombre, el municipio y la cocina de aquellos restaurantes que no pertenecen al municipio de Staten Island o Queens o Bronxor Brooklyn.


20. Escribe una función find() para encontrar el ID del restaurante, el nombre, el municipio y la cocina de aquellos restaurantes que obtuvieron una puntuación que no sea superior a 10.


21. Escribe una función find() para encontrar el ID del restaurante, el nombre, el municipio y la cocina de aquellos restaurantes que prepararon platos excepto 'Americano' y 'Chinees' o el nombre del restaurante comienza con la letra 'Wil'.


22. Escribe una función find() para encontrar el ID del restaurante, el nombre y las calificaciones de los restaurantes que obtuvieron una calificación de "A" y obtuvieron una puntuación de 11 en un ISODate "2014-08-11T00: 00: 00Z" entre muchas de las fechas de la encuesta. .


23. Escribe una función find() para encontrar el ID del restaurante, el nombre y las calificaciones de aquellos restaurantes donde el segundo elemento de la matriz de calificaciones contiene una calificación de "A" y una puntuación de 9 en un ISODate "2014-08-11T00: 00: 00Z".


24. Escribe una función find() para encontrar el ID del restaurante, el nombre, la dirección y la ubicación geográfica para aquellos restaurantes donde el segundo elemento de la matriz de coordenadas contiene un valor que sea más de 42 y hasta 52 ..


25. Escribe una función find() para organizar el nombre de los restaurantes en orden ascendente junto con todas las columnas.


26. Escribe una función find() para organizar el nombre de los restaurantes en orden descendente junto con todas las columnas.


27. Escribe una función find() para organizar el nombre de la cocina en orden ascendente y para ese mismo distrito de cocina debe estar en orden descendente.


28. Escribe una función find() para saber si todas las direcciones contienen la calle o no.


29. Escribe una función find() que seleccionará todos los documentos de la colección de restaurantes donde el valor del campo coord es Double.


30. Escribe una función find() que seleccionará el ID del restaurante, el nombre y las calificaciones para esos restaurantes que devuelve 0 como resto después de dividir la puntuación por 7.


31. Escribe una función find() para encontrar el nombre del restaurante, el municipio, la longitud y la actitud y la cocina de aquellos restaurantes que contienen "mon" como tres letras en algún lugar de su nombre.


32. Escribe una función find() para encontrar el nombre del restaurante, el distrito, la longitud y la latitud y la cocina de aquellos restaurantes que contienen 'Mad' como las primeras tres letras de su nombre.



