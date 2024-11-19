---
jupytext:
  formats: ipynb,md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.16.4
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

# Integration of Ordinary Differential Equations. III

+++

## Numerical Stability of Integrators

Numerical Stability in the context of Ordinary Differential Equation (ODE) solvers refers to the ability of a numerical method to control the growth of errors introduced during the iterative process of approximation.
A method is considered stable if the numerical errors do not amplify uncontrollably as computations proceed.
This concept is crucial for ensuring that the numerical solution remains meaningful and behaves similarly to the true solution, especially over long integration intervals.

+++

### Definition and Importance

A numerical method is stable for a given problem if the errors---whether from truncation or round-off---do not grow exponentially with each time step.
Stability ensures that the numerical solution does not diverge from the true solution due to the accumulation of numerical errors.
This is particularly important in long-term simulations where even small errors can accumulate to produce significant deviations from the true behavior of the system.

+++

### Stability vs. Accuracy

It's essential to distinguish between stability and accuracy:
* Stability: Pertains to the boundedness of errors over time.
  A stable method ensures that errors remain controlled and do not grow exponentially, preventing the solution from becoming meaningless.
* Accuracy: Refers to how closely the numerical solution approximates the exact solution.
  An accurate method minimizes the difference between the numerical and true solutions.

A method can be stable but not accurate, meaning it controls error growth but doesn't closely follow the exact solution.
Conversely, a method can be accurate but unstable, producing precise results initially but diverging over time due to uncontrolled error growth.
For reliable numerical solutions, both stability and accuracy are required.

+++

### Stability Regions for Explicit Methods

To analyze the stability of numerical integrators, we commonly use the linear test equation introduced in previous lectures:
\begin{align}
\frac{dx}{dt} = \lambda x
\end{align}
where $\lambda \in \mathbb{C}$.
The exact solution to this ODE is:
\begin{align}
x(t) = x_0 e^{\lambda t}.
\end{align}
For a numerical method to be stable when applied to this equation, it must ensure that the numerical errors do not grow exponentially.
This is evaluated using the concept of the **stability region**.

+++

### Forward Euler Method Stability

In [ODE I](ode1.md), we learned that the Forward Euler method is the simplest explicit numerical integrator for solving ODEs.
Its update formula for the linear test equation is derived as follows:

Starting with the Forward Euler update:
\begin{align}
x_{n+1} = x_n + \Delta t \cdot f(x_n, t_n)
\end{align}
For the linear test equation $f(x, t) = \lambda x$, this becomes:
\begin{align}
x_{n+1} = x_n + \Delta t \cdot \lambda x_n = (1 + \lambda \Delta t) x_n
\end{align}
Here, the amplification factor $R(z)$ is defined as:
\begin{align}
R(z) = 1 + z \quad \text{where} \quad z = \lambda \Delta t
\end{align}
The stability condition requires that:
\begin{align}
|R(z)| \leq 1
\end{align}
Substituting the amplification factor:
\begin{align}
|1 + z| \leq 1
\end{align}
This condition defines the stability region for the Forward Euler method.

+++

To better understand the stability regions, we can visualize them in the complex plane.
The stability region for Forward Euler is a circle centered at $(-1, 0)$ with a radius of 1.

```{code-cell} ipython3
import numpy as np
from matplotlib import pyplot as plt
import matplotlib.patches as mpatches
```

```{code-cell} ipython3
# Define the grid for the complex plane
Re = np.linspace(-3, 3, 601)
Im = np.linspace(-3, 3, 601)
Re, Im = np.meshgrid(Re, Im)
Z = Re + 1j * Im

# Forward Euler stability condition |1 + Z| <= 1
abs_R_fE = np.abs(1 + Z)

# Plotting
plt.figure(figsize=(8, 6))
plt.contourf(Re, Im, abs_R_fE, levels=[0, 1], colors=['red'])

plt.title('Stability Region for Forward Euler Method')
plt.xlabel(r'Re($\lambda \Delta t$)')
plt.ylabel(r'Im($\lambda \Delta t$)')
plt.gca().set_aspect('equal')

# Adding legend manually
import matplotlib.patches as mpatches
blue_patch = mpatches.Patch(color='red', label='Forward Euler')
plt.legend(handles=[blue_patch])
```

