# Physics-Informed Reinforcement Learning (PIRL)

## Four Injection Points

Physics information can be injected into 4 positions in the RL framework:

```
              +--------------------------+
              |     RL Training Loop     |
              |                          |
  +--------+  |  +---------+  obs S     |
  | Env    |--+--| Agent   |<--------   |
  |        |  |  | (network)|-------->  |
  | Trans  |<-+--| D:loss  |  action A  |
  | C:reward|  |  +---------+           |
  +--------+  |                          |
              +--------------------------+

A: State transition (dynamics fidelity)
B: Observation (what agent can see)
C: Reward (what behavior is encouraged)
D: Network/Loss (physics equations as constraints)
```

| Position | Method | Example |
|----------|--------|---------|
| **A** | More accurate physics model | Simulink Fossen model |
| **B** | Expose physics quantities in observation | Add accel, angular_accel |
| **C** | Physics-aware reward terms | Energy cost, jerk penalty |
| **D** | Physics equations in loss function | PINN-style PDE residual |

## Phase 2 Plan: B + C

Phase 2 implements positions B and C (lowest risk, smallest code change):

### B: Observation Expansion

| Config | Dimensions | Content |
|--------|:----------:|---------|
| Baseline (current) | 184 | 180 LiDAR + 4 nav |
| + velocity | 186 | + speed_norm + turn_rate_norm |
| + acceleration | 188 | + accel_norm + angular_accel_norm |
| Full physics | 190 | All of the above |

Code changes: `usv_dynamics.py` (expose accel in `get_state()`), `usv_env.py` (parameterize `_get_observation()`).

**Agent code: zero changes** (state_dim is already parameterized).

### C: Physics-Aware Reward

| Reward Term | Formula | Purpose |
|-------------|---------|---------|
| Energy cost | `-energy_scale * abs(accel)` | Penalize frequent speed changes |
| Jerk penalty | `-jerk_scale * abs(angular_accel)` | Penalize sharp turns |

Code changes: `reward.py` (add 2 terms), config YAML (add enable flags and scale params).

### Why Not D (PINN-style)?

DQN's training target is TD error minimization. Q-values are not physical quantities -- there is no PDE to constrain them. PINN-style physics loss has no mature paradigm in value-based RL. Position D could be explored as **auxiliary prediction heads** (multi-task learning), but this is a separate research contribution.

## Experiment Matrix

| ID | Algorithm | Observation | Physics Reward | Config |
|----|-----------|:-----------:|:--------------:|--------|
| A1 | ID3QN | 184D (baseline) | No | `improved_d3qn.yaml` |
| A2 | ID3QN | 190D (physics) | No | `pirl_obs_only.yaml` |
| A3 | ID3QN | 184D (baseline) | Yes | `pirl_reward_only.yaml` |
| A4 | ID3QN | 190D (physics) | Yes | `improved_d3qn_pirl.yaml` |

Metrics: success rate, avg reward, convergence episode, trajectory smoothness (curvature variance), energy consumption (cumulative acceleration).

## Two Complementary Paths

```
Path 1 (Phase 3): Improve environment fidelity
  Make A better -> Simulink Fossen model
  Agent learns physics implicitly through experience
  Cost: heavy, slow, expensive

Path 2 (Phase 2): Improve agent's physics awareness
  Make B and C better -> PIRL in lightweight env
  Agent learns physics explicitly through observation and reward
  Cost: light, fast, but physics may be inaccurate
```

**Phase 4 is the intersection**: A + B + C all contain physics. The key experiment is whether B+C provides additional value when A is already high-fidelity.
