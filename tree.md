## 树和二叉树
​**​作者​**​：李昀卓
​**​日期​**​：2025年4月14日
### 一些定义

#### 基本术语
1.结点的度：结点拥有的子树数。

2.树的度：max结点度。

3.层次：根的层次为1。

4.深度：最大层次。

5.有序树：树中结点的各子树有次序，从左至右，最左边子树的根称第一个孩子。

6.森林：m棵互不相交的树的集合。
> **说明**：就逻辑结构而言，任何一棵树都是一个二元组`Tree=(root,F)`,其中`root`是数据，`F`是包含m棵树的森林

#### 二叉树(Binary Tree)
1.可为空。

2.子树有左右之分。

### 二叉树的操作
#### 定义二叉树的存储结构
我们这里使用链式存储结构来存储树的结点`TreeNode`
```cpp
struct TreeNode {
    char val;
    TreeNode *left;
    TreeNode *right;
    TreeNode(char x) : val(x), left(NULL), right(NULL) {}
};
```
#### 创建一个二叉树
```cpp
void createBinaryTree(TreeNode *&T) {  //我们这里采用先序建立二叉树
    char c;
    cin >> c;
    if(c == '#') {
        T = NULL;
    } else {
        T = new TreeNode(c);
        createBinaryTree(T->left);
        createBinaryTree(T->right);
    }
}
```
这里我们通过补足虚结点`#`，结合先序遍历来确定一棵树。接下来我们尝试输入中序和后序遍历的结果来确定一棵树。
这有一些难办，但没关系，我们一步一步来进行。
我们先思考后序遍历能给我们提供什么最关键的信息呢？是的，后序遍历的最后一个元素一定是根结点。
那我们如何递归进行呢？我们这时候把目光转向中序遍历，我们既然知道了根结点是什么，那么我们找到根结点在中序遍历中的位置，然后就可以得到左右子树的大小。
然后在后序遍历中再得到子树的根，重复这个过程就好了。
```cpp
TreeNode* helper(char* inOrder,int inLeft,int inRight,char* postOrder,int postLeft,int postRight,unordered_map<char, int>& indexMap) {
    if(inRight < inLeft || postRight < postLeft) return nullptr;
    TreeNode* root = new TreeNode(postOrder[postRight]);
    int index = indexMap[postOrder[postRight]];
    int leftSize = index - inLeft;

    root->left = helper(inOrder,inLeft,index-1,postOrder,postLeft,postLeft+leftSize-1,indexMap);
    root->right = helper(inOrder,index+1,inRight,postOrder,postLeft + leftSize,postRight - 1,indexMap);
    return root;
}

TreeNode* buildTree(char *inOrder,int lenInOrder,char *postOrder,int lenPostOrder) {
    unordered_map<char,int> indexMap;
    for(int i = 0; i < lenInOrder; i++) {
        indexMap[inOrder[i]] = i;  //哈希表方便查找索引
    }
    return helper(inOrder,0,lenInOrder-1,postOrder,0,lenPostOrder,indexMap);
}
```
同理可得由先序和中序也可以得到唯一二叉树，思路与这个几乎完全一样。
#### 遍历二叉树
```cpp
void preOder(TreeNode *T) {
    while(T != NULL) {
        cout<<T->val<<' ';
        preOder(T->left);
        preOder(T->right);
    }
}
```
如果说在这里你已经变写好了上面的代码，运行后会发现你的输入和输出应该相同，因为它们都是对同一棵树进行先序遍历的结果。并且很显然，中序遍历和后序遍历的代码只需要调整三行代码的顺序即可。
#### 线索二叉树的构建
我们为每一个结点添加两个`tag`标签来判断其左右是结点还是空，`tag=0`表示指向孩子，`tag=1`表示指向直接前驱或后继。更改后的存储结构如下：
```cpp
struct bithrnode {
    char data;
    bithrnode *lchild, *rchild;
    int ltag, rtag;
    bithrnode(char x) : data(x), lchild(NULL), rchild(NULL), ltag(0), rtag(0) {}
};
```
接下来我们来创建给定二叉树的后续线索树，我们的思路是，对于非空结点，我们先递归线索化左子树，再线索化右子树，然后判断有没有到达叶子结点。
如果说左子树为空，那么我们设置一个为空的pre结点，让左子树的前驱指向pre结点。
如果pre非空并且没有右子树，那么我们更新pre的直接后继为现在结点。
最后更新pre结点为现在的结点。
这里pre结点很关键，它就如同名字一样是记录前一个结点的，方便我们构建前驱和后继。
```cpp
bithrnode *pre = NULL;
void createThreadBinaryTree(bithrnode *p) {
    if(p) {
        createThreadBinaryTree(p->lchild);
        createThreadBinaryTree(p->rchild);
        if(p->lchild == NULL) {
            p->ltag = 1;
            p->lchild = pre;
        }
        if(pre && pre->rchild == NULL) {
            pre->rtag = 1;
            pre->rchild = p;
        }
        pre = p;
    }
}
```
### 二叉树应用：计算表达式
我们先实现二叉树计算波兰式：
```cpp
TreeNode* buildExpressionTree(vector<string>& tokens, int& index) {
    if (index >= tokens.size()) return nullptr;
    
    string current = tokens[index++];
    TreeNode* node = new TreeNode(current);
    
    // 如果是操作符，则递归构建左右子树
    if (isOperator(current)) {
        node->left = buildExpressionTree(tokens, index);
        node->right = buildExpressionTree(tokens, index);
    }
    // 操作数作为叶子节点，无需处理子树
    
    return node;
}
```
我们引入一中类似文件输入的处理字符串的方法：
```cpp
string input;
getline(cin, input);
    
// 将输入拆分为tokens
vector<string> tokens;
stringstream ss(input);
string token;
while (ss >> token) {
    tokens.push_back(token);
}
```
由于波兰式就是方便读取计算的方便形式，对于构建好的表达式树，我们递归求解即可：
```cpp
int valueexptree(TreeNode *T) {
    if (T == NULL) return 0;
    if (T->left == NULL && T->right == NULL) return stoi(T->val); //stoi将字符串转换为整数
    
    int leftValue = valueexptree(T->left);
    int rightValue = valueexptree(T->right);
    
    return getvalue(T->val, leftValue, rightValue);
}
```
