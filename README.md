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

## Modelo Matemático

El modelo se basa en un sistema de ecuaciones diferenciales parciales (PDEs) acopladas que describen el transporte de iones(en específico protones,V4 Y V5.

### Conservación de Masa
Para las especies iónicas móviles (como $VO^{2+}$, $VO_{2}^{+}$, y $H^{+}$), la conservación de masa en el dominio de la membrana se describe como:

$$
\frac{\partial c_{i}^{m}}{\partial t} = -\nabla \cdot \vec{N}_{i}^{m}
$$

Donde:
- $c_{i}^{m}$: Concentración de la especie $i$ en la membrana.
- $\vec{N}_{i}^{m}$: Flujo molar de la especie.

Para especies aniónicas como el bisulfato ($HSO_{4}^{-}$), se asume la condición de electroneutralidad:

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

Para resolver el sistema acoplado, se utiliza el **Método de Volúmenes Finitos (FVM)** en un dominio 1D discretizado en $N$ nodos.

La ecuación de conservación discretizada para un volumen de control toma la forma:

$$
\frac{dc_{i}}{dt} \cdot \Delta x = N_{i, entra} - N_{i, sale}
$$

Los flujos en las caras de los volúmenes finitos se calculan utilizando promedios aritméticos para las concentraciones y diferencias finitas centrales para los gradientes entre nodos adyacentes:

$$
N_{i, cara} \approx -D_{eff} \frac{c_{i+1} - c_{i}}{\Delta x} - \frac{D_{eff} z_i F}{RT} \left( \frac{c_{i+1} + c_{i}}{2} \right) \frac{\phi_{i+1} - \phi_{i}}{\Delta x}
$$

El sistema resultante es un sistema de Ecuaciones Diferenciales Algebraicas (DAE), donde las concentraciones se resuelven diferencialmente y el potencial $\phi$ se resuelve algebraicamente para satisfacer la conservación de corriente.

---

## Condiciones de Borde

El dominio se define entre $x=0$ (interfaz Cátodo/Membrana) y $x=L$ (interfaz Membrana/Ánodo).

**1. Transporte de Masa ($H^+$):**
* En $x=0$ (Izquierda): Condición de Dirichlet basada en el equilibrio de Donnan con el tanque.
    $$c_{H^{+}} = c_{H^{+}}^{T} \cdot \exp\left(\frac{-z F \Delta \phi_{Donnan}}{RT}\right)$$
* En $x=L$ (Derecha): Condición de Neumann basada en la corriente aplicada (Ley de Faraday).
    $$-\vec{n} \cdot \vec{N}_{H^+} = \frac{j_{appl}}{F}$$

**2. Potencial Iónico ($\phi_e$):**
* En $x=0$: Se fija un potencial de referencia (Dirichlet).
    $$\phi_{e} = \phi_{ref}$$
* En $x=L$: Se establece el flujo de corriente total (Neumann).
    $$j_{total} = j_{appl}$$

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

### Ejecución
Instalar dependencias:
```bash
pip install -r requirements.txt


