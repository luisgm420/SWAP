**Práctica 5**
==============

1°. Crear una BD con al menos una tabla y algunos datos.

Para crear una base de datos introduciremos los siguientes comandos:
```shell
mysql -u root -p
```
```mysql
mysql> create database contactos;
mysql> use contactos;
mysql> show tables;
mysql> create table datos(nombre varchar(100),tlf int);
mysql> show tables;
```
Ahora introduciremos los datos en la tabla:
```mysql
mysql> insert into datos(nombre,tlf) values ("Carlos",958271883);
mysql> select * from datos;
```

2°. Realizar la copia de seguridad de la BD completa usando mysqldump.

Antes de hacer la copia de seguridad en el archivo .SQL debemos evitar que se acceda a la BD para cambiar nada.
```shell
mysql -u root -p
```
```mysql
mysql> FLUSH TABLES WITH READ LOCK; 
mysql> quit
```
Ahora ya sí podemos hacer el mysqldump para guardar los datos.
```shell
mysqldump contactos -u root -p > /root/contactos.sql
```
Ahora desbloqueamos las tablas:
```shell
mysql -u root -p
```
```mysql
mysql> UNLOCK TABLES; 
mysql> quit
```

3°. Restaurar dicha copia en la segunda máquina (clonado manual de la BD).

Ya podemos ir a la máquina esclavo (maquina2, secundaria) para copiar el archivo .SQL con todos los datos salvados desde la máquina principal (maquina1):
```shell
scp root@192.168.98.128:/root/contactos.sql /root/
```
Con el archivo de copia de seguridad en el esclavo ya podemos importar la BD completa en el MySQL. Para ello, en un primer paso creamos la BD:
```shell
mysql -u root -p
```
```mysql
mysql> CREATE DATABASE ‘contactos’; 
mysql> quit
```
Y en un segundo paso restauramos los datos contenidos en la BD:
```shell
mysql -u root -p ejemplodb < /root/contactos.sql
```

4°. Realizar la configuración maestro-esclavo de los servidores MySQL para que la replicación de datos se realice automáticamente.

Lo primero que debemos hacer es la configuración de mysql del maestro. Para ello editamos, como root, el /etc/mysql/my.cnf. Una vez aqui, comentamos:
```shell
#bind-address 127.0.0.1
```
Y descomentamos:
```shell
log_error = /var/log/mysql/error.log
server-id = 1
log_bin = /var/log/mysql/bin.log
```
Una vez realizados dichos cambios, guardamos el documento y reiniciamos el servicio con la siguiente instrucción:
```shell
/etc/init.d/mysql restart
```
A continuación introducimos los siguientes comandos en mysql para configurar la máquina "MAESTRO":
```mysql
mysql> CREATE USER esclavo IDENTIFIED BY 'esclavo';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'esclavo'@'%' IDENTIFIED BY 'esclavo';
mysql> FLUSH PRIVILEGES;
mysql> FLUSH TABLES;
mysql> FLUSH TABLES WITH READ LOCK;
```
Por último introducimos los siguientes comandos en mysql para configurar la máquina "ESCLAVO":
```mysql
mysql> CHANGE MASTER TO MASTER_HOST='192.168.98.128', MASTER_USER='esclavo', MASTER_PASSWORD='esclavo', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=980, MASTER_PORT=3306;
mysql> START SLAVE;
```
