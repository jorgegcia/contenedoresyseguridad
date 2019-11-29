## /etc/config/daemon.json


En este punto queremos aplicar todas las configuraciones seguras posibles. 
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

Una vez aplicadas estas configuraciones, reiniciamos el servicio docker:
````
sudo systemctl restart docker
````

Las imágenes y y contenedores previos ya no existen en este espacio de usuarios, ya que se crea una nueva carpeta en /var/lib/docker/"uid"."gid" .
En este directorio se almacenan todos los archivos de docker. Contenedores, imágenes, volúmenes, etc.

Es necesa