# SysDD: visión técnica unificada

> Estado: Vigente. Mapa de los repositorios y cómo encajan; el flujo está en [Arquitectura del sistema](Arquitectura_Sistema.md).

## Los tres módulos

| | TRT-cli | TRT-core | TRT_HW |
|---|---|---|---|
| Qué es | App Python | Runtime C++ | Proyecto KiCad |
| Corre en | PC (Windows/Linux/macOS) | Microcontrolador o simulador | — (es la placa física) |
| Tecnología | Typer, Rich, pyserial, pytest | C++, CMake, PlatformIO | KiCad 8, JLCPCB |
| Habla con | Placa, vía protocolo TRT | CLI (protocolo) y hardware (interfaces) | Firmware (pines, buses) |
| Doc técnica propia | `TRT-cli/docs/` | `TRT-core/docs/` | `TRT_HW/hardware/carrier-board/README.md` |

## Interacciones

- **TRT-cli ↔ TRT-core**: solo por el [Protocolo TRT](Protocolo_TRT.md). No comparten código.
- **TRT-core ↔ TRT_HW**: TRT-core define interfaces (`IGpio`, `IPwm`, `IAdc`…); cada familia de placas en `TRT-core/boards/` las implementa para su MCU. El firmware de la placa base STM32G474 usará `boards/stm32`.
- **TRT-cli ↔ TRT_HW**: ninguna directa.

## Dónde está cada cosa

| Quiero… | Ir a |
|---|---|
| Añadir un comando al CLI | [Arquitectura del CLI](../04_Software/Arquitectura_CLI.md) |
| Añadir un comando/servicio al firmware | [Arquitectura del firmware](../04_Software/Arquitectura_Firmware.md) |
| Portar a una placa nueva | `TRT-core/docs/porting-guide.md` |
| Compilar y cargar un sketch | [Entorno de desarrollo](../04_Software/Entorno_Desarrollo.md) |
| Ver pines y conectores | [HWDD](../03_Hardware/HWDD.md) |

## Estado de integración

- Validado de extremo a extremo: TRT-cli ↔ Arduino Uno (COM4) con TRT-core.
- La placa base de TRT_HW aún no tiene firmware TRT-core integrado.
- Existe un proyecto STM32 de prueba (`TRT-core/TRT-stm32`, STM32L073RZ, CubeIDE) sin relación con TRT-core todavía.
