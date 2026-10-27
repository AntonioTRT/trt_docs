# Arquitectura del sistema

> Estado: Vigente.

## Vista general

```text
 PC (desarrollador)                         Placa
┌──────────┐   tramas TRT    ┌───────────┐   ┌─────────────────┐   ┌──────────┐
│ TRT-cli  │ ──────────────► │ Transporte│──►│    TRT-core     │──►│ Hardware │
│ (Python) │ ◄────────────── │ (serie/…) │◄──│ (firmware C++)  │◄──│ (TRT_HW) │
└──────────┘                 └───────────┘   └─────────────────┘   └──────────┘
```

- **TRT-cli** inicia las peticiones (comandos) y muestra los resultados.
- El **protocolo TRT** define el formato de las tramas. Es el único contrato entre PC y placa (ver [Protocolo TRT](Protocolo_TRT.md)).
- El **transporte** mueve bytes (hoy: puerto serie/USB CDC; futuro: CAN, TCP).
- **TRT-core** interpreta las tramas y responde, sin conocer la placa concreta: usa interfaces (GPIO, PWM, ADC…) que implementa cada placa.
- **TRT_HW** es el hardware físico sobre el que corre el firmware.

## Flujo general

**Comando (PC → placa):** el usuario escribe `trt board info 101` → el CLI crea una petición y la codifica en una trama → la envía por el transporte → el firmware la parsea, la despacha al servicio correspondiente → responde con `ACK`, `DATA` o `NACK` + error → el CLI decodifica y muestra el resultado.

**Evento (placa → PC):** un servicio del firmware (encoder, botón…) genera un evento → se encola y se envía al host sin que este lo pida. *Previsto; en V0.1 solo hay comandos.*

## Principios que atraviesan todo el sistema

1. **Autonomía de la placa**: el firmware sigue funcionando sin host y nunca se bloquea esperando mensajes.
2. **Un solo protocolo**: el mismo formato de trama y los mismos IDs para todas las placas.
3. **Independencia de la placa**: el CLI no tiene lógica específica por placa; el firmware declara **capacidades** y responde `ERROR_UNSUPPORTED_COMMAND` si algo no se soporta.
4. **Los comandos piden intención, no manejan hardware directamente**; la máquina de estados decide.

## Responsabilidades por módulo

| Módulo | Responsable de | No es responsable de |
|---|---|---|
| TRT-cli | Experiencia de usuario, descubrimiento, formateo de salida, codificar/decodificar tramas del lado PC | Lógica específica de una placa |
| TRT-core | Parser/encoder, despacho de comandos, capacidades, módulos, estado y servicios | Drivers concretos de una placa (los aporta cada familia en `boards/`) |
| TRT_HW | Diseño eléctrico, PCB, conectores, identificación de módulos | Protocolo y lógica de firmware |

Detalle en [SysDD](SysDD.md), [Arquitectura del firmware](../04_Software/Arquitectura_Firmware.md) y [Arquitectura del CLI](../04_Software/Arquitectura_CLI.md).
