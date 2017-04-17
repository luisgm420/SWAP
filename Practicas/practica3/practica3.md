**Práctica 3**
==============

En primer lugar creamos una nueva máquina virtual, la cual configuraremos mas tarde como un balanceador de carga *nginx*. Para ello una vez tenemos creada nuestra máquina virtual ponemos lo siguiente para instalar *nginx*.
```shell
cd /tmp/
wget http://nginx.org/keys/nginx_signing.key
apt-key add /tmp/nginx_signing.key
rm -f /tmp/nginx_signing.key

echo "deb http://nginx.org/packages/ubuntu/ lucid nginx" >> /etc/apt/sources.list 
echo "deb-src http://nginx.org/packages/ubuntu/ lucid nginx" >> /etc/apt/sources.list

apt-get update 
apt-get install nginx
```
Una vez instalado *nginx* cambiamos el archivo de configuración para unsar un algoritmo *round-robin* con la misma prioridad para todos los servidores.
```shell
upstream apaches {
  server 192.168.98.128; 
  server 192.168.98.129;
}

server{
  listen 80;
  server_name balanceador;
  access_log /var/log/nginx/balanceador.access.log; error_log /var/log/nginx/balanceador.error.log; 
  root /var/www/;
  location / 
  {
    proxy_pass http://apaches;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
    proxy_http_version 1.1;
    proxy_set_header Connection ""; 
  }
}

service nginx restart
```
Por último comprobamos que todo funciona correctamente mediante la orden curl:

<img src="https://github.com/luisgm420/SWAP/blob/master/Practicas/practica3/Capturas%20de%20pantalla/nginx.png">*Captura funcionamiento nginx*

En segundo lugar creamos otra máquina virtual, la cual configuraremos mas tarde como un balanceador de carga *haproxy*. Para ello una vez tenemos creada nuestra máquina virtual ponemos lo siguiente para instalar *haproxy*.
```shell
apt-get install haproxy

cd /etc/
cd haproxy/ ifconfig
nano haproxy.cfg

global
  daemon
  maxconn 256 
defaults
  mode http
  timeout connection 4000
  timeout client 42000
  timeout server 43000
frontend http-in 
  bind *:80
  default_backend servers
backend servers
  server m1 192.168.98.128:80 maxconn 32 
  server m2 192.168.98.129:80 maxconn 32
```
Por último comprobamos que todo funciona correctamente mediante la orden curl:

<img src="https://github.com/luisgm420/SWAP/blob/master/Practicas/practica3/Capturas%20de%20pantalla/haproxy.png">*Captura funcionamiento haproxy*

Ahora realizaremos un *Benchmark* con otra máquina virtual a parte a nginx y a haproxy. Nos hemos encontrado con un problema a la hora de realizar el benchmark ya que no podemos realizar la prueba para 500 de nivel de concurrencia por lo que lo hemos bajado a 50 para poder realizar la comparativa. Para ello ejecutamos la siguiente instrucción en la máquina virtual que hemos creado a parte para ello.
```shell
ab -n 1000000 -c 50 http://192.168.98.130/index.html
```
<img src="https://github.com/luisgm420/SWAP/blob/master/Practicas/practica3/Capturas%20de%20pantalla/benchmark_nginx.png">*Captura benchmark nginx*
```shell
ab -n 1000000 -c 50 http://192.168.98.131/index.html
```
<img src="https://github.com/luisgm420/SWAP/blob/master/Practicas/practica3/Capturas%20de%20pantalla/benchmark_haproxy.png">*Captura benchmark haproxy*
