---
title: Numpy 函数中axis的含义
tags:
  - Python
  - Numpy
categories:
  - Python
  - Numpy
abbrlink: 31a06e56
date: 2019-03-18 15:35:21
---

Numpy中很多的基本操作函数，都有一个参数`axis`，比如：
1. argmax 返回最大元素的索引
2. argmin 返回最大元素的索引
3. sum
4. max
5. min
6. mean
7. average
8. median

### 官方解释
我们从`numpy doc`里面的`argmax`函数可以看到下面的解释（有删减）
```python
def argmax(a, axis=None, out=None):
    """
    Returns the indices of the maximum values along an axis.

    Parameters
    ----------
    a : array_like
        Input array. # Numpy数组
    axis : int, optional # 整数（可正可负），可选
        By default, the index is into the flattened array, otherwise
        along the specified axis.
        # 默认是一个扁平数组，否则根据指定的axis
    Returns
    -------
    index_array : ndarray of ints
        Array of indices into the array. It has the same shape as `a.shape`
        with the dimension along `axis` removed.
        # 返回数组的一个切片，和a的shape相同，但是移除了axis这个维度。
```

### 官方 `Examples`

```python
    >>> a = np.arange(6).reshape(2,3)
    >>> a
    array([[0, 1, 2],
           [3, 4, 5]])
    >>> np.argmax(a) # 未指定axis，模式整个数组是一个扁平数组，取所有元素的最大值的下标
    5
    >>> np.argmax(a, axis=0) # a.shape=(2,3), 当axis=0时，输出的shape是(3)
    array([1, 1, 1])
    >>> np.argmax(a, axis=1) # 当axis=1时，输出的shape是(2)
    array([2, 2])

    Indexes of the maximal elements of a N-dimensional array:
    # 不展开index, 保留原始结构
    >>> ind = np.unravel_index(np.argmax(a, axis=None), a.shape)
    >>> ind
    (1, 2)
    >>> a[ind]
    5

    >>> b = np.arange(6)
    >>> b[1] = 5
    >>> b
    array([0, 5, 2, 3, 4, 5])
    >>> np.argmax(b)  # Only the first occurrence is returned. # 只输出第一个出现的最大值下标
    1

    """
```

### 怎么样更好的理解`axis`呢？

以`argmax`为例，功能是返回最大元素的索引。可以这么来理解。

* 当数组是一维的，里面的元素就是一维数组里面的单个值，此时`axis`是没有作用的，只能取值为 `0`，比如
    ```python
    >>> import numpy as np
    >>> a = np.array([1, 2, 3, 4, 5, 6, 7])
    >>> np.argmax(a)
    6
    ```

* 当数组是二维的，就分了几种情况：
    - 不指定axis时，把整个数组当作一维数组来处理，假定数组是3x3，二维数组
        ```python
        >>> arr = np.array([[1, 3, 5],[2, 4, 6],[3, 5, 8]])
        >>> print(arr.shape)
        (3, 3)
        >>> print(arr)
        [[1 3 5]
         [2 4 6]
         [3 5 8]]
        >>> np.argmax(arr) # 不指定axis时，把整个数组展开成一维数组来处理
        8
        ```
    - 当`axis=0`时，假定数组是2x3，二维数组，输出的shape应该是有3个元素的索引的一维数组，按列统计，共有3列，给出每列最大值在列方向上的索引
        ```python
        >>> arr = np.array([[1, 3, 5],[6, 4, 2]])
        >>> print(arr)
        [[1 3 5]
         [6 4 2]]
        # 第一列最大值时6，在列方向上的索引为1
        # 第二列最大值为4，在列方向上的索引为1 
        # 第二列最大值为5，在列方向上的索引为0
        >>> np.argmax(arr, axis=0) 
        array([1, 1, 0], dtype=int64)
        >>> np.argmax(arr, axis=-2)
        array([1, 1, 0], dtype=int64)
        ```
    - 当`axis=1`时，假定数组是2x3，二维数组，输出的shape应该是有2个元素的索引的一维数组，按行统计，共有3行，给出每行最大值在行方向上的索引
        ```python
        >>> arr = np.array([[1, 3, 5],[6, 4, 2]])
        >>> print(arr)
        [[1 3 5]
         [6 4 2]]
        # 第一行最大值时5，在行方向上的索引为2
        # 第二行最大值为6，在行方向上的索引为0 
        >>> np.argmax(arr, axis=1)
        array([2, 0], dtype=int64)
        >>> np.argmax(arr, axis=-1)
        array([2, 0], dtype=int64)
        ```

