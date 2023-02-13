# Despliegue de una arquitectura EFS- EC2-MultiAZ. 

### Marcos Cáceres García 


### Creación de los Grupos de Seguridad para la EFS y las EC2. 

Para ello debemos ingresar a la sección de las EC2 y en la barra lateral pulsamos para crear un nuevo grupo de seguridad. 

Una vez allí al primero de ellos le llamaremos SGweb y lo configuraremos con HTTP abriendo el puerto 80 a todas las IP: 

![](Aspose.Words.f2f5dc65-86c1-4f57-a90b-e26679aac793.001.jpeg)

Para el SG de la EFS crearemos un grupo llamado SGefs y habilitaremos nfs con el puerto 2049 con el puerto abierto a todas las IP: 

![](Aspose.Words.f2f5dc65-86c1-4f57-a90b-e26679aac793.002.jpeg)

### Creación de las EC2. 

Para la creación de las EC2 usaremos Linux con el hardware t2.micro y para el par de claves usaremos vokey. 

Ahora debemos seleccionar nuestro grupo de seguridad SGweb y cambiar la zona de disponibilidad a la 1a para la primera máquina y 1b para la segunda: 

![](Aspose.Words.f2f5dc65-86c1-4f57-a90b-e26679aac793.003.jpeg)

Y en los ajustes de usuario debemos copiar los comandos para instalar e iniciar apache: 

![](Aspose.Words.f2f5dc65-86c1-4f57-a90b-e26679aac793.004.jpeg)

### Creación de la EFS. 

Para la creación de una EFS debemos dirigirnos a la sección de servicios y podemos buscar EFS en el buscador y nos aparecerá esta página: 

![](Aspose.Words.f2f5dc65-86c1-4f57-a90b-e26679aac793.005.jpeg)

Ahora pulsamos para crear la EFS y elegimos el nombre y la zona de disponibilidad: 

![](Aspose.Words.f2f5dc65-86c1-4f57-a90b-e26679aac793.006.jpeg)

Una vez todo elegido pulsamos en crear y  nos dispondremos a configurar algunos parámetros adicionales. 

### Configuración de la EFS. 

Una vez la EFS esté operativa veremos la siguiente pestaña: 

![](Aspose.Words.f2f5dc65-86c1-4f57-a90b-e26679aac793.007.jpeg)

Una vez hecho esto pulsamos sobre ella y nos vamos en la sección de red y aquí ponemos que solo puedan acceder a la EFS las EC2 que se encuentren en el grupo de seguridad SGweb: 

![](Aspose.Words.f2f5dc65-86c1-4f57-a90b-e26679aac793.008.jpeg)

Esto impedirá que cualquier otra máquina fuera de este mismo grupo de seguridad acceda al contenido de nuestra EFS. 

### Conexión y configuración en terminal. 

Para empezar nos conectaremos a las EC2 haciendo click derecho sobre la EC2 y pulsando en conectar: 

![](Aspose.Words.f2f5dc65-86c1-4f57-a90b-e26679aac793.009.jpeg)

Una vez hecho esto ingresaremos a la terminal de las EC2 y para verificar que todo se encuentra en correcto funcionamiento escribiremos el comando que se ve en la imagen: 

![](Aspose.Words.f2f5dc65-86c1-4f57-a90b-e26679aac793.010.jpeg)

Ahora nos posicionaremos en la carpeta /var/www/html con el comando “cd /var/www/html” y después montaremos la EFS con el comando “sudo mkdir efs-mount”. 

Ahora debemos poner el comando con la id de nuestra EFS para que nuestra máquina pueda montarla sin problemas: 

![](Aspose.Words.f2f5dc65-86c1-4f57-a90b-e26679aac793.011.jpeg)

Ahora para verificar el correcto montaje ecribimos el comando “df -h”: 

![](Aspose.Words.f2f5dc65-86c1-4f57-a90b-e26679aac793.012.jpeg)

Ahora nos posicionamos en el directorio “efs-mount” con el comando “cdefs-mount”.

Ahora tenemos que descargar el archivo con el comando “wget” y la ruta del archivo, en este archivo viene el estilo de la página web el html el JavaScript,en general todo lo que necesitamos: 

![](Aspose.Words.f2f5dc65-86c1-4f57-a90b-e26679aac793.013.jpeg)

Ahora debemos descomprirmir el archvio “Netflix.zip” con el comando “unzip” y veremos  todos los archivos que contenía el archivo .zip. 

Podemos visualizar el contenido del html con el archivo que se ve en la imagen: 

![](Aspose.Words.f2f5dc65-86c1-4f57-a90b-e26679aac793.014.jpeg)

Para verificar que toda la página funcióna debemos escribir la IP pública de alguna de las dos máquinas y debemos ver la siguiente página: 

![](Aspose.Words.f2f5dc65-86c1-4f57-a90b-e26679aac793.015.jpeg)

