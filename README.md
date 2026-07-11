# Mass-Spring-Damper System

Classical and physics-informed neural network (PINN) approaches to the damped harmonic
oscillator, extending forward and inverse modeling of a simple mechanical system.

## Contents

- **`Oscillator.ipynb`** — Classical forward simulation using `scipy.integrate.odeint`.
- **`Oscillator_Inverse.ipynb`** — Inverse parameter estimation (recovering `c`, `k` from
  noisy position data) using `scipy.optimize.curve_fit`, with uncertainty quantification
  from the fit covariance matrix.
- **`Oscillator_PINN.ipynb`** — Forward Physics-Informed Neural Network solving the same
  ODE, trained via automatic differentiation instead of numerical integration.

## Governing Equation

$$ m\ddot{x} + c\dot{x} + kx = 0, \quad x(0) = x_0, \quad \dot{x}(0) = v_0 $$

Default parameters: `m = 1.0 kg`, `c = 0.5 N·s/m`, `k = 2.0 N/m`, `x0 = 1.0 m`, `v0 = 0.0 m/s`.

## PINN Notebook — Summary

A small fully-connected network (4 layers, 15 hidden units, Tanh activation) is trained
to approximate `x(t)` directly, using a loss built from three terms: the ODE residual
(via automatic differentiation of the network's own output), the initial-condition error
at `t = 0`, and a sparse data-anchoring term at ~29 points across the domain.

**A real failure mode was encountered and documented rather than hidden**: training with
only the physics and initial-condition losses produced a network that matched the true
solution well for early time but flattened to zero for larger `t`, since a near-zero
output trivially satisfies the ODE residual once the true solution's amplitude decays.
Sparse data anchoring was added to correct this — and an implementation bug (the anchor
loss was initially computed under `torch.inference_mode()`, silently preventing any
gradient from flowing through it) meant the first attempted fix wasn't actually training
on the anchor points at all. Once corrected, the fix worked as intended.

**Final result**: Position MSE `0.00076`, Velocity MSE `0.00140` (validated against the
classical `odeint` solution, including an independent check of velocity obtained purely
via automatic differentiation of the network's output).

## Setup

```bash
pip install torch numpy scipy matplotlib
```

Run notebooks top to bottom; no interactive input required.

## Notes

- This is a **forward** PINN for one fixed parameter set — it does not generalize to
  different `m`, `c`, `k` without retraining. An inverse PINN (recovering unknown
  parameters, extending `Oscillator_Inverse.ipynb`) is a natural next step.
- Random seed is fixed for reproducibility.
