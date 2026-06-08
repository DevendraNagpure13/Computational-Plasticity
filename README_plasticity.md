# Computational Plasticity — Homework Assignments

**Author:** Devendra Nagpure  
**Roll Number:** ME22D034

This repository contains three homework assignments for a graduate-level course in Computational Plasticity. Each assignment progressively builds from 1D elasto-plastic bar models to full 3D continuum plasticity with numerical integration.

---

## Repository Structure

```
├── Homework_1/
│   ├── ME22D034_Devendra_Homework_1.pdf     # Handwritten theory derivations
│   └── homework1.ipynb                      # Python simulation (1D bar)
├── Homework_2/
│   ├── ME22D034_Devendra.pdf                # Jupyter notebook — beam bending
│   └── homework2.ipynb
├── Homework_3/
│   ├── ME22D034_Devendra_Homework_3.pdf     # Handwritten theory + Python notebook
│   └── homework3.ipynb
└── README.md
```

---

## Homework 1 — 1D Elasto-Plastic Bar: Theory and Simulation

### Theory (Handwritten)

Derives the complete 1D rate-independent plasticity framework for a spring-slider model with combined hardening:

| Component | Expression |
|---|---|
| Elastic law | σ = E(ε − εᵖ), rate form: σ̇ = E(ε̇ − ε̇ᵖ) |
| Yield criterion | f = \|β\| − σ_y(s) ≤ 0, where β = σ − α |
| Flow rule | ε̇ᵖ = λ̇ sgn(β) |
| Kinematic hardening | α̇ = K ε̇ᵖ (back-stress evolution) |
| Isotropic hardening | σ_y = σ_y0 + H sⁿ, ṡ = \|ε̇ᵖ\| = \|λ̇\| |
| KKT conditions | λ̇ ≥ 0, f ≤ 0, λ̇f = 0 |
| Consistency condition | if λ̇ > 0, then ḟ = 0 |

From the consistency condition, the plastic multiplier is derived as:

$$\dot{\lambda} = \max\left\{0,\ \frac{E\dot{\varepsilon}}{K + E + h}\,\operatorname{sgn}(\beta)\right\}$$

The elasto-plastic tangent modulus:

$$E^t = \frac{E(K + nHs^{n-1})}{K + E + nHs^{n-1}}$$

**Special cases:**
- H = 0 (pure kinematic): `E^t = EK/(K+E)` — yield surface translates only
- K = 0 (pure isotropic): `E^t = E·nHs^(n-1)/(E + nHs^(n-1))` — yield surface expands only
- n = 1 (linear combined): `E^t = E(K+H)/(K+E+H)` — linear expansion and translation

### Simulation (Python)

Strain loading: ε(t) = 0.002 sin(2πt) over 10 cycles

```python
E        = 100000       # Young's modulus
sigma_y0 = 1e-3 * E    # Initial yield strength
dt       = 0.01         # Time step
```

Three material models are simulated and compared:

**(i) Elastic-Perfectly Plastic** — K = 0, H = 0
- Stress clamps at ±σ_y0; constant energy dissipation per cycle
- Square hysteresis loop in σ–ε space

**(ii) Linear Kinematic Hardening** — K = E/4, H = 0 → E^t = 0.2E
- Back-stress shifts yield surface; closed hysteresis loops → shakedown
- Energy dissipation decreases and stabilizes cycle-by-cycle

**(iii) Linear Isotropic Hardening** — K = 0, H = 0.2E
- Yield surface expands; plastic strain range shrinks each cycle
- Energy dissipation decays to zero → elastic shakedown

---

## Homework 2 — Elasto-Plastic Beam Bending with Voce Hardening

### Problem

Cantilevered beam, rectangular cross-section: width a = 10 mm, height 2a = 20 mm, length 20a.  
Material: isotropic hardening with **Voce hardening law**:

$$\sigma_y(s) = \sigma_0 + (\sigma_u - \sigma_0)\left[1 - \exp\!\left(-\frac{s}{s_0}\right)\right]$$

```
E  = 200,000 MPa
σ₀ = E/500 = 400 MPa     (initial yield stress)
σᵤ = 1.5 σ₀ = 600 MPa   (ultimate stress)
s₀ = 0.1                 (Voce saturation parameter)
```

### Solution 1 — Monotonic Loading

Monotonically increasing bending moment M up to M/M_y = 1.2, 1.4, 1.6, 1.8, 2.0.  
Newton-Raphson used at each material point for the nonlinear Voce yield condition.  
Iterative moment-curvature loop enforces global equilibrium.

