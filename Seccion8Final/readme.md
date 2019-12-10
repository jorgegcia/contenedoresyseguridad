## /etc/config/daemon.json


En este punto se procederá a aplicar todas las configuraciones seguras posibles. 
Para ello crearemos el fichero **/etc/docker/daemon.json** en el cual añadiremos las opciones vistas hasta ahora en formato JSON:

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

Una vez aplicadas estas configuraciones, se reinicia el servicio docker:
````
sudo systemctl restart docker
````

Las imágenes y y contenedores previos ya no existen en este espacio de usuarios, ya que se crea una nueva carpeta en /var/lib/docker/"uid"."gid" .
En este directorio se almacenan todos los archivos de docker. Contenedores, imágenes, volúmenes, etc.

Con todas las configuraciones vistas hasta ahora aplicadas, se procedera a tratar de ejecutar el mismo CVE que mostramos en la parte 1 de este taller .

Se observar un error al ejecutar la imagen de ubuntu:

````
[administrator@localhost Seccion4DockerEscape]$ docker run --name contcomprometido -v /home/administrator/contenedoresyseguridad/Seccion4DockerEscape/main:/tmp/main -it ubuntu /bin/bash
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
7ddbc47eeb70: Pull complete
c1bbdc448b72: Pull complete
8c3b70e39044: Pull complete
45d437916d57: Pull complete
Digest: sha256:6e9f67fa63b0323e9a1e587fd71c561ba48a034504fb804fd26fd8800039835d
Status: Downloaded newer image for ubuntu:latest
docker: Error response from daemon: OCI runtime create failed: container_linux.go:348: starting container process caused "process_linux.go:402: container init caused \"rootfs_linux.go:58: mounting \\\"mqueue\\\" to rootfs \\\"/var/lib/docker/165536.165536/overlay2/bc26a761dee0baf2791c654fd85453eefdee71b68ed88697f15a73c5ef3b4982/merged\\\" at \\\"/dev/mqueue\\\" caused \\\"operation not permitted\\\"\"": unknown.
````

Esto se debe a que las etiquetas de las políticas selinux incluidas en el paquete container-selinux no se aplican correctamente en el directorio debido a conflictos entre los user namespaces y selinux. Sería necesario modificar las etiquetas o generar una política determinada con herramientas externas.

Véase UDICA: https://github.com/containers/udica

En nuestro caso para solventar este error, se mapeará directamente el directorio /dev/mqueue para que las etiquetas selinux se apliquen correctamente:

````
docker run --name contcomprometido -v /home/administrator/contenedoresyseguridad/Seccion4DockerEscape/main:/tmp/main -v /dev/mqueue:/dev/mqueue -it ubuntu /bin/bash
````

