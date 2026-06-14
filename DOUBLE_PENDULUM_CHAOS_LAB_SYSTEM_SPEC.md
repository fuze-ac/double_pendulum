# Double Pendulum Chaos Lab
## Source-of-Truth System Specification

**Document status:** Active source of truth  
**Product type:** Interactive educational physics simulation  
**Primary stack:** Next.js App Router, TypeScript, HTML Canvas 2D  
**Initial deployment:** Static export  
**Primary numerical method:** Euler–Richardson (explicit midpoint)  
**Reference numerical methods:** Explicit Euler and classical RK4  
**Audience:** Students, educators, programmers, and general learners  
**Last updated:** 2026-06-14

---

## 1. Purpose

This document defines the authoritative product, physics, software architecture, simulation behavior, module boundaries, state models, numerical interfaces, validation requirements, page structure, and implementation order for **Double Pendulum Chaos Lab**.

All implementation work must follow this specification unless this document is explicitly revised.

The project must teach users:

1. how simple and double pendulums move;
2. why the double pendulum is nonlinear and coupled;
3. what deterministic chaos means;
4. why sensitivity to initial conditions limits long-term prediction;
5. how mechanical energy behaves in ideal and damped systems;
6. why ideal continuous motion is not a perpetual-motion energy source;
7. how Euler and Euler–Richardson approximate differential equations;
8. how time-step size and numerical error affect simulation results.

---

## 2. Product principles

### 2.1 Scientific correctness first

Visual attractiveness must never override physical correctness.

The implementation must distinguish among:

- exact mathematical laws;
- idealized physical assumptions;
- numerical approximation;
- floating-point error;
- chaotic divergence;
- real-world dissipation.

### 2.2 Educational transparency

The application must show not only the animation but also:

- current state variables;
- physical parameters;
- solver name;
- time step;
- total mechanical energy;
- energy error;
- formulas used;
- assumptions and limitations.

### 2.3 Deterministic reproducibility

Given the same:

- initial state;
- physical parameters;
- solver;
- time step;
- simulation duration;

the simulation must produce the same numerical result on the same JavaScript number implementation, apart from acceptable rendering differences.

### 2.4 Separation of concerns

The following layers must remain independent:

1. physics;
2. numerical solvers;
3. simulation runtime;
4. rendering;
5. user interface;
6. educational content;
7. data export;
8. validation tests.

React components must not contain the authoritative equations of motion.

Canvas renderers must not modify physical state.

### 2.5 Progressive learning

The product must guide users from:

1. simple pendulum;
2. double pendulum;
3. coupled energy;
4. regular and chaotic motion;
5. numerical methods;
6. time-step error;
7. damping and energy extraction;
8. free laboratory exploration.

---

## 3. Technology decisions

### 3.1 Required stack

- Next.js App Router
- TypeScript
- React
- HTML Canvas 2D
- CSS Modules or a documented project-wide CSS system
- Vitest or Jest for unit tests
- Playwright for browser-level tests
- Static JSON or TypeScript content definitions for presets and experiments

### 3.2 Initial deployment mode

The initial production build must use:

```ts
// next.config.ts
const nextConfig = {
  output: "export",
};

export default nextConfig;
```

The initial release must not depend on:

- a Node.js runtime server;
- a database;
- authentication;
- server actions;
- runtime API routes;
- external physics APIs.

### 3.3 Client and server boundaries

Educational text pages should be Server Components where possible.

Interactive simulation shells must use Client Components.

The `"use client"` boundary should be introduced only at the smallest practical interactive component boundary.

### 3.4 Physics execution

Initial implementation:

- physics runs on the main browser thread;
- Canvas renders on the main browser thread;
- `requestAnimationFrame` controls rendering;
- a fixed-step accumulator controls physics.

Future optimization:

- Web Workers may be introduced only after profiling;
- OffscreenCanvas may be introduced only if browser compatibility and measured performance justify it;
- the core physics and solver APIs must remain worker-compatible from the start.

---

## 4. Non-goals for the first release

The first release will not include:

- multiplayer;
- cloud synchronization;
- accounts;
- database persistence;
- collaborative experiments;
- machine-learning prediction;
- formal symbolic algebra;
- three-dimensional rendering;
- WebGL;
- automatic proof of chaos;
- high-order Lyapunov-spectrum computation;
- arbitrary mechanical-system construction.

---

## 5. Scientific conventions

### 5.1 Units

The simulation uses SI units internally:

| Quantity | Unit |
|---|---|
| Length | metre |
| Mass | kilogram |
| Time | second |
| Angle | radian |
| Angular velocity | radian per second |
| Angular acceleration | radian per second squared |
| Energy | joule |
| Power | watt |
| Gravity | metre per second squared |

User interfaces may display degrees, but all internal calculations must use radians.

### 5.2 Coordinate system

