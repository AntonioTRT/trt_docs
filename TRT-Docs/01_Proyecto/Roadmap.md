# Roadmap

> Estado: Vigente (según `TRT-core/docs/firmware-v0.1-roadmap.md` y el changelog de TRT-cli).

## Orden de validación del protocolo V0.1

| Fase | Objetivo | Estado |
|---|---|---|
| 1. Núcleo | Runtime arranca, parsea tramas y responde a los 6 comandos obligatorios. | Hecho |
| 2. Simulador | Validar protocolo sin hardware (pruebas automáticas). | Hecho |
| 3. Arduino | Primera placa real por puerto serie. | Hecho (hito 2026-09-08, Arduino Uno) |
| 4. STM32 | Mismo perfil V0.1 sobre STM32, compilado con STM32CubeIDE. | En preparación |
| 5. ESP32 | Mismo perfil V0.1 sobre ESP32. | Pendiente |
| Después | RP2040, comandos de periféricos, CRC16, configuración, depuración. | Pendiente |

## Hardware (TRT_HW)

1. Esquemáticos de la placa base — hecho.
2. Ruteo de pistas del PCB — pendiente.
3. Fabricación y bring-up (JLCPCB) — pendiente.
4. Placas hijas — pendiente.

## Criterio de "hecho" para una placa nueva

Arranca, responde correctamente a los 6 comandos, devuelve `ERROR_UNSUPPORTED_COMMAND` al resto y sigue funcionando sin host conectado.
