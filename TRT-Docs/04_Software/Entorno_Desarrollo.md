# Entorno de desarrollo

> Estado: Vigente. Detalle: `TRT-core/docs/development-environment.md`, `TRT-core/tools/README.md`, `TRT-cli/QUICKSTART.md`.

## TRT-cli

```bash
cd TRT-cli
pip install -e .
trt discover
trt board info 101
trt listen
pytest
```

## TRT-core: cargar un sketch Arduino

1. Instalar: `python -m pip install platformio truststore python-certifi-win32` (Python 3.10+).
2. Editar las constantes al inicio de `TRT-core/tools/upload.py` (`COM_PORT`, `PROJECT`, `BOARD`, `BOARD_ID`, `FW_VERSION`…).
3. Ejecutar `python tools/upload.py`.

La herramienta no toca el sketch original: genera un proyecto aislado en `.trt-build/<proyecto>/`, compila y sube. Si todo sale bien añade una fila a `build_registry.csv` (ID de build, placa, versión, commit).

En redes corporativas con inspección SSL, `truststore` resuelve los errores de certificados de PlatformIO.

## TRT-core: simulador y pruebas

Se construyen con CMake (`TRT-core/trt-core/CMakeLists.txt`); el simulador implementa los comandos V0.1 sin hardware.

## STM32

Se compila y depura con **STM32CubeIDE**; TRT-core se integrará como librería, no genera el proyecto.

## Verificación rápida de todo el flujo

Cargar un sketch en un Arduino → `trt listen` (debe verse `Board <BOARD_ID>: HEARTBEAT` cada segundo) → `trt discover` → `trt board info <BOARD_ID>`.

`trt listen` valida firmware, carga, transporte y runtime; `trt discover` valida la identidad. Ver `Arquitectura_CLI.md`.
