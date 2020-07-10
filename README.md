[toc]

# 20200524

## 1-Leetcode 1302--最深层次节点的和

2020/5/24 下午3:25

### 1.广度优先遍历：二叉树层次遍历，最后一层加和。

```python 
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None
class Solution(object):
    def deepestLeavesSum(self, root):
        """
        :type root: TreeNode
        :rtype: int
        """
        if root==None:
            return 0
        stack=[root]
        level_val=[]
        while(stack):
            l=len(stack)
            level_val.append([])
            for i in range(l):
                node=stack.pop(0)
                level_val[-1].append(node.val)
                if node.left:
                    stack.append(node.left)
                if node.right:
                    stack.append(node.right)
        return sum(level_val[-1])

```

### 2.深度优先遍历：对于每个节点，更新最大深度和最大

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None
class Solution(object):
    def __init__(self):
        self.max_deep=-1
        self.max_val=0
    def deepestLeavesSum(self, root):
        """
        :type root: TreeNode
        :rtype: int
        """
        def dfs(node,l):
            if node==None:
                return 
            if l>self.max_deep:
                self.max_deep=l      # 更新完就一样了
                self.max_val=node.val
            elif l==self.max_deep:
                self.max_val+=node.val
            dfs(node.left,l+1)
            dfs(node.right,l+1)
        dfs(root,0)
        return self.max_val

```



## 2-1339.分裂二叉树的最大乘积

2020/5/24 下午4:06

给你一棵二叉树，它的根为 root 。请你删除 1 条边，使二叉树分裂成两棵子树，且它们子树和的乘积尽可能大。由于答案可能会很大，请你将结果对 10^9 + 7 取模后再返回。

思路：怎么体现断开呢？给定一棵树，节点和为固定值，所以两棵子树节点的和差值最接近时乘积最大。

先计算总的和，再计算一棵子树的和，最靠近中点。两次dfs

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None
class Solution(object):
    def __init__(self):
        self.tree_sum=0
        self.best_sum=0 # 用于存最优的子树和
    def maxProduct(self, root):
        """
        :type root: TreeNode
        :rtype: int
        """
        def dfs_sum(node):
            if node==None:
                return 
            self.tree_sum+=node.val
            dfs_sum(node.left)
            dfs_sum(node.right)
        def dfs_best_sum(node):
            if node==None:
                return 0
            cur=dfs_best_sum(node.left)+dfs_best_sum(node.right)+node.val
            if abs(cur*2-self.tree_sum)<abs(self.best_sum*2-self.tree_sum):
                self.best_sum=cur
            return cur
        dfs_sum(root)
        dfs_best_sum(root)
        return self.best_sum*(self.tree_sum-self.best_sum)%(10**9+7)

```



## 3-Leetcode53-最大子序和

2020/5/24 下午9:20

给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

前缀和小于0，不可要设置该前缀和为0。

```python
class Solution(object):
    def maxSubArray(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        n=len(nums)
        dp=[0]*n # dp[i]表示到nums[0:i]最大和
        res=0
        dp[0]=max(nums[0],0)
        for i in range(1,n):
            # if nums[i]>=0:
            #     dp[i]=dp[i-1]+nums[i]
            # else:
            #     dp[i]=max(dp[i-1]+nums[i],0)
            dp[i]=max(dp[i-1]+nums[i],0)  # 如果小于0，说明前面的前缀可以不要。
            res=max(res,dp[i])
        print(dp)
        return res

```

# 20200525

## 1-leetcode1091 二进制数组中的最短路径

在一个 N × N 的方形网格中，每个单元格有两种状态：空（0）或者阻塞（1）。

一条从左上角到右下角、长度为 k 的畅通路径，由满足下述条件的单元格 C_1, C_2, ..., C_k 组成：

相邻单元格 C_i 和 C_{i+1} 在八个方向之一上连通（此时，C_i 和 C_{i+1} 不同且共享边或角）
C_1 位于 (0, 0)（即，值为 grid[0]\[0]）
C_k 位于 (N-1, N-1)（即，值为 grid[N-1]\[N-1]）
如果 C_i 位于 (r, c)，则 grid[r]\[c] 为空（即，grid[r]\[c] == 0）
返回这条从左上角到右下角的最短畅通路径的长度。如果不存在这样的路径，返回 -1 

思路：回溯的思路，对每个点都要判断做选择，更新路径，撤销选择,一个点走过了，要不要标记成不能走呢，如果在走回这个点，说明第一次到达和第二次到达之间的路径白走了，这条必定不是最短路。

### 1-1深度优先遍历，先判定有效，再往下迭代

```python
class Solution(object):
    def __init__(self):
        self.short_path=float("INF")
    def shortestPathBinaryMatrix(self, grid):
        """
        :type grid: List[List[int]]
        :rtype: int
        """
        m=len(grid)
        n=len(grid[0])
        def dfs(i,j,l):
            if i==m-1 and j==n-1:
                #print("gen",i,j,l)
                self.short_path=min(self.short_path,l)
                return
            # 进来即下标合理
            #print(i,j,l)
            grid[i][j]=1  # 设置现场
            if 0<=i-1:
                if grid[i-1][j]==0:
                    dfs(i-1,j,l+1)
                if 0<=j-1 and grid[i-1][j-1]==0:
                    dfs(i-1,j-1,l+1)
                if j+1<n and grid[i-1][j+1]==0:
                    dfs(i-1,j+1,l+1)
            if i+1<m :
                if grid[i+1][j]==0:
                    dfs(i+1,j,l+1)
                if 0<=j-1 and grid[i+1][j-1]==0:
                    dfs(i+1,j-1,l+1)
                if j+1<n and grid[i+1][j+1]==0:
                    dfs(i+1,j+1,l+1)
            if 0<=j-1 and grid[i][j-1]==0:
                dfs(i,j-1,l+1)
            if j+1<n and grid[i][j+1]==0:
                dfs(i,j+1,l+1)
            grid[i][j]=0   # 恢复现场
        if grid[0][0]==0:
            dfs(0,0,1)
        return self.short_path if self.short_path<=m*n else -1
        #return self.short_path

```

### 1-2深度优先遍历，往下迭代，再判断有效

```python
class Solution(object):
    def __init__(self):
        self.short_path=float("INF")
    def shortestPathBinaryMatrix(self, grid):
        """
        :type grid: List[List[int]]
        :rtype: int
        """
        m=len(grid)
        n=len(grid[0])
        def dfs(i,j,l):
            if i==m-1 and j==n-1 and grid[i][j]==0:
                print("gen",i,j,l)
                self.short_path=min(self.short_path,l)
                return
            
            if 0<=i<m and 0<=j<n and grid[i][j]==0:
                print(i,j,l)
                grid[i][j]=1  # 设置现场
                dfs(i-1,j-1,l+1)
                dfs(i-1,j,l+1)
                dfs(i-1,j+1,l+1)
                dfs(i,j-1,l+1)
                dfs(i,j+1,l+1)
                dfs(i+1,j-1,l+1)
                dfs(i+1,j,l+1)
                dfs(i+1,j+1,l+1)
                grid[i][j]=0   # 恢复现场

        dfs(0,0,1)
        return self.short_path if self.short_path<=m*n else -1
        #return self.short_path
```

### 1.3 广度优先遍历

逐层遍历，每一层加入有效的节点，当有一层的节点达到目标之后（第一次达到，最短路径），就可以返回结果。

```python
class Solution(object):
    def shortestPathBinaryMatrix(self, grid):
        """
        :type grid: List[List[int]]
        :rtype: int
        """

        res=1
        if grid[0][0]==1 or grid[-1][-1]:
            return -1
        m=len(grid)
        if m==1:
            return 1   # 默认是个方阵
        que=[(0,0)]
        while(que):
            l=len(que)
            for _ in range(l):
                i,j=que.pop(0)
                print(i,j,res)
                for (new_i,new_j) in [(i-1,j-1),(i-1,j),(i-1,j+1),(i,j-1),(i,j+1),(i+1,j-1),(i+1,j),(i+1,j+1)]:
                    if new_i==m-1 and new_j==m-1:
                        return res+1
                    if not (0<=new_i<m) or not (0<=new_j<m):
                        continue
                    if grid[new_i][new_j]==1:
                        continue
                    que.append((new_i,new_j))
                    grid[new_i][new_j]=1  # 已经观测的1要撤销才对
            res+=1
        return -1
  
```

## 2-Leetcode -面试题10.03 旋转有序数组找一个数字

搜索旋转数组。给定一个排序后的数组，包含n个整数，但这个数组已被旋转过很多次了，次数不详。请编写代码找出数组中的某个元素，假设数组元素原先是按升序排列的。若有多个相同元素，返回索引值最小的一个。

思路：二分法，丢弃没有用的那一半

都在左边那一半进行判断操作：

如果左边[l,mid]一半升序：if arr[l]<=target and target<=arr[mid]：左边：r=mid; 右边：l=mid+1

如果左边的一半无序：if arr[l]<=target or  target<=arr[mid]：左边：r=mid; 右边：l=mid+1. **注意判断条件**

如果[l,mid] 的数字都相同，先判断是不是等于目标，是直接返回，不是去除重复值在进行操作。

[5,5,5,1,2,3,4,5] 5 这个例子真是太反人类了。

```python
class Solution(object):
    def search(self, arr, target):
        """
        :type arr: List[int]
        :type target: int
        :rtype: int
        """
        r=len(arr)-1
        l=0
        while(l<r):
            mid=(l+r)//2
            print(l,mid,r)
            if arr[l]<arr[mid]: # 前一段有序 ,相等也是有序
                if arr[l]<=target and target<=arr[mid]:
                    r=mid
                else:
                    l=mid+1
            elif arr[l]>arr[mid]:  # 左边无序
                if arr[l]<=target or  target<=arr[mid]:   # 比左端点大，或者右端点小，都不可能在右边
                    r=mid
                else:
                    l=mid+1
            elif arr[l]==arr[mid]:
                if arr[l]==target:
                    r=l
                while(arr[l]!=target and l<r):
                    l+=1           # 清楚重复值

        return l if arr[l]==target else -1

```

## 3-Leetcode 9 回文数

判断一个整数是否是回文数。回文数是指正序（从左向右）和倒序（从右向左）读都是一样的整数。

### 思路1-转换成list+双指针

双指针技巧？其他回文怎么看起来都很难？

```python
class Solution(object):
    def isPalindrome(self, x):
        """
        :type x: int
        :rtype: bool
        """
        flag=1
        if x<0:
            flag=-1
            x=-x
        num_lis=[]
        while(x):
            num_lis.append(x%10)
            x//=10
        if flag==-1:
            num_lis.append(-1)
        l,r=0,len(num_lis)-1
        while(l<r):
            if num_lis[l]==num_lis[r]:
                l+=1
                r-=1
            else:
                return False
        return True
```

### 思路2-官方求解思路-反转数字的一半

临界情况：所有负数都不可能是回文，

反转数字的一半：当反转数字大于原始数字的剩余时，已经处理了一半了。

<img src="/Users/chenyingying/cyy/TyporaNote/2-Daily-Algorithm.assets/截屏2020-06-22下午8.01.56.png" alt="截屏2020-06-22下午8.01.56" style="zoom:33%;" />

```python
class Solution(object):
    def isPalindrome(self, x):
        """
        :type x: int
        :rtype: bool
        """
        if x<0 or (x%10==0 and x!=0):
            return False
        half_x=0
        while(half_x<x):
            # print(x,half_x,x%10,half_x*10+x%10)
            half_x=half_x*10+x%10
            x//=10
        # print(half_x,x)
        return half_x==x or half_x//10==x
```



## 回文相关题

### leetcode 234 .回文链表

请判断一个链表是否为回文链表。

1.翻转链表，正反链表比较

2.翻转链表的后半部分，两个指针分别遍历前后一半

### leetcode 647.回文子串-数量

给定一个字符串，你的任务是计算这个字符串中有多少个回文子串。

具有不同开始位置或结束位置的子串，即使是由相同的字符组成，也会被计为是不同的子串。

中心扩展法：长度为n的字符串可能的回文中心有2n-1个。从每个中心开始，向两边扩展字符，直至不是回文串为止（abc->a b c可能的中心：三个字母加两个间隔）

```python 
class Solution(object):
    def countSubstrings(self, s):
        """
        :type s: str
        :rtype: int
        """
        n=len(s)
        m=n*2-1
        res=0
        # 每次以单个字符为中心时，left=right=i//2
        for i in range(m):
            left=i/2  # i=0=>left=0,right=0; i=1=>left=0,right=1; i=2
            right=left+i%2
            while(left>=0 and right<n and s[left]==s[right]):
                res+=1
                left-=1
                right+=1
        return res
```

