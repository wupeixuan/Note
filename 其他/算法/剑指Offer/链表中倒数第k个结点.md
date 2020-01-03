## 题目描述
输入一个链表，输出该链表中倒数第k个结点。

## 思路
为了能够只遍历一次就能找到倒数第k个节点，可以定义两个指针：
1. 快指针从链表的头指针开始遍历向前走k-1，慢指针保持不动；
2. 从第k步开始，慢指针也开始从链表的头指针开始遍历；
3. 由于两个指针的距离保持在k-1，当快指针到达链表的尾结点时，慢指针正好是倒数第k个结点。

下图是找到该链表的倒数第3个结点的示意图。
![链表中倒数第k个结点](http://images.cnblogs.com/cnblogs_com/wupeixuan/1199352/o_207c1801-2335-4b1b-b65c-126a0ba966cb.png)

## 代码实现
```java
package LinkedList;

/**
 * 链表中倒数第k个结点
 * 输入一个链表，输出该链表中倒数第k个结点。
 */
public class Solution50 {

    /**
     * 设链表的长度为 N。设两个指针 fast 和 slow，先让 fast 移动 k-1 个节点，则还有 N - k 个节点可以移动。此时让 fast 和 slow 同时移动，可以知道当 fast 移动到链表结尾时，slow 移动到 N - k + 1 个节点处，该位置就是倒数第 k 个节点。
     *
     * @param head
     * @param k
     * @return
     */
    public ListNode FindKthToTail(ListNode head, int k) {
        if (head == null || k == 0) {
            return null;
        }
        ListNode fast = head;
        ListNode slow;
        for (int i = 0; i < k - 1; i++) {
            if (fast.next != null) {
                fast = fast.next;
            } else {
                return null;
            }
        }
        slow = head;
        while (fast.next != null) {
            fast = fast.next;
            slow = slow.next;
        }
        return slow;
    }

    public class ListNode {
        int val;
        ListNode next = null;

        ListNode(int val) {
            this.val = val;
        }
    }
}

```