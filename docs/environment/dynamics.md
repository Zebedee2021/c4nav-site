# 动力学模型

## c4nav-core：一阶惯性模型

```python
# 转向动力学
max_angular_accel = max_turn_rate * 3.0  # rad/s^2
turn_diff = clip(target_turn_rate - turn_rate, -max_angular_accel*dt, +max_angular_accel*dt)
turn_rate = (turn_rate + turn_diff) * turn_damping  # 0.90

# 速度动力学
max_accel = 2.0  # m/s^2
speed_diff = clip(target_speed - speed, -max_accel*dt, +max_accel*dt)
speed = (speed + speed_diff) * speed_damping  # 0.95

# 位置更新
x += speed * cos(heading) * dt
y += speed * sin(heading) * dt
```

简单、快速，但缺少关键水动力学效应。

## SpaceR-USV：Fossen 三自由度模型

$$M\dot{v} + C(v)v + D(v)v + g(\eta) = \tau$$

其中 $v = [u, v, r]^T$（纵荡、横荡、艏摇角速度）。

## 对比

| 特性 | 一阶惯性 | Fossen 三自由度 |
|------|---------|----------------|
| 自由度 | 2（纵荡、艏摇） | 3（纵荡、横荡、艏摇） |
| 横荡（侧向漂移） | 无 | 有 |
| 附加质量 | 无 | 有 |
| 科里奥利力 | 无 | 有 |
| 非线性阻尼 | 线性（damping=0.95） | 二次型 D(v) |
| 螺旋桨模型 | 简化目标速度 | 推力-转速曲线 |
| 风/浪/流扰动 | 无 | 有 |
| 计算耗时 | ~0.01 ms/步 | ~1 ms/步 |

## 对策略的影响

物理差距在**激进机动**时影响最大：

- **直线航行**：两种模型行为相似
- **缓慢转向**：差异较小
- **高速急转**：Fossen 模型会产生横荡（侧向漂移），实际转弯半径更大
- **紧急停车**：Fossen 模型有非线性减速

在 c4nav-core 中训练出的策略倾向于**更激进**（更近距离通过障碍物、更急的转弯），因为简化模型低估了极端机动的后果。
