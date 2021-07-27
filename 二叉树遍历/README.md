主要介绍JS实现二叉树的深度优先（前序，中序，后序）遍历和广度优先遍历算法，每种遍历法都有递归和非递归两种思路，也比较详细的介绍了 `leetCode` 上的锯齿形层序遍历算法，简单的写了下求二叉树的深度，求二叉树的宽度算法



## 写在前面



本文主要是介绍JS实现二叉树的深度优先（前序，中序，后序）遍历和广度优先遍历算法，每种遍历法都有递归和非递归两种思路，也比较详细的介绍了 `leetCode` 上的[锯齿形层序遍历](https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal/)算法。另外简单的写了下求二叉树的深度，求二叉树的宽度两种算法。



当然，很多算法的思路都是参考网上实现的，图片也是从网上找的，配上图片好理解。



补充一个知识，二叉树节点的定义：

```js
function TreeNode(val, left, right) {
    this.val = (val===undefined ? 0 : val)
    this.left = (left===undefined ? null : left)
    this.right = (right===undefined ? null : right)
}
```



在介绍之前，先简单说一下二叉树的几种遍历方式。



## 二叉树的遍历方式



1. 深度优先遍历(Depth First Search)：沿着树的深度遍历树的节点，尽可能深的搜索树的分支。

   又分为以下三种方式：

   - 前序遍历：访问根结点的操作发生在遍历其左右子树之前。

   - 中序遍历：访问根结点的操作发生在遍历其左右子树之间。

   - 后序遍历：访问根结点的操作发生在遍历其左右子树之后。

2. 广度优先遍历(Breadth First Search)：按照树的层次，每层从左至右依次遍历。

3. 锯齿形层序遍历：先从左往右，再从右往左进行下一层遍历，以此类推，层与层之间交替进行。



































![preorderTraversal1](.\images\preorderTraversal1.png)





![preorderTraversal2](.\images\preorderTraversal2.png)





![inorderTraversal1](.\images\inorderTraversal1.png)





![inorderTraversal2](.\images\inorderTraversal2.png)





![postorderTraversal1](.\images\postorderTraversal1.png)





![postorderTraversal2](.\images\postorderTraversal2.png)







1. [JS实现二叉树的前序、中序、后续、层序遍历](https://juejin.cn/post/6844904063650234375)
2. [JavaScript解：前序遍历二叉树](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/solution/javascriptjie-qian-xu-bian-li-er-cha-shu-by-user77/)
3. [JavaScript实现二叉树的遍历](https://www.jianshu.com/p/1e6f0228211e)