# Mass-Spring-Damper (MSD) System
Forward and Inverse Modeling of Mass-Spring-Damper System

## Project Overview
This repository contains a two-part basic computational physics project modeling a Mass-Spring-Damper system. The project aims to demonstrate an entire cycle: from Forward Modeling (simulating system physics from known parameters) to Inverse Modeling (extracting physical properties from noisy, real-world-style sensor data).

## Methodology
* **Language:** Python
* **Visualization:** `matplotlib`
* **Equation Solver:** `scipy`

## Governing Physics
The mechanical oscillator is governed by the following second-order Ordinary Differential Equation (ODE):

$$m \frac{d^2x}{dt^2} + c \frac{dx}{dt} + kx = 0$$

To solve this numerically, we decompose the system into a state-space formulation (two first-order ODEs):

$$\frac{dx}{dt} = v$$ and $$\frac{dv}{dt} = - \frac{(cv + kx)}{m}$$

## Project Components
**Forward Simulation:** 
* **Objective:** Solve the ODE using `scipy.integrate.odeint` to predict system position and velocity over time.
* **Validation**: Implemented an automated classification logic to distinguish between underdamped, critically damped, and overdamped regimes based on the critical damping coefficient ($c_c = 2\sqrt{mk}$).

An example shown with the following parameters:
* **Mass** = 10 kg
* **Damping factor** = 1.5
* **Spring Constant** = 20 $\frac{N}{m}$
* **Initial Position** = 0 m
* **Initial velocity** = -2 $\frac{m}{s}$
  
<img width="1012" height="547" alt="image" src="https://github.com/user-attachments/assets/4798767b-5a87-443f-8b7f-b48af19ef6f2" />
<img width="1189" height="590" alt="image" src="https://github.com/user-attachments/assets/06655400-18dd-4d88-a1a4-03a6c635f62f" />

**Inverse Simulation:**
* **Objective:** Reverse-engineer physical properties ($c$ and $k$) from synthetic sensor data injected with Gaussian noise.
* **Methodology:** Generate actual time-series data, corrupt it with Gaussian Noise on purpose. Utilize `scipy.optimize.curve_fit` and pivot one parameter (currenly $m$ is fixed) to estimate the values of missing parameters.
* **Validation:** Use the covariance matrix `pcov` to determine the standard error $\sigma$.

An example shown with the following parameters:
* **Mass** = 20 kg
* **Damping factor** = 1.5
* **Spring Constant** = 10 $\frac{N}{m}$
* **Initial Position** = 2 m
* **Initial velocity** = 0 $\frac{m}{s}$

Estimated parameters:
* **$c$** = 1.63
* **$k$** = 10.02 $\frac{N}{m}$

<img width="999" height="547" alt="image" src="https://github.com/user-attachments/assets/ebb0ec07-30f4-4363-adcd-da3c41e6009a" />
<img width="1012" height="547" alt="image" src="https://github.com/user-attachments/assets/34c3d94b-77c0-4bb9-966e-0437f9951f9b" />

## Result
The optimization pipeline successfully recovered the system parameters with high precision (e.g., stiffness ($k$) error < 0.7%) despite the presence of sensor noise. 
  
---
*Developed as part of an autonomous systems and computational engineering portfolio.*
