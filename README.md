# CAUDAL Simulador

Simulador virtual del acueducto (gemelo digital) en **Python 3.12**: como no hay datos reales de la región, genera lo que llegaría del campo (clima, fuente, tanque, consumo por sector, fugas, fontanero con errores humanos, sensores con fallas, válvulas motorizadas y reportes de la comunidad) y lo envía al backend **por las mismas puertas que usaría la realidad**. Incluye un tablero web.

## Estado

**Implementado** (verificado el 2026-11-16 en `develop`, `521048a`): 42 módulos en `src/caudal_sim/`, 5 escenarios YAML con semilla en `scenarios/` y **373 pruebas** que pasan (`uv run pytest -q`), con `ruff check` limpio.

| Pieza | Qué hace | Módulos principales |
|---|---|---|
| Modelo físico | Clima, fuente, tanque (balance de masa), demanda por sector y fugas | `climate`, `source`, `tank`, `demand`, `leaks`, `simulation_step` |
| Actores de campo | Fontanero con errores humanos y sin señal, sensores con fallas, válvulas motorizadas | `readings`, `offline_buffer`, `sensors`, `devices`, `valve_actuator`, `valve_command_executor` |
| Puertas reales | Cliente de la API (JWT), firma Ed25519 de dispositivos, guardia de acueducto demo | `api_client`, `signing`, `demo_guard`, `telemetry_transport` |
| Modos | `generate`, `backfill`, `live`, `demo`, `tablero`, `export-actas`, `export-backtest` | `cli`, `backfill`, `live`, `demo`, `export` |
| Tablero | API local (FastAPI, Swagger en `/docs`) y tablero web (Vite/React) | `dashboard_api`, `dashboard/` |

Solo escribe en el acueducto **demo**: `DemoTargetGuard` se niega a enviar datos a un acueducto con `is_demo = false`, y el backend lo vuelve a comprobar. La especificación está en el backend: [Datos simulados](https://github.com/Jenifrutica/caudal-backend/blob/develop/docs/Datos-simulados.md), [Simulador y hardware](https://github.com/Jenifrutica/caudal-backend/blob/develop/docs/Simulador-y-hardware.md) y [Protocolo de dispositivos](https://github.com/Jenifrutica/caudal-backend/blob/develop/docs/Protocolo-de-dispositivos.md). Por qué es un proceso separado: [ADR-006](https://github.com/Jenifrutica/caudal-backend/blob/develop/docs/ADRs/ADR-006-simulador-como-proceso-separado.md).

## Arranque rápido

```bash
uv sync                                                        # Python 3.12 y dependencias
uv run caudal-sim --help                                       # comandos disponibles
uv run caudal-sim generate --scenario normal-year --days 365 --seed 42   # sin backend: genera datos
uv run caudal-sim tablero                                      # tablero en http://127.0.0.1:8000
uv run pytest -q && uv run ruff check .                        # pruebas y lint
```

Contra el backend local (`make up` en `caudal-backend`): copia `.env.example` a `.env` con las cuentas que la Junta creó en el acueducto demo y usa `caudal-sim live` o `caudal-sim demo`. Con Docker, desde `caudal-backend`: `docker compose --profile sim up -d sim` (tablero en http://localhost:8090). Detalle en [Despliegue](docs/despliegue.md) y [Operación en vivo](docs/operacion-live.md).

## Documentación

- [AGENTS.md](AGENTS.md): reglas obligatorias para personas y agentes.
- [Flujo de trabajo](.agents/workflow.md): cuentas, ramas, commits y PR.
- [Política de seguridad](SECURITY.md).
- [Escenarios](docs/escenarios.md): catálogo de escenarios YAML, sus supuestos y cómo crear uno nuevo.
- [Despliegue](docs/despliegue.md): ejecución local y en contenedor, variables de entorno, puertos y tablero.
- [Operación en vivo](docs/operacion-live.md) y [claves y secretos](docs/claves-y-secretos.md).

## Equipo

Nicolas Diaz (`NicoalsD`) · Drako Salazar (`Drako2305`) · Nicolas Mora (`nicomora70`). Proyecto de la materia Patrones de Software.