The fixed pivot is the origin.

- Positive \(x\): right
- Positive \(y\): downward for Canvas convenience
- Angles: measured from the downward vertical
- Positive angle: clockwise in screen coordinates

The coordinate convention must be documented in the mathematics page.

### 5.3 Ideal-model assumptions

Unless damping is enabled:

- rods are rigid and massless;
- masses are point particles;
- pivots are frictionless;
- air resistance is zero;
- gravity is uniform;
- no external force is applied;
- total physical mechanical energy is conserved.

Numerical energy is only approximately conserved.

---

## 6. Physical models

# 6.1 Simple pendulum

## 6.1.1 State

\[
\mathbf y =
\begin{bmatrix}
\theta \\
\omega
\end{bmatrix}
\]

where:

\[
\omega = \dot{\theta}
\]

## 6.1.2 Equation of motion

Without damping:

\[
\ddot{\theta} = -\frac{g}{l}\sin\theta
\]

With linear angular damping:

\[
\ddot{\theta}
=
-\frac{g}{l}\sin\theta
-\frac{c}{ml^2}\omega
\]

## 6.1.3 Position

\[
x=l\sin\theta
\]

\[
y=l\cos\theta
\]

## 6.1.4 Energy

\[
T=\frac12ml^2\omega^2
\]

Using the lowest position as zero potential:

\[
V=mgl(1-\cos\theta)
\]

\[
E=T+V
\]

---

# 6.2 Double pendulum

## 6.2.1 Parameters

\[
m_1,\quad m_2,\quad l_1,\quad l_2,\quad g
\]

All must be finite and positive, except optional damping coefficients, which may be zero.

## 6.2.2 State

\[
\mathbf y =
\begin{bmatrix}
\theta_1 \\
\omega_1 \\
\theta_2 \\
\omega_2
\end{bmatrix}
\]

where:

\[
\omega_1=\dot{\theta}_1
\]

\[
\omega_2=\dot{\theta}_2
\]

## 6.2.3 Cartesian positions

\[
x_1=l_1\sin\theta_1
\]

\[
y_1=l_1\cos\theta_1
\]

\[
x_2=x_1+l_2\sin\theta_2
\]

\[
y_2=y_1+l_2\cos\theta_2
\]

## 6.2.4 Kinetic energy

\[
T=
\frac12(m_1+m_2)l_1^2\omega_1^2
+\frac12m_2l_2^2\omega_2^2
+m_2l_1l_2\omega_1\omega_2
\cos(\theta_1-\theta_2)
\]

## 6.2.5 Potential energy

Using the lowest reachable configuration as zero potential:

\[
V=
(m_1+m_2)gl_1(1-\cos\theta_1)
+
m_2gl_2(1-\cos\theta_2)
\]

## 6.2.6 Total energy

\[
E=T+V
\]

## 6.2.7 Angular accelerations

Define:

\[
\Delta=\theta_1-\theta_2
\]

\[
D=m_1+m_2-m_2\cos^2\Delta
\]

Then:

\[
\alpha_1=
\frac{
-m_2l_1\omega_1^2\sin\Delta\cos\Delta
+m_2g\sin\theta_2\cos\Delta
-m_2l_2\omega_2^2\sin\Delta
-(m_1+m_2)g\sin\theta_1
}{
l_1D
}
\]

\[
\alpha_2=
\frac{
(m_1+m_2)
\left(
l_1\omega_1^2\sin\Delta
-g\sin\theta_2
+g\sin\theta_1\cos\Delta
\right)
+m_2l_2\omega_2^2\sin\Delta\cos\Delta
}{
l_2D
}
\]

The derivative is:

\[
f(\mathbf y)=
\begin{bmatrix}
\omega_1 \\
\alpha_1 \\
\omega_2 \\
\alpha_2
\end{bmatrix}
\]

## 6.2.8 Denominator safety

For positive masses:

\[
D=m_1+m_2-m_2\cos^2\Delta \ge m_1
\]

Therefore, \(D\) must remain positive when \(m_1>0\).

If any calculation produces:

- `NaN`;
- `Infinity`;
- a non-positive mass or length;
- a non-finite derivative;

the runtime must pause the simulation and display a numerical-error message.

---

# 6.3 Damping and generator model

Damping is an educational approximation, not a detailed aerodynamic model.

## 6.3.1 Linear pivot damping

For each pivot:

\[
\tau_{d1}=-c_1\omega_1
\]

\[
\tau_{d2}=-c_2\omega_2
\]

The damping implementation must be derived consistently through generalized forces or a documented approximation.

It must not simply subtract arbitrary values from angles or velocities.

## 6.3.2 Generator load

Generator extraction is modeled as additional resisting torque:

\[
\tau_{g1}=-k_{g1}\omega_1
\]

\[
\tau_{g2}=-k_{g2}\omega_2
\]

