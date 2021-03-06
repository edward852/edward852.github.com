
本文主要记录一些常见算法。  
<!--more-->

# 基本类型
## 枚举算法
枚举问题的所有情况并逐一检查得到问题的解。  

### 回溯算法
枚举算法可以通过回溯实现。  
回溯算法通过不断地尝试(下一种可能)、检查、回滚得到问题的解。  
回溯算法本质是多叉决策树的遍历。  
比较典型的就是八皇后问题、全排列问题。  

## 分治算法
将问题分解为多个规模更小、同类型的子问题逐个解决，最终组合得到问题的解。  
本质上类似于数学的归纳法，基于**基础解**和**递推公式**解决问题。  

分治算法一般可以通过递归解决问题。  
递归简洁性、可读性较好，但是可能引起栈溢出，另外性能可能也不是最佳。  
因此一般会改成迭代的形式，或者借助堆上的栈数据结构模拟。  
比较典型的是归并排序、快速排序。  

## 贪婪算法
每一步只根据当前已知条件采取最优的选择，以求问题的最优解。  
贪婪算法不一定能得到最优解，但是合适的选择策略能够得到最优解。  
比较典型的是赫夫曼编码树。  

## 动态规划
将问题分解为多个(相关)简化子问题逐个解决，保存子问题结果以便后续复用，最终组合得到最优解。  
动态规划其实也是分治，主要的改进就是缓存**重复子问题**的结果、避免不必要的重复运算。  
比较典型的是最长公共子序列、最短路径问题。  

## 随机算法
涉及到随机数使用的算法。  
可以减少最坏情况的发生，比如说快排(QuickSort)中pivot的选择。  
也可以模拟概率事件，比如说SkipList中的插入操作。  

# 常见算法

## 排序算法
具体可以参考 [常见排序算法](/post/常见排序算法/) 。  

## 二分查找
基于有序序列的查找，能够在O(logN)内快速查找。  
STL中可以使用`binary_search`, `lower_bound`, `upper_bound` 。  
简化版本可以参考 [这里](https://github.com/edward852/algorithm/tree/master/binary_search) 。
另外golang的实现也挺有想法，没有使用泛型和接口，一个闭包就搞定。  
另外二分查找可以快速定位bug在哪个版本引入。  

## 一致性哈希
主要用于解决分布式缓存尽量选择相同节点(即使发生扩容或缩容)。  
具体参考 [Consistent hashing](https://en.wikipedia.org/wiki/Consistent_hashing) 以及[Consistent Hashing with Bounded Loads](https://ai.googleblog.com/2017/04/consistent-hashing-with-bounded-loads.html)的说明。  

## 广度优先搜索
按照广度优先搜索解空间，一般通过队列实现。  

## 深度优先搜索
按照深度优先搜索解空间，一般通过递归或者栈实现。  

## work stealing
参考[wiki](https://en.wikipedia.org/wiki/Work_stealing)上的说明。  

## 随机抽样 Reservoir sampling
参考[wiki](https://en.wikipedia.org/wiki/Reservoir_sampling)上的说明。  
可以采用双指针(或迭代器)，以`k/i`的概率移动慢指针(迭代器)。  

## Approximate counting algorithm
近似计数算法，用少量空间进行大数据量的计数(统计)。  
具体参考[wiki](https://en.wikipedia.org/wiki/Approximate_counting_algorithm)上面的说明。  

## HyperLogLog
具体参考[wiki](https://en.wikipedia.org/wiki/HyperLogLog)上面的说明。  

## 最短路径
Dijkstra算法，贪婪+动态规划解决正权重图的最短路径问题。  
简单来说就是访问已知最短路径节点，更新其连接节点路径长度，再从未访问节点选最短的重复上述过程。  
具体可以参考 [维基](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm) 上的说明。  
```cpp
using edge=pair<int, int>; // srcVertex, dstVertex
using weight=int; // keep consistent with MAX_WEIGHT, MIN_WEIGHT when using double

class PositiveWeightGraph {
    using vertex_cost=pair<weight, int>;

    map<edge, weight> m;
    vector<vector<int>> rel; // relationship between vertex
    vector<weight> dist; // distance from vertex 1
    int N; // number of vertex

    void dijkstra() {
        priority_queue<vertex_cost, vector<vertex_cost>, greater<vertex_cost>> q;

        q.push(make_pair(MIN_WEIGHT, 1));
        while (!q.empty())
        {
            auto p = q.top(); // min unvisited dist[]
            q.pop();

            int u = p.second;
            if (p.first > dist[u]) // visited
            {
                continue;
            }

            for (auto v: rel[u]) // neighbor relaxing
            {
                int dist1uv = dist[u] + getWeight(make_pair(u, v));

                if (dist1uv < dist[v])
                {
                    dist[v] = dist1uv;
                    q.push(make_pair(dist1uv, v));
                }
            }
        }
    }

public:
    static const weight MAX_WEIGHT;
    static const weight MIN_WEIGHT;

    PositiveWeightGraph(const map<edge, weight> &g) {
        N = 0;
        for (auto &kv: g) {
            int src = kv.first.first;
            int dst = kv.first.second;

            if (src > dst) swap(src, dst);

            m.insert(make_pair(make_pair(src, dst), kv.second));
            N = max(N, dst);
        }

        dist.resize(N+1); // dist[0] unused
        dist[1] = MIN_WEIGHT;
        for (int i=2; i<=N; i++)
        {
            dist[i] = MAX_WEIGHT;
        }

        rel.resize(N+1); // rel[0] unused
        for (auto &kv: m) {
            int src = kv.first.first;
            int dst = kv.first.second;

            rel[src].push_back(dst);
            rel[dst].push_back(src);
        }

        dijkstra();
    }

    weight getWeight(edge e) const {
        if (e.first > e.second) swap(e.first, e.second);

        return m.at(e);
    }

    weight getShortestDist(int vertex) {
        if (1<=vertex && vertex<=N)
        {
            return dist[vertex];
        }

        return MAX_WEIGHT;
    }

    void showShortestDist() {
        cout << "Shortest distance from vertex 1 to" << endl;
        for (int v=2; v<=N; v++)
        {
            auto sd = getShortestDist(v);

            cout << "vertex " << v << ":";
            if (MAX_WEIGHT == sd)
            {
                cout << "INFINITY" << endl;
            }
            else
            {
                cout << sd << endl;
            }
        }
    }
};

const weight PositiveWeightGraph::MAX_WEIGHT = INT_MAX;
const weight PositiveWeightGraph::MIN_WEIGHT = 0;
```


# 参考链接
- [程序算法与人生选择](https://coolshell.cn/articles/8790.html) 
- [程序员必须掌握哪些算法？](https://www.zhihu.com/question/23148377)
- [Algorithm and its types](https://www.includehelp.com/data-structure-tutorial/algorithm-and-its-types.aspx)
- [互联网公司最常见的面试算法题有哪些？](https://www.zhihu.com/question/24964987)
