池化层也叫**汇聚层**。它通常放在卷积层之后，用一个小窗口在特征图上滑动，并把窗口内的多个数压缩成一个数。

最常见的池化方法有：

- **最大池化（Max Pooling）**：取窗口内的最大值；
- **平均池化（Average Pooling）**：取窗口内的平均值。

池化层的主要目标不是学习新的特征，而是对已有特征进行汇总和降采样。

## 1. 池化层的基本概念

假设输入特征图为：

$$
X=
\begin{bmatrix}
1&2&3&4\\
5&6&7&8\\
9&10&11&12\\
13&14&15&16
\end{bmatrix}
$$

使用大小为 `2 × 2`、步幅为 `2` 的池化窗口：

```text
┌─────┬─────┐
│ 1 2 │ 3 4 │
│ 5 6 │ 7 8 │
├─────┼─────┤
│9 10 │11 12│
│13 14│15 16│
└─────┴─────┘
```

每个 `2 × 2` 区域会被汇总成一个数，所以输入从 `4 × 4` 变成 `2 × 2`。

池化层也有两个重要参数：

- **池化窗口大小（kernel size）**：每次查看多大的局部区域；
- **步幅（stride）**：窗口每次移动多少格。

与卷积不同，标准最大池化和平均池化通常没有需要训练的权重，也不会在不同通道之间混合信息。

## 2. 最大池化

最大池化取每个局部窗口中的最大值。

对上面的输入使用 `2 × 2` 最大池化：

$$
\begin{bmatrix}1&2\\5&6\end{bmatrix}\longrightarrow 6,
\qquad
\begin{bmatrix}3&4\\7&8\end{bmatrix}\longrightarrow 8
$$

$$
\begin{bmatrix}9&10\\13&14\end{bmatrix}\longrightarrow 14,
\qquad
\begin{bmatrix}11&12\\15&16\end{bmatrix}\longrightarrow 16
$$

因此输出为：

$$
Y_{max}=
\begin{bmatrix}
6&8\\
14&16
\end{bmatrix}
$$

### PyTorch 代码

PyTorch 的二维池化输入形状通常为：

```text
批量大小 × 通道数 × 高度 × 宽度
```

因此需要给二维矩阵补上批量维度和通道维度：

```python
import torch
import torch.nn.functional as F

X = torch.tensor([[[
    [1.0,  2.0,  3.0,  4.0],
    [5.0,  6.0,  7.0,  8.0],
    [9.0, 10.0, 11.0, 12.0],
    [13.0, 14.0, 15.0, 16.0],
]]])

Y = F.max_pool2d(X, kernel_size=2, stride=2)

print(X.shape)  # torch.Size([1, 1, 4, 4])
print(Y.shape)  # torch.Size([1, 1, 2, 2])
print(Y)
# tensor([[[[ 6.,  8.],
#           [14., 16.]]]])
```

也可以把最大池化定义成网络层：

```python
import torch.nn as nn

pool = nn.MaxPool2d(kernel_size=2, stride=2)
Y = pool(X)
```

### 最大池化保留了什么？

最大池化倾向于保留窗口中响应最强的特征。

如果某个卷积核负责检测边缘，那么较大的特征值可以理解为：“这个位置附近很可能出现了目标边缘。”最大池化会保留这个最强响应，而忽略同一区域内较弱的响应。

因此，最大池化常用于回答：

> 这个局部区域中，是否出现了某种明显特征？

## 3. 平均池化

平均池化计算每个局部窗口中所有数的平均值。

对同一个输入使用 `2 × 2` 平均池化：

$$
\frac{1+2+5+6}{4}=3.5,
\qquad
\frac{3+4+7+8}{4}=5.5
$$

$$
\frac{9+10+13+14}{4}=11.5,
\qquad
\frac{11+12+15+16}{4}=13.5
$$

因此输出为：

$$
Y_{avg}=
\begin{bmatrix}
3.5&5.5\\
11.5&13.5
\end{bmatrix}
$$

### PyTorch 代码

```python
import torch
import torch.nn.functional as F

X = torch.tensor([[[
    [1.0,  2.0,  3.0,  4.0],
    [5.0,  6.0,  7.0,  8.0],
    [9.0, 10.0, 11.0, 12.0],
    [13.0, 14.0, 15.0, 16.0],
]]])

Y = F.avg_pool2d(X, kernel_size=2, stride=2)

print(Y)
# tensor([[[[ 3.5000,  5.5000],
#           [11.5000, 13.5000]]]])
```

也可以定义为网络层：

```python
import torch.nn as nn

pool = nn.AvgPool2d(kernel_size=2, stride=2)
Y = pool(X)
```

### 平均池化保留了什么？

平均池化保留的是区域内的整体平均响应。与最大池化相比，它不会只关注最强的一个值，而会综合窗口内的所有信息，因此结果通常更加平滑。

平均池化更接近回答：

