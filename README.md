[model]: ./image/eqns.png "model"
[final]: ./image/mpc_final.gif "Final run on track"
# CarND-Controls-MPC
Self-Driving Car Engineer Nanodegree Program

---
## Overview
This project implements a Model Predictive Controller (MPC). This controller is tasked to  follow a trajectory path to optimize the path the car should take. A simulator is provided by Uacity which communiates track data via websocket. We need to provide different actuator inputs (steering, acceleration and braking) back to simulator to keep the car on track in most efficient way. 
## Dependencies

* cmake >= 3.5
 * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1(mac, linux), 3.81(Windows)
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools]((https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
  * Run either `install-mac.sh` or `install-ubuntu.sh`.
  * If you install from source, checkout to commit `e94b6e1`, i.e.
    ```
    git clone https://github.com/uWebSockets/uWebSockets
    cd uWebSockets
    git checkout e94b6e1
    ```
    Some function signatures have changed in v0.14.x. See [this PR](https://github.com/udacity/CarND-MPC-Project/pull/3) for more details.

* **Ipopt and CppAD:** Please refer to [this document](https://github.com/udacity/CarND-MPC-Project/blob/master/install_Ipopt_CppAD.md) for installation instructions.
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page). This is already part of the repo so you shouldn't have to worry about it.
* Simulator. You can download these from the [releases tab](https://github.com/udacity/self-driving-car-sim/releases).
* Not a dependency but read the [DATA.md](./DATA.md) for a description of the data sent back from the simulator.

NOTE: Additionally, I've also confiigured this project to build using `Visual Studio` on Windows platform with `uWebSockets` using instructions listed [here](http://www.codza.com/blog/udacity-uws-in-visualstudio)


## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./mpc`.
## Tips

1. It's recommended to test the MPC on basic examples to see if your implementation behaves as desired. One possible example
is the vehicle starting offset of a straight line (reference). If the MPC implementation is correct, after some number of timesteps
(not too many) it should find and track the reference line.
2. The `lake_track_waypoints.csv` file has the waypoints of the lake track. You could use this to fit polynomials and points and see of how well your model tracks curve. NOTE: This file might be not completely in sync with the simulator so your solution should NOT depend on it.
3. For visualization this C++ [matplotlib wrapper](https://github.com/lava/matplotlib-cpp) could be helpful.)
4.  Tips for setting up your environment are available [here](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/0949fca6-b379-42af-a919-ee50aa304e6a/lessons/f758c44c-5e40-4e01-93b5-1a82aa4e044f/concepts/23d376c7-0195-4276-bdf0-e02f1f3c665d)
5. **VM Latency:** Some students have reported differences in behavior using VM's ostensibly a result of latency.  Please let us know if issues arise as a result of a VM environment.

## Editor Settings

We've purposefully kept editor configuration files out of this repo in order to
keep it as simple and environment agnostic as possible. However, we recommend
using the following settings:

* indent using spaces
* set tab width to 2 spaces (keeps the matrices in source code aligned)

## Code Style

Please (do your best to) stick to [Google's C++ style guide](https://google.github.io/styleguide/cppguide.html).

## Project Instructions and Rubric

Note: regardless of the changes you make, your project must be buildable using
cmake and make!

More information is only accessible by people who are already enrolled in Term 2
of CarND. If you are enrolled, see [the project page](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/f1820894-8322-4bb3-81aa-b26b3c6dcbaf/lessons/b1ff3be0-c904-438e-aad3-2b5379f0e0c3/concepts/1a2255a0-e23c-44cf-8d41-39b8a3c8264a)
for instructions and the project rubric.

### The Model

This MPC model is based on kinematic bicycle model that neglects the complex interaction between the tires and the road. The parameters for this models are following: 

* x, y: Car's position (where x= horizontal, y=vertical cordinates)
* psi(ψ): Car's orientation/direction
* v: Car's velocity
* cte: Cross-track error
* epsi (eψ): Orientation error

Actuator values (steering angle i.e. delta and throttle) calculated by the MPC. The model combines the state and actuations from the previous timesteps to calculate current state to minimize the cost function using the equations below:

![model]

### Timestep Length and Elapsed Duration (N & dt)

N   — timestep lenth

dt  — elapsed duration between timesteps

These are the two hyperparameters required to be tuned to make this controller work. The final values I selected were `N=10` and `dt=0.1`. Honestly, these values are per suggestion in Udacity's office hours for the project. However, I tried to fine tune and play around with other values e.g. ```(N:20, dt:0.05), (N:12, dt:0.05), (N:20, dt:0.5), etc```. In my observation with lower value of N, the vehicle may continue to drive straight and off the track. On contrary, increasing the `N` value causes the vehicle to oscillate and overshot the reference trajectory. 

A final result will look like this: 

<a href="https://youtu.be/oODwWR3s-us" target="_blank">![final]</a>

### Polynomial Fitting and MPC Preprocessing

The waypoints provided by the simulator are transformed to the car coodrdinate system line line: 112 to 121. Then a third degree polynomial is fitted to the transformed waypoint. These polynomical coefficient are used to calculate the `cte` and `epsi`. They are then used by the `Solve()` to create a reference trajectory.

### Model Predictive Control with Latency

In a real worl scenario the actuation command won't execute instantly. There will be delays for command to physcially take an effect due to nature of the system. We assume a realistic delay might take about 100 ms. 

Therefore, we have added the latency of 100ms in our system to simiulate the real world scenarios. Thus, our intial state should be after 100ms in order to get the optimum control values. 