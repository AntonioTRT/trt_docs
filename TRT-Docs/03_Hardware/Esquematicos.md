# Esquemáticos

> Estado: Vigente. Ubicación: `TRT_HW/hardware/carrier-board/`.

Proyecto KiCad jerárquico. Abrir `carrier-board.kicad_pro`.

| Archivo | Contenido |
|---|---|
| `carrier-board.kicad_sch` | Hoja raíz; instancia las sub-hojas |
| `sheets/power.kicad_sch` | USB-C, protección ESD, regulador 3,3 V |
| `sheets/mcu.kicad_sch` | STM32G474, desacople, cristal, NRST, BOOT0 |
| `sheets/connectors.kicad_sch` | Depuración SWD y conector B2B |
| `mcu_core`, `peripherals`, `power_usb`, `expansion` (`.kicad_sch`) | Hojas adicionales en la raíz; revisar si sustituyen o duplican a `sheets/` |

Librerías propias en `libraries/` (`JLCPCB.*`): símbolos, huellas y modelos 3D descargados de LCSC con `easyeda2kicad`; cada componente guarda su código LCSC. Para regenerar uno: `python -m easyeda2kicad --lcsc_id=C123456 --full --output=hardware/carrier-board/libraries/JLCPCB`.

Pinout y función de cada señal: ver [HWDD](HWDD.md).
