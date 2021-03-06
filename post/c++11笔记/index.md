
本文主要记录C++11标准的学习笔记。  
<!--more-->

# auto
auto可以对变量的类型自动推导，不用再写冗长的类型了，特别是迭代器变量的类型。  
需要注意的是引用类型还是要加`&`，特别是在for循环里面。否则会有复制值的性能开销。  

# decltype
获取表达式的声明类型，`decltype(auto)`与auto类似，不过会保留引用的类型。  

# for迭代
```cpp
std::array<int, 5> arr {1, 2, 3, 4, 5};
for (auto& x: arr)
{
  x *= 2;
}
// arr == { 2, 4, 6, 8, 10 }

map<int, char> m = { {1,'a'}, {3,'b'} };
for (auto &p: m)
{
    cout << p.first << ":" << p.second << endl;
}
```
使用引用需要`&`，否则是值语义。  
    
# 初始化列表
支持花括号赋初值的方式。  

# move, 右值引用
右值引用绑定的是临时变量值(右值)，通过`std::move`可以高效的移交所有权(完成赋值)。  
新增移动构造函数和移动赋值函数。  

# STL
支持unordered容器、智能指针，array，tuple等。  

STL容器可以参考 [STL容器](/post/stl容器/) 的说明。  
智能指针可以参考 [C++智能指针](/post/c++智能指针/) 的说明。   

# delete
更优雅的防止方法实现，比如说复制构造函数、复制赋值函数。  

# lamda
形式如下：  
```
[捕获变量](参数列表) -> 返回值 { 函数体 }
```
捕获变量基本形式如下：  
- [] 不捕获变量  
- [&] 以引用形式捕获外部作用域变量  
- [=] 以值形式捕获外部作用域变量  
- [=, &v] 基本同上，但变量v以引用形式捕获  
- [v] 仅捕获变量v(值形式)  

与STL中需要仿函数的算法一起使用会比较简洁。  

# 参考链接
- [现代C++教程](https://changkun.de/modern-cpp/)
- [modern cpp features](https://github.com/AnthonyCalandra/modern-cpp-features)  
