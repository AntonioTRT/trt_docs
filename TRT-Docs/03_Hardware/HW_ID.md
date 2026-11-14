# Identificación de hardware (HWID)

> Estado: Parcial. El hardware está definido; la tabla de códigos aún no existe.

## Cómo funciona

El conector B2B tiene 4 líneas, `HWID_0..HWID_3`, conectadas a **PC0–PC3** del MCU (entradas ADC/digitales). La placa hija fija un valor en esas líneas (puentes o divisores resistivos) y el firmware lo lee al arrancar para saber qué módulo está conectado.

- Lectura digital: 4 bits → hasta 16 códigos.
- Lectura analógica (ADC): permite más códigos mediante valores de resistencia distintos.

## Relación con el software

El código leído debería traducirse en un módulo registrado en `ModuleManager` de TRT-core con sus capacidades. Hoy esa asignación no está implementada.

## Pendiente

- Tabla código HWID → tipo de placa hija.
- Decidir si la lectura es digital o analógica.

## Información pendiente

> TODO: Por hacer. Información no encontrada o no suficientemente documentada.

Falta: tabla código HWID → tipo de placa hija y decisión entre lectura digital o analógica.
