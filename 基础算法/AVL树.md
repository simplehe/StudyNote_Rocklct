### AVL树
经典平衡树, 所有操作的最坏复杂度都是O(logn)的。

二叉查找树就不用多说了，AVL树就是一个平衡的二叉查找树。avl树中，任何节点的两个子树高度差都不能超过1.增加和删除节点都需要通过旋转操作来保持平衡。

#### AVL树的定义

通过编程实现AVL树，定义这个数据结构需要以下几个信息。节点结构如下定义

1.节点的权值
2.左节点指针(索引)
3.右节点指针(索引)
4.这个节点的高度

规定叶节点的高度为0

``` c++
struct Node{
 Node_t left;
 Node_t right;
 int height;
 Type data;
};

```

#### AVL树失衡

插入节点时，我们一个一个节点进行插入。遇到**失衡**的情况，就进行旋转调整。

失衡一共有四种情况，本质都是左右两边发生高度差大于1的情况。

![](image/avl1.jpg)

**如图可知，失衡可以大致分为四种情况LL LR RL RR**

LL意为往某节点的左子树的左子树插入节点的时候发生失衡。同理LR为在某节点左子树的右子树上插入节点发生失衡。每种失衡情况都有对应的解决方法。


#### AVL树旋转调整

对于每一种情况都有对应的调整方法。

**LL和RR旋转的时候万分注意记得调整每个节点的高度**

##### LL

![](image/avlll.jpg)

LL的情况要**进行右旋**，根据代码可以看出过程。并且旋转的时候要返回根节点。


**LL旋转的时候万分注意记得调整每个节点的高度**

``` c++
static Node* left_left_rotation(AVLTree k2)
{
    AVLTree k1;

    k1 = k2->left;
    k2->left = k1->right;
    k1->right = k2;

    k2->height = MAX( HEIGHT(k2->left), HEIGHT(k2->right)) + 1;
    k1->height = MAX( HEIGHT(k1->left), k2->height) + 1;

    return k1;
}
```

由于LL旋转的时候，注意图中，X,Y,Z的节点的高度都是没有发生改变的，所有无需去更新这两些节点的高度，只需要关注k1，k2,即可，旋转后，显然我们需要先更新k2,再更新k1。

##### RR

![](image/avlrr.jpg)

RR的**左旋**代码,进行一次坐旋转

**RR旋转的时候万分注意记得调整每个节点的高度**

``` c++
static Node* right_right_rotation(AVLTree k1)
{
    AVLTree k2;

    k2 = k1->right;
    k1->right = k2->left;
    k2->left = k1;

    k1->height = MAX( HEIGHT(k1->left), HEIGHT(k1->right)) + 1;
    k2->height = MAX( HEIGHT(k2->right), k1->height) + 1;

    return k2;
}
```

高度调整的理由同LL。

##### LR

![](image/avllr.jpg)

LR 的旋转就不是一次旋转可以解决的了，必须先对调整节点的左节点进行一次左旋转(即RR旋转)，然后再对调整节点进行一次右旋转(LL旋转)。

``` c++
static Node* left_right_rotation(AVLTree k3)
{
    k3->left = right_right_rotation(k3->left);

    return left_left_rotation(k3);
}
```

##### RL

![](image/avlrl.jpg)

RL同理

``` c++
static Node* right_left_rotation(AVLTree k1)
{
    k1->right = left_left_rotation(k1->right);

    return right_right_rotation(k1);
}
```

#### AVL树的插入

avl树进行插入操作，就是先按照普通二叉排序树的方法进行插入，比节点小的放到右子树，否则放到左子树。这里要注意的是插入操作是个递归的操作，流程就是如果把要传入的节点一层一层传递给自己的子节点进行插入，每层节点插入传递操作完成后，都要判断一下节点高度差，看看是不是发生失衡，如果发生失衡就要进行旋转。旋转完后更新节点的高度。这样才是一个完整的插入过程，每次插入节点都要把这个插入过程递归的传递下去。

见代码

