---
typora-root-url: ..
---

# Dream to Control: Learning Behaviors by Latent Imagination

[ICLR 2020]: https://iclr.cc/virtual_2020/poster_S1lOTC4tDS.html
[GitHub]: https://github.com/google-research/dreamer	"TensorFlow 1"
[GitHub]: https://github.com/danijar/dreamer	"TensorFlow 2"



[TOC]

## World Model

### Encoder: Convolutional Neural Network

* Stride = 2
* Kernel: $4\times4$
* Channel: $3 \rightarrow 32 \rightarrow 64 \rightarrow 128 \rightarrow 256$
* No padding

### Decoder: Linear Function + Transpose Convolution Neural Network + Gaussian

Fully-connected layer:

* $? \rightarrow 1024 $
* No activation

Transposed CNN:

* Channel: $1024 \rightarrow 128 \rightarrow 64 \rightarrow 32 \rightarrow 3$
* Kernel: $5\times5\rightarrow5\times5\rightarrow6\times6\rightarrow6\times6$
* Stride = 1 except for the last layer, which is 2.

The output of the network is the mean of a diagonal Gaussian distribution, and its standard deviation is identity.

### Reward Predictor: Fully-Connected Layers + Gaussian

* Size of hidden layers: 400, 400
* Activation: ELU

The output of the network is the mean of a Gaussian distribution, and its standard deviation is identity.

### Value Function: Fully-Connected Layers + Gaussian

* Size of hidden layers: 400, 400, 400
* Activation: ELU

The output of the network is the mean of a Gaussian distribution, and its standard deviation is identity.

### Actor: Fully-Connected Layers + tanh-normal

Fully-connected layers:

* Size of hidden layers: 400, 400, 400, 400, 2 × ac_dim
* Activation: ELU

The output of the network splits into two vectors: $\mu, \sigma\in\mathbb{R}^{\rm ac\_dim}$ (`mean` and `std`). Then:

* $\mu\leftarrow s\cdot\tanh(\mu/s)$ ($s$ is called `mean_scale` and equals $5$ by default)
* $\sigma\leftarrow\ln(1+e^{\sigma+\tilde{\sigma}_0}) + \varepsilon$ ($f(x)=\ln(1+e^x)$ is called *soft-plus* function. $\varepsilon=10^{-4}$ by default. $\tilde{\sigma}_0 = \ln(e^{\sigma_0}-1)$ and $\sigma_0=5$ by default)

$\mu$ and $\sigma$ define a diagonal Gaussian distribution ${\rm a}\sim\mathcal{N}(\mu, \sigma)$ and the final action the policy predict is $a=\tanh({\rm a})$.

### Dynamics Model: Recurrent State Space Model

This work uses GRU Cell. And a rollout is shown as the following:

![](/assets/Dreamer/dreamer_test_time.svg)

## Policy

### Theory

