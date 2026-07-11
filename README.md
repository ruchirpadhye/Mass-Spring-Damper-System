# Mass-Spring-Damper System

Classical and physics-informed neural network (PINN) approaches to the damped harmonic
oscillator — covering forward simulation, inverse parameter estimation, and their
neural-network counterparts, built as a progression from classical numerical methods to
physics-informed machine learning.

## Contents

| Notebook | Description |
|---|---|
| `Oscillator.ipynb` | Classical forward simulation via `scipy.integrate.odeint`. |
| `Oscillator_Inverse.ipynb` | Classical inverse parameter estimation (`c`, `k` from noisy data) via `scipy.optimize.curve_fit`, with uncertainty quantification from the fit covariance matrix. |
| `Oscillator_PINN.ipynb` | Forward Physics-Informed Neural Network, learns `x(t)` directly from the ODE residual and initial conditions, validated against `Oscillator.ipynb`. |
| `Oscillator_inverse_PINNs.ipynb` | Inverse PINN, jointly learns `x(t)` and recovers unknown `c`, `k` from noisy data, validated against `Oscillator_Inverse.ipynb`. |

## Governing Equation

$$ m\ddot{x} + c\dot{x} + kx = 0, \quad x(0) = x_0, \quad \dot{x}(0) = v_0 $$

## Forward PINN — Summary

A small fully-connected network (4 layers, 15 hidden units, Tanh) learns `x(t)` directly
via automatic differentiation, trained on an ODE-residual loss plus an initial-condition
loss. A real failure mode was identified and documented: with only these two loss terms,
the network learns to flatten toward zero for large `t`, since a near-zero output
trivially satisfies the ODE residual once the true solution's amplitude has decayed. This
was corrected by adding a small sparse data-anchoring loss term, an early attempt at this
fix silently failed due to an implementation bug (`torch.inference_mode()` blocking
gradient flow through the anchor loss), documented and corrected in the notebook.
**Final result**: 
* Position `MSE: 0.00076`
* Velocity `MSE: 0.00140` (validated against `odeint`, including an independent velocity check via automatic differentiation).

## Inverse PINN — Summary

Extends the forward PINN to recover unknown `c` and `k` from noisy synthetic
observations, treating them as trainable parameters optimized jointly with the network's
weights via a single shared optimizer. A second implementation bug was found and fixed
during development (the optimizer was initially constructed without `c`/`k` in its
parameter list, so they never updated despite the loss decreasing normally, documented
in the notebook). Results are compared directly against a classical `curve_fit` baseline
solving the same recovery problem from the same noisy data.

**Result (m=5, c=1.5, k=10, noise σ=0.3):**

| Parameter | True | `curve_fit` | PINN |
|---|---|---|---|
| c | 1.5 | 1.498 | 1.943 |
| k | 10.0 | 10.026 | 10.259 |

`k` recovery is accurate for both methods. `c` recovery shows a real, ~30% bias in the
PINN relative to `curve_fit`'s near-exact estimate, documented as an open limitation
rather than resolved, with a discussion of possible causes and next steps in the
notebook.

## Setup

```bash
pip install torch numpy scipy matplotlib
```

Notebooks run top to bottom with no interactive input required. Random seeds are fixed
for reproducibility.

## Notes

- The forward PINNs are parameter-specific, they do not generalize to different `m, c, k`
  without retraining.
- Debugging notes are left in the notebooks deliberately (not edited out), including two
  genuine implementation bugs and their diagnoses, since the debugging process is treated
  as part of the technical record rather than something to hide.
- The inverse PINN's damping-recovery limitation is an open question, not a settled result
  — see `Oscillator_inverse_PINNs.ipynb` for discussion.
