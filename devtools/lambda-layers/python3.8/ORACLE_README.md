# Instrucciones para generar el Lambda Layer de Python para Oracle (con cx_Oracle)

## Prerequisitos:

* AWS CLI instalado
* Python 3.8 Instalado
* Utilidades: yum, wget, zip

## Procedimiento

* Asegurarse que la versión de Python instalada en la estación de desarrollo coincida con el Runtime que se usará en AWS (Python 3.8).  

```
    python --version
    Python 3.8.10
```

* Instalar la librería Oracle Client localmente en la VM (se selecciona una versión compacta).  

Fuente de Downloads de Oracle:  [Oracle Instant Client libraries](https://www.oracle.com/database/technologies/instant-client/linux-x86-64-downloads.html)

```
    wget https://yum.oracle.com/repo/OracleLinux/OL8/oracle/instantclient21/x86_64/getPackage/oracle-instantclient-basiclite-21.3.0.0.0-1.x86_64.rpm
    sudo yum install -y oracle-instantclient-basiclite-21.3.0.0.0-1.x86_64.rpm
```

Verificación: El directorio de instalación es:  

    /usr/lib/oracle/21/client64

Se hizo un backup del arcihvo RPM en S3, URL:

    https://ccb-binary-artifacts.s3.amazonaws.com/oracle-instantclient-basiclite-21.3.0.0.0-1.x86_64.rpm

* Instalar la librería cx_Oracle de Python

```
sudo python -m pip install cx_Oracle --upgrade
```

Verificación: Pedir las librerías instaladas de Python

    python -m pip list | grep Oracle

* Crear una carpeta "oracle_lambda_layer" y dentro de la misma crear una carpeta para Python y otra para las librerías nativas

```
mkdir oracle_lambda_layer
mkdir oracle_lambda_layer/lib
mkdir oracle_lambda_layer/python
```

* **NOTA**: para copiar los links en los pasos que vienen usar la opcion "-P" del comando "cp".

* Copiar dentro de la carpeta "oracle_lambda_layer/lib" los archivos de las librerías nativas que tiene el Oracle Client

Carpeta Origen:  

    /usr/lib/oracle/21/client64/lib     (Tener presente que en esa carpeta muchos archivos son "links")

A fin de optimizar el tamaño copiar únicamenete los siguientes 8 archivos (se deben crear los links indicados en el listado con "->")

	lrwxrwxrwx 1 ec2-user ec2-user       21 Aug 19 23:41 libclntshcore.so -> libclntshcore.so.21.1
	-rwxr-xr-x 1 ec2-user ec2-user  8094056 Aug 19 23:38 libclntshcore.so.21.1
	lrwxrwxrwx 1 ec2-user ec2-user       17 Aug 19 23:42 libclntsh.so -> libclntsh.so.21.1
	-rwxr-xr-x 1 ec2-user ec2-user 83357368 Aug 19 23:38 libclntsh.so.21.1
	-rwxr-xr-x 1 ec2-user ec2-user  7097376 Aug 19 23:38 libnnz21.so
	lrwxrwxrwx 1 ec2-user ec2-user       15 Aug 19 23:42 libocci.so -> libocci.so.21.1
	-rwxr-xr-x 1 ec2-user ec2-user  2374192 Aug 19 23:38 libocci.so.21.1
	-rwxr-xr-x 1 ec2-user ec2-user  8819280 Aug 19 23:38 libociicus.so

* Adicionalmente copiar dentro de la carpeta "oracle_lambda_layer/lib" los siguientes archivos:

Carpeta Origen:  

    /usr/lib64

    lrwxrwxrwx. 1 root root    15 Jan 27  2021 libaio.so.1 -> libaio.so.1.0.1
    -rwxr-xr-x. 1 root root 16576 Jan 27  2021 libaio.so.1.0.1

Verificación: Son en total 10 archivos dentro de "oracle_lambda_layer/lib"

* Instalar en la carpeta "oracle_lambda_layer/python" la libreria cx_Oracle:

```
python -m pip install cx_Oracle -t python
```

* Luego comprimir ambas carpetas en un archivo .zip y subirlo a s3

```
zip -r -y oracle_lambda_layer_python3_8.zip python/ lib/     (NOTA: ejecutar este comando estando dentro del folder oracle_lambda_layer)
aws s3 cp oracle_lambda_layer_python3_8.zip s3://ccb-binary-artifacts/oracle_lambda_layer_python3_8.zip --region us-east-1
```

* Crear el Lambda Layer a partir del archivo en S3

```
aws lambda publish-layer-version --layer-name oracle_lambda_layer_4_python --description 'The cx_Oracle layer with native oracle client libs for python 3.8' \
    --content S3Bucket=ccb-binary-artifacts,S3Key=oracle_lambda_layer_python3_8.zip --compatible-runtimes python3.8 --region us-east-1
```


























