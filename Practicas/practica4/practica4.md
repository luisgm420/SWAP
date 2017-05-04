**Práctica 4**
==============

1°. Instalar un certificado SSL autofirmado para configurar el acceso HTTPS a los servidores.

Para generar un certificado SSL autofirmado en Ubuntu Server activamos el módulo SSL de Apache, generar los certificados y especificarle la ruta a los certificados en la configuración. Para ello introducimos los siguientes comandos:
```shell
sudo a2enmod ssl
sudo service apache2 restart
sudo mkdir /etc/apache2/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.crt
```
Ahora nos pedirá una serie de datos para configurar el dominio, introducimos los datos y modificamos el archivo de configuración con el siguiente comando:
```shell
sudo nano /etc/apache2/sites--available/default--ssl.conf
```
E insertamos las siguientes lineas:
```shell
SSLCertificateFile /etc/apache2/ssl/apache.crt 
SSLCertificateKeyFile /etc/apache2/ssl/apache.key
```
Por último Activamos el sitio default--ssl y reiniciamos apache.

2°. Configurar las reglas del cortafuegos con IPTABLES para asegurar el acceso alos servidores web.

Para configurar el cortafuegos utilizamos el siguiente script:
```script
iptables -F
iptables -X
iptables -Z
iptables -t nat -F

iptables -P INPUT DROP
iptables -P OUTPUT DROP

iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

iptables -A INPUT -i eth0 -p tcp -m multiport --dports 22,80,443 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -o eth0 -p tcp -m multiport --sports 22,80,443 -m state --state ESTABLISHED -j ACCEPT
```
Por último comprobamos que todo funciona correctamente mediante la orden curl:

<img src="https://github.com/luisgm420/SWAP/blob/master/Practicas/practica4/funcionamiento_cortafuegos.png">*Captura funcionamiento cortafuegos*
