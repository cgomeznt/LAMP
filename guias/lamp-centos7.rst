Cómo instalar LAMP en CentOS 7
===================================


Instalar LAMP en CentOS 7, cómo montar un servidor Linux con Apache, MariaDB (en lugar de MySQL) y PHP

Configurar los repositorios EPEL
+++++++++++++++++++++++++++++++++++

Debido a que los repositorios oficiales de CentOS sólo ofrecen la versión 5.4.16 de PHP, una versión ya obsoleta e insegura, habilitaremos el soporte EPEL para disponer de paquetes más actualizados.

Abrimos un terminal y añadimos las herramientas necesarias al sistema::

	$ sudo yum -y install epel-release yum-utils

Configurar los repositorios para PHP
+++++++++++++++++++++++++++++++++++++++

Ahora podemos añadir el repositorio donde encontraremos las versiones actualizadas de PHP::

	$ sudo yum install -y http://rpms.remirepo.net/enterprise/remi-release-7.rpm

En el repositorio existen varias versiones de PHP, así que podemos habilitar inicialmente la versión estable que más nos interese, por ejemplo para PHP 7.4::

	$ sudo yum-config-manager --enable remi-php74

Si prefieres alguna de las versiones estbles más recientes, puedes habilitar PHP 8.0 u 8.1 (que pueden presentar problemas de compatibilidad con bastantes aplicaciones)::

	$ sudo yum-config-manager --enable remi-php81

Pero también podrías configurar versiones anteriores de PHP, por ejemplo, si necesitases PHP 7.3::

	$ sudo yum-config-manager --enable remi-php73

Configurar los repositorios para MariaDB
+++++++++++++++++++++++++++++++++++++++

La versión de MariaDB incluida en los repositorios oficiales de CentOS 7 es muy antigua (MariaDB 5.5).

Si te interesa, puedes añadir el repositorio para la última versión estable, MariaDB 10.6, o tal vez MariaDB 10.5. Para ello crearemos un nuevo archivo de repositorio, por ejemplo para la versión 10.6 (si te interesa otra versión, sustituye a continuación 10.6 por 10.x, según corresponda)::

	$ sudo nano /etc/yum.repos.d/mariadb-10.6.repo

Y añadimos el siguiente contenido::

	[mariadb]
	name = MariaDB
	baseurl = http://yum.mariadb.org/10.6/centos7-amd64
	gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
	gpgcheck=1

Guardamos los cambios y cerramos el archivo.

Actualización de los repositorios
Únicamente queda actualizar la información de los repositorios::

	$ sudo yum update -y

Ahora nuestro sistema CentOS 7 está listo para empezar con la instalación y configuración del sistema LAMP.


Cómo instalar un servidor LAMP en CentOS 7
++++++++++++++++++++++++++++++++++++++++++++

Para instalar todo el software LAMP los paquetes que necesitaremos son los siguientes::

	httpd
	mariadb-server
	php
	php-mysqlnd

Además, automáticamente se instalarán todas las dependencias correspondientes.

Abrimos una consola y lanzamos yum para realizar las de descarga e instalación de los paquetes::

	$ sudo yum -y install httpd mariadb-server php php-mysqlnd

En este momento ya está instalado todo el software necesario, pero obviamente habrá que hacer ajustes para poder trabajar.


Arranque de los servicios
+++++++++++++++++++++++++

Los servicios web y de base de datos no arrancan por defecto tras la instalación. Tampoco arrancan cada vez que se inicia el sistema. En un sistema LAMP lo habitual es que los servicios estén disponibles constantemente, así que vamos a realizar la configuración pertinente.

En primer lugar habilitamos los servicios, para que arranquen automáticamente en cada inicio del sistema::


	$ sudo systemctl enable httpd mariadb

	$ sudo systemctl start httpd mariadb


Ajustes del firewall
++++++++++++++++++++++++


Añadimos una excepción para el servicio HTTP::

	# firewall-cmd --get-active-zones
	# firewall-cmd --zone=public --list-all

	$ sudo firewall-cmd --permanent --zone=public --add-service=http
	$ sudo firewall-cmd --permanent --zone=public --add-service=https
	$ sudo firewall-cmd --permanent --zone=public --add-port=3306/tcp

Y finalmente recargamos la configuración del firewall para que hagan efecto los cambios::

	$ sudo firewall-cmd --reload


Cómo configurar LAMP en CentOS 7
++++++++++++++++++++++++++++++++++

