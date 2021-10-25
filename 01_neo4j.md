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
$ sudo add-apt-repository -y ppa:openjdk-r/ppa
$ sudo apt-get update
$ wget -O - https://debian.neo4j.com/neotechnology.gpg.key | sudo apt-key add -
$ echo 'deb https://debian.neo4j.com stable latest' | sudo tee -a /etc/apt/sources.list.d/neo4j.list
$ sudo add-apt-repository universe
$ sudo apt-get update
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
$ apt list -a neo4j
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
$ sudo apt-get install neo4j=1:4.3.6
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