> 这个局部区域中，某种特征的平均强度是多少？

## 4. 最大池化与平均池化的区别

| 对比项 | 最大池化 | 平均池化 |
|---|---|---|
| 计算方式 | 取窗口最大值 | 取窗口平均值 |
| 强调的信息 | 最强、最明显的响应 | 整体、平均的响应 |
| 对突出特征 | 更敏感 | 相对温和 |
| 输出效果 | 更容易保留显著特征 | 更加平滑 |
| 常见用途 | CNN 中的局部降采样 | 平滑特征、全局平均池化 |

下面用同一个输入直接比较两种结果：

```python
max_result = F.max_pool2d(X, kernel_size=2, stride=2)
avg_result = F.avg_pool2d(X, kernel_size=2, stride=2)

print(max_result)
# tensor([[[[ 6.,  8.],
#           [14., 16.]]]])

print(avg_result)
# tensor([[[[ 3.5000,  5.5000],
#           [11.5000, 13.5000]]]])
```

## 5. 池化层的作用

### 5.1 降低特征图的空间尺寸

使用 `2 × 2` 窗口和步幅 `2` 时，特征图的高度和宽度通常各缩小一半：

```text
输入：32 × 32
  ↓ 2 × 2 池化，stride=2
输出：16 × 16
```

空间尺寸变小后，后续网络需要处理的数据量也会减少。

```python
X = torch.randn(8, 64, 32, 32)
Y = F.max_pool2d(X, kernel_size=2, stride=2)

print(X.shape)  # torch.Size([8, 64, 32, 32])
print(Y.shape)  # torch.Size([8, 64, 16, 16])
```

注意，池化通常只改变高度和宽度，不改变批量大小和通道数。

### 5.2 减少计算量和内存占用

假设池化把 `32 × 32` 的特征图变成 `16 × 16`，空间位置数量会从：

$$
32\times32=1024
$$

减少到：

$$
16\times16=256
$$

只剩原来的四分之一。后续卷积层需要处理的位置更少，因此计算量和中间特征所占内存也会下降。

需要注意：标准池化层本身没有可训练参数，所以它减少的主要是**后续计算量和特征图内存**，而不是直接减少池化层自身的参数。

### 5.3 扩大后续神经元的有效感受野

池化后，一个输出位置汇总了输入中的一个局部区域。后续卷积层在较小的特征图上计算时，一个位置对应到原图中的范围会更大。

简单理解：

```text
池化前：一个特征值描述一个较小区域
池化后：一个特征值概括一个更大的区域
```

这有助于更深层网络逐渐从边缘等局部特征，过渡到物体部件和整体结构。

### 5.4 提供一定的平移鲁棒性

假设强响应在一个 `2 × 2` 窗口内稍微移动：

```text
移动前          移动后
1  9            9  1
2  3            2  3
```

两者经过最大池化后都得到 `9`。这说明只要显著特征仍位于同一个池化窗口中，小幅位置变化可能不会改变输出。

这种性质通常称为对小幅平移具有一定的**鲁棒性**或**不敏感性**。它不是严格的平移不变性：如果特征跨过窗口边界，池化结果仍可能改变。

## 6. 池化在每个通道上独立进行

普通池化不会把不同通道相加，也不会改变通道数量。每个通道都独立使用相同的池化规则。

例如，输入有三个通道：

```python
X = torch.randn(1, 3, 8, 8)
Y = F.max_pool2d(X, kernel_size=2, stride=2)

print(X.shape)  # torch.Size([1, 3, 8, 8])
print(Y.shape)  # torch.Size([1, 3, 4, 4])
```

这里：

- 批量大小仍然是 `1`；
- 通道数仍然是 `3`；
- 高度和宽度从 `8 × 8` 变成 `4 × 4`。

这与卷积层不同：卷积层可以通过不同卷积核把输入通道组合成新的输出通道，而池化层通常只在各通道内部做空间汇总。

## 7. 池化输出尺寸如何计算？

对于一维空间尺寸，若输入大小为 $H$，池化窗口为 $K$，步幅为 $S$，填充为 $P$，默认向下取整时，输出大小为：

$$
H_{out}=\left\lfloor\frac{H+2P-K}{S}\right\rfloor+1
$$

宽度方向使用同样的公式。

例如，输入高度为 `5`，窗口大小为 `2`，步幅为 `2`，不填充：

$$
H_{out}=\left\lfloor\frac{5-2}{2}\right\rfloor+1=2
$$

```python
X = torch.randn(1, 1, 5, 5)
Y = F.max_pool2d(X, kernel_size=2, stride=2)

print(Y.shape)  # torch.Size([1, 1, 2, 2])
```

多出来的最后一行和最后一列无法组成完整的 `2 × 2` 窗口，因此默认不会进入输出。

如果设置 `ceil_mode=True`，PyTorch 会在计算输出尺寸时使用向上取整，使边缘处的部分窗口也可能参与计算：