### leetcode 5 .最长回文子串-子串是什么

对于一个子串，如果它是回文串(长度大于2)，那么将首尾两个字母 去除之后，仍然是一个回文串.用动态规划来解题dp[i,j]表示s[i,j]是否是字符串。
$$
dp(i,j)=dp(i+1,j-1)\ \ and \ \ S_{i+1}==S_{j-1}
$$
边界情况是，只有一个字符是为回文串即P(i,i)=True，两个字符且相等时为回文串即P(i,i+1)=True

```python
class Solution(object):
    def longestPalindrome(self, s):
        """
        :type s: str
        :rtype: str
        """
        n=len(s)
        if n<2:  # 空字符和单个字符直接输出
            return s
        dp=[[False]*n for _ in range(n)]
        for i in range(n):
            dp[i][i]=True
        res=1
        res_s=s[0]    # 两个字符的的时候，不相等时结果为单个字符
        for i in range(n-2,-1,-1):
            for j in range(i+1,n):
                
                if j==i+1:					# 次对角线上的元素要单独判断。
                    if s[i]==s[j]:
                        #print("a")
                        dp[i][j]=True
                        if res<j-i+1:
                            res=j-i+1
                            res_s=s[i:j+1]
                elif (dp[i+1][j-1] and s[i]==s[j]):
                    dp[i][j]=True
                    if res<j-i+1:
                        res=j-i+1
                        res_s=s[i:j+1]
        return res_s
        
```

### leetcode 516.最长回文序列

给定一个字符串`s`，找到其中最长的回文子序列。可以假设`s`的最大长度为`1000`。

回文序列可是不连续的。

从单个字符构成的回文子序列开始，不断增加子序列的长度，直至遍历所有的子序列。

dp[i]\[j]表示s[i:j+1]中最长回文子序列的长度

$$s[i]==s[j]=>dp[i][j]=dp[i+1][j-1]+2$$

$$s[i]!=s[j]=>dp[i][j]=max(dp[i+1][j],dp[i][j+1])$$ 左右少一个字符

```python
class Solution(object):
    def longestPalindromeSubseq(self, s):
        """
        :type s: str
        :rtype: int
        """
        n=len(s)
        dp=[[0 for j in range(n)] for i in range(n)]
        for i in range(n):
            dp[i][i]=1
        for i in range(n-2,-1,-1):
            for j in range(i+1,n):
                if s[i]==s[j]:
                    dp[i][j]=dp[i+1][j-1]+2
                else:
                    dp[i][j]=max(dp[i+1][j],dp[i][j-1])
        return dp[0][-1]
```

用于回溯求解最长回文串是什么：

```python
        i,j=0,n-1
        res_lis=[" "]*dp[0][n-1]
        k=0
        while(i<=j):
            if dp[i][j]>dp[i+1][j]:
                if dp[i][j]>dp[i][j-1]:
                    res_lis[k]=s[i]   # s[i]==s[j] s[i]是回文串中的一部分
                    res_lis[dp[0][n-1]-1-k]=s[i]
                    k+=1
                else:
                    i-=1                      
                j-=1
            i+=1
        print(res_lis)
```

# 20200526

## 1-leetcode 235.二叉搜索树的最近公共祖先

思路：一个节点为最近祖先：pq 在该节点的两棵子树/p=root 或者q=root。

首先判断 p 和 q 是否相等，若相等，则直接返回 p 或 q 中的任意一个，程序结束
若不相等，则判断 p 和 q 在向左还是向右的问题上，是否达成了一致
如果 p 和 q 都小于root, 哥俩一致认为向左👈，则 root = root.left
如果 p 和 q 都大于root, 哥俩一致认为向右👉，则 root = root.right
如果 p 和 q 哥俩对下一步的路线出现了分歧，说明 p 和 q 在当前的节点上就要分道扬镳了，当前的 root 是哥俩临别前一起走的最后一站
返回当前 root

### 1.递归

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def lowestCommonAncestor(self, root, p, q):
        """
        :type root: TreeNode
        :type p: TreeNode
        :type q: TreeNode
        :rtype: TreeNode
        """
        def dfs(node,p,q):
            if node.val>p.val and node.val>q.val:
                return dfs(node.left,p,q)      # 一定要将结果返回去
            if node.val<p.val and node.val<q.val:
                return dfs(node.right,p,q)
            return node
        return dfs(root,p,q)
            
```

迭代：

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def lowestCommonAncestor(self, root, p, q):
        """
        :type root: TreeNode
        :type p: TreeNode
        :type q: TreeNode
        :rtype: TreeNode
        """
        if root==None:
            return root
        node=root
        while(node):
            if node.val>p.val and node.val>q.val:
                node=node.left
            elif node.val<p.val and node.val<q.val:
                node=node.right
            else:
                return node
        return None
            
```



## 2-删除链表节点

### leetcode 237.删除链表中的节点

请编写一个函数，使其可以删除某个链表中给定的（非末尾）节点，你将只被给定要求被删除的节点。**给定**只有待删除的节点。已知：要删除的节点存在下一个节点

本题要求直接删除一个节点，因为找不到前驱节点，所以应该将这个节点转变为下一个节点：

（1）直接用curr_node=curr_node.next可以么？**不可以**

（2）复制了后续一个节点之后将后续节点删除

curr_node.val=curr_node.next.val
curr_node.next=curr_node.next.next

```python
class Solution(object):
    def deleteNode(self, node):
        """
        :type node: ListNode
        :rtype: void Do not return anything, modify node in-place instead.
        """
        node.val=node.next.val
        node.next=node.next.next
```

### leetcode 面试题18.删除链表中的节点

给定单向链表的头指针和一个要删除的节点的值，定义一个函数删除该节点。返回删除后的链表的头节点。

思路：删除节点记得引入哑巴节点

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None
class Solution(object):
    def deleteNode(self, head, val):
        """
        :type head: ListNode
        :type val: int
        :rtype: ListNode
        """
        dummy=ListNode(0)
        dummy.next=head
        pre_node=dummy
        curr_node=dummy.next
        while(curr_node):
            next_node=curr_node.next
            if curr_node.val==val:
                pre_node.next=next_node
                return dummy.next
            pre_node=curr_node
            curr_node=next_node
        return dummy.next
```

### leetcode 19.删除链表的倒数第n个节点

给定一个链表，删除链表的倒数第 *n* 个节点，并且返回链表的头结点。

思路：双指针技用于计数倒数第n个节点，并且引入dummy节点。

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def removeNthFromEnd(self, head, n):
        """
        :type head: ListNode
        :type n: int
        :rtype: ListNode
        """
        dummy=ListNode(0)
        dummy.next=head
        p1=dummy.next     
        for i in range(n):
            p1=p1.next
        p2=dummy.next
        pre_node=dummy        # 记录前一个节点，p1,p2从head节点开始，到none
                              # 不记录前一个节点，p1,p2从dummy 节点开始，到最后一个节点，跳出循环时，p2指向要删除节点的前一个节点
        while(p1):
            p1=p1.next
            pre_node=p2
            p2=p2.next
        pre_node.next=p2.next
        return dummy.next
            
```

## 3-leetcode Nim游戏

你和你的朋友，两个人一起玩 Nim 游戏：桌子上有一堆石头，每次你们轮流拿掉 1 - 3 块石头。 拿掉最后一块石头的人就是获胜者。你作为先手。

你们是聪明人，每一步都是最优解。 编写一个函数，来判断你是否可以在给定石头数量的情况下赢得游戏如果石头堆里只有1块，2块，或者3块，轮到你时，你可以把石头全部拿走，从而在游戏中取胜。你要做的就是 ，控制自己拿石头的块数，使留给对手的石头刚好使4块。

当石头市5，6，7 时，你可以控制自己拿1，2，3，使对手时4块；但是为8时，对手总可以让你遇到4块石头的时候。

依次类推，石头块数是4的整数倍时，你注定会输掉比赛。

```python
class Solution(object):
    def canWinNim(self, n):
        """
        :type n: int
        :rtype: bool
        """
        return n%4!=0
```



# 20200527

## 1-leetcode 231.2的幂

给定一个整数，编写一个函数来判断它是否是 2 的幂次方。

思路1：一个数取余2不为0，肯定不是

​			一个数取余2为0，那么取商再进行相同的操作，直至减为1.

```python
class Solution(object):
    def isPowerOfTwo(self, n):
        """
        :type n: int
        :rtype: bool
        """
        if n<1:
            return False
        while(n>1):
            if n%2!=0:
                return False
            else:
                n//=2
        return True
```

时间复杂度：o(logn)

二进制数只有一个位置为1，所以用布莱恩-尼克根算法可以判断是否只有一个1.

```
class Solution(object):
    def isPowerOfTwo(self, n):
        """
        :type n: int
        :rtype: bool
        """
        if n==0:
            return False
        if (n-1)&n==0:
            return True
        else:
            return False
```

## 2-leetcode 89.格雷编码

格雷编码是一个二进制数字系统，在该系统中，两个连续的数值仅有一个位数的差异。

给定一个代表编码总位数的非负整数 n，打印其格雷编码序列。即使有多个不同答案，你也只需要返回其中一种。

格雷编码序列必须以 0 开头。

n阶格雷码G(n)推G(n+1)阶：

前一半元素复制G(n)所有元素，并在每个元素前面加0，

后一半元素为G(n)的倒置后在每个元素前加1。

```python
class Solution(object):
    def grayCode(self, n):
        """
        :type n: int
        :rtype: List[int]
        """
        res,head=[0],1
        for i in range(n):
            for j in range(len(res)-1,-1,-1):   # 只需要执行后续的处理
                res.append(res[j]+head)
            head<<=1
        return res
```

## 3-leetcode 43-字符串相乘

给定两个以字符串形式表示的非负整数 `num1` 和 `num2`，返回 `num1` 和 `num2` 的乘积，它们的乘积也表示为字符串形式。

```python
class Solution(object):
    def multiply(self, num1, num2):
        """
        :type num1: str
        :type num2: str
        :rtype: str
        """
        res=str(int(num1)*int(num2))
        return res
```

手动实现这两个函数，直接将字符串转换成数字相乘后再转回字符串。

```python
class Solution(object):
    def multiply(self, num1, num2):
        """
        :type num1: str
        :type num2: str
        :rtype: str
        """
        table={"0":0,"1":1,"2":2,"3":3,"4":4,"5":5,"6":6,"7":7,"8":8,"9":9}
        table1={0:"0",1:"1",2:"2",3:"3",4:"4",5:"5",6:"6",7:"7",8:"8",9:"9"}
        num1_n,e=0,1
        for i in range(len(num1)-1,-1,-1):
            num1_n+=table[num1[i]]*e
            e*=10
        #print(num1_n)
        num2_n,e=0,1
        for i in range(len(num2)-1,-1,-1):
            num2_n+=table[num2[i]]*e
            e*=10
        res_n=num1_n*num2_n
        # print(num1_n,num2_n,res_n)
        if res_n==0:
            return "0"
        res_lis=[]
        while(res_n):
            res_lis.append(table1[res_n%10])
            res_n//=10
        res=""
        for i in range(len(res_lis)-1,-1,-1):
            res+=res_lis[i]
        return res
        
```

**不能使用任何标准库的大数类型（比如 BigInteger）**或**直接将输入转换为整数来处理**。

按照笔算乘法的方式进行计算，将num2的每一位乘以num1,然后每次计算的成果相加。我觉得肯定不会直接考这个题目。

# 20200528 

## 1-leetcode 112.路径总和

给定一个二叉树和一个目标和，判断该树中是否存在根节点到叶子节点的路径，这条路径上所有节点值相加等于目标和。

要点：只要判断是否**存在一条**就行了，用自顶向下的递归，每个节点附带由根节点到其父亲节点路径和。

对每个节点是需要判断是否是叶子节点：

--如果是，判断是否是给定的目标值，

--如果不是，往下递归左右子树。

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def hasPathSum(self, root, sum):
        """
        :type root: TreeNode
        :type sum: int
        :rtype: bool
        """
        # 存在一条就行了
        def dfs_top_down(node,path_val):
            if node==None:    # 上一层的叶子节点都没有使path_val满足条件，所以输出False
                return False
            path_val+=node.val
            if node.left==None and node.right==None:
                if path_val==sum:
                    return True
                else:
                    return False
            if dfs_top_down(node.left,path_val):
                return True
            if dfs_top_down(node.right,path_val):
                return True
            return False
        return dfs_top_down(root,0)

