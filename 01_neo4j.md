# Bases de Datos de Grafos - Neo4j

Mientras que para las BDs relacionales, cada registro es un ejemplo o instancia de una entidad, o una tabla, y las relaciones son solo llaves copiadas entre tablas, para las BDs de grafos lo que se almacena es la conexión o la relación entre 2 nodos.

Encima de esto, dado que cada nodo puede tener diferentes atributos, las bases de datos de grafos **no tienen esquema**, lo cual las vuelve un poco lentas para la escritura VS las BDs relacionales, sobre todo cuando llegamos al orden de millones de registros, debido a que cada nodo tendrá estructura diferente y no tenemos una estructura definida y fija como una tabla.

## Otros ejemplos de BDs de grafos

- Amazon Neptune
- ArangoDB
- TerminusDB

## El modelo de datos de una BD de grafos

Los componentes principales de una BD de grafos es:

1. **Nodos:** los objetos principales en un diseño de grafos. Representan una entidad particular (una instancia de un objeto, o una instancia de un sustantivo - i.e. no representan un _tipo de objeto_ o _clase_, sino el _objeto en sí_, no un tipo de _marca_ropa_deportiva_ sino _adidas_, o _Patagonia_).
2. **Edges:** la conexión entre 2 nodos. Representa una relación (un verbo) entre 2 objetos.
3. **Labels/Etiquetas:** podemos usarlas para definir una _clase_ o _tipo de objeto_ y así agrupar nodos con características comunes.
4. **Propiedades:** dentro de cada _nodo_ o _edge_ podemos tener un diccionario _llave:valor_ con propiedades que califican a cada uno.

### Modelo lógico de grafos

Supongamos la siguiente narrativa:

> _"Si, mira, quiero una red social mamalona, como facebook, pero que llamaremos "el libro de caras del bienestar" ya sabes? que nuestros datos no estén en servidores gringos neoliberales, sino en infraestructura 100% mexicana. Primero tendremos solamente **Personas** y **Lugares**, y cada **Persona** tendrá su **nombre**, **lugar**, **género** y **correo electrónico**; mientras que cada **Lugar** tendrá **nombre**, **latitud** y **longitud**. Las **Personas** estarán conectadas por una relación llamada **Amistad**, donde podemos tener **años de amistad**, y mientras que los **Lugares** estarán conectados a las **Personas** a través de la relación llamada **Vive en**, sin atributos, de momento."_

Cómo podemos representar esto?

