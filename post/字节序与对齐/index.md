
本文主要介绍C/C++中的字节序(含位域)和对齐。  
<!--more-->
    
# 基本类型大小
| 类型        | 32位  | 64位  |
|-------------|-------|-------|
| char        | 1     | 1     |
| short       | 2     | 2     |
| int         | 4     | 4     |
| **long**    | **4** | **8** |
| long long   | 8     | 8     |
| **pointer** | **4** | **8** |
| float       | 4     | 4     |
| double      | 8     | 8     |

signed和unsigned只影响最高位的含义，不影响类型大小。  
常见系统32位和64位基本类型大小的差异主要在long和pointer，其他是一致的。  

# 对齐
出于性能或者地址合法性(如MIPS平台)的考虑，变量存放地址要求为该变量类型对齐值的倍数。  
基本类型默认对齐值为类型大小，结构体默认对齐值为各个成员对齐值的最大值。  
通过 `#pragma pack(数值)` 可以指定对齐值，`#pragma pack()` 恢复默认对齐值。  
实际对齐值是默认对齐值和指定对齐值中较小的值。  

## 默认对齐
```cpp
struct DefSt{
    char a;  // 1的倍数，变量地址+0~0
    int b;   // 4的倍数，变量地址+4~7
    short c; // 2的倍数，变量地址+8~9
} ds[2];  // 4的倍数，变量地址+12

void structSizeDef()
{
    cout << "sizeof(DefSt): " << sizeof(DefSt) << endl;
    // 大小为12
    
    cout << "ds[0] @" << (void *)&ds[0] << endl;
    cout << "ds[0].a @" << (void *)&ds[0].a << endl;
    cout << "ds[0].b @" << (void *)&ds[0].b << endl;
    cout << "ds[0].c @" << (void *)&ds[0].c << endl;
    cout << "ds[1] @" << (void *)&ds[1] << endl;
}
```
不难看出结构体大小是其对齐值的倍数(否则后续变量地址就可能不对齐)。  

## 指定对齐
指定对齐值为1就是我们通常意义上理解的紧凑布局。  
```cpp
#pragma pack(1)
struct SpecSt{
    char a;  // 变量地址+0~0
    int b;   // 变量地址+1~4
    short c; // 变量地址+5~6
} ss0, ss1;  // 变量地址+7
// 恢复默认对齐
#pragma pack()

void structSizeSpec()
{
    cout << "sizeof(SpecSt): " << sizeof(SpecSt) << endl;
    // 大小为7
    
    cout << "ss0 @" << (void *)&ss0 << endl;
    cout << "ss0.a @" << (void *)&ss0.a << endl;
    cout << "ss0.b @" << (void *)&ss0.b << endl;
    cout << "ss0.c @" << (void *)&ss0.c << endl;
    cout << "ss1 @" << (void *)&ss1 << endl;
}
```
GCC编译器的话，也可以用`__attribute__ ((__packed__))`。  

# 字节序
字节序就是字节存放的顺序，分为大端和小端两种字节序。  
大端字节序先存放高位的字节。  
小端字节序先存放低位的字节。  
网络字节序就是大端字节序。  
主机序就是系统的字节序，一般由CPU类型(x86, arm, mips...)决定。  

通过检查整型1首地址的存放内容即可知道字节序类型：  
```cpp
void byte_order()
{
    int num = 1;

    string order = (1 == *(char *)&num)? "little": "big";
    cout << "Your system use " << order << " endian" << endl;
}
```
Linux可以通过endian.h中的宏 `__BYTE_ORDER` 判断是 `__LITTLE_ENDIAN` 还是 `__BIG_ENDIAN` 。  

## 转换
通过 `ntohs` 或 `ntohl` 转换网络序为主机序。  
通过 `htons` 或 `htonl` 转换主机序为网络序。  
`n`是network缩写，`h`是host缩写，`s`是short类型缩写，`l`是long类型缩写。  

## 位域
位域是指按位存储的字段，存储的顺序跟系统字节序一致。  
小端先从低位开始存储，大端先从高位开始存储。  
如果需要兼容不同字节序的平台，那么需要根据宏分别定义(位域定义顺序相反)。  

```cpp
// f0, f1, f2, f3共用同一字节中的不同位
struct ModFlags
{
    char f0: 2;
    char f1: 2;
    char f2: 2;
    char f3: 2;
} mf;

void bit_field_order()
{
    uint8_t *ptr = (uint8_t *)&mf;

    cout << "sizeof(ModFlags): " << sizeof(ModFlags) << endl;
    // 大小为1(字节)
    
    mf.f0 = 3;
    mf.f3 = 1;
    cout << "mf: 0x" << hex << (int)*ptr << endl;
    // 小端是 0x43
    // 大端是 0xC1
}
```
