# Calculadora — Backend

API en FastAPI de una calculadora simple, con historial de operaciones persistido opcionalmente en PostgreSQL.

Este repositorio es un **fork** de [`MatyAlts/calculadora-backend`](https://github.com/MatyAlts/calculadora-backend), el repositorio base provisto por la cátedra de **Programación 3** (Tecnicatura Universitaria en Programación, UTN FRM) para el Trabajo Práctico Integrador de la unidad de despliegue en un VPS.

## Integrantes del grupo

| Nombre | Legajo |
|---|---|
| Mariano Chirino | 41031 |
| Facundo Quiroga | 52737 |
| Andrés Fabre | 53885 |

## Qué hace esta API

- `POST /api/calcular`: recibe dos números y una operación (`suma`, `resta`, `multiplicacion`, `division`, `potencia`) y devuelve el resultado.
- `GET /api/historial`: devuelve las últimas operaciones guardadas, de la más reciente a la más vieja. Solo disponible si hay base de datos configurada.
- `GET /api/salud`: informa el estado del servicio y si la persistencia está disponible (`{"estado": "ok", "persistencia": true|false}`).

La persistencia en base de datos es **opcional**: si la variable de entorno `DATABASE_URL` no está definida, la API funciona igual (calcula y responde), simplemente no guarda ni lista historial. Es un ejemplo de degradación elegante: una funcionalidad secundaria (guardar el historial) nunca puede tumbar la funcionalidad principal (calcular).

## Cambios propios sobre el repositorio original

- Se agregó la operación **`potencia`**, con manejo explícito de los casos límite:
  - Base `0` elevada a exponente negativo (en Python explota con `ZeroDivisionError`) → se captura y devuelve `400`.
  - Resultados que desbordan el rango de un `float` (`OverflowError`) → `400`.
  - Base negativa con exponente fraccionario (resultado complejo en Python) → `400`.
- Se actualizó la suite de tests (`test_main.py`) para reflejar que `potencia` es ahora una operación válida, y se agregó un test específico para ella.
- Se corrigió el trigger de despliegue automático del workflow de CI/CD para forzar IPv4 (`curl -4`), resolviendo un problema de conectividad intermitente entre los runners de GitHub Actions y la VPS.

## Despliegue

Desplegado en un VPS propio administrado con [Easypanel](https://easypanel.io), con HTTPS emitido automáticamente por Let's Encrypt.

- API en producción: `https://api.marianochirino.me`
- Documentación interactiva (Swagger): `https://api.marianochirino.me/docs`

## Integración continua y despliegue

El workflow en `.github/workflows/tests.yml` corre la suite de `pytest` en cada push y pull request. Si las pruebas pasan y el push fue directo a `main`, se dispara automáticamente un redespliegue en Easypanel mediante su URL de trigger (guardada como secreto del repositorio, nunca en el código). Si las pruebas fallan, el despliegue no se ejecuta.

## Variables de entorno

| Variable | Descripción |
|---|---|
| `ORIGENES_PERMITIDOS` | Origen (esquema + host) autorizado para CORS, ej. `https://calculadora.marianochirino.me` |
| `DATABASE_URL` | Cadena de conexión a PostgreSQL (opcional). Si no está definida, la API arranca sin historial. |
