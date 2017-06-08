# 写给前端看的基本面试算法

## 前言

如果你想去更大的平台发展，比如微软、阿里、Airbnb（抱歉我还不敢奢望FLAG），即使是前端的岗位，在面试时（甚至每一轮）也会考验一些算法题目（虽然我觉得没什么道理）。别问我是怎么知道的，我的面试史就是一部血泪史（有血、有泪、有屎）。所以前端同学懂一些基本的算法也是有必要的。我不知道其他语言的程序员在找工作时面试的算法题有哪些，但前端面试题而言还算是比较基础并且不难，对于一些明明课本上有的送分题答不上来实在是太可惜了。

其实在百度上一搜有很多关于算法的文章，很多文章写的很好，通俗易懂，图文并茂，可以用整篇来描述一个算法，还附上好几种实现。所以为什么我要只用两篇的篇幅做这么一个汇总或者说速成呢，是因为我自己需要速查用。就像一份试卷一样，要有问题，要有答案，要有解题思路。

标题有点用了“面试”这样博眼球的字眼，道个歉。文章内容其实没什么特别的，大部分是大学数据结构里的内容，排序算法、插入算法、时间复杂度、二叉树啦。精选了一部分吧，以后打算每个星期都练一次，算法这东西，个人经验是不练真就会忘的。

然而为什么要写出来，因为一方面有给自己出一份试卷的意思，另一方面我觉得只有你能从容的给别人讲解知识的时候你自己才算是真的懂了。

如果你已经很厉害了就不用继续往下看啦，抓紧时间多刷刷 leetcode 吧。

## 目录：不（保证）会出现但是你必须要了解的算法知识

**重要：建议这些算法至少每个月抽出时间练习一次；建议务必明白它们的排序原理而不是死记硬背代码**

- 排序
    
- 二叉树
    - 前序遍历
    - 中序遍历
    - 后序遍历
    - 找出一棵二叉树结点相加最大的某一列（分支）
    - 如果你在写上述算法时用的是递归方式，请用非递归方式再写一遍
- 数组/字符串相关
    - 合并两个有序数组
    - 移除数组中的重复项
    - 判断一个数组是否是另一个的子数组
    - 数组反转
    - 单词反转
- 其他
    - 判断质数
    - 找到一个数的所有因子
    - 找到最小公倍数
    - 找到最大公因数

## 时间复杂度

时间复杂度是衡量一个算法快慢的标志（当然也存在空间复杂度）。时间复杂度说白了就是语句的执行次数，但这个执行次数我们要从抽象和宏观的角度去考虑。

例如你写了一段十行的代码，但代码语句不一定只执行了十条，因为其中可能有一条语句需要循环执行N次。当这个N等于10或20时，循环执行对于性能开销微乎其微，但如果N变成百万或者千万级别的，性能的开销就很可观了。更甚者，如果这10行代码中有一条语句是被N次嵌套循环执行，那么执行的次数就是`N*N = N^2`，如果此时N依旧是百万或者千万级别，那么后果就可想而知了。

通过以上的实例可以看出，一段代码执行速度的瓶颈其实在于循环语句的执行效率（次数），所以当需要计算一段代码的时间复杂度时，我们着重把代码中影响最大的循环部分抽取出来，统计它的执行次数即可。时间复杂度用大O（Big O notion）表示。常见的时间复杂度情况有以下几种

- 常数（Constant Time）级别`O(1)`：当执行代码中不存在循环语句时时间复杂度即为常数级别，值得一提的是此时即使你的代码有上百行也算是这个级别的，因为相对于循环中无限可能的N来说不值一提；
- 线性（Linear Time）级别`O(n)`：即一层嵌套循环，最常见的操作就是遍历数组，查找最小值什么的；
- 对数（Logarithmic Time）级别`O(logN)`: 即执行次数是N的一半，例如在一个有序的数组中做折半查找、或者是二叉树搜索：

```
while ( low <= high ) {
  mid = ( low + high ) / 2;
  if ( target < list[mid] )
    high = mid - 1;
  else if ( target > list[mid] )
    low = mid + 1;
  else break;
}
```
- 平方数（Quadratic Time）级别`O(n^2)`，执行次数是N的平方，也就是我们最常见的两层嵌套循环。这种场景在接下来的冒泡排序或者选择排序中都能看到

以上是常见的一些复杂度级别，基于以上的认知，还可以推断出当代码中有c层嵌套循环时，时间复杂度可以是指数级别`O(n^c)`；又例如代码有有一层循环执行，但每层循环执行中又有一个折半查找，那么这段代码的时间复杂度就是一个组合情况`O(N * logN)`。

