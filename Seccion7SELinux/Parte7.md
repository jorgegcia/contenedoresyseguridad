# Guía de laboratorio Parte 7
Guía del laboratorio impartido por Sidertia en las jornadas CCN-STIC
***
## ÍNDICE 📋
1. [SELinux](#id1)

<div id='id1'></div>
## 1 SELinux

Vemos si selinux está habilitado por defecto en docker:
````
docker info
....
security options:

selinux
````

Selinux debe estar habilitado como enforcing (rechaza todo menos lo permitido) en el host:
````
[administrator@localhost sysconfig]$ getenforce
Enforcing
````

En este momento se están aplicando las políticas y etiquetas que la instalación docker por defecto aplica.

Para ver las etiquetas relativas a un proceso, se utiliza el flag Z.

Listar etiquetas de un proceso:
````
ps Z
LABEL                             PID TTY      STAT   TIME COMMAND
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 3763 pts/2 Ss   0:00 -bash

````

Etiquetas SELinux del directorio de docker:
````
sudo ls -lZ /var/lib/docker
....
system_u:object_r:container_var_lib_t:s0
````
**Usuario:rol:tipo:Rango**

Lanzamos dockerd con la opción selinux habilitada:
````
sudo dockerd --selinux-enabled=true
````

Por defecto, selinux permite a un contenedor docker leer de los directorios /etc y /usr pero no escribir.
Vamos a comprobarlo:

Creamos un fichero con contenido en el directorio /etc y listamos sus etiquetas selinux:
````
echo "primer test selinux" | sudo tee /etc/testselinux
ls -lZ /etc/testselinux

....
[administrator@localhost ~]$ ls -lZ /etc/testselinux
-rw-r--r--. 1 root root unconfined_u:object_r:etc_t:s0 20 nov 28 04:47 /etc/testselinux
....
````

Comprobamos si podemos leer y escribir en este fichero desde un contenedor:
````
[administrator@localhost ~]$
docker run -ti -v /etc/testselinux:/tmp/testselinux ubuntu bash

root@f70db111b835:/#
cat /tmp/testselinux

....
primer test selinux
....

root@f70db111b835:/#
echo "nl" | tee /tmp/testselinux

....
tee: /tmp/testselinux: Permission denied
....
````

Selinux permite la lectura, pero está denegando la escritura en etc por parte de un proceso con la etiqueta de contenedor container_t:

````
root@f70db111b835:/# ps auxZ
LABEL                           USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
system_u:system_r:container_t:s0:c360,c461 root 1 0.0  0.0 18508 3408 pts/0    Ss   09:50   0:00 bash

````

Podemos observar que en otro directorio no permite leer ni escribir, por ejemplo /home:
```
[administrator@localhost ~]$
docker run -ti -v /home/testselinux:/tmp/testselinux ubuntu bash

root@f70db111b835:/#
cat /tmp/testselinux

....
cat: /tmp/testselinux: Permission denied
....

root@f70db111b835:/#
echo "nl" | tee /tmp/testselinux

....
tee: /tmp/testselinux: Permission denied
....
```


##Control de permisos SELinux en volúmenes

Es posible utilizar etiquetas de volumen para permitir acceso de un contenedor a un fichero o directorio mapeado. 
Esto se realiza con las opciones z minúscula y Z mayúscula.

z minúscula  hace un fichero accesible a todos los contenedores de tipo container_t.

````
docker run -ti -v /home/testselinux:/tmp/testselinux:z ubuntu bash
````

Z mayúscula hace un fichero accesible únicamente a un contenedor

````
docker run -ti -v /home/testselinux:/tmp/testselinux:Z ubuntu bash
````
