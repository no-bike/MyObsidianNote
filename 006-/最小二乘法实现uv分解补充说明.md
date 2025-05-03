

## 1.  (`__init__` 和 `fit` 开始部分)

```python
def __init__(self, factors=10, iterations=15, regularization=0.1,
             alpha=40, random_state=None, verbose=True):
    # 参数初始化
    self.factors = factors       # 潜在因子维度
    self.iterations = iterations # 迭代次数
    self.regularization = regularization # 正则化系数
    self.alpha = alpha           # 置信度权重系数
    self.random_state = random_state # 随机种子
    self.verbose = verbose       # 是否显示进度
    self.user_factors = None     # 用户因子矩阵
    self.item_factors = None     # 物品因子矩阵
    self.loss_history = []       # 损失历史记录
```

在 `fit` 方法开始时：
```python
if self.random_state is not None:
    np.random.seed(self.random_state)

n_users, n_items = X.shape

# 使用随机SVD初始化因子矩阵
_, _, Vt = randomized_svd(X, n_components=self.factors, random_state=self.random_state)
self.user_factors = np.random.normal(scale=1. / self.factors,
                                     size=(n_users, self.factors)).astype(np.float32)
self.item_factors = Vt.T.astype(np.float32)

# 预计算正则化矩阵
lambda_eye = np.eye(self.factors, dtype=np.float32) * self.regularization
```

- 初始化策略：使用随机SVD初始化物品因子矩阵，随机正态分布初始化用户因子矩阵
- 正则化矩阵：预先计算一个对角正则化矩阵 `lambda_eye`，避免在每次更新时重复计算

## 2. 主训练循环 (`fit` 方法)

```python
for it in range(self.iterations):
    start_time = time.time()
    
    # 交替更新用户和物品因子
    self.user_factors = self._update_factors(X, self.item_factors, lambda_eye, user=True)
    self.item_factors = self._update_factors(X.T, self.user_factors, lambda_eye, user=False)
    
    # 计算并记录损失
    loss = self._calculate_loss(X, lambda_eye)
    self.loss_history.append(loss)
    
    if self.verbose:
        progress.update(1)
        progress.set_postfix({
            "Loss": f"{loss:.4f}",
            "Time": f"{time.time() - start_time:.2f}s"
        })
```

- 交替更新：每次迭代先固定物品因子更新用户因子，再固定用户因子更新物品因子
- 损失计算：每次迭代后计算当前损失并记录
- 进度显示：使用tqdm显示迭代进度、损失和时间信息

## 3. 因子更新核心 (`_update_factors` 方法)

```python
def _update_factors(self, X, fixed_factors, lambda_eye, user=True):
    n, _ = X.shape
    new_factors = np.zeros_like(fixed_factors)
    X_csr = X.tocsr()

    for i in range(n):
        # 获取该用户/物品的所有交互项
        start, end = X_csr.indptr[i], X_csr.indptr[i + 1]
        if start == end:
            continue  # 无交互则跳过

        items = X_csr.indices[start:end]
        ratings = X_csr.data[start:end]

        # 应用置信度权重
        if self.alpha != 0:
            ratings = 1 + self.alpha * ratings

        V = fixed_factors[items]
        A = V.T.dot(V * ratings[:, np.newaxis]) + lambda_eye
        b = V.T.dot(ratings)

        try:
            new_factors[i] = np.linalg.solve(A, b)
        except np.linalg.LinAlgError:
            new_factors[i] = np.linalg.lstsq(A, b, rcond=None)[0]

    return new_factors
```

- 稀疏矩阵处理：使用CSR格式高效遍历非零元素
- 置信度权重：对隐式反馈数据应用 `1 + alpha * rating` 的权重
- 线性方程求解：
  - 构建正规方程 `A = V^T*V + λI`
  - 构建右侧向量 `b = V^T*ratings`
  - 使用 `np.linalg.solve` 求解，失败时回退到最小二乘解

## 4. 损失计算 (`_calculate_loss` 方法)

```python
def _calculate_loss(self, X, lambda_eye):
    X_csr = X.tocsr()
    predictions = np.zeros_like(X.data)
    
    # 计算预测值
    for i in range(X.shape[0]):
        start, end = X_csr.indptr[i], X_csr.indptr[i + 1]
        if start == end:
            continue
        items = X_csr.indices[start:end]
        pred = self.user_factors[i].dot(self.item_factors[items].T)
        predictions[start:end] = pred
    
    # 计算加权损失
    if self.alpha != 0:
        weights = 1 + self.alpha * X.data
        errors = (X.data - predictions) ** 2
        loss = np.sum(weights * errors)
    else:
        errors = (X.data - predictions) ** 2
        loss = np.sum(errors)
    
    # 添加正则化项
    reg_loss = self.regularization * (
        np.sum(self.user_factors ** 2) +
        np.sum(self.item_factors ** 2)
    )
    
    return (loss + reg_loss) / X.nnz if X.nnz > 0 else 0
```

- 加权平方误差：对隐式反馈数据应用置信度权重
- 正则化项：包含用户和物品因子的L2正则化
- 归一化：除以非零元素数量得到平均损失
