# Alcance del proyecto TRT

## Conectividad

- El dispositivo podrá conectarse a una computadora mediante USB.
- Permitirá la comunicación entre la computadora y las placas electrónicas.
- Utilizará comunicación serial para enviar y recibir información.

## Compatibilidad

- El sistema tendrá soporte para diferentes microcontroladores.
- Se contempla compatibilidad con Arduino, STM32 y ESP32.
- Contará con un simulador para realizar pruebas sin necesidad de hardware físico.

## Identificación

- El sistema permitirá detectar las placas conectadas.
- Podrá consultar la versión del firmware.
- Permitirá obtener información sobre el estado y las capacidades del dispositivo.

## Software

- Contará con una herramienta de línea de comandos (TRT-cli).
- Permitirá consultar información de las placas desde una computadora.
- Tendrá un firmware común (TRT-core) para gestionar la comunicación.

## Limitaciones

- La primera versión utilizará comunicación USB/serial.
- No incluirá control remoto de todos los periféricos.
- Algunas funciones estarán disponibles en futuras versiones.