Approximate extracted instantaneous power:

\[
P_{\mathrm{out}}
=
k_{g1}\omega_1^2+k_{g2}\omega_2^2
\]

Cumulative extracted energy:

\[
E_{\mathrm{out}}(t)
=
\int_0^t P_{\mathrm{out}}\,dt
\]

The UI must explain that extracted energy comes from the pendulum's mechanical energy.

---

## 7. Deterministic-chaos model

### 7.1 Nearby trajectories

Two simulations may be initialized as:

\[
\mathbf y_B(0)=\mathbf y_A(0)+\delta\mathbf y(0)
\]

The default perturbation should modify one angle only.

Recommended default:

\[
\delta\theta_1=10^{-5}\text{ radians}
\]

### 7.2 Wrapped angular difference

Angular differences must use:

\[
\operatorname{wrap}(\Delta\theta)
=
((\Delta\theta+\pi)\bmod 2\pi)-\pi
\]

### 7.3 Normalized state-space separation

Because angles and angular velocities have different units, the preferred educational metric is:

\[
d(t)=
\sqrt{
\left(\frac{\Delta\theta_1}{s_\theta}\right)^2+
\left(\frac{\Delta\omega_1}{s_\omega}\right)^2+
\left(\frac{\Delta\theta_2}{s_\theta}\right)^2+
\left(\frac{\Delta\omega_2}{s_\omega}\right)^2
}
\]

Recommended scales:

\[
s_\theta=1\text{ radian}
\]

\[
s_\omega=1\text{ radian per second}
\]

The UI must label this as a normalized state-space distance, not a physical distance in metres.

### 7.4 Chaos-language restriction

The UI may use:

- “sensitive divergence”;
- “chaotic behavior” for a validated preset;
- “chaotic-like” for exploratory states.

The UI must not claim that a single visual divergence formally proves chaos.

### 7.5 Optional Lyapunov estimate

A future advanced module may estimate the largest Lyapunov exponent using periodic perturbation renormalization.

It is not required for the first release.

---

## 8. Numerical solvers

# 8.1 Common solver contract

```ts
export type DerivativeFunction<State, Params> = (
  time: number,
  state: Readonly<State>,
  params: Readonly<Params>,
  out: State
) => State;

export interface FixedStepSolver<State, Params> {
  readonly id: SolverId;
  readonly name: string;
  readonly order: number;

  step(
    time: number,
    state: Readonly<State>,
    params: Readonly<Params>,
    dt: number,
    derivative: DerivativeFunction<State, Params>,
    out: State
  ): State;
}
```

Requirements:

- solvers must be deterministic;
- solvers must not mutate input state;
- solvers may reuse caller-provided output and scratch buffers;
- solvers must not allocate new arrays during every physics step;
- solvers must validate that `dt` is finite and positive.

# 8.2 Explicit Euler

\[
\mathbf y_{n+1}
=
\mathbf y_n+h f(t_n,\mathbf y_n)
\]

Purpose:

- educational comparison;
- intentionally low-order baseline;
- never the default production solver.

Global order:

\[
O(h)
\]

# 8.3 Euler–Richardson

Euler–Richardson is the project's primary named method and is implemented as the explicit midpoint Runge–Kutta method.

\[
\mathbf k_1=f(t_n,\mathbf y_n)
\]

\[
\mathbf y_{\mathrm{mid}}
=
\mathbf y_n+\frac h2\mathbf k_1
\]

\[
\mathbf k_2=
f\left(
t_n+\frac h2,
\mathbf y_{\mathrm{mid}}
\right)
\]

\[
\mathbf y_{n+1}
=
\mathbf y_n+h\mathbf k_2
\]

Global order:

\[
O(h^2)
\]

This is the default solver for educational simulations.

# 8.4 Classical RK4

\[
\mathbf k_1=f(t_n,\mathbf y_n)
\]

\[
\mathbf k_2=f\left(t_n+\frac h2,\mathbf y_n+\frac h2\mathbf k_1\right)
\]

\[
\mathbf k_3=f\left(t_n+\frac h2,\mathbf y_n+\frac h2\mathbf k_2\right)
\]

\[
\mathbf k_4=f(t_n+h,\mathbf y_n+h\mathbf k_3)
\]

\[
\mathbf y_{n+1}
=
\mathbf y_n+
\frac h6
\left(
\mathbf k_1+2\mathbf k_2+2\mathbf k_3+\mathbf k_4
\right)
\]

Purpose:

- short-term reference;
- solver comparison;
- not a claim of exact truth;
- not exactly energy-conserving.

# 8.5 Solver registry

```ts
export type SolverId =
  | "euler"
  | "euler-richardson"
  | "rk4";

export interface SolverDescriptor {
  id: SolverId;
  label: string;
  order: number;
  educationalSummary: string;
}
```

