---
layout: post
date: 2016-06-02 22:46
status: public
tags: [CUDA, keras, python, deep-learning]
title: 'Windows 系统中 Keras 深度学习环境的配置'
categories: [Machine-Learning]
---

# Keras 是基于 Python 的深度学习框架，然而在 Windows 下的配置有许多需要注意的地方。

## 0x00 安装Python
直接使用 Anaconda，在[这里](https://www.continuum.io/downloads)下载。自带了SciPy，NumPy等库，不用再配置了。版本选择 2.7的64位版，3.5版本似乎还有问题。
安装完以后可以用下面的命令更新

```
conda update conda
conda update --all
```

## 0x01 安装 Theano

Keras 的后端可以是 [Theano](https://github.com/Theano/Theano) 或者 [TensorFlow](https://github.com/tensorflow/tensorflow) ，本文使用 Theano 作为Keras 后端。使用 Anaconda 可以很方便的安装 Theano。

首先安装依赖

```
conda install mingw libpython
```

然后通过pip直接安装 Theano

```
pip install Theano
```

## 0x02 安装 keras

直接pip安装就可以，但是推荐使用Github来安装，因为要用到其中的例子。

```
git clone https://github.com/fchollet/keras.git
python setup.py install
```

现在基础的安装已经完成，可以通过example目录下的脚本来验证。如果运行正常就说明安装成功。

但是现在依靠CPU运行，速度很慢。下面进入本文的重点，基于 CUDA 使得Keras 可以使用 GPU 进行计算。

## 0x03 配置GPU支持

### 安装CUDA toolkit

   直接去官网下载 https://developer.nvidia.com/cuda-downloads，然后安装就可以

### 安装PyCUDA

   去[http://www.lfd.uci.edu/~gohlke/pythonlibs/?cm_mc_uid=18815490224114598605748&cm_mc_sid_50200000=1464880701#pycuda](http://www.lfd.uci.edu/~gohlke/pythonlibs/?cm_mc_uid=18815490224114598605748&cm_mc_sid_50200000=1464880701#pycuda) 下载所需要的版本的PyCUDA，注意CUDA，Python的版本。然后用pip安装下载的内容

   ```
   pip install pycuda-2016.1.1+cuda7518-cp27-cp27m-win_amd64.whl
   ```

### 验证安装结果

   可以使用[下面一段代码](https://documen.tician.de/pycuda/?cm_mc_uid=18815490224114598605748&cm_mc_sid_50200000=1464880701#)验证安装结果

```python
import pycuda.autoinit
import pycuda.driver as drv
import numpy

from pycuda.compiler import SourceModule
mod = SourceModule("""
__global__ void multiply_them(float *dest, float *a, float *b)
{
  const int i = threadIdx.x;
  dest[i] = a[i] * b[i];
}
""")

multiply_them = mod.get_function("multiply_them")

a = numpy.random.randn(400).astype(numpy.float32)
b = numpy.random.randn(400).astype(numpy.float32)

dest = numpy.zeros_like(a)
multiply_them(
        drv.Out(dest), drv.In(a), drv.In(b),
        block=(400,1,1), grid=(1,1))

print dest-a*b
```

### 配置Theano

在主目录下新建`.theanorc`文件，写入以下内容

```
[global]
device = gpu
floatX = float32

[nvcc]
compiler_bindir=E:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin
fastmath = True
```

   其中`compiler_bindir` 要改成对应 Visual Studio 的目录

### 验证安装结果

```python
 from theano import function, config, shared, sandbox
 import theano.tensor as T
 import numpy
 import time

 vlen = 10 * 30 * 768  # 10 x #cores x # threads per core
 iters = 1000

 rng = numpy.random.RandomState(22)
 x = shared(numpy.asarray(rng.rand(vlen), config.floatX))
 f = function([], T.exp(x))
 print f.maker.fgraph.toposort()
 t0 = time.time()
 for i in xrange(iters):
     r = f()
 t1 = time.time()
 print 'Looping %d times took' % iters, t1 - t0, 'seconds'
 print 'Result is', r
 if numpy.any([isinstance(x.op, T.Elemwise) for x in f.maker.fgraph.toposort()]):
     print 'Used the cpu'
 else:
     print 'Used the gpu'
```

​	运行的结果应该类似如下

```
Using gpu device 0: GeForce GT 640M (CNMeM is disabled, cuDNN not available)

[GpuElemwise{exp,no_inplace}(<CudaNdarrayType(float32, vector)>), HostFromGpu(GpuElemwise{exp,no_inplace}.0)]
Looping 1000 times took 0.430000066757 seconds
Result is [ 1.23178029  1.61879349  1.52278066 ...,  2.43982935  1.8292855
  1.06241667]
Used the gpu
```

### 配置cuDNN

可以看到上面有显示 `cuDNN not available`，为了启用cuDNN，还需要进行以下的配置。
首先去[官网](https://developer.nvidia.com/cuDNN)下载对应版本的cuDNN。下载以后其实是一个压缩包，把解压出来的东西放在 CUDA 的安装目录下，然后修改`.theanorc`如下。
```
[global]
device = gpu
floatX = float32
optimizer_including = cudnn

[nvcc]
compiler_bindir=E:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin
fastmath = True
```
现在再运行就不会报错了。
### 启用cnmem

在 `.theanorc` 中增加
```
[lib]
cnmem=.75
```
就可以使用cnmem获得更好的性能了