```



## 2-leetcode 113.路径总和II

给定一个二叉树和一个目标和，找到所有从根节点到叶子节点路径总和等于给定目标和的路径。

要点：找到所有的路径

思路：和第一题一样，使用自顶向下的递归，不过每个节点附带的信息多加了路径信息，用于向结果添加符合条件的路径。

注意点1：list信息的深度拷贝，不然就会得到空路径。copy.copy()也能实现其功能

注意点2：处理当前节点：更新当前节点的路径和，路径信息（添加到路径中），

​			     递归左右子树之后需要将路径中本节点的信息删除，使得返回上一层节点是，下一层节点的信息不会对其造成影响。

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None
class Solution(object):
    def __init__(self):
        self.res=[]
    def pathSum(self, root, sum):
        """
        :type root: TreeNode
        :type sum: int
        :rtype: List[List[int]]
        """
        # 到叶子节点只需要递归往下，由顶向下的递归
        def dfs_top_down(node,path_val,path):
            if node==None:
                return 
            path_val+=node.val 
            path.append(node.val)         # 增加路径节点
            print(path)
            if node.left==None and node.right==None:
                if path_val==sum:
                    self.res.append(path[:])
            dfs_top_down(node.left,path_val,path)
            dfs_top_down(node.right,path_val,path)
            path.pop()                    # 删除路径节点
        dfs_top_down(root,0,[])
        return self.res
```

## 3-leetcode 437.路径总和III

给定一个二叉树，它的每个结点都存放着一个整数值。找出路径和等于给定数值的路径总数。路径不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是向下的（只能从父节点到子节点）。

要点：现在不求完整的路径，而是求有多少条路径满足要求

思路：前缀和，从根节点到本节点的前缀和-目标，如果来的路径上存在这个前缀和，那么该节点到本节点的路径满足条件。

注意点1：递归处理完某一节点的左右子树时，本节点形成的前缀和需要删除。

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def __init__(self):
        self.res=0
    def pathSum(self, root, sum):
        """
        :type root: TreeNode
        :type sum: int
        :rtype: int
        """
        # 前缀和变成树的形式，s_i中存储的是根节点到该节点的和，和数组前缀和的区别就是si累加是带树遍历的
        # 深度优先遍历好不好
        has_map={0:1}
        def dfs(node,pre_sum):
            if node==None:
                return 
            pre_sum+=node.val    # 前缀和需要撤销，避免影响其他递归。
            if has_map.get(pre_sum-sum):
                self.res+=has_map.get(pre_sum-sum)
            if has_map.get(pre_sum):
                has_map[pre_sum]+=1
            else:
                has_map[pre_sum]=1
            dfs(node.left,pre_sum)
            dfs(node.right,pre_sum)
            has_map[pre_sum]-=1
        dfs(root,0)
        return self.res
```



# 20200531

## 1-leetcode 700.二叉搜索树中的搜索

### 递归

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def searchBST(self, root, val):
        """
        :type root: TreeNode
        :type val: int
        :rtype: TreeNode
        """
        def dfs(node,val):
            if node==None:
                return None
            if node.val==val:
                return node
            if node.val>val:
                return dfs(node.left,val) 
            else:
                return dfs(node.right,val)   # 有什么办法提前终止
        return dfs(root,val)
```

### 迭代

```python
class Solution(object):
    def searchBST(self, root, val):
        """
        :type root: TreeNode
        :type val: int
        :rtype: TreeNode
        """
        node=root
        while(node):
            if node.val==val:
                return node
            if node.val>val:
                node=node.left
            else:
                node=node.right
        return None
```

迭代和递归的时间复杂度是一致的，都是o(H),H为树的高度，平均情况下$H=\log n$,最糟糕的情况下为$H=N$

空间复杂度不同，递归需要保留每层递归的内容，空间复杂度较高，而迭代的时间复杂度为o(1).

递归调用函数一次，需要在内存栈中分配空间保存参数、返回地址以及临时变量，如栈和出栈操作都是需要时间的。递归层数过多时，容易造成内存溢出。



# 20200601

## 1-leetcode 面试题01.01 判断字符是否唯一

逐个访问字符串中的字符，将访问过的字符存在hash表中，处理每个字符的时候去找，这个字符出现过了么，出现过了，直接输出false

```python
class Solution(object):
    def isUnique(self, astr):
        """
        :type astr: str
        :rtype: bool
        """
        hash_map={}
        for char in astr:
            if hash_map.get(char):
                return False
            hash_map[char]=1
        return True
```

## 2-leetcode 面试题01.02 判断是否为字符串重排

比较两个字符串组成字母是否一致，如果一致则可以通过重排得到

```python
class Solution(object):
    def CheckPermutation(self, s1, s2):
        """
        :type s1: str
        :type s2: str
        :rtype: bool
        """
        has_s1={}
        for char in s1:
            if has_s1.get(char):
                has_s1[char]+=1
            else:
                has_s1[char]=1
        for char in s2:
            if has_s1.get(char):
                has_s1[char]-=1
                if has_s1[char]==0:
                    has_s1.pop(char)
            else:
                return False
        if has_s1:
            return False
        else:
            return True
```

# 20200602

## 1.堆排序

**二叉树基础**

**满二叉树：**每一层节点数都达到最大值

**完全二叉树**：若二叉的深度为K，除了最后一层，其余各层的节点数都达到最大值，最后一层节点都连续集中在最左边。

**平衡二叉树：**前提是一棵二叉搜索树-左子树和右子树深度的差绝对值不超过1，左子树和右子树都是一棵平衡二叉树。

**最优二叉树：**（哈夫曼树）-带权路径长度达到最小。？？？



二叉树的顺序存储--将二叉树存在列表中：

父亲节点与左孩子节点在列表中index 的关系：i->2i+1

父亲节点与右孩子节点在列表中index 的关系：i->2i+2

<img src="/Users/chenyingying/cyy/TyporaNote/2-Daily-Algorithm.assets/duipaixu.png" alt="duipaixu" style="zoom:33%;" />

大顶堆：一棵完全二叉树，满足任一节点都比其孩子节点大

小顶堆：一棵完全二叉树，满足任一节点都比其孩子节点小

<img src="/Users/chenyingying/cyy/TyporaNote/2-Daily-Algorithm.assets/duipaixu1.png" alt="duipaixu1" style="zoom: 33%;" />

堆排序向下调整--根节点的左右子树都是堆，但是根节点不满足堆的性质，通过一次向下调整，将其变成一个堆



堆排序过程：

1.建立堆

2.得到堆顶元素(最大元素)

3.去掉堆顶，**将堆最后一个元素放到堆顶，通过一次调整重新调整堆变得有序**

4.堆顶元素为第二大元素

5.重复步骤3，直至堆变空

难点：如何建立堆，如何进行向下调整

```python
class Solution(object):
    def sortArray(self, nums):
        """
        :type nums: List[int]
        :rtype: List[int]
        """
        n=len(nums)
        for i in range((n-2)//2,-1,-1):
            self.shift(nums,i,n-1)

        for i in range(n-1,-1,-1):
            nums[0],nums[i]=nums[i],nums[0]     
            self.shift(nums,0,i-1)
        return nums

    def shift(self,nums,low,high):
        tmp=nums[low]
        i=low
        j=2*i+1
        while(j<=high):
            if j+1<=high and nums[j]<nums[j+1]:
                j=j+1
            if nums[j]>tmp:											# 很重要，是和最原始的根比较。
                nums[i]=nums[j]
                i=j
                j=2*i+1
            else:
                nums[i]=tmp
                break
        nums[i]=tmp         # j>high的情况下跳出循环，保证最小值下沉到叶子节点时有有把值保存下来

```

时间复杂度：O(n\log n)O(nlogn)。初始化建堆的时间复杂度为 O(n)，建完堆以后需要进行 n-1次调整，一次调整（即 maxHeapify） 的时间复杂度为 O(\log n)，那么 n-1次调整即需要 O(n\log n) 的时间复杂度。因此，总时间复杂度为 O(n+n\log n)=O(n\log n)。



# 20200603

## 1.leetcode 300.最长上升子序列

给定一个无序的整数数组，找到其中最长上升子序列的长度。

子序列不连续，元素的相对位置不变。

dp[i]：以第i个元素为结尾的最长上升子序列的长度。对第i个元素，dp[i]的更新，

是要遍历所有dp[:i]，看看以nums[j]为结尾能够构成的最长上升子序列的长度是多少，判断nums[i]能不能被添加到对应的序列中，在所有的dp[0]-dp[i-1]种情况中寻找一个最大情况，更新dp[i].

最后输出max(dp)

```python
class Solution(object):
    def lengthOfLIS(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        # dp[i]表示以nums[i]为结尾的最长上升子序列的长度
        n=len(nums)
        if n==0:
            return 0
        dp=[1]*n
        for i in range(1,n):
            for j in range(i):
                if nums[i]>nums[j]:
                    dp[i]=max(dp[i],dp[j]+1)
        return max(dp)
```



# 20200604

## 1-Leetcode 209.长度最小的子数组

给定一个含有 n 个正整数的数组和一个正整数 s ，找出该数组中满足其和 ≥ s 的长度最小的连续子数组，并返回其长度。如果不存在符合条件的**连续子数组**，返回 0。

快慢指针：快指针不断向后走，累加走过的元素，当 当前和大于target 之后，移动左端点-慢指针，直至不满足大于等于的条件。当快指针移动至数组结尾时停止处理。

```python
class Solution(object):
    def minSubArrayLen(self, s, nums):
        """
        :type s: int
        :type nums: List[int]
        :rtype: int
        """
        left=0
        n=len(nums)
        res=n+1
        tmp_sum=0
        for right in range(n):
            tmp_sum+=nums[right]
            while(tmp_sum>=s):
                res=min(res,right-left+1)
                tmp_sum-=nums[left]
                left+=1
        if res==n+1:
            return 0
        return res

```

## 前缀和

### 2-leetcode 560.和为k的子数组

给定一个整数数组和一个整数 **k，**你需要找到该数组中和为 **k** 的连续的子数组的个数。

思路：

前缀和，$s_i=\sum nums[0]+...+nums[i]$ ,如果前缀和中有出现$s_i-k$,就说明该前缀到i直接的子数组和为k，则符合条件的子数组数增加s。用hask表来记录某一个前缀和出现的次数，用于确认s

```python
class Solution(object):
    def subarraySum(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: int
        """
        has_map={0:1}
        si=0
        n=len(nums)
        res_n=0
        for i in range(n):
            si+=nums[i]
            if has_map.get(si-k):
                res_n+=has_map.get(si-k)
            if has_map.get(si):
                has_map[si]+=1
            else:
                has_map[si]=1
        return res_n
```

### 3.leetcode 437.路径总和3

给定一个二叉树，它的每个结点都存放着一个整数值。

找出路径和等于给定数值的路径总数。

路径不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是向下的（只能从父节点到子节点）。

二叉树不超过1000个节点，且节点数值范围是 [-1000000,1000000] 的整数。

**注意点：**递归处理完一个节点之后，需要撤销该点的前缀和，使得递归返回上一层时，下一层的前缀和不会上一层的运算。

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def __init__(self):
        self.res=0
    def pathSum(self, root, sum):
        """
        :type root: TreeNode
        :type sum: int
        :rtype: int
        """
        # 前缀和变成树的形式，s_i中存储的是根节点到该节点的和，和数组前缀和的区别就是si累加是带树遍历的
        # 深度优先遍历好不好
        has_map={0:1}
        def dfs(node,pre_sum):
            if node==None:
                return 
            pre_sum+=node.val    # 前缀和需要撤销，避免影响其他递归。
            if has_map.get(pre_sum-sum):
                self.res+=has_map.get(pre_sum-sum)
            if has_map.get(pre_sum):
                has_map[pre_sum]+=1
            else:
                has_map[pre_sum]=1
            dfs(node.left,pre_sum)
            dfs(node.right,pre_sum)
            has_map[pre_sum]-=1
        dfs(root,0)
        return self.res

