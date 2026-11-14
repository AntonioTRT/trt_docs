# Arquitectura de módulos de hardware

> Estado: Parcial. Solo existe la placa base; las placas hijas aún no están en el repositorio.

## Modelo

```text
┌──────────────┐   conector B2B (J2)   ┌──────────────┐
│  Placa base  │ ◄───────────────────► │  Placa hija  │
│ STM32G474    │  I2C · SPI · GPIO     │ (encoder, LCD│
│              │  HWID · 5 V · 3,3 V   │  LED, relé…) │
└──────────────┘                       └──────────────┘
```

- La **placa base** aloja el MCU, la alimentación USB-C y la depuración.
- Cada **placa hija** aporta una función (entrada, salida, pantalla…) y se identifica con el HWID.
- Las placas hijas comparten los buses I2C/SPI del conector.

## Correspondencia con el firmware

Cada placa hija se refleja en TRT-core como un *módulo de firmware* (`IModule`) con tipo, revisión, nombre y capacidades. Familias previstas: relé, LCD, NeoPixel/LED, DAC, E/S analógica, E/S digital (carpetas `TRT-core/trt-core/modules/`, vacías por ahora).

## Pendiente

Definir el catálogo de placas hijas, su consumo y la forma en que se encadenan (ver [Conexión magnética](../05_Mecanica/Conexion_Magnetica.md)).

## Información pendiente

> TODO: Por hacer. Información no encontrada o no suficientemente documentada.

Falta: catálogo de placas hijas, consumos y forma de encadenarlas.
