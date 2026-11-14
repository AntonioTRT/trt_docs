# Riesgos

> Estado: Vigente (riesgos detectados al revisar los repositorios).

| Riesgo | Impacto | Mitigación |
|---|---|---|
| El runtime general no compila en AVR (STL, memoria dinámica, 2 KB RAM) | Arduino Uno solo funciona con el runtime mínimo actual | Implementar la variante de memoria estática (`TRT_HAS_STL=0`) |
| CRC16 no se calcula ni valida | Tramas corruptas no se detectan | Definir el algoritmo y activarlo en parser y encoder en ambos lados |
| Ruteo del PCB sin hacer | Retrasa la fabricación | Completar siguiendo el checklist de layout |
| PB8 compartido con BOOT0 | La placa arranca en DFU si no se configuran los option bytes | Configurarlos en el primer arranque |
| Documentación de hardware contradictoria (LDO, B2B 20 vs 16 pines) | Errores en BOM/ensamble | Confirmar con los esquemáticos y unificar los README |
| Documentos del CLI con protocolo distinto (`future-protocol.md`, trama V1) | Confusión al implementar | Tratar [Protocolo TRT](../02_Sistema/Protocolo_TRT.md) como fuente vigente y marcar los otros como diseño |
| Integración STM32 y placas hijas sin implementar | Hardware propio sin firmware TRT | Seguir el plan de `stm32-trt-integration-plan.md` |
| Sin especificación del bus I2C entre módulos ni de HWID | Bloquea las placas hijas | Completar [Protocolo I2C](../04_Software/Protocolo_I2C.md) y [HW_ID](../03_Hardware/HW_ID.md) |
| Errores de certificados de PlatformIO en redes corporativas | Falla la carga de firmware | Usar `truststore` (ya integrado en `upload.py`) |