```

## 4-leetcode 862.和至少为K的最短子数组。

返回 `A` 的最短的非空连续子数组的**长度**，该子数组的和至少为 `K` 。如果没有和至少为 `K` 的非空子数组，返回 `-1` 。（难点在于数组有负数）

这题考查的知识点：前缀和+单调栈+二分查找

1.前缀和-子数组相关求解，一般解题思路

2.二分查找-前缀和presum已经获得，以j为起点，数组和为k的最短数组可以使用二分查找。

3.单调栈-由于存在负数，因此前缀和不再是单调递增序列

我觉得不会考

## 5-leetcode 270.最接近的二叉搜索树值

给定一个不为空的二叉搜索树和一个目标值 target，请在该二叉搜索树中找到最接近目标值 target 的数值。

注意：给定的目标值 target 是一个浮点数，题目保证在该二叉搜索树中只会存在一个最接近目标值的数

思路：二叉搜索树结构特点，左子树<根节点<右子树，决定了递归方向，要求绝对值最小，

针对每个节点更新结果--比较和最小值的关系，直接更新。

如果当前节点值小于目标值，结果只能是当前节点或者右子树中的值，如果当前节点的值大于目标值，结果只能在当前节点和左子树节点中。

```python
class Solution:
    def __init__(self):
        self.ab=float("INF")
        self.ans=0
    def closestValue_rec(self,root,target):

        def dfs(node):
           if node==None:
               return
           if abs(node.val - target) < self.ab:
               self.ab = abs(node.val - target)
               self.ans = node.val
           if node.val>target:
                dfs(node.left)
           elif node.val<target:  # 只能选一边，然后逐层返回
                dfs(node.right)
        dfs(root)
        return self.ans

    def closestValue_iter(self, root, target):
        min_abs=float("INF")
        res_val=0
        node=root   # 直接递归一半，把另外一半丢掉，不需要堆栈
        while(node):
            if node.val==target:
                return node.val
            if abs(node.val-target)<min_abs:
                min_abs=abs(node.val-target)
                res_val=node.val
            if node.val>target:
                node=node.left
            elif node.val<target:
                node=node.right
        return res_val

```

## 6-leetcode 72 .编辑距离

给你两个单词 word1 和 word2，请你计算出将 word1 转换成 word2 所使用的最少操作数 。

你可以对一个单词进行如下三种操作：

插入一个字符
删除一个字符
替换一个字符

```python
class Solution(object):
    def minDistance(self, word1, word2):
        """
        :type word1: str
        :type word2: str
        :rtype: int
        """
        l1,l2=len(word1),len(word2)
        if l1*l2==0:
            return max(l1,l2)
        dp=[]
        for i in range(l1+1):
            dp.append([0]*(l2+1))
        for i in range(l1+1):
            dp[i][0]=i
        for j in range(l2+1):
            dp[0][j]=j
        for i in range(1,l1+1):
            for j in range(1,l2+1):
                if word1[i-1]==word2[j-1]:
                    dp[i][j]=dp[i-1][j-1]
                else:
                    dp[i][j]=min(dp[i-1][j-1],dp[i-1][j],dp[i][j-1])+1
        return dp[-1][-1]
```

## 7-leetcode 583.两个字符串的删除操作

给定两个单词 *word1* 和 *word2*，找到使得 *word1* 和 *word2* 相同所需的最小步数，每步可以删除任意一个字符串中的一个字符。

思路：l1=len(word1),l2=len(word2),求两个字符串最大公共子串的长度lcon ，答案为：l1+l2-2*lcon

```python
	pyclass Solution(object):
    def minDistance(self, word1, word2):
        """
        :type word1: str
        :type word2: str
        :rtype: int
        """
        l1,l2=len(word1),len(word2)
        dp=[]
        for i in range(l1+1):
            dp.append([0]*(l2+1))
        for i in range(1,l1+1):
            for j in range(1,l2+1):
                if word1[i-1]==word2[j-1]:
                    dp[i][j]=dp[i-1][j-1]+1
                else:
                    dp[i][j]=max(dp[i-1][j],dp[i][j-1])
        lcon=dp[-1][-1]
        return l1+l2-2*lcon
```

## 7-2-leetcode 583.两个字符串删除操作II

具体要删除那些字符呢？

思路：用回溯法求出最长公共子串，然后将两个字符串除掉公共字符串即可

回溯法求公共子串具体是什么，只有对角线操作才是因为存在公共子串的操作，直线操作用于**定位**来时的路径。

```python
class Solution(object):
    def longestCommonSubsequence(self, text1, text2):
        """
        :type text1: str
        :type text2: str
        :rtype: int
        """
        l1,l2=len(text1),len(text2)  
        dp=[]
        for i in range(l1+1):
            dp.append([0]*(l2+1))
        for i in range(1,l1+1):
            for j in range(1,l2+1):
                if text1[i-1]==text2[j-1]:
                    dp[i][j]=dp[i-1][j-1]+1
                else:
                    dp[i][j]=max(dp[i-1][j],dp[i][j-1])
        res=[0]*dp[-1][-1]
        i,j=l1,l2
        while(i>0):
            if dp[i][j]>dp[i-1][j]:
                if dp[i][j]>dp[i][j-1]:
                    res[dp[i][j]-1]=text1[i-1]     # dp[i][j]进行了斜对角线操作，text1[i-1]为公共子串内容
                else:
                    i+=1                           # dp[i][j]从dp[i][j-1]来的，递归路径，应该是dp[i][j-1],由于i,作为循环控制变量，所以先要进行+1操作，之后减1，才会不变
                j-=1                               # 以上两种情况j都是要减-1的, 往上或者往斜上方走。
            i-=1                                   # dp[i][j]从左边来的。当dp[i][j]==dp[i-1][j]==dp[i][j-1]时，只需要选一条方向，输出一个可能的最长公共子串即可。
        print(res)
        return dp[-1][-1]
```

参考博文：https://www.cnblogs.com/Inkblots/p/5247070.html

## 8-leetcode 1143.最长公共子序列


给定两个字符串 `text1` 和 `text2`，返回这两个字符串的最长公共子序列的长度。

一个字符串的 *子序列* 是指这样一个新的字符串：它是由原字符串在**不改变字符的相对顺序**的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。
例如，"ace" 是 "abcde" 的子序列，但 "aec" 不是 "abcde" 的子序列。两个字符串的「公共子序列」是这两个字符串所共同拥有的子序列。

若这两个字符串没有公共子序列，则返回 0。

### 思路1:递归

rec(s1,s2,i,j)-返回s1[:i+1],s2[:j+1]最长公共子序列长度，依据s1[i]与s2[j]的情况分不同的情况往下递归

s1[i]==s2[j]. return rec(s1,s2,i-1,j-1)+1

s1[i]!=s2[j],return max(rec(s1,s2,i-1,j),rec(s1,s2,i,j-1))

时间超出限制。11/38

```python
class Solution(object):
    def longestCommonSubsequence(self, text1, text2):
        """
        :type text1: str
        :type text2: str
        :rtype: int
        """
        def rec(text1,text2,i,j):
            #print(i,j,text1[i],text2[j] )
            if i<0 or j<0:
                return 0
            if text1[i]==text2[j]:
                return rec(text1,text2,i-1,j-1)+1
            return max(rec(text1,text2,i-1,j),rec(text1,text2,i,j-1))
        return rec(text1,text2,len(text1)-1,len(text2)-1)
```

### 思路2:dp

dp[i]\[j]:s1[:i+1]与s2[:j+1]的最长公共子序列的长度

用dp[i-1]\[j-1]来更新dp[i]\[j]: 

如果s1[i]!=s2[j]:dp[i]\[j]=max(dp[i-1]\[j],dp[i]\[j-1])

如果s1[i]==s2[j]:dp[i]\[j]=dp[i-1]\[j-1]+1

初始化：多一个空字符串

```python
class Solution(object):
    def longestCommonSubsequence(self, text1, text2):
        """
        :type text1: str
        :type text2: str
        :rtype: int
        """
        l1,l2=len(text1),len(text2)  
        dp=[]
        for i in range(l1+1):
            dp.append([0]*(l2+1))
        for i in range(1,l1+1):
            for j in range(1,l2+1):
                if text1[i-1]==text2[j-1]:
                    dp[i][j]=dp[i-1][j-1]+1
                else:
                    dp[i][j]=max(dp[i-1][j],dp[i][j-1])
        # print(dp)
        return dp[-1][-1]
```

## 9-leetcode 700.二叉搜索树中的搜索

给定二叉搜索树（BST）的根节点和一个值。 你需要在BST中找到节点值等于给定值的节点。 返回以该节点为根的子树。 如果节点不存在，则返回 NULL。

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def searchBST(self, root, val):
        """
        :type root: TreeNode
        :type val: int
        :rtype: TreeNode
        """
        def dfs(node,val):
            if node==None:
                return None
            if node.val==val:
                return node
            if node.val>val:
                return dfs(node.left,val) 
            else:
                return dfs(node.right,val)   # 有什么办法提前终止
        return dfs(root,val)
```



# 20200607

## 1-leetcode4.寻找两个正序数组的中位数

```python
        """
        - 主要思路：要找到第 k (k>1) -非下标索引-小的元素，那么就取 pivot1 = nums1[k/2-1] 和 pivot2 = nums2[k/2-1] 进行比较
        - 这里的 "/" 表示整除
        - nums1 中小于等于 pivot1 的元素有 nums1[0 .. k/2-2] 共计 k/2-1 个
        - nums2 中小于等于 pivot2 的元素有 nums2[0 .. k/2-2] 共计 k/2-1 个
        - 取 pivot = min(pivot1, pivot2)，两个数组中小于等于 pivot 的元素共计不会超过 (k/2-1) + (k/2-1) <= k-2 个
        - 这样 pivot 本身最大也只能是第 k-1 小的元素--pivort 本身也可以丢弃。
        - 如果 pivot = pivot1，那么 nums1[0 .. k/2-1] 都不可能是第 k 小的元素。把这些元素全部 "删除"，剩下的作为新的 nums1 数组
        - 如果 pivot = pivot2，那么 nums2[0 .. k/2-1] 都不可能是第 k 小的元素。把这些元素全部 "删除"，剩下的作为新的 nums2 数组
        - 重中之重-由于我们 "删除" 了一些元素（这些元素都比第 k 小的元素要小），因此需要修改 k 的值，减去删除的数的个数
        """
```



```python 
class Solution(object):
    def findMedianSortedArrays(self, nums1, nums2):
        """
        :type nums1: List[int]
        :type nums2: List[int]
        :rtype: float
        """

        def getKthElement(k):
            index1, index2 = 0, 0
            while (True):
                if index1 == m:
                    return nums2[index2 + k - 1]  # index下标的累计量
                if index2 == n:
                    return nums1[index1 + k - 1]
                if k == 1:
                    return min(nums1[index1], nums2[index2])

                newindex1 = min(index1 + k // 2 - 1, m - 1)  # 下标索引 [0,...,k//2-2]一共k//2-1个元素个元素小于nums[newindex]]
                newindex2 = min(index2 + k // 2 - 1, n - 1)
                pivort1, pivort2 = nums1[newindex1], nums2[newindex2]
                if pivort1 <= pivort2:
                    k -= newindex1 - index1 + 1  # 删除掉的元素数量
                    index1 = newindex1 + 1  # 这个个加一操作不明白了
                else:
                    k -= newindex2 - index2 + 1   # 把pivort 本身也减掉
                    index2 = newindex2 + 1

        m, n = len(nums1), len(nums2)
        totalLenght = m + n
        if totalLenght % 2 == 1:
            return getKthElement((totalLenght + 1) // 2)
        else:
            return (getKthElement(totalLenght // 2) + getKthElement(totalLenght // 2 + 1)) / 2.0  														# python2中这个2.0一定要加零，不然就默认整除

```



## 2-leetcode10.正则匹配表达式

给你一个字符串 `s` 和一个字符规律 `p`，请你来实现一个支持 `'.'` 和 `'*'` 的正则表达式匹配。

```
'.' 匹配任意单个字符
'*' 匹配零个或多个前面的那一个元素
```

所谓匹配，是要涵盖 整个 字符串 s的，而不是部分字符串。

说明:

s 可能为空，且只包含从 a-z 的小写字母。
p 可能为空，且只包含从 a-z 的小写字母，以及字符 . 和 *。

思路：dp
递归到最底层：当规则p为空时，s为空，则匹配，s 不为空则不匹配。

其他时候按照分p[i]决定递归条件：
	p[i]\=="." 递归判断s[1:] p[1:]
	p[i]\=="*" 递归判断s[]   p[:]   前缀该如何取处理啊

没有星号，直接匹配

```python
    def isMatch(self, s, p):
        if not p:                  # 如果规则为空，则返回 s的取反
            return not s
        first_match=bool(s)  and p[0] in {s[0],"."}   # 在 s 非空的条件下，p[0]是否构成匹配
        return first_match and self.isMatch(s[1:],p[1:])
```

如果模式串中的星号出现在第二个位置：这个星号的可能做用有两个

​	-当s[0]==p[0],p[1]="\*",\"*"可以变换出一个2*p[0],这样问题变为s[:]和p的匹配

​	-忽略掉模式串中的这一部分，0*p[0]，问题变换为s[:] 和p[2:]的匹配

其余和没有星号是的匹配是一样的。

```python
class Solution(object):
    def isMatch(self, s, p):
        # print (s,p)
        """
        :type s: str
        :type p: str
        :rtype: bool
        """
        if not p:                  # 如果规则为空，则返回 s的取反
            return not s
        first_match=bool(s)  and p[0] in {s[0],"."}   # 在 s 非空的条件下，p[0]是否构成匹配
        if len(p)>=2 and p[1]=="*":
            #print("a")
            return self.isMatch(s,p[2:]) or (first_match and self.isMatch(s[1:],p))  # 核心重要
        #print("b")
        return first_match and self.isMatch(s[1:],p[1:])
        
```

