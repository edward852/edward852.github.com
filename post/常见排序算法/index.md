
本文参考[白话经典算法之七大排序](http://download.csdn.net/download/morewindows/4443208)，介绍常见的排序算法，包括原理和实现。  
<!--more-->

# 对比因素
## 时间复杂度
最差、最好、平均的复杂度。  
O(N<sup>2</sup>), O(NlogN)还是其它。  

## 空间复杂度
就地排序还是需要额外的空间。  

## 稳定性
排序是否稳定，也就是按排序标准相等的元素是否维持原序列的相对位置。  
不稳定的有快速排序、选择排序、希尔排序、堆排序。  

## 内部排序、外部排序
是否需要外部存储设备辅助排序。  

# 冒泡排序
冒泡排序原理是每一轮依次按需交换相邻元素，使得较大元素排在后面。  
这样第N轮最差能够排序好前N大元素。对于N个元素来说，最多只需要N-1轮。  
```cpp
void BubbleSortV1(int nums[], int size)
{
    if (!nums) return;

    for (int end=size-1; 0<end; end--)
    {
        for (int idx=0, end=size-1; idx<end; idx++)
        {
            if (nums[idx] > nums[idx+1])
            {
                swap(nums[idx], nums[idx+1]);
            }
        }
    }
}
```
冒泡排序第一种优化是每一轮检查是否发生交换，如果没有则说明排序已经完成。  
```cpp
void BubbleSortV2(int nums[], int size)
{
    if (!nums) return;

    for (int end=size-1; 0<end; end--)
    {
        int swapped = 0;

        for (int idx=0, end=size-1; idx<end; idx++)
        {
            if (nums[idx] > nums[idx+1])
            {
                swapped = 1;
                swap(nums[idx], nums[idx+1]);
            }
        }

        if (!swapped)
        {
            break;
        }
    }
}
```
冒泡排序第二种优化是每一轮记录最后一次交换的位置，下一轮只要遍历到这个位置即可。  
```cpp
void BubbleSortV3(int nums[], int size)
{
    if (!nums) return;

    for (int end=size-1; 0<end;)
    {
        int last = 0;

        for (int idx=0; idx<end; idx++)
        {
            if (nums[idx] > nums[idx+1])
            {
                last = idx;
                swap(nums[idx], nums[idx+1]);
            }
        }

        end = last;
    }
}
```

# 计数排序
适合数值范围较小(比如说分数:0~100)，重复次数较多的排序场合。  
以分数(整数)排序来说，只要101个元素的数组统计然后按序(+次数)输出即可。  
计数排序是 [基数排序](#基数排序) 的简化版。  

# 基数排序 (桶排序)
基数排序是RadixSort，有的也叫BucketSort。  
基数排序(LSD最低位优先)依次按位数(个,十,百...)将元素放入0~9号桶中排成新序列，最终完成排序。  
首先我们按照正常思维实现：  
```cpp
int maxDigits(vector<int> &nums)
{
    int d = 1;
    int r = 10;

    auto maxItr = max_element(nums.begin(), nums.end());
    if (maxItr != nums.end())
    {
        auto maxNum = *maxItr;

        while (maxNum >=r)
        {
            maxNum /= r;
            d++;
        }
    }

    return d;
}

void RadixSort(vector<int> &nums)
{
    int digits = maxDigits(nums);
    int radix = 1;

    while (digits--)
    {
        vector<vector<int>> buckets(10);

        for (auto n: nums)
        {
            int digit = (n/radix) % 10;

            buckets[digit].push_back(n);
        }

        int idx = 0;
        for (auto &v: buckets)
        {
            for (auto n: v)
            {
                nums[idx++] = n;
            }
        }

        radix *= 10;
    }
}
```
通过 `partial_sum` 我们可以在静态数组上实现，而不用动态数组。  
```cpp
int maxDigits(int arr[], int size)
{
    int d = 1;
    int r = 10;

    for (int idx=0; idx<size; idx++)
    {
        if (arr[idx] >= r)
        {
            d++;
            r *= 10;
        }
    }

    return d;
}

void RadixSort(int arr[], int size)
{
    int digits = maxDigits(arr, size);
    unique_ptr<int[]> tmp(new int[size]);
    int radix = 1;
#define DIGITS_CNT (10)

    for (int idx=0; idx<digits; idx++)
    {
        int cnt[DIGITS_CNT] = {};

        for (int pos=0; pos<size; pos++)
        {
            int digit = (arr[pos]/radix) % 10;

            cnt[digit]++;
        }

        // partial sum
        for (int pos=1; pos<DIGITS_CNT; pos++)
        {
            cnt[pos] += cnt[pos-1];
        }

        // start from the end
        for (int pos=size-1; 0<=pos; pos--)
        {
            int digit = (arr[pos]/radix) % 10;

            tmp[cnt[digit]-1] = arr[pos];
            cnt[digit]--;
        }

        for (int pos=0; pos<size; pos++)
        {
            arr[pos] = tmp[pos];
        }

        radix *= 10;
    }
}
```
# 选择排序
简单来说选择排序就是每次从余下序列中选取最小元素。  
```cpp
void SelectSort(int arr[], int size)
{
    for (int idx=0; idx<size; idx++)
    {
        int min = idx;

        for (int pos=idx+1; pos<size; pos++)
        {
            if (arr[min] > arr[pos])
            {
                min = pos;
            }
        }

        swap(arr[idx], arr[min]);
    }
}
```

# 堆排序
类似于选择排序，不过选择的是最大元素，而且能够高效选择最大元素。  
先了解 [大小堆](/post/常用数据结构/#大小堆) 的概念。  
最大堆建立好之后，堆顶(数组第一个元素)就是堆中最大的元素。  
不断重复获取、移除堆顶直到堆变空为止，我们就得到了排序好的序列。  

实际上我们并不需要移除堆顶，只需要与最后的元素交换，然后假装堆中元素减一执行恢复堆操作即可。  
这样的话，可以减少额外空间的使用、节省拷贝排序后数据的操作。  
实现从小到大排列的堆排序代码如下：
```cpp
void HeapSort(vector<int>& arr)
{
    int idx = arr.size()-1;

    MaxHeapMake(arr);
    while (0<idx)
    {
        swap(arr.front(), arr[idx]);
        MaxHeapFixDownEx(arr, idx, 0);

        idx--;
    }
}
```

# 插入排序
插入排序的基本思路是：  
- 单个元素的序列是有序序列  
- 依次将后续元素插入到前面的有序序列  

怎么将元素插入到前面的有序序列？  
一种方法是通过交换，前面的元素较大则交换，一直往前交换到不大于的元素为止(类似冒泡排序)。  
```cpp
void InsertSortV1(int arr[], int size)
{
    for (int idx=1; idx<size; idx++)
    {
        for (int pos=idx; 0<pos; pos--)
        {
            if (arr[pos-1] <= arr[pos])
            {
                break;
            }

            swap(arr[pos-1], arr[pos]);
        }
    }
}
```
另外一种方法就是通过后移腾出空位，编码容易出错。  
```cpp
void InsertSortV2(int arr[], int size)
{
    for (int idx=1; idx<size; idx++)
    {
        int backup = arr[idx];
        int pos = idx;

        while (0<pos)
        {
            if (arr[pos-1] <= backup)
            {
                break;
            }

            // 一边后移腾出空位，一边搜索合适位置
            arr[pos] = arr[pos-1];

            pos--;
        }
        // 1) pos==0，前面没有元素可比较，直接插入
        // 2) pos>0 && arr[pos-1] <= backup
        arr[pos] = backup;
    }
}
```

# 希尔(Shell)排序
希尔排序是分组+插入排序。  
按间距(gap)分组，组内插入排序，然后间距折半重复上述过程，直到间距为0。  
版本一：  
```cpp
void ShellSortV1(int arr[], int size)
{
    for (int gap=size/2; 0<gap; gap/=2)
    {
        for (int group=0; group<gap; group++)
        {
            for (int idx=group+gap; idx<size; idx+=gap)
            {
                for (int pos=idx; 0<pos && arr[pos-gap]>arr[pos]; pos-=gap)
                {
                    swap(arr[pos-gap], arr[pos]);
                }
            }
        }
    }
}
```
从gap开始，每个元素与自己分组内的元素进行插入排序。  
版本二：  
```cpp
void ShellSortV2(int arr[], int size)
{
    for (int gap=size/2; 0<gap; gap/=2)
    {
        for (int idx=gap; idx<size; idx++)
        {
            for (int pos=idx; 0<pos && arr[pos-gap]>arr[pos]; pos-=gap)
            {
                swap(arr[pos-gap], arr[pos]);
            }
        }
    }
}
```

# 归并排序
归并排序是递归+合并，重点在于合并(两个有序序列)。  

合并两个有序序列的步骤：  
比较两个序列的首元素，取出较小的，重复上述步骤直到有序列为空，然后从另一序列取出剩余元素。  

对于一般的序列，将其二分为相邻序列递归进行归并排序，然后合并两个有序序列得到最终的有序序列。  
易知单个元素序列是有序的，所以递归过程是收敛的(不会死循环)。  

代码如下(用queue实现简洁性、可读性会更好，但是效率差些)：  
```cpp
void MergeSortHelper(int nums[], int l, int r, int temp[])
{
    if (l >= r)
    {
        return;
    }

    int m = l + (r-l)/2; // prevent overflow

    MergeSortHelper(nums, l,   m, temp);
    MergeSortHelper(nums, m+1, r, temp);
    // [l, m] and [m+1, r] are sorted
    // merge [l, m] and [m+1, r]

    int i=l;
    int j=m+1;
    int k=l;

    while (i<=m && j<=r)
    {
        if (nums[i] < nums[j])
        {
            temp[k] = nums[i++];
        }
        else
        {
            temp[k] = nums[j++];
        }
        k++;
    }

    while (i<=m)
    {
        temp[k++] = nums[i++];
    }

    while (j<=r)
    {
        temp[k++] = nums[j++];
    }

    while (l<=--k)
    {
        nums[k] = temp[k];
    }
}

void MergeSort(int nums[], int size)
{
    int *temp = new int[size];

    MergeSortHelper(nums, 0, size-1, temp);

    delete[] temp;
}
```

# 快速排序
快速排序基本思路如下：  
1. 先从数列中选出一个数(如第一个数，也有可能随机选)作为基准数  
2. 比基准数大的数移到它的右边，小于或等于它的数移到它的左边  
3. 对于基准数左右两边区间重复第2步，直到区间内只有一个数  

其中第二步可以简单理解为挖坑填数。  
挖坑填数步骤如下：  
1. frnt = l; back = r; 将基准数挖出形成第一个坑arr[frnt]  
2. back自减然后从后往前找比基准数小的数，填坑arr[frnt]，形成新坑arr[back]  
3. frnt自增然后从前往后找比基准数大的数，填坑arr[back]，形成新坑arr[frnt]  
4. 重复执行第2，3步骤直到 frnt==back，将基准数填坑arr[frnt]  

以一个数组作为示例，取区间第一个数为基准数。  

| 0 | 1 |  2 |  3 |  4 |  5 |  6 |  7 |  8 |  9 |
| - | - | - | - | - | - | - | - | - | - |
| ~~72~~ | 6 | 57 | 88 | 60 | 42 | 83 | 73 | 48 | 85 |

找到48，此时frnt=0, back=8

| 0 | 1 |  2 |  3 |  4 |  5 |  6 |  7 |    8 |  9 |
| - | - | - | - | - | - | - | - | - | - |
| **48** | 6 | 57 | 88 | 60 | 42 | 83 | 73 | ~~48~~ | 85 |

找到88，此时frnt=3, back=8

|  0 | 1 |  2 |    3 |  4 |  5 |  6 |  7 |    8 |  9 |
| - | - | - | - | - | - | - | - | - | - |
| 48 | 6 | 57 | ~~88~~ | 60 | 42 | 83 | 73 | **88** | 85 |

找到42，此时frnt=3, back=5

|  0 | 1 |  2 |    3 |  4 |    5 |  6 |  7 |  8 |  9 |
| - | - | - | - | - | - | - | - | - | - |
| 48 | 6 | 57 | **42** | 60 | ~~42~~ | 83 | 73 | 88 | 85 |

没找到，此时frnt=5, back=5

|  0 | 1 |  2 |  3 |  4 |    5 |  6 |  7 |  8 |  9 |
| - | - | - | - | - | - | - | - | - | - |
| 48 | 6 | 57 | 42 | 60 | **72** | 83 | 73 | 88 | 85 |

从上面可以看到，比72小的数都在72左边，比72大的数都在72右边。  
代码如下：  
```cpp
void partitionArray(int arr[], int l, int r)
{
    int seperator = arr[l];
    int frnt = l;
    int back = r;

    if (l>=r)
    {
        return;
    }

    while (frnt<back)
    {
        while (frnt<back && seperator<arr[back])
        {
            back--;
        }

        if (frnt<back)
        {
            arr[frnt++] = arr[back];
        }

        while (frnt<back && seperator>=arr[frnt])
        {
            frnt++;
        }

        if (frnt<back)
        {
            arr[back--] = arr[frnt];
        }
    }

    arr[frnt] = seperator;

    partitionArray(arr, l, frnt-1);
    partitionArray(arr, frnt+1, r);
}

void QuickSort(int nums[], int size)
{
    partitionArray(nums, 0, size-1);
}
```

# 外部排序
当数据量太大不能一次放入内存排序时，我们需要借助外部存储协助排序，相对应的就是外部排序。  
外部排序与归并排序类似，本质上就是多个有序序列合并为一个有序序列。  

将数据切分为多个序列，在内存排序后单独保存文件，然后经过多次合并最终输出一个有序序列。  
合并时需要从多个文件读取数据到内存，选取最小值写入最终的文件。  

虽然可以采用归并排序一样的二路合并，但是中间(待合并)序列会很多(文件很多，不止2个)。  
每个中间序列也需要读写硬盘，而硬盘读写比内存慢多了，严重降低排序的速度。  

二路合并不行，那么就多路合并。多路合并主要需要解决选取最小值的问题。  
以K路合并为例，最直接的方法就是依次比较K路取最小值，但是复杂度是O(nk)。  
对数据结构比较熟悉的朋友，可能会想到用小堆的。这是一种解决方法，实际使用也比较多。  

还有另外的方法就是使用赢家树或者输家树(loser tree)，速度会快一些。  
参考下图基本就懂了，复杂度是O(n log k)，小堆因为要比较左右节点所以是其2倍左右。  
具体可以参考 [K路归并算法](https://en.wikipedia.org/wiki/K-way_merge_algorithm#Direct_k-way_merge) 的说明。  

赢家树(中间节点存winner)：  
![](https://upload.wikimedia.org/wikipedia/commons/c/c0/Tournament_tree.png)  
输家树(中间节点存loser)：  
![](https://upload.wikimedia.org/wikipedia/commons/4/4d/Loser_tree.png)  

