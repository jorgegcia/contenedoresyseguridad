# Guía de laboratorio Parte 6
Guía del laboratorio impartido por Sidertia en las jornadas CCN-STIC
Tiempo estimado: **5 min**
***
## ÍNDICE 📋
1. [Disponibilidad](#id1)

<div id='id1'></div>
## 1 Disponibilidad

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

Si restauramos de nuevo el daemon, podemos controlar los contenedores de nuevo.