## 3-leetcode 547.朋友圈

班上有 N 名学生。其中有些人是朋友，有些则不是。他们的友谊具有是传递性。如果已知 A 是 B 的朋友，B 是 C 的朋友，那么我们可以认为 A 也是 C 的朋友。所谓的朋友圈，是指所有朋友的集合。

给定一个 N * N 的矩阵 M，表示班级中学生之间的朋友关系。如果M[i][j] = 1，表示已知第 i 个和 j 个学生互为朋友关系，否则为不知道。你必须输出所有学生中的已知的朋友圈总数。

给定图可以看作无向图的邻接矩阵，问题转换成求无向图中连通块的个数

**深度优先遍历**：从每个节点开始，使用一个大小为N的visted数组, visited[i]表示第i 个元素是够深度优先搜索访问过。

从一个节点开始，不断往下访问它的相邻节点，直至没有未访问过的相邻节点。回溯到之前的节点进行访问

```python
class Solution(object):
    def findCircleNum(self, M):
        """
        :type M: List[List[int]]
        :rtype: int
        """
        def dfs(M,viseted,i):
            for j in range(len(M)):
                if M[i][j]==1 and visited[j]==0:
                    visited[j]=1
                    dfs(M,visited,j)
        n=len(M)
        visited=[0]*n
        count=0
        for i in range(n):
            if visited[i]==0:
                dfs(M,visited,i)
                count+=1
        return count
```

**并查集**

使用一个大小为N的parent数组，遍历图，每个节点都遍历所有相邻的点，并让相同的点指向它，设置一个由parent节点决定的单独数组。每个组中都有一个唯一的parent节点，父亲节点为值为-1

并查集合的主要操作-为集合中的元素找到一个代表（根节点），基本操作是合并两个集合。当拿到两个节点，第一步找到各自的根节点，选择一个节点作为新的代表，可以完成两个集合的合并。

可以使用数组表示并查集，数组下标表示节点编号，对应的值表示该节点的父亲节点的编号。

为什么一致就表示有环，本身有链接，所以会成环

```python
class Solution(object):
    def findCircleNum(self, M):
        """
        :type M: List[List[int]]
        :rtype: int
        """
        def find(parent,i):
            while(parent[i]!=-1):    # 编号为i的节点的父亲节点为-1，则本身为根
                i=parent[i]          # 更新当前节点为父亲节点
            return i								 # 返回的是 根节点，而不是-1 的值。
        def union(parent,x,y):
            x_parent=find(parent,x)
            y_parent=find(parent,y)
            if x_parent!=y_parent:		# 有联系，父亲还不一样 ，把父亲变得与一样。
                parent[x_parent]=y_parent   # 将x_parent 作为y_parent的儿子，成为y_parent的一个分支
           
        n=len(M)
        parent=[-1]*n
        for i in range(n):
            for j in range(n):
                if (M[i][j]==1 and i!=j): # 当两条变之间有联系的时候才合并，一个接到另一个的父亲节点下面
                    union(parent,i,j)
        count=0
        for i in range(n):
            if parent[i]==-1:
                count+=1
        return count
```

## 4-leetcode103.二叉树的锯齿形状遍历

在每层节点添加时，使用双端队列，使得可以从两边加入元素。

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None
from collections import deque

class Solution(object):
    def zigzagLevelOrder(self, root):
        """
        :type root: TreeNode
        :rtype: List[List[int]]
        """
        if root==None:
            return []
        stack=[root]
        res=[]
        right_flag=True         # 从列表尾部添加元素
        while(stack):
            res.append([])
            l=len(stack)
            level_list=deque()
            for i in range(l):
                node=stack.pop(0)
                if right_flag:
                    level_list.append(node.val)
                else:
                    level_list.appendleft(node.val)
                if node.left:
                    stack.append(node.left)
                if node.right:
                    stack.append(node.right)
            for val in level_list:         #额外写一个语句将双端列表的值传递给res 
                res[-1].append(val)
            right_flag= not right_flag
        return res
```



避免出现这个结果：双端队列直接输出的结果不符合格式要求？ **[deque([3]), deque([20, 9]), deque([15, 7])]**



# 2020/06/08

## 1-leetcode 15.三数之和

给你一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？请你找出所有满足条件且不重复的三元组。

注意：答案中不可以包含重复的三元组。

```python
class Solution(object):
    def threeSum(self, nums):
        """
        :type nums: List[int]
        :rtype: List[List[int]]
        """
        nums.sort()													# 排个序，有序的结构，总能遍历很多
        res=[]
        n=len(nums)
        if n<3:
            return res
        for i in range(n-2):
            if nums[i]>0:					# 从某个i开始，如果nums[i]>0，那么其后的三个数字之和哦不可能为0
                break
            elif i>0 and nums[i-1]==nums[i]:# 如果后一个数字和前一个数字相同，避免相同答案，直接跳过
                continue
            else:
                left,right=i+1,n-1# 左右index 三数和大于0，调整右指针；三数之和小于0，调整左指针
                while(left<right):
                    sum_t=nums[i]+nums[left]+nums[right]
                    if sum_t>0:
                        right-=1
                    elif sum_t<0:
                        left+=1
                    else:									# 等于0 之后还需要继续往下查找呦
                        res.append([nums[i],nums[left],nums[right]])
                        left+=1
                        right-=1					#为了避免相同的三元组，指针要指向相同元素的最后一个
                        while(left<right and nums[right]==nums[right+1]):
                            right-=1
                        while(left<right and nums[left]==nums[left-1]):
                            left+=1
        return res

```

为了不包含重复的三元组，有两处处理：1.迭代每个元素时，当前元素与前一个元素相同，跳过

​																	 2.左右指针找到一个合理的组合之后，往后遍历时，从不同的元素开始。

## 2-leetcode 面试题12. 矩阵中的路径

请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一格开始，每一步可以在矩阵中向左、右、上、下移动一格。如果一条路径经过了矩阵的某一格，那么该路径不能再次进入该格子。例如，在下面的3×4的矩阵中包含一条字符串“bfce”的路径（路径中的字母用加粗标出）。

```python
class Solution(object):
    def exist(self, board, word):
        """
        :type board: List[List[str]]
        :type word: str
        :rtype: bool
        """
        m=len(board)
        if m==0:
            return not word
        n=len(board[0])
        visted=[[False]*n for _ in range(m)]
        def dfs(string,i,j):
            if not string:        # 如果字符串为说明这个路径可行了
                return True
            if i>m-1 or j>n-1 or i<0 or j<0:    # 字符串非空，下标已经不符合，这个方向不行
                return False
            if  (-1<i<m and -1<j<n and visted[i][j]):
                return False      # 下标行了，这个节点已经被访问过了
            if board[i][j]==string[0]:# 下标合格，节点没有被访问过，外加节点匹配上了，往下递归。四个方向有一个为真就行。
                visted[i][j]=True       
                if dfs(string[1:],i+1,j) or dfs(string[1:],i,j+1) or dfs(string[1:],i-1,j) or dfs(string[1:],i,j-1):
                    return True 
                visted[i][j]=False
            return False

        for i in range(m):
            for j in range(n):
                if dfs(word,i,j):
                    return True
        return False
        
```

在递归后判断是否符合递归条件。递归的时候就不用判断下标条件了。

先判断条件，再递归



## 3-leetcoe 329.矩阵中的最长递增路径

给定一个整数矩阵，找出最长递增路径的长度。

对于每个单元格，你可以往上，下，左，右四个方向移动。 你不能在对角线方向上移动或移动到边界外（即不允许环绕）。


### dfs 时间超出限制 135/138

```python
class Solution(object):
    def __init__(self):
        self.res=1
    def longestIncreasingPath(self, matrix):
        """
        :type matrix: List[List[int]]
        :rtype: int
        """
        m=len(matrix)
        if m==0:
            return 0
        n=len(matrix[0])
        # 不用visted 如果一个元素已经在递增序列中，它不会再出现在递增序列中。
        # matrix[i][j] 的值要比val 大 ，才会往下递归
        def dfs(i,j,val,l):  
            # print(i,j,val,l)
            if (not -1<i<m) or (not -1<j<n) or matrix[i][j]<=val: # 1.下标到达边界时，2.matrix[i][j]<=val时结算结果
                self.res=max(self.res,l)
                return 
            dfs(i+1,j,matrix[i][j],l+1)
            dfs(i-1,j,matrix[i][j],l+1)
            dfs(i,j+1,matrix[i][j],l+1)
            dfs(i,j-1,matrix[i][j],l+1)
        for i in range(m):
            for j in range(n):
                dfs(i,j,float("-INF"),0)
        return self.res
            
```



### 递归带备忘录

```python
class Solution(object):
    def __init__(self):
        self.res=1
        self.direction=[[0,1],[0,-1],[1,0],[-1,0]]
    def longestIncreasingPath(self, matrix):
        """
        :type matrix: List[List[int]]
        :rtype: int
        """
        m=len(matrix)
        if m==0:
            return 0
        n=len(matrix[0])
        # 不用visted 如果一个元素已经在递增序列中，它不会再出现在递增序列中。
        dp=[[0]*n for _ in range(m)]
        # matrix[i][j] 的值要比val 大 ，才会往下递归
        def dfs(i,j):  
            if dp[i][j]!=0:
                return dp[i][j]
            for d in self.direction:
                next_i,next_y=i+d[0],j+d[1]
                if (0<=next_i<m and 0<=next_y<n and matrix[next_i][next_y]>matrix[i][j]):
                    dp[i][j]=max(dp[i][j],dfs(next_i,next_y))
            dp[i][j]+=1        # 递归到最低层时，往上一层出返回一个1--最右下脚的元素，如果b不满足递增条件就会时初始的0
            return dp[i][j]    # 满足递增 则在下一层的基础上，再加1，所以才初始化时才是0
            
        for i in range(m):
            for j in range(n):
                self.res=max(self.res,dfs(i,j))
                
        return self.res
            
```

### 自底向上的dp--剥洋葱-不会写-不会考

https://leetcode-cn.com/problems/longest-increasing-path-in-a-matrix/submissions/



# 2020/06/09

## 1-leetcode 146.lru缓存机制

用hash 表用来定位，找到缓存项在双向链表中的位置，将其移动到链表的头部，双向链表从头到尾的顺序表示由近及远的访问hash={key:node}

访问一个节点后将节点移动到双向链表的头部--删除一个节点+在头部添加一个节点实现

添加节点时：

对应的key 不在cache中：新建一个节点将节点加在头部，如果长度超过capacity ，删除尾部节点，并移除该节点在cache 中的缓存

对应的key 在cache中：用key取到node,修改Node 的值。



```python
class DLinkedNode:
    def __init__(self,key=0,value=0):           # 自己实现双向链表
        self.key=key
        self.value=value
        self.prev=None
        self.next=None

class LRUCache(object):

    def __init__(self, capacity):
        """
        :type capacity: int
        """
        self.cache={}                   # key
        self.head=DLinkedNode()
        self.tail=DLinkedNode()
        self.head.next=self.tail
        self.tail.pre=self.head
        self.capacity=capacity
        self.size=0


    def get(self, key):
        """
        :type key: int
        :rtype: int
        """
        if key not in self.cache:
            return -1
        node=self.cache[key]
        self.moveToHead(node)
        return node.value
    def removeNode(self,node):          # 双向链表给节点直接删除该节点
        node.prev.next=node.next
        node.next.prev=node.prev
    def addToHead(self,node):
        node.prev=self.head             # 顺序很重要，新节点指向头，下一个，下一个指向新，头指向新
        node.next=self.head.next
        self.head.next.prev=node
        self.head.next=node
    
    def moveToHead(self,node):
        self.removeNode(node)
        self.addToHead(node)
    def removeTail(self):
        node=self.tail.prev
        self.removeNode(node)
        return node

    def put(self, key, value):
        """
        :type key: int
        :type value: int
        :rtype: None
        """
        if key not in self.cache:
            node=DLinkedNode(key,value)
            self.cache[key]=node
            self.addToHead(node)
            self.size+=1
            if self.size>self.capacity:
                removed=self.removeTail()
                self.cache.pop(removed.key)
                self.size-=1

        else:
            node=self.cache[key]
            node.value=value
            self.moveToHead(node)


