# TRT_HW: diseño de hardware

> Estado: Vigente. Fuente: `TRT_HW/README.md` y `TRT_HW/hardware/carrier-board/README.md` (especificación detallada, pinout completo y guía de layout).

## Qué es

Plataforma de hardware abierta con arquitectura **placa base + placas hijas**, pensada para audio digital, DSP y control. Todo está en KiCad 8 y se fabrica/ensambla en JLCPCB.

## Placa base (*carrier board*)

| Bloque | Implementación |
|---|---|
| MCU | STM32G474RET6 (LQFP-64, Cortex-M4F 170 MHz, FMAC y CORDIC) |
| Alimentación | USB-C 5 V → LDO 3,3 V (3,3 V analógico filtrado con ferrita) |
| Reloj | Cristal HSE 8 MHz |
| Depuración | SWD por Tag-Connect (SWDIO, SWCLK, SWO, NRST) |
| Buses hacia placas hijas | I2C1 (pull-ups conmutables por PC15), SPI1, GPIO/ADC/DAC |
| Identificación | 4 bits HWID (PC0–PC3), ver [HW_ID](HW_ID.md) |
| Extras | Driver PWM low-side para motor (MOSFET AO3400A), LED de alimentación y LED de usuario |

## Conector B2B (J2, 2x8)

Señales disponibles para la placa hija: SPI (CS, MOSI, MISO, SCK), I2C (SDA, SCL), 3 GPIO (`TRT_GPIO_1..3`), HWID_0..3, +5 V, +3,3 V y GND. La tabla pin a pin está en el README de la carrier board.

## Reglas que afectan al firmware

- **PB8 = SCL = BOOT0**: hay que configurar los option bytes (`nBOOT0_SEL=0`, `nBOOT0=1`) o la placa arranca en el bootloader DFU.
- **PC15 (`I2C_PU_EN`)**: nivel alto = pull-ups activos; alta impedancia = apagados. **Nunca** salida en bajo.

## Inconsistencias a resolver

`TRT_HW/README.md` menciona LDO AP2112K y bus B2B de 20 pines; el README de la carrier board describe AMS1117-3.3 y J2 de 16 pines. Confirmar con los esquemáticos cuál es el diseño vigente.

## Documentos relacionados

[Arquitectura de módulos](Arquitectura_Modulos.md) · [HW_ID](HW_ID.md) · [Esquemáticos](Esquematicos.md) · [PCB](PCB.md) · [BOM](BOM.md)
