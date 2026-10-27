# SWDD: visión general del software

> Estado: Vigente.

## Qué incluye

| Módulo | Lenguaje | Estructura y detalle |
|---|---|---|
| TRT-cli | Python 3.13+ | [Arquitectura del CLI](Arquitectura_CLI.md) |
| TRT-core | C++ | [Arquitectura del firmware](Arquitectura_Firmware.md) |
| Compilación y carga | Python + PlatformIO / CMake / CubeIDE | [Entorno de desarrollo](Entorno_Desarrollo.md) |

Ambos hablan el [Protocolo TRT](../02_Sistema/Protocolo_TRT.md) y no comparten código.

## Estructura de TRT-core (a nivel de repositorio)

| Carpeta | Contenido |
|---|---|
| `trt-core/` | El runtime: `core/`, `interfaces/`, `modules/`, `simulator/`, `tests/`, `configs/` |
| `boards/` | Integración por familia de placa (arduino, stm32, esp32, rp2040, simulator) |
| `TRT-arduino/` | Sketches de ejemplo y librerías Arduino de terceros |
| `TRT-stm32/` | Proyecto STM32CubeIDE de prueba |
| `TRT-esp32/`, `TRT-simulator/` | Ejemplos |
| `tools/upload.py` | Compila y carga un sketch con TRT-core |
| `docs/` | Documentación técnica (inglés) |

## Convenciones

- Documentación de repos: `docs/` de cada repositorio (inglés). Documentación de onboarding: `trt_docs` (español).
- El [bus I2C entre módulos](Protocolo_I2C.md) aún no está especificado.