```

# 2020/06/10

## 1-leetcode 200.岛屿的数量

```python
class Solution(object):
    def numIslands(self, grid):
        """
        :type grid: List[List[str]]
        :rtype: int
        """
        def dfs(r,c):
            grid[r][c]="0"
            for x,y in [(r+1,c),(r-1,c),(r,c+1),(r,c-1)]:
                if 0<=x<m and 0<=y<n and grid[x][y]=="1":
                    dfs(x,y)                # 不需要求解，只需要更改状态
        m=len(grid)
        if m==0:
            return 0
        n=len(grid[0])
        count=0
        for i in range(m):
            for j in range(n):
                if grid[i][j]=="1":
                    count+=1
                    dfs(i,j)
        return count

            
```



## 2-leetcode 20.有效括号

从整体表达式中每次删除一个较小的表达式子，如果是一个有效表达式子，最后应该留下一个空的表达式子。

```python
class Solution(object):
    def isValid(self, s):
        """
        :type s: str
        :rtype: bool
        """
        dit={")":"(","]":"[","}":"{"}
        stack=[]
        for char in s:
            if char in dit:     # char为右括号
                left=stack.pop() if stack else "#"
                if dit[char]!=left:
                    return False
            else:              # char 为左括号入栈
                stack.append(char)
        if len(stack)!=0:
            return False
        return True

```



## 3-leetcode 21.合并两个有序链表

-递归-省掉判断哪个加在哪个后面+一个遍历完，另外一个需要加在其后的尴尬

```python
class Solution(object):
    def mergeTwoLists(self, l1, l2):
        """
        :type l1: ListNode
        :type l2: ListNode
        :rtype: ListNode
        """
        if l1==None:
            return l2
        elif l2==None:
            return l1
        elif l1.val<=l2.val:
            l1.next=self.mergeTwoLists(l1.next,l2)
            return l1
        else:
            l2.next=self.mergeTwoLists(l1,l2.next)
            return l2
```

迭代--新建一个链表头。往新链表的头部一次往后添加对应节点，l1,不是单个节点也没关系，会往后覆盖

```python
class Solution(object):
    def mergeTwoLists(self, l1, l2):
        """
        :type l1: ListNode
        :type l2: ListNode
        :rtype: ListNode
        """
        dummy=ListNode(0)
        curr_node=dummy
        while(l1 and l2):     # 为什么出不去呢,因为粗心
            # print(l1.val,l2.val)
            if l1.val<=l2.val:
                curr_node.next=l1
                l1=l1.next
            else:
                curr_node.next=l2
                l2=l2.next
            curr_node=curr_node.next
        curr_node.next=l1 if l1 else l2
        return dummy.next
```

# 20200611

## 1.leetcode 887高楼扔鸡蛋

你将获得 K 个鸡蛋，并可以使用一栋从 1 到 N  共有 N 层楼的建筑。

每个蛋的功能都是一样的，如果一个蛋碎了，你就不能再把它掉下去。

你知道存在楼层 F ，满足 0 <= F <= N 任何从高于 F 的楼层落下的鸡蛋都会碎，从 F 楼层或比它低的楼层落下的鸡蛋都不会破。（F的值，从F楼往下丢，鸡蛋不会破）

每次移动，你可以取一个鸡蛋（如果你有完整的鸡蛋）并把它从任一楼层 X 扔下（满足 1 <= X <= N）。

你的目标是确切地知道 F 的值是多少。

无论 F 的初始值如何，你确定 F 的值的最小移动次数是多少？

理解：**无论F的值是多少==最坏的情况下**，**至少**需要尝试的**扔鸡蛋的次数**

1.无论F的初始值是多少--考虑最**坏的情况下**F的值：鸡蛋破碎发生在搜索区间的端点

2.**至少**需要尝试的**扔鸡蛋的次数**（鸡蛋个数没有限制）-二分法查找对应的区间定点，log N的时间复杂度

转折点：限制鸡蛋的个数为K，直接使用二分不行-- 一个鸡蛋，7层楼，试一次，不能用二分，只能线性搜索从1楼扔到7楼，找到恰巧碎的那一层，最坏的情况应该是要尝试7次。

### dp

base case 

if N==0: return 0

if K==1:return N 

理论上遍历所有的查找方式（二分三分四分），找最小的尝试次数；每次确定N分之后，选择最坏的情况

for i in range(1,N):

​	res=min(res,max(dp(K-1,i-1),dp(k,N-i))+1)

时间复杂度：o(n)*o(nk)--时间超出限制 61/121

```python
class Solution(object):
    def superEggDrop(self, K, N):
        """
        :type K: int
        :type N: int
        :rtype: int
        """
        def dp(k,n):
            # print(k,n)
            if k==1:
                return n
            if n==0:
                return 0
            if (k,n) in memo:
                return memo[(k,n)]
            res=float("INF")
            for i in range(1,n+1):
                res=min(res,max(dp(k-1,i-1),dp(k,n-i))+1)
            memo[(k,n)]=res
            return res
        memo={}
        val=dp(K,N)
        print(val,memo)
        return dp(K,N)

```

使用二分查找遍历所有的选择：

```python
class Solution(object):
    def superEggDrop(self, K, N):
        """
        :type K: int
        :type N: int
        :rtype: int
        """
        def dp(k,n):
            # print(k,n)
            if k==1:
                return n
            if n==0:
                return 0
            if (k,n) in memo:
                return memo[(k,n)]
            res=float("INF")
            low,hight=1,n
            while(low<=hight):
                mid=(low+hight)//2
                broken=dp(k-1,mid-1)
                not_broken=dp(k,n-mid)
                if broken>not_broken:
                    hight=mid-1
                    res=min(res,broken+1)
                else:
                    low=mid+1
                    res=min(res,not_broken+1)

            memo[(k,n)]=res
            return res
        memo={}
        return dp(K,N)

```

 ### 数学解法

反向求解：如果我们可以做T次操作，而且有K个鸡蛋，能够找到的答案最高是N用f(T,K).依次求出f(T,K)，找到一个最小的T，使其满足f(T,K)>=N 即可。

还是用dp求解f(T,K）,要找最高的N，不必思考到底在哪里扔这个鸡蛋，只需要扔出一个鸡蛋，看到底发生了什么。

f(T,K)：T 次操作，K个鸡蛋能够找到的最高楼数，只考虑在某一层扔出一个鸡蛋，鸡蛋的可能状态有两种：碎或者不碎。

如果鸡蛋没有碎，说明在这一层的上方可以有f(T-1,K)层

如果鸡蛋碎了，说明这一层的下方可以有f(t-1,K-1)层

f(T,K)=1+f(T-1,K-1)+f(T-1,K)

if T>=1: f(T,1)=T.  # 最多能到达的楼层是T

if K>=1: f(1,K)=1	# 只操作一次，最多能够达到的平台高度

```python
class Solution:
    def superEggDrop(self, K: int, N: int) -> int:
        if N == 1:												# 只有一层楼？
            return 1
        f = [[0] * (K + 1) for _ in range(N + 1)]
        for i in range(1, K + 1):
            f[1][i] = 1
        ans = -1
        for i in range(2, N + 1):
            for j in range(1, K + 1):
                f[i][j] = 1 + f[i - 1][j - 1] + f[i - 1][j]
            if f[i][K] >= N:
                ans = i
                break
        return ans

```

## 2-leetcode 22.括号生成

数字 *n* 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 **有效的** 括号组合。

思路：在序列有效时，才放置左右括号。如果左括号的数不大于n,可以放一个左括号，如果右括号数量小于左括号数，可以放一个右括号。

（（（））） 第一个生成的。（（）（））第二个生成的。回溯法做。

```python
class Solution(object):
    def generateParenthesis(self, n):
        """
        :type n: int
        :rtype: List[str]
        """
        def backtrack(s="",left=0,right=0):
            if len(s)==2*n:
                res.append(s)
                return 
            if left<n:		# 边界条件
                backtrack(s+"(",left+1,right)
            if right<left:
                backtrack(s+")",left,right+1)
        res=[]
        backtrack()
        return res
```

# 20200612

## 1-合并K个排序链表

合并 *k* 个排序链表，返回合并后的排序链表。请分析和描述算法的复杂度。

### 两两合并

ans 初始化为第一个链表，然后进行两两合并

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def mergeKLists(self, lists):
        """
        :type lists: List[ListNode]
        :rtype: ListNode
        """
        def merge2list(l1,l2):
            dummy=ListNode(0)
            curr_node=dummy
            while(l1 and l2):
                if l1.val<=l2.val:
                    curr_node.next=l1
                    l1=l1.next
                else:
                    curr_node.next=l2
                    l2=l2.next
                curr_node=curr_node.next
            curr_node.next=l1 if l1 else l2
            return dummy.next
        n=len(lists)
        if n==0:
            return None
        res=lists[0]
        for i in range(1,n):
            res=merge2list(res,lists[i])
        return res
        
```

### 优先队列

每次在队列中放K个链表的第一个元素，用小顶堆找到最小元素，放到答案链表中。然后再降最小元素所在的链表中的下一个元素添加到优先队列中。直至处理完所有的元素。



## 2-leetcode 31.下一个排列

实现获取下一个排列的函数，算法需要将给定数字序列重新排列成字典序中下一个更大的排列。

如果不存在下一个更大的排列，则将数字重新排列成最小的排列（即升序排列）。

必须原地修改，只允许使用额外常数空间。



思路：如果是降序排列的话，不可能有更大的排列

从数组的右边扫描到左边，找到第一对a[i]>a[i-1],a[i:]不可能有更大的排列

需要将a[i-1]的位置替换成右侧区域的数字中比它更大的数字，然后翻转后半部分形成一个最小字典排序

```python
class Solution(object):
    def nextPermutation(self, nums):
        """
        :type nums: List[int]
        :rtype: None Do not return anything, modify nums in-place instead.
        """
        n=len(nums)
        flag=0
        for i in range(n-1,0,-1):
            if nums[i-1]<nums[i]:           # 找到从右到左，出现下降的点
                flag=i
                for k in range(n-1,i-1,-1):  # 替换掉下降的点，为一个较大的点。
                    if nums[k]>nums[i-1]:
                        nums[k],nums[i-1]=nums[i-1],nums[k]
                        break
                break
        nums[flag:]=reversed(nums[flag:])   # 将转折点后面的的数字逆序排列，得到可行解的最小值
        return nums
```



# 3-leetcode 33.搜索旋转排序数组

假设按照升序排序的数组在预先未知的某个点上进行了旋转。

( 例如，数组 [0,1,2,4,5,6,7] 可能变为 [4,5,6,7,0,1,2] )。

搜索一个给定的目标值，如果数组中存在这个目标值，则返回它的索引，否则返回 -1 。

你可以假设数组中不存在重复的元素。

你的算法时间复杂度必须是 O(log n) 级别。

二分法：

```python
class Solution(object):
    def search(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: int
        """
        n=len(nums)
        if n==0:
            return -1								# 两个道题的测试用例是不一样的。
        left,right=0,len(nums)-1
        while(left<right):
            mid=(left+right)//2
            if nums[left]<nums[mid]:  # 左边有序
                if nums[left]<=target<=nums[mid]:
                    right=mid
                else:
                    left=mid+1
            elif nums[left]>nums[mid]:                      # 左边无序
                if nums[left]<=target or target<=nums[mid]:
                    right=mid
                else:
                    left=mid+1
            elif nums[left]==nums[mid]:                  # 左一半存在重复数字，去除重复数字
                while(left<right and nums[left]!=target):
                    left+=1
                if nums[left]==target:                  # 避免nums=[1],target=1，循环进不来的情况
                    right=left
        return left if nums[left]==target else -1
                
```

 

 # 20200613

## 1-leetcode 32.最长有效括号数

给定一个只包含 `'('` 和 `')'` 的字符串，找出最长的包含有效括号的子串的长度。

动态规划：dp[i], s[:i+1]中最长有效括号数量，s[i]字符串数，决定了dp 的更新式子。

> if s[i]\==")" and s[i-1]=="(": dp[i]=dp[i-2]+2
>
> \# 倒数第二个右括号可以形成的有效括号的长度dp[i-1]此前如果有有效左括号可以匹配的话，就可以形成更长的有效括号
>
> if s[i]\==")" and s[i-1]==")":  
>
> ​	if i-1-dp[i-1]>=0 and s[ i-1-dp[i-1]]=="(": dp[i]=dp[i-1]+dp[ i-2-dp[i-1]]+2
>
> 每更新一个dp 都需要更新一次最后的结果，因为中间结果可能更大
>
> 



```python
class Solution(object):
    def longestValidParentheses(self, s):
        """
        :type s: str
        :rtype: int
        """
        n=len(s)
        if n<2:
            return 0
        dp=[0]*n
        res=0
        for i in range(1,n):
            if i==1:
                if s[i]==")" and s[i-1]=="(":
                    dp[1]=2
            else:
                if s[i]==")" and s[i-1]=="(":
                    dp[i]=dp[i-2]+2               
                if s[i]==")" and s[i-1]==")":
                    if i-1-dp[i-1]>=0 and s[i-1-dp[i-1]]=="(":
                        dp[i]=dp[i-1]+2 # index 有效性没有验证
                        if i-2-dp[i-1]>=0:
                            dp[i]+=dp[i-2-dp[i-1]]
            res=max(res,dp[i]) 
        return res

```



