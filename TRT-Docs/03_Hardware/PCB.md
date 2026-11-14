# PCB

> Estado: Parcial. Archivo: `TRT_HW/hardware/carrier-board/carrier-board.kicad_pcb`.

## Estado actual

- Huellas colocadas en rejilla y todas las redes asignadas.
- Planos de GND en capas superior e inferior.
- **El ruteo de pistas no está hecho.**
- Copias de seguridad en `carrier-board-backups/` (zip).

## Reglas de layout a respetar

Resumen del checklist del README de la carrier board:

1. Bucle de potencia del motor (J1, AO3400A, diodo) lo más compacto posible; pistas de 1,2–1,5 mm.
2. Condensadores de 100 nF pegados a cada pin VDD; ferrita y condensador de VDDA/VREF+ junto a los pines 18 y 19.
3. Cristal de 8 MHz con pistas cortas, simétricas y anillo de guarda a GND; nada rápido por debajo.
4. Par USB (PA11/PA12) a 90 Ω diferencial, con plano de masa continuo.

Número de capas y reglas de fabricación: **por definir** (se asume proceso estándar de JLCPCB).

## Información pendiente

> TODO: Por hacer. Información no encontrada o no suficientemente documentada.

Falta: número de capas, reglas de fabricación y ruteo completado.
