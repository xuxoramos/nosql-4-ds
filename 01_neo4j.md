# Bases de Datos de Grafos - Neo4j

Mientras que para las BDs relacionales, cada registro es un ejemplo o instancia de una entidad, o una tabla, y las relaciones son solo llaves copiadas entre tablas, para las BDs de grafos lo que se almacena es la conexión o la relación entre 2 nodos.

Encima de esto, dado que cada nodo puede tener diferentes atributos, las bases de datos de grafos **no tienen esquema**, lo cual las vuelve un poco lentas para la escritura VS las BDs relacionales, sobre todo cuando llegamos al orden de millones de registros, debido a que cada nodo tendrá estructura diferente y no tenemos una estructura definida y fija como una tabla.

## El modelo de datos de una BD de grafos

Los componentes principales de una BD de grafos es:

1. **Nodos:** los objetos principales en un diseño de grafos. Representan una entidad particular (una instancia de un objeto, o una instancia de un sustantivo - i.e. no representan un _tipo de objeto_ o _clase_, sino el _objeto en sí_, no un tipo de _marca_ropa_deportiva_ sino _adidas_, o _Patagonia_).
2. **Vértices:** la conexión entre 2 nodos. Representa una relación (un verbo) entre 2 objetos.
3. **Labels/Etiquetas:** podemos usarlas para definir una _clase_ o _tipo de objeto_ y así agrupar nodos con características comunes.
4. **Propiedades:** dentro de cada _nodo_ o _vértice_ podemos tener un diccionario _llave:valor_ con propiedades que califican a cada uno.

### Ejemplo: redes sociales

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