## 2-leetcode

给定一个按照升序排列的整数数组 nums，和一个目标值 target。找出给定目标值在数组中的开始位置和结束位置。

你的算法时间复杂度必须是 O(log n) 级别。

如果数组中不存在目标值，返回 [-1, -1]。

### 线性搜索时间o(n)

一次扫描

```python
class Solution(object):
    def searchRange(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: List[int]
        """
        n=len(nums)
        if n==0:
            return [-1,-1]
        left,right=0,n-1
        while(left<=right and nums[left]!=target):
            left+=1
        while(left<=right and nums[right]!=target):
            right-=1
        return [left,right] if left<=right else [-1,-1]
```

### 对数搜索时间log(n)

有序二分每次可以丢弃一半的信息。

二分搜索，每次丢弃一半的区间用于确定左索引或右索引

```python
class Solution(object):
    def searchRange(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: List[int]
        """
        left=self.extreme_index(nums,target,True)
        if left==len(nums) or nums[left]!=target:     # 没有找到target 就退出的两种情况
            return [-1,-1]
        right=self.extreme_index(nums,target,False)-1   # 已经找到过左极限，右极限最多也就是左极限
        return [left,right]
        
    def extreme_index(self,nums,target,left):
       
        low,high=0,len(nums)
        while(low<high):
            mid=(low+high)//2
            # 找左边极限 右边都比左边大，或者nums[mid]落在tagert区间内，向左找数
            # 找右边极限，右边比左边大，就应该往左边走，剩下的nums[mid]落在tagert区间内，向右
            if nums[mid]>target or (nums[mid]==target and left ):
                high=mid        #[7,8]=>mid=7 跳出循环。
            else:
                low=mid+1
        return low

```

## 3.组合总和

### 3.1-leetcode 39.组合总数

给定一个无重复元素的数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。candidates 中的数字可以**无限制重复被选取**。

```python 
class Solution(object):
    def combinationSum(self, candidates, target):
        """
        :type candidates: List[int]
        :type target: int
        :rtype: List[List[int]]
        """
        res=[]
        def backtack(left,right,nums,path,target):
            if target==0:
                res.append(path[:])  # 实现了第一层的深度拷贝,灵魂
            if target<0:
                return 
            for i in range(left,right):
                next_target=target-nums[i]
                path.append(nums[i])
                backtack(i,right,nums,path,next_target) # 配合排序使用，灵魂3
                path.pop()
        candidates.sort()						# 排序，避免重复答案，灵魂2
        backtack(0,len(candidates),candidates,[],target)
        return res
                
```

### 3.2-leetcode 40.组合总数2

给定一个数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。

candidates 中的每个数字在每个组合中**只能使用一次**。有重复。

说明：所有数字（包括目标数）都是正整数。解集不能包含重复的组合。 

误区：不能去重，去重就得不到答案了。

在前一题的基础上，依旧排序，left下标在递归时，加一往下递归，在同一级递归时，相同元素跳过处理，不同级别递归时，可以有重复元素：i>left 是灵魂

```python
class Solution(object):
    def combinationSum2(self, candidates, target):
        """
        :type candidates: List[int]
        :type target: int
        :rtype: List[List[int]]
        """
        res = []

        def backtack(left, right, nums, path, target):
            if target == 0:
                res.append(path[:])
                return
            if target < 0:
                return
            for i in range(left, right):  
                next_target = target - nums[i]
                if i > left and nums[i] == nums[i - 1]:  # i>left是灵魂 同一级上的递归，重复元素去除，不同级别上，相同新元素可以存在
                    continue
                path.append(nums[i])
                backtack(i + 1, right, nums, path, next_target) 
                path.pop()

        candidates.sort()
        backtack(0, len(candidates), candidates, [], target)
        return res
if __name__=="__main__":
    so=Solution()
    print(so.combinationSum2([10,1,2337,7,6,1,5],8))
```



# 20200614

## 1-leetcode.337 打家劫舍3

在上次打劫完一条街道之后和一圈房屋后，小偷又发现了一个新的可行窃的地区。这个地区只有一个入口，我们称之为“根”。 除了“根”之外，每栋房子有且只有一个“父“房子与之相连。一番侦察之后，聪明的小偷意识到“这个地方的所有房屋的排列类似于一棵二叉树”。 如果两个直接相连的房子在同一天晚上被打劫，房屋将自动报警。

计算在不触动警报的情况下，小偷一晚能够盗取的最高金额。

小偷偷二叉树小区-

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def rob(self, root):
        """
        :type root: TreeNode
        :rtype: int
        """
        visted={}
        def dfs(node):
            if node==None:
                return 0
            if visted.get(node):
                return visted[node]
            money=node.val
            if node.left:
                money+=dfs(node.left.left)+dfs(node.left.right)
            if node.right:
                money+=dfs(node.right.left)+dfs(node.right.right)
            child_money=dfs(node.left)+dfs(node.right)
            #print(money,child_money)
            node_money=max(money,child_money)
            visted[node]=node_money
            return node_money        # 一定要返回，
        dfs(root)
        return visted[root] if root else 0
```

TypeError: unsupported operand type(s) for +: 'int' and 'NoneType'



## 2-leetcode 42.接雨水

```python
class Solution(object):
    def trap(self, height):
        """
        :type height: List[int]
        :rtype: int
        """
        n=len(height)
        if n<3:
            return 0
        max_left,max_right=[0]*n,[0]*n
        max_left[0],max_right[-1]=height[0],height[-1]
        curr_max=height[0]
        for i in range(1,n):
            curr_max=max(curr_max,height[i-1])
            max_left[i]=curr_max
        curr_max=height[-1]
        for i in range(n-2,-1,-1):
            curr_max=max(curr_max,height[i+1])
            max_right[i]=curr_max
        rain_val=0
        # print(max_left,max_right)
        for i in range(n):
            h=min(max_left[i],max_right[i])
            rain_val+=max(0,h-height[i])
        return rain_val
```

## 3-leetcode 47.全排列2

给定一个可包含重复数字的序列，返回所有不重复的全排列。

无重复数字的全排列：有先后顺序，所以在选择列表中作出选择之后，用nums[:i]+nums[i+1:]更新选择列表，而不是用下标[i,right]-组合总数

```python
class Solution(object):
    def permuteUnique(self, nums):
        """
        :type nums: List[int]
        :rtype: List[List[int]]
        """
        res=[]
        def dfs(nums,path):
            if len(nums)==0:
                res.append(path[:])
                return
            for i in range(len(nums)):
                if i>0 and nums[i]==nums[i-1]:    # 关键点2: nums[i]与前一个数字nums[i-1]相比，不是相同就可以
                    continue
                else:
                    path.append(nums[i])
                    dfs(nums[:i]+nums[i+1:],path)
                    path.pop()
        nums.sort()                             # 关键点1:排序
        dfs(nums,[])
        return res
