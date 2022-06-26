## 关于n皇后问题在matlab中的实现

```matlab
clear all;
clc;
n=input('皇后的个数为：');
res=0;%表示总的方案数
X=zeros(1,100);%长度为100的一维数组，存储每一行皇后的列坐标，我们在这里假设问题规模不超过100
t=1;%开始放置第一行的皇后
while t>0
    X(t)=X(t)+1;
    while X(t)<=n
        isConflict=0;%0表示不发生冲突
        for i=1:t-1%遍历之前的所有行
            if abs(t-i)==abs(X(t)-X(i))||X(t)==X(i)%如果出现矛盾
                X(t)=X(t)+1;
                isConflict=1;
                break;%跳出for循环
            end
        end
        if isConflict==0%未发生冲突，跳出循环
            break;
        end
    end
    if X(t)<=n
        if t==n
            res=res+1;
        else
            t=t+1;
            X(t)=0;%0表示未放置
        end
    else
        t=t-1;
    end
end
```

## 层次分析法模型

常用于评价类模型。

不同的评价指标具有不同的权重，对不同的评价指标进行打分。

### 评价指标的选取

对于评价类问题，评价指标并不是显式给出的，而是要去查找相关资料。

### 指标权重的确定

我们可以通过将所有指标两两进行比较，并给定如下图所示的重要性判断：

