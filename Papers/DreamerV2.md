# Mastering Atari with Discrete World Models

[ICLR]: https://iclr.cc/virtual/2021/poster/2742
[GitHub]: https://github.com/danijar/dreamerv2

According to the paper, compared with dreamerV1, the following changes that were tried and were found to help:

* **Categorical latents**: Using categorical latent states using straight-through gradients in the
  world model instead of Gaussian latents with reparameterized gradients.
* **KL balancing**: Separately scaling the prior cross entropy and the posterior entropy in the KL
  loss to encourage learning an accurate temporal prior, instead of using free nats.
* **Reinforce only**: Reinforce gradients worked substantially better for Atari than dynamics backpropagation. For continuous control, dynamics backpropagation worked substantially better.
* **Model size**: Increasing the number of units or feature maps per layer of all model components,
  resulting in a change from 13M parameters to 22M parameters.
* **Policy entropy**: Regularizing the policy entropy for exploration both in imagination and during data collection, instead of using external action noise during data collection.

Changes that were tried but were found to not help substantially:

* **Binary latents**: Using a larger number of binary latents for the world model instead of categorical latents, which could have encouraged a more disentangled representation, was worse.
* **Long-term entropy**: Including the policy entropy into temporal-difference loss of the value
  function, so that the actor seeks out states with high action entropy beyond the planning horizon.
* **Mixed actor gradients**: Combining Reinforce and dynamics backpropagation gradients for
  learning the actor instead of Reinforce provided marginal or no benefits.
* **Scheduling**: Scheduling the learning rates, KL scale, actor entropy loss scale, and actor gradient  mixing (from 0.1 to 0) provided marginal or no benefits.
* **Layer norm**: Using layer normalization in the GRU that is used as part of the RSSM latent
  transition model, instead of no normalization, provided no or marginal benefits.

## Model Ensemble

​	In DreamerV2, the RSSM model is an ensemble, although the ensemble size is one by default. However, not networks in this model need to be an ensemble. Only the The output MLP of the `img_step` are ensembles. The output of each MLP refers to the distribution, either continuous or discrete, of the stochastic variables of the latent state. That is, it outputs `num_ensemble` distributions. Then, it chooses one distribution randomly.

## Target Network

According to the paper, this work stabilizes value learning using a target network, namely, it computes the targets using a copy of the critic that is updated every 100 gradient steps.

## Categorical Latents

The data structure of a Gaussian latent state is its mean and standard deviation, and a sample from the distribution. For a categorical latent state, this distribution is represented as `logit`, a tensor of shape: (the number of categories, the number of classes in each category). We can get the distribution by calling `tfp.distributions.OneHotCategorical(logit)`. Details of this function can be seen [here](https://tensorflow.google.cn/probability/api_docs/python/tfp/distributions/OneHotCategorical).
