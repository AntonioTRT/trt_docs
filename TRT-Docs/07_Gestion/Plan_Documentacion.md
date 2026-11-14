# Plan de documentación

> Estado: Vigente.

## Reglas

1. `trt_docs` está en **español** y es la puerta de entrada (onboarding y mantenimiento). Los detalles técnicos finos viven en `docs/` de cada repositorio (inglés) y aquí solo se **enlazan**.
2. Cada tema tiene **un único documento**; los demás lo enlazan en vez de repetirlo.
3. Cada documento empieza con su **Estado**: Vigente, Parcial o Pendiente.
4. Usar los términos del [Glosario](../02_Sistema/Glosario.md).
5. Archivos en Markdown; nombres con la convención de carpeta numerada y `Nombre_Con_Guion_Bajo.md`.
6. Explicar qué es, para qué sirve y con quién interactúa; evitar detalle de implementación.

## Cuándo actualizar

| Cambio | Documento a revisar |
|---|---|
| Nuevo comando o ID de protocolo | [Protocolo TRT](../02_Sistema/Protocolo_TRT.md) |
| Nueva placa o familia soportada | [Arquitectura del firmware](../04_Software/Arquitectura_Firmware.md), [Roadmap](../01_Proyecto/Roadmap.md) |
| Nuevo comando de CLI | [Arquitectura del CLI](../04_Software/Arquitectura_CLI.md) |
| Cambio de pines o conector | [HWDD](../03_Hardware/HWDD.md) |
| Decisión importante | [Registro de decisiones](Registro_Decisiones.md) |
