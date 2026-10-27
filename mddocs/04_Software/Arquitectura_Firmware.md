# Arquitectura del firmware (TRT-core)

> Estado: Vigente. Detalle: `TRT-core/docs/architecture.md`, `execution-model.md`, `state-machine.md`, `portability.md`.

## Qué es

Runtime de firmware portable compartido por todas las placas. **No** es una aplicación ni firmware de una placa concreta: está entre el protocolo y la implementación de cada placa.

## Capas

```text
Protocolo TRT → Transporte (ITransport) → TRT-core → Interfaces (IGpio, IPwm…) → Placa (boards/<familia>)
```

## Componentes (`TRT-core/trt-core/`)

| Carpeta | Responsabilidad |
|---|---|
| `core/protocol/` | Parser y encoder de tramas; IDs de comandos |
| `core/dispatcher/` | Registro de comandos y despacho al manejador |
| `core/board/` | `BoardInfo` (identidad) y `BoardContext` (reúne todo lo de la placa) |
| `core/capabilities/` | `CapabilityManager`: qué soporta la placa |
| `core/services/` | `SystemService`: responde los comandos V0.1 |
| `core/errors/`, `core/logging/` | Códigos de error y log |
| `core/config/features.h` | Flags de compilación (STL, memoria dinámica, módulos, tamaño de trama) |
| `interfaces/` | Contratos de hardware: GPIO, PWM, ADC, DAC, I2C, SPI, UART, LCD, NeoPixel, relé, storage |
| `modules/` | `IModule` y `ModuleManager` (módulos concretos aún vacíos) |
| `simulator/` | Placa simulada para pruebas sin hardware |
| `configs/` | Valores por defecto de configuración |

## Integración por placa (`TRT-core/boards/`)

Cada familia implementa las interfaces y un `ITransport`, registra capacidades y módulos y ejecuta el bucle:

| Familia | Estado |
|---|---|
| `arduino` | Integrada y validada (runtime serie en `boards/arduino/runtime`) |
| `stm32` | Runtime base presente (transporte UART, bucle); integración con CubeIDE pendiente |
| `esp32`, `rp2040`, `simulator` | Solo estructura |

## Flujo en ejecución

```text
bytes → Parser → cola RX → cola de comandos → Dispatcher → máquina de estados → Servicio → cola TX → Transporte
                                              eventos: Servicio → cola de eventos → cola TX
```

El bucle principal es **cooperativo y no bloqueante**: actualiza servicios, procesa eventos y comandos, actualiza el estado y transmite pendientes, con un máximo de elementos por iteración. Nunca espera mensajes del host.

## Conceptos clave

- **Capacidades**: la placa declara lo que soporta; lo demás recibe `ERROR_UNSUPPORTED_COMMAND`.
- **Máquina de estados** (BOOT, IDLE, READY, BUSY, WARNING, ERROR, FIRMWARE_UPDATE): decide el comportamiento; los comandos piden intención, no manejan hardware.
- **Servicios**: encoder, botón, LED ring, comunicación, módulos, configuración.
- **Un solo código fuente**: los flags `TRT_HAS_*` eligen la variante de memoria estática (AVR) o completa; el protocolo no cambia.

## Implementado hoy vs. diseñado

Implementado: parser/encoder, dispatcher, capacidades, `SystemService` con los comandos V0.1, simulador y pruebas, runtime Arduino. Diseñado pero sin implementar: máquina de estados, servicios de encoder/botón/LED, cola de eventos, configuración persistente, validación CRC16, variante AVR.

## Cómo añadir una placa

Ver `TRT-core/docs/porting-guide.md`. Regla: nada específico de placa dentro de `core/`.
