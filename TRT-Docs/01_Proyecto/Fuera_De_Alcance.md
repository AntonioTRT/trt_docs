# Fuera de alcance

> Estado: Vigente.

Para mantener el foco, **no** se contempla por ahora:

- **Control de periféricos por protocolo**: V0.1 solo descubre y reporta (identidad, estado, capacidades). No hay comandos de GPIO, PWM, LCD, etc. en el firmware.
- **Placa como expansor remoto de GPIO**: la placa es autónoma; el host no gobierna cada pin.
- **Transportes distintos del serie/USB CDC** en producción (UART, CAN FD y TCP/IP son futuros).
- **Validación CRC16**: el campo existe en la trama pero el algoritmo no está definido ni verificado.
- **Configuración remota** (`CONFIG_GET/SET/SAVE/LOAD/RESET`) y niveles de depuración `-d` a `-dddd`: están especificados pero no implementados.
- **Actualización de firmware por protocolo** (estado `FIRMWARE_UPDATE`): solo planificado.
- **Soporte AVR completo** (Arduino Uno) del runtime general: requiere un subconjunto de memoria estática. Ver [Riesgos](../07_Gestion/Riesgos.md).
- **Inalámbrico, batería integrada** y plataformas no soportadas por el hardware actual (visión de producto).
