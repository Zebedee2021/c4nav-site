# DQN Family: 4 Variants

## Progressive Architecture

```
DQN (Baseline) --> DDQN (+Double) --> D3QN (+Dueling) --> ID3QN (+PER +Shaping +SoftUpdate +LR)
```

## Network Structure (ID3QN / DuelingQNetwork)

```
Input: 184 floats (180 LiDAR + 4 nav)
  |
  v
[Linear] 184 -> 256 (47,360 params)
[ReLU]
  |
[Linear] 256 -> 256 (65,792 params)
[ReLU]
  |
  +--- Value Stream ---+--- Advantage Stream ---+
  |  [Linear] 256->128 |  [Linear] 256->128     |
  |  [ReLU]            |  [ReLU]                |
  |  [Linear] 128->1   |  [Linear] 128->5       |
  |  Output: V(s)      |  Output: A(s,a)        |
  +--------------------+------------------------+
              |
        Q(s,a) = V + (A - mean(A))
              |
        argmax --> Action (0-4)
```

Total parameters: ~180K. Model file size: ~2.8 MB (including optimizer state).

## Algorithm Switches

| Switch | DQN | DDQN | D3QN | ID3QN |
|--------|:---:|:----:|:----:|:-----:|
| `use_double_dqn` | - | Y | Y | Y |
| `network_type: dueling` | - | - | Y | Y |
| `use_per` | - | - | - | Y |
| `enable_shaping` | - | - | - | Y |
| `use_soft_update` | - | - | - | Y |
| `lr_schedule: step` | - | - | - | Y |
| `loss_fn: huber` | - | - | - | Y |

## Results

| Metric | DQN | DDQN | D3QN | **ID3QN** |
|--------|:---:|:----:|:----:|:---------:|
| Success Rate | 20.0% | 4.0% | 2.0% | **83.0%** |
| Avg Reward | -57.4 | -101.4 | -101.2 | **84.0** |
| Avg Steps | 275 | 115 | 163 | **86** |
| Convergence | - | - | - | **Ep 294** |
