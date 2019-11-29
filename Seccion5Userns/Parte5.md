# Guía de laboratorio Parte 5
Guía del laboratorio impartido por Sidertia en las jornadas CCN-STIC
***
## ÍNDICE 📋
1. [User namespace mapping](#id1)

<div id='id1'></div>
### 1 User namespace mapping

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
