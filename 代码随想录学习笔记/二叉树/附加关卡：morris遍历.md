## morris 遍历介绍

> morris遍历是二叉树遍历算法的超强进阶算法，跟递归、非递归（栈实现）的空间复杂度，morris遍历可以将非递归遍历中的空间复杂度降为O(1)。从而实现时间复杂度为O(N)，而空间复杂度为O(1)的精妙算法。  
> Morris 遍历利用的是树的叶节点左右孩子为空（树的大量空闲指针），实现空间开销的极限缩减。

## morris 遍历的实现原则

**记作当前节点为cur。**
1. 如果cur无左孩子，cur向右移动（cur=cur.right）
2. 如果cur有左孩子，找到cur左子树上最右的节点，记为mostright

1. 如果mostright的right指针指向空，让其指向cur，cur向左移动（cur=cur.left）
2. 如果mostright的right指针指向cur，让其指向空，cur向右移动（cur=cur.right）

实现以上的原则，即实现了morris遍历。

## morris遍历的实质

> 建立一种机制，对**于没有左子树的节点只到达一次，对于有左子树的节点会到达两次**

## morris遍历的实例

> 一个树若按层遍历的结构为{1,2,3,4,5,6,7}，即该树为满二叉树，头结点值为 1，左右孩子为 2,3，叶节点为 4,5,6,7

**一开始图示：**

![](https://pic1.zhimg.com/80/v2-f0dd0d96eb60aaa987122a09129b192c_720w.webp)

我们按照morris遍历来遍历该树。

1）首先cur来到头结点1，按照morris原则的第二条第一点，它存在左孩子，cur左子树上最右的节点为5，它的right指针指向空，所以让其指向1，cur向左移动到2。

2）2有左孩子，且它左子树最右的节点4指向空，按照morris原则的第二条第一点,让4的right指针指向2，cur向左移动到4

3）4不存在左孩子，按照morris原则的第一条，cur向右移动，在第二步中，4的right指针已经指向了2，所以cur会回到2

4）重新回到2，有左孩子，它左子树最右的节点为4，但是在第二步中，4的right指针已经指向了2，不为空。所以按照morris原则的第二条第二点，2向右移动到5，同时4的right指针重新指向空

5）5不存在左孩子，按照morris原则的第一条，cur向右移动，在第一步中，5的right指针已经指向了1，所以cur会回到1

6）cur回到1，回到头结点，左子树遍历完成，1有左孩子，左子树上最右的节点为5，它的right指针指向1，按照morris原则的第二条第二点，1向右移动到3，同时5的right指针重新指回空

……

当到达最后一个节点7时，按照流程下来，此时7无左右孩子，遍历结束。

**ps：建议用一张纸一步一步按照上面的讲解来画，你会更加明白整个morris遍历的。**

## morris代码实现（前序、中序、后序遍历）

前序遍历：

```java
public static void morrisPre(Node head) {
    if(head == null){
        return;
    }
    Node cur = head;
    Node mostRight = null;
    while (cur != null){
        // cur表示当前节点，mostRight表示cur的左孩子的最右节点
        mostRight = cur.left;
        if(mostRight != null){
            // cur有左孩子，找到cur左子树最右节点
            while (mostRight.right !=null && mostRight.right != cur){
                mostRight = mostRight.right;
            }
            // mostRight的右孩子指向空，让其指向cur，cur向左移动
            if(mostRight.right == null){
                mostRight.right = cur;
                System.out.print(cur.value+" ");
                cur = cur.left;
                continue;
            }else {
                // mostRight的右孩子指向cur，让其指向空，cur向右移动
                mostRight.right = null;
            }
        }else {
            System.out.print(cur.value + " ");
        }
        cur = cur.right;
    }
    System.out.println();
}
```

中序遍历：

```java
public static void morrisIn(Node head) {
    if(head == null){
        return;
    }
    Node cur = head;
    Node mostRight = null;
    while (cur != null){
        mostRight = cur.left;
        if(mostRight != null){
            while (mostRight.right !=null && mostRight.right != cur){
                mostRight = mostRight.right;
            }
            if(mostRight.right == null){
                mostRight.right = cur;
                cur = cur.left;
                continue;
            }else {
                mostRight.right = null;
            }
        }
        System.out.print(cur.value+" ");
        cur = cur.right;
    }
    System.out.println();
}
```

后续遍历（复杂一点）

```java
public static void morrisPos(Node head) {
       if(head == null){
           return;
       }
       Node cur = head;
       Node mostRight = null;
       while (cur != null){
           mostRight = cur.left;
           if(mostRight != null){
               while (mostRight.right !=null && mostRight.right != cur){
                   mostRight = mostRight.right;
               }
               if(mostRight.right == null){
                   mostRight.right = cur;
                   cur = cur.left;
                   continue;
               }else {
                   mostRight.right = null;
                   printEdge(cur.left);
               }
           }
           cur = cur.right;
       }
       printEdge(head);
       System.out.println();
   }
   public static void printEdge(Node node){
       Node tail =reverseEdge(node);
       Node cur = tail;
       while (cur != null ){
           System.out.print(cur.value+" ");
           cur =cur.right;
       }
       reverseEdge(tail);
   }
   public static Node reverseEdge(Node node){
       Node pre = null;
       Node next = null;
       while (node != null){
           next = node.right;
           node.right = pre;
           pre = node;
           node = next;
       }
       return pre;
   }
```