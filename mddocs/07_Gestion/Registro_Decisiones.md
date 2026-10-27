# Registro de decisiones

> Estado: Vigente. Decisiones inferidas de la documentación de los repositorios; añadir fecha cuando se conozca.

| # | Decisión | Motivo |
|---|---|---|
| 1 | La placa es autónoma y el bucle principal no bloquea | Funcionar sin host y procesar entradas en tiempo real |
| 2 | Comandos piden intención, no manejan hardware | La máquina de estados mantiene el comportamiento determinista |
| 3 | Trama V0.1 congelada | Evitar ambigüedad entre CLI, simulador y placas |
| 4 | V0.1 sin comandos de periféricos | Validar primero descubrimiento e identidad |
| 5 | Capacidades declaradas por la placa; el CLI no filtra por placa | Un solo CLI para todas las placas |
| 6 | Orden de validación: simulador → Arduino → ESP32/STM32 | Reducir riesgo del protocolo antes del hardware |
| 7 | Un único código de TRT-core con flags de compilación (sin versión "lite") | Evitar bifurcar el runtime por la RAM de AVR |
| 8 | STM32 se compila con STM32CubeIDE y TRT-core entra como librería | CubeIDE es el entorno validado; no duplicar su build |
| 9 | `upload.py` genera un proyecto aislado en `.trt-build/` | No modificar los sketches originales |
| 10 | CLI en capas (comandos → servicios → protocolo → transporte) | Aislar la lógica de la UI y del transporte |
| 11 | Carrier board con bit PB8/BOOT0 forzado por option bytes | PB8 es SCL con pull-up y arrancaría en DFU |
