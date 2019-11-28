# Guía de laboratorio Parte 2
Guía del laboratorio impartido por sidertia en las jornadas CCN-STIC

Se ejecutará un exploit desarrollado por Nick Frichette el cual aprovecha una vulnerabilidad descubierta en febrero de 2019 identificada como CVE-2019-5736.
***
## ÍNDICE 📋
1. [CVE-2019-5736](#id1)
    1. [Ejecución ](#id12)
2. []

## Requisitos

1. cambiar a directorio Parte2
````
cd ~/contenedoresyseguridad/Parte2
````
2. Backup runc
````
sudo cp /usr/sbin/runc /usr/sbin/runc.backup
````


<div id='id1'></div>
###1 CVE-2019-5736

Si se desea más información se puede consultar una explicación detallada del funcionamiento en el siguiente repositorio:

https://github.com/Frichetten/CVE-2019-5736-PoC

El funcionamiento general del exploit es el siguiente, teniendo en cuenta que un atacante ha conseguido una shell en uno de nuestros contenedores:

1. Se ejecuta el exploit en el contenedor. Queda a la escucha.
2. Cuando desde el host se ejecute un exec en el contenedor llamando a sh, el contenedor sobreescribe runC por un payload deseado.

Esto se realiza mediante el acceso a runc por medio del link simbólico /proc/self/exe que apuntará a runC en el momento de ejecutar el exploit en el contenedor.
Al realizar un exec desde fuera, el exploit sobreescribe runC por el payload deseado

En nuestro caso, vamos a ejecutar el exploit en nuestra instalación docker por defecto versión 18.09.1.
Ya estamos listos para escapar de un contenedor sobreescribiendo runC y ejecutando nuestro payload como root.

<div id='id12'></div>
#### Ejecución

1. **En el Host**. Para simular a un atacante, ejecutaremos una shell y mapearemos el exploit compilado en un contenedor de base ubuntu en la carpeta tmp. 
````
docker run --name contcomprometido -v /home/administrator/CVE-2019-5736-PoC/main:/tmp/main -it ubuntu /bin/bash --
````
2. **En la shell del contenedor comprometido** recién creado cambiar de directorio a tmp y ejecutar el exploit.
````
cd /tmp
./main
````
3. **Abrir una nueva sesión ssh con el host.** No cerrar ni salir de la del punto 2. Ejecutar /bin/sh en el contenedor comprometido, momento en el que se sobreescribirá runC por el payload y se ejecutará con permisos de root en el Host.
````
docker exec -it contcomprometido /bin/sh
````
4. Observamos que en la **shell de contcomprometido** obtenemos la salida overwritten succesfully.
**En el host** observamos el fichero /tmp/ShadowOwnedBySidertia , el cual es una copia del fichero de contraseñas shadow del host.

<div id='id5'></div>
### 5 User namespace mapping

Es posible mapear un usuario root del contenedor a un usuario sin privilegios en el host. Esto se realiza ejecutando el daemon con el flag userns-remap, o añadiendolo directamente al fichero /etc/docker/daemon.json:

Añadir:
````
dockerd --usersns-remap=default
````

Con default automáticamente se crea el usuario **dockremap** en el host y se crea un nuevo directorio con nombre **"uid de dockremap":"guid dockremap"** en el directorio principal de docker. Ahora el engine trabaja sobre este directorio, por lo que los contenedores, imágenes, etc, creadas previamente no estarán disponibles.

El mapeo puede observarse en los ficheros /etc/subuid y /etc/subgid:
````
[administrator@localhost docker]$ cat /etc/subuid
administrator:100000:65536
dockremap:165536:65536
````

Usuario:(uid en el host):(rango)

Como ejemplo, en este caso dockremap tendrá UID 0 en el contenedor y UID 165536 en el host. 

Vamos a comprobar que un usuario root en un contenedor no tiene acceso root fuera de este:
Ejecutamos un contenedor de prueba que tenga mapeado el fichero shadow perteneciente a root:
````
docker run -it -v /etc/shadow:/tmp/shadow ubuntu /bin/bash
````
Desde el contenedor intentar ver o modificar el fichero devuelve un "permiso denegado":
````
root@47c1156d2f2c:/home# cat shadow
cat: shadow: Permission denied
````

## Disponibilidad

Cada contenedor se gestiona por un proceso "shim", de modo que el estado de runc, docker y containerd es irrelevante para que el contenedor siga funcionando. Esto se consigue mediante el flag live-restore

Ejecutamos el daemon con el flag live restore:
````bash
sudo dockerd --live-restore
````
A continuación levantamos los contenedores nginx y myapp
````
docker start myapp && docker start nginx
````

observamos que podemos acceder a la aplicación dotnet desde el navegador y podemos ver los dos procesos :
````
ps aux|grep docker



root     25124  0.0  0.1  10744  5136 ?        Sl   17:37   0:00 containerd-shim -namespace moby -workdir /var/lib/containerd/io.containerd.runtime.v1.linux/moby/48f85c8459816824cf5ba40d69f0718f5ed263839a7ebbd84d54b378f3aa05c9 -address /run/containerd/containerd.sock -containerd-binary /usr/bin/containerd -runtime-root /var/run/docker/runtime-runc
root     25304  0.0  0.1  10744  5584 ?        Sl   17:37   0:00 containerd-shim -namespace moby -workdir /var/lib/containerd/io.containerd.runtime.v1.linux/moby/c45074597bcbc9be496238eb0cdd2d136b58cf30ca3897764372815794726a62 -address /run/containerd/containerd.sock -containerd-binary /usr/bin/containerd -runtime-root /var/run/docker/runtime-runc
adminis+ 26523  0.0  0.0  12112  1088 pts/1    S+   17:40   0:00 grep --color=auto docker

````

Terminamos el demonio con ctrl + C y volvemos a utilizar el comando ps aux|grep docker. Los dos procesos siguen apareciendo y la aplicación dotnet sigue siendo accesible desde el navegador.

## 5.2 Selinux

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


###Control de permisos SELinux en volúmenes

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


En este punto queremos aplicar todas las configuraciones seguras posibles. 
Para ello crearemos el fichero **/etc/docker/daemon.json** en el cual añadiremos las opciones deseadas de la siguiente manera en formato JSON:

```json
{
        "selinux-enabled":true,
        "live-restore":true,
        "no-new-privileges":true,
        "userns-remap":"default",
        "default-ulimits": {
                "nofile": {
                        "Name": "nofile",
                        "Hard": 64000,
                        "Soft": 64000
                },
                "nproc": {
                        "Name": "nproc",
                        "Hard": 200,
                        "Soft": 100
                }
        }


}
```

Una vez aplicadas estas configuraciones, levantamos el servicio docker:
````
sudo systemctl start docker
````

Como vimos en la parte 1 , sección 2 Namespaces, Las imágenes y y contenedores previos ya no esxisten en este espacio de usuarios, por lo que vamos a construir de nuevo las imágenes dotnet y nginx:

Compilamos la aplicación dotnet:
````
````

# Necesario mapear /dev/mqueue. Permisos? 
sudo docker run -ti -v /dev/mqueue:/dev/mqueue ubuntu /bin/bash

## /etc/config/daemon.json