The stability region plot leads to:
* The Forward Euler method is conditionally stable, meaning it is only stable within a specific region of the complex plane.
* Specifically, it is stable for values of $\lambda \Delta t$ that lie within the circle centered at $(-1, 0)$ with a radius of 1.
* For real positive $\lambda$ (i.e., $\lambda > 0$), the Forward Euler method becomes **unconditionally unstable** because such values fall outside the stability region.
  This implies that even with small time steps, the method cannot control error growth for these cases.

+++

The implication is:
* When dealing with ODEs where $\lambda > 0$, especially in stiff equations, the Forward Euler method may lead to solutions that diverge from the true behavior due to uncontrolled error amplification.
* This limitation necessitates the use of more stable methods, such as implicit integrators, which will be discussed in subsequent sections.

+++

### Stability Analysis Using Simple Harmonic Oscillator

From the previous lecture [ODE I](ode1.md), we also solved the simple harmonic oscillator:
\begin{align}
\frac{d}{dt}\begin{bmatrix} \theta(t) \\ \Omega(t) \end{bmatrix} =
\begin{bmatrix} \Omega(t) \\ -\frac{g}{l} \theta(t) \end{bmatrix} =
\begin{bmatrix} 0 & 1 \\ -\frac{g}{l} & 0 \end{bmatrix}
\begin{bmatrix} \theta(t) \\ \Omega(t) \end{bmatrix}
\end{align}
using the forward Euler method:
\begin{align}
\begin{bmatrix} \theta_{n+1} \\ \Omega_{n+1} \end{bmatrix} =
\begin{bmatrix} \theta_{n} \\ \Omega_{n} \end{bmatrix} +
\begin{bmatrix} 0 & 1 \\ -\frac{g}{l} & 0 \end{bmatrix}
\begin{bmatrix} \theta_{n} \\ \Omega_{n} \end{bmatrix} =
\begin{bmatrix} 1 & \Delta t \\ -\frac{g}{l}\Delta t & 1 \end{bmatrix}
\begin{bmatrix} \theta_{n} \\ \Omega_{n} \end{bmatrix}
\end{align}
The "amplification factor" is no longer a scalar but a matrix.
The stability condition $|R| \le 1$ become
\begin{align}
\det \begin{bmatrix} 1 & \Delta t \\ -\frac{g}{l}\Delta t & 1 \end{bmatrix} \le 1.
\end{align}
Hence,
\begin{align}
\frac{g}{l}\Delta t^2 \le 0
\end{align}
which canno be satisfied.
The forward Euler method is therefore again unconditional unstable.

+++

### Stability Regions for RK2 and RK4 Methods

Beyond the Forward Euler method, higher-order explicit Runge-Kutta (RK) methods like RK2 and RK4 offer improved accuracy while maintaining conditional stability.
However, their stability regions differ from that of the Forward Euler method, allowing for larger regions of stability in the complex plane.
Understanding these stability regions is essential for selecting appropriate numerical methods based on the problem's characteristics.

* RK2 Stability Function:
  For the classical RK2 method (Heun’s method), the stability function is given by:
  \begin{align}
  R_{\text{RK2}}(z) = 1 + z + \frac{1}{2} z^2
  \end{align}

* RK4 Stability Function:
  For the classical RK4 method, the stability function is more complex:
  \begin{align}
  R_{\text{RK4}}(z) = 1 + z + \frac{1}{2} z^2 + \frac{1}{6} z^3 + \frac{1}{24} z^4
  \end{align}

