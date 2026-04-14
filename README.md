# Rocket-Optimization: Gemini Project

## About the Project
This repository contains the source code, datasets, and Machine Learning architecture developed for the predictive optimization and risk analysis of launch parameters for the "Gemini" amateur sub-orbital vehicle (Team Pietralata). 

The system acts as an automated "Mission Control": it analyzes the meteorological conditions detected in the field and instantaneously calculates the optimal launch setup (Ballast, Launch Rail Angle, Parachute Delay) to hit a specific altitude target while guaranteeing maximum flight stability and safety.

Context Note: This project was developed for an aerospace competition in Italy. To ensure maximum operational consistency during launch procedures, the entire codebase (variables, functions, comments) and associated datasets are natively written in Italian.

## Objectives & Aerodynamic Context
The flight dynamics of a rocket are heavily influenced by the wind (a phenomenon known as weathercocking). Therefore, it is impossible to fix the launch parameters a priori. This algorithm solves the problem by calculating the exact combination to:
1. Reach the Target Apogee: Ensure the rocket hits the desired altitude.
2. Minimize the Angle of Inclination Error: Guarantee that the rocket flies as perfectly vertical as possible (approximately 90 degrees relative to the ground) during the ascent phase.

## Dataset & The "Perfect Multiplier"
The project relies on a comprehensive dataset generated through advanced deterministic simulations using OpenRocket, organized via Latin Hypercube Sampling (LHS) to ensure a perfectly distributed exploration of the parameter space.

A major component of the data preparation was solving the simulator's "spatial illusion." OpenRocket provides spatial landing coordinates rather than a direct vector of aerodynamic drift. To resolve this, we implemented a geometric deduction based on the "Perfect Multiplier": calculating the ideal ratio between the launch angle and wind speed. This mathematical baseline allowed us to accurately classify if a launch setup resulted in overcompensation (flying into the wind) or undercompensation (drifting with the wind), creating a physically coherent Ground Truth.

## Methodology & Modeling
Because of the non-linearity of aerodynamic variables, developing a closed physical equation is impractical. The predictive core of this project is a Stacking Ensemble Regressor.

Level 0 Models (Base Learners): The architecture utilizes four different algorithms in parallel to capture distinct aspects of the flight dynamics:
* Random Forest Regressor (Variance management)
* XGBoost (Complex pattern extraction)
* Support Vector Regression (SVR) (Geometric tolerance)
* Multi-Layer Perceptron (MLP) (Neural Network)

Meta-Model: A Ridge Regression (L2 Penalty) analyzes the predictions of the base models, weighting the reliability of each algorithm through Out-of-Fold (OOF) cross-validation to prevent overfitting.

Note: The architecture utilizes Deep Cloning (`sklearn.base.clone`) to create two physically distinct model architectures in memory—one specialized in predicting the apogee and the other the flight angle—preventing weight overwriting.

## Risk Analysis Engine
Beyond ballistic prediction, the system integrates a probabilistic module dedicated to Risk Analysis. Utilizing a scenario-based statistical approach (Monte Carlo simulations), the system:
* Estimates the trajectory Bias and Dispersion (Standard Deviation) based on live turbulence.
* Calculates the limits of the Safety Windows by evaluating the Worst-Case Positive (+) and Worst-Case Negative (-) scenarios.
* Ensures that the maximum tolerable dispersion never forces the rocket outside the allowed safety cone of the launch field.

## Optimization Strategy & Safety Logic

The final layer of the project is the optimization algorithm (`calcola_setup`) that explores thousands of parameter permutations to find the ideal setup, utilizing a dual-stage logic:

* Plan A (Target First): The system filters for the specific Target Apogee (with a strict +/- 5m tolerance) and extracts the setup that guarantees the straightest possible flight.
* Plan B (Safety First): If the desired apogee is physically unreachable due to weather conditions or payload constraints, the software drops the altitude target. It filters only for absolute safety (maximum 1 degree error from vertical), sacrificing altitude to protect the structural integrity of the mission.

Additionally, the algorithm neutralizes OpenRocket's systematic "optimism" by applying a 10% bias reduction to the training target, forcing the network to model real-world friction. It also incorporates adjustments for "Open Field" launches, factoring in the wind boundary layer gradient and laminar turbulence to transpose ground-level anemometer readings into realistic predictions at a 200m altitude.

## Author
- Simone Massinelli