```python
Y = F.max_pool2d(
    X,
    kernel_size=2,
    stride=2,
    ceil_mode=True,
)

print(Y.shape)  # torch.Size([1, 1, 3, 3])
```

## 8. 步幅不一定等于窗口大小

当 `kernel_size=2, stride=1` 时，相邻池化窗口会发生重叠：

```python
X = torch.tensor([[[
    [1.0, 2.0, 3.0],
    [4.0, 5.0, 6.0],
    [7.0, 8.0, 9.0],
]]])

Y = F.max_pool2d(X, kernel_size=2, stride=1)

print(Y)
# tensor([[[[5., 6.],
#           [8., 9.]]]])
```

四个池化窗口分别是：

```text
[1 2; 4 5] → 5    [2 3; 5 6] → 6
[4 5; 7 8] → 8    [5 6; 8 9] → 9
```

在 PyTorch 中，如果省略 `stride`，池化层默认令 `stride` 等于 `kernel_size`。

## 9. 全局平均池化

全局平均池化（Global Average Pooling，GAP）会把每个通道的整个空间区域取平均，使每个通道最终只保留一个数。

```text
输入：batch_size × channels × height × width
输出：batch_size × channels × 1 × 1
```

PyTorch 可以使用自适应平均池化实现：

```python
import torch.nn as nn

X = torch.randn(8, 128, 7, 7)
global_avg_pool = nn.AdaptiveAvgPool2d(output_size=1)
Y = global_avg_pool(X)

print(Y.shape)  # torch.Size([8, 128, 1, 1])
```

也可以直接对高度和宽度维度求平均：

```python
Y = X.mean(dim=(2, 3), keepdim=True)
print(Y.shape)  # torch.Size([8, 128, 1, 1])
```

全局平均池化常放在分类网络末尾，用来把每个通道的空间信息汇总成一个数。与把整张特征图展平后连接一个很大的全连接层相比，它通常能够显著减少参数量。

类似地，`nn.AdaptiveMaxPool2d(1)` 可以实现全局最大池化。

## 10. 池化层有没有缺点？

池化属于有损降采样。输入中的多个数被压缩成一个数后，其余信息无法恢复。

例如最大池化：

```text
[1 2; 3 9] → 9
```

输出只保留了 `9`，其余三个值以及最大值的精确位置都丢失了。

因此，池化可能带来这些问题：

- 丢失细节和精确位置信息；
- 对需要像素级输出的任务可能不利，例如语义分割；
- 过多池化会让特征图缩小得太快。

现代 CNN 并不一定使用独立池化层。有些网络会使用**步幅大于 1 的卷积**完成可学习的降采样：

```python
import torch.nn as nn

downsample = nn.Conv2d(
    in_channels=64,
    out_channels=128,
    kernel_size=3,
    stride=2,
    padding=1,
)
```

这种卷积既进行特征提取，也降低空间尺寸，但它有可训练参数，计算方式和池化并不相同。

## 11. 一个包含卷积和池化的小网络

下面的网络先使用卷积提取特征，再使用最大池化降低空间尺寸：

```python
import torch
import torch.nn as nn

model = nn.Sequential(
    nn.Conv2d(
        in_channels=3,
        out_channels=16,
        kernel_size=3,
        padding=1,
    ),
    nn.ReLU(),
    nn.MaxPool2d(kernel_size=2, stride=2),
    nn.Conv2d(
        in_channels=16,
        out_channels=32,
        kernel_size=3,
        padding=1,
    ),
    nn.ReLU(),
    nn.AdaptiveAvgPool2d(output_size=1),
)

X = torch.randn(8, 3, 32, 32)
Y = model(X)

print(X.shape)  # torch.Size([8, 3, 32, 32])
print(Y.shape)  # torch.Size([8, 32, 1, 1])
```

形状变化过程为：

```text
[8, 3, 32, 32]
        ↓ Conv2d
[8, 16, 32, 32]
        ↓ MaxPool2d
[8, 16, 16, 16]
        ↓ Conv2d
[8, 32, 16, 16]
        ↓ AdaptiveAvgPool2d
[8, 32, 1, 1]
```

## 12. 总结

- 池化层也叫汇聚层，用于汇总局部区域并降低特征图的空间尺寸；
- 最大池化保留局部区域内最强的响应；
- 平均池化保留局部区域内的平均响应；
- 普通池化通常没有可训练参数，并且在每个通道上独立进行；
- 池化可以减少后续计算量和内存占用，并提供一定的小幅平移鲁棒性；
- 池化会丢失细节和精确位置信息，不能无限制地使用；
- 全局平均池化把每个通道压缩成一个数，常用于分类网络末尾；
- 步幅卷积也能实现降采样，但它是带可训练参数的卷积操作。

可以用一句话理解：

> 卷积负责“寻找特征”，池化负责“汇总特征并缩小尺寸”。