The UI must select solvers through the registry, not by importing arbitrary implementations inside components.

---

## 9. State models

# 9.1 Branded units

TypeScript cannot enforce physical dimensions automatically, but branded units should be used at public boundaries where practical.

```ts
export type Radians = number & { readonly __unit: "radians" };
export type Seconds = number & { readonly __unit: "seconds" };
export type Kilograms = number & { readonly __unit: "kilograms" };
export type Metres = number & { readonly __unit: "metres" };
```

Internal hot-loop storage may use plain numbers after validation.

# 9.2 Double-pendulum state

```ts
export interface DoublePendulumState {
  theta1: number;
  omega1: number;
  theta2: number;
  omega2: number;
}
```

# 9.3 Double-pendulum parameters

```ts
export interface DoublePendulumParams {
  mass1: number;
  mass2: number;
  length1: number;
  length2: number;
  gravity: number;
  damping1: number;
  damping2: number;
  generator1: number;
  generator2: number;
}
```

# 9.4 Simulation configuration

```ts
export interface SimulationConfig {
  solverId: SolverId;
  fixedDt: number;
  timeScale: number;
  maxSubSteps: number;
  sampleInterval: number;
  trailLength: number;
  paused: boolean;
}
```

# 9.5 Runtime state

```ts
export interface SimulationRuntimeState {
  simulationTime: number;
  accumulator: number;
  stepCount: number;
  droppedTime: number;
  status: "idle" | "running" | "paused" | "error";
  errorCode?: SimulationErrorCode;
}
```

# 9.6 Observable snapshot

React must receive throttled immutable snapshots rather than high-frequency mutable physics objects.

```ts
export interface SimulationSnapshot {
  time: number;
  state: DoublePendulumState;
  kineticEnergy: number;
  potentialEnergy: number;
  totalEnergy: number;
  absoluteEnergyError: number;
  relativeEnergyError: number;
  extractedEnergy: number;
  stepCount: number;
}
```

Recommended UI snapshot frequency:

- 10 Hz for text values;
- 20–30 Hz for graphs;
- browser refresh rate for Canvas.

---

## 10. Simulation runtime

# 10.1 Fixed-step accumulator

Rendering frame time must not be used directly as the physics step.

Required algorithm:

```ts
accumulator += Math.min(realDeltaSeconds, maxFrameDelta) * timeScale;

let subSteps = 0;

while (accumulator >= fixedDt && subSteps < maxSubSteps) {
  integrateOneStep(fixedDt);
  accumulator -= fixedDt;
  subSteps += 1;
}

if (subSteps === maxSubSteps && accumulator >= fixedDt) {
  droppedTime += accumulator;
  accumulator = 0;
}

render(interpolationAlpha);
```

Recommended defaults:

```ts
fixedDt = 1 / 240;
maxSubSteps = 32;
maxFrameDelta = 0.1;
timeScale = 1;
```

# 10.2 Rendering interpolation

The first release may render the latest state without interpolation.

Interpolation may be added later, but it must never feed interpolated values back into physics.

# 10.3 Page visibility

When the page becomes hidden:

- rendering may pause;
- the accumulator must not accumulate unbounded hidden-tab time;
- resuming must reset the wall-clock timestamp;
- simulation time must continue only if explicitly designed and documented.

Default behavior: pause wall-clock advancement while hidden.

# 10.4 Error handling

Pause and report an error when:

- state becomes non-finite;
- energy becomes non-finite;
- invalid parameters are supplied;
- fixed time step is invalid;
- maximum safe velocity threshold is exceeded due to instability;
- a solver throws.

---

## 11. Rendering architecture

# 11.1 Canvas renderer contract

```ts
export interface PendulumRenderer {
  resize(viewport: CanvasViewport): void;

  render(
    context: CanvasRenderingContext2D,
    model: RenderModel
  ): void;

  destroy(): void;
}
```

# 11.2 Render model

```ts
export interface RenderModel {
  current: DoublePendulumState;
  previous?: DoublePendulumState;
  params: DoublePendulumParams;
  viewport: CanvasViewport;
  trails: ReadonlyArray<TrailPoint>;
  comparisonState?: DoublePendulumState;
  showVectors: boolean;
  showAngles: boolean;
  showTrails: boolean;
}
```

# 11.3 High-DPI support

Canvas backing dimensions must account for `devicePixelRatio`.

CSS dimensions and backing-store dimensions must be separated.

The renderer must apply a controlled transform so physical drawing coordinates remain stable.

# 11.4 Rendering restrictions

The renderer must not:

- integrate physics;
- change the state;
- normalize angles for the solver;
- calculate authoritative energy;
- modify presets;
- dispatch React updates per drawing operation.

---

## 12. Graph architecture

Graphs required:

- angle versus time;
- angular velocity versus time;
- energy versus time;
- energy error versus time;
- normalized trajectory separation;
- phase plots.

