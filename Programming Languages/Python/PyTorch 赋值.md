# PyTorch 赋值与求梯度

​	我有一次写 Recurrent Neural Network 时, 在训练集一个序列结束，下一个序列开始的位置强行为 hidden state 赋值, 导致梯度传播错误, 下面用简化的例子来分析.

​	先看一个简单的例子:

```python
import torch
from torch import nn

class foo(nn.Module):
    def __init__(self) -> None:
        super(foo, self).__init__()
        self.A = nn.Parameter(torch.tensor([[0., 1.], [2., 0.]]))
    
    def forward(self, x):
        return self.A @ x


if __name__ == '__main__':
    b = torch.tensor([1, -1], dtype=torch.float32)
    c = torch.ones(2, dtype=torch.float32)
    f = foo()
    loss = torch.sum(f(f(b - c) + c))
    print("loss: ", loss)
    opt = torch.optim.SGD(f.parameters(), lr=1.0)

    opt.zero_grad()
    loss.backward()
    opt.step()

    print("f.A: ", f.A)
```

这段代码的 loss 计算 $A(A(b-c)+c)$ 两维的和, 其中$$A=\left(\matrix{0, 1\\2, 0}\right), b=\left(\matrix{1\\-1}\right), c=\left(\matrix{1\\1}\right)$$.

会输出:

```bash
loss:  tensor(-1., grad_fn=<SumBackward0>)
f.A:  Parameter containing:
tensor([[1., 4.],
        [3., 1.]], requires_grad=True)
```

