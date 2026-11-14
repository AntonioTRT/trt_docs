# BOM (lista de materiales)

> Estado: Pendiente. No hay BOM exportada en el repositorio.

## Fuente de la información

Cada componente de la placa base lleva su código **LCSC/JLCPCB** como propiedad en KiCad (`libraries/JLCPCB.kicad_sym`). La BOM debe generarse desde el esquemático (*Herramientas → Generar BOM*) y guardarse en `TRT_HW/hardware/carrier-board/` con fecha.

## Componentes principales conocidos

| Función | Componente |
|---|---|
| MCU | STM32G474RET6 (LQFP-64) |
| Regulador 3,3 V | AMS1117-3.3 (ver inconsistencia en [HWDD](HWDD.md)) |
| MOSFET de motor | AO3400A |
| Diodo flyback | 1N5819HW |
| Cristal | 8 MHz, SMD 3,2×2,5 mm |
| Protección ESD USB | USBLC6-2SC6 |
| Conector USB | USB-C SMD |

## Información pendiente

> TODO: Por hacer. Información no encontrada o no suficientemente documentada.

Falta: lista de materiales completa exportada desde KiCad (referencias, cantidades, números de parte).
