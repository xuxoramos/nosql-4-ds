# Data Lake

1. La idea es aventar TODOS, TODOS, TODOS nuestros datos ahí.
2. Obviamente de manera ordenada y catalogada para saber día de la carga, tamaño de la carga, fuente, etc.
   - Esto se llama _metadata_ (i.e. _data about your data_), y es crítico para limpieza de datos y su catalogación en el lake.
3. Cuando necesitamos hacer un análisis, la metadata nos dice qué tenemos sin tener que voltear a ver los archivotes (i.e. ver el catálogo en lugar del producto).
4. El datalake es un sistema de archivos distribuído y replicado. Usualmente en una variante de HDFS (Hadoop File System)
   - Se carga un archivo
   - Se parte en bloques de 64MB
   - Se hacen 3 copias de esos cada bloque
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
8. Mencioné que es baratísimo?

## Cons

1. Mucha chamba para configurar el cluster más mínimo
2. Cómputo y storage no están suficientemente separados porque aún tenemos que agregar **máquinas completas** al cluster.
3. Complejo crecer el cluster y agregar máquinas completas
4. Mantenimiento y troubleshooting pesadísimo
5. Todos los datos tienen la misma prioridad e importancia

## Y qué alternativa tenemos?

AWS Lake Formation. De los productos más chonchos de AWS.

Qué nos da AWS Lake Formation?

1. Secciones dedicadas para limpieza y procesamiento de datos de menor refinamiento a mayor refinamiento.
2. Secciones dedicadas para diferentes frecuencias de uso de datos, de mayor frecuencia hasta el "glaciar", donde están los datos de menor uso.
3. Separación total de cómputo y storage: podemos aumentar disco, o aumentar procesamiento totalmente por separado.
4. Servicio administrado: 0 configuración de archivos, 0 mantenimiento.

Pasos para crear el Lake:

## 0. Seguridad

Lake Formation va a preguntarles que asignen un administrador y si quieren ser uds (osea, su usuario). Obvio acepten los defaults.

![image](https://user-images.githubusercontent.com/1316464/142796934-34b6cf90-73a6-4e3c-b989-f043bc8c54e8.png)



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


