
给定含有N个互不相同元素的序列，输出这N个元素的所有排列（全排列）。  
<!--more-->

以字符串"abc"为例，全排列包括"abc","acb","bac","bca","cab","cba"这6个结果。  
从排列组合来计算6=3!=3\*2\*1，也就是我们依次选择3个元素：  
- 第一个元素有3种选择(a,b,c)，假设选择c  
- 第二个元素只有2种选择(a,b)，假设选择b  
- 第三个元素只有1种选择(a)  

# 递归

首先我们采用递归(+回溯)的方式实现。  
假设我们已经完成N-1个元素的全排列，那么怎么完成N个元素的全排列？  

由上面的简介可知，N个元素的全排列结果数目等于N! = N\*(N-1)!。  
也就是说遍历任选N个元素中的一个`+`剩下N-1个元素全排列。  

## 版本一

代码实现如下：

```cpp
void permuteV1(char arr[], int size, int selIdx)
{
    if (selIdx >= size)
    {
        for (int idx=0; idx<size; idx++)
        {
            cout<<arr[idx];
        }
        cout<<endl;

        return;
    }

    for (int idx=selIdx; idx<size; idx++)
    {
        // selIdx之前为已选择元素，之后(包括selIdx)为待选择元素
        // 从待选择元素从选一个交换到selIdx，产生新的全排列
        swap(arr[selIdx], arr[idx]);
        permuteV1(arr, size, selIdx+1);

        // 上面选择了idx元素，接下来选择下一个待选择元素(idx+1)，首先需要撤销选择idx元素
        swap(arr[selIdx], arr[idx]);
    }
}
```

对于\"abc\"，上述代码输出结果为\"abc\",\"acb\",\"bac\",\"bca\",\"cba\",\"cab\"。  
细心的朋友可能会发现先输出\"cba\"，然后再输出\"cab\"。  
是的，上述代码输出次序并不是按照 [字典序](https://zh.wikipedia.org/wiki/%E5%AD%97%E5%85%B8%E5%BA%8F) 。  

## 版本二

在序列本身有序(升序)的前提下，把上面的swap改成如下操作即可实现字典序输出：  
-   把\[selIdx, idx)区间的元素整体后移一位  
-   再把idx元素放到selIdx位置  

代码实现如下：  
```cpp
void selElem(char arr[], int selIdx, int idx)
{
    int offset = idx-selIdx;
    char backup = arr[idx];

    memmove(arr+selIdx+1, arr+selIdx, offset);
    arr[selIdx] = backup;
}

void unselElem(char arr[], int selIdx, int idx)
{
    int offset = idx-selIdx;
    char backup = arr[selIdx];

    memmove(arr+selIdx, arr+selIdx+1, offset);
    arr[idx] = backup;
}

void permuteV2(char arr[], int size, int selIdx)
{
    if (selIdx >= size)
    {
        for (int idx=0; idx<size; idx++)
        {
            cout<<arr[idx];
        }
        cout<<endl;

        return;
    }

    for (int idx=selIdx; idx<size; idx++)
    {
        // selIdx之前为已选择元素，之后(包括selIdx)为待选择元素
        // 从待选择元素从选一个移到selIdx，产生新的排列
        selElem(arr, selIdx, idx);
        permuteV2(arr, size, selIdx+1);

        // 上面选择了idx元素，接下来选择下一个待选择元素(idx+1)，首先需要撤销选择idx元素
        unselElem(arr, selIdx, idx);
    }
}
```

当然从效率上来说，相比swap交换，上述代码效率有所降低。  

# 非递归

要以非递归方式实现全排列，首先要解决的是根据当前排列得到下一个排列。  
另外需要注意的是全排列中所有排列也是有顺序的，比较自然的想法就是某种意义上的"升序"。  
以"abc"序列来说，"bac"应该在"bca"之前，因为这两个排列都以b开头，由后面的a和c决定顺序。  

## 字典序

上述顺序其实就是 [字典序](https://zh.wikipedia.org/wiki/%E5%AD%97%E5%85%B8%E5%BA%8F)，简单来说就是以字母顺序依次比较序列中元素，直到决出顺序。  

那么如何得到字典序的下一个排列？算法如下：  
- 从排列末尾寻找相邻升序子序列seq[i]<seq[i+1]，如果不存在说明是最后的排列  
- 从排列末尾寻找seq[i]<seq[j] (由上述步骤可知j一定存在)  
- 交换seq[i]与seq[j]  
- 反转seq[i+1]之后的元素  

其实i+1元素前面是升序，后面是降序。  
降序说明子序列已经遍历完，i元素需要更新更大的值，所以从右边选取。  
更新完i元素之后反转是因为要开始新的一轮子序列全排列(降序反转后为升序)。  

代码实现如下：  

```cpp
int hasNextPermutation(char arr[], int size)
{
    int i = size-2;
    int j = 0;

    // find seq[i]<seq[i+1]
    while (0<=i)
    {
        if (arr[i] < arr[i+1])
        {
            break;
        }

        i--;
    }

    // not found
    if (0>i)
    {
        return 0;
    }

    // find seq[j]>seq[i]
    j=size-1;
    while (arr[i] >= arr[j])
    {
        j--;
    }

    swap(arr[i], arr[j]);
    reverse(arr+i+1, arr+size);

    return 1;
}

void permuteIter(char arr[], int size)
{
    do
    {
        for (int idx=0; idx<size; idx++)
        {
            cout<<arr[idx];
        }
        cout<<endl;
    } while (hasNextPermutation(arr, size));
}
```

## STL之next_permutation
实际上STL已经提供了这样的算法，通过next_permutation可以获取下一个排列。  
示例代码如下：  
```cpp
void permuteStl(string& str)
{
    do
    {
        cout<<str<<endl;
    } while(next_permutation(str.begin(), str.end()));
}
```

变种
====

上述讨论的序列中元素是互不相同的，其中一种变种就是允许重复元素。  
由于非递归采用的是字典序保持"升序"，所以不存在问题。  
而递归方法会重复打印，需要调整为在待选择元素中选择下一个与当前选择不同的元素。  

```cpp
void permuteV2dup(char arr[], int size, int selIdx)
{
    if (selIdx >= size)
    {
        for (int idx=0; idx<size; idx++)
        {
            cout<<arr[idx];
        }
        cout<<" ";

        return;
    }

    for (int idx=selIdx; idx<size; idx++)
    {
        // selIdx之前为已选择元素，之后(包括selIdx)为待选择元素
        // 从待选择元素从选一个移到selIdx，产生新的排列
        selElem(arr, selIdx, idx);
        permuteV2dup(arr, size, selIdx+1);

        // 上面选择了idx元素，接下来选择下一个待选择元素(idx+1)，首先需要撤销选择idx元素
        unselElem(arr, selIdx, idx);

        // 选择下一个与当前选择不同的元素
        while (idx+1<size && arr[idx]==arr[idx+1])
        {
            idx++;
        }
    }
}
```
