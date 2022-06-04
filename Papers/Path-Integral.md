# Deep Dynamics Models for Learning Dexterous Manipulation

[PMLR]: http://proceedings.mlr.press/v100/nagabandi20a.html
[GitHub]: https://github.com/google-research/pddm

[TOC]

## Deep Dynamics Models

## Online Planning

### Cross Entropy Method

1. Initialize mean and variance: it sets mean to be zero (in fact, an zero array of shape: horizon * ac_dim), and variance to be 5. 

2. Sampling: it uses `scipy.stats.truncnorm`, with variants:
   $$
   \sigma_{constrained}^2 = \min \{\left(\frac{lb-\mu}{2}\right)^2, \left(\frac{ub-\mu}{2}\right)^2, sigma^2\}
   $$
   for cross entropy method, $lb=-1, ub=1$.

3. The loop stop not only if enough iterations, but also **if variance is very low**.

### Path Integral

#### Theory

From the paper, general update rule takes the following form for time step $t$, reward-weighting factor $\gamma$ (in its code, it uses `mppi_kappa`), and reward $R_k$ from each of the $N$​ predicted trajectories:
$$
\mu_{t}=\frac{\sum\limits_{k=0}^{N}e^{\gamma \cdot R_{k}}\cdot a_{t}^{(k)}}{\sum\limits_{j=0}^{N} e^{\gamma \cdot R_{j}}} \forall t \in\{0 \ldots H-1\}
$$
Given the iteratively updating mean distribution $\mu_t$ from above, we generate $N$ action sequences
$a^i_t = n^i_t + \mu_t$, where each noise sample $n^i_t$ is generated using filtering coefficient $\beta$​ as follows:
$$
\begin{aligned}
&u_{t}^{i} \sim \mathcal{N}(0, \Sigma), \forall i \in\{0 \ldots N-1\}, \ t \in\{0 \ldots H-1\} \\
&n_{t}^{i}=\beta \cdot u_{t}^{i}+(1-\beta) \cdot n_{t-1}^{i}, \text { where } n_{t<0}=0
\end{aligned}
$$

#### Code

1. There are two 

#### Default variables (for Baoding balls)

* `sample_velocity`: True
* `mppi_beta`: 0.6
* `mppi_mag_noise`: 0.9
* `horizon`: 7
* `num_control_samples`: 700

## Other Notes

### `pddm.utils.helper_funcs.turn_acs_into_acsK`

```python
def turn_acs_into_acsK(actions_taken_so_far, all_samples, K, N, horizon):

    """
    start with array, where each entry is (a_t)
    end with array, where each entry is (a_{t-(K-1)}..., a_{t-1}, a_{t})
    
    Args:
    	actions_taken_so_far: np.ndarray of shape (?, ac_dim) (We assume that ? >= K-1)
    	all_samples: np.ndarray of shpae (N, horizon, ac_dim)
    
    Return:
    	all_acs: np.ndarray of shape (N, horizon, K, ac_dim)
    """
```

If the input `all_samples` is 
$$
\left[
\begin{array}{c}
a^{(1)}_1 & a^{(1)}_2 & \cdots & a^{(1)}_H \\
a^{(2)}_1 & a^{(2)}_2 & \cdots & a^{(2)}_H \\
\vdots & \vdots &  &\vdots \\
a^{(N)}_1 & a^{(N)}_2 & \cdots & a^{(N)}_H
\end{array}
\right]
$$
where $a_{i,j}\in R^{ac\_dim}$ and `actions_taken_so_far` is $[\cdots, a_{-2}, a_{-1}, a_0]$, then the output is (for example, K=3)
$$
\left[
\begin{array}{c}
[a_{-1}, a_0, a^{(1)}_1] & [a_{0}, a_1, a^{(1)}_2] & \cdots & [a_{H-2}, a_{H-1}, a^{(1)}_H] \\
[a_{-1}, a_0, a^{(2)}_1] & [a_{0}, a_1, a^{(2)}_2] & \cdots & [a_{H-2}, a_{H-1}, a^{(2)}_H] \\
\vdots & \vdots &  &\vdots \\
[a_{-1}, a_0, a^{(N)}_1] & [a_{0}, a_1, a^{(N)}_2] & \cdots & [a_{H-2}, a_{H-1}, a^{(N)}_H]
\end{array}
\right]
$$

However, by default K=1. I do not know what the function of `K` is.

### `Dyn_Model.do_forward_sim`

```python
def do_forward_sim(self, states_true, actions_toPerform):
    """
    Args:
    	states_true: a list
    	actions_toPerform: np.ndarray of shape (N, horizon, K, ac_dim) 
    		(output of turn_acs_into_acsK)
    
    Return:
    	
    """
```

