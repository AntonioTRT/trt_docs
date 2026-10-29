# Alcance

> Estado: Vigente (protocolo V0.1 y CLI 1.0.0).

## Incluido en la versión actual

- **Protocolo TRT V0.1**: formato de trama fijo y seis comandos obligatorios de descubrimiento y observación: `PING`, `FW_VERSION`, `BOARD_VERSION`, `INFO`, `STATUS`, `CAPABILITIES`. Se añade `BUILD_ID` como extensión de descubrimiento.
- **TRT-cli**: descubrimiento de placas por puerto serie, `board info`, `board capabilities`, escucha pasiva (`trt listen`, `trt board listen <id>`), ayuda, versión y actualización.
- **Heartbeat**: evento periódico (~1/s) de presencia emitido por las placas TRT-core.
- **TRT-core**: runtime portable (parser, despachador, capacidades, módulos, servicio de sistema) y un simulador sin hardware.
- **Placas objetivo**: simulador (validación), Arduino (primera integración real, validada), después STM32 y ESP32.
- **TRT_HW**: placa base STM32G474 con conector board-to-board, bus I2C/SPI, identificación de hardware de 4 bits y driver PWM.

## Estado por módulo

| Módulo | Estado |
|---|---|
| TRT-cli | Funcional con hardware real (descubrimiento e info). Los comandos de LCD/LED existen pero no controlan hardware real aún. |
| TRT-core | V0.1 implementado; Arduino integrado; STM32 en preparación; ESP32/RP2040 solo estructura. |
| TRT_HW | Esquemáticos y huellas completos; ruteo de pistas pendiente. |

Lo que no se hace en esta etapa está en [Fuera de alcance](Fuera_De_Alcance.md).
