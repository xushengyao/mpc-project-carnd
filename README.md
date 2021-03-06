# CarND-Controls-MPC
Self-Driving Car Engineer Nanodegree Program

# Overview
This project implements a Model Predictive Controller in the simulator to drive a car with telemetry information.

# Rubric Points

## Compilation

### Your code should compile.
> Code must compile without errors with cmake and make.

It compiles on my ubuntu 16.04 laptop without warning and error.

## Implementation

### The Model
> Student describes their model in detail. This includes the state, actuators and update equations.

This project uses a Kinematic model, which is a simplification of dynamic model that ignore tire forces, gravity, and mass.

The model consists 4 States, 2 Control Inputs, and 2 Errors.

#### States:
* x and y: the coordinates of the vehicle. <br>
* ψ: the orientation of the vehicle. <br>
* v: is the velocity of the vehicle. <br>

#### Control Inputs:
* δ: the steering angle.
* a: the change of speed, positive for acceleration, negative for braking.

The states of the next time step can be calculated at the next time step with the following equations:
* x(t+dt) = x(t) + v(t) ∗ cos(ψ(t)) ∗ dt
* y(t+dt) = y(t) + v(t) ∗ sin(ψ(t)) ∗ dt
* ψ(t+dt) = ψ(t) + (v(t)/Lf) ∗ δ(t) ∗ dt
* v(t+dt) = v(t) + a(t) ∗ dt

#### Errors and Cost Function:
Cross Track Error (cte) and Psi Error (eψ) are used to build the cost function of the model. They are calculated at the next time step with the following equations:
* cte(t+dt) = cte(t) + v(t) ∗ sin(eψ(t)) * dt
* eψ(t+dt) = eψ(t) + (v(t)/Lf) ∗ δ(t) ∗ dt


### Timestep Length and Elapsed Duration (N & dt)
> Student discusses the reasoning behind the chosen N (timestep length) and dt (elapsed duration between timesteps) values. Additionally the student details the previous values tried.

* Timestep Length: N = 10
* Elapsed Duration: dt = 0.1

  The prediction horizon T is the duration over which future predictions are made, which is the product of two other variables, N and dt. I started with N = 50 and dt = 0.1 to predict 5 seconds but it doesn't produce a good green line prediction, so I keep tuning down N to 10 and the green line looks reasonable.


### Polynomial Fitting and MPC Preprocessing
> A polynomial is fitted to waypoints.

> If the student preprocesses waypoints, the vehicle state, and/or actuators prior to the MPC procedure it is described.

Waypoints are converted to the vehicle coordinate, and a 3rd degree polynomials is used to fit to waypoints:
```cpp
for (size_t i = 0; i < ptsx.size(); i++) {
  Eigenpts_X[i] = (ptsx[i] - px) * cos(psi) + (ptsy[i] - py) * sin(psi);
  Eigenpts_Y[i] = -(ptsx[i] - px) * sin(psi) + (ptsy[i] - py) * cos(psi);
}
Eigen::VectorXd fit_curve_coeffs = polyfit(Eigenpts_X, Eigenpts_Y, 3);
double cte = polyeval(fit_curve_coeffs, px);
double epsi = -atan(fit_curve_coeffs[1]);
```

### Model Predictive Control with Latency
> The student implements Model Predictive Control that handles a 100 millisecond latency. Student provides details on how they deal with latency.

The coordinates are already transform to the car coordinate system, so x, y, psi before latency would be 0, and the states after latency incorporated is computed using previous states and 2 Control Inputs:

```cpp
// dealing with latency:
const double Lf = 2.67;
double delta_angle = j[1]["steering_angle"];
delta_angle = delta_angle * (-1.0);
double acceleration = j[1]["throttle"];
// predict state in 100ms
double latency = 0.1;
double l_x = 0 + v * cos(0) * latency;
double l_y = 0 + v * sin(0) * latency;
double l_psi = 0 + v * delta_angle / Lf * latency;
double l_v = v + acceleration * latency;
double cte = polyeval(fit_curve_coeffs, px);
double epsi = -atan(fit_curve_coeffs[1]);
double l_cte = cte + v * sin(epsi) * latency;
double l_epsi = epsi + v * delta_angle / Lf * latency;
state << l_x, l_y, l_psi, l_v, l_cte, l_epsi;
auto solution = mpc.Solve(state, fit_curve_coeffs);
```


## Simulation

### The vehicle must successfully drive a lap around the track.
> No tire may leave the drivable portion of the track surface. The car may not pop up onto ledges or roll over any surfaces that would otherwise be considered unsafe (if humans were in the vehicle).

> The car can't go over the curb, but, driving on the lines before the curb is ok.

Result video youtube: [link](https://youtu.be/3Oi2jr_bZ_k)

## Installation
Installation instruction from Udacity repository: [link](https://github.com/udacity/CarND-MPC-Project/blob/master/README.md)
