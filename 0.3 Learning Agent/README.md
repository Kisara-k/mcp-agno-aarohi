# Learning Agent Tutorial - Q-Learning Snake Game

## Overview

This tutorial demonstrates the implementation of a **Learning Agent** using Q-Learning in a Snake game environment. Unlike reflex and goal-based agents, learning agents can improve their performance over time through experience and feedback from the environment.

## Learning Objectives

By completing this tutorial, you will:

- Understand the concept of learning agents and reinforcement learning
- Learn how to implement Q-Learning algorithm from scratch
- Explore the balance between exploration and exploitation
- Visualize learning progress in real-time
- Compare learning agents with other agent types

## What is a Learning Agent?

A Learning Agent consists of four main components:

1. **Learning Element**: Updates knowledge based on feedback
2. **Performance Element**: Makes decisions based on current knowledge
3. **Critic**: Evaluates performance and provides feedback
4. **Problem Generator**: Suggests exploratory actions for learning

In this Q-Learning implementation:

- **State**: Snake position, food position, and environment
- **Actions**: Move Up, Down, Left, Right
- **Rewards**: +10 for eating food, -10 for collision, -1 for each step
- **Q-Table**: Stores action-value pairs for decision making

## Getting Started

### Prerequisites

```bash
pip install pygame numpy
```

### Project Structure

```
0.3 Learning Agent/
├── demo.py                     # Q-Learning Snake implementation
└── README.md                   # This tutorial
```

## How to Run

```bash
python demo.py
```

**Controls:**

- The agent learns automatically (no human input required)
- Watch the learning progress in real-time
- Close the window to stop the simulation

## Implementation Details

### 1. Environment Setup

**Grid World**: 5×5 grid for simplified learning

- **Snake**: Starts at center position (2,2)
- **Food**: Randomly placed on the grid
- **Walls**: Grid boundaries cause collisions

```python
WIDTH = 300
HEIGHT = 300
GRID_SIZE = 5
CELL_SIZE = WIDTH // GRID_SIZE
```

### 2. Q-Learning Algorithm

**Core Concept**: Learn optimal action-value function Q(s,a)

```python
# Q-Learning Update Rule
Q[state][action] = Q[state][action] + α * (reward + γ * max(Q[next_state]) - Q[state][action])
```

**Parameters:**

- **α (Learning Rate)**: 0.1 - How much to update Q-values
- **γ (Discount Factor)**: 0.9 - Importance of future rewards
- **ε (Epsilon)**: 0.1 - Exploration vs exploitation balance

### 3. State Representation

States are defined by key environmental features:

- Snake head position (x, y)
- Food position relative to snake
- Collision detection in each direction

### 4. Reward System

```python
# Reward structure
+10   # Successfully eating food
-10   # Collision with wall or self
-1    # Each step (encourages efficiency)
```

### 5. Exploration Strategy

**ε-Greedy Policy:**

- **90%** of the time: Choose best known action (exploitation)
- **10%** of the time: Choose random action (exploration)

```python
if random.random() < self.epsilon:
    action = random.choice(ACTIONS)  # Explore
else:
    action = self.get_best_action(state)  # Exploit
```

## Key Concepts Demonstrated

### 1. **Reinforcement Learning**

- Learning through trial and error
- Reward-based feedback system
- Temporal difference learning

### 2. **Exploration vs. Exploitation**

- Balance between trying new actions and using known good actions
- ε-greedy strategy implementation
- Adaptive learning behavior

### 3. **Value Function Learning**

- Q-table for storing action values
- Bellman equation for optimal policy
- Convergence to optimal behavior

### 4. **Real-time Learning**

- Online learning during gameplay
- Immediate feedback and adaptation
- Performance improvement over time

## Expected Learning Progression

### Phase 1: Random Exploration (Episodes 0-50)

- Agent moves randomly
- Poor performance
- Q-table initialization

### Phase 2: Pattern Recognition (Episodes 50-200)