| Output plot | Description |
|---|---|
| Normalized moment vs. normalized curvature (aρ) | Nonlinear stiffening due to Voce hardening |
| Stress distribution across beam height | Linear elastic core; saturating plateau at outer fibers |
| Plastic strain distribution across beam height | Grows from outer fibers inward with increasing M |

**Key observations:**
- At M/M_y = 1.2 only a thin outer layer has yielded
- By M/M_y = 2.0 plasticity penetrates nearly the full cross-section (plastic hinge forming)
- Voce saturation causes the M–κ curve to flatten at high curvature

### Solution 2 — Cyclic Loading

Curvature prescribed as ρ(t) = ρ₀ sin(2π · 10 · t), amplitude ρ₀ = 3M_y / (2EI), over 10 cycles.

- Hysteresis loops confirm energy dissipation due to cyclic plasticity
- Loops stabilize as isotropic hardening saturates → approach to elastic shakedown
- Permanent curvature accumulates; beam does not return to zero state on unloading

---

## Homework 3 — 3D Continuum Plasticity: Theory and Radial Return

### Theory (Handwritten)

Extends the framework to 3D with von Mises yield criterion and combined hardening:

| Component | Expression |
|---|---|
| Elastic law | σ̇ᵢⱼ = 2μ(ε̇ᵢⱼ − ε̇ᵢⱼᵖ) + λ̄ ε̇ₖₖ δᵢⱼ |
| Yield criterion | f(β, s) = σ_eq − σ_y(s) ≤ 0 |
| Equivalent stress | σ_eq = √(3/2 · βᵢⱼ βᵢⱼ), βᵢⱼ = τᵢⱼ − αᵢⱼ |
| Flow rule | ε̇ᵢⱼᵖ = (3/2) λ̇ βᵢⱼ / σ_eq |
| Kinematic hardening | α̇ᵢⱼ = (2/3) K ε̇ᵢⱼᵖ = K λ̇ βᵢⱼ / σ_eq |
| Plastic multiplier | λ̇ = 2μ βᵢⱼ ε̇ᵢⱼ / [σ_eq(3μ + K + h)] |
| Tangent modulus tensor | C^t_ijkl = C_ijkl − 9μ² Bᵢⱼ Bₖₗ / [σ_y²(3μ + K + h)] |

The discrete **elastic predictor – radial return** algorithm is derived.  
The nonlinear scalar equation for the plastic increment δλ:

$$f^t - \sigma_y(s + \delta\lambda) + \sigma_y(s) - 3\mu\,\delta\lambda = 0$$

is solved by Newton-Raphson, followed by state variable updates for s, εᵖᵢⱼ, τᵢⱼ.

### Simulation (Python)

**Material:** Power-law isotropic hardening — σ_y(s) = σ₀(1 + 500s)ⁿ

```python
E        = 100000
nu       = 0.3
sigma_0  = E / 500
n_values = [0, 0.1]      # Perfectly plastic vs. strain hardening
```

Two proportional strain loading paths up to ε₁₁ = 0.05:

| Load path | Strain tensor | Physical meaning |
|---|---|---|
| (a=−1, b=0) | diag(1, −1, 0) · dε | Uniaxial-like with lateral contraction |
| (a=−0.5, b=−0.5) | diag(1, −0.5, −0.5) · dε | Axisymmetric constraint |

**Results summary:**
- n = 0 (perfectly plastic): σ₁₁ saturates immediately at yield; stress components plateau
- n = 0.1 (power-law hardening): σ₁₁ continues rising; hardening rate decelerates due to power-law saturation
- Path 2 (a=−0.5, b=−0.5): σ₂₂ = σ₃₃ by symmetry (confirmed by overlapping curves); higher σ₁₁ than Path 1 at equal ε₁₁ due to triaxiality
- Plastic incompressibility tr(εᵖ) = 0 verified in all cases

---

## Dependencies

```bash
pip install numpy matplotlib scipy
```

All simulations run in Python 3 (Jupyter notebooks).

---

## Key Algorithms Summary

| Assignment | Numerical Method |
|---|---|
| HW1 | Explicit return mapping (1D), consistency condition |
| HW2 | Iterative moment-curvature integration, Newton-Raphson (Voce) |
| HW3 | Elastic predictor – radial return (3D), Newton-Raphson for δλ |