These stability functions indicate how each method propagates errors over time steps, affecting the overall stability of the numerical solution.

```{code-cell} ipython3
def R_RK2(z):
    """Stability function for RK2"""
    return 1 + z + 0.5 * z**2

def R_RK4(z):
    """Stability function for RK4 (classical Runge-Kutta)"""
    return 1 + z + 0.5 * z**2 + (1/6) * z**3 + (1/24) * z**4

# Compute |R(z)| for each method
abs_R_RK2 = np.abs(R_RK2(Z))
abs_R_RK4 = np.abs(R_RK4(Z))

# Define a list of methods and their corresponding data
methods = {
    'Forward Euler':   abs_R_fE,
    'RK2 ':            abs_R_RK2,
    'RK4 (Classical)': abs_R_RK4,
}

plt.figure(figsize=(10, 8))

# Plot stability regions for each method
colors = ['red', 'green', 'blue']
for idx, (title, abs_R) in enumerate(methods.items()):
    # Contour where |R(z)| = 1
    plt.contour(Re, Im, abs_R, levels=[1], colors=colors[idx], linewidths=2, linestyles='-')
    # Fill the stability region
    plt.contourf(Re, Im, abs_R, levels=[0, 1], colors=[colors[idx]], alpha=0.1)

plt.title('Stability Regions for Forward Euler, RK2, and RK4 Methods')
plt.xlabel(r'Re($\lambda \Delta t$)')
plt.ylabel(r'Im($\lambda \Delta t$)')
plt.gca().set_aspect('equal')

# Adding legend manually
patches = []
for idx, title in enumerate(methods.keys()):
    patches.append(mpatches.Patch(color=colors[idx], label=title))
plt.legend(handles=patches)
```

The above plot suggests:
* RK2 (Green):
  * Stability Region: Larger than Forward Euler but still bounded.
  * Extended Stability: Allows for slightly larger time steps while maintaining stability.
* RK4 (Classical) (Blue):
  * Stability Region: Even larger than RK2, encompassing a more extensive area in the complex plane.
  * Greater Stability: Facilitates larger time steps compared to Forward Euler and RK2.

+++

Some key observations include:
* Order vs. Stability: Higher-order methods like RK2 and RK4 generally have larger stability regions compared to lower-order methods like Forward Euler.
  This allows them to take larger time steps while maintaining stability.
* Still Conditionally Stable: Despite their larger stability regions, RK2 and RK4 are still conditionally stable.
  They are not A-stable, meaning they cannot remain stable for all values of  $\lambda \Delta t$ in the left half of the complex plane.

+++

In terms of trade-offs:
* Forward Euler: Simple and easy to implement but has a very limited stability region.
* RK2 and RK4: More complex and computationally intensive due to additional stages but offer better stability and accuracy.

+++

In this section, we extended our analysis of stability regions to include higher-order explicit Runge-Kutta methods: RK2 and RK4 (classical Runge-Kutta).
Both methods exhibit larger stability regions compared to the Forward Euler method, allowing for greater flexibility in choosing time steps while maintaining stability.
However, they remain conditionally stable, meaning their stability depends on the specific characteristics of the ODE being solved and the chosen time step.

Understanding the stability regions of these methods is essential for selecting the appropriate numerical integrator based on the problem's nature.
While RK2 and RK4 offer improved stability and accuracy, they still require careful consideration of the time step to ensure that the numerical solution remains stable and accurate.

+++

### Implicit Methods

In the realm of numerical ODE solvers, methods are broadly classified into explicit and implicit integrators based on how they compute the next state of the system.

Explicit Methods:
* Definition: Compute the state of the system at the next time step solely based on information from the current and previous steps.
* Example: Forward Euler Method.
* Advantages:
  * Simplicity and ease of implementation.
  * Computational efficiency for non-stiff problems.
* Disadvantages:
  * Limited stability regions, making them unsuitable for stiff ODEs without very small time steps.
  * Potential for instability and error amplification in certain scenarios.