- Agent starts recognizing good/bad actions
- Gradual performance improvement
- Q-values stabilization

### Phase 3: Optimization (Episodes 200+)

- Near-optimal behavior
- Consistent food collection
- Efficient path planning

## Comparison: Different Agent Types

| Aspect           | Reflex Agent | Goal-Based Agent | Learning Agent              |
| ---------------- | ------------ | ---------------- | --------------------------- |
| **Adaptability** | Fixed rules  | Fixed goals      | Adaptive through experience |
| **Performance**  | Static       | Static           | Improves over time          |
| **Knowledge**    | Hardcoded    | Programmed       | Learned from environment    |
| **Flexibility**  | Limited      | Moderate         | High                        |
| **Complexity**   | Low          | Medium           | High                        |

## Advanced Features

### 1. **Dynamic Learning Rate**

Implement decreasing learning rate over time:

```python
self.alpha = max(0.01, self.alpha * 0.995)  # Decay learning rate
```

### 2. **Experience Replay**

Store and replay past experiences for better learning:

```python
self.memory = deque(maxlen=10000)  # Experience buffer
```

### 3. **Deep Q-Learning (DQN)**

Replace Q-table with neural network for complex state spaces.

## Exercises

### Beginner

1. Modify the reward system (change values for different actions)
2. Adjust learning parameters (α, γ, ε) and observe effects
3. Add performance tracking (score over episodes)

### Intermediate

1. Implement different exploration strategies (UCB, Thompson Sampling)
2. Add obstacles to the environment
3. Create a larger grid size and observe learning time

### Advanced

1. Implement Double Q-Learning to reduce overestimation bias
2. Add multi-objective rewards (speed, efficiency, safety)
3. Compare Q-Learning with other RL algorithms (SARSA, TD(λ))

## Performance Metrics

Track these metrics to evaluate learning:

### 1. **Score Progression**

- Average score per episode
- Moving average trends
- Learning curve analysis

### 2. **Convergence Metrics**

- Q-value stability
- Policy convergence
- Exploration rate decay

### 3. **Efficiency Measures**

- Steps per food collected
- Success rate over time
- Optimal path length

## Understanding Q-Learning

### The Q-Function

Q(s,a) represents the expected cumulative reward of taking action 'a' in state 's' and following the optimal policy thereafter.

### Bellman Equation

The optimal Q-function satisfies:

```
Q*(s,a) = E[r + γ * max Q*(s',a') | s,a]
```

### Temporal Difference Error

The difference between predicted and actual rewards:

```
TD_error = r + γ * max Q(s',a') - Q(s,a)
```

## Next Steps

After mastering learning agents, explore:

1. **Deep Reinforcement Learning** (DQN, PPO, A3C)
2. **Multi-Agent Reinforcement Learning**
3. **Agno Framework** (see `1 Agno/`)
4. **Advanced RL techniques** (Actor-Critic, Policy Gradients)

## Further Reading

- Sutton, R. & Barto, A. "Reinforcement Learning: An Introduction"
- [OpenAI Spinning Up in Deep RL](https://spinningup.openai.com/)
- [Q-Learning Tutorial](https://en.wikipedia.org/wiki/Q-learning)
- [Reinforcement Learning Algorithms](https://github.com/dennybritz/reinforcement-learning)

## Research Extensions

### 1. **Multi-Agent Snake**

- Multiple snakes learning simultaneously
- Cooperative vs competitive scenarios
- Social learning mechanisms

### 2. **Transfer Learning**

- Train agent on simple grid, test on complex environment
- Domain adaptation techniques
- Meta-learning approaches

### 3. **Hierarchical RL**

- Break down snake game into sub-tasks
- Learn high-level strategies
- Temporal abstraction

---

**Note**: Learning agents represent the pinnacle of adaptability in AI, capable of improving performance through experience. This Q-Learning implementation provides a foundation for understanding more complex reinforcement learning algorithms used in modern AI systems.
