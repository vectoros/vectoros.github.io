---
title: Numpy 函数中axis的含义
date: 2019-03-18 15:35:21
tags: [Python,Numpy]
categories:
- Python
- Numpy
---

Numpy中很多的基本操作函数，都有一个参数`axis`，比如：
1. argmax
2. argmin
3. sum
4. max
5. min
6. mean
7. average
8. median

### 官方解释
我们从`numpy doc`里面的`argmax`函数可以看到下面的解释（有删减）
```
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

```
    Examples
    --------
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

未完，待续！