1. **Nodos**
<details><summary>Ustedes primero</summary>

  ![image](https://user-images.githubusercontent.com/1316464/138636550-a2a764fb-9aaf-426a-9971-eb0b6bf0ca84.png)

</details>

2. **Edges**
<details><summary>Ustedes primero</summary>

  ![image](https://user-images.githubusercontent.com/1316464/138636703-d4d27127-a699-476e-ad70-2b4c11514c41.png)

</details>

3. **Labels**
<details><summary>Ustedes primero</summary>

  ![image](https://user-images.githubusercontent.com/1316464/138636927-632cbd4c-40db-458d-95c5-762b85e53da0.png)

</details>


4. **Propiedades**

<details><summary>Ustedes primero</summary>
  
  ![image](https://user-images.githubusercontent.com/1316464/138636864-c540e915-5266-4104-8969-8ad46e899312.png)
  
</details>

Ya podemos ver las diferencias entre los modelos relacionales y los de grafos.

Desde el punto de vista lógico, el modelo de grafos no promueve la **generalización**, es decir, el centrarnos en **clases** o **entidades**, sino en los ejemplos específicos de cada clase o entidad, sino que promueve la **especialización**. No nos importa que un _Person_ viva en un _Location_, sino que _Karl Marx_ vive en _Alemania_, y solo como **conveniencia** usamos la etiqueta _Person_ para agrupar a Karl Marx junto a, a aunque se revuelque en su tumba, Adam Smith, Keynes, Miller, Modigliani, _et al_.

### Modelo físico de grafos

Aparte de tener un nodo por cada _Person_, la diferencia más grande es al recorrer relaciones o **edges**.

La forma de registrar que 2 nodos están relacionados con un edge en bases de datos de grafos es utilizando **apuntadores**, esto es, direcciones de memoria que nos llevan de un lugar dentro de ella donde está un nodo, a otro lugar en la memoria donde está otro nodo.

Esto se le llama ["Index-free Adjacency"](https://thomasvilhena.com/2019/08/index-free-adjacency). Esto es, no necesitamos un diccionario, ni una operación de intersección de conjuntos, ni un mapeo de columnas, como en los modelos relacionales, para poder ir de una tabla a otra.

En las BDs de grafos, las relaciones se encuentran ya físicamente en memoria, expresadas con el objeto de más bajo nivel que nuestra máquina puede utilizar. Esto implica que el recorrer un grafo para ir de nodo en nodo recolectando información, el performance **NO DEPENDE DEL TAMAÑO DEL GRAFO**! Podemos decir que tiene un running time linear de **O(n)**, donde N es el num de edges a recorrer, que siempre será más reducido que la N involucrada en un `JOIN`.

En contraste, las BDs relacionales al viajar de una tabla a otra con `JOIN`, estamos utilizando una operación de intersección para ver en qué parte los 2 índices de las 2 llaves de la relación se traslapan, y por tanto su performance **DISMINUYE A MEDIDA QUE HAY MÁS REGISTROS**. Podemos decir que los `JOIN` tienen un running time de **O(log n)** cuando usamos llaves indexadas, y **O(n^2)** cuando son _soft joins_ entre columnas que no son llaves o tienen índices.

## Cuándo SÍ debemos usar una BD de grafos?

### 1. Cuando mis datos estén altamente conectados

Esto es, cuando el elemento central para nuestro análisis sea la conexión o la relación entre entidades particulares, y por tanto nuestros datos **no sean transaccionales**, entonces probablemente nos conviene una BD de grafos. Frecuentemente solo es necesario guardar los datos y realizar análisis sofisticado después.

### 2. Cuando la lectura sea más importante que la escritura

Si mi problema es _transaccional_ en su naturaleza, y los analíticos que voy a ejecutar en estos datos con ayuda de `JOIN` no recorren la mayoría de las entidades, entonces quizá no requiera una BD de grafos.

### 3. Cuando mi modelo de datos cambie constantemente

Dado que las BDs de grafos **no tienen esquema**, al igual que las [Document Databases](https://github.com/xuxoramos/nosql-4-ds/blob/gh-pages/01_mongodb.md), y por tanto para cada nodo o edge podemos agregar atributos a como deseemos, serán adecuadas cuando tengamos alto nivel de incertidumbre sobre la definición de nuestros datos, y a la postre nos permitirán que no todos los nodos tengan forzosamente valores en todos los atributos, y los que tengan, que no sean consistentes en cuanto a los tipos (i.e. el nodo "Adam Smith" tendrá el atributo _tiene\_sentido\_del\_humor_ en **FALSE**, mientras que el nodo "Milton Friedmann" lo tendrá en **TRUE**, y finalmente, el nodo "Karl Marx" lo tendrá en **null**).

## Cuándo NO debemos usar una BD de grafos?

### 1. Cuando mis analíticos hagan _table scan_ constantemente

Cuando los analíticos que vaya a correr sobre esos datos impliquen constantes _table scans_ sean parciales o _full_, o secuenciales o con índices, entonces una BD de grafos puede que no sea la mejor opción.

### 2. Cuando mis búsquedas por ID sean constantes

Como vimos en [nuestra intro a BDs columnares](https://github.com/xuxoramos/nosql-4-ds/blob/gh-pages/01_monetdb.md), los queries propios de una solución transaccional siempre obtienen todo el renglón, no se centran en relaciones, y frecuentemente buscarán un ID en toda la tabla. Esto es porque este tipo de queries no aprovecha el performance que dan las BDs de grafos para recorrer varios nodos.

### 3. Cuando debamos almacenar atributos de gran tamaño

Por ejemplo, si para un nodo hipotético "AMLO" debemos poner un atributo _mañanera_, y ahí debemos de colocar TODAS esas conferencias, resultará en un atributo de varios cientos de gigas. El precio de este almacenamiento es alto comparado con la capacidad de movernos y viajar a lo largo de nodos para recoger información.

## ⚠️ Ya se dieron cuenta? ⚠️

Ya se dieron cuenta que todas las BDs alternas a PostgreSQL están orientadas a analíticos?

Entonces hace todo el sentido del mundo que tengamos al frente de nuestra administración de datos una BD relacional para capturar TODO LO TRANSACCIONAL, y luego, dependiendo del tipo de analíticos que deseemos hacer, mover estos datos a una BD que propicie dicha actividad.

1. Alimentar dashboards o modelos de ML: MonetDB o BDs columnares
2. Redes de corrupción/fraude o investigaciones judiciales: Neo4j o BDs de grafos
3. Visualización de actividad web o de APIs: MongoDB o BDs de documentos

Pero necesitamos un "buffer" intermedio para no cargarle la mano a ese PostgreSQL. Ese buffer intermedio es el Data Lake que veremos al rato 😉.

## Instalando Neo4j en AWS EC2

Para esto usaremos sus cuentas de AWS Academy.

### 1. Entrar al Lab

Debió llegarles un correo de invitación a AWS Academy.

Esto los va a llevar a algo similar a Canvas.

⚠️ **ESTE CANVAS NO TIENE NADA QUE VER CON EL CANVAS DEL ITAM, ES OTRO CANVAS TOTALMENTE DIFERENTE** ⚠️

Una vez dentro van a ver esta pantalla:

![image](https://user-images.githubusercontent.com/1316464/138730354-816f01ec-1b2f-4d97-b566-957f80f84dba.png)

Hay que dar click en **Modules**:

![image](https://user-images.githubusercontent.com/1316464/138730574-e92d9a92-3d67-4201-a25a-2c40d5ad22f3.png)

Y luego en **Learner Lab - Foundational Services**:

![image](https://user-images.githubusercontent.com/1316464/138730772-98f20a63-c810-4d29-8da0-19eddc20a1ea.png)

Y esperar a que cargue la terminal de nuestro ambiente.

En esta pantalla debemos de verificar que nuestra "instancia" del AWS Lab esté abajo. Lo sabemos por el "foquito rojo" arriba de la terminal:

![image](https://user-images.githubusercontent.com/1316464/138731704-e0c71604-dbdc-48e2-a06a-e4cdbbdf9808.png)

Para arrancar nuestro lab, debemos dar click a **Start Lab**:

![image](https://user-images.githubusercontent.com/1316464/138732001-ac08560f-255c-40a1-8b8a-fc5a8c9b43c6.png)


Después de unos buenos mins, tendremos esto:

![image](https://user-images.githubusercontent.com/1316464/138732078-f7c7ed89-7756-4cdc-8603-87a5fdd505b8.png)

Y estamos listos para levantar nuestra 1a EC2.

### 2. Levantar una EC2

Elastic Compute Cloud (EC2) de AWS son máquinas virtuales de diferentes tipos, PERO para el caso de AWS Academy, tenemos algunas restricciones.

Vamos a comenzar por abrir la AWS console dando click aquí:

![image](https://user-images.githubusercontent.com/1316464/138733006-8e00636b-23c3-4110-a3ce-4220b117c318.png)

Aquí ya vemos nuestras restricciones dentro de AWS Academy:

1. Tenemos un usuario de AWS ya creado: el usr `voclabs/userXXXXXXX=[Nombre] @ XXXX-XXXX-XXXX`. Esto no lo podemos cambiar, ni nos conviene hacerlo.
2. Podemos ver que la región por default es `N. Virginia`. Esto tampoco nos conviene cambiarlo. AWS tiene varios data centers en bunkers debajo de los cerros o la tierra para mayor seguridad, porque saben que, literal, el internet corre sobre AWS. Uno de estos bunkers está en North Virginia, y es donde AWS Academy nos dejará levantar recursos y servicios. Fuera de esa región, los servicios de AWS para Academy no están disponibles.

![image](https://user-images.githubusercontent.com/1316464/138733716-56c61d7a-b8be-44ca-a089-e381c2c8c16b.png)

Accedamos ahora a la consola de EC2 dando click aquí:

![image](https://user-images.githubusercontent.com/1316464/138733120-8cf4fc12-461e-4a8e-bc1f-efa716ab59ef.png)


Contrario a una cuenta normalita de AWS, para poder cumplir con las restricciones de AWS Academy, debemos crear la máquina virtual primero seleccionando una AMI (imagen o template) ya existente:

![image](https://user-images.githubusercontent.com/1316464/138735330-166b2960-03c3-4e12-8431-d759f34030e5.png)

Seleccionamos **Public Images**

![image](https://user-images.githubusercontent.com/1316464/138735497-4123c796-5d3d-4e73-8058-b04e74cc3738.png)

Vamos a filtrar las AMIs con los criterios: 1) que sean AMIs pertenecientes a Amazon, y 2) que se llamen "Cloud9":

https://user-images.githubusercontent.com/1316464/138736396-89262074-792d-4e1b-ad3c-5c3062338496.mp4

Vamos a ordernar la lista de AMIs por nombre y de manera descendente para poder tomar la AMI más reciente:

![image](https://user-images.githubusercontent.com/1316464/138736673-882958c3-da61-4698-98d2-ec43623f40f5.png)

Y vamos a dar click en **Launch**:

![image](https://user-images.githubusercontent.com/1316464/138736805-5aecf2ea-b502-45f2-8dd0-0269f4e66725.png)

Vamos a definir la siguiente configuración con ayuda del otro video en donde [introdujimos cloud computing](https://drive.google.com/file/d/19hobazsdMgyCyrg_9Y3Hk7c8VIVMa607/view?usp=sharing):

- Instancia tipo t2.micro
- 10GB de storage de General Purpose SSD
- **Security group:** Servicios SSH, HTTP, HTTPS. Puertos 50000, 5432, 5433, 7474, 7473, 7687, 27017

Cuando lleguemos a la parte de llaves de acceso, vamos a seleccionar la llave que viene por default en nuestra cuenta de AWS administrada por Academy:

![image](https://user-images.githubusercontent.com/1316464/138745290-ea33a8ea-c828-4265-920d-5af353c4771b.png)


Y finalmente dar click en **Launch Instances**.

Para acceder a la instancia, vamos a usar una terminal de Ubuntu. Primero necesitamos ver la ventana de detalles de AWS de regreso en nuestro Lab:

![image](https://user-images.githubusercontent.com/1316464/138747053-55c4112f-1454-474b-988d-aa414677fb45.png)


Luego dar click en **Download PEM**:

![image](https://user-images.githubusercontent.com/1316464/138747119-19f57730-cc9d-4b33-a18e-f36e3fe55d12.png)

Ya con la llave PEM descargada, tenemos que hacer lo siguiente para entrar a la instancia:

```console
$ cp /mnt/c/Users/ramos/Downloads/labsuser.pem .
$ chmod 600 labsuser.pem
$ ssh -i "labsuser.pem" ubuntu@3.86.144.108
```

La IP de la instancia la obtenemos así:


https://user-images.githubusercontent.com/1316464/138751707-edbb6931-c056-45b9-807f-3481043b392f.mp4

### Instalar Neo4j

Ejecutamos los siguientes comandos en la terminal:

```console
sudo add-apt-repository -y ppa:openjdk-r/ppa
sudo apt-get update
wget -O - https://debian.neo4j.com/neotechnology.gpg.key | sudo apt-key add -
echo 'deb https://debian.neo4j.com stable latest' | sudo tee -a /etc/apt/sources.list.d/neo4j.list
sudo add-apt-repository universe
sudo apt-get update
```

Qué estamos haciendo aquí?

1. agregando un nuevo repo a la lista de fuentes autorizadas por Ubuntu para descargar paquetes
2. actualizando para comprobar que llegamos a esa fuente nueva
3. bajando la llave privada del repo de Neo4j
4. comprobando que esta llave es válida para el nuevo repo de paquetes
5. agregando el repo "universe" donde están paquetes de versiones anteriores de utilerías de linux
6. actualizando accesos

Una vez hecho esto, vamos validar si los paquetes de Neo4j están disponibles para Ubuntu:

```console
apt list -a neo4j
```

La salida debe ser:

```console
Listing... Done
neo4j/stable 1:4.3.6 all
neo4j/stable 1:4.3.5 all
neo4j/stable 1:4.3.4 all
neo4j/stable 1:4.3.3 all
neo4j/stable 1:4.3.2 all
neo4j/stable 1:4.3.1 all
neo4j/stable 1:4.3.0 all
```

Finalmente, podemos instalar Neo4j:

```console
sudo apt-get install neo4j=1:4.3.6
```

Para arrancar el server de Neo4j debemos primero asignar un password a nuestro usuario `ubuntu`:

```console
sudo passwd

Enter new UNIX password: XXXXX
Retype new UNIX password: XXXXX
passwd: password updated successfully
```

Y luego vamos a habilitar el servicio con los siguientes comandos:

```console
sudo systemctl enable neo4j

Synchronizing state of neo4j.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable neo4j
```

Y con esto ya podemos arrancar el server:

```console
$ sudo systemctl start neo4j
```

### Conectándonos a Neo4j

Antes de conectarnos debemos modificar el archivo `/etc/neo4j/neo4j.conf` para que Neo4j acepte conexiones de todo el mundo 🌐:

```console
nano /etc/neo4j/neo4j.conf
```

Hay que buscar la siguiente línea y descomentarla (quitarle el `#`):

```console
#dbms.default_listen_address=0.0.0.0
```

Y reiniciar el server

```console
sudo systemctl restart neo4j
```

Ahora si, conectémonos a Neo4j.

Vamos a abrir un browser y entrar a `https://[IP ADDRESS]:7474`. Nos va a pedir user y password. Los de default son `neo4j`/`neo4j`, pero en cuanto los usemos, Neo4j nos va a pedir que los cambiemos.

Lo que responde en el puerto `7474` es solamente un cliente ligero web hacia Neo4j, no el Neo4j como tal.

Ese responde en el `7687`, a través de un servidorsito de conexiones llamado `Bolt`, con el URL `jdbc:neo4j:bolt://[IP ADDRESS]:7687/`.

### Cargando la versión de grafos de Northwind

Los cuates de Neo4j hicieron una versión de grafos de Neo4j. No tiene todas las tablas, pero es suficiente para contrastar los paradigas relacionales y de grafos.

![image](https://user-images.githubusercontent.com/1316464/139016561-f3307f1c-5a7f-4398-83fb-d88ba8b9399c.png)

Vamos a establecer algunas similitudes con SQL antes de continuar:

#### Similitudes con SQL

1. Un registro es un _Node_
2. El nombre de una tabla es un _Label_
3. Un `join` o `foreign key` es un _edge_ o _relationship_

En particular, al tratarse de la BD de Northwind:

4. Cada registro de la tabla `orders` en la BD de Northwind se vuelve un _Node_ con _Label_ `Order` en nuestro modelo de grafos
   - Y así sucesivamente con el resto de las tablas
5. El `join` entre `suppliers` y `products` se convierte en un _edge_ o _relationship_ llamado `SUPPLIES` (un `supplier` `SUPPLIES` N `products`), y así sucesivamente con otras tablas, salvo los siguientes casos especiales:
6. El `join` recursivo entre `employees` y `employees` se convierte en un _edge_ con el nombre `REPORTS_TO`.
7. La tabla intermedia `order_details` que soporta la relación **N:M** entre `products` y `orders` desaparece y se convierte en un _edge_ o _relationship_ llamado `CONTAINS` y con atributos `unitPrice`, `quantity`, `discount`.

Neo4j está construido casi en su totalidad en Java, por lo que ver este `camelCaseEnLosAtributos` no es extraño, como tampoco lo es que los nombres de los _Labels_ estén en mayúscula, porque su análogo en grafos son **Clases**, y sabemos que las clases en Java van con mayúscula inicial.

De esta forma, tenemos el siguiente diagrama de grafos que representa nuestra BD de Northwind:

![image](https://user-images.githubusercontent.com/1316464/139118689-133ecfc0-718d-47dd-931f-ff20e6a4c9d5.png)

#### Diferencias con SQL

1. No hay nulos! Un nulo, al ser la ausencia de algo, es simplemente la ausencia del atributo, o del _node_ o de un _edge_.
2. Dada la "Index-free Adjacency", sabemos qué _nodes_ tienen particular _relationship_ con otro _node_, en lugar de hacer un `join` y realizar la búsqueda de overlap entre 1 llave primaria de una tabla y la llave foránea de otra tabla.
3. Aunque puede haber normalización justo como en el modelo E-R, ésta no es forzosa ni rígida, y consiste principalmente en convertir _attributes_ en _nodes_, aunque al hacer esto debemos tener en mente que al convertir, los _nodes_ son instancias particulares, no clases ni _Labels_.

### Ahora si, la carga.

Para cargarla vamos a utilizar el lenguaje **Cypher**, que es como el SQL para Neo4j.

1. Products

```
LOAD CSV WITH HEADERS FROM "http://data.neo4j.com/northwind/products.csv" AS row
CREATE (n:Product)
SET n = row,
n.unitPrice = toFloat(row.unitPrice),
n.unitsInStock = toInteger(row.unitsInStock), n.unitsOnOrder = toInteger(row.unitsOnOrder),
n.reorderLevel = toInteger(row.reorderLevel), n.discontinued = (row.discontinued <> "0")
```

Qué estamos haciendo aquí?

Al igual que SQL, Cypher es un _4th generation language_, que en simples términos significa que se parece muchísimo a como se estructuran órdenes y declaraciones en inglés. 

1. `LOAD CSV`: el comando de Cypher `LOAD` es similar al `COPY` de PostgreSQL y MonetDB. La parte de `WITH HEADERS`, al igual que el `COPY` en PostgreSQL indica que el CSV a cargar tiene los nombres de las columnas en el 1er renglón. El keyword `FROM` es para indicar la fuente, y afortunadamente para nosotros, Cypher acepta URLs aquí, por lo que no es necesario descargar los CSVs a nuestro storage local y solo jalarlos del internet.
2. `CREATE (n:Product)` crea el _node_ `n` con el _label_ `Product`. Aquí vale la pena que cubramos unos aspectos de la sintaxis de Cypher:
   - Un _node_ es representado por dos paréntesis, a manera de "bolita". El _node_ `n` lo representamos como `(n)`. Por ejemplo: `create (tonyStark:SuperHero {group:'Avengers'}`
   - Un _edge_ es representado por una flechita como esta `-[:LABEL]->` y obviamente debe conectar 2 _nodes_. el `LABEL` es igualito a los labels que califican a los nodes, como sigue: `create (tonyStark)-[:MENTORS]->(peterParker)-[:WORKSFOR]->(jjJameson)`
   - Las _properties_ de un _node_ se fijan con `{}` acompañando a los _nodes_, como sigue: `CREATE (tonyStark:SuperHero {group: 'Avengers'})`
   - Igual, las _properties_ de un _edge_ se fijan con `{}`, así: `create (tonyStark)-[:MENTORS {since:2017}]->(peterParker)-[:WORKSFOR {at:'Daily Bugle'}]->(jjJameson)`
   - Las _properties_, sean de un _edge_ o de un _node_ pueden ser arrays: `(tonyStark:SuperHero {suits:['Mark IV', 'Mark V']})-[HOLDS {on:['Glove','Avengers Compound']}]->(soulStone:INFINITY_STONE)`
   - Es buena práctica primero crear los _nodes_, luego los _edges_.
3. `SET n = row` está indicando que al crear los nodos con la variable `n`, haga una equivalencia entre esa variable y el renglón del archivo CSV que estamos cargando.
4. `n.unitPrice` y demás comandos están preprocesando los datos del archivo CSV para poder guardarlos de forma correcta. `toInteger` está transformando a entero, mientras que `n.discontinued` se está evaluando a la expresión `row.discontinued <> "0"`.
   - 👀 OJO 👀: esto nos indica que durante la lectura la mayoría de los campos se están importando como strings.
   - 👀 OJO #2 👀: la comparación `<> "0"` no es válida en Java, pero si lo es en Javascript. Neo4j está hecho en Java por debajo, pero tiene un preprocessor de LISP, que es la base de Javascript, que si entiende esta expresión.

Vamos a ejecutar este import en DBeaver.

![image](https://user-images.githubusercontent.com/1316464/140169392-0fd0b895-24ad-4397-9c13-f2704b55ad57.png)

Y dónde están las tablas?!

![image](https://user-images.githubusercontent.com/1316464/140170073-f45bd98f-c876-42cf-946b-46034dc5ac33.png)

Vamos a abrir el cliente de Neo4J apuntando nuestro browser a `http://3.95.241.22:7474/`

Veremos que ahí están nuestros 25 productos:

![image](https://user-images.githubusercontent.com/1316464/140170423-1bdc05a0-3832-40ca-bb3f-63902c95603b.png)

Fijémonos en la cajita de texto de arriba:

![image](https://user-images.githubusercontent.com/1316464/140170519-3ca8f983-b189-4d52-a622-a2e1ec212a21.png)


En esta caja vamos a poder escribir queries en "Cypher". Qué está haciendo este query? `MATCH (n:Product) RETURN n LIMIT 25`.

1. `MATCH` es parecido que el `FROM`.
2. La expresión `n:Product` va a buscar los _nodes_ que tengan el _label_ `Product`.
3. `RETURN n` es como la parte del `SELECT` donde indicamos las columnas que queremos obtener; en este caso, queremos los nodos, pero bien pudieramos obtener `n.discontinuied`, o `n.reorderLevel`, es decir, atributos del (o los) nodo(s) que han hecho match.

Vamos a cargar el resto de la BD:

**Categories**

```
LOAD CSV WITH HEADERS FROM "http://data.neo4j.com/northwind/categories.csv" AS row
CREATE (n:Category)
SET n = row
```

**Suppliers**

```
LOAD CSV WITH HEADERS FROM "http://data.neo4j.com/northwind/suppliers.csv" AS row
CREATE (n:Supplier)
SET n = row
```

Una vez creados, nuestra BD se ve así:

![graph](https://user-images.githubusercontent.com/1316464/140180834-4e92aca7-f68c-4acc-8ffe-7faf551917cb.png)

Esto lo sacamos con `MATCH (n) RETURN n`, que es como un `SELECT *` pero recursivo a todas las tablas.

⚠️ Faltan las relaciones! ⚠️

Vamos a crear los _edges_ que van desde `Product` a `Category`:

```
MATCH (p:Product),(c:Category)
WHERE p.categoryID = c.categoryID
CREATE (p)-[:PART_OF]->(c)
```

Qué estamos haciendo aquì?

1. Estamos buscando todos los nodos que tengan el _label_ `Product` y los que tengan el _label_ `Category`, y los estamos poniendo en las variables `p` y `c`, respectivamente.
2. Estamos agregando una condición **que se parece muuuuuuucho** al `JOIN` de SQL. Esto es para poder ejecutar la siguiente parte:
3. Estamos creando un _edge_ con _label_ `PART_OF` entre `p` y `c`.

Ahora vamos a crear los _edges_ para relacionar `Product` y `Supplier`:

```
MATCH (p:Product),(s:Supplier)
WHERE p.supplierID = s.supplierID
CREATE (s)-[:SUPPLIES]->(p)
```

Misma estructura que el comando anterior.

Después de todo esto, cómo se ve nuestra BD?

![graph (3)](https://user-images.githubusercontent.com/1316464/140183130-b535b583-ea84-4101-b1f5-d17436ab737a.png)


Vemos que se han formado 2 "comunidades": los productos lácteos, y los no-lácteos. De esto podemos deducir que los productos lácteos tienen un grupo de suppliers **que no suministran otro tipo de productos**, mientras que los no-lácteos son suministrados por el resto de los proveedores.

Las **comunidades** son grupos de _nodes_ que están conectados por sus relaciones, pero que no están conectados a otro conjunto de _nodes_. Son importantes en el análisis de grafos para elaborar hipótesis o realizar investigaciones.

Vamos a lanzar los siguientes queries:

#### 1. Qué categorías nos vende cada proveedor?

```
MATCH (s:Supplier)-->(:Product)-->(c:Category)
RETURN s.companyName as Company, collect(distinct c.categoryName) as Categories
```

Qué estamos haciendo aquí?

1. `MATCH` - más formalmente, este comando busca un patrón dentro de nuestro grafo. En este caso, está buscando las rutas, de cualquier _label_, entre `Supplier`, `Product` y `Cateogory`.
   - La ausencia de _label_ en los _edges_ indica que no nos importa la etiqueta de la relación, solo que exista.
   - Estamos asignando los nodos de las etiquetas `Supplier` y `Category` a las variables `s` y `c`, respectivamente. Dado que para responder la pregunta, NO NOS INTERESAN los `Product`, no le estamos asignando variable, porque no nos vamos a referir a ellos, solo necesitamos sus relaciones.
2. `RETURN s.companyName as Company` es self-explanatory, no?
3. `collect(distinct c.categoryName) as Categories` es una función de agregación similar a `count()` o `avg()` en SQL. Esta función recolecta los resultados y los mete en una lista (entre `[]`).
   - Qué pasa si no ponemos el `distinct`? Y si no usamos el `collect()`?

#### 2. Qué proveedores nos venden frutas y verduras?

```
MATCH (c:Category {categoryName:"Produce"})<--(:Product)<--(s:Supplier)
RETURN DISTINCT s.companyName as ProduceSuppliers
```
Qué estamos haciendo aquí?

1. Estamos buscando los _nodes_ con _label_ `Category` cuyo _attribute_ `categoryName` sea "Produce" (noten las doble comillas), y sus relaciones con `Product` y `Supplier`, de nuevo sin importar el _label_ de dichas relaciones.
2. `RETURN DISTINCT s.companyName as ProduceSuppliers` regresa el _attribute_ `companyName` de los nodos elegidos en el `MATCH`.

### Cargar el resto de la BD:

**Customers**

```
LOAD CSV WITH HEADERS FROM "http://data.neo4j.com/northwind/customers.csv" AS row
CREATE (n:Customer)
SET n = row
```

**Orders**
```
LOAD CSV WITH HEADERS FROM "http://data.neo4j.com/northwind/orders.csv" AS row
CREATE (n:Order)
SET n = row
```

**Relación Customers -> Orders**
```
MATCH (c:Customer),(o:Order)
WHERE c.customerID = o.customerID
CREATE (c)-[:PURCHASED]->(o)
```

**Relación Order -> Product**

Esto era en PostgreSQL **la tabla intermedia `order_details`** pero en grafos NO NECESITAMOS tablas intermedias para expresar relaciones N a M!

```
LOAD CSV WITH HEADERS FROM "http://data.neo4j.com/northwind/order-details.csv" AS row
MATCH (p:Product), (o:Order)
WHERE p.productID = row.productID AND o.orderID = row.orderID
CREATE (o)-[details:ORDERS]->(p)
SET details = row,
details.quantity = toInteger(row.quantity)
```

Vamos a tener el siguiente grafo:

![graph (4)](https://user-images.githubusercontent.com/1316464/140794802-a9db64af-bb09-4324-8dc5-5ea41335fa34.png)

Podemos ver que al menos en esta versión de Northwind, tenemos bastantes clientes que nunca nos han pedido, pero **esto no es una comunidad**, porque no hay ninguna conexión entre estos. Tienen cosas en común, pero ninguna relación, por lo que no podemos decir que es una comunidad.

## Ejercicios del semestre pasado...ahora en Neo4j

### Cuál es la orden más reciente por cliente?

En SQL:
```sql
select max(o.order_date), o.order_id , o.customer_id 
from orders o
group by o.customer_id;
```

En Cypher
```
match (c:Customer)-[:PURCHASED]->(o:Order)
return c.contactName as name, max(o.orderDate) as max_ord_date
order by name
```

### De nuestros clientes, qué función desempeñan y cuántos son?

```sql
select c.contact_title , count(c.contact_title) conteo
from customers c
group by c.contact_title
order by conteo desc;
```

```
match (c:Customer)
return c.contactTitle as title, count(c.contactTitle) as title_count
order by title
```

### Cuántos productos tenemos de cada categoría?

```sql
select c.category_name, count(c.category_name) 
from categories c join products p on c.category_id =p.category_id 
group by c.category_name;
```

```
match (c:Category)<-[:PART_OF]-(p:Product)
return c.categoryName as name, count(c.categoryName) as name_count
order by name
```

### Cómo podemos generar el reporte de reorder?

```sql
select product_id, product_name, units_in_stock, reorder_level 
from products p 
where (units_in_stock<reorder_level);
```

```
match (p:Product)
where p.unitsInStock < p.reorderLevel
return p.productName as prod_name, p.unitsInStock as units_stock, p.reorderLevel as reord_level
order by units_stock
```

### A donde va nuestro envío más voluminoso?

```sql
SELECT 
	o.ship_country, 
	max(od.quantity) AS units 
FROM order_details od
JOIN orders o ON o.order_id  = od.order_id 
	GROUP BY(o.ship_country)
	ORDER BY(units) desc limit 2 -- Verificar si si es el máximo;
```

```
match (o:Order)
return o.shipCountry as ship_country, max(toFloat(o.freight)) as max_freight
order by max_freight desc
```

### Qué productos mandamos en navidad?

```sql
select p.product_name 
from products p join order_details od on p.product_id =od.product_id 
join orders o on od.order_id = o.order_id
where extract(month from o.shipped_date) = 12 and extract(day from o.shipped_date) = 25;
```

```

```

### Cuál es el promedio de flete gastado para enviar productos de un proveedor a un cliente?

```sql
select c.company_name as customer, s.company_name as shipper, avg(o.freight) as flete
from orders o join shippers s on (o.ship_via = s.shipper_id) 
join order_details od on (od.order_id = o.order_id) 
join customers c on (c.customer_id = o.customer_id)
group by c.company_name, s.company_name;
```