Aunque tu nuevo LAMP server CentOS 7 ya está funcionando, vamos a realizar unos mínimos ajustes en la configuración de los servicios que integran la pila LAMP en CentOS .

De este modo dejaremos el sistema en el estado óptimo para empezar a trabajar.

Apache
++++++++

El archivo de configuración es /etc/httpd/conf/httpd.conf, archivo que modificaremos para darle un nombre al servidor. Por defecto el servidor no tiene nombre, por lo que podría registrar un aviso en cada inicio del servicio si no tienes correctamente configurado el nombre de tu máquina CentOS 7.

Editamos el archivo de configuración con nuestro editor de texto favorito y privilegios de sudo::

	$ sudo nano /etc/httpd/conf/httpd.conf

Hay que buscar la directiva ServerName, que está desactivada mediante comentario por defecto::

	...
	# ServerName gives the name and port that the server uses to identify itself.
	# This can often be determined automatically, but we recommend you specify
	# it explicitly to prevent problems during startup.
	#
	# If your host doesn't have a registered DNS name, enter its IP address here.
	#
	#ServerName www.example.com:80
	...
	Borramos el carácter # al inicio de la línea y asignamos un valor (normalmente la dirección IP, nombre DNS, dominio, etc. del servidor CentOS 7):

	...
	ServerName centos7.local.lan:80
	...

Podemos poner el nombre que queramos o necesitemos. Para que los cambios tomen efecto, hay que recargar la configuración del servidor web::

	$ sudo systemctl reload httpd

La carpeta de archivos web se encuentra configurada por defecto en /var/www/html/.

Servicio de base de datos
++++++++++++++++++++++++++++

Es importante ejecutar el script mysql_secure_installation para hacer más segura la instalación de Mariadb, cuyos valores por defecto no son aconsejables para montar un servidor en producción.::

	$ sudo mariadb-secure-installation

Con este script conseguiremos:

Crear una contraseña para el usuario root de MariaDB. La primera pregunta del script es la contraseña de root que, por defecto, viene en blanco.
Eliminar los usuarios anónimos.
Desactivar el acceso remoto para el usuario root de MariaDB.
Eliminar la base de datos de pruebas.
Ya está listo el servicio de bases de datos para trabajar con él. Tienes más información sobre creación de usuarios y acceso remoto en la entrada sobre la instalación de Mariadb en CentOS 7.


PHP
+++++++++++++

La configuración de PHP se realiza a través de los ajustes del archivo /etc/php.ini. Lo básico a modificar en una nueva instalación sería::

	Zona horaria del servidor
	Tratamiento de errores

Para obtener el valor que necesitas para ajustar la zona horaria, puedes consultar en http://php.net/manual/es/timezones.php.

En cuanto a los valores para el tratamiento de errores de PHP, en el propio archivo /etc/php.ini vienen como ejemplo los valores de desarrollo y de producción.

Por ejemplo, editamos php.ini::

	$ sudo nano /etc/php.ini

Para un servidor de desarrollo situado en España, podríamos establecer estos valores en /etc/php.ini::

	...
	[Date]
	; Defines the default timezone used by the date functions
	; http://php.net/date.timezone
	date.timezone = Europe/Madrid
	...
	error_reporting = E_ALL
	...
	display_errors = On
	...
	display_startup_errors = On
	...

Si necesitas un servidor de producción (que oculte los mensajes de error) no necesitas cambiar los valores por defecto.

En el caso de que en otro momento necesites hacer cambios, los valores de producción y desarrollo se detallan en los comentarios intercalados en el propio archivo de configuración, junto a cada directiva.

Tras estos mínimos cambios, podemos guardar y cerrar php.ini.

No olvides recargar la configuración del servidor web tras cada cambio en la configuración de PHP::

	$ sudo systemctl reload httpd

Tienes mayor información sobre estas configuraciones, añadir y configurar módulos, etc. en la entrada sobre la instalación de PHP en CentOS.

Probar la pila LAMP en CentOS 7
++++++++++++++++++++++++++++++++++++++

Para probar la pila LAMP en CentOS 7 crearemos un pequeño script en PHP accesible vía web::

	$ sudo nano /var/www/html/info.php

El contenido de este archivo será únicamente la siguiente línea:

<?php phpinfo();
Guardamos los cambios y cerramos el archivo.

Ahora accedemos desde el navegador, añadiendo la ruta /info.php a la dirección IP o dominio del servidor CentOS 7 en el que hemos alojado la pila LAMP:
