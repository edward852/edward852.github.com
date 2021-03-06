
本文主要记录位操作相关的笔记。  
<!--more-->

# 数据结构
bitmap。可以使用bitset或者vector<bool>。  

# 二进制表示中1的个数

- 直白版  
```cpp
int getBitCnt(int num)
{
    int cnt = 0;

    while (num)
    {
        if (num & 0x1)
        {
            cnt++;
        }
        
        num >>= 1;
    }

    return cnt;
}
```
- 依次清除最右边的1  
```cpp
int getBitCnt(int num)
{
    int cnt = 0;

    while (num)
    {
        num &= num - 1;
        cnt++;
    }

    return cnt;
}
```

- 分治版本  
  具体参考 [Counting 1 bits](https://everything2.com/title/Counting+1+bits) 。  
```cpp
unsigned numbits(unsigned i)
{
    const unsigned MASK1  = 0x55555555;
    const unsigned MASK2  = 0x33333333;
    const unsigned MASK4  = 0x0f0f0f0f;
    const unsigned MASK8  = 0x00ff00ff;
    const unsigned MASK16 = 0x0000ffff;

    i = (i&MASK1 ) + (i>>1 &MASK1 );
    i = (i&MASK2 ) + (i>>2 &MASK2 );
    i = (i&MASK4 ) + (i>>4 &MASK4 );
    i = (i&MASK8 ) + (i>>8 &MASK8 );
    i = (i&MASK16) + (i>>16&MASK16);

    return i;
}
```

- __builtin_popcount  
  gcc自带版本，效率比较高。  