Ahora lo que queremos es que solo con poner la ip de nuestra máquina aparezca la página, es decir, que no haga falta poner la ruta completa, para ello, ingresamos en el archivo “/etc/httpd/conf/httpd.conf” y editaremos la línea del archivo DocumentRoot "/var/www/html/efs-mount": 

![](Aspose.Words.f2f5dc65-86c1-4f57-a90b-e26679aac793.016.jpeg)

Una vez modificado y guardado el archivo nos debería dirigir a la página solamente poniendo la IP pública de cualquiera de las dos máquinas. 

### Creación del Balanceador de Carga. 

Para la creación del balanceador de carga debemos de crear una nueva máquina EC2 en este caso solo debemos configurar el par de claves. 

Una vez nos conectemos a la máquina por medio del método antes mencionado debemos instalar apache con el comando “sudo apt install apache2” y una vez instalado podremos ver la siguiente pestaña: 

Cuando hayamos verificado que nuestra máquina funciona perfectamente debemos comenzar a instalar una serie de servicios por medio de las líneas :  

a2enmod proxy a2enmod proxy\_http a2enmod proxy\_ajp a2enmod rewrite a2enmod deflate a2enmod headers a2enmod proxy\_balancer a2enmod proxy\_connect a2enmod proxy\_html 

a2enmod lbmethod\_byrequests Una vez instaladas todos los servicios debemos reiniciar apache, para ello escribimos el comando “sudo systemctl restart apache2”. 

Ahora debemos ingresar al archivo “sudo nano /etc/apache2/sites-enabled/000-default.conf” y buscaremos la línea “Virtual Host”, justo arriba escribiremos el siguiente comando sustituyendo HTTP-server por las IP de nuestras máquinas: 

![](Aspose.Words.f2f5dc65-86c1-4f57-a90b-e26679aac793.017.jpeg)

Una vez escrito, abajo debemos pegar la siguiente línea para acabar de configurar el balanceador de carga: 

ProxyPassReverse / balancer://clusterasir/ <Location /balancer-manager> 

`       `SetHandler balancer-manager 

`       `Order Deny,Allow 

`       `Allow from all 

</Location> 

Una vez guardemos y cerremos el archivo y reiniciemos la máquina, podemos verificar que todo funciona bien poniendo la IP del balanceador de carga+balancer-manager. 

Y ahora sí podemos poner la IP del balanceador de carga y se nos abrirá la página web que no esta siendo ejecutada por nuestros balanceador si no por las máquinas. 

### Instalación de php.
Para que nuestras máquinas ec2 puedan leer archivos php debemos instalarles php con los siguientes comandos:
```
$ sudo yum install -y amazon-linux-extras
$ sudo yum update
$ sudo amazon-linux-extras | grep php
$ sudo amazon-linux-extras enable php8.0
$ sudo yum clean metadata
$ sudo yum install php-cli php-pdo php-fpm php-json php-mysqlnd php-gd
```
Una vez hecho esto debemos crear en la carpeta del punto de montaje un archivo php con el nombre "prueba.php" que nos servirá para verificar que php se ha instalado correctamnte poniendo la ip de una de las máquinas y prueba.php:
![text](./PHP%208.0.27%20-%20phpinfo()%20-%20Opera%2013_02_2023%2012_49_30.png)

### Creación de los archivos php.

Para empezar vamos a /var/www/html/efs-mount y creamos los archivos de grabar.php, conexion.php y formulario.php con los siguientes códigos:
![text](Screenshot%2013_02_2023%2012_52_38.png)
![text](EC2%20Instance%20Connect%20-%20Opera%2013_02_2023%2012_52_56.png)

![alt](EC2%20Instance%20Connect%20-%20Opera%2013_02_2023%2012_54_51.png)

![alt](EC2%20Instance%20Connect%20-%20Opera%2013_02_2023%2012_55_48.png)

### Creación de la base de datos.
Ahora debemos crear la RDS que almacenará la información de las donaciónes, para ello vamos a la pestañas de RDS y la creamos con la opción de producción y con la opción de despliegue multi A-Z y le creamos un grupo de seguridad en el cual estará abierto el puerto 3306 y accesible públicamente:
![alt](Screenshot%2013_02_2023%2012_59_45.png)

Y ahora nos conectamos a nuestra base de datos con el cliente heidi sql para crear una base de datos llamada cluster y en la cual solo crearemos una tabla que se llamará donaciónes:
![alt](Screenshot%2013_02_2023%2013_02_40.png)

Una vez creada la base de datos debemos securizarla para que solo las ec2 con el SGweb puedan acceder a ella.

### Testeo de la página.
Para testear la página ponemos la IP pública de nuestro balanceador y pulsamos en el enlace de donaciones y añadimos los datos que queramos, una vez hecho esto y pulsando en enviar debemos ver esto:
![alt](Unnamed_cluster_%20-%20HeidiSQL%2012.3.0.6589%2013_02_2023%2013_05_22.png)
