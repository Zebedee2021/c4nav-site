# Dynamics Model

## c4nav-core: 1st-Order Inertia Model

```python
# Turn dynamics
max_angular_accel = max_turn_rate * 3.0  # rad/s^2
turn_diff = clip(target_turn_rate - turn_rate, -max_angular_accel*dt, +max_angular_accel*dt)
turn_rate = (turn_rate + turn_diff) * turn_damping  # 0.90

# Speed dynamics
max_accel = 2.0  # m/s^2
speed_diff = clip(target_speed - speed, -max_accel*dt, +max_accel*dt)
speed = (speed + speed_diff) * speed_damping  # 0.95

# Position update
x += speed * cos(heading) * dt
y += speed * sin(heading) * dt
```

Simple, fast, but missing key hydrodynamics.

## SpaceR-USV: Fossen 3-DOF Model

$$M\dot{v} + C(v)v + D(v)v + g(\eta) = \tau$$

Where $v = [u, v, r]^T$ (surge, sway, yaw rate).

## Comparison

| Feature | 1st-Order Inertia | Fossen 3-DOF |
|---------|-------------------|--------------|
| DOF | 2 (surge, yaw) | 3 (surge, sway, yaw) |
| Sway (lateral drift) | No | Yes |
| Added mass | No | Yes |
| Coriolis forces | No | Yes |
| Nonlinear damping | Linear (damping=0.95) | Quadratic D(v) |
| Propeller model | Simplified target speed | Thrust-RPM curve |
| Wind/wave/current | No | Yes |
| Computation | ~0.01 ms/step | ~1 ms/step |

## Impact on Strategy

The physics gap matters most during **aggressive maneuvers**:

- **Straight line**: Both models produce similar behavior
- **Gentle turns**: Minor differences
- **Hard turns at speed**: Fossen model shows lateral drift (sway), larger actual turning radius
- **Emergency stop**: Fossen model has nonlinear deceleration

Strategies trained in c4nav-core tend to be **more aggressive** (closer passes, sharper turns) because the simplified model underestimates the consequences of extreme maneuvers.
