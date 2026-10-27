# Glosario

> Estado: Vigente. Estos términos se usan de forma consistente en toda la documentación.

| Término | Significado |
|---|---|
| **TRT** | *Tool Runtime Terminal*: el framework completo. |
| **TRT-cli** | Herramienta de línea de comandos en el PC (comando `trt`). |
| **TRT-core** | Runtime de firmware portable compartido por todas las placas. |
| **TRT_HW** | Repositorio de hardware (KiCad). |
| **Host** | El PC donde corre TRT-cli. |
| **Placa** (*board*) | Dispositivo con microcontrolador que corre TRT-core. |
| **Placa base** (*carrier board*) | Tarjeta principal de TRT_HW con el MCU STM32G474. |
| **Placa hija / módulo de hardware** | Tarjeta que se conecta a la placa base por el conector B2B. No confundir con *módulo de firmware*. |
| **Módulo (firmware)** | Abstracción `IModule` de TRT-core que describe un complemento (LCD, relé, NeoPixel…). |
| **Familia de placas** | Grupo de placas con el mismo MCU/plataforma (`boards/arduino`, `stm32`, …). |
| **Simulador** | "Placa" en software para probar el protocolo sin hardware. |
| **Protocolo TRT** | Formato binario de comunicación CLI ↔ placa. |
| **Trama** (*frame*) | Un mensaje del protocolo (cabecera + payload + CRC16). |
| **Transporte** | Canal que mueve bytes (serie/USB CDC; futuro CAN, TCP). Interfaz `ITransport`. |
| **Capacidad** (*capability*) | Función que una placa declara soportar (GPIO, PWM, LCD…). |
| **Servicio** | Tarea periódica del firmware (encoder, botón, LED ring, comunicación…). |
| **Evento** | Mensaje placa → host sobre algo ocurrido (ej. giro de encoder). |
| **Comando** | Petición host → placa. |
| **BoardContext** | Objeto del firmware que reúne info de placa, capacidades, módulos, transporte e interfaces. |
| **HWID** | Código de 4 bits en el conector B2B con el que la placa base identifica qué placa hija está conectada. |
| **B2B** | *Board-to-board*, conector entre placa base y placa hija. |
