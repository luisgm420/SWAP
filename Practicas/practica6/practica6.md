**Práctica 6**
==============

1°. Añadimos dos discos en la máquina virtual mediantes las opciones de VMware del mismo tamaño.

2° realizar la configuración de dos discos en RAID 1 bajo Ubuntu, automatizandoel montaje del dispositivo creado al inicio del sistema.

Instalamos el software necesario para configurar el RAID:
```shell
sudo apt-get install mdadm
```

Creamos el RAID 1, usando el dispositivo /dev/md0, indicando el número de dispositivos a utilizar (2), así como su ubicación:
```shell
sudo mdadm -C /dev/md0 --level=raid1 --raid-devices=2 /dev/sdb /dev/sdc
```

Ahora le damos formato mediante el siguiente comando:
```shell
sudo mkfs /dev/md0
```

Ahora ya podemos crear el directorio en el que se montará la unidad del RAID:
```shell
sudo mkdir /dat
sudo mount /dev/md0 /dat
```

Para que el sistema monte el dispositivo RAID creado al arrancar el sistema. Para ello añadimos la siguiente linea al archivo /etc/fstab:
```shell
UUID=3f8752eb-b137-4b6a-9456-a79f92a73115 /dat ext2 defaults 0 
```

3° Simular un fallo en uno de los discos del RAID (mediante comandos con el mdadm), retirarlo “en caliente”, comprobar que se puede acceder a la información que hay almacenada en el RAID, y por último, añadirlo al conjunto y comprobar que se reconstruye correctamente.

Simulación fallo en uno de los discos:
```shell
sudo mdadm --manage --set-faulty /dev/md0 /dev/sdb
```

Retirar disco en caliente:
```shell
sudo mdadm --manage --remove /dev/md0 /dev/sdb
```
<img src="https://github.com/luisgm420/SWAP/blob/master/Practicas/practica6/Capturas%20de%20pantalla/retirar_en_caliente.png">*Retirar disco en caliente*

Añadimos disco en caliente:
```shell
sudo mdadm --manage --add /dev/md0 /dev/sdb
```
<img src="https://github.com/luisgm420/SWAP/blob/master/Practicas/practica6/Capturas%20de%20pantalla/aniadir_disco.png">*Añadir disco*
