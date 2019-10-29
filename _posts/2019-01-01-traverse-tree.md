---
layout: post
title: 二叉树遍历树
categories: [数据结构]
tags: 树

---

### 二叉树的遍历

二叉树的遍历是指：从根节点出发，按照某种次序一次访问二叉树中的所有节点，使得每个节点被访问一次且仅被访问一次。

4种遍历方式：先序遍历，中序遍历，后序遍历，层次遍历；其中先序，中序，后序都属于深度遍历（ **Depth First Search**  DFS），每一种实现都可以有递归（最无含金量的实现）和非递归的方式，采用栈和队列等来实现； 

<img src="https://raw.githubusercontent.com/devin-jade/devin-imag/master/web/20191025100149.png" alt="树A" style="zoom:80%;" />



#### 先序遍历

思路：根-左-右

1. 访问根节点；
2. 访问当前节点的左子[树](http://data.biancheng.net/view/23.html)；
3. 若当前节点无左子树，则访问当前节点的右子树；

 结果：1 2 4 5 7 8 3 6 

##### 递归

```java
public static List<Integer> peOrderTraversalByRecursion(TreeNode treeNode) {
        LinkedList<Integer> queue = new LinkedList<>();
        if (treeNode != null) {
            // 先根节点
            queue.add(treeNode.getVal());
            // 再访问左子树
            queue.addAll(peOrderTraversalByRecursion(treeNode.getLeftNode()));
            // 再访问右子树
            queue.addAll(peOrderTraversalByRecursion(treeNode.getRightNode()));
        }
        return queue;
    }
```



##### 迭代  

利用栈的先进后出的特性

```java
public static List<Integer> peOrderTraversalByIteration(TreeNode treeNode) {
        List<Integer> result = new ArrayList<>();
        Stack<TreeNode> stack = new Stack<>();
        stack.push(treeNode);
        while (!stack.isEmpty()) {
            TreeNode temp = stack.pop();
            result.add(temp.getVal());
            if (temp.getRightNode() != null) {
                // 右节点压栈
                stack.push(temp.getRightNode());
            }
            if (temp.getLeftNode() != null) {
                // 左节点压栈
                stack.push(temp.getLeftNode());
            }
        }
        return result;
    }
```



#### 中序遍历

思路：左-根-右

1. 访问当前节点的左子[树](http://data.biancheng.net/view/23.html)；
2. 访问根节点；
3. 访问当前节点的右子树；

结果： 4 2 7 5 8 1 3 6

##### 递归  

```java
public static List<Integer> inOrderTraversalByRecursion(TreeNode treeNode) {
        List<Integer> result = new ArrayList<>();
        if (treeNode != null) {
            result.addAll(inOrderTraversalByRecursion(treeNode.getLeftNode()));
            result.add(treeNode.getVal());
            result.addAll(inOrderTraversalByRecursion(treeNode.getRightNode()));
        }
        return result;
    }
```

##### 迭代

中序遍历最先遇到的根节点不是最先访问的，我们需要先访问左子树，再回到根节点，再访问右节点。**关键点在于，如何从左子树回到根节点** ， **当前树的根节点的左节点，是它的左子树的根节点** 

```java
public static List<Integer> inOrderTraversalByIteration(TreeNode treeNode) {
        List<Integer> result = new ArrayList<>();
        Stack<TreeNode> stack = new Stack<>();
        TreeNode temp = treeNode;
        while (temp != null || !stack.isEmpty()) {
            // 递归添加左节点，其实就是找到每一颗子树的最左节点
            while (temp != null) {
                stack.push(temp);
                temp = temp.getLeftNode();
            }
            // 对根节点的遍历，同时将指针指向了右子树，在下轮中遍历右子树
            temp = stack.pop();
            result.add(temp.getVal());
            temp = temp.getRightNode();
        }
        return result;
    }
```

##### Morris（莫里斯） 遍历法

中序遍历的复杂的原因是因为，我们最先遇到的节点不是最先访问的，但是前序遍历确是最先遇到的就是最先访问的，那么能不能把我们要遍历的树先转化为可以按照前序遍历的树呢？中序遍历的顺序是左-根-右，加入这棵树没有左结点呢？是不是就可以变成 根-右了。也就满足了最先遇到的就是最先访问的。这就是Morris遍历法做的事情。

怎么来消灭左子树呢？

左-根-右，所有的根节点的访问都在左子树之后，左子树的结束就是左子树最右节点访问完。因此Morris算法思路：

>Step 1: 将当前节点current初始化为根节点
>
>Step 2: While current不为空，
>
>若current没有左子节点
>
>    a. 将current添加到输出
>    
>    b. 进入右子树，亦即, current = current.right
>
>否则
>
>    a. 在current的左子树中，令current成为最右侧节点的右子节点
>    
>    b. 进入左子树，亦即，current = current.left
>

```java
 public static List<Integer> inOrderTraversalByIterationMorris(TreeNode treeNode) {
        List<Integer> result = new ArrayList<>();
        TreeNode temp = treeNode;
        TreeNode rightMost;
        while (temp != null) {
            if (temp.getLeftNode() == null) {
                // 无左子树，那么就是根-右
                result.add(temp.getVal());
                temp = temp.getRightNode();
            } else {
                rightMost = temp.getLeftNode();
                while (rightMost.getRightNode() != null) {
                    //  寻找左子树的最右节点
                    rightMost = rightMost.getRightNode();
                }
                // 当前节点作为左子树的最右节点的右孩子
                rightMost.setRightNode(temp);
                TreeNode oldRoot = temp;
                // 将左子树作为新的顶层节点
                temp = temp.getLeftNode();
                // 消除左子树，防止出现无限循环
                oldRoot.setLeftNode(null);
            }
        }
        return result;
    }
```



其实莫里斯遍历和中序的迭代遍历思想是一样的，就是**消灭左子树** 

```java
			// 递归添加左节点
            while (temp != null) {
                stack.push(temp);
                temp = temp.getLeftNode();
            }
```

这段代码一直在再找根节点为左节点的最小左子树，然后遍历顺序就是"根-右"，返回根节点后，由于左节点已经遍历过，所以剩下的还是“根-右”



#### 后序遍历

思路：左-右-根

1. 访问当前节点的左子树；
2. 访问当前节点右子树；
3. 访问根节点；

结果： 4 7 8 5 2 6 3 1

##### 递归

```java
public static List<Integer> postOrderTraversalByRecursion(TreeNode treeNode) {
        List<Integer> result = new ArrayList<>();
        if (treeNode != null) {            result.addAll(postOrderTraversalByRecursion(treeNode.getLeftNode()));      result.addAll(postOrderTraversalByRecursion(treeNode.getRightNode()));
            result.add(treeNode.getVal());
        }
        return result;
    }
```

##### 迭代

```java
public static List<Integer> postOrderTraversalByIteration(TreeNode treeNode) {
        List<Integer> result = new ArrayList<>();
        Stack<TreeNode> stack = new Stack<>();
        TreeNode temp = treeNode;
        TreeNode pre = null;
        while (temp != null || !stack.isEmpty()) {
            while (temp != null) {
                stack.push(temp);
                temp = temp.getLeftNode();
            }
            // 这里使用的是peek而不是pop，这是因为我们需要首先去访问右节点
            temp = stack.peek();
            // 判断是否存在右节点，或者右节点是否已经访问过了，如果右节点已经访问过了，则接下来的操作就和中序遍历的情况差不多了
            if (temp.getRightNode() == null || temp.getRightNode() == pre) {
                stack.pop();
                result.add(temp.getVal());
                /**
                 * 这两步的目的都是为了在下一轮遍历中不再访问自己，cur = null很好理解，因为我们必须在一轮结束后改变cur的值，以添加下一个节点，所以它和cur = cur.right一样，目的都是指向下一个待遍历的节点，
                 * 只是在这里，右节点已经访问过了，则以当前节点为根节点的整个子树都已经访问过了，接下来应该回退到当前节点的父节点，而当前节点的父节点已经在栈里了，
                 * 所以我们并没有新的节点要添加，直接将cur设为null即可
                 *
                 * pre = cur 的目的有点类似于将当前节点标记为已访问，它是和if条件中的cur.right == pre配合使用的
                 */
                pre = temp;
                temp = null;
            } else {
                temp = temp.getRightNode();
            }
        }
        return result;
    }
```

后续遍历是三种中最复杂的，因为**最先遇到的节点是最后遍历的， 回到根节点之后要先去遍历右节点**

##### 双栈法

导致遍历复杂的原因是**先访问的节点不是先遍历的**，左-右-根，根是最先访问的，但是却要最后输出，那么 **根-右-左** 倒序输出不就是后续的正确输出结果吗。先访问，后输出，符合栈的先进后出的规则，所以双栈法就是：

1. 栈1实现 根-右-左 遍历
2. 栈2 实现遍历的倒序输出，变成 左-右-根

```java
public static List<Integer> postOrderTraversalByIterationDoubleStack(TreeNode treeNode) {
        List<Integer> result = new ArrayList<>();
        Stack<TreeNode> stack1 = new Stack<>();
        Stack<Integer> stack2 = new Stack<>();
        TreeNode temp;
        stack1.push(treeNode);
        while (!stack1.isEmpty()) {
            temp = stack1.pop();
            stack2.push(temp.getVal());
            if (temp.getLeftNode() != null) {
                // 左节点压栈
                stack1.push(temp.getLeftNode());
            }
            if (temp.getRightNode() != null) {
                // 右节点压栈
                stack1.push(temp.getRightNode());
            }
        }
        while (!stack2.isEmpty()) {
            result.add(stack2.pop());
        }
        return result;
    }
```



##### 双端队列

使用多一个栈来仅仅实现倒序输出，显得是那么的浪费，那么的不优雅，所以我们使用双向链表来实现，每次将元素都加在头部，已实现倒序的排列。

```java
public static List<Integer> postOrderTraversalByIterationDoubleDeque(TreeNode treeNode) {
        LinkedList<Integer> result = new LinkedList<>();
        Stack<TreeNode> stack = new Stack<>();
        TreeNode temp;
        stack.push(treeNode);
        while (!stack.isEmpty()) {
            temp = stack.pop();
            result.addFirst(temp.getVal());
            if (temp.getLeftNode() != null) {
                // 左节点压栈
                stack.push(temp.getLeftNode());
            }
            if (temp.getRightNode() != null) {
                // 右节点压栈
                stack.push(temp.getRightNode());
            }
        }
        return result;
    }
```



#### 层次遍历(广度优先遍历  **Breadth First Search**  BFS )

思路：

1. 访问根节点；
2. 访问当前节点的兄弟节点；
3. 当前节点无兄弟节点，从左至右访问子节点；

结果：1 2 3 4 5 6 7 8 

包含从上到下和从下到上，一个是另一个的反序。核心思想是使用队列，先进先出，按照层次从左到右进入队列，然后正序输出。

##### 递归

关键点在于构建层次

```java
public static List<Integer> levelOrderTraversalByRecursion(TreeNode treeNode) {
        LinkedList<Integer> result = new LinkedList<>();
        List<List<Integer>> levels = new ArrayList<>();
        if (treeNode != null) {
            levelOrderTraversalByRecursionHelper(treeNode, 0, levels);
        }
        if (!CollectionUtils.isEmpty(levels)) {
            levels.forEach(result::addAll);
        }
        return result;
    }

    public static void levelOrderTraversalByRecursionHelper(TreeNode treeNode, int level, List<List<Integer>> levels) {
        // 构建当前层
        if (levels.size() == level) {
            levels.add(new ArrayList<>());
        }
        // 设置当前层的值，按照从左到右的顺序添加
        levels.get(level).add(treeNode.getVal());

        // 处理下一层
        if (treeNode.getLeftNode() != null) {
            levelOrderTraversalByRecursionHelper(treeNode.getLeftNode(), level + 1, levels);
        }
        if (treeNode.getRightNode() != null) {
            levelOrderTraversalByRecursionHelper(treeNode.getRightNode(), level + 1, levels);
        }
    }
```

##### 迭代

```java
public static List<Integer> levelOrderTraversalByIteration(TreeNode treeNode) {
        LinkedList<Integer> result = new LinkedList<>();
        Queue<TreeNode> queue = new LinkedList<>();
        queue.add(treeNode);
        while (!queue.isEmpty()) {
            for (int i = 0; i < queue.size(); i++) {
                // 逐层遍历
                TreeNode node = queue.remove();
                result.add(node.getVal());
                if (node.getLeftNode() != null) {
                    queue.add(node.getLeftNode());
                }
                if (node.getRightNode() != null) {
                    queue.add(node.getRightNode());
                }
            }
        }
        return result;
    }
```



##### 从下往上迭代

 [ 二叉树的层次遍历 II](https://leetcode-cn.com/problems/binary-tree-level-order-traversal-ii/)

```java
public static List<Integer> levelOrderTraversalByIterationReversal (TreeNode treeNode) {
        LinkedList<List<Integer>> temp = new LinkedList<>();
        LinkedList<Integer> result = new LinkedList<>();
        Queue<TreeNode> queue = new LinkedList<>();
        queue.add(treeNode);
        while (!queue.isEmpty()) {
            List<Integer> onelist = new ArrayList<>();
            int count = queue.size();
            for (int i = 0; i < count; i++) {
                // 逐层遍历
                TreeNode node = queue.remove();
                onelist.add(node.getVal());
                if (node.getLeftNode() != null) {
                    queue.add(node.getLeftNode());
                }
                if (node.getRightNode() != null) {
                    queue.add(node.getRightNode());
                }
            }
            temp.addFirst(onelist);
        }
        if (!CollectionUtils.isEmpty(temp)) {
            temp.forEach(result::addAll);
        }
        return result;
    }
```



#### 时间复杂度

首先对于时间复杂度，由于树的每一个节点我们都是要去遍历的，所以它是难以优化的，都是O(n)，对于Morris算法，这个复杂度的计算要稍微复杂一点，但是可以证明，它同样是O(n)。对于空间复杂度，对递归方法而言，最坏的空间复杂度是O(n)，平均空间复杂度是O(log(n))。对于普通的迭代法而言，由于我们使用到了栈，其时间复杂度和空间复杂度一致，都是O(n)，对于Morris算法，由于我们并没有使用到栈，只使用到临时变量，因此其空间复杂度是O(1)。



#### 小结

| 遍历方法 | 顺序                     |                            示意图                            |                             应用                             |
| :------: | :----------------------- | :----------------------------------------------------------: | :----------------------------------------------------------: |
|   前序   | **根 ➜ 左 ➜ 右**         | <img src="https://raw.githubusercontent.com/devin-jade/devin-imag/master/web/Pre-Order.png" alt="img" style="zoom: 15%;" /> |         想在节点上直接执行操作（或输出结果）使用先序         |
|   中序   | **左 ➜ 根 ➜ 右**         | <img src="https://raw.githubusercontent.com/devin-jade/devin-imag/master/web/In-Order.png" alt="img" style="zoom:15%;" /> | 在**二分搜索树**中，中序遍历的顺序<br />符合从小到大（或从大到小）顺序的 <br />要输出排序好的结果使用中序 |
|   后序   | **左 ➜ 右 ➜ 根**         | <img src="https://raw.githubusercontent.com/devin-jade/devin-imag/master/web/Post-Order.png" alt="img" style="zoom:15%;" /> | 后续遍历的特点是在执行操作时<br />肯定**已经遍历过该节点的左右子节点** <br/>适用于进行破坏性操作 比如删除所有节点，<br />比如判断树中是否存在相同子树 |
| 广度优先 | **层序，横向访问**       | <img src="https://raw.githubusercontent.com/devin-jade/devin-imag/master/web/Breadth-First.png" alt="img" style="zoom:15%;" /> |    当**树的高度非常高**（非常瘦） 使用广度优先剑节省空间     |
| 深度优先 | **纵向，探底到叶子节点** | <img src="https://raw.githubusercontent.com/devin-jade/devin-imag/master/web/Deep-First.png" alt="img" style="zoom:15%;" /> | 当**每个节点的子节点非常多**（非常胖），<br />使用深度优先遍历节省空间 <br />（访问顺序和入栈顺序相关，想当于先序遍历） |

#### 参考

[数据结构](http://data.biancheng.net/view/vip_265.html)

 [二叉树遍历算法总结](https://charlesliuyx.github.io/2018/10/22/[直观算法]树的基本操作/) 

 [二叉树的前序，中序，后序遍历方法总结](https://segmentfault.com/a/1190000016674584)

[Leetcode](https://leetcode.com/problems/binary-tree-inorder-traversal/solution/)

[大话数据结构](https://pan.baidu.com/s/1KCFykjTNpjHxpvv9Ngr0pQ)

