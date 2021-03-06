
本文主要记录二叉树相关的笔记。  
<!--more-->

# 遍历
二叉树节点定义如下：  
```cpp
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};
```
N叉树节点定义如下：  
```cpp
class Node {
public:
    int val;
    vector<Node*> children;

    Node() {}

    Node(int _val) {
        val = _val;
    }

    Node(int _val, vector<Node*> _children) {
        val = _val;
        children = _children;
    }
};
```

## 深度优先
### 先序
以中左右顺序遍历。  

- 递归  
```cpp
void dfsPreOrder(TreeNode *root, vector<int> &res)
{
    if (!root) { return; }

    res.push_back(root->val);
    dfsPreOrder(root->left, res);
    dfsPreOrder(root->right, res);
}
```

- 迭代  
```cpp
void dfsPreOrderIter(TreeNode *root, vector<int> &res)
{
    stack<TreeNode *> s;

    if (root)
    {
        s.push(root);
    }

    while (!s.empty())
    {
        TreeNode *node = s.top();

        res.push_back(node->val);
        s.pop();

        if (node->right)
        {
            s.push(node->right);
        }
        if (node->left)
        {
            s.push(node->left);
        }
    }
}
```
可以推广到N叉树的先序：  
```cpp
vector<int> postorder(Node* root) {
    vector<int> res;
    stack<Node*> s;

    if (root)
    {
        s.push(root);
    }

    while (!s.empty())
    {
        Node* node = s.top();
        s.pop();

        auto &v = node->children;
        for (auto it=v.rbegin(); it!=v.rend(); it++)
        {
            s.push(*it);
        }

        res.push_back(node->val);
    }

    return res;
}
```

### 中序
以左中右顺序遍历。  

- 递归  
```cpp
void dfsInOrder(TreeNode *root, vector<int> &res)
{
    if (!root) { return; }

    dfsInOrder(root->left, res);
    res.push_back(root->val);
    dfsInOrder(root->right, res);
}
```

- 迭代  
```cpp
void dfsInOrderIter(TreeNode *root, vector<int> &res)
{
    stack<TreeNode *> s;
    TreeNode *node = root;

    while (!s.empty() || node)
    {
        // 从当前节点一直往左走到底路线上的节点压栈
        while (node)
        {
            s.push(node);
            node = node->left;
        }

        // s一定不为空(不管node是否为NULL)
        // 节点没有左子树或者左子树已经遍历
        node = s.top();

        res.push_back(node->val);
        s.pop();
            
        // 迭代处理右子树
        node = node->right;
    }
}
```

### 后序
以左右中顺序遍历。  

- 递归  
```cpp
void dfsPostOrder(TreeNode *root, vector<int> &res)
{
    if (!root) { return; }

    dfsPostOrder(root->left, res);
    dfsPostOrder(root->right, res);
    res.push_back(root->val);
}
```

- 迭代一  
以中右左的顺序遍历(类似先序)，然后把结果反转即为左右中的顺序。  
```cpp
void dfsPostOrderIter1(TreeNode *root, vector<int> &res)
{
    stack<TreeNode *> s;

    if (root)
    {
        s.push(root);
    }

    while (!s.empty())
    {
        TreeNode *node = s.top();

        res.push_back(node->val);
        s.pop();

        if (node->left)
        {
            s.push(node->left);
        }
        if (node->right)
        {
            s.push(node->right);
        }
    }

    reverse(res.begin(), res.end());
}
```
可以推广到N叉树的后序：  
```cpp
vector<int> postorder(Node* root) {
    vector<int> res;
    stack<Node*> s;

    if (root)
    {
        s.push(root);
    }

    while (!s.empty())
    {
        Node* node = s.top();
        s.pop();

        for (auto child: node->children)
        {
            s.push(child);
        }

        res.push_back(node->val);
    }
    reverse(res.begin(), res.end());

    return res;
}
```

