# HC--LS: Homotopy Continuation–Least Squares for SEIR Models

## Overview

The **HC--LS framework** implements parameter estimation for SEIR-type epidemiological models using **integral elimination**. It is designed for situations where only **partial observations of the system states** are available, which is common in epidemiological data. In such settings, traditional derivative-based methods can struggle due to noise or missing state variables.

The key idea is to transform the parameter estimation problem into a **polynomial system** using integral elimination. Homotopy continuation is then applied to compute all candidate parameter solutions without requiring initial guesses. These candidate solutions are subsequently **projected into the feasible parameter region** and refined using a **least-squares optimization step**.

To improve numerical stability, several modifications are incorporated into the framework, including ratio-based state reformulation, time rescaling, and improved numerical integration procedures.

### Key Features

- Converts the estimation problem into a **polynomial system using integral elimination**
- Applies **homotopy continuation** to compute candidate parameter solutions without requiring initial guesses
- Applies **projection to candidate solutions** to ensure they are real-valued and satisfy physical parameter bounds
- Refines candidates using **least-squares optimization**, enforcing physically meaningful parameter constraints
- Includes **numerical stabilization techniques**, such as:
  - ratio-based reformulation of state variables  
  - time rescaling to balance integral magnitudes  
  - improved numerical integration (e.g., cumulative Simpson's rule)

---

## Motivation

Estimating parameters in epidemiological models can be challenging, especially when only partial observations are available.

Two common approaches have important limitations:

- **Derivative-based methods** require estimating derivatives from time series data, which is often unreliable in the presence of noise.
- **Least-squares optimization methods** may converge to local minima and typically depend on good initial parameter guesses.

To address these challenges, the HC--LS framework combines **integral elimination**, **symbolic computation**, and **homotopy continuation**.

Integral elimination transforms the original differential equations into algebraic relations by integrating the model equations. This avoids numerical differentiation and improves robustness to noisy observations. The resulting algebraic formulation leads to a polynomial optimization problem that can be solved using homotopy continuation to explore all candidate parameter configurations.

These candidates are then refined using a least-squares step to identify the parameter set that best matches the observed data.

---

## Features

The HC--LS framework includes the following components:

- **Integral elimination** to transform the dynamical system into polynomial relations
- **Homotopy continuation** to compute all candidate solutions of the resulting polynomial system
- **Least-squares refinement** to identify the parameter configuration that best fits the data

To improve numerical stability and estimation accuracy, several stabilization techniques are incorporated:

- **Ratio-based reformulation of state variables**
- **Time rescaling** to balance the magnitude of the integral blocks
- **Projection of candidate solutions** to enforce physical parameter bounds and handle partial non-identifiability

The framework supports multiple numerical integration rules, with **Simpson’s rule** generally producing more stable results than the trapezoidal rule.

---

## Repository Structure

The repository contains both **Julia** and **Python** implementations used throughout the thesis.

- The **Julia implementation** is used for the **SEIR model experiments** and the main **HC--LS framework**.
- The **Python implementation** is mainly used for the **SIR model motivation** and supporting experiments.

### `julia_app/`

This folder contains the main implementation for the **SEIR model**.

- `SEIR ratio/`  
  Contains the core implementation of the **HC--LS framework**, including the introduced techniques to improve numerical stability, and the functions/scripts used for the main numerical experiments.

- `SEIR/`  
  Contains an earlier **non-ratio formulation** of the SEIR model. This version was ultimately abandoned in favor of the ratio-based approach but is kept for reference.

### `python_app/`

This folder contains the **Python implementation**, primarily used for the **SIR model motivation section** of the thesis and additional exploratory experiments.

---

## Installation

1. Clone the repository:  
```bash
git clone https://github.com/JingyanZhang-Chloe/model-integration.git
cd model-integration
```

2. Install Python dependencies:
```bash
pip install -r requirements.txt
```

3. Start Julia with the project environment:
```bash
julia --project=julia_app
```

Alternatively, you can activate the environment directly from the Julia REPL:

```julia
using Pkg
Pkg.activate("julia_app")
Pkg.instantiate()
```

---

## Usage

### Julia Example: Final HC–LS Pipeline

The following example demonstrates the complete HC–LS parameter estimation workflow for the SEIR model.

```julia
include("SEIRModels_ratio.jl")

using .Logic_R
using .Value_R
using HomotopyContinuation

@var αT σT γT S0 E0
variables = [αT, σT, γT, S0, E0]

# Time grid
t = 0.0:10.0:1000.0 |> collect

# Simulate SEIR model
S, E, I, R = Logic_R.simulate_seir(t)

# Add observational noise
noise = 0.01
I_data = I .+ noise .* I .* randn(length(I))

# Run HC--LS parameter estimation
results = Logic_R.HC_LS_complete(t, I_data, variables, "S")

# Print results
Logic_R.print_HC_LS(results)
```

#### Options

- **Time grid (`t`)**  
  Defines the observation times used for simulation and estimation.

- **Noise level (`noise`)**  
  Controls the amount of Gaussian noise added to the simulated data.

- **Integration rule (`method`)**
  - `"S"` — Simpson's rule  
  - `"T"` — Trapezoidal rule

