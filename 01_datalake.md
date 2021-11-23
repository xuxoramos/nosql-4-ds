# Data Lake

1. La idea es aventar TODOS, TODOS, TODOS nuestros datos ahí.
2. Obviamente de manera ordenada y catalogada para saber día de la carga, tamaño de la carga, fuente, etc.
   - Esto se llama _metadata_ (i.e. _data about your data_), y es crítico para limpieza de datos y su catalogación en el lake.
3. Cuando necesitamos hacer un análisis, la metadata nos dice qué tenemos sin tener que voltear a ver los archivotes (i.e. ver el catálogo en lugar del producto).
4. El datalake es un sistema de archivos distribuído y replicado. Usualmente en una variante de HDFS (Hadoop File System)
   - Se carga un archivo
   - Se parte en bloques de 64MB
   - Se hacen 3 copias de cada bloque
   - Se reparten en un cluster de máquinas (mínimo óptimo de 4: 3 _workers_ + 1 _master_ o _main_)
   - Las máqunas no son hardware especializado, y se pueden armar con hardware _commodity_, es decir, normalito, incluso con partes de compus "de yonke"
      - El 1er datalake de Grupo Expansión y de TERAN/TBWA fue armado con _sparte parts_ del almacén.
   - De ese modo, cuando se cae 1 máquina worker, todos los archivos siguen disponibles

## Pros

1. BA.
2. RA.
3. TÍ.
4. SI.
5. MO!!!!!!!
6. Podemos incorporar máquinas de diferentes capacidades al cluster.
7. Y de esta manera podemos crecer storage y poder de cómputo por separado (aunque no tanto, aún debemos agregar una máquina completa).
8. Mencioné que es baratísimo?🤣

## Cons

1. Mucha chamba para configurar el cluster más mínimo
2. Cómputo y storage no están suficientemente separados porque aún tenemos que agregar **máquinas completas** al cluster.
3. Complejidad administrativa el crecer el cluster y agregar máquinas completas
4. Mantenimiento y troubleshooting pesadísimo
5. Todos los datos tienen la misma prioridad e importancia

## Y qué alternativa tenemos?

AWS Lake Formation. De los productos más chonchos de AWS.

⚠️Lamentablemente, este producto no está disponible en AWS Academy⚠️

Qué nos da AWS Lake Formation?

1. Secciones dedicadas para limpieza y procesamiento de datos de menor refinamiento a mayor refinamiento.
2. Secciones dedicadas para diferentes frecuencias de uso de datos, de mayor frecuencia hasta el "glaciar", donde están los datos de menor uso. La morgue, pues☠️
3. Separación total de cómputo y storage: podemos aumentar disco, o aumentar procesamiento totalmente por separado.
4. Servicio administrado: 0 configuración de archivos, 0 mantenimiento.
   - Aún hay que hacer algunas maromas con redes en AWS, pero nada que ver con la config de bajo nivel que se requiere cuando hacemos esto _on premises_, osea, local, no en la nube.

## Qué vamos a provisionar?

