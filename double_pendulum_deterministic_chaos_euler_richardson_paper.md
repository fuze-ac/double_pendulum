# Double Pendulum Dynamics, Deterministic Chaos, Perpetual Motion, and the Euler–Richardson Algorithm

## Abstract

The double pendulum is one of the simplest mechanical systems capable of exhibiting deterministic chaos. Its motion is governed by deterministic equations, yet small differences in initial conditions can grow rapidly and make long-term prediction practically impossible. This paper examines the physical structure and equations of the double pendulum, the meaning of deterministic chaos, the thermodynamic limitations of perpetual-motion machines, and the use of the Euler–Richardson algorithm for numerical simulation. Particular attention is given to the distinction between conservation laws in the ideal physical model and numerical error introduced by the integration method.

## 1. Introduction

A double pendulum consists of one pendulum attached to the end of another. Although the system contains only two moving masses and two angular coordinates, its equations of motion are nonlinear and dynamically coupled.

At low energies, the motion may be regular and approximately periodic. At higher energies, the system may enter chaotic regimes in which two nearly identical initial states develop very different trajectories.

This makes the double pendulum useful for studying several important scientific concepts:

- nonlinear mechanics;
- deterministic chaos;
- sensitivity to initial conditions;
- numerical integration;
- conservation of mechanical energy;
- thermodynamic irreversibility;
- limits of long-term prediction.

The double pendulum also provides a useful test case for numerical algorithms because a correct simulation must handle strong coupling, rapidly changing accelerations, energy error, and chaotic amplification of numerical uncertainty.

## 2. The Double Pendulum

### 2.1 Idealized physical model

A standard ideal double pendulum consists of:

- a first point mass \(m_1\);
- a second point mass \(m_2\);
- two massless rigid rods of lengths \(l_1\) and \(l_2\);
- a fixed upper pivot;
- a uniform gravitational field with acceleration \(g\);
- no pivot friction;
- no air resistance;
- no deformation of the rods.

The configuration of the system is described by two generalized coordinates:

\[
\theta_1,\qquad \theta_2
\]

These angles are usually measured from the downward vertical.

Because two independent coordinates are required to describe the system, the double pendulum has two degrees of freedom.

### 2.2 Cartesian coordinates

The position of the first mass is

\[
x_1=l_1\sin\theta_1
\]

\[
y_1=-l_1\cos\theta_1
\]

The position of the second mass is

\[
x_2=l_1\sin\theta_1+l_2\sin\theta_2
\]

\[
y_2=-l_1\cos\theta_1-l_2\cos\theta_2
\]

The position of the second mass depends on both angles. Consequently, the motion of the two pendulum arms is coupled.

### 2.3 Kinetic energy

For point masses and massless rods, the kinetic energy is

\[
T=
\frac12(m_1+m_2)l_1^2\dot{\theta}_1^2
+\frac12m_2l_2^2\dot{\theta}_2^2
+m_2l_1l_2\dot{\theta}_1\dot{\theta}_2
\cos(\theta_1-\theta_2)
\]

The final term is a coupling term. It shows that the kinetic energy associated with one arm depends on the velocity of the other arm.

### 2.4 Potential energy

Taking the fixed pivot as a convenient reference, the gravitational potential energy can be written as

\[
V=-(m_1+m_2)gl_1\cos\theta_1
-m_2gl_2\cos\theta_2
\]

The total mechanical energy is

\[
E=T+V
\]

In the ideal physical model, \(E\) is constant.

### 2.5 Lagrangian formulation

The Lagrangian is

\[
L=T-V
\]

For each generalized coordinate \(\theta_i\), the Euler–Lagrange equation is

\[
\frac{d}{dt}
\left(
\frac{\partial L}{\partial \dot{\theta}_i}
\right)
-
\frac{\partial L}{\partial \theta_i}
=0
\]

Applying these equations to \(\theta_1\) and \(\theta_2\) produces two coupled nonlinear second-order differential equations.

### 2.6 Explicit equations of motion

Define

\[
\Delta=\theta_1-\theta_2
\]

A commonly used explicit form for the angular accelerations is

