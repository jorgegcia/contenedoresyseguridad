# Guía de laboratorio Parte 1
Guía del laboratorio impartido por sidertia en las jornadas CCN-STIC
***
## ÍNDICE 📋
1. [Comandos básicos de docker](#id1)
2. [Namespaces ](#id2)
3. [CGroups ](#id3)
    1. [ulimits](#id31)
4. [docker-bench-security](#id4)
## Requisitos

-Descargar la máquina virtual que servirá de host de docker e importar en virtualbox.
-Descargar repositorio del laboratorio 
```
git clone https://github.com/SidertiaLabs/contenedoresyseguridad.git
```

<div id='id1'></div>
###1 Comandos básicos de docker

lista todos los contenedores
``` 
docker ps -a
```
lista las redes de docker
```
docker network ls
```
Muestra en JSON los detalles de una red
```
docker network inspect workshopnetdoc
```

<div id='id2'></div>
###2 Namespaces

Deja de compartir el espacio de nombres. Fork crea un proceso hijo antes de lanzar bash
```
sudo unshare --fork --pid --mount-proc bash
```
En la nueva bash ejecutar
```
ps aux
```
Observar que en la salida solo aparecen 2 procesos, el propio de ps y la bash como PID 1.
Observar desde fuera el proceso.

<div id='id3'></div>
###3 CGroups

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
cd ~/contenedoresyseguridad/cgroups
```
2. Inspeccionar contenido de Dockerfile
```
cat Dockerfile

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
6. Parar y elminar el contenedor
````
docker rm -f estresa1
````

#### Utilizar flags de límite
Ejecutar de nuevo el contenedor con los flags comentados al inicio:
````
docker run -d --name estresa1 --cpuset-cpus 0 --cpu-shares 512 estresacpu 
````
Observamos que solo se utiliza el 50% de la cpu 0

<div id='id31'></div>
##3.1 ulimits
La configuración ulimits permite utilizar cgroups a nivel de daemon.
añadir propiedad default-ulimit a daemon.json
Ejecutar forkbomb:
````
:(){ :|: & };:
````

Observar el resultado.
Hemos visto el uso de namespaces y cgroups y cómo utilizarlos en linux. A continuación ejecutaremos el benchmark de seguridad de docker y propondremos varias configuraciones para securizar una instalación por defecto de docker. 

<div id='id4'></div>
### 4 docker-bench-security
Repositorio oficial: https://github.com/docker/docker-bench-security.git

Ejecutamos el benchmark:
````
cd ~/benchmark/docker-bench-security

sudo ./docker-bench-security
````

Observamos el resultado. 
En este taller se intentará resolver los puntos más significativos.
Si se desea saber más sobre todos y cada uno de los controles, es posible descargarse la guía del CIS en el siguiente enlace: https://www.cisecurity.org/benchmark/docker/


# Summary 

Hemos visto como utilizar namespaces y cgroups de forma directa a través de ejemplos. Estos mismos mecanismos son los que utiliza docker para gestionar completamente un proceso aislado o contenedor, aunque lo hará de forma transparente para el usuario.

Por último hemos utilizado la herramienta docker-bench-security para listar el cumplimiento de las configuraciones recomendadas por el CIS para docker.

En la parte 2 veremos cómo escapar de un contenedor al host, y como evitar esto mediante diversos mecanismos, tales como el mapeado de nombres y selinux.


