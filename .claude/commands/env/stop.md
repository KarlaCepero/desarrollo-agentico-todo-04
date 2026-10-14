---
description: Stop the TODO development environment without removing containers, volumes or data
allowed-tools: Bash, Read
---

# Para el entorno de desarrollo

Detén solo lo que pertenezca a este repositorio. No elimines contenedores, volúmenes ni datos, y no uses `docker compose down`.

## Detecta el modo

- **Docker completo:** `docker compose ps` muestra el servicio `app` en marcha.
- **Host:** hay un proceso `buffalo dev` cuyo directorio de trabajo es la raíz de este repositorio.

Si no hay nada en marcha, informa de ello y termina.

## Modo Docker completo

1. Ejecuta `docker compose stop`.
2. Comprueba con `docker compose ps` que ya no queda ningún servicio en marcha.

## Modo host

1. Localiza los procesos `buffalo dev` con `pgrep -f "buffalo dev"`. Para cada uno, comprueba con `lsof -a -p <pid> -d cwd` que su directorio de trabajo es la raíz de este repositorio. Si pertenece a otro proyecto, no lo toques.
2. Detén los de este repositorio con `kill <pid>`.
3. Ejecuta `docker compose stop db`.

## Comprobación

Antes de parar, anota los puertos publicados (`docker compose port app 3000` y `docker compose port db 5432`; en modo host, el `PORT` de `buffalo dev`). Después, `/healthz` ya no responde en ese puerto y nada de este repositorio sigue escuchando en ellos. Si otro proyecto escucha en `3000` o `5432`, no es de este repositorio: déjalo intacto.

## Resultado

Informa de:

- modo detectado;
- servicios y procesos detenidos;
- resultado de la comprobación;
- procesos de otros proyectos que hayas dejado intactos.