* 当数组是三维数组时。
    - 不指定`axis`时，将数组展开成一维数组，很好理解
    - 当`axis=0`时，假定数组时2x3x2的三维数组，输出的shape应该是3x2个元素的索引的二维数组。
        ```python
        >>> arr = np.random.randint(12, size=(2, 3, 2))
        >>> print(arr)
        [[[11  8]
          [11  5]
          [ 7  0]]

         [[ 8 10]
          [ 9  5]
          [ 4 10]]]
        #   
        >>> print(arr[0,:,:])
        [[11  8]
         [11  5]
         [ 7  0]]
        >>> print(arr[1,:,:])
        [[ 8 10]
         [ 9  5]
         [ 4 10]]
        # 分别对比 arr[0,:,:], arr[1,:,:] 对应位置，取其中最大值的索引，索引取值范围[0-1]
        # out[0][0] = index(max(11, 8)) = 0
        # out[0][1] = index(max(8, 10)) = 1
        # out[1][0] = index(max(11, 9)) = 0
        # out[1][1] = index(max(5,  5)) = 0
        # out[2][0] = index(max(7,  4)) = 0
        # out[2][1] = index(max(0, 10)) = 1 
        # out = [[0, 1],
        #        [0, 0],
        #        [0, 1]]
        >>> print(np.argmax(arr, axis=0).shape)
        (3, 2)  
        >>> np.argmax(arr, axis=0)
        array([[0, 1],
               [0, 0],
               [0, 1]], dtype=int64)
        >>> np.argmax(arr, axis=-3)
        array([[0, 1],
               [0, 0],
               [0, 1]], dtype=int64)
        >>> np.argmax(arr, axis=0-arr.ndim)
        array([[0, 1],
               [0, 0],
               [0, 1]], dtype=int64)
        ```
    - 当`axis=1`时，假定数组时2x3x2的三维数组，输出的shape应该是2x2个元素的索引的二维数组。
        ```python
        >>> arr = np.random.randint(12, size=(2, 3, 2))
        >>> print(arr)
        [[[11  8]
          [11  5]
          [ 7  0]]

         [[ 8 10]
          [ 9  5]
          [ 4 10]]]
        >>> print(arr[:,0,:])
        [[11  8]
         [ 8 10]]
        >>> print(arr[:,1,:])
        [[11  5]
         [ 9  5]]
        >>> print(arr[:,2,:])
        [[ 7  0]
         [ 4 10]]
        # 分别对比 arr[:,0,:], arr[:,1,:], arr[:,2,:] 对应位置，取其中最大值的索引，索引取值范围[0-2]  
        # out[0][0] = index(max(11, 11, 7 )) = 0
        # out[0][1] = index(max(8 ,  5, 0 )) = 0
        # out[1][0] = index(max(8 ,  9, 4 )) = 1
        # out[1][1] = index(max(10,  5, 10)) = 0
        # out = [[0, 0],
        #        [1, 0]]
        >>> print(np.argmax(arr, axis=1).shape)
        (2, 2)
        >>> np.argmax(arr, axis=1)
        array([[0, 0],
               [1, 0]], dtype=int64)
        >>> np.argmax(arr, axis=-2)
        array([[0, 0],
               [1, 0]], dtype=int64)
        >>> np.argmax(arr, axis=1-arr.ndim)
        array([[0, 0],
               [1, 0]], dtype=int64)
        ```
    - 当`axis=2`时，假定数组时2x3x2的三维数组，输出的shape应该是2x3个元素的索引的二维数组。我们将数组的三个维度依次称为行，列，高
        ```python
        >>> arr = np.random.randint(12, size=(2, 3, 2))
        >>> print(arr)
        [[[11  8]
          [11  5]
          [ 7  0]]

         [[ 8 10]
          [ 9  5]
          [ 4 10]]]
        >>> print(arr[:,:,0])
        [[11 11  7]
         [ 8  9  4]]
        >>> print(arr[:,:,1])
        [[ 8  5  0]
         [10  5 10]] 
        >>> print(np.argmax(arr, axis=2).shape)
        (2, 3)  
        # 分别对比 arr[:,:,0], arr[:,:,1] 对应位置，取其中最大值的索引，索引取值范围[0-1]
        # out[0][0] = index(max(11,  8)) = 0
        # out[0][1] = index(max(11,  5)) = 0
        # out[0][2] = index(max(7 ,  0)) = 0
        # out[1][0] = index(max(8 , 10)) = 1
        # out[1][1] = index(max(9 ,  5)) = 0
        # out[1][2] = index(max(4 , 10)) = 1 
        # out = [[0, 0, 0],
        #        [1, 0, 1]]
        >>> np.argmax(arr, axis=2)
        array([[0, 0, 0],
               [1, 0, 1]], dtype=int64)
        >>> np.argmax(arr, axis=-1)
        array([[0, 0, 0],
               [1, 0, 1]], dtype=int64)
        >>> np.argmax(arr, axis=2-arr.ndim)
        array([[0, 0, 0],
               [1, 0, 1]], dtype=int64)
        ```