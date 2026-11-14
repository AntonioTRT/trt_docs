# Protocolo TRT V0.1

> Estado: Vigente. Especificación completa en `TRT-core/docs/protocol*.md` y `TRT-core/docs/commands-v0.1.md`; aquí solo el resumen.

Es el contrato entre **TRT-cli** y **TRT-core**. Es binario, independiente del transporte y **congelado** en V0.1.

## Trama

```text
SYNC | VERSION | FLAGS | BOARD_ID | SEQ_ID | COMMAND | LENGTH | PAYLOAD | CRC16
 2 B     1 B      1 B      2 B       2 B      2 B       2 B     variable   2 B
```

- `SYNC` = `0xAA55`, `VERSION` = `0x01`. Tamaño mínimo: 14 bytes.
- `BOARD_ID` identifica la placa (ej. `101`); `SEQ_ID` relaciona petición y respuesta.
- `CRC16`: el campo existe, pero hoy se emite como `0x0000` y no se valida.

## Comandos (PC → placa)

| ID | Comando | Para qué |
|---|---|---|
| 0x0001 | PING | ¿Estás vivo? |
| 0x0002 | FW_VERSION | Versión de firmware |
| 0x0003 | BOARD_VERSION | Versión/revisión de la placa |
| 0x0004 | INFO | Identidad de la placa |
| 0x0005 | STATUS | Estado actual |
| 0x0006 | CAPABILITIES | Qué soporta la placa |
| 0x0007 | BUILD_ID | Número de build (extensión) |

Una placa es "compatible TRT V0.1" si implementa los comandos 0x0001–0x0006.

## Heartbeat (evento)

Toda implementación de TRT-core emite periódicamente un **heartbeat**: una trama de evento no solicitada por el host.

| Campo | Valor |
|---|---|
| `FLAGS` | `0x02` (EVENT) |
| `COMMAND` | `0x9001` (HEARTBEAT) |
| `BOARD_ID` | La placa que lo emite |
| `SEQ_ID` | Contador propio de la placa (no responde a ninguna petición) |
| Payload | Vacío (trama de 14 bytes) |

- **Frecuencia:** una vez por segundo (~14 bytes/s); la primera, nada más arrancar. Es constante en tiempo de compilación.
- **Para qué sirve:** confirma que el firmware corre, que la carga (flash) fue correcta, que el transporte funciona y que el runtime TRT está vivo. Pensado para desarrollo y validación; es seguro dejarlo activo.
- **No sustituye al descubrimiento:** el heartbeat solo indica presencia. La identidad sigue siendo cosa de `FW_VERSION`, `BOARD_VERSION` (GET_ID), `CAPABILITIES` y `BUILD_ID`.
- **Independiente del transporte:** la misma trama sirve por serie, USB, CAN o TCP.
- **En el PC:** se ve con `trt listen` o `trt board listen <id>`. Las peticiones normales ignoran las tramas de evento mientras esperan su respuesta.

Los eventos usan su propio rango de IDs (`0x9000` en adelante), separado de comandos (`0x0001`–`0x0007`) y respuestas (`0x8001`–`0x8003`).

## Respuestas y errores

| Respuesta | ID | Significado |
|---|---|---|
| ACK | 0x8001 | Hecho, sin datos |
| NACK | 0x8002 | Rechazado; el payload lleva el error |
| DATA | 0x8003 | Hecho, con datos |

Errores: `UNKNOWN_COMMAND` (1), `UNSUPPORTED_COMMAND` (2), `INVALID_ARGUMENT` (3), `BUSY` (4), `TIMEOUT` (5), `INTERNAL` (6).

## Nota sobre documentos del CLI

`TRT-cli/docs/trt-protocol.md` es una especificación de diseño más amplia (V1) con comandos futuros, y `future-protocol.md` describe una trama anterior (`MAGIC "TR"`). **La fuente vigente de la trama implementada es la de TRT-core V0.1 descrita arriba.**
