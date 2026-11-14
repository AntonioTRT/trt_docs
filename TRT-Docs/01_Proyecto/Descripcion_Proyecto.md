# Descripción del proyecto

> Estado: Vigente (la parte de producto modular/magnético es visión, ver [Alcance](Alcance.md)).

## Qué es TRT

TRT es un framework para **controlar placas embebidas desde el PC con un mismo conjunto de comandos**, sin importar la placa (Arduino, STM32, ESP32, simulador…). Cubre tres frentes:

- **TRT-cli**: la herramienta que usa el desarrollador (`trt discover`, `trt board info 101`…).
- **TRT-core**: el runtime de firmware común que hace que cualquier placa "hable" TRT.
- **TRT_HW**: el hardware propio, basado en una placa base (*carrier*) con placas hijas intercambiables.

## Para qué sirve

- Descubrir e identificar placas conectadas por USB/serie de forma automática.
- Consultar su identidad, versión, estado y capacidades con un protocolo único.
- Reutilizar la misma lógica de firmware en distintas familias de microcontroladores.
- Visión de producto: bloques de control USB modulares (encoders, LEDs, pantallas, relés…) que se acoplan entre sí y se personalizan sin configuración compleja.

## Hito actual

Primer hito validado de extremo a extremo (CLI 1.0.0, 2026-09-08): `trt board info 101` abre el puerto serie, envía tramas TRT a un Arduino Uno con TRT-core y muestra ID, versión de firmware y build ID. Ver [Roadmap](Roadmap.md).

## Dónde seguir

[Arquitectura del sistema](../02_Sistema/Arquitectura_Sistema.md) · [Glosario](../02_Sistema/Glosario.md)
