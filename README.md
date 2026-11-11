# Simulador Cuántico en Python

Un simulador de computación cuántica desarrollado desde cero utilizando **Python**, **NumPy** y **SciPy**. El objetivo del proyecto es implementar el modelo de circuitos cuánticos a bajo nivel (álgebra lineal pura: vectores de estado, matrices de densidad, productos de Kronecker y medición proyectiva) sin recurrir a frameworks de alto nivel como Qiskit o Cirq.

El proyecto forma parte de mi portfolio y aborda tanto la física matemática de la computación cuántica como los retos de ingeniería asociados a la explosión dimensional del espacio de Hilbert ($2^n$).

---

## Características principales

- **Soporte para estados puros y estados mezcla:**
  - `SistemaBasico`: trabaja con el vector de estado $|\psi\rangle \in \mathbb{C}^{2^n}$, ideal para circuitos deterministas o sin decoherencia.
  - `SistemaMezcla`: utiliza matrices de densidad $\rho = \sum p_i |\psi_i\rangle\langle\psi_i|$ de dimensión $2^n \times 2^n$, permitiendo simular ensambles estadísticos y calcular la **entropía de von Neumann** ($S = -\mathrm{Tr}(\rho \ln \rho)$).
- **Optimización con matrices dispersas (`scipy.sparse.csr_matrix`):**  
  Tanto los vectores como los operadores y matrices de densidad se gestionan en formato CSR para minimizar el uso de memoria RAM y acelerar los productos tensoriales.
- **Ruteo de puertas multiqubit no contiguas mediante SWAPs:**  
  Para aplicar puertas como CNOT o Toffoli sobre qubits que no están en posiciones contiguas (por ejemplo, control en qubit 0 y objetivo en qubit 3), el simulador calcula y aplica los intercambios adyacentes necesarios, ejecuta la operación en bloque y deshace los swaps en orden inverso.
