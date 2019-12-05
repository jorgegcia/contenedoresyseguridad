# Guía de laboratorio Parte 4
Guía del laboratorio impartido por Sidertia en las jornadas CCN-STIC
***
Tiempo estimado: **10 min**
## ÍNDICE 📋
1. [CGroups ](#id1)
    1. [ulimits](#id31)


<div id='id1'></div>
##1 CGroups

Se puede limitar los recursos a nivel de daemon o a nivel de cada contenedor.
Para un contenedor docker run tiene varios flags:

```
--cpu-shares            de 0-1024, limita la porción máxima de CPU de un contenedor
...
--cpuset-cpus           CPUs permitidas para la ejecución. Ej (0-3)
...
--pids-limit            limite de process ids
```

Para probar cómo limitar los recursos de un contenedor, utilizaremos el binario stress sobre una imagen de ubuntu. Esto es, un contenedor que realiza raíces cuadradas continuamente, lo que causa un alto uso de recursos.

1. Cambiar a directorio cgroups
```
cd ~/contenedoresyseguridad/Seccion2CGroups
```
2. Crear Dockerfile
```
FROM ubuntu:latest

RUN apt-get update && apt-get install -y stress

CMD stress -c 2
```
3. Vemos que podemos limitar recursos también en la etapa de build.
````
docker build -h
````
Build
````
docker build -t estresacpu . 
````
4. Ejecutar contenedor basado en la imagen que hemos construido
````
docker run -d --name estresa1 estresacpu
````
5. Observar la carga sobre las 2 cpus con htop al 100%
````
docker stats -a
````
6. Parar y elminar el contenedor
````
docker rm -f estresa1
````

### Utilizar flags de límite
Ejecutar de nuevo el contenedor con los flags comentados al inicio:
````
docker run -d --name estresa1 --cpuset-cpus 0 --cpu-shares 512 --pids-limit 100 estresacpu 
````
Observamos que solo se utiliza el 50% de la cpu 0

````
docker exec -it estresa1 /bin/bash
````

Ejecutar forkbomb:
````
:(){ :|: & };:
````

<div id='id31'></div>
##3.1 ulimits
La configuración ulimits permite utilizar cgroups a nivel de daemon.
añadir propiedad default-ulimit a daemon.json

Observar el resultado.

# Resumen
Hemos visto el uso de namespaces y cgroups y cómo utilizarlos en linux. En las siguientes partes veremos selinux y user namespaces.