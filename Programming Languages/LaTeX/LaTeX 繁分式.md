# ${\rm \LaTeX}$: 繁分式

在 ${\rm \LaTeX}$ 中输入繁分式, 如果直接输入两个 `\frac` 就会非常丑陋. 例如, 如果我们输入:

```latex
\lambda_g=\frac{\lambda_0}{\sqrt{1-\left(\frac{\lambda_0}{2a}\right)^2}}
```

显示结果就像:
$$
\lambda_g=\frac{\lambda_0}{\sqrt{1-\left(\frac{\lambda_0}{2a}\right)^2}}
$$
分母中分式$\lambda_0$和$2a$的字号明显偏小. 如果加入 `\displaystyle`, 即输入:

```latex
\lambda_g=\frac{\lambda_0}{\sqrt{1-\left(\frac{\displaystyle\lambda_0}{\displaystyle2a}\right)^2}}
```

则效果如下:
$$
\lambda_g=\frac{\lambda_0}{\sqrt{1-\left(\frac{\displaystyle\lambda_0}{\displaystyle2a}\right)^2}}
$$
这里$\lambda_0$和$2a$的字号恢复正常了, 但它们离分数线过近, 仍然很别扭.

${\rm \LaTeX}$ 专门提供了繁分式的语法(我不知道这个词怎么表述) `\dfrac`. 如果输入



```latex
\lambda_g=\frac{\lambda_0}{\sqrt{1-\left(\dfrac{\lambda_0}{2a}\right)^2}}
```

则效果如下:
$$
\lambda_g=\frac{\lambda_0}{\sqrt{1-\left(\dfrac{\lambda_0}{2a}\right)^2}}
$$
这样就非常美观了.

似乎还可以把 `\dfrac` 换成 `\cfrac`, 在 XeLaTeX 2021 编译效果如下:

<img src="../../assets/LaTeX 繁分式/image-20220508170336701.png" alt="image-20220508170336701" style="zoom:80%;" />