- **Medición cuántica y colapso:**  
  Soporta medición qubit a qubit o de todo el registro, colapsando el vector de estado o actualizando la matriz de densidad mediante proyectores normalizados ($\rho' = \frac{P \rho P}{\mathrm{Tr}(P \rho P)}$). Permite medir en:
  - Base computacional $Z$ ($|0\rangle, |1\rangle$)
  - Base $X$ ($|+\rangle, |-\rangle$)
  - Base $Y$ ($|+i\rangle, |-i\rangle$)
- **Visualización en la Esfera de Bloch:**  
  Cálculo de coordenadas cartesianas $(x, y, z)$ y ángulos esféricos $(\theta, \phi)$ para estados de 1 qubit, con renderizado 3D en Matplotlib (`bloch.py`).
- **Algoritmos cuánticos reales implementados:**  
  Casos de prueba completos para Deutsch-Jozsa, Bernstein-Vazirani y el algoritmo de búsqueda de Grover.

---

## Estructura del proyecto

```text
.
├── sistema.py           # Clases núcleo: SistemaBasico y SistemaMezcla
├── puertas.py           # Definición de puertas lógicas unitarias (1, 2 y 3 qubits)
├── bases.py             # Definición de bases de medición (Z, X, Y)
├── bloch.py             # Renderizado 3D de la esfera de Bloch con Matplotlib
├── examples.py          # Demostración paso a paso de los algoritmos cuánticos
└── documentacion.html   # Documentación interactiva y desglose de costes computacionales
```

### Puertas implementadas (`puertas.py`)

- **1 Qubit:** Identidad ($I$), Pauli ($X, Y, Z$), Hadamard ($H$), Fase ($S$), $\sqrt{\text{NOT}}$.
- **2 Qubits:** $\text{SWAP}$, $\sqrt{\text{SWAP}}$, $\text{CNOT}$ (controlada por X), $\text{MCY}$, $\text{MCZ}$.
- **3 Qubits:** $\text{Toffoli}$ ($\text{CCNOT}$), $\text{Fredkin}$ ($\text{CSWAP}$), $\text{MCZ3}$.

---

## Instalación y Requisitos

Solo requiere Python 3.8+ y las librerías matemáticas estándar:

```bash
pip install numpy scipy matplotlib
```

---

## Ejemplos rápidos de uso

### 1. Crear un estado de Bell ($|\Phi^+\rangle$)

```python
from sistema import SistemaBasico
from puertas import GATE_HADAMARD, GATE_CNOT

# Inicializamos un sistema de 2 qubits en el estado |00>
circuito = SistemaBasico(numQbits=2)

# Ponemos el qubit 0 en superposición: (|00> + |10>) / sqrt(2)
circuito.aplicar_puerta(GATE_HADAMARD, [0])

# Aplicamos CNOT con control en 0 y target en 1: (|00> + |11>) / sqrt(2)
circuito.aplicar_puerta(GATE_CNOT, [0, 1])

# Mostramos el estado resultante y probabilidades
circuito.print_estado()
circuito.print_probabilidades()

# Medir el sistema (colapsará en 00 o 11 con 50% de probabilidad cada uno)
resultado = circuito.medir()
print(f"Resultado de la medición: {resultado}")
```

### 2. Estados mezcla y Entropía de von Neumann

```python
from sistema import SistemaMezcla

# Ensamble con 50% de probabilidad |0> y 50% |1> (mezcla máxima)
mezcla = SistemaMezcla(numQbits=1, lista=[
    [50, [1, 0]],
    [50, [0, 1]]
])

print("Matriz de densidad:")
mezcla.print_matriz_densidad()
print("Entropía de von Neumann:", mezcla.entropia_von_neuman())
```

### 3. Coordenadas en la Esfera de Bloch

```python
from sistema import SistemaBasico
from puertas import GATE_HADAMARD
import bloch as b

q = SistemaBasico(1)
q.aplicar_puerta(GATE_HADAMARD, [0])

theta, phi = q.angulos_bloch()
x, y, z = q.coords_bloch()
print(f"Coordenadas: ({x}, {y}, {z})")

# Renderizar en 3D
b.draw_bloch_sphere(theta, phi)
b.plt.show()
```

---

## Ejecutar los algoritmos cuánticos

El archivo `examples.py` incluye una suite interactiva con explicaciones por consola. Puedes lanzarlo directamente:

```bash
python examples.py
```

Incluye:
1. **Deutsch-Jozsa:** Determina si una función booleana evaluada por un oráculo es constante o balanceada en una sola consulta.
2. **Bernstein-Vazirani:** Averigua una cadena de bits oculta (ej. `10100`) realizando una única llamada al oráculo.
3. **Algoritmo de Grover:** Búsqueda en espacio desestructurado mediante oráculo de cambio de fase e inversión sobre la media (difusor).
4. **Comparativa de estados con SistemaMezcla:** Comprobación de invariancia de entropía tras transformaciones unitarias vs mediciones.

---

## Notas sobre diseño y complejidad

En el archivo `documentacion.html` se incluye un desglose completo de la complejidad teórica de cada operación:

- **Espacio:** Gracias a `scipy.sparse.csr_matrix`, un vector de estado disperso solo almacena las amplitudes no nulas, requiriendo $\Omega(1)$ a $\mathcal{O}(2^n)$ elementos en vez de reservar siempre $2^n$ o $4^n$ complejos de doble precisión en memoria.
- **Ruteo de qubits:** En lugar de computar matrices unitarias globales de tamaño $2^n \times 2^n$ reordenadas para índices arbitrarios, se optó por una red de puertas $\text{SWAP}$ adyacentes antes y después de la puerta objetivo. Esto mantiene las operaciones locales sencillas de construir mediante productos de Kronecker con la matriz identidad ($\text{kron}(I, \dots, U, \dots, I)$).

---

## Autor

**Javier Bautista**  
Proyecto desarrollado con fines didácticos y para portfolio personal.
