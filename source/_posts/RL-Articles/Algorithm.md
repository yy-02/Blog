---
title: An Article about Double Q-PID Algorithm
categories:
  - RL-Articles
date: 2022-03-13 20:45:10
layout:
mathjax: true
tags:
- Double Q-PID
- Robot control
---

In article [*Double $Q$-PID algorithm for mobile robot control*](https://doi.org/10.1016/j.eswa.2019.06.066), the authors using Double $Q$-PID algorithm in PID controllers, which is similar with DQN and DDQN.
<!--more-->
# Knowledge before reading this article
## $Q$-learning
Reinforcement learning involves an agent, a set of states *__S__*, and a set *__A__* of actions per state. By performing an action *__a__* $\in$ *__A__* , the agent transitions from state to state. Executing an action in a specific state provides the agent with a reward (a numerical score).
  
We define function $Q$ to calculates the quality of a state-action combination. The core of $Q$-learning is *Bellman Equation*, which is used to update $Q$-Function in value iteration. 
$$Q^{new}(s_t,a_t)\leftarrow \underset{old\ value}{\underbrace{Q(s_t,a_t)}}+\underset{learning\ rate}{\underbrace{\alpha}}\overset{temporal\ difference}{\overbrace{[\underset{new\ value(temporal\ difference\ target)}{\underbrace{\underset{reward}{\underbrace{r_{t+1}}}+\underset{discount\ factor}{\underbrace{\gamma}}
{\underset{estimate\ of\ optimal\ future\ value}{\underbrace{\underset{a}{max}Q_B(s_{t+1},a_{t+1})}}}}} -\underset{old\ value}{\underbrace{Q(s_t,a_t)}}]}}$$

## DQN & DDQN
### DQN
DQN is an algorithm that combine deep learning with $Q$-learning. In traditional $Q$-learning, we usually use a matrix to save the value of $Q$ for each state-action combination(Or using function approximation in large questions). 

In DQN we have :

- A $Q$-estimation network
- A $Q$-target network
- An experience pool  

In a learning period, the weight of $Q$-target network is fixed and $Q$-estimation network is updated by the value of state-action combination in $Q$-target network. At the end of this period, the weight of $Q$-target network is updated by $Q$-estimation network.

__Disadvantage :__ In DQN the value of $Q$ will be overestimated, because $Q$-esitimation network is updated by $Q$-target network.
### DDQN
To solve the over estimation problem above, DDQN is proposed. The difference between DQN and DDQN is that, in DDQN action is selected by its value in $Q$-estimation network and then $Q$-target network will use this action to update the value of $Q$. 
# About this article
## Double Q-learning
This algorithm is very similar with DQN and DDQN. It also includes two neural networks, but they will be updated by each other. For example, if the policy uses the function $Q_A$ to choose the action the optimal action, then $Q_A$ will be updated as:
  
  $$Q_A(s_t,a_t)\leftarrow Q_A(s_t,a_t)+\alpha[r_{t+1}+\gamma\underset{a}{max} Q_B(s_{t+1},a_{t+1})-Q_A(s_t,a_t)]$$
  Alternatively, if the policy uses the function $Q_B$, then it will be updated as:
  $$Q_B(s_t,a_t)\leftarrow Q_B(s_t,a_t)+\alpha[r_{t+1}+\gamma\underset{a}{max} Q_A(s_{t+1},a_{t+1})-Q_B(s_t,a_t)]$$
  As the article says, "Since the update of $Q_A$ and $Q_B$ is done with its opposite, each update takes into consideration a different subset of the data set, thus removing the bias in the value estimation."
  ## Incremental discretization of the states and actions space
  In this article they presented the main ideas of incremental discretization of the state space and action space developed in [Carlucho et al.(2017)](https://doi.org/10.1016/J.ESWA.2017.03.002). Briefly speaking, at first they have a fairly coarse discretization of the action space $A$, then if action $a$ is selected it will be divided into two parts (or we can say it is a splitting process). As the discretization process going on, actions which have been chosen more will have higher discretization level.
  The final result represents the discretization of action space $A$.
  
  ![image](https://ars.els-cdn.com/content/image/1-s2.0-S0957417419304749-gr3_lrg.jpg)
  ## Double $Q$-PID algorithm 
  ### Incremental active learning
  Using discretization methods above, the agent can avoid unnecessary exploration of large regions, concentrating in a more relevant space. This method may be useful in working with autonomous mobile robots.

  ![](https://ars.els-cdn.com/content/image/1-s2.0-S0957417419304749-gr4_lrg.jpg)
  ### Algorithm statement
  The pseudocode is as follows. $x,k$ are separately referred to as *state* and *action*.

  ![](https://ars.els-cdn.com/content/image/1-s2.0-S0957417419304749-gr19_lrg.jpg)
  ## Some meaningful results

  - Double $Q$-PID algorithm had quickly learned to cancel the derivative terms, which means in the simulation PID controllers were equal to PI controllers.
  
  - Double $Q$-PID algorithm shows its ability in robot control, especially in balance control. It has strong robustness, as it only needs to provide an approximate initial coarse discretization of the action space. This advantage enables us to use it in different kinds of control models.