Graph data must use bounded ring buffers.

```ts
export interface TimeSeriesPoint {
  time: number;
  value: number;
}
```

The system must avoid unbounded arrays during long sessions.

Recommended maximum displayed samples per series: 2,000–5,000.

Older samples may be decimated or discarded.

---

## 13. Simulation modules

The first production release contains eight simulations.

# 13.1 Simulation 1: Simple Pendulum Foundation

Purpose:

- introduce angle, gravity, velocity, period, kinetic energy, and potential energy.

Required features:

- one pendulum;
- start, pause, step, reset;
- initial angle and velocity;
- mass, length, gravity;
- energy bars;
- angle graph;
- ideal and damped modes.

# 13.2 Simulation 2: Double Pendulum Motion

Purpose:

- introduce two degrees of freedom and coupled nonlinear motion.

Required features:

- all physical parameters;
- all initial-state variables;
- trails;
- state display;
- energy display;
- formula panel;
- regular and high-energy presets.

# 13.3 Simulation 3: Coupling and Energy Transfer

Purpose:

- show that energy moves among parts while total ideal energy remains approximately constant numerically.

Required visualizations:

- total kinetic energy;
- total potential energy;
- total mechanical energy;
- optional per-mass kinetic contribution;
- synchronized motion and graph cursor.

# 13.4 Simulation 4: Initial-Condition Sensitivity

Purpose:

- compare nearby deterministic trajectories.

Required features:

- two overlaid or side-by-side systems;
- perturbation control;
- normalized state-space separation;
- logarithmic separation graph;
- reset with exact reproducibility.

# 13.5 Simulation 5: Regular Versus Chaotic Motion

Purpose:

- demonstrate that the double pendulum is not chaotic for every state.

Required presets:

- low-energy regular;
- transitional;
- high-sensitivity chaotic;
- rotational.

Language must avoid presenting visual classification as formal proof.

# 13.6 Simulation 6: Euler Versus Euler–Richardson

Purpose:

- teach numerical integration.

Required features:

- Euler and Euler–Richardson side by side;
- optional RK4 reference;
- one-step mode;
- visual midpoint;
- current derivative;
- state difference;
- energy-error comparison.

# 13.7 Simulation 7: Time-Step Error Laboratory

Purpose:

- teach convergence and numerical instability.

Required runs:

\[
h,\quad h/2,\quad h/4
\]

Required output:

- short-time state differences;
- energy errors;
- computation counts;
- convergence ratio.

For a second-order solver:

\[
R=
\frac{
\|y_h-y_{h/2}\|
}{
\|y_{h/2}-y_{h/4}\|
}
\]

should approach 4 in a suitable convergence region.

# 13.8 Simulation 8: Energy Dissipation and Perpetual Motion

Purpose:

- distinguish ideal conservation from unlimited energy production.

Required modes:

- ideal;
- damped;
- generator load.

Required output:

- remaining mechanical energy;
- cumulative dissipated energy;
- cumulative extracted energy;
- total accounting balance;
- explanatory energy-flow diagram.

---

## 14. Presets

Each preset must include:

```ts
export interface SimulationPreset {
  id: string;
  title: string;
  description: string;
  model: "simple" | "double";
  state: DoublePendulumState;
  params: DoublePendulumParams;
  config: Partial<SimulationConfig>;
  learningTags: string[];
  validationNotes?: string;
}
```

Required initial presets:

1. Small Regular Oscillation
2. Moderate Coupling
3. Chaotic Sensitivity
4. Near-Identical Pair
5. Full Rotation
6. Damped Motion
7. Generator Load
8. Large-Step Failure Demonstration

Preset values must be validated numerically before release.

---

## 15. Page structure and sitemap

```text
/
├── learn/
│   ├── simple-pendulum/
│   ├── double-pendulum/
│   ├── deterministic-chaos/
│   ├── energy/
│   ├── perpetual-motion/
│   └── numerical-integration/
│
├── simulations/
│   ├── simple-pendulum/
│   ├── double-pendulum/
│   ├── energy-transfer/
│   ├── initial-condition-sensitivity/
│   ├── regular-vs-chaotic/
│   ├── solver-comparison/
│   ├── timestep-error/
│   └── damping-and-generator/
│
├── laboratory/
├── mathematics/
├── experiments/
├── glossary/
├── sources/
└── about/
```

# 15.1 Home page

Required sections:

- product introduction;
- animated but lightweight hero preview;
- “Start with the simple pendulum” entry;
- simulation cards;
- learning pathway;
- scientific disclaimer;
- source summary.

# 15.2 Learn pages

Each learning page must include:

- concept explanation;
- formula;
- variable definitions;
- physical interpretation;
- common misconception;
- link to associated simulation;
- source references.

# 15.3 Simulation pages

Common layout:

