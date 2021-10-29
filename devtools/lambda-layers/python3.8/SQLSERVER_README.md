# Instrucciones para generar el Lambda Layer de Python para SQL Server (con pyodbc)

## Prerequisitos:

* AWS CLI instalado
* Python 3.8 Instalado
* Python Development Instalado (se instaló para este caso python3-devel)
* Utilidades: yum, wget, zip
* Openssl instalado
* [Compiladores de GNU y su Toolset](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/compile-software.html)

## Procedimiento

* Bajar el [código fuente de unixODBC](http://www.unixodbc.org/download.html)

```
curl ftp://ftp.unixodbc.org/pub/unixODBC/unixODBC-2.3.7.tar.gz -O
tar xzvf unixODBC-2.3.7.tar.gz
cd unixODBC-2.3.7
```

* Se respaldan los fuentes en S3:

	https://ccb-binary-artifacts.s3.amazonaws.com/unixODBC-2.3.7.tar.gz

* Configurar la compilación, compilarlo e instalarlo

```
./configure --sysconfdir=/opt --disable-gui --disable-drivers --enable-iconv --with-iconv-char-enc=UTF8 --with-iconv-ucode-enc=UTF16LE --prefix=/opt
make
make install
cp include/*.h /usr/include/
```

* Bajar e instalar el [driver ODBC de MSSQL Server](https://docs.microsoft.com/en-us/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server?view=sql-server-2017)

```
curl https://packages.microsoft.com/config/rhel/6/prod.repo > /etc/yum.repos.d/mssql-release.repo
sudo ACCEPT_EULA=Y yum install -y msodbcsql17
```

* Agregar al archivo ~/.bashrc las siguientes variables:

```
export CFLAGS="-I/opt/include"
export LDFLAGS="-L/opt/lib"
```

* Volver a cargar la configuración del ambiente:

	`source ~/.bashrc`

* Crear una carpeta "sqlserver_lambda_layer" y dentro de la misma crear una carpeta para los drivers de microsoft, otra para Python y otra para las librerías nativas

```
mkdir sqlserver_lambda_layer
mkdir sqlserver_lambda_layer/lib
mkdir sqlserver_lambda_layer/python
mkdir sqlserver_lambda_layer/msodbcsql17
```

* Copiar los archivos de configuración de ODBC:

Carpeta Origen: /opt

```
cd sqlserver_lambda_layer
cp /opt/*.ini .
```

* Configurar los archivos .ini, para que tenga la ubicación, el nombre y versión de la librería:

1. **odbcinst.ini**  

```
[ODBC Driver 17 for SQL Server]
Description=Microsoft ODBC Driver 17 for SQL Server
Driver=/opt/msodbcsql17/lib64/libmsodbcsql-17.7.so.2.1
UsageCount=1
```

2. **odbc.ini**

```
[ODBC Driver 17 for SQL Server]
Driver = ODBC Driver 17 for SQL Server
Description = My ODBC Driver 17 for SQL Server
Trace = No
```

* Copiar las librerías ODBC nativas de Linux:

```
cd sqlserver_lambda_layer
cp -P /usr/lib64/libodbc*.* .
``` 

* Copiar el driver ODBC de MS

```
cd sqlserver_lambda_layer
cp -Pr /opt/microsoft/msodbcsql17 .
```

* Instalar pyodbc en la carpeta 

```
cd sqlserver_lambda_layer
python -m pip install pyodbc -t python
```

* Empaquetar y subir a S3:

```
cd sqlserver_lambda_layer
zip -r -y mssql_lambda_layer_python3_8.zip python/ msodbcsql17/ lib/ *.ini
aws s3 cp mssql_lambda_layer_python3_8.zip s3://ccb-binary-artifacts/mssql_lambda_layer_python3_8.zip --region us-east-1
```

* Publicar la capa

```
aws lambda publish-layer-version --layer-name mssql_python_3_8 \
    --description 'The pyodbc Python library to access SQL Server from Python' \
    --content S3Bucket=ccb-binary-artifacts,S3Key=mssql_lambda_layer_python3_9.zip \
    --compatible-runtimes python3.8 --region us-east-1
```

