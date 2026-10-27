# Requisitos del sistema

> Estado: Parcial (derivado de los repositorios; no hay documento formal de requisitos).

## Host (TRT-cli)

- Python 3.13+. Dependencias: Typer, Rich, pyserial. Pruebas: pytest.
- Windows, Linux o macOS.
- Puerto serie/USB disponible hacia la placa.

## Firmware (TRT-core)

- Placas con runtime completo (STL y memoria dinámica): simulador, ESP32, STM32.
- Placas con poca RAM (AVR/Uno, 2 KB): no soportadas aún por el runtime general; requieren un subconjunto de memoria estática.
- Toolchain: PlatformIO (Arduino), STM32CubeIDE (STM32), CMake (simulador y pruebas).
- Cargar firmware Arduino: Python 3.10+, `platformio`, `truststore`.

## Hardware (TRT_HW)

- Alimentación: USB-C 5 V, regulador a 3,3 V.
- MCU: STM32G474RET6 (Cortex-M4F, 170 MHz).
- Interconexión con placas hijas: I2C, SPI, GPIO/ADC/DAC y 4 bits de HWID.

## Requisitos funcionales de protocolo (V0.1)

1. Implementar los 6 comandos obligatorios.
2. Responder `ERROR_UNSUPPORTED_COMMAND` a lo no soportado.
3. No bloquear el bucle principal.
4. Seguir operando sin host.

Consumos, ancho de banda USB y compatibilidad de SO más allá de lo anterior: **pendiente de definir**.

## Información pendiente

> TODO: Por hacer. Información no encontrada o no suficientemente documentada.

Falta: consumos de energía, ancho de banda USB y compatibilidad de sistemas operativos.