![avatar](https://github.com/YottabyteM/Stack-Overflow/blob/main/img/AHP/%E5%88%A4%E6%96%AD%E7%9F%A9%E9%98%B5.jpg)

### 一致矩阵

定义见下图：

![avatar](https://github.com/YottabyteM/Stack-Overflow/blob/main/img/AHP/%E4%B8%80%E8%87%B4%E7%9F%A9%E9%98%B5.jpg)

### 一致性检验

就是检查我们构造的判断矩阵与一致矩阵是否有太大的差别。

判断矩阵是否为一致矩阵的一种判断方法就是：当n阶正互反矩阵的最大特征值=n时，为一致矩阵。当它非一致时，一定有最大特征值>n，且最大特征值与n相差越大，判断矩阵越不一致。

该方法的matlab实现如下：

```matlab
a = [1:1:8]
b = []
for i = 1:size(a,2)
    A = [1,2,a(i);1/2,1,2;1/a(i),1/2,1]
    b = [b,max(eig(A))]%max(eig(A))即可得到矩阵A的最大特征值
end
plot(a,b)
```

### 一致性检验的一般步骤
![avatar](https://github.com/YottabyteM/Stack-Overflow/blob/main/img/AHP/%E4%B8%80%E8%87%B4%E7%9F%A9%E9%98%B5.jpg)
上述一致性检验的matlab代码实现：

```matlab
%% 计算一致性比例CR
clc
CI = (Max_eig - n) / (n-1);
RI=[0 0 0.52 0.89 1.12 1.26 1.36 1.41 1.46 1.49 1.52 1.54 1.56 1.58 1.59];  %注意哦，这里的RI最多支持 n = 15
CR=CI/RI(n);
disp('一致性指标CI=');disp(CI);
disp('一致性比例CR=');disp(CR);
if CR<0.10
    disp('因为CR < 0.10，所以该判断矩阵A的一致性可以接受!');
else
    disp('注意：CR >= 0.10，因此该判断矩阵A需要进行修改!');
end
```

当CR>0.1时，我们需要对判断矩阵进行修改。修改的思路为：一致矩阵的各行成倍数关系。

### 判断矩阵在经过一致性检验后，要计算各个指标的权重，可采用如下方法：

#### 算数平均法求权重：

```matlab
% 第一步：将判断矩阵按照列归一化（每一个元素除以其所在列的和）
Sum_A = sum(A)

[n,n] = size(A)  % 也可以写成n = size(A,1)
% 因为我们的判断矩阵A是一个方阵，所以这里的r和c相同，我们可以就用同一个字母n表示
SUM_A = repmat(Sum_A,n,1)   %repeat matrix的缩写
% 另外一种替代的方法如下：
    SUM_A = [];
    for i = 1:n   %循环哦，这一行后面不能加冒号（和Python不同），这里表示循环n次
        SUM_A = [SUM_A; Sum_A]
    end
clc;A
SUM_A
Stand_A = A ./ SUM_A
% 这里我们直接将两个矩阵对应的元素相除即可

% 第二步：将归一化的各列相加(按行求和)
sum(Stand_A,2)

% 第三步：将相加后得到的向量中每个元素除以n即可得到权重向量
disp('算术平均法求权重的结果为：');
disp(sum(Stand_A,2) / n)
% 首先对标准化后的矩阵按照行求和，得到一个列向量
% 然后再将这个列向量的每个元素同时除以n即可（注意这里也可以用./哦）
```

#### 几何平均法求权重：

```matlab
% 第一步：将A的元素按照行相乘得到一个新的列向量
clc;A
Prduct_A = prod(A,2)
% prod函数和sum函数类似，一个用于乘，一个用于加  dim = 2 维度是行

% 第二步：将新的向量的每个分量开n次方
Prduct_n_A = Prduct_A .^ (1/n)
% 这里对每个元素进行乘方操作，因此要加.号哦。  ^符号表示乘方哦  这里是开n次方，所以我们等价求1/n次方

% 第三步：对该列向量进行归一化即可得到权重向量
% 将这个列向量中的每一个元素除以这一个向量的和即可
disp('几何平均法求权重的结果为：');
disp(Prduct_n_A ./ sum(Prduct_n_A))
```

#### 特征值法求权重：

```matlab
% 第一步：求出矩阵A的最大特征值以及其对应的特征向量
clc
[V,D] = eig(A)    %V是特征向量, D是由特征值构成的对角矩阵（除了对角线元素外，其余位置元素全为0）
Max_eig = max(max(D)) %也可以写成max(D(:))哦~
% 那么怎么找到最大特征值所在的位置了？ 需要用到find函数，它可以用来返回向量或者矩阵中不为0的元素的位置索引。
% 那么问题来了，我们要得到最大特征值的位置，就需要将包含所有特征值的这个对角矩阵D中，不等于最大特征值的位置全变为0
% 这时候可以用到矩阵与常数的大小判断运算
D == Max_eig
[r,c] = find(D == Max_eig , 1)
% 找到D中第一个与最大特征值相等的元素的位置，记录它的行和列。

% 第二步：对求出的特征向量进行归一化即可得到我们的权重
V(:,c)
disp('特征值法求权重的结果为：');
disp( V(:,c) ./ sum(V(:,c)) )
% 我们先根据上面找到的最大特征值的列数c找到对应的特征向量，然后再进行标准化。
```

### 层次分析法的一些局限性
![avatar](https://github.com/YottabyteM/Stack-Overflow/blob/main/img/AHP/%E5%B1%82%E6%AC%A1%E5%88%86%E6%9E%90%E6%B3%95%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B1%80%E9%99%90%E6%80%A7.jpg)


### 拓展：多层次层次分析法模型

![avatar](https://github.com/YottabyteM/Stack-Overflow/blob/main/img/AHP/%E5%A4%9A%E5%B1%82%E6%AC%A1%E5%B1%82%E6%AC%A1%E5%88%86%E6%9E%90%E6%A8%A1%E5%9E%8B.jpg)


### 课后习题

1.在输入判断矩阵时，自动检查矩阵是否为判断矩阵。

2.二阶判断矩阵的结果有什么问题？如何修正这一问题？

```matlab
disp('请输入判断矩阵A')     
A=input('A=');     
ERROR = 0;  
[r,c]=size(A);
if r ~= c  || r <= 1
    ERROR = 1;
end
if ERROR == 0
    [n,n] = size(A);
    if sum(sum(A <= 0)) > 0
        ERROR = 2;
    end
end
if ERROR == 0
    if n > 15
        ERROR = 3;
    end
end
if ERROR == 0
    if sum(sum(A' .* A ~=  ones(n))) > 0
        ERROR = 4;
    end
end

if ERROR == 1
    disp('请检查矩阵A的维数是否不大于1或不是方阵')
elseif ERROR == 2
    disp('请检查矩阵A中有元素小于等于0')
elseif ERROR == 3
    disp('A的维数n超过了15，请减少准则层的数量')
elseif ERROR == 4
    disp('请检查矩阵A中存在i、j不满足A_ij * A_ji = 1')
end
```

n=2时，一定是一致矩阵，所以CI = 0，我们为了避免分母为0，在进行一致性检验时将第二个元素改为了很接近0的正数。
