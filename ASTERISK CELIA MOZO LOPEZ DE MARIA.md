
![](Aspose.Words.1cd81a9a-4bb2-4642-b88e-077d8f3ad278.001.png)

![Diagrama

Descripción generada automáticamente](Aspose.Words.1cd81a9a-4bb2-4642-b88e-077d8f3ad278.002.jpeg)

Configuración de un sistema de telefonía IP 

*Celia Mozo López de María* 

*Universidad de Alcalá de Henares, Escuela Politécnica Superior.* 

*Laboratorio de Redes, Sistemas y Servicios*

*Celia.mozo@edu.uah.es*
# <a name="_toc163068911"></a>**Índice**

[](#_toc163068911)

[Introducción	3](#_toc163068912)

[IP Estática	4](#_toc163068913)

[Fuentes y dependencias	5](#_toc163068914)

[Instalación de Asterisk	5](#_toc163068915)

[Instalación del español como idioma en Asterisk	6](#_toc163068916)

[Instalación de Festival, conversor de texto a voz (TTS)	7](#_toc163068917)

[Voces en español para Festival	8](#_toc163068918)

[Integración de Festival en Asterisk	8](#_toc163068919)

[Instalación de MySQL	9](#_toc163068920)

[Creación de las tablas de nuestro centro de atención telefónica	10](#_toc163068921)

[Estableciendo la comunicación entre Asterisk y sistemas externos mediante conectores ODBC.	12](#_toc163068922)

[Interconexión con otra PBX utilizando el módulo DUNDi.	17](#_toc163068923)

[Creación de claves privadas y públicas	18](#_toc163068924)

[Funcionalidades de nuestra PBX	20](#_toc163068925)

[Buzón de Voz	20](#_toc163068926)

[Llamadas a grupo.	21](#_toc163068927)

[Mensajería de texto	22](#_toc163068928)

[Música Personalizada	22](#_toc163068929)

[Redireccionamiento con horario establecido	23](#_toc163068930)

[Menú de opciones para usuarios	23](#_toc163068931)

[Videollamada	23](#_toc163068932)

[Capturar llamadas	24](#_toc163068933)

[Conclusion	25](#_toc163068934)

[Webgrafía	26](#_toc163068935)







# <a name="_toc163068912"></a>**Introducción**
El presente documento aborda el despliegue y configuración de una centralita telefónica para comunicaciones VoIP (Voz sobre Protocolo de Internet), lo que implica la transmisión de voz y video sobre el protocolo de capa 3 IP (Internet Protocol). Para este propósito, se empleará Asterisk, una implementación en lenguaje C de una PBX (Private Branch Exchange) de código abierto. Con el objetivo de optimizar el rendimiento y adaptar la instalación a las necesidades del sistema, se optará por compilar las fuentes de Asterisk de forma manual. Posteriormente, se explorarán las implicaciones de esta elección.

Una vez completada la instalación del programa, se procederá a su configuración. La mayor parte de esta configuración se llevará a cabo mediante archivos estáticos con extensión .conf, los cuales se encuentran ubicados en el directorio /etc/asterisk. Durante el proceso, se describirá detalladamente el contenido de cada archivo y se incluirán los archivos pertinentes con comentarios explicativos sobre las opciones seleccionadas. Se seguirá un orden secuencial para mantener la cohesión del documento.

Con el objetivo de ofrecer una visión completa, se incluirá un anexo que abordará los aspectos básicos del manejo de Asterisk a través de su interfaz por la línea de comandos (CLI). Además, se explicará cómo se ha preparado el sistema para facilitar el desarrollo del proyecto. Se proporcionará información sobre los softphones utilizados para realizar pruebas, con el fin de asegurar la reproducibilidad del sistema.

Todos los archivos de configuración y scripts utilizados durante la preparación de la PBX están disponibles en un repositorio público en GitHub. Se hará referencia a las rutas de los archivos a lo largo del documento utilizando enlaces directos al repositorio, aunque también se proporcionarán enlaces directos a los archivos específicos para facilitar la accesibilidad. Esto se hace considerando la extensión y complejidad de algunos archivos, lo cual podría dificultar la gestión del informe.





##


# <a name="_toc163068913"></a>**IP Estática**
Para simplificarnos configuraciones y evitarnos de errores lo mejor que podemos hacer es manejar una ip estática en un sistema Ubuntu. Una vez en nuestro Ubuntu, para no tener que estar todo el rato configurando las Ip, cosa que se me hizo bastante molesto, las vamos a poner estáticas, además así nos evitamos errores futuros en el manejo de claves.

`  `Para ello seguiremos los siguientes pasos:

\# Paso 1 Desde la consola introducimos el siguiente comando 

echo -e "[Match]\nName=ens33\n\n[Network]\nAddress=192.168.1.180/24\nGateway=192.168.1.1" | sudo tee /etc/systemd/network/10-ens33.network


\# Paso 2:  Reinicia el servicio systemd-networkd para aplicar los cambios

sudo systemctl restart systemd-networkd

\# Paso 3: Verifica la configuración de la interfaz ens33

ip address show ens33

Ahora para que nos vaya bien el DNS y no tengamos problemas:

Hacmoes igual que antes ponemos en consola 

echo -e "[Resolve]\nDNS=8.8.8.8 8.8.4.4" | sudo tee /etc/systemd/resolved.conf

\# Paso 2: Reinicia el servicio systemd-resolved para aplicar los cambios

sudo systemctl restart systemd-resolved

\# Paso 3: Verifica que la nueva configuración de DNS esté en efecto

\# Puedes hacerlo mediante una prueba de resolución de nombres, por ejemplo, con nslookup

nslookup [www.debian.org](http://www.debian.org)






# <a name="_toc163068914"></a>**Fuentes y dependencias**
Antes de la creación de nuestro maravilloso centro de atención telefónica, la cual ha sido inspirada en la propia universidad de Alcalá, vamos a descargar archivos, fuentes y dependencias para que todo funcione en orden y correctamente, por ello quiero dejar claro que primero haremos las instalaciones y después pasaremos a la configuración de los archivos necesarios. 
## <a name="_toc163068915"></a>Instalación de Asterisk
Antes de instalar el software Asterisk, debemos asegurarnos de que nuestro sistema operativo esté actualizado, para ello ejecutamos los siguientes comandos y después procedemos con la instalación de asterisk guiada en los siguientes pasos:

\# Actualiza la lista de paquetes disponibles y actualiza los paquetes instalados a sus versiones más recientes

$ sudo apt update && sudo apt upgrade

\# Instala wget para la descarga de archivos y build-essential para las herramientas de compilación, y subversion para la gestión de versiones

$ sudo apt install wget build-essential subversion

\# Cambia al directorio /usr/src/ donde se descargará y compilará Asterisk

$ cd /usr/src/

\# Descarga el paquete de Asterisk versión 15.7.1 desde el servidor de descargas de Asterisk

$ sudo wget http://downloads.asterisk.org/pub/telephony/asterisk/releases/asterisk-15.7.1.tar.gz

\# Descomprime el paquete descargado de Asterisk

$ sudo tar zxf asterisk-15.7.1.tar.gz

\# Cambia al directorio recién creado de Asterisk

$ cd asterisk-15.7.1/

\# Ejecuta el script get\_mp3\_source.sh para obtener el códec MP3 necesario para ciertas funciones de Asterisk

$ sudo contrib/scripts/get\_mp3\_source.sh

\# Ejecuta el script install\_prereq con el argumento "install" para instalar todas las dependencias necesarias para Asterisk

$ sudo contrib/scripts/install\_prereq install

\# Ejecuta el script de configuración para preparar Asterisk para la compilación

$ sudo ./configure

\# Después de ejecutar el script de configuración, se abrirá una interfaz de menú para seleccionar los módulos que se desean compilar

$ sudo make menuselect

\# Compila Asterisk. El argumento "-j2" indica que se deben utilizar 2 núcleos de procesador para la compilación, lo que acelera el proceso

$ sudo make -j2

\# Instala Asterisk en el sistema

$ sudo make install

\# Opcionalmente, instala la documentación generada automáticamente por Asterisk

$ sudo make progdocs

\# Instala los archivos de configuración de muestra que pueden ser útiles como base para la configuración personalizada

$ sudo make samples

\# Instala el script de inicio para Asterisk y actualiza la memoria caché de las bibliotecas compartidas

$ sudo make config

$ sudo ldconfig
## <a name="_toc163068916"></a>Instalación del español como idioma en Asterisk
Por defecto al instalar Asterisk, viene configurado de manera inicial con el inglés. Este idioma predeterminado puede cambiarse y lo vamos a hacer ya que, aunque nuestro centro de atención telefónica tenga un apartado que sea internacional vamos a pensar que las personas que nos llaman hablan y entienden el español. Para cambiar el idioma a español haremos lo siguientes pasos: 

\# Crear la carpeta para los sonidos en español

mkdir /var/lib/asterisk/sounds/es

\# Descargar el paquete core en español

cd /var/lib/asterisk/sounds/es

wget -O core.zip https://www.asterisksounds.org/es-es/download/asterisk-sounds-core-es-ES-sln16.zip

\# Descargar el paquete extra en español

wget -O extra.zip https://www.asterisksounds.org/es-es/download/asterisk-sounds-extra-es-ES-sln16.zip

\# Descomprimir los paquetes descargados

unzip core.zip

unzip extra.zip

\# Cambiar el propietario de los archivos al usuario y grupo de Asterisk

sudo chown -R asterisk.asterisk /var/lib/asterisk/sounds/es

\# Instalar el programa sox para la conversión de archivos de sonido

sudo apt-get install sox

\# Crear el script para la conversión de archivos de sonido

cd /var/lib/asterisk/sounds/es/

sudo vim convertir

Dentro del del archivo copiamos el siguiente contenido 

#!/bin/bash

for a in $(find . -name '\*.sln16'); do

`  `sox -t raw -e signed-integer -b 16 -c 1 -r 16k $a -t gsm -r 8k "$(echo $a | sed "s/.sln16/.gsm/")"

`  `sox -t raw -e signed-integer -b 16 -c 1 -r 16k $a -t raw -r 8k -e a-law "$(echo $a | sed "s/.sln16/.alaw/")"

`  `sox -t raw -e signed-integer -b 16 -c 1 -r 16k $a -t raw -r 8k -e mu-law "$(echo $a | sed "s/.sln16/.ulaw/")"

done

Después seguimos con los comandos

\# Dar permisos de ejecución al script

sudo chmod +x convertir

\# Ejecutar el script de conversión

sudo ./convertir
## <a name="_toc163068917"></a>Instalación de Festival, conversor de texto a voz (TTS)
Quiero que os pongáis a recordar… o si sois de mala memoria como yo podéis hacer la prueba. Cuando llamamos a algun centro como por ejemplo en la universidad de Alcalá siempre aparecen unas voces así un poco robóticas como si nos hablara la Siri o Alexa, y normalmente nos preguntan que tipo de consulta estamos haciendo, si estamos registrados o simplemente que hemos llamado fuera del horario. 

Para implementar esta función en nuestro centro de atención telefónica vamos a usar Festival, una aplicación que convierte el texto a voz. La verdad que fue difícil encontrar unas voces en español, pero gracias a la maravillosa Junta de Andalucía, podemos sin tener que pagar acceder a dichas voces. Pero como todo lo gratis tiene un contra y es que una de las voces se escucha distorsionada y no es nada clara.

Los pasos que debemos seguir son los siguientes:

\# Instalar Festival TTS

sudo apt-get install festival

\# Detener Asterisk

sudo /etc/init.d/asterisk stop

\# Modificar el archivo festival.scm

sudo vim /usr/share/festival/festival.scm

\# Agregar el siguiente contenido antes de la última línea (provide ‘festival):

\# (define (tts\_textasterisk string mode)

\#   (let ((wholeutt (utt.synth (eval (list 'Utterance 'Text string)))))

\#     (utt.wave.resample wholeutt 8000)

\#     (utt.wave.rescale wholeutt 5)

\#     (utt.send.wave.client wholeutt)))

\# Guardar y salir del editor

\# Reiniciar el sistema operativo

sudo reboot

\# Crear el servicio para Festival

sudo vim /etc/systemd/system/festival.service

\# Copiar y pegar el siguiente contenido en el archivo y guardar:

\# Description=Servicio para Festival TTS

\# Wants=network.target

\# After=syslog.target network-online.target

\# [Service]

\# Type=simple

\# ExecStart=/usr/bin/festival --server

\# Restart=on-failure

\# RestartSec=10

\# KillMode=process

\# [Install]

\# WantedBy=multi-user.target

\# Recargar los servicios

sudo systemctl daemon-reload

\# Habilitar el servicio

sudo systemctl enable festival.service

\# Reiniciar el sistema y verificar que el servicio esté activo

sudo reboot

sudo systemctl status festival.service

Ahora una vez bien configurado asterisk procederemos con la instalación de voces en español.


### <a name="_toc163068918"></a>Voces en español para Festival
Para la instalación de las voces haremos uso de un reposito de GitHub de la Junta de Andalucía, para ello seguiremos los siguientes pasos:

\# Descargar el archivo comprimido con las voces en español

wget https://github.com/franjvasquezg/festival-spanish-voices/archive/master.zip

\# Descomprimir el archivo descargado

unzip master.zip

\# Cambiar al directorio de las voces en español

cd festival-spanish-voices-master/

\# Crear el directorio para las voces en español en Festival

sudo mkdir -p /usr/share/festival/voices/spanish

\# Copiar las voces al directorio correspondiente

sudo cp -r JuntaDeAndalucia\_es\_\* /usr/share/festival/voices/spanish/

\# Verificar que los archivos se hayan copiado correctamente

ls -l /usr/share/festival/voices/spanish/

\# Modificar el archivo festival.scm para establecer la voz femenina como la voz por defecto

sudo vim /etc/festival.scm

\# Agregar la siguiente línea al final del archivo y guardar:

\# (set! voice\_default 'voice\_JuntaDeAndalucia\_es\_sf\_diphone)

\# Reiniciar el servicio de Festival

sudo systemctl restart festival.service
### <a name="_toc163068919"></a>Integración de Festival en Asterisk
Ya por último nos queda fusionar asterisk y festival para que todo funcione en orden y no tengamos problemas. Para ello seguiremos los pasos a continuación:

\# Cambiar al directorio de Asterisk

cd /usr/src/asterisk-15.7.1/

\# Ejecutar menuselect para configurar las opciones de Asterisk

Sudo make menuselect

\# En la pantalla que aparece, asegúrate de marcar Applications --> app\_festival (bajo el apartado Extended)

\# Luego guarda los cambios y sal de menuselect

\# Instalar las configuraciones seleccionadas

sudo make install

\# Iniciar Asterisk

sudo /etc/init.d/asterisk start

\# Acceder a la consola de Asterisk

sudo asterisk -rvvvvvvc

\# Verificar que la aplicación festival se haya instalado correctamente

CLI> core show application festival

## <a name="_toc163068920"></a>Instalación de MySQL
Para nuestro centro de atención telefónica he pensado que sería una buena idea y de utilidad real   tener una base de datos, ya que normalmente cuando llamas algun centro te piden una identificación como puede ser el dni, para ello he simulado gracias a mysql esto. También doy la posibilidad de que, aunque no estes registrado puedas realizar tu consulta igualmente para ello MySQL gestionara en una base de datos que modificare a para las necesidades de nuestro centro de atención telefónica.

La instalación en un sistema basado en Debian se realiza fácilmente mediante el comando sudo apt install mysql-server. Una vez completada la instalación, podemos verificar su correcto funcionamiento ejecutando systemctl status mysql. Es importante destacar que MySQL no constituye la base de datos en sí misma, sino un sistema gestor capaz de manejar múltiples bases de datos simultáneamente. En este contexto, hemos optado por crear la base de datos "asteriskdb" para su uso con la centralita, manteniendo la convención estándar. Además, hemos establecido un usuario específico, denominado también "asterisk", para permitir que Asterisk se comunique únicamente con esta base de datos. Para crear este usuario, primero debemos iniciar sesión como root en la consola de MySQL mediante el comando sudo mysql -u root -p, seguido de las órdenes correspondientes para crear tanto al usuario como la base de datos "asterisk".

\# Inicia sesión como usuario 'root' en MySQL

sudo mysql -u root -p

\# Ingresa la contraseña del usuario 'root' cuando se solicite

\# Crea el usuario 'asterisk' y la base de datos 'asterisk'

mysql> CREATE USER 'asterisk'@'localhost' IDENTIFIED BY 'asterisk';

mysql> CREATE DATABASE asteriskdb;

mysql> GRANT ALL PRIVILEGES ON asterisk.\* TO 'asterisk'@'localhost';

mysql> FLUSH PRIVILEGES;

\# Salir de la consola de MySQL

mysql> quit;

A partir de este punto, tenemos la capacidad de gestionar la base de datos ingresando al servidor de MySQL con cualquiera de los dos usuarios creados. Para acceder como el usuario "asterisk", ejecutamos el comando mysql -u asterisk -p y proporcionamos la contraseña cuando se solicite, la contraseña como hemos indicado es asterisk. Ahora, es importante reconocer que, si Asterisk puede acceder a estas bases de datos, esperará recibir la información a través de tablas bien definidas.




### <a name="_toc163068921"></a>Creación de las tablas de nuestro centro de atención telefónica
Para la creación de nuestras tablas que serán las que contengan la información de los usuarios y el personal seguiremos los siguientes pasos:

\# Acceder a la consola de MySQL

sudo mysql -u root -p

CREATE DATABASE asteriskdb;

USE asteriskdb;

CREATE TABLE personal(extension INTEGER NOT NULL, nombre VARCHAR(20) NOT NULL, equipo VARCHAR(10) NOT NULL, id INTEGER NOT NULL, PRIMARY KEY (id));

CREATE TABLE usuarios(nombre VARCHAR(20) NOT NULL, id INTEGER NOT NULL, PRIMARY KEY (id));

\# Insertar datos en la tabla personal

INSERT INTO personal VALUES (101, "CELIA", "RECEPCION", 1001);

INSERT INTO personal VALUES (102, "CRISTINA", "RECEPCION", 1002);

INSERT INTO personal VALUES (103, "ELENA", "RECEPCION", 1003);

INSERT INTO personal VALUES (110, "CARMEN", "RECEPCION", 1004);

INSERT INTO personal VALUES (201, "MARIA", "ERASMUS", 2001);

INSERT INTO personal VALUES (202, "JUAN", "ERASMUS", 2002);

INSERT INTO personal VALUES (203, "GABRIEL", "ERASMUS", 2003);

INSERT INTO personal VALUES (301, "JOSE", "FACULTADES", 3001);

INSERT INTO personal VALUES (302, "LUIS", "FACULTADES", 3002);

INSERT INTO personal VALUES (303, "ANTONIO", "FACULTADES", 3003);

INSERT INTO personal VALUES (401, "ALEX", "BECAS", 4001);

INSERT INTO personal VALUES (402, "CUCA", "BECAS", 4002);

INSERT INTO personal VALUES (403, "SOFIA", "BECAS", 4003);

\# Insertar datos en la tabla usuarios

INSERT INTO usuarios VALUES ("ALARILLA", 50);

INSERT INTO usuarios VALUES ("ALBERTO", 51);

INSERT INTO usuarios VALUES ("PAULA", 52);

INSERT INTO usuarios VALUES ("EUSTAQUIO", 53);

INSERT INTO usuarios VALUES ("ADEFESIO", 54);

INSERT INTO usuarios VALUES ("CALISTA", 55);

INSERT INTO usuarios VALUES ("MELIBEO", 56);

INSERT INTO usuarios VALUES ("LORCA", 57);

INSERT INTO usuarios VALUES ("SATURNINO", 58);

INSERT INTO usuarios VALUES ("VILLLAFRANCA", 59);

En las siguientes capturas de pantalla podemos apreciar que desde la CLI  de MySQL hemos creado un base de datos llamada asteriskdb con dos tablas que contienen la información del personal de la universidad y de los usuarios registrados en nuestra universidad.




![Texto

Descripción generada automáticamente](Aspose.Words.1cd81a9a-4bb2-4642-b88e-077d8f3ad278.003.png)

![Texto

Descripción generada automáticamente](Aspose.Words.1cd81a9a-4bb2-4642-b88e-077d8f3ad278.004.png)


## <a name="_toc163068922"></a>Estableciendo la comunicación entre Asterisk y sistemas externos mediante conectores ODBC.
Considerando la diversidad de bases de datos disponibles, es esencial crear una interfaz que unifique el acceso a todas ellas. Por esta razón, surgen los conectores ODBC, que actúan como intermediarios entre Asterisk y el sistema gestor de bases de datos correspondiente, facilitando la traducción de las solicitudes y respuestas entre ambos sistemas. Para lograr esto en sistemas UNIX, necesitamos instalar la implementación de estos conectores, junto con sus dependencias, lo cual podemos hacer ejecutando simplemente:

sudo apt update

sudo apt -y install unixodbc libltdl7 libltdl-dev

wget <https://dev.mysql.com/get/Downloads/Connector-ODBC/8.0/mysql-connector-odbc-8.0.28-linux-glibc2.12-x86-64bit.tar.gz>

tar -zxvf mysql-connector-odbc-8.0.28-linux-glibc2.12-x86-64bit.tar.gz

cd mysql-connector-odbc-8.0.28-linux-glibc2.12-x86-64bit

sudo cp bin/\* /usr/local/bin/

sudo cp -r lib/\* /usr/local/lib/

sudo myodbc-installer -a -d -n "MySQL\_ODBC" -t "Driver=/usr/local/lib/libmyodbc8a.so" sudo myodbc-installer -a -d -n "MySQL\_ODBC" -t "Driver=/usr/local/lib/libmyodbc8w.so"

Podemos verificar la instalación para asegurarnos de que se ha realizado correctamente ejecutando el siguiente comando y comprobando que aparezca el nombre que proporcionamos anteriormente, MySQL\_ODBC.

![](Aspose.Words.1cd81a9a-4bb2-4642-b88e-077d8f3ad278.005.png)

Debemos destacar que los controladores ODBC que acompañan a ciertas versiones están diseñados para procesar únicamente caracteres ASCII, lo que los hace un poco más rápidos, mientras que otros están preparados para manejar caracteres Unicode. Dado que somos españoles y es posible que se introduzcan caracteres no ASCII, hemos optado por utilizar esta última opción. Si se prefiere la primera opción, simplemente se puede cambiar la cadena "Driver=/usr/local/lib/libmyodbc8w.so" por "Driver=/usr/local/lib/libmyodbc8a.so" durante el proceso de instalación.


Ahora procedemos a informar a UnixODBC la ubicación del nuevo controlador que hemos configurado. Para ello, necesitamos modificar el archivo /etc/odbcinst.ini e incluir la siguiente información:

[MySQL ODBC 8.0]

Description = Driver ODBC ANSI para MySQL

Driver = /usr/local/lib/libmyodbc8a.so

Setup = /usr/local/lib/libmyodbc8S.so

FileUsage = 1

UsageCount=1

[MySQL ODBC 8.0 Driver]

Description = Driver ODBC Unicode para MySQL

Driver = /usr/local/lib/libmyodbc8w.so

Setup = /usr/local/lib/libmyodbc8S.so

FileUsage = 1

UsageCount=2

Simplemente configuramos UnixODBC para que utilice el controlador recién instalado. Podemos confirmar la instalación correcta ejecutando:

odbcinst -q -d

Ahora, necesitamos describir esta conexión para que Asterisk pueda utilizarla. Todo esto se puede lograr a través del archivo **/etc/odbc.ini**, donde especificamos el controlador que Asterisk debe utilizar para conectarse, la ubicación del socket de conexión, las credenciales del usuario, la base de datos a utilizar, entre otros detalles. Al final, deberíamos tener algo como:

[asterisk-connector]

Description = Conexión MySQL a la base de datos 'asteriskdb'

Driver = MySQL ODBC 8.0

Database = asteriskdb

Server = localhost

Port = 3306

Socket = /var/run/mysqld/mysqld.sock

Además, para asegurarnos de que no tengamos errores podemos hacerlo siguiendo estos pasos 

\# Accede al directorio donde descomprimiste Asterisk

cd /usr/src/asterisk-15.7.1/

\# Ejecuta el script de configuración

sudo ./configure

\# Selecciona los módulos necesarios en el menú de configuración

sudo make menuselect

\# En el menú que se muestra, selecciona los siguientes módulos:

\# Call Detail Recording: cdr\_odbc, cdr\_adaptive\_odbc

\# Dialplan Functions: func\_odbc, func\_realtime

\# PBX Modules: pbx\_realtime

\# Resource Modules: res\_config\_odbc, res\_odbc

\# Guarda los cambios y sal del menú

\# Instala los módulos seleccionados

sudo make install

\# Edita el archivo de configuración res\_odbc.conf

sudo vim /etc/asterisk/res\_odbc.conf

\# Agrega la siguiente configuración al archivo:

#[asterisk]

#enabled => yes

#dsn => asterisk-connector

#username => asterisk

#password => asterisk

#pre-connect => yes

\# Recarga el módulo res\_odbc desde la consola de Asterisk

\# Ejecuta asterisk -rvvv para acceder a la consola de Asterisk

\# Una vez dentro de la consola de Asterisk, ejecuta los siguientes comandos:

#CLI> module reload res\_odbc.so

#CLI> odbc show

e![Texto

Descripción generada automáticamente con confianza media](Aspose.Words.1cd81a9a-4bb2-4642-b88e-077d8f3ad278.006.png) ![Interfaz de usuario gráfica, Texto

Descripción generada automáticamente con confianza media](Aspose.Words.1cd81a9a-4bb2-4642-b88e-077d8f3ad278.007.png) ![Texto

Descripción generada automáticamente](Aspose.Words.1cd81a9a-4bb2-4642-b88e-077d8f3ad278.008.png) ![Texto

Descripción generada automáticamente con confianza media](Aspose.Words.1cd81a9a-4bb2-4642-b88e-077d8f3ad278.009.png)

![Captura de pantalla de computadora

Descripción generada automáticamente con confianza media](Aspose.Words.1cd81a9a-4bb2-4642-b88e-077d8f3ad278.010.png)

Finalmente, tras toda esta instalación y la verdad que dificultosa ya que muchas veces la librería libssl1.1 aparecía como inexistente en el sistema de Ubuntu y tras varios blogs y búsquedas me toco instalarla manualmente. Después de instalarla manualmente seguía sin funcionar y opte otra vez por reinstalar mi Ubuntu de mi maquina virtual. Por lo visto fue cosa de que la instalación primera no funciono como debía. Ya con las librerías y dependencias con corrección, verificamos que efectivamente asterisk este conectado a nuestra base de datos gracias a los ODBC.

` `Usamos el siguiente comando para verificar

echo "select \* from usuarios" | isql -v asterisk-connector asterisk asterisk

` `Tendría que salirnos lo siguiente:

![Texto

Descripción generada automáticamente](Aspose.Words.1cd81a9a-4bb2-4642-b88e-077d8f3ad278.011.png)







## <a name="_toc163068923"></a>Interconexión con otra PBX utilizando el módulo DUNDi.
Para nuestro call center queremos interconectar dos PBX, vamos a tener dos PBX  , la PBX180 y la PBX200 , ambas van a tener la misma configuración  en extensions.conf alfinal  y agregaremos en las dos:

;\*\*####-INI-DUNDI-####

[dundi-priv-canonical]

exten => \_XXXX,1,Dial(Zap/g1/${EXTEN},20,rtT)

[dundi-priv-customers]

[dundi-priv-via-pstn]

[dundi-priv-local]

include => dundi-priv-canonical

include => dundi-priv-customers

include => dundi-priv-via-pstn

[dundi-priv-switch]

switch => DUNDi/priv>

[dundi-priv-lookup]

include => dundi-priv-local

include => dundi-priv-switch

[macro-dundi-priv]

exten => s,1,NoOp(Macro dundi-prinv de PBX180/PBX200)

same => n,Goto(${ARG1},1)

include => dundi-priv-lookup

;\*\*####-FIN-DUNDI-####

Y bajo nuestro contexto [llamar]

;\*\*####-Conexion DUNDi-####

exten => \_99999XXXX,1,NoOP(Llamando a ${EXTEN} - ${EXTEN:5})

same => n,Macro(dundi-priv,${EXTEN:5})

Con esto llamando al 99999 conecta con la otra PBX

Por otro lado nuestro archivo sip.conf de ambas PBX tiene que tener esto:

[priv]

type=peer

context=dundi-priv-local

disallow=all

allow=ilbc



### <a name="_toc163068924"></a>Creación de claves privadas y públicas
Para esta funcionalidad usaremos astgenkey. Seguiremos los siguientes pasos:

cd /var/lib/asterisk/keys

sudo astgenkey -n PBX180

sudo scp PBX180.pub celia@192.168.1.200:/var/lib/asterisk/keys

ls -l

![Texto

Descripción generada automáticamente](Aspose.Words.1cd81a9a-4bb2-4642-b88e-077d8f3ad278.012.png)

Estos comandos cambian al directorio /var/lib/asterisk/keys, generan una clave pública para la PBX180 con astgenkey, copian la clave pública a la PBX192.168.1.200 a través de SCP y luego muestran el contenido del directorio. Asegúrate de tener los permisos necesarios y la configuración adecuada para ejecutar estos comandos.

Para mover los archivos sudo scp PBX180.pub celia@192.168.1.147:/home/celia y después mv /home/celia/PBX180.pub /var/lib/asterisk/keys

En la PBX 180 debemos tener

` `![Texto

Descripción generada automáticamente](Aspose.Words.1cd81a9a-4bb2-4642-b88e-077d8f3ad278.013.png)

Y en la PBX 200 debemos tener

` `![Texto

Descripción generada automáticamente](Aspose.Words.1cd81a9a-4bb2-4642-b88e-077d8f3ad278.014.png)

Modificamos los ficheros DUNDi en ambas terminales

En la PBX 180 

[general]

ttl=32

autokill=yes

[mappings]

priv => dundi-priv-canonical,0,SIP,192.168.1.180/${NUMBER},nopartial

priv => dundi-priv-customers,100,SIP,192.168.1.180/${NUMBER},nopartial

priv => dundi-priv-via-pstn,400,SIP,192.168.1.180/${NUMBER},nopartial

[00:0c:29:1d:99:dd] ; Dirección MAC de la PBX200

model = symmetric

host = 192.168.1.200; IP de la PBX200

inkey = PBX200 ; Clave pública de la PBX200

outkey = PBX180 ; Clave privada de la PBX180

include = priv

permit = priv

qualify = yes

order = primary

En la PBX 200

[general]

ttl=32

autokill=yes

[mappings]

priv => dundi-priv-canonical,0,SIP,192.168.1.200/${NUMBER},nopartial

priv => dundi-priv-customers,100,SIP,192.168.1.200/${NUMBER},nopartial

priv => dundi-priv-via-pstn,400,SIP,192.168.1.200/${NUMBER},nopartial

[00:0c:29:1d:99:dd] ; Dirección MAC de la PBX180

model = symmetric

host = 192.168.1.180; IP de la PBX180

inkey = PBX180 ; Clave pública de la PBX180

outkey = PBX200 ; Clave privada de la PBX200

include = priv

permit = priv

qualify = yes

order = primary

Llegado a este punto la lie un poco ya que trabajando desde mi wifi en mi casa no había problema con las direcciones ip que yo misma asigné, pero en cambio al usar la wifi de la universidad tuve un gran problema. Para solucionarlo lo que hice fue tener dos nat de maquinas virtuales que derivaran a una bridged, y además con dhclient asignar las nuevas ip automáticamente.



# <a name="_toc163068925"></a>**Funcionalidades de nuestra PBX**
## <a name="_toc163068926"></a>Buzón de Voz
El buzón de voz es una característica esencial en los sistemas de telefonía, que permite a los usuarios recibir mensajes de voz cuando no pueden atender una llamada. En el contexto de una PBX (Centralita Telefónica Privada), configurar buzones de voz es fundamental para garantizar una comunicación eficiente y efectiva. En Asterisk, el archivo voicemail.conf desempeña un papel crucial en la configuración de los buzones de voz. Este archivo define la información asociada con cada buzón, incluyendo el número de buzón, el código de acceso, el nombre del usuario y su dirección de correo electrónico. En este contexto, agregaremos buzones de voz para usuarios de prueba siguiendo el formato específico establecido en voicemail.conf. Una vez que hayamos realizado los cambios necesarios y guardado el archivo, recargaremos la configuración en Asterisk utilizando el comando voicemail relaod, asegurándonos de que los nuevos buzones de voz estén listos para su uso.

\# Abre el archivo voicemail.conf en el editor de texto Nano

$ sudo nano /etc/asterisk/voicemail.conf

\# Dentro del archivo, agrega las siguientes líneas para definir los buzones de voz 

[general]

format=wav49|gsm|wav

serveremail=asterisk

attach=yes

skipms=3000

maxsilence=10

silencethreshold=128

maxlogins=3

emaildateformat=%A, %B %d, %Y at %r

pagerdateformat=%A, %B %d, %Y at %r

sendvoicemail=yes ; Permitir al usuario componer y enviar un correo de voz mientras está dentro

[zonemessages]

eastern=America/New\_York|'vm-received' Q 'digits/at' IMp

central=America/Chicago|'vm-received' Q 'digits/at' IMp

central24=America/Chicago|'vm-received' q 'digits/at' H N 'hours'

military=Zulu|'vm-received' q 'digits/at' H N 'hours' 'phonetic/z\_p'

european=Europe/Copenhagen|'vm-received' a d b 'digits/at' HM

[default]

;; Para entrar se verificará en el dialplan y la clave

; Coincidirá con el ID de la tabla personal

101 => 1,101,101@universidadalcala.es

102 => 1,102,102@universidadalcala.es

103 => 1,103,103@universidadalcala.es

110 => 1,110,110@universidadalcala.es

201 => 1,201,201@universidadalcala.es

202 => 1,202,202@universidadalcala.es

203 => 1,203,203@universidadalcala.es

301 => 1,301,301@universidadalcala.es

302 => 1,302,302@universidadalcala.es

303 => 1,303,303@universidadalcala.es

401 => 1,401,401@universidadalcala.es

402 => 1,402,402@universidadalcala.es

403 => 1,403,403@universidadalcala.es

\# Recarga la configuración de buzones de voz en la consola de Asterisk para aplicar los cambios

$ asterisk -rx "voicemail reload"

Tendremos esto:

![Texto

Descripción generada automáticamente](Aspose.Words.1cd81a9a-4bb2-4642-b88e-077d8f3ad278.015.png)

Cuando los usuarios marcan \* y su extension, se accede a sus respectivos buzones de voz utilizando la función VoiceMailMain, donde se les solicitará un PIN configurado previamente en voicemail.conf.
## <a name="_toc163068927"></a>Llamadas a grupo.
En el ámbito de las comunicaciones empresariales, la gestión efectiva de las llamadas es esencial para garantizar la eficiencia operativa. En este contexto, la función de llamada a un grupo permite replicar una llamada entrante en varias extensiones simultáneamente, permitiendo que cualquier miembro del grupo pueda atenderla. Esto emula la dinámica de un departamento, donde una llamada a una extensión suena en todos los terminales de los empleados, siendo atendida por el primero que responda. 

Tenemos que agregar estas líneas en el archivo extensions.conf

[llamar-grupo]

;recepcion

exten => 100,1,Dial(SIP/101&SIP/102&SIP/103&SIP/110,60))

same => n,Hangup()

;direccion

exten => 200,1,Dial(SIP/201&SIP/202&SIP/203,60)))

same => n,Hangup()

;doctores

exten => 300,1,Dial(SIP/301&SIP/302&SIP/303,60)))

same => n,Hangup()

;enfermeros

exten => 400,1,Dial(SIP/401&SIP/402&SIP/403,60)))

same => n,Hangup()

;otra

exten => s,1,Festival(Grupo ${extension} no se encuentra disponible)

same => n,Hangup()

En estas extensiones llamando al 300 ,400 o 200 se llamará a los miembros de dichos departamentos, ya sea facultades, admisión y ayudas o internacional.
## <a name="_toc163068928"></a>Mensajería de texto
Esta opción la decidí agregar por diversión, ya que me parecía interesante que el usuario exterior pudiera enviar mensajes instantáneos y los agentes pudieran recibirlos. 

Para ello en extensions.conf agregue:

[mensajes]

exten => \_X.,1,Set(dest=${EXTEN})

same => n,Set(remitente=${CUT(MESSAGE(from),<,2)})

same => n,Set(remitente=${CUT(remitente,@,1)})

same => n,Set(remitente=${CUT(remitente,:,2)})

same => n,Set(texto=${MESSAGE(body)})

same => n,Set(MESSAGE(body)=${remitente}: ${texto})

same => n,Set(nombre=${ODBC\_extension(${remitente})})

same => n,Set(nombre=${IF($[ "${nombre}" = ""]?desconocido)})

same => n,MessageSend(sip:${dest}, Centro de Mensajes(CdM))

same => n,Noop(Estado del mensaje ${MESSAGE\_SEND\_STATUS})

same => n,GotoIf($["${MESSAGE\_SEND\_STATUS}" != "SUCCESS"]?fallo,s,1)

same => n,Hangup()


## <a name="_toc163068929"></a>Música Personalizada
La personalización de la música de espera en un sistema de central telefónica añade un toque distintivo y profesional a la experiencia del usuario durante las llamadas. En este contexto, se ha descargado y convertido la canción "Flashing Lights" de Kayne West a un formato compatible con Asterisk, utilizando la herramienta Sox para convertir el archivo de audio a un formato sln16. Este archivo de música personalizada se ha ubicado en la ruta adecuada dentro del sistema de archivos de Asterisk. Posteriormente, se ha modificado el archivo de configuración extensions.conf para incorporar la reproducción de esta música personalizada cuando un usuario llama al contexto 'empleado'. Esto se logra utilizando la función Playback de Asterisk, que reproduce el archivo de música después de un breve período de silencio..

\# Modificar el archivo extensions.conf para agregar la reproducción de música personalizada al contexto 'empleado'

$ sudo nano /etc/asterisk/extensions.conf

\# Agregar las siguientes líneas para reproducir la música personalizada después de un segundo de silencio

[recepción]

exten => 2,1,Set(CHANNEL(musicclass)=recepcion)

Estas configuraciones permitirán la reproducción de la música personalizada "Flashing Lights" de Kayne West cuando un usuario está en espera de que le atiendan la llamada.


## <a name="_toc163068930"></a>Redireccionamiento con horario establecido
Como he querido que la simulación de mi centralita telefónica sea lo mas posible a una real, he agregado unos parámetros de horario, que si no es cumplen reproducirán un mensaje. El horario es de 8 a 20, de lunes a viernes. Recomiendo agregarlo lo último ya que yo los findes de semana tuve el error mas tonto de no quitarlo y claro no me dejaba realizar llamadas hasta que me percaté de que yo misma lo había configurado para descansar.

exten => 918856400,1, GotoIfTime(8:00-20:00, mon-fri,\*,\*?abierto,1,1)

same => n,Festival(En estos momentos no hay ningun agente disponible)

same => n,Festival(Nuestro horario es de 8 a 20 horas de lunes a viernes)

same => n,Hangup()

## <a name="_toc163068931"></a>Menú de opciones para usuarios 
Como hemos decidió simular la universidad de Alcalá. Decidimos crear un menú IVR que priorice a los alumnos registrados, para ello hicimos una base de datos en mysql que implementada al menú IVR permite que el usuario elija entre registrado o no e introduzca sus claves y se verifique en el sistema.
## <a name="_toc163068932"></a>Videollamada
Para habilitar la videollamada es tan sencillo como en el archivo sip.conf  habilitar  videossupport=yes  y además habilitar los codec v8 y h264. En mi opinión lo que fue difícil encontrar fue la aplicación de prueba, ya que la mayoría me hacían pagar y para que nos vamos a engañas somos estudiantes… en mi caso la que elegí fue linphone y la verdad que personalmente me ha gustado más que el resto.

![Una captura de pantalla de un celular

Descripción generada automáticamente](Aspose.Words.1cd81a9a-4bb2-4642-b88e-077d8f3ad278.016.png)
## <a name="_toc163068933"></a>Capturar llamadas
En un entorno con múltiples terminales y extensiones, la función de "robo de llamadas" resulta ser una herramienta valiosa que permite a los usuarios capturar una llamada dirigida a otro terminal que aún no ha sido respondida. Esta función es especialmente útil en entornos de oficina donde la movilidad y la eficiencia son prioritarias. En el caso de Asterisk, esta funcionalidad viene preconfigurada y lista para ser utilizada. Al marcar un prefijo (\*8 seguido de la extensión llamada), cualquier usuario puede "robar" la llamada en espera desde su propio terminal, evitando así la necesidad de desplazarse físicamente para responderla. Esta capacidad facilita una comunicación más fluida y eficiente dentro de la organización, permitiendo una rápida respuesta a las llamadas entrantes.

![Interfaz de usuario gráfica, Texto

Descripción generada automáticamente](Aspose.Words.1cd81a9a-4bb2-4642-b88e-077d8f3ad278.017.jpeg)

También podemos ver el archivo desde la CLI de Asterisk

![Texto

Descripción generada automáticamente](Aspose.Words.1cd81a9a-4bb2-4642-b88e-077d8f3ad278.018.jpeg)


# <a name="_toc163068934"></a>**Conclusion**
Mi viaje con Asterisk ha sido más que una simple implementación técnica en soledad a diferencia de mis compañeros; ha sido una oportunidad para recrear la experiencia telefónica totalmente personalizada de la Universidad de Alcalá de Henares. ¿Y por qué la Universidad de Alcalá de Henares? Desde el primer día que entre hasta ahora siempre he llamado y siempre me hacía gracia, me preguntaba que como era posible que una señora con voz de maquina me hablara o como es que sabían mi dni o cosas así, pero gracias a esta práctica he descubierto un nuevo mundo. Para ello un usuario que llame al 918856400 estaría llamando a la UAH. Aquellos usuarios registrados tendrían prioridad y además el tiempo de espera seria menor ya que habría 3 agentes atendiendo dichas llamadas. Por otro lado, los miembros que no fueran pues deberían tener más paciencia. La dinámica también que aplique fue que se llamaría al usuario que más tiempo colgó una llamada de la cola de espera. Los agentes serían los encargados de redireccionar las llamadas a otros agentes o departamentos para ello cree tres: Internacional, rememorando que pude irme de erasmus gracias a la universidad, admisión y ayudas, y luego facultades. Para la interconexión de dos PBX se me ocurrió la idea de que los agentes o personal de los departamentos pudieran llamar exclusivamente a otra universidad como la UPM, que sería la otra PBX configurada.

Por otro lado, montando mi Pbx, me di cuenta al configurar los buzones de voz en el archivo voicemail.conf, que no solo estaba definiendo números y códigos de acceso, sino también creando un espacio donde los miembros de la universidad pudieran recibir mensajes de voz de manera eficiente, como si estuvieran realmente dentro del campus, listos para escuchar y responder. La función de llamada a grupos fue un punto culminante, ya que pude replicar la sensación de estar en un departamento universitario ocupado, donde las llamadas entrantes resuenan en todos los terminales, esperando ser atendidas por el primero en responder. Esta característica no solo mejoró la comunicación interna, sino que también creó un sentido de comunidad dentro de la centralita telefónica.

La inclusión de mensajería de texto instantánea agregó un toque de modernidad y conveniencia a la experiencia, permitiendo que los usuarios externos se conecten con los agentes de la universidad de una manera rápida y directa. Fue un recordatorio de que la comunicación no se limita a las llamadas de voz, sino que puede adaptarse a las preferencias y necesidades individuales de cada usuario. Personalizar la música de espera con la canción "Flashing Lights" de Kayne West fue un momento de verdadera expresión artística y diversión para que negarnos. No solo estaba seleccionando una melodía, sino que estaba creando una atmósfera única que reflejaba el espíritu y la identidad de la universidad. La incorporación de videollamadas y la capacidad de capturar llamadas en espera fueron los toques finales que transformaron mi proyecto en una experiencia completa de comunicación.

` `Al ver cómo cada característica se unía para crear un sistema cohesivo y personalizado, me sentido realmente orgullosa de haber creado algo que no solo era funcional, sino también emocionante, memorable y divertido para todos los usuarios. Sobre todo por que ahora cada vez que llame algun sitio, sabre con exactitud que lo que he realizado en miniatura las empresas lo replican en grande y con muchas más funciones y servidores. 

Finalmente, todos los archivos de este proyecto estarán en <https://github.com/Celiusky13/intro-asterisk>
# <a name="_toc163068935"></a>**Webgrafía**
Instalaciones de Asterisk

1. [Instalación de Asterisk - YouTube](https://www.youtube.com/watch?v=dyfaFu2yn78&feature=youtu.be)
1. [Instalación de Asterisk - YouTube](https://www.youtube.com/watch?v=do_4HbBXv4k&feature=youtu.be)

Funcionalidades de Asterisk PBX:

1. [Funcionalidades de Asterisk PBX - YouTube](https://www.youtube.com/watch?v=tlYm1Hjq7M0&feature=youtu.be)
1. [Funcionalidades de Asterisk PBX - YouTube](https://www.youtube.com/watch?v=nBXX5b6bepw&feature=youtu.be)
1. [Funcionalidades de Asterisk PBX - YouTube](https://www.youtube.com/watch?v=gBAx421MlJ4&feature=youtu.be)
1. [Asterisk - Distribución de llamadas - VozToVoice](https://www.voztovoice.org/?q=node/549)
1. [Tutorial de Asterisk - Pascom](https://www.pascom.net/en/blog/asterisk-tutorial-20-asterisk-call-distribution/)
1. [Convertir archivos de audio para Asterisk - Minestron](https://www.minestron.it/asterisk/convertir-archivos-de-audio-para-asterisk)
1. [Cómo instalar y configurar Festival para Asterisk - Computing for Geeks](https://computingforgeeks.com/how-to-install-and-configure-festival-for-asterisk-use/)
1. [Cambiar la voz predeterminada de Festival para Asterisk - GitHub](https://github.com/RealCabot/hci_node/wiki/Change-default-festival-voice)

Documentación y recursos adicionales:

1. [Wiki de Asterisk](https://wiki.asterisk.org/wiki/)
1. [The Asterisk Book](http://the-asterisk-book.com/)
1. [Lista de reproducción de Asterisk - YouTube](https://www.youtube.com/watch?v=jMQfSsO1da4&list=PLnzEbgyK52Gu9fdVDHburrsG3KBIntXFK)
1. [VoIP Info](https://www.voip-info.org/)
1. [DUNDi - Red de servidores Asterisk - Centralita 2018 Cos115](https://centralita2018cos115.home.blog/2018/12/09/8-dundi-red-de-servidores-asterisk/)

![Texto

Descripción generada automáticamente con confianza media](Aspose.Words.1cd81a9a-4bb2-4642-b88e-077d8f3ad278.019.jpeg)~ 29 ~