\[
\ddot{\theta}_1=
\frac{
-m_2l_1\dot{\theta}_1^2\sin\Delta\cos\Delta
+m_2g\sin\theta_2\cos\Delta
-m_2l_2\dot{\theta}_2^2\sin\Delta
-(m_1+m_2)g\sin\theta_1
}{
l_1\left(m_1+m_2-m_2\cos^2\Delta\right)
}
\]

and

\[
\ddot{\theta}_2=
\frac{
(m_1+m_2)
\left(
l_1\dot{\theta}_1^2\sin\Delta
-g\sin\theta_2
+g\sin\theta_1\cos\Delta
\right)
+m_2l_2\dot{\theta}_2^2\sin\Delta\cos\Delta
}{
l_2\left(m_1+m_2-m_2\cos^2\Delta\right)
}
\]

Equivalent equations may appear in different algebraic forms. A simulation must use a consistent definition of the angles, coordinates, potential energy, and acceleration equations.

### 2.7 Why the system is nonlinear

The equations contain nonlinear expressions such as

\[
\sin\theta_1,\qquad
\sin(\theta_1-\theta_2),\qquad
\cos(\theta_1-\theta_2)
\]

They also contain products involving angular velocities.

These terms prevent the system from being represented as a simple linear combination of coordinates and velocities.

For sufficiently small angles,

\[
\sin\theta\approx\theta
\]

and

\[
\cos(\theta_1-\theta_2)\approx1
\]

The system can then be approximated by linear equations and represented as a combination of normal modes. In this low-energy regime, its motion is generally regular.

### 2.8 State-space representation

Numerical solvers normally operate on first-order systems. Define

\[
\omega_1=\dot{\theta}_1,\qquad
\omega_2=\dot{\theta}_2
\]

and the state vector

\[
\mathbf y=
\begin{bmatrix}
\theta_1\\
\omega_1\\
\theta_2\\
\omega_2
\end{bmatrix}
\]

The derivative of the state is

\[
\dot{\mathbf y}=
\begin{bmatrix}
\omega_1\\
\alpha_1\\
\omega_2\\
\alpha_2
\end{bmatrix}
\]

where

\[
\alpha_1=\ddot{\theta}_1,\qquad
\alpha_2=\ddot{\theta}_2
\]

The simulation therefore solves an initial-value problem of the form

\[
\dot{\mathbf y}=f(t,\mathbf y),
\qquad
\mathbf y(t_0)=\mathbf y_0
\]

## 3. Deterministic Chaos

### 3.1 Determinism and predictability

A deterministic system follows fixed equations. If its equations, parameters, and exact initial conditions are known, its future state is uniquely determined.

Chaos does not mean that the system is random.

A chaotic double pendulum still follows deterministic equations. Its practical unpredictability results from the rapid amplification of tiny uncertainties in the initial state.

The correct distinction is:

> The exact future remains mathematically determined, but long-term practical prediction becomes unreliable because the initial state cannot be measured with unlimited precision.

### 3.2 Sensitive dependence on initial conditions

Consider two initial states

\[
\mathbf y_A(0)
\]

and

\[
\mathbf y_B(0)=\mathbf y_A(0)+\delta\mathbf y(0)
\]

where \(\delta\mathbf y(0)\) is very small.

In a chaotic regime, their separation may initially grow approximately as

\[
\|\delta\mathbf y(t)\|
\approx
\|\delta\mathbf y(0)\|e^{\lambda t}
\]

where \(\lambda\) is a Lyapunov exponent.

A positive largest Lyapunov exponent indicates exponential divergence in at least one phase-space direction.

This exponential model applies only while the separation is small. Because the motion is physically bounded by the available energy, the separation eventually saturates.

### 3.3 Lyapunov time

For a positive Lyapunov exponent, the characteristic Lyapunov time is

\[
\tau_L=\frac{1}{\lambda}
\]

If the initial uncertainty is \(\delta_0\) and the maximum acceptable prediction error is \(\delta_{\max}\), an approximate prediction horizon is