​	According to the paper, we learn an [action model](###Actor: Fully Connected Layers + tanh-normal) and a [value model](###Value Function: Fully-Connected Layers + Gaussian) in the latent space of the world model. The action model implements the policy and aims to predict actions that solve the imagination environment. The value model estimates the expected imagined rewards that the action model achieves from each state $z_{\tau}$. The action model:
$$
\begin{aligned}
a_{\tau}\sim&\ q_\phi(a_\tau|z_\tau)\\
=&\ \tanh(\mu_{\phi}(z_\tau)+\sigma_{\phi}(z_\tau)\cdot\varepsilon), \varepsilon\sim\mathcal{N}(0,\mathbb{I}).
\end{aligned}
$$
The value model 
$$
v_\psi(z_\tau)\approx\mathrm{E}_{q(\cdot|z_\tau)}(\sum_{\tau=t}^{t+H} \gamma^{\tau-t} r_\tau)???
$$
 is a little complicated. we need to estimate the state values of imagined trajectories $\{z_\tau, a_\tau, r_\tau\}_{\tau=t}^{t+H}$. 

​	The vanilla value function is:
$$
\mathrm{V}_{\mathrm{R}}\left(z_{\tau}\right) \doteq \mathrm{E}_{q_{\theta}, q_{\phi}}\left(\sum_{n=\tau}^{t+H} r_{n}\right)
$$
it simply sums the rewards from $\tau$ until the horizon and ignores rewards beyond it. This allows learning the action model without a value model, an ablation we compare to in our experiments. $\mathrm{V_N^k}$, which is defined as  
$$
\mathrm{V}_{\mathrm{N}}^{k}\left(z_{\tau}\right) \doteq \mathrm{E}_{q_{\theta}, q_{\phi} }\left(\sum_{n=\tau}^{h-1} \gamma^{n-\tau} r_{n}+\gamma^{h-\tau} v_{\psi}\left(z_{h}\right)\right),\\ \text{with } h=\min (\tau+k, t+H),\text{ for }\tau=t, t+1,\cdots,t+H-1, k=1,2,\cdots,H
$$
estimates rewards beyond $k$ steps with the learned value model. Dreamer uses $\mathrm{V}_\lambda$, which is defined as
$$
\mathrm{V}_{\lambda}\left(z_{\tau}\right) \doteq(1-\lambda) \sum_{n=1}^{H-1} \lambda^{n-1} \mathrm{~V}_{\mathrm{N}}^{n}\left(z_{\tau}\right)+\lambda^{H-1} \mathrm{~V}_{\mathrm{N}}^{H}\left(z_{\tau}\right),
$$
an exponentially-weighted average of the estimates for different $k$ to balance bias and variance. In addition, this $\lambda$-target value function can also be defined recursively as follows:
$$
V_{\lambda}(z_{\tau})\doteq
\begin{cases}
r_{\tau}+\gamma\cdot[(1-\lambda)v_{\psi}(z_{\tau+1})+\lambda V_{\lambda}(z_{\tau+1})], \text{if }\tau-t<H-1\\
r_{\tau}+\gamma\cdot v_{\psi}(z_{t+H}), \text{if }\tau-t=H-1.
\end{cases}
$$
​	To update the action and value models, we first compute the value estimates $\mathrm{V}_{\lambda}(z_{\tau})$ for all states $z_\tau$ along the imagined trajectories. The objective for the action model $q_\phi(a_\tau|z_\tau)$ is to predict actions that result in state trajectories with high value estimates. The objective for the value model $v_\psi (z_\tau)$​, in turn, is to regress the value estimates (*I don't know why the value functions should be summed over*):
$$
\max _{\phi} \mathrm{E}_{q_{\theta}, q_{\phi}}\left(\sum_{\tau=t}^{t+H-1} \mathrm{~V}_{\lambda}\left(z_{\tau}\right)\right), \quad \quad \min _{\psi} \mathrm{E}_{q_{\theta}, q_{\phi}}\left( \sum_{\tau=t}^{t+H-1} \frac{1}{2}\| v_{\psi}(z_{\tau})-\mathrm{V}_{\lambda}(z_{\tau}) \|^{2} \right) .
$$

## Training Procedure

### Dynamics Model

### Actor

### Value function



## Code

### `RSSM.img_step`

​	**Input**: `prev_state` (the latent state) and `prev_action`.

​	**Output**: `prior`.

​	`prev_state` and `prior` all refer to the latent state, which contains two composition: stochastic variables (diagonal Gaussian) and deterministic variables. In reality, they are *dictionaries* of keys:

* `"mean"` ($\mu$): mean of the distribution of the stochastic variables;
* `"std"` ($\sigma$): standard deviation of the distribution of the stochastic variables;
* `"stoch"` ($s$): one sample from the distribution of the stochastic variables.
* `"deter"` ($h$): the deterministic variables.

​	The detailed function is shown as the following figure:

![](assets/Dreamer/dreamer_img_step.svg)

### `RSSM.obs_step` 

​	**Input**: `prev_state` (the latent state), `prev_action` and `embed` (the output of the `encoder`).

​	**Output**: `post` and `prior`.

​	`prev_state`, `post` and `prior` all refer to the latent state (see [`img_step`](###img_step) for the details about their structure). The function is shown as the following figure:

![](/assets/Dreamer/dreamer_obs_step-16590819858682.svg)

### `RSSM.observe`

**Input**:

* `embed`: $e^{(i)}_t\in\mathbb{R}^{\rm e\_dim}, i=1,2,\cdots,B, t=0, 1,\cdots,T-1$;
* `action`: $a^{(i)}_t\in\mathbb{R}^{\rm ac\_dim}, i=1,2,\cdots,B, t=0,1,\cdots,T-1$;
* `state`: a tuple including initial prior $\tilde{z}_0^{(i)}$ and posterior $z_0^{(i)}$, where $i=1,2,\cdots,B$.

**Output**:

* `post`: $z^{(i)}_t, i=1,2,\cdots,B, t=1,2,\cdots,T$;
* `prior`: $\tilde{z}^{(i)}_t, i=1,2,\cdots,B, t=1,2,\cdots,T$.

For each state in the batch, the function is shown as the following figure:

![](/assets/Dreamer/dreamer_observe-16590821684134.svg)

### `Dreamer._imagine_ahead`

**Input**: `post`: $z^{(i)}_t, i=1,2,\cdots,B, t=1,2,\cdots,T$

**Output**: Combination of the deterministic variables and stochastic variables of latent states $z^{(j)}_t$, $j=1, 2,\cdots,B\cdot T, t=1,2,\cdots,H$.

At first, reshape the inputs into $z^{(j)}_0, j=1, 2,\cdots,B\cdot T$. Then, for each latent state $z^{(j)}_0$, compute $z^{(j)}_t$, $ t=1,2,\cdots,H$ by `img_step` and the actor.

## Other Tricks

### Action Repeat

To reduce the planning horizon and provide a clearer learning signal to the model, this work repeats each action $R$ times, as common in reinforcement learning. It refers to

[Nature]: https://www.nature.com/articles/nature14236	"DQN"
[MPLR]: http://proceedings.mlr.press/v48/mniha16.html	"Asynchronous Methods for Deep Reinforcement Learning"

Related code is:

```python
def step(self, action):
    done = False
    total_reward = 0
    current_step = 0
    while current_step < self._amount and not done:
        obs, reward, done, info = self._env.step(action)
        total_reward += reward
        current_step += 1
    return obs, total_reward, done, info
```

 where `self._amount` is $R$, and by default $R=2$.
