# 论文阅读

## 摘要

背景：对比学习等现代技术已在许多领域得到有效应用，包括计算机视觉、自然语言处理和图形结构数据。创建有助于模型学习鲁棒性和判别性表示的积极示例是对比学习方法的关键阶段。
通常，预设的人类直觉会指导相关数据增强的选择。由于模式很容易被人类识别，因此这种经验法则在视觉和语言领域非常有效。然而，在时间序列中目视检查时间结构是不切实际的。数据集和实例级别的时间序列增强的多样性使得很难动态选择有意义的增强。因此，尽管很普遍，但在时间序列领域，与数据增强的对比学习研究较少。

方法：这项研究分析了基于信息论的时间序列数据增强，并以统一的形式总结了采用最多的转换。最重要的是，将其推广为参数化增强方法，以支持时间序列表示学习中的自适应使用。

在基准数据集上的实验表明，方法AutoTCL具有很强的竞争力，与主要基线相比，MSE平均降低10.3%，MAE平均降低7.0%。

# Introduction

时间序列数据是复杂的、非结构化的和高维的，这使得收集标签比收集图像或语言更困难。这种特性阻碍了强大的深度学习方法的部署，而深度学习方法通常需要大量的标记数据进行训练。

自监督学习是一种很有前途的解决方案，因为它能够从未标记的数据中学习。与无监督学习类似，自监督学习方法学习的是时间序列数据的固定维度嵌入，保留了其固有特征，具有更好的可迁移性和泛化能力。

难题：在自我调节的环境中，大多数现有研究软化了标签保留特性，并通过减少不同观点之间的信息交流，更加强调增强多样性。他们经常使用更强的转换器作为增强，并取消语义，这不适用于时间序列数据。

方案：
- 引入了一种基于因式分解的新型框架来指导数据增强，以实现没有预制知识的对比自监督学习。
- 为了自动学习时间序列数据的可行转换，我们提供了一种简单而有效的实例化，可以处理各种经常应用的增强。
- 过全面的实验研究，实证验证了所提方法在基准时间序列预测数据集上的优势。在单变量预测中，MSE 下降了 6.5%，MAE 下降了 4.7%，在多变量预测中，MSE 下降了 2.9%，MAE 下降了 1.2%，我们实现了极具竞争力的表现。在分类任务中，我们的方法实现了 1.平均准确度提高 2%。

# 相关工作

### 时间序列的对比学习

背景：对比学习在表征学习中得到了广泛的应用，在各种方法中都取得了优异的结果。最近，人们一直在努力将连续学习应用于时间序列领域。

方法：用子序列生成正对和负对。TNC使用去偏向对比物镜来确保在表示空间中，来自局部邻域的信号与非邻域的信号不同， 通过检查不同样本之间的关系和样本内的关系，使用几种定制设计的增强技术对时间序列进行无监督对比学习 使用分层对比学习来获取每个时间戳， 利用时间分量和频率分量之间的距离作为自监督信号进行表示学习。每个分量都通过对比估计进行了独立优化 进一步包括了光谱信息，并利用一个简单的退出来捕捉每个实例中的长期关系。

难点：这些方法中的数据增强在复杂的现实生活中并未得到广泛应用，因为这些方法具有普遍性或通过反复试验来选择的局限性。

### 自适应数据增强

背景：数据增强是对比学习的一个重要方面。先前的研究表明，最佳增强方法的选择取决于所使用的特定任务和数据集 。

方法：一些研究探索了增强方法的自适应选择，用于视觉领域的对比学习，例如AutoAugment，该方法使用强化学习方法来寻找翻译政策的最佳组合。Faster-AA，这改进了使用可微分策略网络进行数据增强的搜索管道。DADA引入了一种无偏梯度估计器，可实现更高效的一次性优化策略.在趋势学习框架中，应用InfoMin理论指导视觉领域对比学习的良好视图选择，它进一步提出了一种基于流的生成模型，用于将图像从自然色彩空间转移到新的色彩空间以进行数据增强。研究了用于深度学习的自动数据增强算法，重点关注类策略和神经科学信号等复杂数据类型。所提出的可微松弛方法优于其他方法，并为EGG数据分类引入了新的增强操作。提出了RIM是一种用于时间序列增强的递归框架。然而，鉴于时间序列数据的复杂性，直接应用InfoMin框架可能并不合适。

论文方法：重点是时间序列域，提出了一种端到端的可微分方法，用于自动学习每个时间序列实例的最优增强。

# Methodology



![[Pasted image 20240725233817.png]]图：AutoTCL框架

对比学习的优点：
- 是以相关信息原则为指导，从原始实例中提取信息部分的传真功能。
- 是编码器网络，它使用对比物镜进行了优化。

符号：使用 _T_ × _F_ 矩阵来表示时间序列实例 _x_，其中 _T_ 是其序列的长度，_F_ 是特征的维度。

### 对比性自我监督学习

好的视图保留了语义并提供足够的方差。 在有训练标签的监督设置中，实例的语义通常与标签近似。另一方面，在更流行的自监督学习中，语义保持的探索还有很多不足。

与图像和自然语言句子不同，图像和自然语言句子的语义可以手动验证，而时间序列数据的底层语义是人类无法识别的，这使得在如此复杂的数据中包含强大而忠实的数据增强更具挑战性。

给定一个实例 _x_，假设 _x_ 可以分解为两个部分，信息性 _x_∗ 和噪声/任务无关的部分 ∆_x_。
			x = x∗ + ∆x.