![Untitled Diagram drawio](https://user-images.githubusercontent.com/1316464/142990498-5140ff0f-9513-4b3e-ac36-b266d017cf98.png)

1. Un PostgreSQL en un EC2
2. Una tabla de juguete
3. Una función Lambda para insertar datos random en la tabla de juguete
4. Un evento de EventBridge para realizar una inserción cada 2 mins
5. Todo lo anterior es para simular una BD transaccionar con operatividad e I/O de numerosas veces al día
6. Crear un bucket de S3
7. Crear un Data Lake con AWS Lake Formation
8. Asociar bucket de S3 con Data Lake
9. Usar un Blueprint de AWS Glue (el servicio de ETLs de AWS) para "ingerir" data de forma incremental (copiar datos nuevos cada X tiempo desde la tabla transaccional de PostgreSQL)
   - El mismo Blueprint va a crear una base de datos de "catálogos" con _metadata_ de nuestra tabla origen
10. Copiar la data incremental al bucket de S3 en la zona/directorio `bronze`
11. Usar un 2o job de AWS Glue para leer los datos del bucket de S3, limpiarlos y guardarlos en la zona `silver`
12. Usar el mismo job de AWS Glue para leer los datos de la zona/directorio `silver`, agregarlos, formar cubos de info, y guardarlos en la zona/directorio `gold`
13. Visualizar la data en AWS Quicksight, la herramienta de visualización de AWS

Pasos para crear el Lake:

## 0. Seguridad

**0.1. Como este producto no lo tenemos con nuestro usr de AWS Academy, vamos a tener que crear un nuevo usr con su cuenta de correo.**

![image](https://user-images.githubusercontent.com/1316464/142986386-2e82e37c-7eb7-4422-9726-815aeab3033a.png)

![image](https://user-images.githubusercontent.com/1316464/142986508-63361b62-fec1-4d58-8bcd-79068baafc01.png)

**0.2. Este usuario va a ser el _root_, pero esto no es suficiente. El usr _root_ **no puede ser el administrador del data lake**. Tenemos que crear otro usuario, al cual llamaremos `datalakeadmin` y le asignaremos rol de administrador.**

![image](https://user-images.githubusercontent.com/1316464/142992693-12fb4dfd-01a5-43e7-a954-946682c2abdc.png)

![image](https://user-images.githubusercontent.com/1316464/142992768-4a9e7eab-4e15-4abd-a1b3-b784bd28c372.png)


![image](https://user-images.githubusercontent.com/1316464/142992805-faa70e5e-79a0-43fe-a87d-0d8ff786e21b.png)


![image](https://user-images.githubusercontent.com/1316464/142993314-cce09284-c0f4-4ed2-acaa-276aece822aa.png)

⚠️Obviamente en producción no podemos ser tan laxos, pero para propósitos didácticos, vamos a ser permisivos.⚠️

![image](https://user-images.githubusercontent.com/1316464/142996507-6cd13322-8a96-488b-affa-8d63ba1e186f.png)

![image](https://user-images.githubusercontent.com/1316464/142994218-cf8168c9-7304-439a-8594-f6e159b4a614.png)

**0.3. Entrar al servicio de Lake Formation**


![image](https://user-images.githubusercontent.com/1316464/142997613-d5c73b1a-e69a-42dd-bf6e-3414309c4a22.png)

Y asignar al usuario que acabamos de crear como administrador

![image](https://user-images.githubusercontent.com/1316464/142997840-263daee9-6bf8-4731-99f9-e3eaf583afe8.png)

![image](https://user-images.githubusercontent.com/1316464/142998036-6624bbb0-4468-4e97-ab91-4e083dce563a.png)


**0.4. Vamos a hacer _logout_ y vamos a volver a entrar a la consola de AWS con este usuario recién creado.**

Para volver a hacer login con este usuario no _root_ y que está asociado a nuestra cuenta, debemos de fijarnos bien en nuestro _account id_. Lo podemos ver acá:

![image](https://user-images.githubusercontent.com/1316464/142994890-3b80cfd1-f8b0-4a1e-89ff-0512b345ea35.png)

Una vez que lo tengamos guardado, entonces podemos hacer logout y entrar con la nueva cuenta:

![image](https://user-images.githubusercontent.com/1316464/142995020-715f0a4c-aaa6-4b4b-8ea4-593b27a45d0c.png)

Aquí es donde vamos a poner nuestro _account id_:

![image](https://user-images.githubusercontent.com/1316464/142995084-4376f24c-6aa5-4e5b-895d-27dec80e9023.png)

![image](https://user-images.githubusercontent.com/1316464/142995226-e67183b5-7b29-4e4e-bb19-1e5a345b65fe.png)


Posiblemente les pida cambiar el password. Cámbienlo.


Lake Formation va a preguntarles que asignen un administrador y si quieren ser uds (osea, su usuario). Obvio acepten los defaults.

![image](https://user-images.githubusercontent.com/1316464/142997096-187519cd-f00f-47e9-b507-6b6edd0eb4ca.png)

Ya estamos adentro de la consola de Lake Formation. Ahora debemos configurar más perimsos para nuestro usuario nuevo:

![image](https://user-images.githubusercontent.com/1316464/142998741-0c0c8d10-9d02-4a91-acad-31b76ccd5a47.png)






## 1. Creación de buckets de S3

![image](https://user-images.githubusercontent.com/1316464/142792467-b361f579-c778-4e9b-a4c9-20e7bfe35a34.png)

S3 (Simple Storage Service) es el servicio de almacenamiento genérico de AWS. Podemos meter lo que sea ahí.

La unidad mínima de S3 es el _bucket_.

Tenemos que crear 1 bucket con 3 áreas (como directorios):

1. uno para bronze (la data de nuestras fuentes JUSTO como es ingestada)
2. uno para silver (la data refinada, con registros individuales, pero unificada)
3. uno para golden (la data agregada, como "cubos de información")

![image](https://user-images.githubusercontent.com/1316464/142794981-e285f33f-7347-4172-b056-844d007c1365.png)

![image](https://user-images.githubusercontent.com/1316464/142795019-22c9fa22-fdb8-49fe-88f8-fee951011d12.png)

![image](https://user-images.githubusercontent.com/1316464/142795041-19e2472c-27e2-4969-87e3-31ac83c7cba3.png)

![image](https://user-images.githubusercontent.com/1316464/142795072-4fcbc9a2-74be-4d57-89b9-29ee6aa998c1.png)

![image](https://user-images.githubusercontent.com/1316464/142795158-7dfd775c-483a-4caf-a380-3c30c26aab15.png)


![image](https://user-images.githubusercontent.com/1316464/142795220-fb745158-012f-4d2e-94ba-e942cfea5dc0.png)


![image](https://user-images.githubusercontent.com/1316464/142795355-bd02d23c-343c-4351-85f1-875bbde00121.png)


Lo mismo para `silver` y `gold`.

![image](https://user-images.githubusercontent.com/1316464/142795546-3bcb9ba0-2150-410d-93f9-f416181c50cb.png)


![image](https://user-images.githubusercontent.com/1316464/142795644-858e962f-6b4a-4bdb-b485-d96f4584a644.png)


![image](https://user-images.githubusercontent.com/1316464/142795705-e69d4fec-7943-4f2e-8472-2818aefa0219.png)

![image](https://user-images.githubusercontent.com/1316464/142795793-343d8593-cada-4716-8179-1bf15f25b420.png)


![image](https://user-images.githubusercontent.com/1316464/142795884-6fe3ddaf-9aa4-4d4a-9aa5-f60e933e647c.png)