\[
t_{\mathrm{predict}}
\approx
\frac{1}{\lambda}
\ln\left(\frac{\delta_{\max}}{\delta_0}\right)
\]

This expression shows that improving measurement precision extends the prediction horizon only logarithmically.

### 3.4 Not every trajectory is chaotic

A double pendulum is capable of chaotic behaviour, but not every trajectory is chaotic.

Its behaviour depends on:

- total energy;
- initial angles;
- initial angular velocities;
- mass ratio;
- length ratio.

The system may exhibit:

- small regular oscillations;
- periodic motion;
- quasiperiodic motion;
- full rotations;
- chaotic motion;
- a mixture of regular and chaotic phase-space regions.

The scientifically accurate description is therefore:

> The double pendulum is a nonlinear mechanical system capable of deterministic chaos.

### 3.5 Conservative chaos

The ideal double pendulum is a conservative system. It can exhibit chaos while conserving total mechanical energy.

This differs from a dissipative chaotic system, in which energy is lost and trajectories may approach an attractor.

For an ideal conservative double pendulum:

- energy remains on a constant-energy surface;
- there is no dissipative strange attractor;
- chaotic behaviour occurs through stretching and folding of trajectories within the allowed phase-space region.

### 3.6 Numerical demonstration of chaos

A simulation can demonstrate sensitivity to initial conditions by running two systems with nearly identical starting states.

For example,

\[
\theta_{1,B}(0)=\theta_{1,A}(0)+\varepsilon
\]

while all other initial variables are identical.

A state-space separation may be defined as

\[
d(t)=
\sqrt{
\Delta\theta_1^2+
\Delta\omega_1^2+
\Delta\theta_2^2+
\Delta\omega_2^2
}
\]

Angular differences should be normalized because angles repeat every \(2\pi\).

Plotting

\[
\ln d(t)
\]

against time may reveal an approximately linear region. Its slope can be used to estimate the largest Lyapunov exponent.

However, visible divergence alone is not sufficient proof of chaos because numerical error can also separate trajectories. The solver must first be validated through convergence and energy tests.

## 4. Perpetual Motion Machines

### 4.1 Continuous motion and perpetual energy production

Three different ideas must be distinguished:

1. motion continuing indefinitely in an ideal mathematical model;
2. motion continuing in a real physical system;
3. continuous production of useful work without an external energy supply.

An ideal frictionless pendulum may continue moving indefinitely in a mathematical model. This does not mean it can provide unlimited useful energy.

Extracting useful work removes mechanical energy from the system unless an external source replaces it.

### 4.2 Perpetual motion of the first kind

A perpetual motion machine of the first kind would continuously produce energy without an equivalent energy input.

This would violate the first law of thermodynamics.

For a system operating in a complete cycle,

\[
\Delta U=0
\]

Therefore, its net work output cannot exceed its net energy input.

A machine claiming

\[
W_{\mathrm{out}}>E_{\mathrm{in}}
\]

without an additional energy source contradicts energy conservation.

### 4.3 Perpetual motion of the second kind

A perpetual motion machine of the second kind would extract heat from a single thermal reservoir and convert it completely into work through a repeating cycle, with no other effect.

This would violate the second law of thermodynamics.

The second law does not state that heat can never be converted into work. It states that a cyclic process cannot convert all heat from a single reservoir completely into useful work without producing another change.

### 4.4 Perpetual motion of the third kind

A so-called perpetual motion machine of the third kind is imagined to operate forever by eliminating every source of dissipation.

A mathematical model may assume:

- zero bearing friction;
- zero air resistance;
- perfectly rigid materials;
- no sound;
- no vibration;
- no electromagnetic losses.

Real devices cannot satisfy all of these assumptions exactly.

Even a perfectly lossless moving object would not be an unlimited energy source. Any extraction of useful work would reduce its stored mechanical energy.

### 4.5 Why the double pendulum is not a perpetual-motion machine

For the ideal double pendulum,

\[
E=T+V=\text{constant}
\]

Energy changes between gravitational potential energy and kinetic energy, but the total does not increase.

If an electrical generator were attached to the pivot, it would apply a resisting torque. Mechanical energy would be transferred into electrical energy, causing the pendulum's motion to decrease unless energy were supplied from an external source.