**定义1**：给定一个随机变量 x 及其语义 x∗，可以通过 v = g（x∗） + ∆v 来获得对比学习的良好视图 v，其中 g 是不可取函数，∆v 是与任务无关的噪声，满足 H（∆v） ≥ H（∆x）。

一个好的视图可以保持原始变量中的有用信息同时包括一个更大的方差，以提高编码员训练的鲁棒性。

**属性 2**：（与任务无关的标签保留）。如果变量 v 是 x 的良好视图，并且下游任务标签 y（尽管对训练不可见）与 x 中的噪声无关，则 v 和 y 之间的互信息等同于原始输入 x 和 y 之间的互信息，
即 MI（v; y） = MI（x; y）。

**属性 3**（包含更多信息）：与原始输入 x 相比，一个好的视图 v 包含更多信息，
即 H（v） ≥ H（x）。

### 训练算法

![[Pasted image 20240826000828.png]]
算法 1 描述了一个涉及两个网络组件（增强网络 ( f_{aug} ) 和编码器网络 ( f_{enc} )）的训练过程。以下是该算法的详细解释：

1. **初始化**：训练开始时，将迭代次数的计数器（`epoch`）初始化为 0。
    
2. **迭代训练（Epoch Loop）**：在指定的训练轮数 ( E ) 内，进行迭代训练。
    
3. **数据集迭代**：
    
    - 对于数据集中每一个样本 ( x )：
        - **数据增强**：使用增强网络 ( f_{aug} ) 对样本 ( x ) 进行处理，得到增强后的样本 ( x_a )。
        - **特征编码**：
            - 使用编码器网络 ( f_{enc} ) 计算原始样本 ( x ) 的表示 ( z_x )。
            - 使用相同的编码器网络 ( f_{enc} ) 计算增强样本 ( x_a ) 的表示 ( z_a )。
4. **条件参数更新**：
    
    - 每经过 ( M ) 次迭代（即 `count % M == 0`）：
        - 使用公式（11）计算并优化增强网络 ( f_{aug} ) 的损失。这一步通过反向传播来更新 ( f_{aug} ) 的参数。
5. **编码器网络更新**：
    
    - 无论迭代次数如何，都使用公式（16）计算并优化编码器网络 ( f_{enc} ) 的损失。这一步通过反向传播来更新 ( f_{enc} ) 的参数。
6. **迭代次数递增**：完成一轮数据集的训练后，将迭代次数 `epoch` 增加 1。
    
    
总结来说，该算法通过交替更新增强网络和编码器网络来训练模型，其中增强网络的更新频率低于编码器网络。目的是使编码器网络产生有意义的特征表示，同时增强网络提供有用的数据变换。


# 实验复现结果

import warnings
import json
import argparse
import numpy as np
import matplotlib.pyplot as plt

from infots import InfoTS as MetaInfoTS
from utils import init_dl_program, dict2class
import datautils

warnings.filterwarnings("ignore")

dataset = 'electricity.uni'

with open(f'./configures/{dataset}.json') as f:
    configs = json.load(f)

#### 解析命令行参数
parser = argparse.ArgumentParser()
args = dict2class(**configs)

#### 初始化设备
device = init_dl_program(args.gpu, seed=args.seed, max_threads=args.max_threads)

valid_dataset = datautils.load_forecast_csv(args.dataset, univar=True)
data, train_slice, valid_slice, test_slice, scaler, pred_lens, n_covariate_cols = valid_dataset
train_data = data[:, train_slice]

#### 数据预处理
if train_data.shape[0] == 1:
    train_slice_number = int(train_data.shape[1] / args.max_train_length)
    if train_slice_number < args.batch_size:
        args.batch_size = train_slice_number
else:
    if train_data.shape[0] < args.batch_size:
        args.batch_size = train_data.shape[0]

##### 初始化模型
model = AutoTCL(
    batch_size=args.batch_size,
    lr=args.lr,
    meta_lr=args.meta_lr,
    output_dims=args.repr_dims,
    max_train_length=args.max_train_length,
    input_dims=train_data.shape[-1],
    device=device,
    num_cls=args.batch_size,
    aug_p1=args.aug_p1,
    eval_every_epoch=20
)

#### 训练模型
res = model.fit(
    train_data,
    task_type='forecasting',
    meta_beta=args.meta_beta,
    n_epochs=args.epochs,
    n_iters=args.iters,
    beta=args.beta,
    verbose=False,
    miverbose=True,
    split_number=args.split_number,
    supervised_meta=False,  # for forecasting, use unsupervised setting
    valid_dataset=valid_dataset,
    train_labels=None
)

mse, mae = res
mse = np.array(mse)
mae = np.array(mae)
if len(mse) > 1 and len(mae) > 1:
    mse = mse[:-1]
    mae = mae[:-1]
else:
    print("mse 和 mae 的长度不足")

## 绘制结果
x = 20 * np.arange(len(mse))
plt.plot(x, mse, label="mse@24")
plt.plot(x, mae, label="mae@24")
plt.legend()
plt.show()


进行了四个数据集的部分简单单变量预测

1. ETTh1数据集
![[Pasted image 20240825235313.png]]
![[Pasted image 20240825235932.png]]
2. ETTh2数据集
![[Pasted image 20240825235359.png]]
![[Pasted image 20240826000104.png]]
3. ETTm1数据集
![[Pasted image 20240825235428.png]]
![[Pasted image 20240826000234.png]]
4.  electricity数据集
![[Pasted image 20240825235509.png]]
![[Pasted image 20240826000312.png]]

![[Pasted image 20240825235627.png]]

由对比可看出，实验结果与论文中结果接近。

# 总结


