# Ta-Te-Ti · Agente con Red Neuronal (TensorFlow.js)

Juego de Ta-Te-Ti en el navegador donde un humano juega contra una red neuronal entrenada con TensorFlow.js. El modelo predice el mejor movimiento posible a partir del estado actual del tablero.

---

## Archivos del proyecto

```
📁 simple-ttt-model/
├── 📁 model/
│   ├── ttt_model.json        ← Arquitectura de la red neuronal
│   └── ttt_model.weights.bin ← Pesos entrenados del modelo
├── index.html                ← Juego principal
├── style.css                 ← Estilos separados del HTML
├── ttt_states.jpg            ← Imagen de referencia de estados del tablero
└── README.md                 ← Esta documentación
```

---

## Cómo ejecutar

El proyecto se ejecuta a través de un servidor local. No se puede abrir el HTML con doble-click porque el navegador bloquea la carga de archivos externos (como el modelo) por seguridad.

**Método usado — XAMPP:**
1. Instalá [XAMPP](https://www.apachefriends.org/)
2. Copiá la carpeta `simple-ttt-model` dentro de `C:\xampp\htdocs\`
3. Iniciá el servidor Apache desde el panel de control de XAMPP
4. Abrí en el navegador: `http://localhost/simple-ttt-model/index.html`

---

## Cómo se juega

- Vos jugás como **X**, la máquina juega como **O**
- Hacé click en cualquier casilla vacía para realizar tu movimiento
- La IA responde automáticamente después de 600ms
- El marcador acumula victorias, derrotas y empates durante la sesión

**Botones disponibles:**
- `Nueva partida` → reinicia el tablero manteniendo el marcador
- `IA empieza primero` → la máquina hace el primer movimiento
- `Ver probs` → muestra un panel con las probabilidades crudas que asignó la red a cada casilla en el último turno de la IA (útil para entender qué estaba "pensando")

**Dificultad:**
- `Red Neuronal` → usa el modelo entrenado para elegir la mejor jugada
- `Aleatoria` → la IA elige al azar (para comparar)

---

## La red neuronal

### Arquitectura

El modelo es una red neuronal densa secuencial con tres capas:

```
Entrada: 9 neuronas  (una por casilla del tablero)
         ↓
Capa 1:  64 neuronas · activación ReLU
         ↓
Capa 2:  64 neuronas · activación ReLU
         ↓
Salida:  9 neuronas  · activación Softmax
```

La salida es una distribución de probabilidad sobre las 9 casillas del tablero. La IA elige la casilla libre con mayor probabilidad.

### Representación del tablero

El tablero se codifica como un vector de 9 valores:

| Valor | Significado |
|-------|-------------|
| `1`   | Casilla ocupada por el jugador humano (X) |
| `-1`  | Casilla ocupada por la IA (O) |
| `0`   | Casilla vacía |

**Ejemplo:** el tablero vacío es `[0, 0, 0, 0, 0, 0, 0, 0, 0]`.

Las posiciones del vector corresponden al tablero de izquierda a derecha, de arriba a abajo:

```
[0] [1] [2]
[3] [4] [5]
[6] [7] [8]
```

### Cómo elige la IA su movimiento

1. El estado actual del tablero se convierte a un tensor de forma `[1, 9]`
2. Se pasa por la red → el modelo devuelve probabilidades para las 9 casillas
3. Se filtran las casillas ya ocupadas (no se pueden jugar)
4. Se elige la casilla libre con la probabilidad más alta

```javascript
// Pseudocódigo del proceso
const input = tf.tensor2d([board], [1, 9]);
const probs = model.predict(input).dataSync();  // [p0, p1, ..., p8]
const move  = empty_cells.reduce((best, i) => probs[i] > probs[best] ? i : best);
```

### Archivos del modelo

El modelo está guardado en formato TensorFlow.js:

- **`ttt_model.json`**: define la arquitectura (capas, activaciones, forma de entrada/salida)
- **`ttt_model.weights.bin`**: contiene los pesos numéricos aprendidos durante el entrenamiento (binario)

Ambos archivos son necesarios. Si falta el `.bin`, el juego igual corre pero con pesos aleatorios, lo que significa que la IA no sabe jugar.

---

## ¿La IA aprende con cada partida?

**No.** El modelo está congelado. Los pesos se cargan una sola vez al abrir la página y no cambian durante las partidas. Cada movimiento de la IA es simplemente una pasada hacia adelante (*forward pass*) por la red con el estado actual del tablero.

El aprendizaje ocurrió antes, durante el entrenamiento en Python/TensorFlow con datos de partidas de ejemplo. Una vez exportado a TF.js, el modelo solo hace inferencia.

Si la IA no bloquea alguna jugada tuya, significa que ese estado del tablero (o uno similar) no estuvo bien representado en los datos de entrenamiento. Las redes neuronales generalizan, pero pueden tener puntos ciegos.

---

## Tecnologías usadas

| Tecnología | Rol |
|-----------|-----|
| HTML5 + CSS3 + JavaScript | Interfaz del juego |
| TensorFlow.js v3.18 | Carga y ejecución del modelo en el navegador |
| Red neuronal densa (Sequential) | Agente que decide los movimientos de la IA |

---