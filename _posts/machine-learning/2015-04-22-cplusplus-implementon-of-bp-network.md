---
layout: post
date: 2015-04-22 10:17
status: public
tags: 'Neural Network, Backpropagation, c++'
title: 'C++ 实现backpropagation神经网络'
categories: [Machine-Learning]
---

# 0x00 神经元与神经网络

神经网络是机器学习的一种重要方法，如果不去看具体的数学推导，其整体结构和思想还是很容易理解的。实现起来也很方便。

从总体上看，神经网络是这样的样子  (图片来自wikipedia)
![](https://upload.wikimedia.org/wikipedia/commons/4/46/Colored_neural_network.svg)
网络分为输入层，隐藏层和输出层三个部分，每层都有若干神经元节点。

输入层的神经元只是一个缓冲，把输入的数据传递给隐藏层。而隐藏层和输出层，则是把前一层的输出作为自己的输入并进行计算，可以用下面的公式表示。

$ y = f(\sum{w_i * x_i}+\theta) $

其中$x_i$表示输入,$w_i$表示输入的权重，也就是对前一层的每一个神经元的输出，乘以一个权重进行求和，最后加上一个常数。把结果作为函数f的自变量，f的运算结果就是自己的输出。而对于函数f，通常取Sigmoid function

$f = \frac{1}{1 + e^{-x}} $

其图像为 ![](http://upload.wikimedia.org/wikipedia/commons/8/88/Logistic-curve.svg)
(图片来自wikipedia)

也就是把输入映射到$(0,1)$这样的一个区间内。

0x01 backpropagation反向传播算法

神经网络是一种有监督的学习算法。对于训练集中的元素，我们已经知道了输入和对应的输出。输入的数据经过网络以后得到的输出会与应有的输出进行比较，据此可以对神经元的权重进行调整。 定义误差

$ e = \sum{(x_i - x_o)^2} $

其中$x_o$表示实际的输出，$x_i$应该的输出

为了使得误差快速的减小，一般的方法是让所有的变量都沿着梯度的方向下降。对于输出层可以直接求出梯度，而对于中间层也可以通过链式法则求出。本文重点在于实现，不做具体的数学推导。 这里有一篇很形象的文章说明了具体的过程。

# 0x02 实现

1.定义神经元结构体
由最开始的图可以知道，每个神经元都有输入和输出，而权重则是两个神经元之间的关系，可以保存在输入端也可以保存在输出端。这里采用了保存在输入节点的方式。

```c++
struct Node{
    vector < double > weights;
    double output, grid,old_grid;
    Node(){
        output = grid = old_grid = 0.0;
    }
};
```

weights表示前一层的每一个节点作为输入对该神经元的权重，output是输出，grid和oldgrid则用来对权重进行修正，其中oldgrid与历史修正有关。

2.定义层
神经网络中的一个层是由一些神经元组成的，就是结构图中的一列，简单定如下。

```c++
typedef vector<Node> Layer; 
```

3.定义网络
一个神经网络保存了若干个层，包括输入层，隐藏层和中间层。同时提供了正向和方向接口来训练。

```c++
class Net{
public:
    Net(const vector<int>& shape);
    void front(const vector<double>& input);
    static double sig(const double& x) ;
    void backprop(const vector<double>& target);
    Layer getResult() const;
private:
    static double eta,alpha;
    vector<Layer> net;
};
```

构造函数接受一个vector来描述网络的形状，vecrtor中的每一个元素代表了每一层有多少个神经元，这样就能唯一的确定网络的结构。

```c++
Net::Net(const vector<int>& shape){
    int n = shape.size();//总的层数
    for (int i = 0; i < n; i++){
        net.push_back(Layer());
        for (int j = 0; j <= shape[i]; j++){//构造第i层的所有神经元
            net.back().push_back(Node());
            if (i > 0){
                net[i][j].weights = vector < double > (shape[i - 1]+1);
                for (auto& val : net[i][j].weights){
                    val = rand()*(rand()%2?1:-1)*1.0/RAND_MAX;
                }
            }
        }
    }
    for (int i = 0; i < n; i++){
        net[i].back().output = 1;
    }
}
```

考虑到公式中的常数项$\theta$，在每一层的最后都加了一个固定输出为1的神经元。同时将权重都置成$[-1,1]$区间的一个数随机数。

4.前向过程
为了计算每个神经元的输出，先定义sigmod函数，也就是上面说的 $y = \frac{1}{1 + e^{-x}} $

```c++
double Net::sig(const double& x){
    return 1.0 / (1.0 + exp(-x));
}
```

而前向操作的定义如下

```c++
void Net::front(const vector<double>& input){
    //将输入层每个节点的输出设置为网络的输入
    for (int i = 0; i < net[0].size() - 1; i++){
        net[0][i].output = input[i];
    }
    int n = net.size();
    //对于隐藏层和输出层
    for (int i = 1; i < n; i++){
        //对于层中每一个节点（除了最后一个固定输出唯一的节点）
        for (int j = 0; j < net[i].size() - 1; j++){
            double& res = net[i][j].output;
            res = 0.0;
            //对前一层的输出和权重的乘积求和
            for (int k = 0; k < net[i - 1].size(); k++){
                res += net[i - 1][k].output * net[i][j].weights[k];
            }
            res = sig(res);
        }
    }
}
```

也就是输入层简单的把输入传递进来。隐藏层和输出层根据前一层的输出和权重计算自己的输出。

5.反向传播
而反向传播分为三个循环。

```c++
void Net::backprop(const vector<double>& target){  
    //计算每一个输出节点的误差
    for (int i = 0; i < target.size(); i++){
        Node& node = net.back()[i];
        node.grid = target[i] - node.output; 
    }
    //修正输出层的权重
    for (int i = net.size() - 2; i > 0; i--){
        Layer& cur_layer = net[i];
        Layer& nex_layer = net[i + 1];
        Layer& prv_layer = net[i - 1];
        for (int j = 0; j < cur_layer.size()-1; j++){
            Node& node = cur_layer[j];
            node.grid = 0;
            for (int k = 0; k < nex_layer.size() - 1; k++){
                node.grid += nex_layer[k].grid * nex_layer[k].weights[j];
            }
        }
    }
    //修正隐藏层的权重
    for (int i = 1; i < net.size(); i++){
        Layer& layer = net[i];
        Layer& prev = net[i - 1];
        for (int j = 0; j < layer.size(); j++){
            Node& node = layer[j];
            double d = node.output* (1 - node.output);
            for (int k = 0; k < node.weights.size(); k++){
                node.weights[k] += (eta * node.grid + alpha * node.old_grid) * d * prev[k].output;
            }
            node.old_grid = node.grid;
        }
    }
}
```

在函数的第一个循环，计算了每一个输出节点的误差。 在第二个循环，计算了所有输出层神经元的梯度并且更新了权重值。 在第三个循环则更新了隐藏层的权重。 这里不仅考虑了当前的梯度grid还考虑了上一次的梯度。他们分别与常数eta和alpha相乘。

```c++
double Net::eta = 1;
double Net::alpha = 0.3;
```

这两个常数控制了学习的速率。如果太小可能会收敛的很慢，如果太大会导致不稳定。

6.获取结果。
很简单，直接返回输出层。

```c++
Layer Net::getResult()const {
    return net.back();
}
```

# 0x03 验证

这里采取异或网络对刚才的程序进行验证。  
由于异或(XOR)的结果不是线性可分的。然而用神经网络只需要一个中间层就可以很容易的解决这一个问题。

```c++
    srand( time(NULL) );
    vector<int> shape{ 2, 3, 1 };
    Net net(shape);
    int p, q;
    for (int i = 0; i < 20000; i++){
        p = rand() % 2;
        q = rand() % 2;
        vector<double> input{ (double)p, (double)q };
        net.front(input);
        int res = p^q;
        vector<double> tag{ (double)res };
        net.backprop(tag);
    }
    for (int i = 0; i < 4; i++){
        p = i % 2;
        q = i / 2;
        vector<double> input{ (double)p, (double)q };
        net.front(input);
        int res = p^q;
        Layer& result = net.getResult();
        cout << i << ": " << "p = " << p << ", q = " << q << ", res = " << res << " , result = " << result[0].output << " , dis = " <<
            abs(result[0].output - res) << endl;
    }
```

程序首先初始化一个神经网络，每次随机取值为0或1的两个数p,q并计算结果res = p xor q，将输入和结果送给网络进行学习。最终对学习的结果进行检验，输出如下

```console
1: p = 0, q = 0, res = 0 , result = 0.0013878 , dis = 0.0013878  
2: p = 1, q = 0, res = 1 , result = 0.982337 , dis = 0.0176629  
3: p = 0, q = 1, res = 1 , result = 0.98569 , dis = 0.0143104  
4: p = 1, q = 1, res = 0 , result = 0.0218435 , dis = 0.0218435  
```

可以看到误差已经低到完全可以接受的水平。

# 0x04 参考

一个很好的视频教程 https://vimeo.com/19569529