你可能存在的另一个疑问是，如果在计算时间复杂度时不加上其他语句执行的次数，这样岂不是更精确。例如一段一百行的代码，其中有三行负责的是N次循环的语句，其余的97行都是非循环语句，那么时间复杂度可以是`O(N + 97)`？

这么说没有错，但这么做大可不必，因为常量对N的影响微乎其微。想象当N增长到百万级别时，常量（无论是成百还是上千）在N、logN、N^2前都显得太渺小，也就是说，这个算法效率差的本质其实是
N本身以及它的循环次数决定的，增加或者减少一个常量并不会让它执行变得更快或者更慢。所以通常在描述算法是都会撇弃常量。

在某些场景中，例如排序算法，循环的执行次数可能还与初始化等待排序的数组次序有关。所以还要区分最好情况和最坏情况的时间复杂度，这个问题具体情况具体分析。

## 题目

### 排序算法

- 如何计算时间复杂度
- 冒泡排序
- 选择排序
- 插入排序
- 希尔排序（插入排序的改进）
- 快速排序
- 归并排序
- 二叉树排序
- 堆排序
- 以上排序算法中最快的（时间复杂度最低，不考虑空间复杂度）排序算法是？

### 数组相关

- 在一个无序/有序数组中查找指定值、在指定位置插入值、删除指定值/位置的值
- 给出一个数组A以及数字X，找出数组中相加之和等于X的两个数
- 根据出现频率排序
- 找出之和最接近0的两个数
- 找出数组中最小和第二小的两个数
- 分离奇数和偶数
- 找出数组中的重复元素
- 将两个有序数组重新合并为有序数组
- 找到数组中最大的差值（和这两个数）
- 判断一个数组是否是另一个数组的子数组
Majority Element
Find the Number Occurring Odd Number of Times
Largest Sum Contiguous Subarray
Find the Missing Number
Search an element in a sorted and pivoted array
Merge an array of size n into another array of size m+n
Median of two sorted arrays
Write a program to reverse an array
Program for array rotation
Reversal algorithm for array rotation
Block swap algorithm for array rotation
Maximum sum such that no two elements are adjacent
Leaders in an array
Count Inversions in an array
Check for Majority Element in a sorted array
Maximum and minimum of an array using minimum number of comparisons
Segregate 0s and 1s in an array
k largest(or smallest) elements in an array | added Min Heap method
Maximum difference between two elements
Union and Intersection of two sorted arrays
Floor and Ceiling in a sorted array
A Product Array Puzzle
Find the two repeating elements in a given array
Sort an array of 0s, 1s and 2s
Find the Minimum length Unsorted Subarray, sorting which makes the complete array sorted
Find duplicates in O(n) time and O(1) extra space
Equilibrium index of an array
Linked List vs Array
Which sorting algorithm makes minimum number of memory writes?
Turn an image by 90 degree
Next Greater Element
Check if array elements are consecutive | Added Method 3
Find the smallest missing number
Count the number of occurrences in a sorted array
Interpolation search vs Binary search
Maximum of all subarrays of size k (Added a O(n) method)
Find the minimum distance between two numbers
Find the repeating and the missing | Added 3 new methods
Median in a stream of integers (running integers)
Find a Fixed Point in a given array
Maximum Length Bitonic Subarray
Find the maximum element in an array which is first increasing and then decreasing
Count smaller elements on right side
Minimum number of jumps to reach end
Implement two stacks in an array
Find subarray with given sum
Dynamic Programming | Set 14 (Maximum Sum Increasing Subsequence)
Longest Monotonically Increasing Subsequence Size (N log N)
Find a triplet that sum to a given value
Find the smallest positive number missing from an unsorted array
Find the two numbers with odd occurrences in an unsorted array
The Celebrity Problem
Dynamic Programming | Set 15 (Longest Bitonic Subsequence)
Find a sorted subsequence of size 3 in linear time
Largest subarray with equal number of 0s and 1s
Dynamic Programming | Set 18 (Partition problem)
Maximum Product Subarray
Find a pair with the given difference
Replace every element with the next greatest
Dynamic Programming | Set 20 (Maximum Length Chain of Pairs)
Find four elements that sum to a given value | Set 1 (n^3 solution)
Find four elements that sum to a given value | Set 2 ( O(n^2Logn) Solution)
Sort a nearly sorted (or K sorted) array
Maximum circular subarray sum
Find the row with maximum number of 1s
Median of two sorted arrays of different sizes
Shuffle a given array
Count the number of possible triangles
Iterative Quick Sort
Find the number of islands
Construction of Longest Monotonically Increasing Subsequence (N log N)
Find the first circular tour that visits all petrol pumps
Arrange given numbers to form the biggest number
Pancake sorting
A Pancake Sorting Problem
Tug of War
Divide and Conquer | Set 3 (Maximum Subarray Sum)
Counting Sort
Merge Overlapping Intervals
Find the maximum repeating number in O(n) time and O(1) extra space
Stock Buy Sell to Maximize Profit
Rearrange positive and negative numbers in O(n) time and O(1) extra space
Sort elements by frequency | Set 2
Find a peak element
Print all possible combinations of r elements in a given array of size n
Given an array of of size n and a number k, find all elements that appear more than n/k times
Find the point where a monotonically increasing function becomes positive first time
Find the Increasing subsequence of length three with maximum product
Find the minimum element in a sorted and rotated array
Stable Marriage Problem
Merge k sorted arrays | Set 1
Radix Sort
Move all zeroes to end of array
Find number of pairs such that x^y > y^x
Count all distinct pairs with difference equal to k
Find if there is a subarray with 0 sum
Smallest subarray with sum greater than a given value
Sort an array according to the order defined by another array
Maximum Sum Path in Two Arrays
Sort an array in wave form
K’th Smallest/Largest Element in Unsorted Array
K’th Smallest/Largest Element in Unsorted Array in Expected Linear Time
K’th Smallest/Largest Element in Unsorted Array in Worst Case Linear Time
Find Index of 0 to be replaced with 1 to get longest continuous sequence of 1s in a binary array
Find the closest pair from two sorted arrays
Given a sorted array and a number x, find the pair in array whose sum is closest to x
Count 1’s in a sorted binary array
Print All Distinct Elements of a given integer array
Construct an array from its pair-sum array
Find common elements in three sorted arrays
Find the first repeating element in an array of integers
Find the smallest positive integer value that cannot be represented as sum of any subset of a given array
Rearrange an array such that ‘arr[j]’ becomes ‘i’ if ‘arr[i]’ is ‘j’
Find position of an element in a sorted array of infinite numbers
Can QuickSort be implemented in O(nLogn) worst case time complexity?
Check if a given array contains duplicate elements within k distance from each other
Find the element that appears once
Replace every array element by multiplication of previous and next
Check if any two intervals overlap among a given set of intervals
Delete an element from array (Using two traversals and one traversal)
Find the largest pair sum in an unsorted array
Online algorithm for checking palindrome in a stream
Pythagorean Triplet in an array
Maximum profit by buying and selling a share at most twice
Find Union and Intersection of two unsorted Arrays
Count frequencies of all elements in array in O(1) extra space and O(n) time
Generate all possible sorted arrays from alternate elements of two given sorted arrays
Minimum number of swaps required for arranging pairs adjacent to each other
Trapping Rain Water 
Convert array into Zig-Zag fashion
Find maximum average subarray of k length  
Find maximum value of Sum( i*arr[i]) with only rotations on given array allowed  
Reorder an array according to given indexes  
Find zeroes to be flipped so that number of consecutive 1’s is maximized  
Count triplets with sum smaller than a given value  
Find the subarray with least average  
Count Inversions of size three in a give array
Longest Span with same Sum in two Binary arrays
Merge two sorted arrays with O(1) extra space
Form minimum number from given sequence
Subarray/Substring vs Subsequence and Programs to Generate them
Count Strictly Increasing Subarrays
Rearrange an array in maximum minimum form
Find minimum difference between any two elements
Find lost element from a duplicated array
Count pairs with given sum
Count minimum steps to get the given desired array
Find minimum number of merge operations to make an array palindrome
Minimize the maximum difference between the heights

### 二叉树相关




- [Array data structure](http://www.geeksforgeeks.org/array-data-structure/)
- [Binary tree data structure](http://www.geeksforgeeks.org/binary-tree-data-structure/)
- [Computer science in JavaScript: Quicksort](https://www.nczonline.net/blog/2012/11/27/computer-science-in-javascript-quicksort/)
- [Friday Algorithms: JavaScript Merge Sort](http://www.stoimen.com/blog/2010/07/02/friday-algorithms-javascript-merge-sort/)
- [Computer science in JavaScript: Merge sort](https://www.nczonline.net/blog/2012/10/02/computer-science-and-javascript-merge-sort/)
- [Problem Solving with Algorithms and Data Structures using Python](http://interactivepython.org/runestone/static/pythonds/index.html)
- [How to find time complexity of an algorithm](http://stackoverflow.com/questions/11032015/how-to-find-time-complexity-of-an-algorithm)
