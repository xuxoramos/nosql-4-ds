# Bases de Datos de Grafos - Neo4j

Mientras que para las BDs relacionales, cada registro es un ejemplo o instancia de una entidad, o una tabla, y las relaciones son solo llaves copiadas entre tablas, para las BDs de grafos lo que se almacena es la conexión o la relación entre 2 nodos.

Encima de esto, dado que cada nodo puede tener diferentes atributos, las bases de datos de grafos **no tienen esquema**, lo cual las vuelve un poco lentas para la escritura VS las BDs relacionales, sobre todo cuando llegamos al orden de millones de registros, debido a que cada nodo tendrá estructura diferente y no tenemos una estructura definida y fija como una tabla.

## El modelo de datos de una BD de grafos

Los componentes principales de una BD de grafos es:

1. **Nodos:** los objetos principales en un diseño de grafos. Representan una entidad particular (una instancia de un objeto, o una instancia de un sustantivo - i.e. no representan un _tipo de objeto_ o _clase_, sino el _objeto en sí_, no un tipo de _marca_ropa_deportiva_ sino _adidas_, o _Patagonia_).
2. **Vértices:** la conexión entre 2 nodos. Representa una relación (un verbo) entre 2 objetos.
3. **Labels/Etiquetas:** podemos usarlas para definir una _clase_ o _tipo de objeto_ y así agrupar nodos con características comunes.
4. **Propiedades:** dentro de cada _nodo_ o _vértice_ podemos tener un diccionario _llave:valor_ con propiedades que califican a cada uno.

### Modelo lógico de grafos

Supongamos la siguiente narrativa:

> _"Si, mira, quiero una red social mamalona, como facebook, pero que llamaremos "el libro de caras del bienestar" ya sabes? que nuestros datos no estén en servidores gringos neoliberales, sino en infraestructura 100% mexicana. Primero tendremos solamente **Personas** y **Lugares**, y cada **Persona** tendrá su **nombre**, **lugar**, **género** y **correo electrónico**; mientras que cada **Lugar** tendrá **nombre**, **latitud** y **longitud**. Las **Personas** estarán conectadas por una relación llamada **Amistad**, donde podemos tener **años de amistad**, y mientras que los **Lugares** estarán conectados a las **Personas** a través de la relación llamada **Vive en**, sin atributos, de momento."_

Cómo podemos representar esto?

1. **Nodos**
<details><summary>Ustedes primero</summary>

  ![image](https://user-images.githubusercontent.com/1316464/138636550-a2a764fb-9aaf-426a-9971-eb0b6bf0ca84.png)

</details>

2. **Vértices**
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

Aparte de tener un nodo por cada _Person_, la diferencia más grande es al recorrer relaciones o **vértices**.

La forma de registrar que 2 nodos están relacionados con un vértice en bases de datos de grafos es utilizando **apuntadores**, esto es, direcciones de memoria que nos llevan de un lugar dentro de ella donde está un nodo, a otro lugar en la memoria donde está otro nodo.

Esto se le llama ["Index-free Adjacency"](https://thomasvilhena.com/2019/08/index-free-adjacency). Esto es, no necesitamos un diccionario, ni una operación de intersección de conjuntos, ni un mapeo de columnas, como en los modelos relacionales, para poder ir de una tabla a otra.

En las BDs de grafos, las relaciones se encuentran ya físicamente en memoria, expresadas con el objeto de más bajo nivel que nuestra máquina puede utilizar. Esto implica que el recorrer un grafo para ir de nodo en nodo recolectando información, el performance **NO DEPENDE DEL TAMAÑO DEL GRAFO**! Podemos decir que tiene un running time linear de **O(n)**, donde N es el num de vértices a recorrer, que siempre será más reducido que la N involucrada en un `JOIN`.

En contraste, las BDs relacionales al viajar de una tabla a otra con `JOIN`, estamos utilizando una operación de intersección para ver en qué parte los 2 índices de las 2 llaves de la relación se traslapan, y por tanto su performance **DISMINUYE A MEDIDA QUE HAY MÁS REGISTROS**. Podemos decir que los `JOIN` tienen un running time de **O(log n)** cuando usamos llaves indexadas, y **O(n^2)** cuando son _soft joins_ entre columnas que no son llaves o tienen índices.

## Cuándo SÍ debemos usar una BD de grafos?

1. Cuando mis datos estén altamente conectados

Esto es, cuando el elemento central para nuestro análisis sea la conexión o la relación entre entidades particulares, y por tanto nuestros datos **no sean transaccionales**, entonces probablemente nos conviene una BD de grafos. Frecuentemente solo es necesario guardar los datos y realizar análisis sofisticado después.

2. Cuando la lectura sea más importante que la escritura

Si mi problema es _transaccional_ en su naturaleza, y los analíticos que voy a ejecutar en estos datos con ayuda de `JOIN` no recorren la mayoría de las entidades, entonces quizá no requiera una BD de grafos.

3. Cuando mi modelo de datos cambie constantemente

Dado que las BDs de grafos **no tienen esquema**, al igual que las [Document Databases](https://github.com/xuxoramos/nosql-4-ds/blob/gh-pages/01_mongodb.md), y por tanto para cada nodo o vértice podemos agregar atributos a como deseemos, serán adecuadas cuando tengamos alto nivel de incertidumbre sobre la definición de nuestros datos, y a la postre nos permitirán que no todos los nodos tengan forzosamente valores en todos los atributos, y los que tengan, que no sean consistentes en cuanto a los tipos (i.e. el nodo "Adam Smith" tendrá el atributo _tiene\_sentido\_del\_humor_ en **FALSE**, mientras que el nodo "Milton Friedmann" lo tendrá en **TRUE**, y finalmente, el nodo "Karl Marx" lo tendrá en **null**).

## Cuándo NO debemos usar una BD de grafos?

1. Cuando mis analíticos hagan _table scan_ constantemente

Cuando los analíticos que vaya a correr sobre esos datos impliquen constantes _table scans_ sean parciales o _full_, o secuenciales o con índices, entonces una BD de grafos puede que no sea la mejor opción.

2. Cuando mis búsquedas por ID sean constantes

Como vimos en [nuestra intro a BDs columnares](https://github.com/xuxoramos/nosql-4-ds/blob/gh-pages/01_monetdb.md), los queries propios de una solución transaccional siempre obtienen todo el renglón, no se centran en relaciones, y frecuentemente buscarán un ID en toda la tabla. Esto es porque este tipo de queries no aprovecha el performance que dan las BDs de grafos para recorrer varios nodos.

3. Cuando debamos almacenar atributos de gran tamaño

Por ejemplo, si para un nodo hipotético "AMLO" debemos poner un atributo _mañanera_, y ahí debemos de colocar TODAS esas conferencias, resultará en un atributo de varios cientos de gigas. El precio de este almacenamiento es alto comparado con la capacidad de movernos y viajar a lo largo de nodos para recoger información.

## Instalando MonetDB en AWS EC2
