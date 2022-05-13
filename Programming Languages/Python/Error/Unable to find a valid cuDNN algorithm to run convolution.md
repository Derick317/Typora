# Unable to find a valid cuDNN algorithm to run convolution

我在训练 `ManiSkill` 的时候, 当程序运行到

```python
policy_loss.backward()
```

的时候, 报了错：`RuntimeError: Unable to find a valid cuDNN algorithm to run convolution`.

我看网络上有人说是显卡版本的问题, 有人说是内存的问题. 总之我把 batchsize 减小后就没有这个问题了.