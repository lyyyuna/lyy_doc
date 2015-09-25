这篇文章尝试使用 Haskhell 来重写常见的排序算法。这里不考虑*效率*，比如时间和空间上的，所以不会刻意去写成*尾递归*。

插入排序
=======

插入排序是一种简单易懂的排序。这里分为两个步骤：

1. 将一个元素插入一个已被排序的数列
2. 对一个未排序的数列不停施以步骤 1

首先步骤 1，要插入数 x，当前序列中第一个数为 y。将 x, y 较小的数放在前面，然后对去除第一个数之后的子序列不停重复上述过程。

    insert :: Ord a => a -> [a] -> [a]
    insert x [] = [x]
    insert x (y:ys) 
            |  x < y = x:y:ys
            | otherwise = y : insert x ys

接下来，只要施以步骤 2 即可，即将乱序的元素一个个地使用 insert 函数到另一个有序列表里就可以了。

    insertSort :: Ord a => [a] -> [a]
    insertSort [] = []
    insertSort (x:xs) = insert x (insertSort xs)

也可以写成尾递归的形式，用一个列表来存储中间结果：

    insertSort :: Ord a => [a] -> [a] -> [a]
    insertSort xs [] = xs
    insertSort xs (y:ys) = insertSort (insert y xs) ys

冒泡排序
=======

冒泡排序也分为两个步骤：

1. 比较相邻元素的大小，然后交换较小的元素，将最大的数通过这个方式交换到最后
2. 重复步骤 1

第一步是交换

    swaps :: Ord a => [a] -> [a]
    swaps [] = []
    swaps [x] = [x]
    swaps (x1:x2:xs)
            | x1 > x2 = x2 : swaps(x1:xs)
            | otherwise = x1 : swaps(x2:xs)

然后就是不停 swaps，直到列表不再发生变化

    bubbleSort :: Ord a => [a] -> [a]
    bubbleSort xs
            | swaps xs == xs = xs       -- 没发生变化，就停止
            | otherwise = bubbleSort $ swaps xs

可以看到，第二步的效率不高，因为第一轮的 swaps 之后，最后一个数已经是最大的数了，第二步就没有必要来遍历到最后一个数。所以，可以将前一步 swaps 之后的序列分为前 n-1 项和最后一项，当前步下，最后一项可以不动，只需 bubbleSort 前 n-1 项。

    bubbleSort' :: Ord a=> [a] -> [a]
    bubbleSort' [] = []
    bubbleSort' xs = bubbleSort' initElem ++ [lastElem]
            where 
                swappedElem = swaps xs
                initElem = init swappedElem
                lastElem = last swappedElem

选择排序
=======

首先找到最小的元素，将其从序列中取出，放入另一个序列中（初始为空），然后依次类推，直到所有元素从元序列被取出。

1. 寻找序列中最小数，Haskell 有现成的函数 minimum
2. 将最小数从原序列中删除

这里只要写一个将序列中指定元素删除的程序

    deleteFromOri :: Eq a => a -> [a] -> [a]
    deleteFromOri _ [] = []
    deleteFromOri x (y:ys)
            | x == y = ys
            | otherwise = y:deleteFromOri x ys

然后只要将每次 minimum 得到的数从原序列删除放入新序列

    selectSort :: Ord a => [a] -> [a]
    selectSort [] = []
    selectSort xs = mini : selectSort xs'
            where 
                mini = minimum xs
                xs' = deleteFromOri mini xs

快速排序
=======

快排的定义其实非常简单，但在 c 语言中却不好理解，不像 Haskell 这样写起来就像在定义一个数学定理一样。

1. 取出序列中的一个数（简单的取法，直接取第一个元素），将所有小于该数的数作为一组放于该数左边，将所有该数的数作为另一组放于该数右边
2. 对左右两组数分别施以步骤 1

代码为

    quickSort :: Ord a => [a] -> [a]
    quickSort [] = []
    quickSort [x:xs] = quickSort mini ++ [x] quickSort maxi
            where
                mini = filter (<x) xs
                maxi = filter (>=x) xs

当然这里效果不高，会运算过程中会产生许多 []。

归并排序
=======

归并排序这里仍然是两个步骤。

1. 将两个有序数列合为一个有序数列
2. 将原序列不停划分两部分，直至每部分只有一个元素，然后不停调用步骤 1，将其合并成一个有序数列

步骤 1 的实现，只要将两个序列 xs 和 ys 的第一个元素作比较即可。步骤 2 采用对半划分。

    merge :: Ord a => [a] -> [a] -> [a]
    merge xs [] = xs
    merge [] ys = ys
    merge (x:xs) (y:ys)
            | x > y = y:merge (x:xs) ys
            | otherwise = x:merge xs (y:ys)
            
    mergeSort :: Ord a => [a] -> [a]
    mergeSort xs = merge (mergeSort x1) (mergeSort x2)
            where
                (x1, x2) = split xs
                split xs = (take mid xs, drop mid xs)
                mid = (length xs) `div` 2 