``` c++
Node* avltree_insert(AVLTree tree, Type key)
{
    if (tree == NULL)
    {
        // 新建节点
        tree = avltree_create_node(key, NULL, NULL);
        if (tree==NULL)
        {
            printf("ERROR: create avltree node failed!\n");
            return NULL;
        }
    }
    else if (key < tree->key) // 应该将key插入到"tree的左子树"的情况
    {
        tree->left = avltree_insert(tree->left, key);//递归传递插入操作
        // 插入节点后，若AVL树失去平衡，则进行相应的调节。
        if (HEIGHT(tree->left) - HEIGHT(tree->right) == 2)
        {
            if (key < tree->left->key)
                tree = left_left_rotation(tree);
            else
                tree = left_right_rotation(tree);
        }
    }
    else if (key > tree->key) // 应该将key插入到"tree的右子树"的情况
    {
        tree->right = avltree_insert(tree->right, key);
        // 插入节点后，若AVL树失去平衡，则进行相应的调节。
        if (HEIGHT(tree->right) - HEIGHT(tree->left) == 2)
        {
            if (key > tree->right->key)
                tree = right_right_rotation(tree);
            else
                tree = right_left_rotation(tree);
        }
    }
    else //key == tree->key)
    {
        printf("添加失败：不允许添加相同的节点！\n");
    }

    tree->height = MAX( HEIGHT(tree->left), HEIGHT(tree->right)) + 1;

    return tree;
}
```

#### AVL树删除

跟插入类似。假设要删除某个value值得节点，从根节点开始向下寻找，然后根据大小把删除操作递归给左右子树，当发现是要删除的节点时判断左右子树哪个高度更大，然后拿高度更大的那边的节点来补。比如某个要被删除的节点左子树高于右子树，那就把**左子树最大的节点**找到放上来作为根节点，反之右子树取最小的。这样就能保证删除该节点后不失衡。然而这还没完，不失衡的仅仅是删除节点这个位置的子树，**递归完成后还要继续对每个父节点判断是否失衡**，进行旋转调整，这就跟插入时候的差不多了。

这里要注意，**跟二叉搜索树一样**，把左子树最大节点放上来的时候要用搜索的方式往左子树搜索，不能直接就把原节点的左子节点拿过来用，因为要保证取到左边最大的节点放上来才不会破坏原则。

当找到左边最大的节点，也要递归进行删除操作。除此之外，把最大的节点补回后，回溯时还要对每个父节点判断有没有失衡，需要调整平衡。从 targetNode 开始一路向上找到失衡点。此时只有 LL 和 RR 两种失衡情况，判断起来相对容易些

``` c++
static Node* delete_node(AVLTree tree, Node *z)
{
    // 根为空 或者 没有要删除的节点，直接返回NULL。
    if (tree==NULL || z==NULL)
        return NULL;

    if (z->key < tree->key)        // 待删除的节点在"tree的左子树"中
    {
        tree->left = delete_node(tree->left, z);
        // 删除节点后，若AVL树失去平衡，则进行相应的调节。
        if (HEIGHT(tree->right) - HEIGHT(tree->left) == 2)
        {
            Node *r =  tree->right;
            if (HEIGHT(r->left) > HEIGHT(r->right))
                tree = right_left_rotation(tree);
            else
                tree = right_right_rotation(tree);
        }
    }
    else if (z->key > tree->key)// 待删除的节点在"tree的右子树"中
    {
        tree->right = delete_node(tree->right, z);
        // 删除节点后，若AVL树失去平衡，则进行相应的调节。
        if (HEIGHT(tree->left) - HEIGHT(tree->right) == 2)
        {
            Node *l =  tree->left;
            if (HEIGHT(l->right) > HEIGHT(l->left))
                tree = left_right_rotation(tree);
            else
                tree = left_left_rotation(tree);
        }
    }
    else    // tree是对应要删除的节点。
    {
        // tree的左右孩子都非空
        if ((tree->left) && (tree->right))
        {
            if (HEIGHT(tree->left) > HEIGHT(tree->right))
            {
                // 如果tree的左子树比右子树高；
                // 则(01)找出tree的左子树中的最大节点
                //   (02)将该最大节点的值赋值给tree。
                //   (03)删除该最大节点。
                // 这类似于用"tree的左子树中最大节点"做"tree"的替身；
                // 采用这种方式的好处是：删除"tree的左子树中最大节点"之后，AVL树仍然是平衡的。
                Node *max = avltree_maximum(tree->left);
                tree->key = max->key;
                tree->left = delete_node(tree->left, max);
            }
            else
            {
                // 如果tree的左子树不比右子树高(即它们相等，或右子树比左子树高1)
                // 则(01)找出tree的右子树中的最小节点
                //   (02)将该最小节点的值赋值给tree。
                //   (03)删除该最小节点。
                // 这类似于用"tree的右子树中最小节点"做"tree"的替身；
                // 采用这种方式的好处是：删除"tree的右子树中最小节点"之后，AVL树仍然是平衡的。
                Node *min = avltree_maximum(tree->right);
                tree->key = min->key;
                tree->right = delete_node(tree->right, min);
            }
        }
        else
        {
            Node *tmp = tree;
            tree = tree->left ? tree->left : tree->right;
            free(tmp);
        }
    }

    return tree;
}
```
