---
title: An article about DDPG&PI Controllers
date: 2022-03-18 21:30:00
mathjax: true
tags:
- DDPG
- PI
- Water tank system
categories:
  - RL-Articles
---

In article [*__On the Combination of PID control and Reinforcement Learning: A Case Study with Water Tank System__*](https://doi.org/10.1109/ICIEA51954.2021.9516140), DDPG is used to help PI controllers to control system (in this article the target is  maintaining the water level).
I will just paraphrase DDPG that written in article. But I think another blog should be written to fully explain DDPG since it contains many other algorithm about RL (On condition that I have completely understood DDPG).
<!--more-->
# The Model of Two-level Water Tank System
![imagine](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/9516033/9516034/9516140/9516140-fig-1-source-large.gif)
In picture above shows the basic structure of Two-level water tank system. For the $l$-th tank, the model is
$$h_l(t)=\frac{F_{inl}(t)-F_{outl}(t)}{A}$$
where $h_l$ is the height level of water, $A$ is the cross-sectional area of the tank, $F_{inl}$ and $F_{outl}$ represent the quantity of input and output water flow, respectively. And the whole system is gravity drained as their output water flow speeds are related to their own height of water level. Actually we can use second-order characteristics to solve this problem, but in RL training we just regard it as a black-box model.
# Algorithm
The authors used DDPG with traditional PI controllers to maintain the water level, since water tank system is a continuous action space. DDPG (Deep Deterministic Policy Gradient) is a model-free off-policy Actor-Critic algorithm, combining DPG (Deterministic Policy) with DQN (Deep Q-Network). A random process $N$ is added to policy function to encourage agent to explore. 

At each step, the actor takes action $a_t$ based on pre-action state $s_t$ under current policy $\mu_t$. The reward $r_t$ is calculated based on the post-action state $s_{t+1}$. The $(s_t,a_t,r_t,s_{t+1})$ transients will be stored into the replay buffer $R$ for training two networks. A random mini-batch of $N$ transitions $(s_i,a_i,r_i,s_{i+1})$ will be sampled from replay buffer $R$ to train the critic agent through minimizing the following loss:
$$L=\frac{\underset{i}{\sum}(y_i-Q(s_i,a_i|\theta ^Q))^2}{N}$$
where $y_i$ is the sum of the reward $r_i$ and $\gamma$-discounted expected long-term reward under current policy starting from the next time step:
$$y_i=r_i+\gamma Q'(s_{i+1},\mu'(s_{i+1}|\theta^{\mu '})|\theta ^{Q'})$$
And in the actor agent the policy will be updated by gradient as follows:
$$\nabla_{\theta\mu}J\approx\frac{\underset{i}{\sum}\nabla_aQ(s,a|\theta^Q)|_{s=s_i,a=\mu(s_i)}\nabla_{\theta\mu}\mu(s|\theta^\mu)|_{s_i}}{N}$$ 
For actor agent its parameter needs to use gradient ascent while critic agent needs to minimizing its loss function with gradient descent.
$$\theta^\mu=\theta^\mu+\alpha\nabla_{\theta\mu}J$$
$$\theta^Q=\theta^Q-\beta\nabla_{\theta Q}L$$
where $\alpha,\beta$ are  learning rates. At the end of each episode, the target networks will be soft-updated as follows:
$$\theta^{\mu'}\leftarrow\tau\theta^\mu+(1-\tau)\theta^{\mu'}$$
$$\theta^{Q'}\leftarrow\tau\theta^Q+(1-\tau)\theta^{Q'}$$
where $\tau$ is the learning target smooth factor. This is  an important difference between DQN and DDPG as the target network will be fixed during each episode in DQN, but in DDPG it will be updated dynamically, though very slowly (smoothly) due to $\tau\ll1$.

In water tank system, the pseudocode of DDPG is as follows:

![image|690x882](https://shuiyuan.sjtu.edu.cn/uploads/default/original/3X/6/6/669b3f0b8c7706c7eebb8cb757200bf4a3b1e3b1.jpeg)

The state is defined as:
$$state=\bigg [\int(l_d-l_c)dt, l_d-l_c, l_c\bigg]$$

And the reward function is defined as:
$$ reward=\begin{cases}
-|l_d-l_c| & 0\leq l_c\leq4 \\
-500 & else \\
\end{cases}$$ 
# Simulation Results
## DDPG or PI ONLY
![image](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/9516033/9516034/9516140/9516140-fig-3-source-small.gif)

Compared with single PI control, the transient response speed is improved by DDPG agent, and the largest overshoot amplitude is only 12.6%. But we can see that in the steady state, there exists a clear bias in single DDPG control result.
## DDPG+PI


![image](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/9516033/9516034/9516140/9516140-fig-5-source-large.gif)

With the guidance of PI controller, the training process of DDPG agent is accelerated. The overshoot amplitude is only half of the amplitude in solo PI control result. The bias of solo DDPG control also disappeared and the water level converges exactly to the desired value, which means the combination of them has advantages of both PI and DDPG.
## DDPG&PI (With Noise)
![image](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/9516033/9516034/9516140/9516140-fig-6-source-large.gif)

For solo PI control, the impact of noise is mainly showed in the oscillation state before exactly converging to the desire level. On the contrary, the noise affects more on the transient performance in solo DDPG control, and the bias still exists.

For DDPG+PI control, the noise have little influence on the whole control process, the combination has strong anti-disturbance ability. Even under noise the DDPG+PI agent's performance is still better than single PI or DDPG control without noise.
## Changing Desired Water Level
![image](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/9516033/9516034/9516140/9516140-fig-7-source-large.gif)

From my perspective, this result is the most interesting part in simulation results. First they switched the target value from 1.5$m$ to 2.5$m$, then after about 1000 seconds let the target value return to 1.5$m$. As the article said, at the first stage the PI+DDPG agent chose to follow PI controller, and we can see that they have an almost same overshoot amplitude in the figure. But at the final stage the PI+DDPG agent chose to follow DDPG control and shows an improved control performance compared with single PI control.
# Some of my opinions
The authors' explanation for the phenomenon above is that the DDPG+PI agent had learned in the first stage when in the final stage, so it will perform better. Since the authors haven't provide their source code about this article yet, we are not able to know the details of their simulation. If the result is true, why the DDPG+PI agent will choose to follow PI controller at first? In my opinion, in the first stage when the system is switched from a steady state to another state, the agent's response should be the same as what we have seen in DDPG+PI control (without noise) above, since the reward function doesn't pay attention to the absolute value but the relative value of water level.