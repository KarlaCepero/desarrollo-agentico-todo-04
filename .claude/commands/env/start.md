---
description: Start the TODO development environment and verify that the application responds
allowed-tools: Bash, Read
---

# Arranca el entorno de desarrollo

Levanta la aplicación desde la raíz del repositorio. No mates procesos existentes ni elimines volúmenes.

## Modos

- Usa el modo **host** por defecto.
- Usa el modo **Docker completo** si `$ARGUMENTS` contiene `--docker` o si Go/Buffalo no están disponibles en el host.

## Comprobaciones previas

1. Confirma que `docker compose` está disponible y que el daemon responde.
2. Elige los puertos del host. Por defecto son `APP_PORT=3000` (app) y `DB_PORT=5432` (PostgreSQL); si ya vienen definidos en el entorno, parte de esos.
   - Si `docker compose ps` muestra este proyecto en marcha y `/healthz` responde en `docker compose port app 3000`, reutiliza el entorno tal cual y salta al resultado.
   - Comprueba cada puerto con `lsof -nP -iTCP:<puerto> -sTCP:LISTEN`. Si lo ocupa otro proceso o proyecto, no lo finalices: busca el siguiente puerto libre (`3001` a `3005` para la app, `5433` a `5439` para la base de datos) y usa ese. Los permisos solo autorizan `curl` a `localhost:3000`–`3005`: si no queda ninguno libre en ese rango, informa y detente.
   - Exporta `APP_PORT` y `DB_PORT` en **todos** los comandos siguientes de este arranque (`APP_PORT=… DB_PORT=… docker compose …`). Si no, `docker compose up` recrearía los contenedores con los puertos por defecto.

## Modo host

1. Comprueba que `go` y `buffalo` están disponibles.
   - Si falta alguno, explica el bloqueo y continúa mediante Docker completo.
2. Ejecuta `APP_PORT=… DB_PORT=… docker compose up -d db`.
3. Espera a que PostgreSQL figure como saludable en `docker compose ps`.
4. Ejecuta `go mod download`.
5. Inicia `PORT=$APP_PORT DB_PORT=$DB_PORT buffalo dev` en segundo plano desde la raíz y conserva su salida para poder diagnosticar el arranque.
6. Espera hasta que `http://localhost:$APP_PORT/healthz` responda correctamente.

## Modo Docker completo

1. Ejecuta `APP_PORT=… DB_PORT=… docker compose up --build -d`.
2. Espera a que `db` y `app` figuren como saludables en `docker compose ps`.
3. Comprueba `/healthz` en el puerto que publica Docker: `docker compose port app 3000` da `0.0.0.0:<puerto>`, y la URL es `http://localhost:<puerto>/healthz`.

## Resultado

Informa de:

- modo utilizado;
- servicios iniciados;
- puertos usados (`APP_PORT`, `DB_PORT`) y si alguno se movió porque el puerto por defecto lo ocupaba otro proyecto (cuál);
- URL de la aplicación;
- respuesta de `/healthz`;
- cualquier fallback, advertencia o proceso que deba mantenerse activo.