Continuous motion in an ideal simulation is therefore not equivalent to continuous production of useful energy.

### 4.6 Real energy dissipation

A real double pendulum loses organized mechanical energy through:

- pivot and bearing friction;
- air resistance;
- sound;
- structural vibration;
- material deformation;
- internal friction.

The total energy of the pendulum and its environment remains conserved, but useful mechanical energy becomes dispersed as internal or thermal energy.

Thermal energy is not destroyed, but the second law limits the fraction that can be recovered as useful work in a cyclic process.

## 5. Euler–Richardson Algorithm

### 5.1 Definition

The Euler–Richardson algorithm is commonly identified with the explicit midpoint method.

It is a two-stage, second-order Runge–Kutta method. It should not be confused with:

- the basic Euler method;
- Heun's method;
- Richardson extrapolation;
- the implicit midpoint method.

### 5.2 General formulation

For the differential equation

\[
\dot{\mathbf y}=f(t,\mathbf y)
\]

with time step \(h\), the algorithm calculates

\[
\mathbf k_1=f(t_n,\mathbf y_n)
\]

It then estimates the state at the midpoint:

\[
\mathbf y_{\mathrm{mid}}
=
\mathbf y_n+\frac{h}{2}\mathbf k_1
\]

The derivative is evaluated again at the midpoint:

\[
\mathbf k_2=
f\left(
t_n+\frac{h}{2},
\mathbf y_{\mathrm{mid}}
\right)
\]

The complete step is

\[
\mathbf y_{n+1}
=
\mathbf y_n+h\mathbf k_2
\]

### 5.3 Comparison with the basic Euler method

The basic Euler method uses

\[
\mathbf y_{n+1}
=
\mathbf y_n+h f(t_n,\mathbf y_n)
\]

It assumes that the derivative at the beginning of the interval adequately represents the entire interval.

Euler–Richardson estimates the derivative at the midpoint, giving a more accurate representation of the trajectory's curvature.

For sufficiently smooth problems:

- basic Euler has global error \(O(h)\);
- Euler–Richardson has global error \(O(h^2)\).

The local truncation errors are:

- \(O(h^2)\) for basic Euler;
- \(O(h^3)\) for Euler–Richardson.

### 5.4 Double-pendulum implementation

For

\[
\mathbf y=
[\theta_1,\omega_1,\theta_2,\omega_2]
\]

the derivative function returns

\[
f(\mathbf y)=
[\omega_1,\alpha_1,\omega_2,\alpha_2]
\]

A language-independent implementation is:

```text
function derivative(time, state):
    theta1 = state.theta1
    omega1 = state.omega1
    theta2 = state.theta2
    omega2 = state.omega2

    alpha1, alpha2 = calculateAccelerations(
        theta1,
        omega1,
        theta2,
        omega2
    )

    return [omega1, alpha1, omega2, alpha2]


function eulerRichardsonStep(time, state, dt):
    k1 = derivative(time, state)

    midpointState = state + 0.5 * dt * k1

    k2 = derivative(
        time + 0.5 * dt,
        midpointState
    )

    return state + dt * k2
```

Every component of the state must be advanced to the midpoint. A common programming error is to calculate midpoint angles while continuing to use the original angular velocities.

### 5.5 Numerical energy error

The ideal physical equations conserve energy, but the Euler–Richardson method does not conserve energy exactly.

This distinction is essential:

\[
\text{physical conservation}
\neq
\text{exact numerical conservation}
\]

The simulated energy may:

- oscillate around the correct value;
- drift upward;
- drift downward;
- become unstable if the time step is too large.

A reliable simulation should monitor

\[
E_n=T_n+V_n
\]

and the relative energy error

\[
\epsilon_E(t_n)
=
\frac{E_n-E_0}
{\max(|E_0|,E_{\mathrm{scale}})}
\]

where \(E_{\mathrm{scale}}\) prevents division by a value close to zero.

The correct scientific statement is:

> The ideal differential equations conserve mechanical energy, while the numerical approximation preserves it only within a measurable error tolerance.