```

# 20200615

## 1-leetcode 53.最大子序列和

给定一个整数数组 `nums` ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

```python
class Solution(object):
    def maxSubArray(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        n=len(nums)
        dp=[0]*n # dp[i]表示 nums[:i-1]的最大和，加上nums[i]，之后的最大和
        res=0
        dp[0]=max(nums[0],0)
        for i in range(1,n):
            # if nums[i]>=0:
            #     dp[i]=dp[i-1]+nums[i]
            # else:
            #     dp[i]=max(dp[i-1]+nums[i],0)
            dp[i]=max(dp[i-1]+nums[i],0)  # 如果小于0，说明前面的前缀可以不要。
            res=max(res,dp[i])
        print(dp)
        return res
```

只有负数的时候是**不通过的**，所以需要先更新答案，然后在决定这个前缀和是否有意义。



# 20200616

## 1-leetcode 55.跳跃游戏

给定一个非负整数数组，你最初位于数组的第一个位置。

数组中的每个元素代表你在该位置可以跳跃的最大长度。

判断你是否能够到达最后一个位置。

### dfs 时间超出限制

每个可达位置，都检验一下是否可以到达最后，一个位置的下一个位置，如果往下嵌套将是无穷的，深度优先么。72/75

```python
class Solution(object):
    def canJump(self, nums):
        """
        :type nums: List[int]
        :rtype: bool
        """
        n=len(nums)
        def dfs(l):
            if l>=n-1:
                return True
            for i in range(1,nums[l]+1):
                #print("a",l,i)
                if dfs(l+i):
                    #print("b",l,i)
                    return True
            return False
        return dfs(0)

```



### max_reach

维护一个max_reach 变量，用于记录当前能够达到的最远点。如果max_reach 小于当前遍历点，则最后#

```python
class Solution(object):
    def canJump(self, nums):
        """
        :type nums: List[int]
        :rtype: bool
        """
        n=len(nums)
        max_re=0
        for i in range(n):
            if i>max_re:
                return False
            max_re=max(max_re,nums[i]+i)
        return True
```

# 2020/06/20

## 1-leetcode 200.岛屿的数量


给你一个由 `'1'`（陆地）和 `'0'`（水）组成的的二维网格，请你计算网格中岛屿的数量。

岛屿总是被水包围，并且每座岛屿只能由水平方向或竖直方向上相邻的陆地连接形成。

此外，你可以假设该网格的四条边均被水包围。

Dfs 走过的路径置为0，之后往上下左右四个可以走的方向走

```python
class Solution(object):
    def numIslands(self, grid):
        """
        :type grid: List[List[str]]
        :rtype: int
        """
        def dfs(r,c):
            grid[r][c]="0"
            for i,j in [(r+1,c),(r-1,c),(r,c+1),(r,c-1)]:
                if 0<=i<m and 0<=j<n and grid[i][j]=="1":
                    dfs(i,j)
        count=0
        m=len(grid)
        if m==0:
            return count
        n=len(grid[0])
        for r in range(m):
            for c in range(n):
                if  grid[r][c]=="1":
                    count+=1
                    dfs(r,c)
        return count
```



## 2-leetcode 695.岛屿的最大面积

给定一个包含了一些 0 和 1 的非空二维数组 grid 。

一个 岛屿 是由一些相邻的 1 (代表土地) 构成的组合，这里的「相邻」要求两个 1 必须在水平或者竖直方向上相邻。你可以假设 grid 的四个边缘都被 0（代表水）包围着。

找到给定的二维数组中最大的岛屿面积。(如果没有岛屿，则返回面积为 0 。

```python 
class Solution(object):
    def __init__(self):
        self.count=0
    def maxAreaOfIsland(self, grid):
        """
        :type grid: List[List[int]]
        :rtype: int
        """
        def dfs(r,c):
            grid[r][c]=0 
            self.count+=1
            for i,j in [(r+1,c),(r-1,c),(r,c+1),(r,c-1)]:
                if 0<=i<m and 0<=j<n and grid[i][j]==1:
                    dfs(i,j)
        res=0
        m=len(grid)
        if m==0:
            return res
        n=len(grid[0])
        for r in range(m):
            for c in range(n):
                if grid[r][c]==1:
                    self.count=0
                    dfs(r,c)
                res=max(res,self.count)
        return res
```



不维护全局变量

```python
class Solution(object):
    def __init__(self):
        self.count=0
    def maxAreaOfIsland(self, grid):
        """
        :type grid: List[List[int]]
        :rtype: int
        """
        def dfs(r,c,area):
            grid[r][c]=0 
            area+=1
            for i,j in [(r+1,c),(r-1,c),(r,c+1),(r,c-1)]:
                if 0<=i<m and 0<=j<n and grid[i][j]==1:
                    area=dfs(i,j,area)
            return area
        res,area=0,0
        m=len(grid)
        if m==0:
            return res
        n=len(grid[0])
        for r in range(m):
            for c in range(n):
                if grid[r][c]==1:
                    area=dfs(r,c,0)
                res=max(res,area)
        return res
```

# 20200621

## 1-leetcode 215.第K大的数字

堆排序

```python
class Solution(object):
    def findKthLargest(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: int
        """
        n=len(nums)
        for i in range((n-2)//2,-1,-1):
            self.shift(nums,i,n-1)
        for i in range(n-1,n-k-1,-1):
            nums[i],nums[0]=nums[0],nums[i]
            self.shift(nums,0,i-1)
        return nums[-k]
    def shift(self,nums,fa_index,high):
        tmp=nums[fa_index]
        chil_index=fa_index*2+1
        while(chil_index<=high):
            if chil_index+1<=high and nums[chil_index]<nums[chil_index+1]:
                chil_index+=1
            if tmp <nums[chil_index]:
                nums[fa_index]=nums[chil_index]
                fa_index=chil_index
                chil_index=fa_index*2+1
            else:
                nums[fa_index]=tmp
                break
        nums[fa_index]=tmp
```

# 20200626

## 1-leetcode 75.颜色分类

用三个指针：p0,p2,curr 分别追踪0的最右边界，2的最左边界，当前考虑的元素

While curr <= p2 :

若 nums[curr] = 0 ：交换第 curr个 和 第p0个 元素，并将指针都向右移。（p0 所指向的元素是处理过的，换到curr 位置，不需要再处理）

若 nums[curr] = 2 ：交换第 curr个和第 p2个元素，并将 p2指针左移 。 （P2原来位置上的元素没有被处理过，所以换过之后，curr指针不动）

若 nums[curr] = 1 ：将指针curr右移。

```python
class Solution(object):
    def sortColors(self, nums):
        """
        :type nums: List[int]
        :rtype: None Do not return anything, modify nums in-place instead.
        """
        p0,curr,p2=0,0,len(nums)-1
        while(curr<=p2):
            # print(p0,curr,p2)
            if nums[curr]==1:
                curr+=1
            elif nums[curr]==0:
                nums[p0],nums[curr]=nums[curr],nums[p0]
                p0+=1
                curr+=1
            elif nums[curr]==2:
                nums[curr],nums[p2]=nums[p2],nums[curr]
                p2-=1
        return nums

```

## 2-leetcode 134.加油站

在一条环路上有 N 个加油站，其中第 i 个加油站有汽油 gas[i] 升。

你有一辆油箱容量无限的的汽车，从第 i 个加油站开往第 i+1 个加油站需要消耗汽油 cost[i] 升。你从其中的一个加油站出发，开始时油箱为空。

如果你可以绕环路行驶一周，则返回出发时加油站的编号，否则返回 -1。

说明: 

如果题目有解，该答案即为唯一答案。
输入数组均为非空数组，且长度相同。
输入数组中的元素均为非负数。

直觉上：能不能跳到最后+每个开始节点判断-时间超出限制 30/31

维护一个变量：curr_tank 记录当前邮箱里的总油量，如果在判断能不能达下一个加油站时，curr_tank<0 说明，不能到达下一个加油站。则把下一个加油站作为新的起点，curr_tank=0

上一次重制的加油站到下一个加油站之间的任意一加油站出发，都是不可行的出发点？为什么呢？

Total_tank=0

Currently_tank=0

for i station :

​	total_tank=total_tank+gas[i]-cost[i]

​	curr_tank=curr_tank+gas[i]-cost[i]

if curr_tank<0:

​	currently_tak=0

​	sta

算法保证了可以从N_s出发到达0，但是，从0到n_s是否有足够的油呢？

反正法：假定存在一个加油站k，似的从N_s出发，不能到达K号加油站。

能输出结果的前提是：total_tank>=0也就是：
$$
\sum_{i=1}^N\alpha_i>=0
$$
$\alpha_i=gas[i]-cost[i]$

将N_s和k作为分隔点，将上式子分为3个部分：
$$
\sum_{i=1}^k\alpha_i +\sum_{i=k+1}^{N_s-1}\alpha_i+\sum_{i=N_s}^N\alpha_i>=0
$$
第二项为负数，每个出发点到上一个出发点之间的的curr_tank一定为负数，不然就不会更新出发点了。
$$
\sum_{i=k+1}^{N_s-1}\alpha_i<=0
$$

$$
\sum_{i=1}^k\alpha_i +\sum_{i=N_s}^N\alpha_i>=0
$$

因为假设k为从N_s 出发不可到达的节点，所以
$$
\sum_{i=1}^k\alpha_i +\sum_{i=N_s}^N\alpha_i<0
$$
上两式子矛盾，所以假设不成立，所以不存在到达不了的情况，越就是说都可以到达。

因此N_s出发一定能绕一圈，N_s是一个可行解，依旧题目描述也是一个可行解。

代码是十分简单，理解起来有点绕，反正法

```python
class Solution(object):
    def canCompleteCircuit(self, gas, cost):
        """
        :type gas: List[int]
        :type cost: List[int]
        :rtype: int
        """
        curr_tank,total_tank,start=0,0,0
        for i in range(len(gas)):
            if curr_tank<0:
                start=i
                curr_tank=0
            curr_tank=curr_tank+gas[i]-cost[i]
            total_tank=total_tank+gas[i]-cost[i]
        if total_tank<0:
            return -1
        else:
            return start
        
```

## 3-滑动窗口系列

滑动窗口解题框架

```python
def slidingWindow(s,t):
  need_has={}
  left,right,n=0,0,len(s)
  while(right<n):
    c=s[right]		# c将要移动到窗口内的字符
    right+=1			# 窗口右扩展
    #...						窗口内数据更新
    while(window needs shrink):		# 判断左窗口是否要收缩，满足查找条件就收缩
      # 满足收缩条件更新答案
      d=s[left]		# d将要移出窗口的字符,
      left+=1
      #  将要移除的字符对窗口收缩条件更新
     
```

### 3.1 leetcode 76.最小覆盖串

给你一个字符串 S、一个字符串 T，请在字符串 S 里面找出：包含 T 所有字符的最小子串。

```python
class Solution(object):
    def minWindow(self, s, t):
        """
        :type s: str
        :type t: str
        :rtype: str
        """
        need,win={},{}
        for char in t:
            if need.get(char):
                need[char]+=1
            else:
                need[char]=1
        left,right,n=0,0,len(s)
        curr_mathch,goal_match=0,len(need)
        res,l_res="",float("INF")
        print(need)
        while(right<n):
            #print(left,right,res)
            c=s[right]
            right+=1            
            if need.get(c):     # 如果是需要的字符，更新匹配信息
                if win.get(c):
                    win[c]+=1
                else:
                    win[c]=1
                if win[c]==need[c]:
                    curr_mathch+=1
            
            while(curr_mathch==goal_match):
                # print(left,right,win)
                if right-left<l_res:
                    res=s[left:right]
                    l_res=right-left
                d=s[left]
                left+=1
                if need.get(d):
                    win[d]-=1
                    if win[d]<need[d]:
                        curr_mathch-=1
        return res
                
```



### 3.2 leetcode 438.字符串中字母异位词

给定一个字符串 **s** 和一个非空字符串 **p**，找到 **s** 中所有是 **p** 的字母异位词的子串，返回这些子串的**起始索引**。

滑动串口p->need,win[left,right] 只装need中有的字符，还是is_match==goal_match就可以。

```python
class Solution(object):
    def findAnagrams(self, s, p):
        """
        :type s: str
        :type p: str
        :rtype: List[int]
        """
        need,win={},{}
        for char in p:
            if need.get(char):
                need[char]+=1
            else:
                need[char]=1
        left,right,n,m=0,0,len(s),len(p)
        curr_match,need_match=0,len(need)
        res=[]
        while(right<n):
            c=s[right]
            right+=1
            if need.get(c):
                if win.get(c):
                    win[c]+=1
                else:
                    win[c]=1
                if win[c]==need[c]:
                    curr_match+=1
            while(curr_match==need_match):
                if right-left==m:        # 窗口长度==p长度   
                    res.append(left)
                d=s[left]
                left+=1
                if need.get(d):
                    win[d]-=1
                    if win[d]<need[d]:
                        curr_match-=1
        return res
```



### 3.3 无重复字符最长子串

给定一个字符串，请你找出其中不含有重复字符的 **最长子串** 的长度。

只有一个窗口，当窗口中的字母出现重复字符时，就收缩左窗口，使窗口内的字符一直都没有重复，

每次加一个字符就可以更新一次res

```python
class Solution(object):
    def lengthOfLongestSubstring(self, s):
        """
        :type s: str
        :rtype: int
        """
        win={}
        left,right,n=0,0,len(s)
        res=0
        while(right<n):
            c=s[right]
            right+=1
            if win.get(c):
                win[c]+=1
            else:
                win[c]=1
            while(win[c]>1):
                d=s[left]
                left+=1
                win[d]-=1
            res=max(res,right-left)
        return res
```



### 3.4 leetcode 567.字符串的排列

给定两个字符串 **s1** 和 **s2**，写一个函数来判断 **s2** 是否包含 **s1** 的排列。

换句话说，第一个字符串的排列之一是第二个字符串的子串。

438 字母异位要求找出所有的，这里只有一个就可以输出了。

```python
class Solution(object):
    def checkInclusion(self, s1, s2):
        """
        :type s1: str
        :type s2: str
        :rtype: bool
        """
        need,win={},{}
        for char in s1:
            if need.get(char):
                need[char]+=1
            else:
                need[char]=1
        left,right,n,m=0,0,len(s2),len(s1)
        curr_match,goal_match=0,len(need)
        while(right<n):
            c=s2[right]
            right+=1
            if need.get(c):
                if win.get(c):
                    win[c]+=1
                else:
                    win[c]=1
                if win[c]==need[c]:
                    curr_match+=1
            while(curr_match==goal_match):
                if right-left==m:
                    return True
                d=s2[left]
                left+=1
                if need.get(d):
                    win[d]-=1
                    if win[d]<need[d]:
                        curr_match-=1
        return False
```



### 3.5 剑指offer 38.字符串的排列


输入一个字符串，打印出该字符串中字符的所有排列。 

你可以以任意顺序返回这个字符串数组，但里面不能有重复元素。

```
输入：s = "abc"
输出：["abc","acb","bac","bca","cab","cba"]
```

思路1: 回溯

# 20200627

## 1-leetcode 78.子集

给定一组**不含重复元素**的整数数组 *nums*，返回该数组所有可能的子集（幂集）。

**说明：**解集不能包含重复的子集。

人工求解：0，1，2，3 个元素的子集

全排列，组合，子集问题可以用一些通用的策略解决。这些问题的解空间都非常大，

全排列，组合：N！，子集：$2^N$,在他们的指数级的解法当中，要确保生成结果完整且没有冗余，常用的方法有递归，回溯，基于二进制位掩码和对应位掩码之间的映射字典生成排列组合子集，说的什么我怎么不懂呢？

递归：假设输出子集为空，每一步都向子集中添加一个新的元素，并形成一个新的子集.

<img src="/Users/chenyingying/cyy/TyporaNote/2-Daily-Algorithm.assets/截屏2020-06-27下午2.17.58-3238715.png" alt="截屏2020-06-27下午2.17.58" style="zoom:33%;" />

```python
import copy
class Solution(object):
    def subsets(self, nums):
        """
        :type nums: List[int]
        :rtype: List[List[int]]
        """
        res=[[]]
        for i in range(len(nums)):
            m=len(res)
            for j in range(m):
                temp=copy.copy(res[j])   #  切片深拷贝
                temp.append(nums[i])
                res.append(temp)
        return res
```

# 2020/06/30

## 1-leetcode 79.单词搜索

给定一个二维网格和一个单词，找出该单词是否存在于网格中。

单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。

```python
class Solution(object):
    def exist(self, board, word):
        """
        :type board: List[List[str]]
        :type word: str
        :rtype: bool
        """
        m,n=len(board),len(board[0])
        visited=[[False]*n for _ in range(m)]
        def dfs(r,c,word):
            # print(r,c,word)
            if word=="":
                return True
            if r==-1 or r==m or c==-1 or c==n: # 边缘判断
                return False
            if board[r][c]==word[0] and not visited[r][c]:
                visited[r][c]=True
                for x,y in [(r-1,c),(r+1,c),(r,c-1),(r,c+1)]:
                    if -1<=x<=m and -1<=y<=n:
                        if dfs(x,y,word[1:]):
                            return True
                visited[r][c]=False
            return False
        for r in range(m):
            for c in range(n):
                if dfs(r,c,word):
                    return True
        return False
```

# 20200702

## 1.Leetcode 96.不同的二叉搜索树

给定一个有序序列 1,...,n,根据序列构建一颗二叉搜索树，遍历每个数字i，并以该数字作为根，1,...,i-1左子树，i+1到n为右子树，这样可以讲问题划分成规模较小的子问题。将不同的根构成的所有二叉搜索树数量相加，就是结果。

为了几算不同二叉搜索树的个数，定义两个函数：

1.G(n) 长度为 n的序列的不同二叉树个数

2.F(i,n):以i为根的不同二叉树个数

G(n)由F(i,n)求解：
$$
G(n)=\sum_{i=1}^nF(i,n)
$$
边界情况：G(0)=1 空树，G(1)=1 序列长度为1

![截屏2020-07-02下午11.00.11](/Users/chenyingying/cyy/Coding/README.assets/截屏2020-07-02下午11.00.11.png)