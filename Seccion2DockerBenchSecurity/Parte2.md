# Guía de laboratorio Parte 2
Guía del laboratorio impartido por Sidertia en las jornadas CCN-STIC
***
Tiempo estimado: **5 min**
## ÍNDICE 📋
1. [docker-bench-security](#id1)

## Requisitos
<div id='id1'></div>
## 1 docker-bench-security
Repositorio oficial: https://github.com/docker/docker-bench-security.git

Ejecutamos el benchmark:
````
cd ~/benchmark/docker-bench-security

sudo ./docker-bench-security
````

Observamos el resultado. 
En este taller se intentará resolver los puntos más significativos.
Si se desea saber más sobre todos y cada uno de los controles, es posible descargarse la guía del CIS en el siguiente enlace: https://www.cisecurity.org/benchmark/docker/

# Resumen


Hemos utilizado la herramienta docker-bench-security para listar el cumplimiento de las configuraciones recomendadas por el CIS para docker.
