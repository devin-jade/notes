---
layout: post
title: 二叉树的改进
categories: [数据结构]
tags: 树
---

## 线索二叉树

 通过考察各种二叉链表，不管儿叉树的形态如何，空链域的个数总是多过非空链域的个数。准确的说，n各结点的二叉链表共有2n个链域，非空链域为n-1个，但其中的空链域却有n+1个 ,这会造成浪费。

![](https://raw.githubusercontent.com/devin-jade/devin-imag/master/web/20191029174847.png)

因此，提出了一种方法，利用原来的空链域存放指针，指向树中其他结点。这种指针称为线索。加上线索的二叉链表成为线索链表，相应的二叉树就称为线索二叉树。

>记ptr指向二叉链表中的一个结点，以下是建立线索的规则：
>
>（1）如果ptr->lchild为空，则存放指向中序遍历序列中该结点的前驱结点。这个结点称为ptr的中序前驱；
>
>（2）如果ptr->rchild为空，则存放指向中序遍历序列中该结点的后继结点。这个结点称为ptr的中序后继；
>
>显然，在决定lchild是指向左孩子还是前驱，rchild是指向右孩子还是后继，需要一个区分标志的。因此，我们在每个结点再增设两个标志域ltag和rtag，注意ltag和rtag只是区分0或1数字的布尔型变量，其占用内存空间要小于像lchild和rchild的指针变量。结点结构如下所示。

![](https://raw.githubusercontent.com/devin-jade/devin-imag/master/web/20191029175545.png)

>其中：
>
>（1）ltag为0时指向该结点的左孩子，为1时指向该结点的前驱；
>
>（2）rtag为0时指向该结点的右孩子，为1时指向该结点的后继；
>
>（3）因此对于上图的二叉链表图可以修改为下图的养子。

![](https://raw.githubusercontent.com/devin-jade/devin-imag/master/web/20191029175622.png)

线索化的实质就是将二叉链表中的空指针指向改为前驱和后继的线索，由于前驱和后继只有在遍历二叉树的时候才能知道，因此**线索化的过程就是遍历的过程**，如果所用的二叉树需要经常遍历或者查找节点时需要某种比那里序列中的前驱和后继，就可以考虑线索二叉链表的存储结构了。

## 树转二叉树

如果一棵树不是二叉树，一个结点下右N个孩子，那么树的度和深度都很难知道，操作起来会缺少规律和便捷性，但是呢如果是二叉树那我们就爽了，就可以用上一篇的各种遍历方式进行遍历；

![](https://raw.githubusercontent.com/devin-jade/devin-imag/master/web/20191029215855.png)

转换为二叉树的步骤是：

1. 加线。就是在所有兄弟结点之间加一条连线；
2. 抹线。就是对树中的每个结点，只保留他与第一个孩子结点之间的连线，如果没有左孩子只有右孩子结点的话，那就保留右孩子的，删除结点与其它孩子结点之间的连线；
3. 旋转。就是以树的根结点为轴心，将整棵树顺时针旋转一定角度，使之结构层次分明。 

**核心点：每一个结点的第一个子节点是左节点，第一个子节点之外的所有子节点都是第一个子节点的右节点。**

```java
public static NormalTree createNormalTree() {
        NormalTree tree = new NormalTree('A');
        NormalTree node1 = new NormalTree('B');
        NormalTree node2 = new NormalTree('C');
        NormalTree node3 = new NormalTree('D');
        NormalTree node4 = new NormalTree('E');
        NormalTree node5 = new NormalTree('F');
        NormalTree node6 = new NormalTree('G');
        NormalTree node7 = new NormalTree('H');
        NormalTree node8 = new NormalTree('I');
        NormalTree node9 = new NormalTree('J');

        tree.setChildren(new NormalTree[]{node1, node2, node3});
        node1.setChildren(new NormalTree[]{node4, node5, node6});
        node2.setChildren(new NormalTree[]{node7});
        node3.setChildren(new NormalTree[]{node8, node9});
        return tree;
    }
public static TreeNode toBinaryTree(NormalTree normalTree) {
        TreeNode treeNode = new TreeNode(normalTree.getVal());
        TreeNode temp = treeNode;
        if (normalTree.getChildren() != null) {
            for (int i = 0; i < normalTree.getChildren().length; i++) {
                NormalTree norTemp = normalTree.getChildren()[i];
                TreeNode ttemp = toBinaryTree(norTemp);
                if (i == 0) {
                    temp.setLeftNode(ttemp);
                    temp = temp.getLeftNode();
                } else {
                    temp.setRightNode(ttemp);
                    temp = temp.getRightNode();
                }
            }
        }
        return treeNode;
    }
```

## 森林转二叉树

![](https://raw.githubusercontent.com/devin-jade/devin-imag/master/web/20191029221426.png)

思路和树转二叉树思路差不多

- 先将森林的每棵普通树转换为二叉树，不会的看上面。
- 然后将森林里，每棵二叉树的根看做兄弟，进行连线
- 每棵树的根，都是右子树，所以如果森林有三棵树
- 那就是第一棵树作为二叉树的根，第二棵树是根的右子树，第三颗树是第二棵树的右子树

## 二叉树转森林，树

![](https://raw.githubusercontent.com/devin-jade/devin-imag/master/web/20191029223410.png)

- 判断：判断根有没有右子树，有就是森林，没有就是普通树
- 连线：把右孩子们和双亲链接在一起。
- 去线：把所有结点到右孩子的连线去掉
- 调整：剩下的就是普通树或者森林

## 小结

普通树转换为二叉树

- **加线：**兄弟结点加线
- **去线：**每个结点都只保留与第一个孩子的连线
- **调整：**树本身的连线是左子树，我们划的线是右子树

森林转换为二叉树

- **二叉：**先把普通树都转换为二叉树
- **连根：**把所有二叉树的根连接起来
- **调整：**第一棵二叉树是根，其它根都是右孩子

二叉树转换为树、森林

- **判断：**判断根有没有右子树，有就是森林，没有就是普通树
- **连线：**把右孩子们和双亲链接在一起。
- **去线：**把所有结点到右孩子的连线去掉
- **调整：**剩下的就是普通树或者森林

## 参考

[大话数据结构](https://pan.baidu.com/s/1KCFykjTNpjHxpvv9Ngr0pQ)



