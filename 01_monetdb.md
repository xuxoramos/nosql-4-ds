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