Implicit Methods:
* Definition: Compute the state of the system at the next time step based on both current and future information, often requiring the solution of equations that involve the unknown future state.
* Example: Backward Euler Method.
* Advantages:
  * Larger stability regions, making them suitable for stiff ODEs.
  * Enhanced stability allows for larger time steps without sacrificing solution quality.
* Disadvantages:
  * Increased computational complexity due to the need to solve nonlinear equations at each step.
  * More involved implementation, especially for complex or high-dimensional systems.

+++

The main reasons to use implicit methods include:
* Stiff ODEs: Systems with rapidly decaying components alongside slower dynamics.
  Explicit methods require prohibitively small time steps for stability, whereas implicit methods can handle larger steps efficiently.
* Enhanced Stability: Implicit methods can remain stable for a broader range of problems and time steps, making them indispensable for certain applications in physics, engineering, and other fields.

Understanding the distinction between implicit and explicit methods is fundamental for selecting the appropriate numerical integrator based on the characteristics of the ODE being solved.

+++

### Backward Euler Method

The Backward Euler method is one of the simplest implicit methods.
It offers enhanced stability properties compared to its explicit counterpart, making it suitable for stiff ODEs.

Given an ODE:
\begin{align}
\frac{dx}{dt} = f(x, t), \quad x(t_0) = x_0
\end{align}
The Backward Euler update formula is derived by evaluating the derivative at the next time step $t_{n+1} = t_n + \Delta t$:
\begin{align}
x_{n+1} = x_n + \Delta t \cdot f(x_{n+1}, t_{n+1})
\end{align}
Unlike explicit methods, $x_{n+1}$ appears on both sides of the equation, necessitating the solution of this equation at each time step.

+++

### Stability Analysis

Using the linear test equation $\frac{dx}{dt} = \lambda x$, where $\lambda \in \mathbb{C}$, the Backward Euler update becomes:
\begin{align}
x_{n+1} = x_n + \Delta t \cdot \lambda x_{n+1}
\end{align}
Solving for $x_{n+1}$:
\begin{align}
x_{n+1} = \frac{x_n}{1 - \lambda \Delta t}
\end{align}
The amplification factor $R(z)$ is therefore:
\begin{align}
R(z) = \frac{1}{1 - z} \quad \text{where} \quad z = \lambda \Delta t
\end{align}

The stability condition requires:
\begin{align}
|R(z)| = \left| \frac{1}{1 - z} \right| \leq 1
\end{align}
For the method to be stable, the above condition must hold.
Analyzing this:
* A-Stability: A numerical method is A-stable if it is stable for all $\lambda \Delta t$ with $\text{Re}(\lambda) \leq 0$.
  The Backward Euler method satisfies this condition, making it A-stable.
* Implications for Stiff ODEs: The A-stability of the Backward Euler method allows it to handle stiff ODEs effectively, enabling larger time steps without sacrificing stability.

```{code-cell} ipython3
def R_bE(z):
    """Stability function for backward Euler"""
    return 1 / (1 - z)

# Compute |R(z)| for each method
abs_R_bE = np.abs(R_bE(Z))

# Define a list of methods and their corresponding data
methods = {
    'Forward Euler':   abs_R_fE,
    'Backward Euler':  abs_R_bE,
    'RK2':             abs_R_RK2,
    'RK4 (Classical)': abs_R_RK4,
}

plt.figure(figsize=(10, 8))

# Plot stability regions for each method
colors = ['red', 'yellow', 'green', 'blue']
for idx, (title, abs_R) in enumerate(methods.items()):
    # Contour where |R(z)| = 1
    plt.contour(Re, Im, abs_R, levels=[1], colors=colors[idx], linewidths=2, linestyles='-')
    # Fill the stability region
    plt.contourf(Re, Im, abs_R, levels=[0, 1], colors=[colors[idx]], alpha=0.1)

plt.title('Stability Regions for Forward Euler, RK2, and RK4 Methods')
plt.xlabel(r'Re($\lambda \Delta t$)')
plt.ylabel(r'Im($\lambda \Delta t$)')
plt.gca().set_aspect('equal')

# Adding legend manually
patches = []
for idx, title in enumerate(methods.keys()):
    patches.append(mpatches.Patch(color=colors[idx], label=title))
plt.legend(handles=patches)
```

