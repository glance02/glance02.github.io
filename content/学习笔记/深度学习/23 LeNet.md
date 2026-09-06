LeNet-5 是最早成功应用于手写数字识别的卷积神经网络之一。

总体由两部分组成：

- **卷积编码器**：卷积层提取局部特征，池化层降低空间尺寸。
- **全连接分类器**：将特征映射为 10 个数字类别的得分。

![](figures/Pasted%20image%2020260906120208.png)

经典 LeNet-5 的尺寸变化（输入为 `1×32×32`）：

```text
Conv(6, 5×5) → AvgPool(2×2) → Conv(16, 5×5) → AvgPool(2×2)
      28×28          14×14           10×10           5×5
```

最后将 `16×5×5=400` 个特征展平，经过 `120 → 84 → 10` 的全连接层完成分类。原论文使用 `tanh` 和平均池化；实际实现中也常用 `ReLU` 和最大池化。

## PyTorch 实现

```python
import torch
from torch import nn


class LeNet(nn.Module):
    def __init__(self, num_classes=10):
        super().__init__()
        self.features = nn.Sequential(
            nn.Conv2d(1, 6, kernel_size=5),
            nn.Tanh(),
            nn.AvgPool2d(2),
            nn.Conv2d(6, 16, kernel_size=5),
            nn.Tanh(),
            nn.AvgPool2d(2),
        )
        self.classifier = nn.Sequential(
            nn.Flatten(),
            nn.Linear(16 * 5 * 5, 120),
            nn.Tanh(),
            nn.Linear(120, 84),
            nn.Tanh(),
            nn.Linear(84, num_classes),
        )

    def forward(self, x):
        return self.classifier(self.features(x))


model = LeNet()
print(model(torch.randn(8, 1, 32, 32)).shape)  # torch.Size([8, 10])
```
