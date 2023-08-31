给你一个链表，删除链表的倒数第 n 个结点，并且返回链表的头结点。

进阶：你能尝试使用一趟扫描实现吗？

示例 1：
![[删除链表的倒数第N个节点.png]]
输入：head = [1,2,3,4,5], n = 2 输出：[1,2,3,5] 示例 2：

输入：head = [1], n = 1 输出：[] 示例 3：

输入：head = [1,2], n = 1 输出：[1]
## 思路
双指针的经典应用，如果要删除倒数第 n 个节点，让 fast 移动 n 步，然后让 fast 和 slow 同时移动，直到 fast 指向链表末尾。删掉 slow 所指向的节点就可以了。

思路是这样的，但要注意一些细节。

分为如下几步：

- 首先这里我推荐大家使用虚拟头结点，这样方便处理删除实际头结点的逻辑，如果虚拟头结点不清楚，可以看这篇： [链表：听说用虚拟头节点会方便很多？(opens new window)](https://programmercarl.com/0203.%E7%A7%BB%E9%99%A4%E9%93%BE%E8%A1%A8%E5%85%83%E7%B4%A0.html)
    
- 定义 fast 指针和 slow 指针，初始值为虚拟头结点，如图：
![[删除链表的倒数第N个节点-1.png|675]]
- Fast 首先走 n + 1 步，为什么是 n+1 呢，因为只有这样同时移动的时候 slow 才能指向删除节点的上一个节点（方便做删除操作），如图：
![[删除链表的倒数第N个节点-2.png|675]]
- Fast 和 slow 同时移动，直到 fast 指向末尾，如题：
![[删除链表的倒数第N个节点-3.png|700]]
- 删除 slow 指向的下一个节点，如图：
![[删除链表的倒数第N个节点-4.png|700]]
此时不难写出如下 C++代码：
```
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode* dummyHead = new ListNode(0);
        dummyHead->next = head;
        ListNode* slow = dummyHead;
        ListNode* fast = dummyHead;
        while(n-- && fast != NULL) {
            fast = fast->next;
        }
        fast = fast->next; // fast再提前走一步，因为需要让slow指向删除节点的上一个节点
        while (fast != NULL) {
            fast = fast->next;
            slow = slow->next;
        }
        slow->next = slow->next->next; 
        
        // ListNode *tmp = slow->next;  C++释放内存的逻辑
        // slow->next = tmp->next;
        // delete nth;
        
        return dummyHead->next;
    }
};
```
- 时间复杂度: O(n)
- 空间复杂度: O(1)