Implementing the Backward Euler method involves solving the implicit equation at each time step.
The approach varies depending on whether the ODE is linear or nonlinear.

* Linear ODEs: If $f(x, t)$ is linear in $x$, the implicit equation can often be solved directly without iterative methods.
  For example, for $dx/dt = \lambda x$:
  \begin{align}
  x_{n+1} = (1 - \lambda \Delta t)^{-1} x_n.
  \end{align}
  Using matrix inverse, the above formulation even works for system of linear ODEs.

* Nonlinear ODEs: If $f(x, t)$ is nonlinear, solving for $x_{n+1}$ typically requires iterative methods such as the [Newton-Raphson method](opt.md).
  Python's fsolve from the scipy.optimize library can be used for this purpose.

+++

### Stiff ODEs

Let's implement the Backward Euler method for a simple linear ODE and compare it with the Forward Euler method.

Consider the ODE:
\begin{align}
\frac{dx}{dt} = -1000x + 3000 - 2000e^{-t}, \quad x(0) = 0
\end{align}
The exact solution is:
\begin{align}
x(t) = 3 - 0.998 e^{-1000t} - 2.002 e^{-t}
\end{align}
This ODE is stiff because it contains terms with vastly different decay rates ($e^{-1000t}$ vs. $e^{-t}$).

```{code-cell} ipython3
from scipy.optimize import fsolve

# Define the exact solution for comparison
def exact_solution(t):
    return 3 - 0.998 * np.exp(-1000 * t) - 2.002 * np.exp(-t)

# Define the stiff ODE
def stiff_ode(x, t):
    return -1000 * x + 3000 - 2000 * np.exp(-t)

# Forward Euler Method (for comparison)
def forward_euler(f, x0, t0, tf, dt):
    T = np.arange(t0, tf + dt, dt)
    X = [x0]
    
    for i in range(1, len(T)):
        t = T[i-1]
        X.append(X[i-1] + dt * f(X[i-1], t))
    
    return T, X

# Backward Euler Method
def backward_euler(f, x0, t0, tf, dt):
    T = np.arange(t0, tf + dt, dt)
    X = [x0]
    
    for i in range(1, len(T)):
        t = T[i]
        # Define the implicit equation: x_next = x_current + dt * f(x_next, t_next)
        func = lambda x: x - X[i-1] - dt * f(x, t)
        # Initial guess: Forward Euler estimate
        x0 = X[i-1] + dt * f(X[i-1], t)
        # Solve for x_next using Newton-Raphson (fsolve)
        x = fsolve(func, x0)[0]
        X.append(x)
    
    return T, X

# Parameters
x0 = 0
t0 = 0
tf = 0.01 # Short time to observe stiffness effects
dt = 0.002

# Solve using Forward Euler
T_fE, X_fE = forward_euler(stiff_ode, x0, t0, tf, dt)

# Solve using Backward Euler
T_bE, X_bE = backward_euler(stiff_ode, x0, t0, tf, dt)

# Exact solution
T = np.linspace(t0, tf, 1000)
X = exact_solution(T)

# Plotting
plt.figure(figsize=(10, 6))
plt.plot(T,    X,    'k-',  label='Exact Solution')
plt.plot(T_fE, X_fE, 'r-o', label='Forward Euler')
plt.plot(T_bE, X_bE, 'b-o', label='Backward Euler')
plt.xlabel('Time t')
plt.ylabel('x(t)')
plt.legend()
plt.grid(True)
```