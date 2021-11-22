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

## Y qué alternativa tenemos?


