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

De acuerdo a la documentación, este comando "crea una base de datos", pero esto no es enteramente cierto. Esto solo aparta espacio en el MongoDB para comenzar a agregar documentos (ojo: no son registros). No tenemos una BD formalmente creada hasta no agregar documentos a esa BD. Vamos a agregar un documento:

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

![image](https://user-images.githubusercontent.com/1316464/129617366-03acc5a7-cdc1-4e57-866f-f1d037a9eeb7.png)

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

Examinémosla con Robo 3T:




