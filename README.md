# Modelo 1D de Batería de Flujo Redox Híbrida de Vanadio (RHVFC)

Este repositorio contiene la implementación numérica de un modelo unidimensional para la simulación del transporte de especies y carga a través de la membrana de una Batería de Flujo Redox de Vanadio (VRFB) e híbrida de hidrógeno. El proyecto se centra en la conservación de masa, el potencial electroquímico y la discretización de las ecuaciones diferenciales originales presentes en la literatura revisada mediante Volúmenes Finitos.

## Tabla de Contenidos
1. [Introducción](#introducción)
2. [Modelo Matemático](#modelo-matemático)
   - [Conservación de Masa](#conservación-de-masa)
   - [Ecuación de Nernst-Planck](#ecuación-de-nernst-planck)
   - [Conservación de Carga](#conservación-de-carga)
3. [Discretización y Método Numérico](#discretización-y-método-numérico)
4. [Condiciones de Borde](#condiciones-de-borde)
5. [Implementación](#implementación)
6. [Referencias](#referencias)

---

## Introducción

La batería de flujo redox de vanadio (VRFB) es una tecnología emergente de almacenamiento de energía. Se utiliza principalmente para aplicaciones a gran escala, como el almacenamiento de energía renovable en picos de producción y su posterior liberación en momentos de baja producción (Un ejemplo puede ser momentos del día sin luz solar o viento). En estos sistemas, los electrolitos líquidos que contienen las especies activas se almacenan en tanques externos y se bombean a través de una pila de celdas electroquímicas donde ocurren las reacciones de reducción-oxidación (redox).Esto permite que su capacidad sea modificable de forma desacoplada ya sea colocando variables en serie para aumentar la potencia o colocando tanques externos de mayor tamaño para mejorar el almacenamiento, una ventaja si las comparamos con otras tecnologías con focos similares en el mercado como pueden ser las baterías ion de Litio.

### Principio de Funcionamiento
En una VRFB, la energía se almacena en dos electrolitos líquidos que contienen iones de vanadio en diferentes estados de oxidación ($VO^{2+}/VO_2^+$ en el catolito). Estos líquidos son bombeados desde tanques externos,(esto es el desacople que se mencionó anteriormente y es una ventaja frente a otro tipo de baterías) hacia una celda donde ocurre la reacción electroquímica.

Este proyecto modela específicamente el comportamiento de transporte a través de la membrana de intercambio iónico, un componente crítico que debe permitir el paso de protones ($H^+$) para cerrar el circuito eléctrico.

---

## Aplicación en Chile (IPCh)

La implementación de Baterías de Flujo Redox Híbridas (RHVFC) en Chile responde a la urgente necesidad de gestionar la alta penetración de energías renovables variables, como la solar y la eólica, que se proyecta superará el 30% de la generación global para 2040. Chile posee recursos excepcionales en el Desierto de Atacama y la Patagonia, pero la naturaleza intermitente de estas fuentes genera inestabilidad en la red si no se cuenta con sistemas de respaldo adecuados. En este escenario, la tecnología RHVFC se presenta como una solución superior a las baterías convencionales de litio para aplicaciones de red, debido a su capacidad única de desacoplar la energía almacenada de la potencia entregada, permitiendo escalar el tiempo de descarga simplemente aumentando el volumen de los tanques de electrolito y no el tamaño del reactor.

Esta tecnología puede ser la respuesta a los niveles récord de "vertimiento" de energía renovable reportados por el Coordinador Eléctrico Nacional, donde miles de GWh de energía limpia se pierden anualmente por falta de infraestructura para guardarla durante las horas peak de producción (El momento del día con más solo o viento por ejemplo). La batería de flujo modelada en este trabajo ataca directamente este problema mediante aplicaciones de gestión de energía y liberación en periodos de necesidad, las cuales requieren ciclos de carga y descarga profundos y de larga duración (horas a días) que las tecnologías convencionales no pueden ofrecer de manera costo-efectiva.

---

## Modelo Matemático

El modelo se basa en un sistema de ecuaciones diferenciales parciales (PDEs) acopladas que describen el transporte de iones(en específico protones, V4 Y V5).

### Conservación de Masa
Para las especies iónicas móviles (como $VO^{2+}$, $VO_{2}^{+}$, y $H^{+}$), la conservación de masa en el dominio de la membrana se describe como:

$$
\frac{\partial c_{i}^{m}}{\partial t} = -\nabla \cdot \vec{N}_{i}^{m}
$$

Donde:
- $c_{i}^{m}$: Concentración de la especie $i$ en la membrana.
- $\vec{N}_{i}^{m}$: Flujo molar de la especie.

Para el bisulfato ($HSO_{4}^{-}$), se asume la condición de electroneutralidad:

$$
z_{f}c_{f} + \sum_{i}z_{i}c_{i}^{m} = 0
$$

Donde $z_{f}c_{f}$ es la carga fija de la membrana.

### Ecuación de Nernst-Planck
El flujo de especies $\vec{N}_{i}^{m}$ considera difusión, migración y convección:

$$
\vec{N}_{i}^{m} = -D_{i}^{m}\nabla c_{i}^{m} - z_{i}u_{i}^{m}c_{i}^{m}F\nabla\phi_{l}^{m} + \vec{v}_{j}c_{i}^{m}
$$

Donde:
- $D_{i}^{m}$: Coeficiente de difusión.
- $\phi_{l}^{m}$: Potencial iónico.
- $u_{i}^{m}$: Movilidad iónica (relacionada con $D$ por Nernst-Einstein).

### Conservación de Carga
Dado que la membrana es un aislante electrónico, solo existe corriente iónica ($\vec{j}_{l}^{m}$):

$$
\nabla \cdot \vec{j}_{l}^{m} = 0
$$

La densidad de corriente se relaciona con los flujos mediante:

$$
\vec{j}_{l}^{m} = F \sum_{i} z_{i} \vec{N}_{i}^{m}
$$

---

## Discretización y Método Numérico

Para resolver el sistema acoplado, se utiliza el Método de Volúmenes Finitos (FVM) en un dominio 1D discretizado en $N$ nodos (sujeto a modificación si se requiere).

La ecuación de conservación discretizada para un volumen de control toma la forma:

$$
\frac{dc_{i}}{dt} \cdot \Delta x = N_{i, entra} - N_{i, sale}
$$

Los flujos en las caras de los volúmenes finitos se calculan utilizando promedios aritméticos para las concentraciones y diferencias finitas, de forma que se tiene la siguiente ecuación que es la que se termina implementando en el código:

$$
N_{i, cara} \approx -D_{eff} \frac{c_{i+1} - c_{i}}{\Delta x} - \frac{D_{eff} z_i F}{RT} \left( \frac{c_{i+1} + c_{i}}{2} \right) \frac{\phi_{i+1} - \phi_{i}}{\Delta x}
$$

El sistema resultante es un sistema de Ecuaciones Diferenciales Algebraicas (DAE), donde las concentraciones se resuelven diferencialmente y el potencial $\phi$ se resuelve algebraicamente para satisfacer la conservación de corriente mediante la herramienta solver_dae ejecutable en python.

---

## Condiciones de Borde

El dominio computacional se define entre $x=0$ (interfaz Cátodo/Membrana) y $x=L$ (interfaz Membrana/Ánodo).

### 1. Interfaz Izquierda ($x=0$)
En la interfaz con el electrolito (tanque), las concentraciones en la superficie de la membrana no son iguales a las del tanque.Se calculan considerando el Salto de Potencial, lo que genera una discontinuidad en la concentración y por lo tanto modifican las condiciones de concentración iniciales:

$$
c_{i}(x=0) = c_{i}^{bulk} \cdot \exp\left(\frac{-z_i F \Delta \phi_{Donnan}}{RT}\right)
$$

Donde $i$ representa las especies $H^+$, $VO^{2+}$ y $VO_{2}^{+}$.

### 2. Interfaz Derecha ($x=L$)
En la salida hacia el ánodo, se aplican condiciones diferentes según la especie:

* Protones ($H^+$): Se impone un flujo proporcional a la densidad de corriente aplicada ($j_{appl}$), asumiendo que son los principales portadores de carga.
  
$$
-\vec{n} \cdot \vec{N}_{H^+} = \frac{j_{appl}}{F}
$$

* Especies de Vanadio ($VO^{2+}, VO_{2}^{+}$): Se asume una condición de Flujo Nulo (Neumann), implicando que no hay *crossover* de vanadio hacia el ánodo en esta simplificación del modelo.
  
    $$-\vec{n} \cdot \vec{N}_{V} = 0$$

### 3. Potencial Iónico ($\phi_e$)
* En $x=0$: Se fija un potencial de referencia (Dirichlet), $\phi_{e} = 0$.
* En $x=L$: Se establece que la corriente iónica total debe igualar a la corriente aplicada (Neumann) .

---

## Implementación

El código está escrito en **Python** y utiliza las siguientes librerías:
- `numpy`: Para operaciones matriciales.
- `scipy`: Para solvers de optimización (`root`).
- `scipy_dae`: Para la integración temporal del sistema DAE.
- `matplotlib`: Para la visualización de resultados.

## Referencias

1.  **K. W. Knehr et al.** "A transient vanadium flow battery model incorporating vanadium crossover and water transport through the membrane". *Journal of The Electrochemical Society*, 159(9):A1446-A1459, 2012. 
2.  **Catalina A. Pino Muñoz**. "Mathematical Modelling of Vanadium-Based Redox Flow Batteries". PhD thesis, Imperial College London, September 2019. 
3. **Chen et al.** "Progress in electrical energy storage system: A critical review". *Progress in Natural Science*, 19(3):291-312, 2009.
4.  **Alotto et al.** "Redox flow batteries for the storage of renewable energy: A review". *Renewable and Sustainable Energy Reviews*, 29:325-335, 2014. 
5. **Ministerio de Energía, Gobierno de Chile.** (2022). *Ley de Almacenamiento y Electromovilidad (Ley N° 21.505)*. Biblioteca del Congreso Nacional de Chile.
6. **Coordinador Eléctrico Nacional.** (2024). *Estudio de Vertimiento de Energías Renovables en el Sistema Eléctrico Nacional*.


