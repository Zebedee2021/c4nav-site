# SpaceR-USV: C4 Official Platform

C4 官方仿真平台 SpaceR-USV 由北京理工大学 SPAIT Lab（周治国团队）开发。

## Architecture: Three-in-One

```
+--------------------+     ROS/rosbridge     +--------------------+
|  Unity 3D Engine   | <------------------> |  ROS Control System |
|                    |   /sensor_data        |  (Docker container) |
| - 3D scene render  |   /action_cmd         | - Navigation        |
| - Sensor simulation|   /reward_signal      | - Decision making   |
| - Collision detect  |                      | - Algorithm training|
| - Weather effects   |                      |                     |
+--------------------+                      +--------------------+
         |                                           |
         |              ROS/rosbridge                |
         |                                           |
+--------+-------------------------------------------+--------+
|                    Simulink (MATLAB)                        |
|  - Fossen 3-DOF dynamics model                              |
|  - Wind / wave / current disturbance                        |
|  - USV130Demo.slx                                           |
+------------------------------------------------------------|
```

**Key insight:** Unity is a pure simulator, not a training platform. Training algorithms run externally via ROS. This makes the platform algorithm-agnostic.

## USV130 Parameters Alignment

| Parameter | USV130 Real Ship | c4nav-core Config | Match |
|-----------|:----------------:|:-----------------:|:-----:|
| Max speed | 1.8 m/s | `max_speed: 1.8` | Exact |
| Cruise speed | 1.4 m/s | `cruise_speed: 1.4` | Exact |
| Max turn rate | 45 deg/s | `max_turn_rate: 45.0` | Exact |
| Hull length | 1.3 m | `collision_radius: 0.65` (half) | Exact |

## Physics Model: Fossen (not MMG)

The official platform uses **Fossen model** (Norwegian school), not MMG (Japanese school):

- Fossen: forces decomposed by **physical property** (inertia M, Coriolis C(v), damping D(v), restoring g(n))
- MMG: forces decomposed by **source** (hull H, propeller P, rudder R)
- Both are mathematically equivalent (Newton-Euler 3-DOF rigid body dynamics)
- Evidence: MATLAB MSS toolbox (fossen.biz/MSS/), confirmed in student thesis (Zu Bowen)

## Comparison: SpaceR-USV vs c4nav-core

| Dimension | SpaceR-USV (Official) | c4nav-core |
|-----------|----------------------|------------|
| Nature | Engineering simulation platform | Algorithm research testbed |
| Physics | Fossen 3-DOF + wind/wave/current | 1st-order inertia (simplified) |
| Sensors | 5 types (camera/LiDAR/IMU/GPS/depth) | 2D LiDAR only |
| Modes | Virtual simulation + **Digital Twin** | Virtual simulation only |
| Deployment | Unity + MATLAB + Docker(ROS), 3 processes | Single Python process |
| Algorithms | Any (via ROS interface) | DQN family (built-in) |
| Training speed | Slow (cross-process ROS communication) | Fast (in-process) |
| Modifiability | Low (Unity/C#/MATLAB required) | High (pure Python, edit YAML) |
| Ship deployment | Native ROS framework | Needs ROS wrapper |

## Historical Significance

> C4 platform served as a cognitive ladder in USV autonomous navigation research: from learning RL, to recognizing pure RL's limitations in industrial deployment, to the VLA+DRL hierarchical architecture -- a more viable path for real-world applications.
