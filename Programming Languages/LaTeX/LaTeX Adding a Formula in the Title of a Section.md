# $\LaTeX$: 在标题中插入公式 

当我们需要在章节标题中插入数学公式时, 有时可以直接暴力地在 `\section{}` 或者 `\subsection{}` 的 `{}` 内输入 `$...$` 实现. 但有时可能会报错.

如果我们本想输入: 回磁比$\gamma$, $g$因子及弛豫时间$\tau$, 则在标题中输入`回磁比\texorpdfstring{$\gamma$}{TEXT}, \texorpdfstring{$g$}{TEXT}因子及弛豫时间\texorpdfstring{$\tau$}{TEXT}` 即可.
