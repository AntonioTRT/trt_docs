# System Design Description (SysDD) - Plataforma TRT

## 1. Visión General de la Plataforma
La plataforma **TRT** es un ecosistema modular de desarrollo y control embebido diseñado para estandarizar la interacción entre herramientas de software de línea de comandos, núcleos de procesamiento central y arquitecturas de hardware heterogéneas. 

El propósito de este documento es servir como **punto de entrada** para desarrolladores, ingenieros y colaboradores, permitiendo comprender rápidamente la arquitectura general y el funcionamiento del sistema sin necesidad de revisar todo el código fuente[cite: 1].

---

## 2. Componentes Principales

El ecosistema TRT se divide en tres pilares fundamentales:

### 2.1 TRT-CLI (Interfaz de Línea de Comandos)
Es la herramienta de control orientada al desarrollador para interactuar con las placas y el entorno de desarrollo.
* **Comunicación con las placas:** Se encarga de enviar comandos, gestionar la transmisión de datos, compilar y flashear los firmwares directamente hacia los microcontroladores conectados.
* **Automatización:** Facilita tareas repetitivas de despliegue, monitoreo y diagnóstico desde la terminal.

### 2.2 TRT-Core (Núcleo del Sistema)
Es el motor lógico y el framework central que orquesta las operaciones de la plataforma.
* **Lógica de negocio:** Administra las abstracciones de software, el manejo de estados, las tareas en tiempo real y la gestión de eventos del sistema.
* **Estandarización:** Proporciona las APIs y bibliotecas comunes sobre las cuales operan los distintos módulos de software.

### 2.3 TRT-HW (Infraestructura de Hardware)
Define la arquitectura física y el diseño de circuitos de la plataforma, basada en un enfoque modular:
* **Carrier Boards (Placas Base):** Placas principales que proporcionan la alimentación, los buses de comunicación globales y los conectores estandarizados.
* **Daughter Boards (Placas Hija):** Módulos periféricos intercambiables que se acoplan a la *Carrier Board* para añadir funcionalidades específicas (sensores, actuadores, conectividad, etc.).

---

## 3. Arquitectura y Flujo de Comunicación

### 3.1 Arquitectura General
TRT adopta una arquitectura desacoplada donde el software de control (`TRT-CLI`) se comunica a través de interfaces serie/red con el núcleo (`TRT-Core`) ejecutándose sobre el dispositivo de hardware (`TRT-HW`). Esta separación permite escalar el software de forma independiente a los cambios de circuitería.

### 3.2 Flujo de Comunicación
1. El usuario ejecuta comandos o scripts mediante **TRT-CLI**.
2. **TRT-CLI** empaqueta y transmite los comandos hacia el microcontrolador objetivo.
3. El firmware impulsado por **TRT-Core** procesa las instrucciones, interactúa directamente con los pines y registros de la placa (**TRT-HW**) y devuelve las respuestas o estados al sistema host.

### 3.3 Mecanismo de Heartbeat (Latido)
Para garantizar la estabilidad y el monitoreo de la conexión, el sistema implementa un protocolo de **heartbeat**:
* Ambas partes (host y dispositivo) intercambian paquetes periódicos de sincronización.
* Si el *heartbeat* se interrumpe o deja de responder dentro del umbral de tiempo establecido, **TRT-Core** o la interfaz activan protocolos de seguridad o reconexión automática para evitar estados indefinidos en el hardware.

---

## 4. Plataformas Soportadas

TRT está diseñado con portabilidad en mente, ofreciendo soporte nativo o adaptaciones para arquitecturas de microcontroladores de altas prestaciones y uso común:
* **Arduino:** Ideal para prototipado rápido y compatibilidad con Shields tradicionales.
* **STM32:** Enfocado en aplicaciones industriales y de alto rendimiento basadas en ARM Cortex-M.
* **ESP32:** Utilizado ampliamente por su conectividad integrada Wi-Fi y Bluetooth, idóneo para aplicaciones IoT.

---

## 5. Estado Actual y Visión Futura

### 5.1 Estado Actual del Proyecto
* **Funcionalidades implementadas:** 
  * Estructuración inicial de comandos en `TRT-CLI`.
  * Desarrollo de las bases lógicas de control en `TRT-Core`.
  * Diseño preliminar de esquemáticos y distribución física en `TRT-HW` (Carrier y Daughter Boards).

### 5.2 Funcionalidades Planificadas y Visión Futura
* Optimización de los protocolos de transporte de datos y reducción de latencia en el *heartbeat*.
* Expansión del catálogo de *Daughter Boards* soportadas de manera nativa.
* Integración de herramientas de diagnóstico gráfico y automatización avanzada desde la CLI.