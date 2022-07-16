# Deep Dynamics Models for Learning Dexterous Manipulation

[PMLR]: http://proceedings.mlr.press/v100/nagabandi20a.html
[GitHub]: https://github.com/google-research/pddm

[TOC]

## Deep Dynamics Models

### Structure

This work implements the dynamics model as a neural network of 2 fully-connected hidden layers  with ReLU (Rectified Linear Unit) nonlinearities and a final fully-connected output layer.

### Ensemble

For estimating uncertainty, this work uses ensemble of size 3.

### Training and Validation

When training the model, the supervising tuples are from the [data set $\mathcal{D}$](##Data-Set-D$). 

Firstly, concatenate on-policy set and random set. Then, do the following iteratively to train the model:

* Shuffle the dataset (denote the number of training set as $N$, and the size of batch as $B$), and cut them into $N/B$ batches.
* For each batch, calculate loss and back propagate.
* For every 10 iteration, calculate the loss on the random and on-policy validation set.

Note that the capacity of its dataset is infinity, and all data would be used once in one iteration mentioned above.

## Online Planning

### Cross Entropy Method

1. Initialize mean and variance: it sets mean to be zero (in fact, an zero array of shape: horizon * ac_dim), and variance to be 5. 

2. Sampling: it uses `scipy.stats.truncnorm`, with variants:
   $$
   \sigma_{constrained}^2 = \min \{\left(\frac{lb-\mu}{2}\right)^2, \left(\frac{ub-\mu}{2}\right)^2, \sigma^2\}
   $$
   for cross entropy method, $lb=-1, ub=1$.

3. The loop stop not only if enough iterations, but also **if variance is very low**.

### Path Integral

#### Theory

From the paper, general update rule takes the following form for time step $t$, reward-weighting factor $\gamma$ (in its code, it uses `mppi_kappa`), and reward $R_k$ from each of the $N$ predicted trajectories:
$$
\mu_{t}=\frac{\sum\limits_{k=0}^{N}e^{\gamma \cdot R_{k}}\cdot a_{t}^{(k)}}{\sum\limits_{j=0}^{N} e^{\gamma \cdot R_{j}}}, \forall t \in\{0 \ldots H-1\}
$$
Given the iteratively updating mean distribution $\mu_t$ from above, we generate $N$ action sequences
$a^i_t = n_t + \mu^i_t$, where each noise sample $n^i_t$ is generated using filtering coefficient $\beta$ as follows:
$$
\begin{aligned}
&u_{t}^{i} \sim \mathcal{N}(0, \Sigma), \forall i \in\{0 \ldots N-1\}, \ t \in\{0 \ldots H-1\} \\
&n_{t}^{i}=\beta \cdot u_{t}^{i}+(1-\beta) \cdot n_{t-1}^{i}, \text { where } n_{t<0}=0
\end{aligned}
$$

#### Code

* Initialize $n_t\in \mathbb{R}^{ac\_dim}, t\in\{0,1,\cdots,H-1\}$ (mean) to be a zero vector at the very beginning.

* When calling the policy to get the action at every time step:

  * $a_{\text{past}}\leftarrow n_0$, $n_t\leftarrow n_{t+1}(t=1,2,\cdots,H-2)$;

  * Initialize $\mu_t^i\in \mathbb{R}^{ac\_dim},i\in\{1,2,\cdots,N\}, t\in\{0,1,\cdots,H-1\}$ according to `sample_velocity`

  * Calculate candidate actions $a^i_t\in \mathbb{R}^{ac\_dim}, i\in\{1,2,\cdots,N\}, t\in\{0,1,\cdots,H-1\}$: 
    $$
    a^i_0=\beta\cdot(n_0+\mu^i_0)+(1-\beta)\cdot a_{\text{past}},\\
    a^i_t=\beta\cdot(n_t+\mu^i_t)+(1-\beta)\cdot a^i_{t-1}, \text{ for } t=1,2,\cdots,H-1
    $$

  * Do some process (for example, constrain the actions between $[-1,1]$)

  * Forward the actions and calculate their $R^i,i\in\{1,2,\cdots,N\}$;

  * Calculate weight $w^i=e^{\gamma \cdot R_{i}}/\sum\limits_{j=1}^{N} e^{\gamma \cdot R_{j}}$;

  * $n_t\leftarrow \sum\limits_{i=1}^N a^i_t\cdot w^i, \text{ for } t=0,1,\cdots,H-1$ 

  * return $n_0$

#### Default variables (for Baoding balls)

* `sample_velocity`: `True`. If false, the noise (`eps`, $\mu_t^i$) follow $\mathcal{N}(0, \sigma)$; if true, 10% of the noise follow  $\mathcal{N}(0, 0.3\sigma)$, others follow $\mathcal{N}(0, \sigma)$.
* `mppi_beta`: 0.6
* `mppi_mag_noise`: 0.9
* `sigma`: `mppi_mag_noise`
* `horizon`: 7
* `num_control_samples`: 700

## Data-Set $\mathcal{D}$

Although all data is used to train the model, for its policy is non-parametrized, this work uses 4 buffers:

* Random training set 
* On-policy training set 
* Random validation set 
* On-policy validation set 

Each data set saves two forms of data: rollout (`rollouts_trainRand`, `rollouts_trainOnPol`, `rollouts_valRand` and `rollouts_valOnPol`) and processed data (`dataset_trainRand`, `dataset_trainOnPol`, `dataset_valRand` and `dataset_valOnPol`).

### Initialization

​	Initially, `rollouts_trainOnPol` is empty, but `rollouts_trainRand` and `rollouts_valRand` can be initialized by performing random rollouts or can be loaded somewhere. `rollouts_valOnPol` is initialized to be the same as the initial `rollouts_valRand`.

​	`dataset_trainRand` and `dataset_valRand` are converted from `rollouts_trainRand` and `rollouts_valRand`

### Updating

​	For every iteration during training the model, this work does the following thing in turn.

​	Calculate `dataset_trainOnPol` and `dataset_valOnPol` from `rollouts_trainOnPol` and `rollouts_valOnPol` (see [here](###Processing-(Convert-Rollout-to-Data-Set))).

​	Calculate mean and standard deviation over all transition tuples of the training set (including random set and on-policy set): $\mu_X\in\mathbb{R}^{s\_dim}$, $\mu_Y\in\mathbb{R}^{ac\_dim}$, $\mu_Z\in\mathbb{R}^{s\_dim}$,  $\sigma_X\in\mathbb{R}^{s\_dim}$, $\sigma_Y\in\mathbb{R}^{ac\_dim}$, $\sigma_Z\in\mathbb{R}^{s\_dim}$.

​	Normalize the `dataset`:
$$
\begin{aligned}
X_{\text{trainRand}}&\leftarrow (X_{\text{trainRand}}-\mu_X)/\sigma_X \\
X_{\text{valRand}}&\leftarrow (X_{\text{valRand}}-\mu_X)/\sigma_X \\
X_{\text{trainOnPol}}&\leftarrow (X_{\text{trainOnPol}}-\mu_X)/\sigma_X \\
X_{\text{valOnPol}}&\leftarrow (X_{\text{valOnPol}}-\mu_X)/\sigma_X
\end{aligned}
$$
and so as $Y$ and $Z$.

​	Calculate training tuple (input: $X$ and $Y$, output: $Z$) for `dataset_trainRand`, `dataset_trainOnPol`, `dataset_valRand` and `dataset_valOnPol`.

​	Train the model.

​	Using the model to performs rollouts. 90% of these rollouts are added to `rollouts_trainOnPol`, and the remains are added to `rollouts_valOnPol`.

​	Perform 5 rollouts randomly, convert them to dataset and added them to `dataset_trainRand`.

### Processing (Convert Rollout to Data-Set)

​	A rollout has its states $s_t$ and actions $a_t$ ($t=0,1,\cdots,T$). A processed dataset is composed of 3 kinds of data: `dataX`, `dataY` and `dataZ`. $X_t=s_t$, $Y_t=a_t$, $Z_t=s_{t+1}-s_{t}$. The data of all rollouts are concatenated  together.

​	After calculate `dataX`, `dataY` and `dataZ`, some noise may be added to `dataX`, and `dataZ` to help model training. For instance, let's suppose that the `dataX` is of shape: `(data, dim)`, its `meanX` ($\mu$) over all states is of shape `(dim,)`. If a certain dimension of `meanX` is zero, we just set it to be $10^{-6}$. Then each dimension of the data is changed as:
$$
X_{tj}\leftarrow X_{tj}+\varepsilon, \text{ where } \varepsilon\sim\mathcal{N}(0, 0.01\mu_j),\ j=1,2,\cdots,D.
$$


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

