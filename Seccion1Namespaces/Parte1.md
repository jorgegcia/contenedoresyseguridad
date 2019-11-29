# Guía de laboratorio Parte 1
Guía del laboratorio impartido por Sidertia en las jornadas CCN-STIC
Tiempo estimado: **10 min**
***
## ÍNDICE 📋
1. [Comandos básicos de docker](#id1)
2. [Entorno](#id2)
3. [Namespaces ](#id3)

## Requisitos

-Descargar la máquina virtual que servirá de host de docker e importar en virtualbox.
-Descargar repositorio del laboratorio 
````
git clone https://github.com/SidertiaLabs/contenedoresyseguridad.git
````

<div id='id1'></div>
##1 Comandos básicos de docker

Lista todos los contenedores
````
docker ps -a
````
Lista imágenes
````
docker image ls
````
Lista histórico de capas de una imagen
````
docker history "nombre imagen"
````
Lista las redes de docker
````
docker network ls
````
Muestra en JSON los detalles de una red
````
docker network inspect workshopnetdoc
````
Muestra en JSON los detalles de un contenedor
````
docker inspect "nombre contenedor"
````

<div id='id2'></div>
##2 Entorno

El entorno de pruebas se compone de tres elementos:
1. Contenedor nginx que actúa como proxy inverso.
2. Aplicación sencilla de dotnet core accesible únicamente a través de nginx.
3. Red aislada tipo bridge para sostener la red del entorno

En el directorio contenedores se encuentra el código de la aplicación dotnetcore y la configuración de nginx.

Además hay 3 scripts, cada uno de ellos levanta cada uno de los puntos comentados previamente.

Para compilar la aplicación de dotnetcore cambiamos al directorio app, en el cual se encuentra el Dockerfile que construirá las imágenes necesarias para compilar la aplicación:
````
cd app

docker build -t app .
````

Posteriormente levantamos el contenedor:
````
./runDotnetapp.sh
````

<div id='id3'></div>
##3 Namespaces

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