1. title and objective;
2. Canvas;
3. playback controls;
4. parameter controls;
5. live state;
6. graph tabs;
7. formula inspector;
8. guided experiment;
9. explanation;
10. assumptions and limitations.

# 15.4 Laboratory

The laboratory must expose all safe model controls and allow:

- solver selection;
- preset loading;
- manual initial conditions;
- graph selection;
- CSV export;
- configuration export and import;
- reproducible reset.

# 15.5 Mathematics page

Required sections:

- coordinate convention;
- simple-pendulum derivation;
- double-pendulum coordinates;
- kinetic energy;
- potential energy;
- Lagrangian;
- equations of motion;
- state-space conversion;
- Euler method;
- Euler–Richardson method;
- RK4 reference;
- numerical error;
- energy accounting.

---

## 16. Component architecture

```text
app/
├── layout.tsx
├── page.tsx
├── learn/
├── simulations/
├── laboratory/
├── mathematics/
├── glossary/
├── sources/
└── about/

components/
├── simulation/
│   ├── SimulationShell.tsx
│   ├── PendulumCanvas.tsx
│   ├── PlaybackControls.tsx
│   ├── ParameterPanel.tsx
│   ├── StatePanel.tsx
│   └── FormulaInspector.tsx
├── charts/
├── education/
├── layout/
└── common/

src/
├── physics/
├── solvers/
├── runtime/
├── rendering/
├── graphs/
├── presets/
├── experiments/
├── export/
├── validation/
└── types/
```

---

## 17. Required source files

```text
src/
├── physics/
│   ├── simplePendulum.ts
│   ├── doublePendulum.ts
│   ├── doublePendulumEnergy.ts
│   ├── doublePendulumDamping.ts
│   ├── coordinates.ts
│   └── angleMath.ts
│
├── solvers/
│   ├── solverTypes.ts
│   ├── euler.ts
│   ├── eulerRichardson.ts
│   ├── rk4.ts
│   └── solverRegistry.ts
│
├── runtime/
│   ├── SimulationController.ts
│   ├── FixedStepLoop.ts
│   ├── SnapshotPublisher.ts
│   ├── DataRecorder.ts
│   └── SimulationError.ts
│
├── rendering/
│   ├── CanvasViewport.ts
│   ├── DoublePendulumRenderer.ts
│   ├── SimplePendulumRenderer.ts
│   ├── TrailBuffer.ts
│   ├── VectorRenderer.ts
│   └── renderTypes.ts
│
├── graphs/
│   ├── RingBuffer.ts
│   ├── TimeSeriesModel.ts
│   ├── PhaseSpaceModel.ts
│   └── SeparationModel.ts
│
├── validation/
│   ├── parameterValidation.ts
│   ├── stateValidation.ts
│   ├── finiteChecks.ts
│   └── energyBalance.ts
│
├── presets/
│   ├── simplePendulumPresets.ts
│   ├── doublePendulumPresets.ts
│   └── presetTypes.ts
│
├── experiments/
│   ├── experimentTypes.ts
│   └── experiments.ts
│
└── export/
    ├── csvExport.ts
    ├── configExport.ts
    └── configImport.ts
```

---

## 18. Validation strategy

Scientific validation is a release requirement, not an optional test category.

# 18.1 Unit tests

Required unit tests:

- angle wrapping;
- degree-to-radian conversion;
- coordinate conversion;
- energy formulas;
- derivative finiteness;
- solver step behavior;
- ring-buffer capacity;
- parameter validation;
- CSV serialization.

# 18.2 Static-equilibrium test

Initial state:

\[
\theta_1=0,\quad
\omega_1=0,\quad
\theta_2=0,\quad
\omega_2=0
\]

Expected:

- derivatives are zero within floating-point tolerance;
- state remains unchanged;
- energy remains zero under the selected energy reference.

# 18.3 Vertical inverted-state test

At:

\[
\theta_1=\pi,\quad
\theta_2=\pi,\quad
\omega_1=\omega_2=0
\]

Expected:

- instantaneous angular accelerations are zero in exact arithmetic;
- the state is unstable under perturbation;
- numerical noise behavior must be documented.

# 18.4 Energy test

For an undamped short-duration run:

- total energy must remain within method-specific tolerance;
- error must reduce with smaller time step;
- Euler–Richardson should outperform Euler at comparable step size in validated scenarios.

No universal long-duration energy bound should be promised for chaotic states.

# 18.5 Solver-order convergence

For a smooth short-time test:

Euler:

\[
\frac{e_h}{e_{h/2}}\approx2
\]

Euler–Richardson:

\[
\frac{e_h}{e_{h/2}}\approx4
\]

RK4:

\[
\frac{e_h}{e_{h/2}}\approx16
\]

The reference solution must use a much smaller time step and a higher-order method.

# 18.6 Small-angle validation

For sufficiently small angles:

