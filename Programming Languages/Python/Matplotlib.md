# ![logo](https://matplotlib.org/_static/images/logo2.svg)

[TOC]



## 伪彩图

用 matplotlib 画伪彩图非常简单. 设 `data` 是一个二维的 `numpy.ndarray`, 要画其伪彩图, 只需要如下代码:

```python
sc = plt.imshow(data)
sc.set_cmap('coolwarm')
plt.colorbar(orientation='horizontal')
plt.show()
```

如果 `plt.colorbar` 不设置 ` orientation`, 颜色条默认是竖直的. 



## 坐标轴