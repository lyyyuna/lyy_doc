In this blog, I will write some sort algorithms in Haskell.

Insertion sort
======

Insertion sort is simple and easy to unserstand. The algorithm is composed of two steps,

1. insert an element into a sorted list, and keep the list sorted,
2. pick an element from the unsorted list, and repeat the step 1 until the unsorted list is empty.

For the step 1, assume the element is x, the first element in the sorted list is y. If x is smaller, we put x first, otherwise, we put y first and insert x to the tail of the sorted list, and the source code is

    insert :: Ord a => a -> [a] -> [a]
    insert x [] = [x]
    insert x (y:ys) 
            |  x < y = x:y:ys
            | otherwise = y : insert x ys

At the beginning, the list in the step 2 is empty, we can pick a element from the original list, and insert it into the empty list by following the step 1. Then repeat the step 1 until the original list is empty. Finally, the list in step 2 will be a sorted list.

    insertSort :: Ord a => [a] -> [a] -> [a]
    insertSort xs [] = xs
    insertSort xs (y:ys) = insertSort (insert y xs) ys

The usage is 
    
    > insertSort [] [3, 5, 4, 1]
    [1, 3, 4, 5]

Bubble Sort
======

Bubble sort is also a simple sort algorithm which repeatedly stepping through the list until the list is sorted. The algorithm also contains two steps, 

1. compare each pair of adjacent elements and swap them if they are in the wrong order,
2. pass through the whole list until no pairs need to be swapped.

In my code, the *swaps* procedure contains the first step and part of step 2, it just swaps all the elements encounted in one round, and the biggest elem is sure to be swapped to the last place. The next round the second biggest elem is sure to swapped to the last but one place. 

    swaps :: Ord a => [a] -> [a]
    swaps [] = []
    swaps [x] = [x]
    swaps (x1:x2:xs)
            | x1 > x2 = x2 : swaps(x1:xs)
            | otherwise = x1 : swaps(x2:xs)

    bubbleSort :: Ord a => [a] -> [a]
    bubbleSort xs
            | swaps xs == xs = xs       -- 没发生变化，就停止
            | otherwise = bubbleSort $ swaps xs

It is clear to see that the *bubbleSort* above is not very efficient, for after the first round, the last elem is already the biggest one, we do no have to traverse the whole list again in the second round. An improved way is that we only have to traverse the n-1 elements except for the last one,

    bubbleSort' :: Ord a=> [a] -> [a]
    bubbleSort' [] = []
    bubbleSort' xs = bubbleSort' initElem ++ [lastElem]
            where 
                swappedElem = swaps xs
                initElem = init swappedElem
                lastElem = last swappedElem

Selection sort
======

The algorithm divides the list into two parts: one already sorted, which is built up from left to right, and another remaining to be sorted. The algorithm proceeds by finding the smallest element in the unsorted list , delete it and put it to the right of the sorted list.

For Haskell provides for us *minimum* to find the smallest elem, we only have to write a *deleteFromOri* function,

    deleteFromOri :: Eq a => a -> [a] -> [a]
    deleteFromOri _ [] = []
    deleteFromOri x (y:ys)
            | x == y = ys
            | otherwise = y:deleteFromOri x ys

    selectSort :: Ord a => [a] -> [a]
    selectSort [] = []
    selectSort xs = mini : selectSort xs'
            where 
                mini = minimum xs
                xs' = deleteFromOri mini xs

Quick sort
======

Quick sort is a *divide and conquer* algorithm. The algorithm divides the list into: the smaller elements and the bigger elements.

The steps are,

1. pick an element, called a _pivot_, for convenient, pick the first elem,
2. reorder the list so that all smaller elements come before _pivot_, and all bigger elements come after _pivot_,
3. recursively apply the above steps to the smaller list and bigger list.

The code is 

    quickSort :: Ord a => [a] -> [a]
    quickSort [] = []
    quickSort [x:xs] = quickSort mini ++ [x] quickSort maxi
            where
                mini = filter (<x) xs
                maxi = filter (>=x) xs

Merge sort
======

Merge sort is also a *divide and conquer* algorithm. It contains the following steps,

1. divide the unsorted list into n lists, each containing 1 element,
2. repeatedly merge the lists to produce new list which is in order.

The code is 

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