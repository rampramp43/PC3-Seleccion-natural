# 📄 Informe Técnico: Simulación "Minecraft RL Hardcore"

## 1. Resumen General
Este código implementa una **simulación multi-agente** compleja basada en el entorno del videojuego *Minecraft*. Utiliza la librería **SPADE** para la gestión de agentes y **Q-Learning** (Aprendizaje por Refuerzo) para dotar de inteligencia a los personajes ("Steves"). El objetivo es que los agentes aprendan autónomamente a sobrevivir a ciclos de día/noche, gestionar recursos, construir refugios y escapar de amenazas (Zombies).

## 2. Arquitectura del Sistema

### A. Tecnologías Clave
*   **Python Asyncio:** Gestión de concurrencia para el bucle del juego y servidores web.
*   **SPADE:** Framework para la creación de agentes inteligentes (BDI/XMPP).
*   **HTML5/Canvas + JS:** Interfaz gráfica generada dinámicamente (`static/index.html`) para visualizar la simulación en tiempo real.

### B. Entidades Principales
1.  **Steves (Agentes RL):** Agentes autónomos con salud, energía e inventario. Toman decisiones basadas en una tabla Q.
2.  **Zombies (NPCs):** Entidades hostiles con lógica determinista (perseguir al más cercano). Aumentan de nivel (tier) si matan.
3.  **ManagerAgent:** El "Dios" de la simulación. Controla el tiempo, spawnea recursos/enemigos, gestiona la evolución genética y sirve los datos a la web.
4.  **Objetos Pasivos:** Recursos (Comida, Madera, Piedra) y Casas (Refugios construibles/mejorables).

---

## 3. El Cerebro: Q-Learning (`QBrain`)

El núcleo de la inteligencia reside en la clase `QBrain`. Los agentes no están programados con reglas fijas (ej. "si tienes hambre, come"), sino que aprenden qué acción tomar según su estado para maximizar una recompensa.

### 🧠 Definición de Estado (State)
El agente observa el mundo y simplifica su realidad en una cadena de texto (clave del diccionario Q):
*   **Tiempo:** `DAY` o `NIGHT`.
*   **Peligro:** `DANGER` (zombie cerca) o `SAFE`.
*   **Estado Físico:** `DYING` (crítico), `TIRED` (poca energía) u `OK`.
*   **Inventario:** `HAS_FOOD`, `HAS_MATS` (materiales) o `POOR`.
*   **Ubicación:** `INSIDE` (en casa) u `OUTSIDE`.

### ⚡ Acciones Disponibles
`IDLE`, `EAT`, `SLEEP`, `GATHER_FOOD`, `GATHER_MATS`, `BUILD`, `REPAIR`, `FLEE`, `ENTER_HOUSE`.

### 💎 Sistema de Recompensas (Rewards)
El aprendizaje se refuerza mediante premios y castigos numéricos:
*   **+80:** Construir una casa (gran incentivo).
*   **+25:** Dormir en casa de noche (recuperación segura).
*   **+20:** Entrar a una casa o curarse comiendo.
*   **+5:** Sobrevivir la noche o recolectar recursos.
*   **-100:** Morir o recibir daño.
*   **-50:** Quedarse sin energía.

---

## 4. Dinámicas de la Simulación

### Ciclo de Vida y Evolución (Algoritmo Genético)
La simulación ocurre por **Generaciones**:
1.  Si un agente muere, queda eliminado.
2.  Al finalizar un ciclo (día/noche), se seleccionan los supervivientes con mejor desempeño.
3.  **Herencia:** Los nuevos agentes de la siguiente generación heredan la `Q-Table` (el cerebro aprendido) de los mejores supervivientes, acelerando el aprendizaje colectivo.
4.  Si todos mueren, se reinicia la simulación ("Game Over").

### Interfaz Gráfica (GUI)
El código genera un archivo `index.html` que se conecta vía `fetch` al servidor Python.
*   **Visualización:** Muestra agentes (azul), zombies (verde), recursos y casas. Las barras de vida y acciones se dibujan sobre los personajes.
*   **Panel de Control:** Muestra estadísticas en tiempo real (Día, Generación, Puntuación Promedio) y permite controlar la velocidad de la simulación.

---

## 5. Flujo de Ejecución del Código

1.  **Inicialización:** `ManagerAgent` levanta el servidor web en el puerto 10000 y crea la población inicial (Gen 0).
2.  **Bucle (WorldLoop):**
    *   Verifica si es día o noche.
    *   Calcula lógica de movimiento de Zombies.
    *   Solicita a cada Steve que perciba y actúe (`perceive_and_act`).
    *   Si es cambio de día, evalúa supervivientes y crea la nueva generación.
3.  **Agente (Steve):**
    *   Consulta su `QBrain` con su estado actual.
    *   Ejecuta la acción (moverse, interactuar, etc.).
    *   Recibe recompensa inmediata.
    *   Actualiza su tabla Q (`brain.learn`).

## 6. Conclusión
Este script es un ejemplo robusto de **Aprendizaje por Refuerzo aplicado a supervivencia**. Combina la toma de decisiones individual (micro) con un sistema evolutivo (macro), permitiendo observar cómo comportamientos complejos (como esconderse en casas por la noche) emergen de reglas simples y recompensas.

![Vista de la Simulación](Picture.png)