### 5.6 Chaotic amplification of numerical error

In a chaotic regime, small numerical errors may grow approximately as

\[
\epsilon(t)\sim\epsilon_0e^{\lambda t}
\]

until they reach the scale of the accessible phase-space region.

Therefore, two correctly implemented numerical methods may eventually produce visibly different trajectories.

This does not automatically mean that one method is incorrect.

The following quantities may remain scientifically meaningful even after point-by-point trajectories diverge:

- short-term convergence;
- energy statistics;
- phase-space structure;
- Lyapunov exponents;
- distributions of states;
- qualitative transitions between regular and chaotic regimes.

### 5.7 Time-step convergence test

The simulation should be repeated using

\[
h,\qquad \frac h2,\qquad \frac h4
\]

For a second-order method,

\[
e_h\approx Ch^2
\]

Therefore,

\[
\frac{
\|y_h-y_{h/2}\|
}{
\|y_{h/2}-y_{h/4}\|
}
\approx4
\]

over a sufficiently short time interval and before chaotic divergence dominates the comparison.

### 5.8 Comparison with other integration methods

| Method | Global order | Main advantage | Main limitation |
|---|---:|---|---|
| Explicit Euler | 1 | Very simple | Low accuracy and poor stability |
| Euler–Richardson | 2 | Simple and educational | Long-term energy drift |
| Classical RK4 | 4 | Strong short-term accuracy | Not structure-preserving |
| Adaptive RK45 | Variable | Automatic local-error control | May show long-term energy drift |
| Implicit midpoint | 2 | Symplectic for Hamiltonian systems | Requires solving implicit equations |
| Leapfrog or Verlet | 2 | Good long-term geometric behaviour | Less convenient for some coupled coordinate forms |
| Higher-order symplectic methods | Higher | Better Hamiltonian preservation | More complex implementation |

Euler–Richardson is suitable for an educational simulation. For long-duration analysis of conservative dynamics, comparison with a symplectic method is recommended.

## 6. Recommended Simulation Architecture

### 6.1 Model parameters

The program should explicitly define:

\[
m_1,\quad m_2,\quad l_1,\quad l_2,\quad g
\]

It should document:

- the angle convention;
- the coordinate system;
- whether the rods have mass;
- whether the masses are point particles;
- whether damping is included.

### 6.2 Initial conditions

The complete initial state is

\[
\theta_1(0),\quad
\omega_1(0),\quad
\theta_2(0),\quad
\omega_2(0)
\]

Angles should be stored internally in radians.

### 6.3 Solver configuration

The implementation should record:

- numerical integration method;
- time-step size;
- total simulation duration;
- floating-point precision;
- output-sampling interval.

### 6.4 Validation tests

#### Static equilibrium

The state

\[
\theta_1=\theta_2=0
\]

\[
\omega_1=\omega_2=0
\]

should remain static.

#### Small-angle behaviour

Small initial angles should produce approximately regular oscillations.

#### Energy monitoring

The program should plot

\[
E(t)-E(0)
\]

and report the maximum and root-mean-square energy errors.

#### Time-step convergence

The trajectories produced using \(h\), \(h/2\), and \(h/4\) should be compared over a short interval.

#### Nearby trajectories

Two nearly identical initial states should be simulated to demonstrate sensitivity to initial conditions.

#### Independent solver comparison

Euler–Richardson should be compared against a trusted method such as RK4 or adaptive RK45 over a short interval.

#### Limit checks

Simplified parameter choices should be tested against known special cases, such as approximately reducing one part of the system to a simple pendulum.

## 7. Common Conceptual and Programming Errors

### 7.1 Confusing chaos with randomness

Chaotic motion is deterministic. Its unpredictability comes from sensitivity to initial conditions, not random laws.

### 7.2 Claiming every double-pendulum trajectory is chaotic

The double pendulum can exhibit regular or chaotic motion depending on parameters, energy, and initial conditions.

### 7.3 Claiming exact energy conservation in a numerical simulation

The ideal equations conserve energy, but a numerical solver generally introduces energy error.

### 7.4 Using inconsistent angle conventions

