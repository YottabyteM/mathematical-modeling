# Topsis模型

## 概念&代码

### 第一步：将原始矩阵正向化

<img src="S1.png" style="zoom:67%;" />

将所有的指标类型全部转化为极大型指标

**转化方法：**

极小型：
$$
max - x
$$

```matlab
posit_x = max(x) - x; %min2max代码
```

中间型：
$$
M = max\{|x_{i} - x_{最好的值}|\}, x_{i} =  1 - \frac{|x_{i} - x_{最好的值}|}{M}
$$

```matlab
M = max(abs(x-best)); %mid2max代码，其中best为最佳值
posit_x = 1 - abs(x-best) / M;
```

区间型：设最佳区间为[a,b]
$$
M = max\{a - min\{x_{i}\}, max\{x_i\} - b\}, x_{i} = \left\{ 
    \begin{array}{lc}
        1 - \frac{x_{i}-b}{M}&x_{i} < a \\
        1&a\leq x_{i} \leq b \\
        1 - \frac{a - x_{i}}{M}&b < x_i
    \end{array}
\right.
$$

```matlab
M = max([a-min(x),max(x)-b]); %inter2max代码
%基本同上，多了条件语句判定区间，故不再赘述
```

### 第二步：将正向化矩阵标准化

$$
每一个元素/\sqrt{其所在列元素平方和}
$$

```matlab
Z = X ./ repmat(sum(X.*X) .^ 0.5, n, 1);
%repmat(A, m, n) 将A元素复制为m行n列矩阵
```

### 第三步：计算得分并归一化

<img src="S2.png" style="zoom:80%;" />
$$
定义最大值Z_{i}^{+}为第i列的最大值 \\
定义最小值Z_{i}^{-}为第i列的最小值 \\
定义第i个评价对象与最大值的距离D^{+}=\sqrt{\omega_{i}\sum_{j=1}^{m}(Z_{j}^{+}-z_{ij})^{2}} \\
定义第i个评价对象与最小值的距离D^{-}=\sqrt{\omega_{i}\sum_{j=1}^{m}(Z_{j}^{-}-z_{ij})^{2}} \\
第i个对象的未归一化得分S_{i} = \frac{D^{-}_{i}}{D^{+}_{i}+D^{-}_{i}}
$$

```matlab
D_P = sum([(Z - repmat(max(Z),n,1)) .^ 2 ] .* repmat(weigh,n,1) ,2) .^ 0.5;   % D+ 与最大值的距离向量
D_N = sum([(Z - repmat(min(Z),n,1)) .^ 2 ] .* repmat(weigh,n,1) ,2) .^ 0.5;   % D- 与最小值的距离向量
% sum(A, n)当n为1时，按每一列进行加和；当n为2时，按每一行进行加和， 默认值为2
% weigh为权重矩阵
S = D_N ./ (D_P+D_N);    % 未归一化的得分
stand_S = S / sum(S) % 归一化后得分
```

**在正课中，每个评价指标的权值均为1。**

## 熵权法改进Topsis模型

### 第一步：将原始矩阵正向化标准化

基本同上，若正向化后有值为负数，则使用如下公式将其修改为正值：
$$
x_{i} = \frac{x_{i} - min\{x_{i}\}}{max\{x_{i}\}-min\{x_{i}\}}
$$

### 第二步：计算第j项指标第i个样本所占的比重

$$
p_{ij} = \frac{z_{ij}}{\sum_{i = 1}^{n}z_{ij}}
$$

```matlab
%下述代码需要使用循环
p = x / sum(x); % x为n行m列矩阵中第j列
```

### 第三步：计算信息熵，得到熵权

$$
e_j=-\frac{1}{\ln n}\sum_{i=1}^{n}p_{ij}\ln (p_{ij}) \\
d_j = 1 - e_j \\
\omega_j = d_j/\sum_{j = 1}^{m}d_{j}
$$

```matlab
e = -sum(p .* log(p)) / log(n); % 计算第j个指标信息熵
D(j) = 1 - e; % 计算第j个指标信息效用值
% 上述代码需要使用循环
W = D ./ sum(D);  % 将信息效用值归一化，得到权重
```

从公式代码中可以看出，若该项指标每个对象的值相差越大，则该项指标权值越大