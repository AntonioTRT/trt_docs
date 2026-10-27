# Documentación TRT

**TRT** (*Tool Runtime Terminal*) es un framework para controlar hardware embebido desde un PC. Se compone de tres módulos:

| Módulo | Carpeta | Qué es |
|---|---|---|
| TRT-cli | `TRT-cli` | Herramienta de línea de comandos (Python) que corre en el PC. |
| TRT-core | `TRT-core` | Runtime de firmware (C++) que corre dentro de las placas. |
| TRT_HW | `TRT_HW` | Diseño de hardware (KiCad): placa base y placas hijas. |

Se comunican así: `TRT-cli  →  Protocolo TRT  →  TRT-core (firmware)  →  TRT_HW (hardware)`.

## Cómo leer esta documentación

Para entender el framework en 15 minutos, lee en este orden:

1. [Descripción del proyecto](mddocs/01_Proyecto/Descripcion_Proyecto.md)
2. [Arquitectura del sistema](mddocs/02_Sistema/Arquitectura_Sistema.md)
3. [Glosario](mddocs/02_Sistema/Glosario.md)
4. [Visión por módulos (SysDD)](mddocs/02_Sistema/SysDD.md)
5. El documento del módulo en el que vayas a trabajar (ver índice).

## Índice

| Carpeta | Contenido |
|---|---|
| [01_Proyecto](mddocs/01_Proyecto/) | Descripción, alcance, fuera de alcance, roadmap, nombre. |
| [02_Sistema](mddocs/02_Sistema/) | Arquitectura global, glosario, protocolo TRT, requisitos, visión por módulos. |
| [03_Hardware](mddocs/03_Hardware/) | TRT_HW: placa base, módulos, identificación, esquemáticos, PCB, BOM. |
| [04_Software](mddocs/04_Software/) | TRT-core (firmware), TRT-cli, entorno de desarrollo, bus I2C. |
| [05_Mecanica](mddocs/05_Mecanica/) | Concepto mecánico y conexión magnética (pendiente). |
| [06_Marca](mddocs/06_Marca/) | Identidad visual, logo, web (pendiente). |
| [07_Gestion](mddocs/07_Gestion/) | Reglas de documentación, decisiones, riesgos. |

## Estado de la documentación

Cada documento indica su estado al inicio: **Vigente** (basado en el código/diseño actual), **Parcial** o **Pendiente** (sin información en los repositorios; solo se deja la estructura). Los detalles técnicos finos viven en la carpeta `docs/` de cada repositorio y aquí solo se enlazan.