The coordinate equations, potential energy, and acceleration equations must use the same angle definitions.

### 7.5 Mixing old and midpoint state variables

Euler–Richardson requires all state variables to be advanced to the midpoint before the midpoint derivative is calculated.

### 7.6 Using a time step that is too large

An excessively large time step may cause:

- artificial energy growth;
- incorrect rotations;
- unstable velocities;
- numerical divergence;
- false indications of chaos.

### 7.7 Treating continuous ideal motion as free energy

A lossless simulation does not create energy. Extracting work would reduce the system's mechanical energy.

### 7.8 Ignoring angle wrapping

When comparing trajectories, angular differences should be normalized because angles differing by \(2\pi\) represent the same physical orientation.

## 8. Scientific Interpretation

The double pendulum demonstrates that deterministic laws do not guarantee unlimited predictive power.

Its future state is fixed by its equations and exact initial conditions. However, practical measurements and numerical representations always contain finite errors. In chaotic regimes, these small errors grow rapidly, limiting the period over which detailed prediction remains trustworthy.

The ideal model conserves mechanical energy, but that does not make it a perpetual-motion energy source. It merely means that energy is exchanged between kinetic and potential forms without dissipation.

The numerical simulation introduces an additional layer of approximation. Euler–Richardson is more accurate than basic Euler integration, but it is not exactly energy-conserving and does not eliminate chaotic amplification of numerical uncertainty.

A scientifically credible simulation must therefore distinguish among:

- deterministic dynamics;
- practical unpredictability;
- physical energy conservation;
- thermodynamic irreversibility;
- real-world dissipation;
- numerical truncation error;
- floating-point error;
- chaotic growth of perturbations.

## 9. Conclusion

The double pendulum is a compact but powerful model for studying nonlinear mechanics and deterministic chaos. Its coupled equations can produce regular, quasiperiodic, rotational, or chaotic behaviour depending on its parameters and initial state.

Deterministic chaos does not contradict classical mechanics. The system remains governed by deterministic equations, but its sensitivity to initial conditions places a practical limit on long-term prediction.

The ideal double pendulum conserves total mechanical energy, but it cannot create energy or function as a perpetual-motion machine. Real pendulums also lose useful mechanical energy through friction, air resistance, vibration, and deformation.

The Euler–Richardson algorithm provides a clear second-order method for simulating the system. However, it introduces numerical error and does not preserve energy exactly. Reliable results require time-step convergence tests, energy monitoring, comparison with independent solvers, and careful separation of physical chaos from numerical instability.

## References

1. Massachusetts Institute of Technology OpenCourseWare. *Double Pendulum: Lagrange Equations and Nonlinear Motion*.  
   https://ocw.mit.edu/courses/16-07-dynamics-fall-2009/

2. Massachusetts Institute of Technology OpenCourseWare. *Classical Mechanics III: Nonlinear Dynamics and Chaos*.  
   https://ocw.mit.edu/courses/8-09-classical-mechanics-iii-fall-2014/

3. University of Maryland Department of Physics. *Numerical Integration and the Double Pendulum*.  
   https://physics.umd.edu/hep/drew/numerical_integration/pendulum2.html

4. University of Oxford Department of Physics. *Basic Thermodynamics*.  
   https://www.physics.ox.ac.uk/system/files/file_attachments/basic_thermo.pdf

5. SciPy Documentation. *solve_ivp — Solve an Initial Value Problem for a System of ODEs*.  
   https://docs.scipy.org/doc/scipy/reference/generated/scipy.integrate.solve_ivp.html

6. Wolfram MathWorld. *Lyapunov Characteristic Exponent*.  
   https://mathworld.wolfram.com/LyapunovCharacteristicExponent.html

7. Hairer, E., Lubich, C., and Wanner, G. *Geometric Numerical Integration: Structure-Preserving Algorithms for Ordinary Differential Equations*. Springer.

8. Strogatz, S. H. *Nonlinear Dynamics and Chaos*. Westview Press.

9. Goldstein, H., Poole, C., and Safko, J. *Classical Mechanics*. Addison-Wesley.
