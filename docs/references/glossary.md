# 术语表

| 术语 | 定义 |
|------|------|
| **USV** | 无人水面航行器 (Unmanned Surface Vehicle)，在水面上自主运行的船艇 |
| **DRL** | 深度强化学习 (Deep Reinforcement Learning)，使用深度神经网络作为函数逼近器的强化学习 |
| **DQN** | 深度 Q 网络 (Deep Q-Network)，使用神经网络逼近 Q 函数的基于价值的 RL 算法 |
| **DDQN** | 双重 DQN (Double DQN)，使用在线网络选择动作、目标网络评估价值，减少过估计 |
| **D3QN** | 对决双重 DQN (Dueling Double DQN)，将 Q 值分解为状态价值 V(s) 和优势函数 A(s,a) |
| **ID3QN** | 改进型 D3QN (Improved D3QN)，D3QN + PER + 奖励塑形 + 软目标更新 + 学习率调度 |
| **PPO** | 近端策略优化 (Proximal Policy Optimization)，基于策略梯度的 on-policy 算法 |
| **PER** | 优先经验回放 (Prioritized Experience Replay)，按 TD 误差优先采样经验 |
| **MDP** | 马尔可夫决策过程 (Markov Decision Process)，序贯决策的数学框架 |
| **Gymnasium** | Python RL 环境标准接口库（OpenAI Gym 的继任者） |
| **SB3** | Stable-Baselines3，兼容 Gymnasium 的主流 RL 算法库 |
| **ML-Agents** | Unity 机器学习工具包，用于在 Unity 环境中训练智能体 |
| **Fossen 模型** | 挪威学派船舶操纵模型，按物理属性分解力（M, C, D, g） |
| **MMG 模型** | 日本学派船舶操纵模型，按力的来源分解（船体、螺旋桨、舵） |
| **PIRL** | 物理信息强化学习 (Physics-Informed RL)，将物理知识融入 RL 训练 |
| **PINN** | 物理信息神经网络 (Physics-Informed Neural Networks)，以 PDE 残差作为损失约束的神经网络 |
| **Sim-to-Real** | 仿真到实船迁移，将仿真中训练的策略部署到真实环境 |
| **域随机化** | Domain Randomization，训练时随机化仿真参数以提升策略鲁棒性 |
| **ROS** | 机器人操作系统 (Robot Operating System)，机器人软件开发中间件 |
| **具身空间** | Embodied Space，统一建模空间：E = (S, A, T, C, P)，整合状态、动作、任务、约束和物理环境 |
| **GEM** | 几何-环境-任务 (Geometry-Environment-Mission)，具身空间的三元结构模型 |
| **VLA** | 视觉-语言-动作模型 (Vision-Language-Action)，融合视觉理解、语言推理与动作生成的基础模型 |
| **COLREGS** | 国际海上避碰规则 (Convention on the International Regulations for Preventing Collisions at Sea) |
| **数字孪生** | Digital Twin，物理系统的虚拟副本，实时同步 |
| **CPS** | 信息物理系统 (Cyber-Physical Systems)，计算与物理过程的融合 |