- motion should remain regular;
- frequencies should be consistent with the linearized model;
- numerical trajectories should converge as `dt` decreases.

# 18.7 Symmetry tests

For symmetric parameters and sign-reversed initial angles:

\[
(\theta_1,\theta_2,\omega_1,\omega_2)
\rightarrow
(-\theta_1,-\theta_2,-\omega_1,-\omega_2)
\]

the resulting trajectory should reflect the corresponding symmetry within numerical tolerance.

# 18.8 Energy-accounting test with damping

For damped or generator runs:

\[
E_{\mathrm{initial}}
\approx
E_{\mathrm{mechanical}}(t)
+
E_{\mathrm{dissipated}}(t)
+
E_{\mathrm{extracted}}(t)
+
E_{\mathrm{numerical\ error}}(t)
\]

The accounting residual must be measured and displayed in validation reports.

# 18.9 Determinism test

Two identical runs must produce identical state arrays.

# 18.10 Browser integration tests

Required browser tests:

- simulation starts;
- pause stops physics time;
- step advances one fixed step;
- reset restores exact initial state;
- preset loading is deterministic;
- invalid input is rejected;
- Canvas resizes;
- CSV export works;
- hidden-tab resume does not create a giant time jump.

# 18.11 Accessibility tests

Required:

- keyboard operation;
- visible focus states;
- labels for all controls;
- no color-only distinctions;
- reduced-motion option;
- pause availability;
- readable Canvas-adjacent text alternatives.

---

## 19. Numerical tolerances

Tolerances must be centralized.

```ts
export const NUMERICAL_TOLERANCES = {
  equilibriumDerivative: 1e-12,
  finiteLimit: 1e12,
  energyRelativeShortRun: 1e-4,
  solverOrderTolerance: 0.35,
  symmetryTolerance: 1e-9,
} as const;
```

These values are initial targets and must be revised based on measured test fixtures.

Tolerance changes require:

- documented reason;
- before-and-after test evidence;
- update to this specification.

---

## 20. Performance requirements

Initial target devices:

- current desktop browsers;
- mainstream mobile browsers;
- integrated graphics;
- no dedicated GPU requirement.

Performance targets:

- 60 FPS rendering where device refresh allows;
- no more than 16 ms average rendering work per visible frame on target desktop;
- physics simulation stable at default `1/240 s`;
- bounded graph and trail memory;
- no per-step React render;
- no unbounded allocation in the hot loop;
- graceful reduction of trails and graph sampling on low-performance devices.

React state updates must be throttled.

Physics must not depend on React render timing.

---

## 21. Accessibility and learning safety

Required features:

- pause;
- reduced-motion mode;
- trail disable;
- speed control;
- keyboard controls;
- numeric input alternatives to sliders;
- equations in accessible HTML or MathML-compatible rendering;
- textual graph summaries;
- high-contrast-compatible interface;
- responsive controls.

Rapid motion must never be forced without a pause control.

---

## 22. Data export

# 22.1 CSV columns

```text
time
theta1
omega1
theta2
omega2
kineticEnergy
potentialEnergy
totalEnergy
absoluteEnergyError
relativeEnergyError
extractedEnergy
separation
solver
fixedDt
```

# 22.2 Configuration export

Configuration JSON must include:

- schema version;
- model;
- state;
- parameters;
- solver;
- fixed time step;
- time scale;
- preset identifier if applicable.

Imported files must be schema-validated before use.

---

## 23. Content and terminology rules

Use:

- deterministic chaos;
- sensitive dependence on initial conditions;
- numerical approximation;
- physical energy conservation;
- numerical energy drift;
- ideal model;
- damped model;
- generator load.

Avoid:

- “completely random”;
- “future becomes mathematically undetermined”;
- “energy is perfectly conserved by the solver”;
- “free energy”;
- “proof of chaos” based only on animation;
- “perpetual motion” to describe ordinary ideal conservative motion without qualification.

---

## 24. Implementation order

# Phase 0: Repository foundation

Deliverables:

- Next.js App Router project;
- TypeScript strict mode;
- linting;
- formatting;
- test runner;
- Playwright;
- static-export configuration;
- CI scripts.

Exit criteria:

- production build succeeds;
- static export succeeds;
- tests execute in CI.

# Phase 1: Mathematical core

Implementation order:

1. unit and angle utilities;
2. state and parameter types;
3. parameter validation;
4. coordinate functions;
5. simple-pendulum derivative;
6. double-pendulum derivative;
7. kinetic, potential, and total energy;
8. finite-value guards.

Exit criteria:

- equilibrium tests pass;
- energy formulas pass hand-calculated fixtures;
- derivatives remain finite for validated presets.

# Phase 2: Solver core

Implementation order:

1. generic solver interfaces;
2. Explicit Euler;
3. Euler–Richardson;
4. RK4;
5. solver registry;
6. scratch-buffer reuse.

