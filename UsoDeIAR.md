# Informe Técnico: Simulación de Selección Natural con SPADE

## 1. Diseño Arquitectónico y Justificación Teórica
Para la resolución de este problema, se ha optado por una arquitectura híbrida centrada en un **Agente Gestor (*ManagerAgent*)**. A diferencia de una aproximación pura de Sistemas Multiagente (SMA) donde cada entidad sería un agente independiente con su propio ciclo de vida y canal de comunicación (XMPP), aquí se ha implementado un **Modelo Basado en Agentes (ABM)** gestionado centralmente.

### Decisión de Diseño: Objetos vs. Agentes
Si bien SPADE permite la creación de múltiples agentes, instanciar cada "criatura" (*Blob*) como un `spade.Agent` introduciría un *overhead* (sobrecarga) computacional significativo debido a:
1.  La gestión de múltiples conexiones XMPP concurrentes.
2.  La latencia en el paso de mensajes asíncronos para una simulación que requiere sincronización física estricta (tiempo real).

Por tanto, se definió a los *Blobs* como **objetos reactivos pasivos** (`class Blob`) que encapsulan su estado (energía, velocidad, posición) pero delegan su lógica de actualización al agente central. Esto cumple con el principio de **eficiencia computacional** sin sacrificar la complejidad emergente del sistema.

## 2. Implementación del Agente Gestor (ManagerAgent)
El núcleo de la simulación reside en el `ManagerAgent`, el cual implementa los siguientes componentes clave de la librería SPADE:

*   **Ciclo de Vida (`.setup`):** Inicializa el entorno, la población y el servidor web embebido. Se utiliza el mecanismo nativo `self.web.start()` para exponer el estado interno del agente sin bloquear el hilo principal de ejecución.
*   **Comportamiento Cíclico (`CyclicBehaviour`):** Se emplea este patrón de comportamiento para emular el *Game Loop* (Bucle de Juego). En cada iteración, el agente:
    1.  Percibe el entorno.
    2.  Actualiza la física y lógica de los objetos *Blob*.
    3.  Gestiona las transiciones de estado (Forrajeo $\rightarrow$ Retorno $\rightarrow$ Seguridad).
    4.  Aplica las reglas evolutivas al finalizar el ciclo "diario".
*   **Interoperabilidad Web:** El agente actúa como un servidor de estado, exponiendo endpoints REST (GET/POST) que desacoplan la lógica de simulación de la capa de visualización (Frontend en Canvas HTML5).

## 3. Lógica Evolutiva y Algorítmica
La simulación implementa un algoritmo genético simplificado basado en la presión selectiva del entorno:
*   **Función de Aptitud (*Fitness Function*):** Determinada por la capacidad de obtener energía.
    *   $E < 1$: El individuo es eliminado (Selección negativa).
    *   $E \ge 2$: El individuo se reproduce (Selección positiva).
*   **Herencia y Mutación:** Los descendientes heredan el atributo `speed` (velocidad) de sus progenitores, aplicándose una mutación aleatoria $\Delta$ (ruido gaussiano) que permite la variabilidad genética necesaria para que emerja la optimización del comportamiento a lo largo de las generaciones.

---

## 4. Declaración de Uso de Inteligencia Artificial

De conformidad con las instrucciones de la evaluación, se declara el uso de herramientas de Inteligencia Artificial Generativa (LLM) durante el proceso de desarrollo. A continuación se detalla el alcance, los *prompts* y la validación técnica realizada.

**Herramienta utilizada:** Gemini 3.0 Pro Preview

### Propósito y Metodología
La IA se utilizó como herramienta de apoyo para la **generación de código repetitivo (*boilerplate*)** y la estructuración de la interfaz gráfica, permitiendo enfocar el esfuerzo cognitivo en la lógica de los agentes y las reglas de simulación.

### Prompts Empleados (Resumen)
1.  **Infraestructura SPADE:** *"Generar un esqueleto de agente en SPADE que integre un servidor web nativo para servir archivos estáticos y endpoints JSON, evitando el uso de frameworks externos como Flask para minimizar conflictos de dependencias."*
2.  **Visualización:** *"Crear un script HTML5 + Canvas que consuma un endpoint JSON y renderice partículas en 2D, interpolando colores (Azul a Rojo) basado en un valor numérico de velocidad."*
3.  **Lógica de Negocio:** *"Adaptar la lógica del video 'Simulating Natural Selection' de Primer a una estructura de Clases Python, definiendo métodos de movimiento vectorial y reglas de consumo de energía."*

### Validación y Adaptación Humana
El código generado por la IA fue sometido a una revisión técnica exhaustiva y modificado significativamente por el estudiante para cumplir con los requisitos específicos de la PC3:
*   **Refactorización:** Se unificó la arquitectura dispersa propuesta por la IA en un único *Notebook* coherente.
*   **Corrección de Errores:** Se solucionaron problemas de concurrencia en el uso de `aiohttp` dentro de entornos Jupyter (implementación de `nest_asyncio`).
*   **Ajuste de la Rúbrica:** Se aseguró estrictamente que los *Blobs* **no** fueran implementados como agentes independientes, sino como objetos gestionados, demostrando comprensión de la diferencia entre entidad y agente en el diseño de software.