# lane-change
# RL Lateral Controller for Autonomous Vehicles Replication in Highway-Env

This project implements and replicates a Reinforcement Learning (RL) based lateral controller for an autonomous vehicle within the `highway-env` simulation environment. It combines an Intelligent Driver Model (IDM) for longitudinal control with a custom Deep Q-Network (DQN) agent for lateral steering adjustments, specifically focusing on lane-keeping and potentially lane-changing decisions triggered by proximity to a lead vehicle.

## Overview

The core idea is to train an RL agent to output appropriate lateral acceleration commands which are then converted into steering angles. The agent learns a policy to minimize lateral deviations, acceleration, and rate of change based on a custom reward function. Longitudinal control (acceleration/braking) is handled separately by the well-established IDM model.

A key feature is the selective activation of the RL lateral controller. It primarily relies on the IDM for maintaining speed and following distance. Only when the distance to the lead vehicle drops below a specified threshold does a `Gap_Controller` evaluate adjacent lanes for safety and potentially activate the RL agent to steer towards a safer target lane.

## Key Features

*   **Environment:** Utilizes the `highway-env` simulator, configured for continuous actions and kinematic observations.
*   **Hybrid Control:** Combines classical IDM for longitudinal control with RL for lateral control.
*   **RL Agent:** Implements a Deep Q-Network with a specific structure:
    *   `Q(s,a) = A(s) * (B(s) - a)^2 + C(s)`
    *   Three separate networks (A, B, C) approximate different components of the Q-value function. Network B determines the optimal action for a given state.
*   **State Representation:** The RL agent uses a state vector including position (x, y), velocity (vx), heading (theta), target lane width, target lane ID, local road curvature, and the current longitudinal acceleration (provided by IDM).
*   **Action Space:** The agent outputs a continuous value representing lateral acceleration, which is then converted to a steering angle.
*   **Gap Controller:** A rule-based module that checks adjacent lanes for safety (based on distance to the nearest vehicle) when the ego vehicle gets too close to the lead vehicle. It determines the target lane for the RL agent if a lane change is deemed safe and necessary.
*   **Selective Activation:** The RL lateral controller is only active when the gap to the lead vehicle is below a threshold and a safe alternative lane is identified by the `Gap_Controller`. Otherwise, the lateral control command is zero (maintaining the current lane direction).
*   **Reward Function:** A custom reward function penalizes:
    *   High lateral acceleration (action magnitude).
    *   High lateral rate (change in `vy`).
    *   Lateral deviation from the target lane centerline.
*   **Experience Replay:** Uses a replay buffer (`Experience_Buffer`) to store transitions (state, action, reward, next_state, terminal) for efficient training.
*   **Training:** Employs a standard DQN training loop with:
    *   Policy and Target Networks.
    *   Epsilon-greedy exploration strategy with decay.
    *   Adam optimizer and Mean Squared Error (MSE) loss.
    *   Periodic updates of the Target Network.

## Dependencies

You need the following Python libraries:

*   `pygame`: Required by `highway-env` for rendering.
*   `highway-env`: The simulation environment.
*   `torch`: For building and training the neural networks.
*   `numpy`: For numerical operations.
*   `matplotlib`: For plotting results.
*   `gymnasium`: The core environment API (often installed with `highway-env`).

Install them using pip:

```bash
pip install pygame highway-env torch numpy matplotlib gymnasium