Exit criteria:

- solver-order tests pass;
- deterministic repeat tests pass;
- no per-step allocation in measured hot paths.

# Phase 3: Runtime core

Implementation order:

1. fixed-step loop;
2. simulation controller;
3. snapshot publisher;
4. data recorder;
5. error state;
6. pause, step, reset;
7. hidden-tab handling.

Exit criteria:

- physics time is frame-rate independent;
- pause and single-step tests pass;
- no giant resume jump occurs.

# Phase 4: Canvas foundation

Implementation order:

1. viewport and DPI scaling;
2. simple-pendulum renderer;
3. double-pendulum renderer;
4. trails;
5. angle guides;
6. velocity vectors;
7. resize handling.

Exit criteria:

- rendering does not mutate state;
- Canvas works on desktop and mobile;
- reduced-motion mode works.

# Phase 5: First complete simulation

Build the Double Pendulum Motion page first.

Required:

- Canvas;
- controls;
- presets;
- live state;
- energy;
- reset;
- pause;
- step;
- solver and `dt` display.

Exit criteria:

- scientific validation report passes;
- page works as static export.

# Phase 6: Educational simulation set

Build in this order:

1. Simple Pendulum Foundation
2. Energy Transfer
3. Initial-Condition Sensitivity
4. Regular Versus Chaotic
5. Euler Versus Euler–Richardson
6. Time-Step Error Laboratory
7. Damping and Generator

# Phase 7: Graph system

Implementation order:

1. ring buffer;
2. time-series graph;
3. energy graph;
4. phase-space graph;
5. separation graph;
6. synchronized cursor;
7. textual graph summary.

# Phase 8: Learning content

Build:

- Learn pages;
- Mathematics page;
- Glossary;
- Guided experiments;
- Common misconceptions;
- Sources page.

# Phase 9: Export and laboratory

Build:

- CSV export;
- configuration export;
- configuration import;
- free laboratory;
- parameter warnings;
- reproducible experiment links if compatible with static routing.

# Phase 10: Production hardening

Complete:

- browser matrix testing;
- mobile performance;
- accessibility audit;
- numerical validation report;
- source review;
- metadata and SEO;
- static-host deployment;
- error monitoring that does not collect sensitive user data.

---

## 25. Release gates

A simulation may not be called production-ready until:

1. equations match this specification;
2. equilibrium tests pass;
3. solver convergence is demonstrated;
4. energy behavior is measured;
5. invalid input is handled;
6. fixed-step timing is verified;
7. mobile performance is acceptable;
8. accessibility controls are present;
9. assumptions are visible;
10. numerical limitations are disclosed.

---

## 26. Future extensions

Permitted future work:

- Web Worker physics;
- OffscreenCanvas rendering;
- implicit midpoint solver;
- symplectic integrators;
- adaptive solver comparison;
- Lyapunov exponent estimator;
- Poincaré sections;
- parameter-space maps;
- classroom worksheet mode;
- shareable configuration URLs;
- downloadable experiment reports;
- localization.

Future additions must not bypass the core solver and physics contracts.

---

## 27. Authoritative references

1. MIT OpenCourseWare — Dynamics and double-pendulum derivation  
   https://ocw.mit.edu/courses/16-07-dynamics-fall-2009/

2. MIT OpenCourseWare — Nonlinear dynamics and chaos  
   https://ocw.mit.edu/courses/8-09-classical-mechanics-iii-fall-2014/

3. University of Maryland Physics — Double-pendulum numerical integration  
   https://physics.umd.edu/hep/drew/numerical_integration/pendulum2.html

4. Next.js documentation — App Router and static export  
   https://nextjs.org/docs  
   https://nextjs.org/docs/app/guides/static-exports

5. MDN Web Docs — Canvas API and requestAnimationFrame  
   https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API  
   https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame

6. MDN Web Docs — OffscreenCanvas  
   https://developer.mozilla.org/en-US/docs/Web/API/OffscreenCanvas

7. TypeScript Handbook — Modules and type-system guidance  
   https://www.typescriptlang.org/docs/handbook/2/modules.html

8. Hairer, Lubich, and Wanner — *Geometric Numerical Integration*

9. Strogatz — *Nonlinear Dynamics and Chaos*

10. Goldstein, Poole, and Safko — *Classical Mechanics*

---

## 28. Final architecture decision

The production system is:

- Next.js App Router for content, routing, metadata, and UI;
- TypeScript for explicit contracts and maintainability;
- HTML Canvas 2D for simulation rendering;
- framework-independent physics modules;
- framework-independent fixed-step numerical solvers;
- a deterministic runtime controller;
- throttled React snapshots;
- static export for initial deployment;
- scientific validation as a mandatory release gate.

This document is the governing implementation specification for Double Pendulum Chaos Lab.
