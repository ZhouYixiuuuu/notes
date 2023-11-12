# LeetCode刷题记录

## 链表

1. 记得判空链表/只有一个节点的链表

2. 快慢指针, 快指针一次走两步,慢指针一次走一步, 实现慢指针到达链表中点

   快慢指针还可以实现判断链表是否有环：如果有环，快慢指针最终会走到一个环上，如果两者之间相差n个结点，快指针每次走2步，慢指针每次走1步，两者之间的距离就缩小1，那么只要这样走n次，两者就会相遇。

3. 翻转链表

   ```java
   private ListNode reverse(ListNode head)
       {
           ListNode pre = null;
           while (head != null)
           {
               ListNode temp = head.next;  //存head的下一位
               head.next = pre;
               pre = head;  //pre和head并排向后挪一位
               head = temp;
           }
           return pre;
       }
   ```

   

## 字符串

1. 去除前后空格 `s.trim()`

## 树

1. 要注意空树啊!!

2. 二叉树的最大深度

   ```java
   public int maxDepth(TreeNode root) {
           if (root == null) return 0;
           return Math.max(maxDepth(root.left), maxDepth(root.right)) + 1;
       }
   ```

3. 验证二叉搜索树，可以使用中序遍历，中序遍历是一个严格递增的序列

   ```java
   TreeNode prev;
   public boolean isValidBST(TreeNode root) {
       if (root == null) return true;
       if (!isValidBST(root.left)) return false;  //判断左子树
       if (prev != null && prev.val >= root.val) return false;  //是否严格递增
       prev = root;
       if (!isValidBST(root.right)) return false;  //判断右子树
       return true;
   }
   ```

4. 对称二叉树

   也可以用两遍dfs来做, 一遍从左边开始dfs, 一遍从右边开始dfs

   ```java
   public boolean isSymmetric(TreeNode root) {
           if (root == null) return true;
           return dfs(root.left, root.right);
       }
   
   private boolean dfs(TreeNode left, TreeNode right)
   {
       if (left == null && right == null) return true;
   
       if (left == null || right == null) return false;
   
       if (left.val != right.val) return false;
   
       return dfs(left.left, right.right) && dfs(left.right, right.left);
   }
   ```

5. 定义二维数组 `List<List<Integer>> res = new ArrayList<>()`

## 数学

1. 如何将一个数字迅速变成string `num + ""`