- 迭代二  
记录左、右子树的遍历状态，当左、右子树都已遍历才退栈。  
网上还有记录最近遍历节点的方法，这里不赘述。  
虽然不是最优版本，但是比较好理解、好记，还可以推广到N叉树。  
```cpp
void dfsPostOrderIter2(TreeNode *root, vector<int> &res)
{
    stack<TreeNode *> s;
    unordered_map<TreeNode *, char> m;

    if (root)
    {
        s.push(root);
    }

    while (!s.empty())
    {
        TreeNode *node = s.top();
        char cnt = m[node];

        // 0:左子树未遍历 1:左子树已安排遍历 2:右子树已安排遍历
        if (cnt < 2)
        {
            m[node]++;

            if (0==cnt)
            {
                if (node->left)
                {
                    s.push(node->left);
                }
            }
            else
            {
                if (node->right)
                {
                    s.push(node->right);
                }
            }

            // 迭代处理子树
            continue;
        }

        // 左、右子树都已遍历
        res.push_back(node->val);
        s.pop();
        m.erase(node);
    }
}
```
推广到N叉树的后序：  
```cpp
vector<int> postorder(Node *root) {
    vector<int> res;
    stack<Node *> s;
    unordered_map<Node *, int> m;

    if (root)
    {
        s.push(root);
    }

    while (!s.empty())
    {
        Node *node = s.top();
        int cnt = m[node];
        auto &v = node->children; // 不含NULL

        if (cnt<v.size())
        {
            s.push(v[cnt]);

            m[node]++;
            continue;
        }

        res.push_back(node->val);
        s.pop();
        m.erase(node);
    }

    return res;
}
```

## 广度优先

### 版本一
```cpp
void bfsOrder(TreeNode *root, vector<int> &res)
{
    queue<TreeNode *> q;

    if (root)
    {
        q.push(root);
    }

    while(!q.empty())
    {
        TreeNode *node = q.front();

        res.push_back(node->val);
        q.pop();

        if (node->left)
        {
            q.push(node->left);
        }
        if (node->right)
        {
            q.push(node->right);
        }
    }
}
```

### 版本二
在版本一基础上实现逐层遍历。  
```cpp
void bfsOrderV2(TreeNode *root, vector<int> &res)
{
    queue<TreeNode *> q;

    if (root)
    {
        q.push(root);
    }

    while(!q.empty())
    {
        int total = q.size();
        
        // 此时此处队列内所有节点(对应下面循环的node)都在同一层
        while (total--)
        {
            TreeNode *node = q.front();

            res.push_back(node->val);
            q.pop();

            if (node->left)
            {
                q.push(node->left);
            }
            if (node->right)
            {
                q.push(node->right);
            }
        }
    }
}
```

# 树高
- 递归  
```cpp
int maxDepth(Node *root) {
    if (nullptr == root) { return 0; }
     
    int depth = 0;
    for (auto child: root->children)
    {
        depth = max(depth, maxDepth(child));
    }

    return 1 + depth;
}
```
- 广度优先  
```cpp
int maxDepth(Node *root) {
    queue<Node *> q;
    
    if (root)
    {
        q.push(root);
    }

    int depth = 0;
    while (!q.empty())
    {
        depth += 1;

        int cnt = q.size();
        for (int idx=0; idx<cnt; idx++)
        {
            auto node = q.front();
            q.pop();

            for (auto child : node->children)
            {
                q.push(child);
            }
        }
    }

    return depth;
}
```

# 反推
先序和后序不能确定二叉树，因为两者只反映了结点之间的父子关系，没有反映出左右关系。  
举反例可以证明，先序`0 1`和后序`1 0`有两种可能。  

知道先序和中序确定二叉树比较简单，以先序为主在中序迭代划分左右子树即可。  
知道中序和后序确定二叉树麻烦些，不过有个小技巧可以加速。  
后序反转是变种先序，即中右左顺序。然后我们就可以转化成上面类似的问题进行处理了。  

# 二叉查找树
有序二叉树。任意节点，左子树的值都比该节点值小，右子树的值都比该节点值大。  

## 搜索
- 递归  
```cpp
TreeNode* searchBST(TreeNode* root, int val) {
    if (!root)
    {
        return NULL;
    }

    int num = root->val;

    if (num > val)
    {
        return searchBST(root->left, val);
    }
    else if (num < val)
    {
        return searchBST(root->right, val);
    }
    
    return root;
}
```
- 迭代  
```cpp
TreeNode* searchBST(TreeNode* root, int val) {
    TreeNode* node = root;

    while (node)
    {
        int num = node->val;

        if (num > val)
        {
            node = node->left;
        }
        else if (num < val)
        {
            node = node->right;
        }
        
        break;
    }

    return node;
}
```

## 平衡
- 方法一  
  比较易懂的方法就是中序遍历存入数组即为有序序列，再从数组重构二叉树。  
  缺点是性能不佳，还需要额外O(n)空间。  
- 方法二  
  采用 [DSW算法](https://en.wikipedia.org/wiki/Day%E2%80%93Stout%E2%80%93Warren_algorithm) 可以在O(n)时间内，O(1)空间内完成。  
