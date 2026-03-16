# Model Structure, Storage & Inference

## What is a "Model"?

The trained "model" is a neural network with ~180K floating-point numbers (weights and biases). After training, these weights encode the navigation strategy learned from hundreds of thousands of trial-and-error interactions.

## Storage Format

Models are saved as `.pth` files (PyTorch standard):

| File | Size | Contents |
|------|------|---------|
| `baseline_seed0/final.pth` | 1.8 MB | QNetwork + optimizer state |
| `improved_seed0/final.pth` | 2.8 MB | DuelingQNetwork + target network + optimizer + epsilon |

The 2.8 MB includes:

- Q-network weights (~0.7 MB)
- Target network weights (~0.7 MB, a copy)
- Optimizer state (~1.2 MB, Adam momentum/variance)
- Epsilon value (1 float)

For inference only, the Q-network weights (~0.7 MB) are sufficient.

## Loading and Running

```python
# Step 1: Create empty network (random weights)
agent = DQNAgent(state_dim=184, action_dim=5, config=train_cfg)

# Step 2: Load trained weights
agent.load("outputs/compare/improved_seed0/models/final.pth")

# Step 3: Run inference loop
state, info = env.reset()
for step in range(500):
    action = agent.select_action(state, training=False)  # inference here
    state, reward, terminated, truncated, info = env.step(action)
    if terminated or truncated:
        break
```

## What Happens During Inference

One inference call = one matrix multiplication chain:

```
184 input numbers
    x weight matrix --> 256 numbers
    x weight matrix --> 256 numbers
    --> V branch --> 1 number
    --> A branch --> 5 numbers
    Q = V + (A - mean(A)) --> 5 Q-values
    argmax --> action (0-4)
```

Time: < 1 millisecond.

## Training vs Inference

| | Training | Inference |
|---|---|---|
| What it does | Trial and error, adjust weights | Use fixed weights for decisions |
| Computation | Heavy (forward + backward + gradient update) | Light (forward only) |
| Duration | 25 min for 2000 episodes | < 1 ms per step |
| GPU needed | Recommended (speeds up training) | Not needed |
| Replay buffer | Yes (stores hundreds of thousands of experiences) | Not needed |

## Hardware Requirements for Inference

| Hardware | Can run inference? | Per-step latency |
|----------|--------------------|-----------------|
| RTX 4060 GPU | Yes | < 0.1 ms |
| Laptop CPU | Yes | < 1 ms |
| Raspberry Pi 4 | Yes | ~5 ms |
| Jetson Nano (ship embedded) | Yes | ~2 ms |

The environment simulates 0.1s real time per step. Inference takes < 1ms. Real-time operation is easily achievable on any hardware.
