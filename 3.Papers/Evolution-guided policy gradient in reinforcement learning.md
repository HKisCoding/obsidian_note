
- **PublishYear**::  
- **Author**:: Shauharda Khadka, Kagan Tumer
- **Link**:: 
- **Tags**:: #paper
- **Cite Key**:: [@khadka_evolution-guided_nodate]

### Abstract
```
Deep Reinforcement Learning (DRL) algorithms have been successfully applied to a range of challenging control tasks. However, these methods typically suffer from three core difﬁculties: temporal credit assignment with sparse rewards, lack of effective exploration, and brittle convergence properties that are extremely sensitive to hyperparameters. Collectively, these challenges severely limit the applicability of these approaches to real-world problems. Evolutionary Algorithms (EAs), a class of black box optimization techniques inspired by natural evolution, are well suited to address each of these three challenges. However, EAs typically suffer from high sample complexity and struggle to solve problems that require optimization of a large number of parameters. In this paper, we introduce Evolutionary Reinforcement Learning (ERL), a hybrid algorithm that leverages the population of an EA to provide diversiﬁed data to train an RL agent, and reinserts the RL agent into the EA population periodically to inject gradient information into the EA. ERL inherits EA’s ability of temporal credit assignment with a ﬁtness metric, effective exploration with a diverse set of policies, and stability of a population-based approach and complements it with off-policy DRL’s ability to leverage gradients for higher sample efﬁciency and faster learning. Experiments in a range of challenging continuous control benchmarks demonstrate that ERL signiﬁcantly outperforms prior DRL and EA methods.
```

### Notes

#### Drawback of RL:
- Temporal credit assignment with long time horizons and sparse rewards
- RL relies on exploration to find good policies
- Sensitive to the choice of their hyperparamaters.

#### Evolutionary algorithms
- The use of a fitness metric that consolidates returns across an entire episode makes EAs indifferent to the sparsity of reward distribution and robust to long time horizons.
- Inability to leverage powerful gradient descent methods which are at the core of the more sample-efficient DRL approaches.

#### Evolutionary Reinforcement Learning (ERL)
- Incorporates EA’s population-based approach to generate diverse experiences to train an RL agent
- Transfers the RL agent into the EA population periodically to inject gradient information into the EA
- Components: 
	- EA population approach: selection and mutation
	- Deep Deterministic Policy Gradient (DDPG)

#### Deep Deterministic Policy Gradient (DDPG)
- Model-free RL algorithm developed for working with continuous high dimensional actions spaces
- Actor-critic architecture.

#### EA population strategy:
- Each individual in the evolutionary algorithm defines a deep neural network
- Mutation represents random perturbations to the weights (genes) of these neural networks.

#### General flow of ERL:
- A population of actor networks is initialized with random weights
- One additional actor network $rlactor$ is initialized alongside a critic network.
- The population of actors ($rlactor$ excluded) are then evaluated in an episode of interaction with the environment.
- Fitness for each actor is computed as the cumulative sum of the reward that they receive over the timesteps in that episode.
- Selects a portion of the population for survival with probability commensurate on their relative fitness scores.
- Actors in the population are then probabilistically perturbed through mutation and crossover operations to create the next generation of actors
- A select portion of actors with the highest relative fitness are preserved as elites and are shielded from the mutation step.

#### EA -> RL:
- ERL learns from the experiences within episodes.
- Stores each actor’s experiences defined by the tuple (current state, action, next state, reward) in its replay buffer -> The critic samples a random minibatch from this replay buffer and uses it to update its parameters using gradient descent.
- The replay buffer has access to the experiences from the entire evolutionary population.

#### RL -> EA:
- the $rlactor$ network’s weights are copied into the evolving population of actors
- The process of infusing policy learned by the $rlactor$ into the population also serves to stabilize learning and make it more robust to deception.
---

