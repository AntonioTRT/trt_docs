# Arquitectura del CLI (TRT-cli)

> Estado: Vigente. Detalle: `TRT-cli/docs/architecture.md`, `cli-reference.md`, `adding-commands.md`.

## Qué es

Aplicación Python (comando `trt`) que descubre y controla placas TRT desde el PC. Versión 1.0.0, validada con un Arduino Uno.

## Capas

```text
Comandos (Typer/Rich) → Servicios → ProtocolClient → Transporte → Placa
```

| Carpeta (`src/trt/`) | Responsabilidad |
|---|---|
| `cli.py`, `commands/` | Leer argumentos, llamar a servicios, mostrar resultados. Sin lógica de negocio. |
| `services/` | Lógica de aplicación: placas, descubrimiento, capacidades, LCD, LED, actualización. |
| `protocol/` | Peticiones/respuestas tipadas y clientes de protocolo (serie y mock). |
| `transport/` | `SerialTransport` (puertos COM), `MockTransport` (pruebas). |
| `repositories/` | Almacén del estado de placas. |
| `core/` | Modelos de dominio independientes del hardware. |

Dirección de dependencias: de arriba hacia abajo; las capas inferiores no conocen a las superiores.

## Comandos principales

| Comando | Para qué |
|---|---|
| `trt discover` | Enumera puertos serie y sondea cuáles tienen una placa TRT |
| `trt boards` | Lista placas conocidas |
| `trt board info <id>` / `capabilities <id>` | Identidad y capacidades de una placa (reales) |
| `trt listen` | Escucha pasivamente a todas las placas y muestra sus mensajes (`Board 101: HEARTBEAT`) |
| `trt board listen <id>` | Lo mismo, solo para una placa |
| `trt lcd`, `trt led` | Grupos de comandos de LCD y LED (usan cliente simulado por ahora) |
| `trt protocol`, `trt help`, `trt version`, `trt update` | Utilidades |

## Escucha pasiva y heartbeat

`trt listen` no envía nada: abre los puertos serie y muestra los eventos que las placas emiten solas. Cada placa TRT-core envía un `HEARTBEAT` aproximadamente cada segundo, así que tras cargar el firmware basta con ejecutar `trt listen` y ver `Board 101: HEARTBEAT` repetido para confirmar que el firmware, el transporte y el runtime funcionan. No sustituye a `trt discover` ni a `trt board info <id>`.

```text
commands/listen.py → services/listen_service.py (ListenService) → transport/frame_reader.py (FrameReader) → puerto serie
```

- `ListenService` lee cada puerto en su propio hilo y filtra por placa; `FrameReader` es un lector de tramas que se resincroniza tras ruido.
- No pasa por `ProtocolClient`. Las peticiones normales de `SerialTransport` descartan las tramas de evento mientras esperan la respuesta, para que un heartbeat no corrompa un descubrimiento.
- Otros eventos se muestran como `EVENT 0x<id> payload=<hex>`.
- Solo un programa puede usar un puerto serie a la vez: cierre otros monitores antes. El simulador no emite heartbeat.

## Cómo añadir un comando

1. Definir petición/respuesta en `protocol/models.py`.
2. Añadir la lógica en un servicio.
3. Crear el manejador en `commands/` y registrarlo en `cli.py`.

Guía completa: `TRT-cli/docs/adding-commands.md`.

## Pruebas

`pytest` (en `tests/`). El mock existe solo como soporte de pruebas.
