# Bus I2C entre módulos

> Estado: Pendiente. No hay especificación en los repositorios.

## Lo que existe

- La placa base expone **I2C1** (SDA=PB9, SCL=PB8) en el conector B2B, con pull-ups de 2,2 kΩ conmutables por PC15. Ver [HWDD](../03_Hardware/HWDD.md).
- El protocolo de comunicación definido hoy es el [Protocolo TRT](../02_Sistema/Protocolo_TRT.md) entre PC y placa; **no** se aplica al bus I2C interno.

## Por definir

- Mapa de direcciones por tipo de placa hija.
- Velocidad del bus y formato de mensajes.
- Manejo de errores y colisiones.
- Relación con el HWID ([HW_ID](../03_Hardware/HW_ID.md)).

## Información pendiente

> TODO: Por hacer. Información no encontrada o no suficientemente documentada.

Falta: mapa de direcciones, velocidad, formato de mensajes y manejo de errores del bus I2C entre módulos.
