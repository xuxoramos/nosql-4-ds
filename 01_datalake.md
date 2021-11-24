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
   - zona `bronze` para _raw data_, osea, la que se ingiere desde nuestras fuentes transaccionales
   - zona `silver` para data procesada y limpiada
   - zona `gold` para agregados y sumarizados para presentar directo a herramientas de BI
3. Secciones dedicadas para diferentes frecuencias de uso de datos, de mayor frecuencia hasta el "glaciar", donde están los datos de menor uso. La morgue, pues☠️
5. Separación total de cómputo y storage: podemos aumentar disco, o aumentar procesamiento totalmente por separado.
6. Servicio administrado: 0 configuración de archivos, 0 mantenimiento.
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

Vamos a hacer lo mismo con los siguientes permisos:

3. AWSGlueConsoleFullAccess
4. CloudWatchLogsReadOnlyAccess
5. AmazonAthenaFullAccess

Y vamos a agregar 2 _in-line policies_:

![image](https://user-images.githubusercontent.com/1316464/143195589-9a6b0a4a-93d6-48f6-b3bf-81a89c0afeb8.png)

![image](https://user-images.githubusercontent.com/1316464/143195646-aa738a5b-7958-45f5-b88b-8639b23c9d75.png)

Y pegar los siguientes policies, **por separado**, sin olvidar de reemplazar `<account-id>` por el número que sacamos en el paso **0.4** más abajo.

6. In-line policy `LakeFormationSLR`

```javascript
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "iam:CreateServiceLinkedRole",
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "iam:AWSServiceName": "lakeformation.amazonaws.com"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:PutRolePolicy"
            ],
            "Resource": "arn:aws:iam::<>account-id:role/aws-service-role/lakeformation.amazonaws.com/AWSServiceRoleForLakeFormationDataAccess"
        }
    ]
}
```

7. In-line policy `UserPassRole`

```javascript
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PassRolePermissions",
            "Effect": "Allow",
            "Action": [
                "iam:PassRole"
            ],
            "Resource": [
                "arn:aws:iam::<account-id>:role/LakeFormationWorkflowRole"
            ]
        }
    ]
}
```


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



![image](https://user-images.githubusercontent.com/1316464/142999421-649f32f8-9f26-478d-8732-50dc02cbfe9d.png)

Ahora vamos a crear un bucket de S3 que servirá como data lake.

## 1. Creación de buckets de S3

S3 (Simple Storage Service) es el servicio de almacenamiento genérico de AWS. Podemos meter lo que sea ahí.

La unidad mínima de S3 es el _bucket_.

Tenemos que crear 1 bucket con 3 áreas (como directorios):

1. uno para bronze (la data de nuestras fuentes JUSTO como es ingestada)
2. uno para silver (la data refinada, con registros individuales, pero unificada)
3. uno para golden (la data agregada, como "cubos de información")

![image](https://user-images.githubusercontent.com/1316464/142792467-b361f579-c778-4e9b-a4c9-20e7bfe35a34.png)

**1.1. Crear un bucket**

![image](https://user-images.githubusercontent.com/1316464/142794981-e285f33f-7347-4172-b056-844d007c1365.png)

⚠️**OJO! El nombre del bucket debe ser único A LO LARGO DE AWS**⚠️

En cuanto a la región, puede ser donde uds quieran.

![image](https://user-images.githubusercontent.com/1316464/142795019-22c9fa22-fdb8-49fe-88f8-fee951011d12.png)

![image](https://user-images.githubusercontent.com/1316464/142795041-19e2472c-27e2-4969-87e3-31ac83c7cba3.png)

![image](https://user-images.githubusercontent.com/1316464/142795072-4fcbc9a2-74be-4d57-89b9-29ee6aa998c1.png)

![image](https://user-images.githubusercontent.com/1316464/142795158-7dfd775c-483a-4caf-a380-3c30c26aab15.png)

**1.2. Crear folders en el bucket para cada zona `bronze`, `silver` y `gold`**

![image](https://user-images.githubusercontent.com/1316464/142795220-fb745158-012f-4d2e-94ba-e942cfea5dc0.png)


![image](https://user-images.githubusercontent.com/1316464/142795355-bd02d23c-343c-4351-85f1-875bbde00121.png)


Lo mismo para `silver` y `gold`.

Con esto terminamos hasta este momento con S3. Vamos ahora a simular una BD transaccional.

## 2. Simulación de una BD transaccional

Para simular estos movimientos transaccionales, vamos a crear una tabla y disparar un evento cada X tiempo para que se inserte 1 nuevo registro en ella.

Vamos a configurar los siguientes componentes en AWS:

1. Una instancia de EC2 con una _elastic ip_
2. Un PostgreSQL dentro de esa instancia de EC2, de acuerdo [a esta guía](https://computingforgeeks.com/how-to-install-postgresql-13-on-ubuntu/)
3. Crear una tabla de juguete que representará nuestros sistemas transaccionales
4. Crear una _lambda function_ que insertará en dicha tabla de juguete
5. Configurar un evento en AWS EventBridge para que se dispare cada 2 min y llame a la _lambda function_ de arriba

**2.1. Tabla de juguete**

```sql
CREATE TABLE random_data (
	id serial4 NOT NULL,
	valor text NULL,
	fecha timestamp NULL,
	CONSTRAINT random_data_pkey PRIMARY KEY (id)
);
```

**2.2. Lambda Function**

Pueden descargar el código fuente de la _lambda_ de mi repo: https://github.com/xuxoramos/lambda-transactionaldb-insert

![image](https://user-images.githubusercontent.com/1316464/141932380-2b746db0-22b1-4d90-ba28-570e29813ac7.png)

![image](https://user-images.githubusercontent.com/1316464/141932505-c4b30d99-a92d-4053-9a0b-2150b856bc2e.png)

![image](https://user-images.githubusercontent.com/1316464/141932815-60230a59-33d7-418b-8b39-ef5ecb7b2086.png)

![image](https://user-images.githubusercontent.com/1316464/141932954-3044a36a-0b33-4cb3-9adb-e84b2025836b.png)

![image](https://user-images.githubusercontent.com/1316464/141933032-2c4b18ea-442c-4812-a9d1-3babe757530f.png)

Al descargar este repo, deben:

1. modificar el archivo `/db.ini` y capturar sus credenciales de su instalación de PostgreSQL
2. zipear el contenido del repo
3. subirlo a la creación de la función lambda como se muestra acá abajo

![image](https://user-images.githubusercontent.com/1316464/141933120-cc4285e6-54d2-4d16-8964-a843d5df8218.png)


**2.3. Evento en EventBridge**

![image](https://user-images.githubusercontent.com/1316464/141931577-098846ed-7985-4ca7-98e4-f9528ee951a2.png)

![image](https://user-images.githubusercontent.com/1316464/141931836-4f27aec2-3d13-4de3-9eb6-e2c27b7d79de.png)

![image](https://user-images.githubusercontent.com/1316464/141932107-56a319ec-1aeb-4345-83b0-9e84ae96abc7.png)

![image](https://user-images.githubusercontent.com/1316464/141932169-ba8e3b14-0dd4-41d5-92f3-124bf823e32d.png)

![image](https://user-images.githubusercontent.com/1316464/141932249-683a1ca5-0122-4243-8ea8-06369c286b01.png)

**2.4. Verificar**

Chequen su tabla, debe haber un registro cada 2 mins:

![image](https://user-images.githubusercontent.com/1316464/141933580-853e72de-88be-4648-8fa3-c1087df18d54.png)


Antes de regresar a Lake Formation, debemos crear un _endpoint_ de nuestra VPC de AWS.

## 3. 2a parte de Seguridad y Permisos

**3.1. VPC Endpoint**

Cada vez que nosotros creamos recursos en AWS, se crea una _Virtual Private Cloud_, que es una estructura de networking dentr de la cual cae todo lo que creamos.

Sin embargo, hay recursos de AWS, sobre todo los servicios **administrados** (i.e. los que son plataformas más que infraestructura, como RDS en lugar de EC2 + PostgreSQL o DocumentDB en lugar de EC2 + MongoDB) que no tienen VPC.

AWS Lake Formation es un servicio adminsitrado, y va a ingerir datos desde un PostgreSQL en una EC2. EC2 está en una VPC, por lo que para tener acceso a ella, debemos de crear un _VPC Endpoint_ para que Lake Formation pueda a través de él llegar a nuestro PostgreSQL en la EC2.

![image](https://user-images.githubusercontent.com/1316464/143184752-fc24c4c5-0b0e-44b5-8158-7bd4ec1d3348.png)

![image](https://user-images.githubusercontent.com/1316464/143184980-507d5ea7-c26d-4fee-94eb-ebeaae12ed28.png)

![image](https://user-images.githubusercontent.com/1316464/143185041-94b20011-e9f6-406f-a588-917476a1b85a.png)

![image](https://user-images.githubusercontent.com/1316464/143185194-0c05bbff-309e-49ae-82fc-865c6d1bd441.png)

![image](https://user-images.githubusercontent.com/1316464/143185256-f966a998-093d-4c69-9ea8-b521790ce354.png)

![image](https://user-images.githubusercontent.com/1316464/143185309-1813338a-70d1-4a21-9f81-7d13eeadbe85.png)

![image](https://user-images.githubusercontent.com/1316464/143185374-f3c8167b-842a-49de-b62f-502660f3062e.png)

Y listo!

**3.2. Crear un _Service Role_ para ejecutar tareas de ETL**

En AWS, como en todas las nubes, todo, **absolutamente todo** se corre con un usuario o rol asignado.

En AWS hay _Service Roles_ que son un conjunto de permisos con los que se ejeuta un servicio.

`LakeFormationWorkflowRole` es el rol con el que se van a corren los procesos de **Glue**, que es la herramienta de ETLs de AWS y con la cual vamos a "ingerir" datos desde nuestro PostgreSQL en EC2 para copiarlos a nuestro Data Lake.

Necesitamos darle permiso a este rol nuevo de que acceda sin restricciones a S3, y que haga las veces de administrador.

⚠️OJO! Esto obviamente en un setting productivo no es recomendable, pero para propósitos educativos lo vamos a hacer.⚠️

Para hacer esto necesitamos crear un rol:

![image](https://user-images.githubusercontent.com/1316464/143185856-41c25c11-5074-4489-91c9-6a37d1295a88.png)

![image](https://user-images.githubusercontent.com/1316464/143190812-8438fa08-335a-4e45-a671-735608e11d05.png)


![image](https://user-images.githubusercontent.com/1316464/143190861-b8500edc-92ce-4280-ba5b-cc5f3dbd3490.png)


![image](https://user-images.githubusercontent.com/1316464/143190959-7ebee910-ec3f-4f44-afe5-7bea1a68d36d.png)


![image](https://user-images.githubusercontent.com/1316464/143191011-beb08f8d-4bf1-4c0a-96e0-ed6261b238bc.png)


![image](https://user-images.githubusercontent.com/1316464/143191210-52c33b12-5ffa-48a9-b3c8-6e1717d82c16.png)

![image](https://user-images.githubusercontent.com/1316464/143191280-f0ddf17b-b846-45d3-a89b-bfe863734d21.png)


![image](https://user-images.githubusercontent.com/1316464/143191342-08028295-8571-476d-9903-dbf36a6d0b24.png)


![image](https://user-images.githubusercontent.com/1316464/143191529-e749adfd-7220-4554-be8b-84f4301d5dde.png)

Finalmente, agregamos el siguiente _inline policy_:

![image](https://user-images.githubusercontent.com/1316464/143195589-9a6b0a4a-93d6-48f6-b3bf-81a89c0afeb8.png)

![image](https://user-images.githubusercontent.com/1316464/143195646-aa738a5b-7958-45f5-b88b-8639b23c9d75.png)


Y pegar lo siguiente.

⚠️OJO! reemplazar la parte de `<account_id>` por el _account id_ que obtuvieron en el paso **0.4**

```javascript
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                 "lakeformation:GetDataAccess",
                 "lakeformation:GrantPermissions"
             ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": ["iam:PassRole"],
            "Resource": [
                "arn:aws:iam::<account-id>:role/LakeFormationWorkflowRole"
            ]
        }
    ]
}
```


Listo, ahora si regresamos a LakeFormation!

## 4. Usar "Blueprint" de ingesta de BD incremental

⚠️OJO! Fíjense que siempre estén usando la región donde estamos agregando toda la infra. En este caso es `us-east-2`, u "Ohio".⚠️

**4.1. Asignarle a Lake Formation el bucket de S3 que creamos en paso 1)**

![image](https://user-images.githubusercontent.com/1316464/142795546-3bcb9ba0-2150-410d-93f9-f416181c50cb.png)


![image](https://user-images.githubusercontent.com/1316464/142795644-858e962f-6b4a-4bdb-b485-d96f4584a644.png)


![image](https://user-images.githubusercontent.com/1316464/142795705-e69d4fec-7943-4f2e-8472-2818aefa0219.png)

![image](https://user-images.githubusercontent.com/1316464/142795793-343d8593-cada-4716-8179-1bf15f25b420.png)


![image](https://user-images.githubusercontent.com/1316464/142795884-6fe3ddaf-9aa4-4d4a-9aa5-f60e933e647c.png)


**4.2. Crear una BD dentro del Lake**

![image](https://user-images.githubusercontent.com/1316464/143192868-52b5d46a-679f-4969-8b15-8c1cc1fc8280.png)

![image](https://user-images.githubusercontent.com/1316464/143203954-3b966ba3-4c2c-4a32-a3ab-3a2d18bf6dc9.png)

Tenemos ahora una BD vacía.

Con qué la vamos a llenar?

Vamos a dejar que Lake Formation vaya a nuestra BD transaccional de juguete en PostgreSQL, la examine, saque sus datos y sus metadatos, y los meta al data lake:

## 5. Importar data de PostgreSQL al Data Lake

**5.1. Vamos a crear una conexión a nuestra BD en PostgreSQL con AWS Glue**

Recordemos que AWS Glue es la herramienta de AWS para hacer ETLs, es decir, los procesos que pasan datos de un lugar a otro.

![image](https://user-images.githubusercontent.com/1316464/143209472-4f8f1289-e1b0-4c2a-8f0a-3ed197f72cce.png)

![image](https://user-images.githubusercontent.com/1316464/143209584-07abb78d-7546-4ae9-b9fa-5a44462f3078.png)

![image](https://user-images.githubusercontent.com/1316464/143209709-18255765-fb85-4721-8209-0eb77d60c961.png)

En la siguiente pantalla vamos a capturar la info de nuestra BD en PostgreSQL.

La VPC y la subnet las obtenemos de la máquina EC2 donde vive nuestro PostgreSQL.

⚠️OJO! El URL de nuestra BD debe contener la IP address interna de la EC2, no la _Elastic IP_ que le asignamos!⚠️

Esto se debe a que la _Elastic IP_ es visible hacia el mundo exterior, PERO el AWS Glue y demás servicios **ESTÁN EN EL INTERIOR** de la red de AWS! Por lo que Glue **NUNCA** va a alcanzar esa IP pública, por lo que tiene que utilizar la interna!

![image](https://user-images.githubusercontent.com/1316464/143212791-15a02855-2b47-4578-a9c7-7cf5a80fb1d2.png)

Y de dónde sacamos la IP interna? De la consola de EC2.

![image](https://user-images.githubusercontent.com/1316464/143212972-35daa315-7ca3-4ef3-b0cd-9fe259c5bb21.png)

Después de dar click en el botón de _Finish_, podemos probar la conexión:

![image](https://user-images.githubusercontent.com/1316464/143213292-5ce7385a-f8f8-4db8-95b2-ab9578f562c0.png)

**5.2. Vamos ahora a utilizar el blueprint**

![image](https://user-images.githubusercontent.com/1316464/143209173-9510d8d8-2cbe-4082-80a1-ae391b63c42a.png)

Va la explicación:

1. Vamos a elegir "Incremental Database" porque, dado que es una "tabla viva" (aunque con datos de juguete), necesitaremos que los datos se copien periódicamente para "mantenerlos frescos".
2. La fuente de importación va a ser la conexión que acabamos de crear en la consola de Glue
3. La ruta va a ser `database/schema/table`, en nuestro caso `transactionaldb/public/random_data`

![image](https://user-images.githubusercontent.com/1316464/143213662-9d75e0d1-f748-47f8-8dfb-a3d5f96dca0e.png)

Continúa la explicación:

1. En la sección de _Incremental Data_ debemos poner los detalles de la tabla que le ayudará a Glue a saber qué renglones ya se copiaron y qué renglones aún no.
   - _Table name_ es la tabla que contiene el campo que nos va a ayudar a distinguir un renglón de otro
   - _Bookmark Keys_ es el campo de esa tabla que nos va a decir si un registro ya fue copiado o no.
2. En la sección de "Import Target", en la parte de _Target Database_ vamos a asignar la BD dentro del lake que creamos en el paso 4.2
3. El _storage location se va a llenar solo.
4. _Data format_ tenemos la opción de CSV o Parquet. Parquet es la mejor opción para fuentes de datos con millones de registros. Es un formato columnar de archivo **similar** a como los crea MonetDB.

![image](https://user-images.githubusercontent.com/1316464/143214357-7270ec3f-de52-4493-9fa2-f479bcbfb77d.png)

Finalmente, con estos datos terminamos de definir nuestro workflow

1. Definimos una _cron expression_ para ejecutar el workflow cada 59 mins mediante una expresión **CRON**
2. Nombramos el workflow como deseemos
3. Las tablas de catálogo se les agrega el prefijo `_catalog`

![image](https://user-images.githubusercontent.com/1316464/143216890-b298ce93-778f-4e5b-a66e-bc2551520eac.png)

Ya que está creado el workflow, AWS nos preguntará si queremos arrancarlo. Digámos que si.

![image](https://user-images.githubusercontent.com/1316464/143218079-da36a6e8-9152-40ee-a3a7-b20ecab370cd.png)

**5.3. Validando ejecución del workflow**

Cómo podemos ver si está corriendo o qué caramba está haciendo?

Debemos ir a AWS Glue y examinar el "grafo de ejecución" del ETL que acabamos de hacer:

![image](https://user-images.githubusercontent.com/1316464/143218365-cba58012-be16-4ae0-9d0f-b04aaaf76cc8.png)

Dejamos corriendo el workflow toda la noche y a la mañana siguiente nos encontramos con esto:

![image](https://user-images.githubusercontent.com/1316464/143276483-2d38d3b3-7c9f-4940-8f2b-5a57d0c9fcc9.png)

Vamos a inspeccionar las ejecuciones del workflow:

![image](https://user-images.githubusercontent.com/1316464/143276846-fdb2f53a-44c8-4f53-81af-f5c3bcb33355.png)

Examinemos la última ejecución:

![image](https://user-images.githubusercontent.com/1316464/143276982-cd30aefc-a68a-4ac9-8800-516a98d92a0a.png)

![image](https://user-images.githubusercontent.com/1316464/143277230-494783d9-b0d6-43e6-be8e-4346f73a9a66.png)

Vemos que la fase de _crawling_ tuvo un error.

La fase de _crawling_ es en donde Lake Formation inspecciona nuestro PostgreSQL, la tabla, los registros, y las estructuras para en automático generar los catálogos de metadatos y preparar la extracción de datos de SQL a archivos Parquet como lo especificamos en el paso 5.2.

Si hacemos click en esa fase del grafo de ejecución, podemos ver el error:

![image](https://user-images.githubusercontent.com/1316464/143277988-29be2fc9-3c9a-4e62-8cda-c5110ffc5a75.png)

Parece que 2 ejecuciones se comenzaron a "pisar las agujetas".

Esto sucede cuando el espacio entre ejecuciones del workflow no es suficiente para dejar terminar a uno cuando ya está comenzando otro.

Pero esto significa que **la ejecución anterior debió haber terminado**, no? Veamos.

![image](https://user-images.githubusercontent.com/1316464/143278482-544a49d6-87af-48b0-a4c9-12773206090d.png)

![image](https://user-images.githubusercontent.com/1316464/143278570-347002f8-9ef3-48a8-9691-efd1b353a8bb.png)

🥳

Entonces seguramente de todas nuestras ejecuciones, tenemos una que si terminó, y otra que no, y así sucesivamente. Esto se debe a que no dejamos tiempo suficiente entre ejecuciones.

Antes de arreglarlo, vamos a ver el resultado en la zona `bronze` de nuestro data lake en S3:

![image](https://user-images.githubusercontent.com/1316464/143279617-b21def0b-373a-4043-b6f1-d339527e5b9b.png)

Vemos aquí 2 tablas creadas por Lake Formation:

1. una con _Classification_ en `postgresql`, que no es mas que la **descripción** de nuestra tabla dentro de Lake Formation, como lo podemos ver cuando le damos click:

![image](https://user-images.githubusercontent.com/1316464/143280382-7ef53f6d-b292-401a-8e4f-b439364106d9.png)

2. otra con _Classification_ en `s3://lakeformation-nosql4ds/bronze/ingest/catalog__transactionaldb_public_random_data/`, que es la materialización de la tabla de PostgreSQL en archivos Parquet.

![image](https://user-images.githubusercontent.com/1316464/143281415-2feba4d8-3b30-40fc-b5d6-a47c6719c0ea.png)

Demos click en la liga _Location_ para ir a S3 :

![image](https://user-images.githubusercontent.com/1316464/143286399-8b6c0e07-031c-4599-99cd-fa59aaf4b479.png)

Como estamos ejecutando un workflow de un _blueprint_ incremental, esto significa que la 1a ejecución del workflow nos traería toda la BD hasta ese punto, y ejecuciones subsecuentes nos traerían al datalake solamente los incrementos o _deltas_, es decir, los registros creados o presentes desde la última ejecución hasta la siguiente.

Los archivos **parquet** son columnares binarios, por lo que no serviría de mucho descargar uno y explorar su interior. Más bien debemos de explorar esta data con otra herramienta, pero antes, vamos a arreglar la frecuencia de ejecución de nuestro workflow.

**5.4. Corrigiendo la frecuencia de ejecución del workflow**

![image](https://user-images.githubusercontent.com/1316464/143288005-1c0e2f64-e371-438e-a344-315cecc975fe.png)

![image](https://user-images.githubusercontent.com/1316464/143288129-28ca967f-864d-41d2-b04e-5460eaab0cc1.png)

Dar click en el botón `Next` hasta llegar a esta pantalla y seleccionar ejecución CADA HORA y comenzar en el minuto 00:

![image](https://user-images.githubusercontent.com/1316464/143288394-bdeeb1a9-361f-46ce-ac1e-d0f5cdb3a01f.png)

Dar click en `Next` hasta llegar al botón `Finish` y terminar.

Con esto ya hemos corregido las ejecuciones para que sean cada hora, y no cada 59 mins.

Vamos ahora a examinar la data en nuestro data lake:

## 6. Explorando el data lake con AWS Athena

Athena es un servicio de AWS que nos permite correrle queries tipo SQL a archivos que estén guardados en S3.

Vamos a ir al home de este servicio:

![image](https://user-images.githubusercontent.com/1316464/143298129-925220d4-2c55-48a8-8c9b-395ffd5a53cc.png)

Vemos que el costo es de $5 USD por terabyte. Este es uno de los servicios más caros de AWS, y con justa razón, imagínense aventarle lo que sea al S3 y poderle tirar queries con SQL normalito! Esta funcionalidad es poderosa.



## 7. Explorando la data con Quicksight

Primero accedamos a Quicksight.

![image](https://user-images.githubusercontent.com/1316464/143289209-ab934e88-300b-4be9-b55b-e458c2c5eaf9.png)


Y pidamos acceso al servicio.

![image](https://user-images.githubusercontent.com/1316464/143289288-3c290bd4-479e-4ec3-99b8-2c875f9c9ade.png)

Muy vivillos, los de AWS nos quieren enjaretar una suscripción Enterprise, y no conformes con eso, están subrepresentando el botón para las suscripciones estándar usando un viejo truco de UX. No vamos a caer en su trampa y vamos a darle click en _Standard_ arriba a la derecha:

![image](https://user-images.githubusercontent.com/1316464/143289733-5f7cf1df-d1a6-4c31-ba3c-06161d79ef67.png)

Vamos ahora a configurar Quicksight con las siguientes opciones.

![image](https://user-images.githubusercontent.com/1316464/143290079-f9bd1719-429e-4535-a06f-8953a7ed80c2.png)

Más abajo debemos configurar a qué tendrá acceso Quicksight. Lo más importante es que tenga acceso al bucket de S3 donde tenemos nuestro Data Lake.

![image](https://user-images.githubusercontent.com/1316464/143290302-607e9548-db7f-4eda-b6e7-47e53ef790c1.png)


![image](https://user-images.githubusercontent.com/1316464/143290391-ae1df989-f3e5-47e0-a0ce-e5ee6385a9fe.png)

Una vez que nuestra cuenta de Qucksight esté lista, y nos brinquemos el tutorial, 













