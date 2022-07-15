# Lingo使用方法

## 基础用法

```lingo
max = x + y;
x + 2 * y >= 20;
x - y <= 10;
```

## for和sum对于大规模变量的求解

$$
target = min \ \sum_{i = 1}^{20} \sum_{j = 1}^{92}{d_{ij}*x_{ij}}\\
\begin{cases}
1 \leq \sum_{j = 1}^{92}{x_{ij}}\ \ \ i=1...20\\
\sum_{i=1}^{20}{x_{ij}}=1\ \ \ j = 1,2...92\\
x_{ij}=0,1
\end{cases}
$$

假设我们的$d_{ij}$均存在文件"D:\Math\d_1.xlsx"中，且字段为'distance'，那么我们的代码如下

```lingo
model:
sets:
row/1..20/:ans,c;
col/1..92/;
link(row, col):d, X;
endsets

data:
d = @ole("D:\Math\d_1.xlsx",'distance');
enddata

min=1*@sum(link(i, j):X(i, j)*d(i, j));
@for(row(i):@sum(col(j):X(i, j)) >= 1);
@for(col(j):@sum(row(i):X(i, j)) = 1);
@for(link(i, j): @bin(X(i, j)));
end
```

## 一些特殊函数的使用

``` lingo
@free(var) !lingo中默认所有变量都是非负数，所以加上free可以让这些数字的限制变为实数
@bin(var)  !将变量限制为0和1
@gin(var)  !限制变量为整数
@BND(L, X, U) !限制L<=X<=U。
@abs(X)    !绝对值
@sin(X)    !正弦值
@cos(X)    !余弦值
@tan(X)    !正切值
@exp(X)    !返回e的x次方
@log(X)    !返回自然对数
@sign(X)   !符号函数
@floor(X)  !返回整数部分
@smax(X)   !返回X中最大值
@smin(X)   !返回X中最小值
!金融函数
!概率函数
```

## 逻辑运算

```lingo
#not# !否定逻辑值
#eq#  !判断是否相等
#ne#  !若不相等
#gt#  !若大于
#ge#  !若大于等于
#lt#  !若小于
#le#  !若小于等于
#and# !两都为true
#or#  !至少一个true
|     !对循环的条件限制
```

## 综合程序(循环+判断)（邻接矩阵0-1规划）

```lingo
model:
sets:
node/1..6/;
item(node, node)/x1, y1, x2, y2, x3, y3, x4, y4.......xm, ym/:w, f;
end sets

data:
w = w1, w2...wm;
enddata

n = @size(node);
min=@sum(item(i, j): w(i, j) * f(i, j));
@sum(item(i, j) | i #eq# 1: f(i, j)) = 1;
@sum(item(i, j) | j #eq# n: f(i, j)) = 1;
@for(node(i)| i #ne# 1 #and# i #ne# n:@sum(item(i,j):f(i,j)) = @sum(item(j,i): f(j,i)));
@for(item(i, j):@bin(f(i, j)));
end
```

