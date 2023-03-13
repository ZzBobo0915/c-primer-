## 剑指offer03.数组中重复的数字

找出数组中重复的数字。


在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。

示例 1：

输入：
[2, 3, 1, 0, 2, 5, 3]
输出：2 或 3 


限制：

2 <= n <= 100000

​    **利用 鸽巢原理，因为出现的元素值 < nums.size(); 所以我们可以将见到的元素放到索引的位置，如果交换时，发现索引处已存在该元素，则重复 O(N) 空间O(1)。**

```c++
class Solution {
public:
    int findRepeatNumber(vector<int>& nums) {
        for (int i = 0; i < nums.size(); ++i) {
            while (nums[i] != i) {
                // 如果元素位置被占，巢里有人，说明发生重复，返回该元素
                if (nums[nums[i]] == nums[i]) return nums[i];
                // 交换nums[i]位置的数，占巢
                swap(nums[i], nums[nums[i]]);
            }
        }
        return -1;
    }
};
```

## 剑指Offer04.二维数组中的查找

在一个 n * m 的二维数组中，每一行都按照从左到右 非递减 的顺序排序，每一列都按照从上到下 非递减 的顺序排序。请完成一个高效的函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

示例:

现有矩阵 matrix 如下：

[
  [1,   4,  7, 11, 15],
  [2,   5,  8, 12, 19],
  [3,   6,  9, 16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]
给定 target = 5，返回 true。

给定 target = 20，返回 false。

限制：

0 <= n <= 1000

0 <= m <= 1000

​    **把这个二位数组逆时针旋转45°看成一个二叉搜索树，从根节点开始搜索，比它小就向左，比他小就向右：**

```c++
class Solution {
public:
    bool findNumberIn2DArray(vector<vector<int>>& matrix, int target) {
        if (matrix.size() == 0 || matrix[0].size() < 1) return false;
        int i = 0, j = matrix[0].size()-1;
        // 从matrix[0][matrix[0].size()-1]开始遍历“二叉搜索树”
        while (i < matrix.size() && j >= 0) {
            if (target == matrix[i][j]) return true;
            else if (target < matrix[i][j]) j--;  // 小 向左
            else i++;                             // 大 向右
        }
        return false;
    }
};
```

## 剑指Offer05.替换空格

请实现一个函数，把字符串 s 中的每个空格替换成"%20"。

示例 1：

输入：s = "We are happy."
输出："We%20are%20happy."


限制：

0 <= s 的长度 <= 10000

​    **简单的方法就是直接用一个新的字符串替换：**

```c++
class Solution {
public:
    string replaceSpace(string s) {
        string res;
        for (int i = 0; i < s.size(); ++i){
            if (s[i] == ' ')  res += "%20";
            else  res += s[i];
        }
        return res;
    }
};
```

​    **更好的方法是原地修改，首先扩充字符串长度，然后从后往前寻找空格并修改：**

```c++
class Solution {
public:
    string replaceSpace(string s) {
        int space_num = 0;
        int n = s.size();
        for (int i = 0; i < n; ++i)
            if (s[i] == ' ') space_num++;
        s.resize(n + space_num * 2);
        for (int i = s.size()-1, j = n-1; j < i; --i, --j) {
            if (s[j] != ' ') s[i] = s[j];
            else {
                s[i] = '0';
                s[--i] = '2';
                s[--i] = '%'; 
            }
        }
        return s;
    }
};
```

## 剑指Offer06.从尾到头打印链表

输入一个链表的头节点，从尾到头反过来返回每个节点的值（用数组返回）。

示例 1：

输入：head = [1,3,2]
输出：[2,3,1]


限制：

0 <= 链表长度 <= 10000

​    **最直接的就是遍历链表并往vector首部插入值：**

```c++
class Solution {
public:
    vector<int> reversePrint(ListNode* head) {
        vector<int> res;
        while (head != nullptr) {
            res.insert(res.begin(), head->val);
            head = head->next;
        }
        return res;
    }
};
```

​    **还可以首先对链表逆置，然后再对链表进行遍历：**

```c++
class Solution {
public:
    vector<int> reversePrint(ListNode* head) {
        ListNode* pre = NULL;
        ListNode* curr = head;
        while (curr) {
            ListNode* next = curr->next;
            curr->next = pre;
            pre = curr;
            curr = next;
        }
        vector<int> res;
        while (pre) {
            res.push_back(pre->val);
            pre = pre->next;
        }
    }
};
```

## 剑指Offer07.重建二叉树

输入某二叉树的前序遍历和中序遍历的结果，请构建该二叉树并返回其根节点。

假设输入的前序遍历和中序遍历的结果中都不含重复的数字。

示例 1:

<img src="C:\Users\zhuzhibo\AppData\Roaming\Typora\typora-user-images\image-20230119164038334.png" alt="image-20230119164038334" style="zoom:50%;" /> 

Input: preorder = [3,9,20,15,7], inorder = [9,3,15,20,7]
Output: [3,9,20,null,null,15,7]
示例 2:

Input: preorder = [-1], inorder = [-1]
Output: [-1]


限制：

0 <= 节点个数 <= 5000

​    **可以采用一种遍历的方法：**

```c++
class Solution {
public:
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        if (!preorder.size()) {
            return nullptr;
        }
        TreeNode* root = new TreeNode(preorder[0]);
        stack<TreeNode*> stk; // 建立栈
        stk.push(root); 
        int inorderIndex = 0;  // 中序遍历的指针
        for (int i = 1; i < preorder.size(); ++i) {  // 逐个先序遍历
            TreeNode* node = stk.top();
            if (node->val != inorder[inorderIndex]) {  // 栈顶元素的值与中序遍历当前的值不相等
                node->left = new TreeNode(preorderVal); // 前序遍历中处在栈顶元素位置最后一位的元素是栈顶元素的左子树
                stk.push(node->left); // 栈顶元素左子树入栈
            }
            else { // 栈顶元素与中序遍历当前元素相同，栈顶即为最下角的左子树
                while (!stk.empty() && stk.top()->val == inorder[inorderIndex]) { // while循环向上返回，寻找位置进行右子树的重建
                    node = stk.top(); // 指针向右扫描中序遍历
                    stk.pop(); // 栈中所有与当前指针所指元素值相等的节点出栈
                    ++inorderIndex;
                }
                node->right = new TreeNode(preorderVal);  // 循环结束后，node所指向栈顶元素即是需要重建右子树的节点
                stk.push(node->right);
            }
        }
        return root;
    }
};
```

## 剑指Offer09.用两个栈实现队列

用两个栈实现一个队列。队列的声明如下，请实现它的两个函数 appendTail 和 deleteHead ，分别完成在队列尾部插入整数和在队列头部删除整数的功能。(若队列中没有元素，deleteHead 操作返回 -1 )

示例 1：

输入：
["CQueue","appendTail","deleteHead","deleteHead","deleteHead"]
[[],[3],[],[],[]]
输出：[null,null,3,-1,-1]
示例 2：

输入：
["CQueue","deleteHead","appendTail","appendTail","deleteHead","deleteHead"]
[[],[],[5],[2],[],[]]
输出：[null,-1,null,null,5,2]
提示：

1 <= values <= 10000
最多会对 appendTail、deleteHead 进行 10000 次调用

​    **双栈实现队列：**

```c++
class CQueue {
public:
    CQueue() {

    }
    
    void appendTail(int value) {
        one.push(value);
    }
    
    int deleteHead() {
        if (one.empty()) return -1;
        while (!one.empty()) {
            two.push(one.top());
            one.pop();
        }
        int res = two.top();
        two.pop();
        while (!two.empty()) {
            one.push(two.top());
            two.pop();
        }
        return res;
    }
private:
    stack<int> one, two; // one用来存储数据，two用来倒腾一次数据以方便delete
};
```

## 剑指Offer10-I.斐波那契数列

写一个函数，输入 n ，求斐波那契（Fibonacci）数列的第 n 项（即 F(N)）。斐波那契数列的定义如下：

F(0) = 0,   F(1) = 1
F(N) = F(N - 1) + F(N - 2), 其中 N > 1.
斐波那契数列由 0 和 1 开始，之后的斐波那契数就是由之前的两数相加而得出。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

示例 1：

输入：n = 2
输出：1
示例 2：

输入：n = 5
输出：5


提示：

0 <= n <= 100

​    **循环判断：**

```c++
class Solution {
public:
    int fib(int n) {
        if (n == 0 || n == 1) return n; // 0,1直接反回
        int res = 1, tmp = 1;
        for (int i = 2; i < n; ++i) { // 因为n=2时也返回1，所以循环从n=3开始
            res += tmp;
            tmp = res - tmp;
            res %= 1000000007; // 求余
        }
        return res;
    }
};
```

## 剑指Offer10-II.青蛙跳台阶问题

一只青蛙一次可以跳上1级台阶，也可以跳上2级台阶。求该青蛙跳上一个 n 级的台阶总共有多少种跳法。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

示例 1：

输入：n = 2
输出：2
示例 2：

输入：n = 7
输出：21
示例 3：

输入：n = 0
输出：1
提示：

0 <= n <= 100

​    **基础动态规划**

```c++
class Solution {
public:
    int numWays(int n) {
        if (n == 0 || n == 1) return 1;
        vector<int> res(n+1); // 存储每级台阶的跳数
        res[0] = 1; res[1] = 1;
        for (int i = 2; i < n+1; ++i) 
            res[i] = res[i-1]%1000000007 + res[i-2]%1000000007;
        return res[n]%1000000007;
    }
};
```

## 剑指Offer11.旋转数组的最小数字

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。

给你一个可能存在 重复 元素值的数组 numbers ，它原来是一个升序排列的数组，并按上述情形进行了一次旋转。请返回旋转数组的最小元素。例如，数组 [3,4,5,1,2] 为 [1,2,3,4,5] 的一次旋转，该数组的最小值为 1。  

注意，数组 [a[0], a[1], a[2], ..., a[n-1]] 旋转一次 的结果为数组 [a[n-1], a[0], a[1], a[2], ..., a[n-2]] 。

示例 1：

输入：numbers = [3,4,5,1,2]
输出：1
示例 2：

输入：numbers = [2,2,2,0,1]
输出：0


提示：

n == numbers.length
1 <= n <= 5000
-5000 <= numbers[i] <= 5000
numbers 原来是一个升序排序的数组，并进行了 1 至 n 次旋转

​    **只会有一个数比他左边的数小，找到这个数据就可以返回，如果没有就说明已经是顺序排列，返回首元素即可。**

```c++
class Solution {
public:
    int minArray(vector<int>& numbers) {
        for (int i = 1; i < numbers.size(); ++i) 
            if (numbers[i] < numbers[i-1]) return numbers[i];
        return numbers[0];
    }
};
```

## 剑指Offer12.矩阵中的路径

定一个 m x n 二维字符网格 board 和一个字符串单词 word 。如果 word 存在于网格中，返回 true ；否则，返回 false 。

单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。

例如，在下面的 3×4 的矩阵中包含单词 "ABCCED"（单词中的字母已标出）。

<img src="C:\Users\zhuzhibo\AppData\Roaming\Typora\typora-user-images\image-20230121135859320.png" alt="image-20230121135859320" style="zoom:50%;" /> 

示例 1：

输入：board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "ABCCED"
输出：true
示例 2：

输入：board = [["a","b"],["c","d"]], word = "abcd"
输出：false

​    **回溯法：**

```c++
class Solution {
public:
    bool exist(vector<vector<char>>& board, string word) {
        if (board.empty() || board[0].empty()) return word.empty();
        for (int row = 0; row < board.size(); ++row) {
            for (int col = 0; col < board[0].size(); ++col) {
                if (board[row][col] == word[0]) {  // 只有word首元素相等时才进行检查
                    if (check(board, row, col, word, 0)) return true;
                }
            } 
        }
        return false;
    }
private:
    bool check(vector<vector<char>>& board, int row, int col, string word, int loc) {
        if (row < 0 || row >= board.size() || col < 0 || col >= board[0].size() || board[row][col] != word[loc]) return false; 
        if (loc == word.size()-1) return true; // 连续word最后一个元素找到，返回true
        board[row][col] = '*'; // 用来判断本次查找是否遍历过
        bool res = check(board, row-1, col, word, loc+1) ||
                   check(board, row, col-1, word, loc+1) ||
                   check(board, row+1, col, word, loc+1) ||
                   check(board, row, col+1, word, loc+1);
        board[row][col] = word[loc]; // 复原
        return res;
    }
};
```

## 剑指Offer14-I.剪绳子

给你一根长度为 n 的绳子，请把绳子剪成整数长度的 m 段（m、n都是整数，n>1并且m>1），每段绳子的长度记为 k[0],k[1]...k[m-1] 。请问 k[0]*k[1]*...*k[m-1] 可能的最大乘积是多少？例如，当绳子的长度是8时，我们把它剪成长度分别为2、3、3的三段，此时得到的最大乘积是18。

示例 1：

输入: 2
输出: 1
解释: 2 = 1 + 1, 1 × 1 = 1
示例 2:

输入: 10
输出: 36
解释: 10 = 3 + 3 + 4, 3 × 3 × 4 = 36
提示：

2 <= n <= 58

​    **方法一：采用动态规划：**

```c++
class Solution {
public:
    int cuttingRope(int n) {
        vector<int> dp(n+1);
        for (int i = 2; i <= n; ++i) {
            int tmp = 0;
            for (int j = 1; j < i; ++j)
                tmp = max(tmp, max(j * (i-j), j * dp[i-j]));
            dp[i] = tmp
        }
        return dp[n];
    }
}
```

​    **方法二：采用数论，任何大于1的数都可以由2，3相加组成，因为2×2>3×3，所以我们需要更多的3：**

```c++
class Solution {
public:
    int cuttingRope(int n) {
        if (n <= 3) return n-1;
        int div = n / 3;
        int rem = n % 3;
        long res = 1;
        for (int i = 0; i < div; ++i) {
            res *= i < div - 1 ? 3 : (rem == 2 ? 3*rem : (3+rem));
            res = res%1000000007;
        }
        return (int)res;
    }
};
```

## 剑指Offer14-II.剪绳子II

给你一根长度为 n 的绳子，请把绳子剪成整数长度的 m 段（m、n都是整数，n>1并且m>1），每段绳子的长度记为 k[0],k[1]...k[m - 1] 。请问 k[0]*k[1]*...*k[m - 1] 可能的最大乘积是多少？例如，当绳子的长度是8时，我们把它剪成长度分别为2、3、3的三段，此时得到的最大乘积是18。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

示例 1：

输入: 2
输出: 1
解释: 2 = 1 + 1, 1 × 1 = 1
示例 2:

输入: 10
输出: 36
解释: 10 = 3 + 3 + 4, 3 × 3 × 4 = 36


提示：

2 <= n <= 1000

```c++
class Solution {
public:
    int cuttingRope(int n) {
        if (n <= 3) return n-1; // 2和3的情况
        long res = 1;
        while (n > 4) { // 循环后的n一定<=4
            res *= 3;
            res %= 100000007;
            n -= 3;
        }
        return (int)(res*n%100000007); // 最后返回res和剩余的n乘积
    }
};
```

## 剑指Offer15.二进制中1的个数

编写一个函数，输入是一个无符号整数（以二进制串的形式），返回其二进制表达式中数字位数为 '1' 的个数（也被称为 汉明重量).）。

提示：

请注意，在某些语言（如 Java）中，没有无符号整数类型。在这种情况下，输入和输出都将被指定为有符号整数类型，并且不应影响您的实现，因为无论整数是有符号的还是无符号的，其内部的二进制表示形式都是相同的。
在 Java 中，编译器使用 二进制补码 记法来表示有符号整数。因此，在上面的 示例 3 中，输入表示有符号整数 -3。


示例 1：

输入：n = 11 (控制台输入 00000000000000000000000000001011)
输出：3
解释：输入的二进制串 00000000000000000000000000001011 中，共有三位为 '1'。
示例 2：

输入：n = 128 (控制台输入 00000000000000000000000010000000)
输出：1
解释：输入的二进制串 00000000000000000000000010000000 中，共有一位为 '1'。
示例 3：

输入：n = 4294967293 (控制台输入 11111111111111111111111111111101，部分语言中 n = -3）
输出：31
解释：输入的二进制串 11111111111111111111111111111101 中，共有 31 位为 '1'。


提示：

输入必须是长度为 32 的 二进制串 。

​    **方法1：循环检查位：**

```c++
class Solution {
public:
    int hammingWeight(uint32_t n) {
        int res = 0;
        for (int i = 0; i < 32; ++i) 
            if (n & (1 << i)) 
                res++;
        return res;
    }
};
```

​    **方法2：位计算优化：**

```c++
class Solution {
public:
    int hammingWeight(uint32_t n) {
        int res = 0;
        while (n) {
            n &= n - 1;
            res++;
        }
        return res;
    }
};
```

## 剑指Offer16.数值的整数次方

实现 pow(x, n) ，即计算 x 的 n 次幂函数（即，xn）。不得使用库函数，同时不需要考虑大数问题。

示例 1：

输入：x = 2.00000, n = 10
输出：1024.00000
示例 2：

输入：x = 2.10000, n = 3
输出：9.26100
示例 3：

输入：x = 2.00000, n = -2
输出：0.25000
解释：2-2 = 1/22 = 1/4 = 0.25


提示：

-100.0 < x < 100.0
-231 <= n <= 231-1
-104 <= xn <= 104

​    **方法一：快速幂+递归：**

```c++
// 递归1
class Solution {
public:
    double myPow(double x, int n) {
        long long N = n; // long long 的原因是的-INT_MIN = INT_MAX+1超出了int范围
        return N >= 0 ? func(x, N) : 1/func(x, -N); 
    }
    double func(double x, long long n) {
        if (n == 0) return 1;
        double y = func(x, n/2);
        return n%2==0 ? y*y : y*y*x; // 若n为偶数，则直接y*y，若为奇数则y*y*x
    }
};
// 递归2
class Solution {
public:
    double myPow(double x, int n) {
        if (n == 0) return 1;
        if (n == 1) return x;
        if (n == -1) return 1 / x;
        double half = myPow(x, n / 2);
        double mod = myPow(x, n % 2);
        return half * half * mod;
    }
};
```

​    **方法二：快读幂＋迭代：**

```c++
class Solution {
public:
    double myPow(double x, int n) {
        long long N = n;
        return N>=0 ? func(x, N) : 1/func(x, -N);
    }
private:
    double func(double x, long long N) {
        double res = 1;
        double tmp = x;
        while (N > 0) {
            if (N % 2 == 1) {
                res *= tmp;
            }
            tmp *= tmp;
            N /= 2;
        }
        return res;
    }
}
```

## 剑指Offer17.打印从1到最大的n位数

输入数字 n，按顺序打印出从 1 到最大的 n 位十进制数。比如输入 3，则打印出 1、2、3 一直到最大的 3 位数 999。

示例 1:

输入: n = 1
输出: [1,2,3,4,5,6,7,8,9]


说明：

用返回一个整数列表来代替打印
n 为正整数

​    **枚举法：**

```c++
class Solution {
public:
    vector<int> printNumbers(int n) {
        int size = pow(10,n);
        vector<int> res(size-1);
        for (int i = size-1; i > 0; --i) res[i-1] = i;
        return res;
    }
};
```

## 剑指Offer18.删除链表的节点

给定单向链表的头指针和一个要删除的节点的值，定义一个函数删除该节点。

返回删除后的链表的头节点。

注意：此题对比原题有改动

示例 1:

输入: head = [4,5,1,9], val = 5
输出: [4,1,9]
解释: 给定你链表中值为 5 的第二个节点，那么在调用了你的函数之后，该链表应变为 4 -> 1 -> 9.
示例 2:

输入: head = [4,5,1,9], val = 1
输出: [4,5,9]
解释: 给定你链表中值为 1 的第三个节点，那么在调用了你的函数之后，该链表应变为 4 -> 5 -> 9.


说明：

题目保证链表中节点的值互不相同
若使用 C 或 C++ 语言，你不需要 free 或 delete 被删除的节点

```c++
class Solution {
public:
    ListNode* deleteNode(ListNode* head, int val) {
        ListNode* p = head;
        ListNode* pre;
        while (p){
            if (p->val == val){
                if (pre == nullptr) return head->next;
                else{
                    pre->next = p->next;
                    p->next = nullptr;
                    return head;
                }
            }
            pre = p;
            p = p->next;
        }
        return head;
    }
};
```

## 剑指Offer19.正则表达式匹配

请实现一个函数用来匹配包含'. '和'*'的正则表达式。模式中的字符'.'表示任意一个字符，而'*'表示它前面的字符可以出现任意次（含0次）。在本题中，匹配是指字符串的所有字符匹配整个模式。例如，字符串"aaa"与模式"a.a"和"ab*ac*a"匹配，但与"aa.a"和"ab*a"均不匹配。

示例 1:

输入:
s = "aa"
p = "a"
输出: false
解释: "a" 无法匹配 "aa" 整个字符串。
示例 2:

输入:
s = "aa"
p = "a*"
输出: true
解释: 因为 '*' 代表可以匹配零个或多个前面的那一个元素, 在这里前面的元素就是 'a'。因此，字符串 "aa" 可被视为 'a' 重复了一次。
示例 3:

输入:
s = "ab"
p = ".*"
输出: true
解释: ".*" 表示可匹配零个或多个（'*'）任意字符（'.'）。
示例 4:

输入:
s = "aab"
p = "c*a*b"
输出: true
解释: 因为 '*' 表示零个或多个，这里 'c' 为 0 个, 'a' 被重复一次。因此可以匹配字符串 "aab"。
示例 5:

输入:
s = "mississippi"
p = "mis*is*p*."
输出: false
s 可能为空，且只包含从 a-z 的小写字母。
p 可能为空，且只包含从 a-z 的小写字母以及字符 . 和 *，无连续的 '*'。

​    **动态规划：**

```c++
// dp1
class Solution {
public:
    bool isMatch(string s, string p) {
        int m = s.size(), n = p.size();
        // 匹配机制，
        auto matches = [&](int i, int j) { 
            if (i == 0) return false;
            if (p[j-1] == '.') return true;
            return s[i-1] == p[j-1];
        };

        vector<vector<int>> f(m+1, vector<int>(n+1));
        f[0][0] = true;
        for (int i = 0; i <= m; ++i) {
            for (int j = 1; j <= n; ++j) {
                if (p[j-1] == '*') {
                    f[i][j] |= f[i][j-2];
                    if (matches(i, j-1)) f[i][j] |= f[i-1][j];
                }else {
                    if (matches(i, j)) f[i][j] |= f[i-1][j-1];
                }
            }
        }
        return f[m][n];
    }
};
// dp2
class Solution {
public:
    bool isMatch(string s, string p) 
    {
        return compareMatch(s, p, 0, 0);
    }
private:
	bool compareMatch(string& s, string& p, int n, int m){
        if (p[m] == '\0') return s[n] == '\0';
        bool first_match = (s[n] != '\0') && (s[n] == p[m] || p[m] == '.');
        if (p[m + 1] != '\0' && p[m + 1] == '*')
            return (first_match && compareMatch(s, p, n + 1, m)) || compareMatch(s, p, n, m + 2);
        else
            return first_match && compareMatch(s, p, n + 1, m + 1);
    }
};
```

## 剑指 Offer 21. 调整数组顺序使奇数位于偶数前面

输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有奇数在数组的前半部分，所有偶数在数组的后半部分。

示例：

输入：nums = [1,2,3,4]
输出：[1,3,2,4] 
注：[3,1,2,4] 也是正确的答案之一。


提示：

0 <= nums.length <= 50000
0 <= nums[i] <= 10000

​    **双指针：**

```c++
class Solution {
public:
    vector<int> exchange(vector<int>& nums) {
        for (int i = 0, j = nums.size()-1; i < j;){
            while (nums[i]%2==1 && i < j) i++;
            while (nums[j]%2==0 && i < j) j--; 
            if (i < j) swap(nums[i++], nums[j--]);
        }
        return nums;
    }
};
```

## 剑指 Offer 22. 链表中倒数第k个节点

输入一个链表，输出该链表中倒数第k个节点。为了符合大多数人的习惯，本题从1开始计数，即链表的尾节点是倒数第1个节点。

例如，一个链表有 6 个节点，从头节点开始，它们的值依次是 1、2、3、4、5、6。这个链表的倒数第 3 个节点是值为 4 的节点。

示例：

给定一个链表: 1->2->3->4->5, 和 k = 2.

返回链表 4->5.

​    **设计快慢指针：**

```c++
class Solution {
public:
    ListNode* getKthFromEnd(ListNode* head, int k) {
        ListNode* pre = head;  //慢指针
        ListNode* fast = head;  //快指针
        for (int i = 0; i < k; ++i) fast = fast->next;  // 先将快指针向前k
        while (fast != nullptr) {  //快慢一起向前，快指针指向最后时慢指针就是所求
            pre = pre->next;
            fast = fast->next;
        }
        return pre;
    }
};
```

## 剑指 Offer 24. 反转链表

定义一个函数，输入一个链表的头节点，反转该链表并输出反转后链表的头节点。

示例

输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL


限制：

0 <= 节点个数 <= 5000

​    **迭代：**

```c++
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode* pre = nullptr;
        ListNode* curr = head;
        while (curr) {
            ListNode* tmp = curr->next; // 存储当前指针的下一个节点
            curr->next = pre;           // 将当前指针next指向前一个节点
            pre = curr;                  
            curr = tmp;                 // 更改为tmp
        }
        return pre;
    }
};
```

## 剑指 Offer 25. 合并两个排序的链表

输入两个递增排序的链表，合并这两个链表并使新链表中的节点仍然是递增排序的。

示例1：

输入：1->2->4, 1->3->4
输出：1->1->2->3->4->4
限制：

0 <= 链表长度 <= 1000

​    **设置一个用来保存的哨兵节点：**

```c++
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        ListNode ret(0);  // 哨兵节点
        ListNode* p = &ret;  // 操作节点，用来保存l1和l2中的元素
        while(l1 && l2) {
            if(l1->val < l2->val) {
                p->next = l1;
                l1 = l1->next;
            }else {
                p->next = l2;
                l2 = l2->next;
            }
            p = p->next;
        }
        p->next = l1 ? l1 : l2;
        return ret.next;   
    }
};
```

## 剑指 Offer 26. 树的子结构

输入两棵二叉树A和B，判断B是不是A的子结构。(约定空树不是任意一个树的子结构)

B是A的子结构， 即 A中有出现和B相同的结构和节点值。

例如:
给定的树 A:

​      3
​     / \

   4   5
  / \
 1   2
给定的树 B：

   4 
  /
 1
返回 true，因为 B 与 A 的一个子树拥有相同的结构和节点值。

示例 1：

输入：A = [1,2,3], B = [3,1]
输出：false
示例 2：

输入：A = [3,4,5,1,2], B = [4,1]
输出：true
限制：

0 <= 节点个数 <= 10000

​    **判断当前节点是否与B根节点相同，相同就进行判断，然后继续向左右子树中寻找与B头节相同的节点：**

```c++
class Solution {
public:
    bool isSubStructure(TreeNode* A, TreeNode* B) {
        if (B == nullptr) return false;
        if (A == nullptr) return false;
        if (A->val == B->val)
            if (judge(A, B)) return true;
        return isSubStructure(A->left, B) || isSubStructure(A->right, B);
    }
    bool judge(TreeNode* A, TreeNode* B) {
        if (B == nullptr) return true;
        if (A == nullptr) return false;
        return (A->val==B->val) && judge(A->left, B->left) && judge(A->right, B->right);
    }
};
```

## 剑指 Offer 27. 二叉树的镜像

请完成一个函数，输入一个二叉树，该函数输出它的镜像。

例如输入：

​     4

   /   \
  2     7
 / \   / \
1   3 6   9
镜像输出：

​     4

   /   \
  7     2
 / \   / \
9   6 3   1

 示例 1：

输入：root = [4,2,7,1,3,6,9]
输出：[4,7,2,9,6,3,1]


限制：

0 <= 节点个数 <= 1000

​    **先把左子树或右子树存好，然后递归左右子树：**

```c++
class Solution {
public:
    TreeNode* mirrorTree(TreeNode* root) {
        if (root == nullptr) return nullptr;
        TreeNode* p = root->left;  // 先把左子树存好
        root->left = mirrorTree(root->right);  // 新树的左子树等于右子树
        root->right = mirrorTree(p);  // 右子树同理
        return root;
    }
};
```

## 剑指 Offer 28. 对称的二叉树

请实现一个函数，用来判断一棵二叉树是不是对称的。如果一棵二叉树和它的镜像一样，那么它是对称的。

例如，二叉树 [1,2,2,3,4,4,3] 是对称的。

​    1

   / \
  2   2
 / \ / \
3  4 4  3
但是下面这个 [1,2,2,null,3,null,3] 则不是镜像对称的:

​    1

   / \
  2   2
   \   \
   3    3

示例 1：

输入：root = [1,2,2,3,4,4,3]
输出：true
示例 2：

输入：root = [1,2,2,null,3,null,3]
输出：false


限制：

0 <= 节点个数 <= 1000

​    **只需要判断左子树的左与右子树的右还有右子树的左与左子树的右是否完全一致（根节点左右都一样）**

```c++
class Solution {
public:
    bool isSymmetric(TreeNode* root) {
        if (root == nullptr) return true;
        return judge(root->left, root->right);
    }

    bool judge(TreeNode* L, TreeNode* R) {
        if (L==nullptr && R!=nullptr || L!=nullptr && R==nullptr) return false;
        if (L==nullptr && R==nullptr) return true; 
        if (L->val == R->val) return judge(L->left, R->right)&&judge(L->right, R->left);
        else return false;
    }
};
```

## 剑指 Offer 29. 顺时针打印矩阵

输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字。

示例 1：

输入：matrix = [[1,2,3],[4,5,6],[7,8,9]]
输出：[1,2,3,6,9,8,7,4,5]
示例 2：

输入：matrix = [[1,2,3,4],[5,6,7,8],[9,10,11,12]]
输出：[1,2,3,4,8,12,11,10,9,5,6,7]


限制：

0 <= matrix.length <= 100
0 <= matrix[i].length <= 100

​    **设置四个指针来表示上下左右四个边界，每次遍历一个边，遍历完判断边界是否出错，出错则为整个数组遍历完毕：**

```c++
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        if (matrix.empty()) return {};
        vector<int> ret;
        int left = 0, right = matrix[0].size()-1, up = 0, down = matrix.size()-1;  // 设置上、下、左、右四个边界
        while (true) {
            for (int i = left; i <= right; i++) ret.push_back(matrix[up][i]);  // 从左往右
            if (++up > down) break;
            for (int i = up; i <= down; i++) ret.push_back(matrix[i][right]);  // 从上往下
            if (--right < left) break;
            for (int i = right; i >= left; --i) ret.push_back(matrix[down][i]);  // 从右往左
            if (--down < up) break;
            for (int i = down; i >= up; --i) ret.push_back(matrix[i][left]);  // 从下往上
            if (++left > right) break;
        }
        return ret;
    }
};
```

## 剑指 Offer 30. 包含min函数的栈

定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的 min 函数在该栈中，调用 min、push 及 pop 的时间复杂度都是 O(1)。

示例:

MinStack minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
minStack.min();   --> 返回 -3.
minStack.pop();
minStack.top();      --> 返回 0.
minStack.min();   --> 返回 -2.


提示：

各函数的调用总次数不超过 20000 次

​    **用一个栈实现，每次入栈前先把当前最小元素入栈，然后再判断入栈是否小于当前最小元素，然后再入栈元素；出栈先出栈，然后将出栈后的栈顶元素赋给Min，最后在出栈。**

```c++
class MinStack {
public:
    /** initialize your data structure here. */
    int Min = INT_MAX;  // 保存最小的元素
    stack<int> st;  // 栈
    MinStack() {
        
    }
    
    void push(int x) {  // 每次入栈都入两个元素，一个入栈前的最小元素，另一个是入栈元素
        st.push(Min);
        if (x < Min) Min = x;
        st.push(x);
    }
    
    void pop() {  // 这样可以取到出栈后的当前栈最小元素
        st.pop();
        Min = st.top();
        st.pop();
    }
    
    int top() {
        return st.top();
    }
    
    int min() {
        return Min;
    }

};
```

## 剑指 Offer 31. 栈的压入、弹出序列

输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如，序列 {1,2,3,4,5} 是某栈的压栈序列，序列 {4,5,3,2,1} 是该压栈序列对应的一个弹出序列，但 {4,3,5,1,2} 就不可能是该压栈序列的弹出序列。

示例 1：

输入：pushed = [1,2,3,4,5], popped = [4,5,3,2,1]
输出：true
解释：我们可以按以下顺序执行：
push(1), push(2), push(3), push(4), pop() -> 4,
push(5), pop() -> 5, pop() -> 3, pop() -> 2, pop() -> 1
示例 2：

输入：pushed = [1,2,3,4,5], popped = [4,3,5,1,2]
输出：false
解释：1 不能在 2 之前弹出。


提示：

0 <= pushed.length == popped.length <= 1000
0 <= pushed[i], popped[i] < 1000
pushed 是 popped 的排列。

​    **用栈推就行：**

```c++
class Solution {
public:
    bool validateStackSequences(vector<int>& pushed, vector<int>& popped) {
        stack<int> stk;
        int loc = 0;
        for (int i = 0; i < popped.size(); ++i){ // 从popped里一个一个推
            stk.push(pushed[i]);  // 推一个入一个
            while (!stk.empty() && stk.top() == popped[loc]){  // 直到找到入栈元素与出栈元素相同的那个
                stk.pop();
                ++loc;
            }
        }
        return stk.empty(); // 栈不空就不符合
    }
};
```

## 剑指 Offer 32 - I. 从上到下打印二叉树

从上到下打印出二叉树的每个节点，同一层的节点按照从左到右的顺序打印。

例如:
给定二叉树: [3,9,20,null,null,15,7],

​    3

   / \
  9  20
    /  \
   15   7
返回：

[3,9,20,15,7]


提示：

节点总数 <= 1000

​    **队列实现，左子树先进，右子树后进：**

```c++
class Solution {
public:
    vector<int> levelOrder(TreeNode* root) {
        if (root == nullptr) return {};
        vector<int> ret;
        deque<TreeNode*> deq;
        deq.push_back(root);
        while (!deq.empty()) {  // 队列为空时表示把所有节点遍历完毕
            TreeNode* p = deq[0];
            deq.pop_front();
            if (p->left) deq.push_back(p->left);  // 左子树先进
            if (p->right) deq.push_back(p->right);  // 右子树再进
            ret.push_back(p->val);
        }
        return ret;
        
    }
};
```

## 剑指 Offer 33. 二叉搜索树的后序遍历序列

输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历结果。如果是则返回 true，否则返回 false。假设输入的数组的任意两个数字都互不相同。

参考以下这颗二叉搜索树：

​      5
​     / \

   2   6
  / \
 1   3
示例 1：

输入: [1,6,3,2,5]
输出: false
示例 2：

输入: [1,3,2,6,5]
输出: true


提示：

数组长度 <= 1000
通过次数206,173提交次数363,063

​    **每次寻找后序遍历中的父节点，找到后再从父结点反向寻找右**

```c++
class Solution {
public:
    bool verifyPostorder(vector<int>& postorder) {
        return judge(postorder, 0, postorder.size()-1);
    }   
    bool judge(vector<int>& pos, int left, int right) {
        if (left >= right) return  true;  // 区域不对时说明遍历完毕这部分子树返回true
        int root = left;  // 下边遍历寻找根节点
        while (pos[root] < pos[right]) ++root;  // 寻找比根节点大的第一个数，这个数右边区域一定是在右子树中
        int i = root;  
        while (pos[i] > pos[right]) ++i;  // 判断右部分是否都大于根节点
        if (i != right) return false;  // 正常全部大于就会遍历回根节点
        return judge(pos, left, root-1) && judge(pos, root, right-1);  // 检查左子树与右子树是否符合
    }   
};
```

## 剑指 Offer 34. 二叉树中和为某一值的路径

给你二叉树的根节点 root 和一个整数目标和 targetSum ，找出所有 从根节点到叶子节点 路径总和等于给定目标和的路径。

叶子节点 是指没有子节点的节点。

示例 1：

<img src="C:\Users\zhuzhibo\AppData\Roaming\Typora\typora-user-images\image-20230227111834942.png" alt="image-20230227111834942" style="zoom:50%;" />

输入：root = [5,4,8,11,null,13,4,7,2,null,null,5,1], targetSum = 22
输出：[[5,4,11,2],[5,8,4,5]]
示例 2：

输入：root = [1,2,3], targetSum = 5
输出：[]
示例 3：

输入：root = [1,2], targetSum = 0
输出：[]


提示：

树中节点总数在范围 [0, 5000] 内
-1000 <= Node.val <= 1000
-1000 <= targetSum <= 1000

​    **从根节点遍历到所有叶子节点**：

```c++
class Solution {
public:
    vector<vector<int>> ret;
    vector<vector<int>> pathSum(TreeNode* root, int target) {
        vector<int> tmp;
        findT(root, target, tmp);
        return ret;
    }
    void findT(TreeNode* root, int target, vector<int> tmp) {
        if (root == nullptr) return;
        tmp.push_back(root->val);
        target -= root->val;
        if (target == 0 && root->left == nullptr && root->right == nullptr) ret.push_back(tmp);  // 是叶子节点并且满足要求
        findT(root->left, target, tmp);  // 左子树遍历
        findT(root->right, target, tmp);  // 右子树遍历
        
    }
};
```

## 剑指 Offer 35. 复杂链表的复制

请实现 copyRandomList 函数，复制一个复杂链表。在复杂链表中，每个节点除了有一个 next 指针指向下一个节点，还有一个 random 指针指向链表中的任意节点或者 null。

示例 1：

<img src="C:\Users\zhuzhibo\AppData\Roaming\Typora\typora-user-images\image-20230228083815327.png" alt="image-20230228083815327" style="zoom:70%;" />

输入：head = [[7,null],[13,0],[11,4],[10,2],[1,0]]
输出：[[7,null],[13,0],[11,4],[10,2],[1,0]]
示例 2：

输入：head = [[1,1],[2,1]]
输出：[[1,1],[2,1]]
示例 3：

输入：head = [[3,null],[3,0],[3,null]]
输出：[[3,null],[3,0],[3,null]]
示例 4：

输入：head = []
输出：[]
解释：给定的链表为空（空指针），因此返回 null。


提示：

-10000 <= Node.val <= 10000
Node.random 为空（null）或指向链表中的节点。
节点数目不超过 1000 。

​    **用map过渡：**

```c++
class Solution {
public:
    Node* copyRandomList(Node* head) {
        map<Node*, Node*> mp;  // map存储链表元素
        Node* p = head;
        while (p) {  // 先把所有元素存入map中，不考虑next贺random
            Node* tmp = new Node(p->val);
            mp[p] = tmp;
            p = p->next;
        }
        p = head;
        while (p) {  // 再次遍历所有元素，并从map中取出next与random
            mp[p]->next = mp[p->next];
            mp[p]->random = mp[p->random];
            p = p->next;
        }
        return mp[head];
    }
};
```

## 剑指 Offer 36. 二叉搜索树与双向链表

输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的循环双向链表。要求不能创建任何新的节点，只能调整树中节点指针的指向。

为了让您更好地理解问题，以下面的二叉搜索树为例：

<img src="C:\Users\zhuzhibo\AppData\Roaming\Typora\typora-user-images\image-20230228092626608.png" alt="image-20230228092626608" style="zoom:50%;" />

我们希望将这个二叉搜索树转化为双向循环链表。链表中的每个节点都有一个前驱和后继指针。对于双向循环链表，第一个节点的前驱是最后一个节点，最后一个节点的后继是第一个节点。

下图展示了上面的二叉搜索树转化成的链表。“head” 表示指向链表中有最小元素的节点。

<img src="C:\Users\zhuzhibo\AppData\Roaming\Typora\typora-user-images\image-20230228092652463.png" alt="image-20230228092652463" style="zoom:67%;" />

特别地，我们希望可以就地完成转换操作。当转化完成以后，树中节点的左指针需要指向前驱，树中节点的右指针需要指向后继。还需要返回链表中的第一个节点的指针。

​    **采用中序遍历，并且记录当前的前序节点与整个数的最小值节点：**

```c++
class Solution {
public:
    Node* pre, *head;  // 最小值节点head，前序结点pre
    Node* treeToDoublyList(Node* root) {
        midOrder(root);  // 中序遍历，root为当前节点
        head->left = pre;  // 将最小值节点的左指针指向最后的前序节点，即最大值节点
        pre->right = head;  // 反之
        return head;
    }
    void midOrder(Node* root) {
        if (root == nullptr) return;
        midOrder(root->left)                                                     ;
        if (pre != nullptr) pre->right = root;  // 如果前序节点不为空，把前序节点右指针指向当前节点
        else head = root;  // 如果为空，说明当前节点为最左值（最小值）节点，保存为head
        root->left = pre;  // 当前节点的左指针指向前序节点
        pre = root;  // 将当前节点设置为前序节点
        midOrder(root->right);
    }
};
```

## 剑指 Offer 37. 序列化二叉树

请实现两个函数，分别用来序列化和反序列化二叉树。

你需要设计一个算法来实现二叉树的序列化与反序列化。这里不限定你的序列 / 反序列化算法执行逻辑，你只需要保证一个二叉树可以被序列化为一个字符串并且将这个字符串反序列化为原始的树结构。

提示：输入输出格式与 LeetCode 目前使用的方式一致，详情请参阅 LeetCode 序列化二叉树的格式。你并非必须采取这种方式，你也可以采用其他的方法解决这个问题。

示例：

<img src="C:\Users\zhuzhibo\AppData\Roaming\Typora\typora-user-images\image-20230228092834431.png" alt="image-20230228092834431" style="zoom:67%;" />


输入：root = [1,2,3,null,null,4,5]
输出：[1,2,3,null,null,4,5]

​    **使用字符串输入流去做，**

```c++
class Codec {
public:
    TreeNode* stt(istringstream& ss) {  // 转入输入流
        string tmp;
        ss >> tmp;
        if (tmp == "#") return nullptr;
        int x = stoi(tmp);
        TreeNode* root = new TreeNode(x);
        root->left = stt(ss);
        root->right = stt(ss);
        return root;
    }

    // Encodes a tree to a single string.
    string serialize(TreeNode* root) {  // 前序遍历存储为字符串，每个节点中间加空格分开，空节点用"#"表示
        if (root == nullptr) return "#";
        return to_string(root->val) + " " + serialize(root->left) + " " + serialize(root->right);
    }

    // Decodes your encoded data to tree.
    TreeNode* deserialize(string data) {  // 将字符串转为字符串流，依次判断节点
        istringstream ss(data);
        return stt(ss);
    }
};

// Your Codec object will be instantiated and called as such:
// Codec codec;
// codec.deserialize(codec.serialize(root));
```

## 剑指 Offer 38. 字符串的排列

输入一个字符串，打印出该字符串中字符的所有排列。

你可以以任意顺序返回这个字符串数组，但里面不能有重复元素。

示例:

输入：s = "abc"
输出：["abc","acb","bac","bca","cab","cba"]


限制：

1 <= s 的长度 <= 8

​    **遍历每个元素，每个元素遍历时二次循环遍历后面的所有元素，如果二次遍历中有与该元素相同，由于交换了也是重复的，所以不需要交换进行，直到loc与s长度相等，说明找到一个排列：**

```c++
class Solution {
public:
    void dfs(vector<string>& ret, string s, int loc) {
        if (loc == s.size()) ret.push_back(s);  // 招到了排列的，加入vector
        for (int i = loc; i < s.size(); ++i) {  // 从当前loc元素开始遍历
            bool tmp = true;  // 用来判断loc与i中的元素是否有与i相同的
            for (int j = loc; j < i; ++j)
                if (s[i] == s[j]) tmp = false;
            if (tmp) {  // 没有重复的，交换为一个新字符串继续寻找
                swap(s[i], s[loc]);
                dfs(ret, s, loc+1);
                swap(s[i], s[loc]);  // 记得交换回来
            }
        }
    }
    vector<string> permutation(string s) {
        vector<string> ret;
        dfs(ret, s, 0);
        return ret;
    }
};
```

## 剑指 Offer 39. 数组中出现次数超过一半的数字

数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。

你可以假设数组是非空的，并且给定的数组总是存在多数元素。

示例 1:

输入: [1, 2, 3, 2, 2, 2, 5, 4, 2]
输出: 2


限制：

1 <= 数组长度 <= 50000

​    **用map就可以：**

```c++
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        map<int, int> mp;
        for (int i = 0; i < nums.size(); ++i) {
            mp[nums[i]]++;
            if (mp[nums[i]] > nums.size()/2) return nums[i]; 
        }
        return -1;
    }
};
```

## 剑指 Offer 40. 最小的k个数

输入整数数组 arr ，找出其中最小的 k 个数。例如，输入4、5、1、6、2、7、3、8这8个数字，则最小的4个数字是1、2、3、4。

示例 1：

输入：arr = [3,2,1], k = 2
输出：[1,2] 或者 [2,1]
示例 2：

输入：arr = [0,1,2,1], k = 1
输出：[0]


限制：

0 <= k <= arr.length <= 10000
0 <= arr[i] <= 10000

​    **方法1：排序再取值**

```c++
class Solution {
public:
    vector<int> getLeastNumbers(vector<int>& arr, int k) {
        if (k == 0) return {};
        sort(arr.begin(), arr.end());
        return vector<int>(arr.begin(), arr.begin()+k);
    }
};
```

​    **方法2：使用堆**

```c++
class Solution {
public:
    vector<int> getLeastNumbers(vector<int>& arr, int k) {
        vector<int> ret(k, 0);
        if (k == 0) return {};
        priority_queue<int> q;  // 优先队列模拟小根堆默认为priority_queue<int, less<int>>，大数优先
        for (int i = 0; i < arr.size(); ++i) {
            if (i < k) q.push(arr[i]);
            else {
                if (arr[i] < q.top()) {  // 小于小根堆最大值，出队，arr[i]进队
                    q.pop();
                    q.push(arr[i]);
                }
            }
        }
        for (int i = 0; i < k; ++i) {
            ret[i] = q.top();
            q.pop();
        }
        return ret;
    }
};
```

## 剑指 Offer 41. 数据流中的中位数

如何得到一个数据流中的中位数？如果从数据流中读出奇数个数值，那么中位数就是所有数值排序之后位于中间的数值。如果从数据流中读出偶数个数值，那么中位数就是所有数值排序之后中间两个数的平均值。

例如，

[2,3,4] 的中位数是 3

[2,3] 的中位数是 (2 + 3) / 2 = 2.5

设计一个支持以下两种操作的数据结构：

void addNum(int num) - 从数据流中添加一个整数到数据结构中。
double findMedian() - 返回目前所有元素的中位数。
示例 1：

输入：
["MedianFinder","addNum","addNum","findMedian","addNum","findMedian"]
[[],[1],[2],[],[3],[]]
输出：[null,null,null,1.50000,null,2.00000]
示例 2：

输入：
["MedianFinder","addNum","findMedian","addNum","findMedian"]
[[],[2],[],[3],[]]
输出：[null,null,2.00000,null,2.50000]


限制：

最多会对 addNum、findMedian 进行 50000 次调用。

​    **方法一：每次添加数据是按顺序添加，并且统计已加入的个数**

```c++
class MedianFinder {
public:
    /** initialize your data structure here. */
    int length = 0;
    vector<double> vec;
    MedianFinder() {

    }
    
    void addNum(int num) {  // 添加时就按照顺序添加
        auto it = vec.begin();
        for (; it != vec.end(); ++it) {
            if (num < *it) {
                vec.insert(it, num);
                break;
            }
        }
        if (it == vec.end()) vec.push_back(num);
        length++;
    }
    
    double findMedian() {
        if (length % 2 == 1) return vec[length/2];
        else return (vec[length/2]+vec[length/2-1]) / 2;
    }
};
```

​    **方法二：使用堆（优先队列）**

```c++
class MedianFinder {
public:
    /** initialize your data structure here. */
    // 要一直保证qMin.size()+1=qMax.size()或qMin.size()=qMax.size()
    // 默认小根堆priority_queue<T, vector<T>, less<typename Container::value_type>>
    priority_queue<int, vector<int>, less<int>> qMin;  // 小根堆
    priority_queue<int, vector<int>, greater<int>> qMax;  // 大根堆
    MedianFinder() {
    }
    
    void addNum(int num) {
        if (qMin.empty() || num <= qMin.top()) {  // 向小根堆中push
            qMin.push(num);
            if (qMax.size()+1 < qMin.size()) {  // 两个堆size差超过1
                qMax.push(qMin.top());  // 将元素推入大根堆
                qMin.pop();  // 出小根堆
            }
        }else {
            qMax.push(num);  // 向大根堆push
            if (qMin.size() < qMax.size()) {  // 大根堆如果大于小根堆size
                qMin.push(qMax.top());
                qMax.pop();
            }
        }
    }
    
    double findMedian() {
        if (qMax.size() == qMin.size()) return (qMin.top()+qMax.top()) / 2.0;  // 相等就是偶数，返回两个堆顶和的平均值
        else return qMin.top();  // 奇数，返回小根堆堆顶
    }
};
```

## 剑指 Offer 42. 连续子数组的最大和

输入一个整型数组，数组中的一个或连续多个整数组成一个子数组。求所有子数组的和的最大值。

要求时间复杂度为O(n)。

示例1:

输入: nums = [-2,1,-3,4,-1,2,1,-5,4]
输出: 6
解释: 连续子数组 [4,-1,2,1] 的和最大，为 6。


提示：

1 <= arr.length <= 10^5
-100 <= arr[i] <= 100

```c++
// 采用动态规划
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        if (nums.size() == 0) return 0;
        int ret = nums[0];
        for (int i = 1; i < nums.size(); ++i) {
            // 如果当前数小于它与前面数的和，使当前数等于它+它前一个数的和
            // 其实就是每次寻找当前数的最大和值
            if (nums[i] < nums[i]+nums[i-1]) nums[i] = nums[i]+nums[i-1]; 
            if (nums[i] > ret) ret = nums[i];  // 判断是否为最大
        }
        return ret;
    } 
};    
```

## 剑指 Offer 43. 1～n 整数中 1 出现的次数

输入一个整数 n ，求1～n这n个整数的十进制表示中1出现的次数。

例如，输入12，1～12这些整数中包含1 的数字有1、10、11和12，1一共出现了5次。

示例 1：

输入：n = 12
输出：5
示例 2：

输入：n = 13
输出：6


限制：

1 <= n < 2^31

​    **从低位枚举每一个位上的1：**

<img src="C:\Users\zhuzhibo\AppData\Roaming\Typora\typora-user-images\image-20230301094044934.png" alt="image-20230301094044934" style="zoom:100%;" />

```c++
class Solution {
public:
    int countDigitOne(int n) {
        long tmp = 1;  // 1在该位前出现次数（也可以表示位数）
        int ret;
        for (int k = 0; n >= tmp; ++k) {  
            // 比如1234567，假设为第三位(百位) tmp=100
            // n/(tmp*10)表示有1234个循环
            // tmp表示出现了100次
            // 对剩余的n%(tmp*10)=[000, 567](记为n`)中出现1的次数讨论
            // (1)当n`<100时，百位上不出现1
            // (2)当100<=n`<200时，百位上的1的范围为[100, n`],即为n`-100+1次1
            // (3)当n`>=200时，百位上的1全部出现
            // 可以看出在n`<100，我们希望得到0，当n`>=200，我们希望得到100(即tmp)
            // 所以可以得出这部分为min( max(n&(tmp*10)-tmp+1, 0L), tmp);
            // 注意max和min需要两个数类型完全一致，0需要变为0L或者long(0)
            ret += n/(tmp*10) * tmp + min( max(n%(tmp*10)-tmp+1, 0L), tmp);
            tmp *= 10;
        }
        return ret;
    }
};
```

## 剑指 Offer 44. 数字序列中某一位的数字

数字以0123456789101112131415…的格式序列化到一个字符序列中。在这个序列中，第5位（从下标0开始计数）是5，第13位是1，第19位是4，等等。

请写一个函数，求任意第n位对应的数字。

示例 1：

输入：n = 3
输出：3
示例 2：

输入：n = 11
输出：0


限制：

0 <= n < 2^31

​    **按位数寻找到对应的数字的位数，再寻找到第n位的对应数字：**

```c++
class Solution {
public:
    int findNthDigit(int n) {
        int zhanwei = 1;  // 位数
        // long和num如果位int会超
        long sum = 9;  // 位数和，下标从0开始，所以sum为9，如果下表从1开始sum应该为10
        long num = 1;  // 最大数
        while (n > sum) {  // 找到n对应的的位数
            n -= sum;
            zhanwei++;
            num *= 10;
            sum = zhanwei * 9 * num;
        }
        sum = pow(10, zhanwei-1) + (n-1) / zhanwei;  // 置sum为该位数所处的数字
        return to_string(sum)[(n-1)%zhanwei]-'0';  
    }
};
```

## 剑指 Offer 46. 把数字翻译成字符串

给定一个数字，我们按照如下规则把它翻译为字符串：0 翻译成 “a” ，1 翻译成 “b”，……，11 翻译成 “l”，……，25 翻译成 “z”。一个数字可能有多个翻译。请编程实现一个函数，用来计算一个数字有多少种不同的翻译方法。

示例 1:

输入: 12258
输出: 5
解释: 12258有5种不同的翻译，分别是"bccfi", "bwfi", "bczi", "mcfi"和"mzi"

​    **动态规划：**

```c++
class Solution {
public:
    // (1)一位 1~9
    // (2)两位 第一位1或者2
    // (3)两位 其他
    int dp(string s, int loc) {
        if (loc == s.size()) return 1;
        if (loc < s.size()-1) 
            if (s[loc] == '1' || s[loc] == '2' && s[loc+1] >= '0' && s[loc+1] <= '5')  // 判断是否满足(2)
                return dp(s, loc+1) + dp(s, loc+2);  // 满足(2)
        return dp(s, loc+1);  // 满足(1)        
    }

    int translateNum(int num) {
        return dp(to_string(num), 0);
    }
};
```

## 剑指 Offer 47. 礼物的最大价值

在一个 m*n 的棋盘的每一格都放有一个礼物，每个礼物都有一定的价值（价值大于 0）。你可以从棋盘的左上角开始拿格子里的礼物，并每次向右或者向下移动一格、直到到达棋盘的右下角。给定一个棋盘及其上面的礼物的价值，请计算你最多能拿到多少价值的礼物？

 

示例 1:

输入: 
[
  [1,3,1],
  [1,5,1],
  [4,2,1]
]
输出: 12
解释: 路径 1→3→5→2→1 可以拿到最多价值的礼物


提示：

0 < grid.length <= 200
0 < grid[0].length <= 200

​    **动态规划，每格的礼物最大值等于他自己+他左面和上面里礼物最大值最大的：**

```c++
class Solution {
public:
    int maxValue(vector<vector<int>>& grid) {
        if (grid.empty()) return 0;
        for (int i = 0; i < grid.size(); ++i) {
            for (int j = 0; j < grid[i].size(); ++j) {
                if (i == 0) {  // 第一行礼物，只能从左边加礼物
                    if (j > 0) 
                        grid[i][j] += grid[i][j-1];
                    continue;  // 继续j循环，节约时间
                }
                if (j == 0) {  // 其他行的第一列，只能从上面加礼物
                    grid[i][j] += grid[i-1][j];
                } else {  // 取上面和左边最大的礼物值相加
                    grid[i][j] += grid[i][j-1]>grid[i-1][j]?grid[i][j-1]:grid[i-1][j];
                }
            }
        }
        return grid[grid.size()-1][grid[0].size()-1];
    }
};
```

## 剑指 Offer 48. 最长不含重复字符的子字符串

请从字符串中找出一个最长的不包含重复字符的子字符串，计算该最长子字符串的长度。

示例 1:

输入: "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
示例 2:

输入: "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
示例 3:

输入: "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
     请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。


提示：

s.length <= 40000

​    **方法1：vector（时间复杂度高）**

```c++
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        if (s == "")
            return 0;
        int res = 1;
        vector<char> v;  // vector维护一个不重复的字符数组
        for (int i = 0; i < s.size(); ++i){
            vector<char>::iterator it = find(v.begin(), v.end(), s[i]);  // 这里慢一点
            if (it != v.end()){  // 出现重复
                if (v.size() > res)
                    res = v.size();
                if (it == v.begin())
                    v.erase(v.begin());
                else
                    v.erase(v.begin(), ++it);
            }
            v.push_back(s[i]);
        }
        if (v.size() > res)
            res = v.size();
        return res;
    }
};
```

​    **方法二：滑动窗口**

```c++
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        // 哈希集合，记录每个字符是否出现过
        unordered_set<char> occ;
        int n = s.size();
        // 右指针，初始值为 -1，相当于我们在字符串的左边界的左侧，还没有开始移动
        int rk = -1, ret = 0;
        // 枚举左指针的位置，初始值隐性地表示为 -1
        for (int i = 0; i < n; ++i) {
            if (i != 0) {
                // 左指针向右移动一格，移除一个字符
                occ.erase(s[i - 1]);
            }
            while (rk + 1 < n && !occ.count(s[rk + 1])) {
                // 不断地移动右指针
                occ.insert(s[rk + 1]);
                ++rk;
            }
            // 第 i 到 rk 个字符是一个极长的无重复字符子串
            ret = max(ret, rk - i + 1);
        }
        return ret;
    }
}
```

## 剑指 Offer 49. 丑数

我们把只包含质因子 2、3 和 5 的数称作丑数（Ugly Number）。求按从小到大的顺序的第 n 个丑数。

 

示例:

输入: n = 10
输出: 12
解释: 1, 2, 3, 4, 5, 6, 8, 9, 10, 12 是前 10 个丑数。
说明:  

1 是丑数。
n 不超过1690。

```c++
class Solution {
public:
    int nthUglyNumber(int n) {
        vector<int> vec(n+1);
        vec[1] = 1;  // 1是第一个丑数
        // ugly表示三个丑数指针
        int ugly2 = 1, ugly3 = 1, ugly5 = 1;  
        for (int i = 2; i <= n; ++i) {
            // 算当前指针指向的丑数乘对应的质因数
            int num2 = vec[ugly2]*2, num3 = vec[ugly3]*3, num5 = vec[ugly5]*5;
            // 寻找三个中最小的丑数
            vec[i] = min(min(num2, num3), num5);
            // 判断为哪个丑数成上来的，将那个丑数指针+1
            if (vec[i] == num2) ugly2++;
            if (vec[i] == num3) ugly3++;
            if (vec[i] == num5) ugly5++;        
        }
        return vec[n];
    }
};
```

## 剑指 Offer 50. 第一个只出现一次的字符

在字符串 s 中找出第一个只出现一次的字符。如果没有，返回一个单空格。 s 只包含小写字母。

示例 1:

输入：s = "abaccdeff"
输出：'b'
示例 2:

输入：s = "" 
输出：' '


限制：

0 <= s 的长度 <= 50000

​    **哈希表**

```c++
class Solution {
public:
    char firstUniqChar(string s) {
        unordered_map<char, int> u_map;
        for (char c : s)  u_map[c]++;
        for (char c : s)  
            if (u_map[c] == 1)  return c;
        return ' ';
    }
};
```

## 剑指 Offer 51. 数组中的逆序对

在数组中的两个数字，如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组，求出这个数组中的逆序对的总数。

示例 1:

输入: [7,5,6,4]
输出: 5


限制：

0 <= 数组长度 <= 50000

​    **采取归并排序，右组的数位置一定位于左组，所以先分组在进行排序：**

```c++
class Solution {
public:
    // 归并排序
    int mergeSort(vector<int>& nums, vector<int>& tmp, int l, int r){
        if (l >= r) return 0;
        int mid = (l+r) / 2;
        // 先归并，为了使左右的数组都排好序，注意l与r的取值
        int res = mergeSort(nums, tmp, l, mid) + mergeSort(nums, tmp, mid+1, r); 
        int i = l, j = mid + 1, loc = l;  // i左侧 j右侧
        while (i <= mid && j <= r) {
            // 由于每次归并后都将当时的按照大小排序的tmp赋值给nums，所以都是从小到大，所以需要将nums[i]与nums[j]中小的放在临时数组tmp前面，并且根据当前j-(mid+1)判断右侧有几个数比他小，也就是有几个逆序对，加入res中
            if (nums[i] <= nums[j]) {  
                tmp[loc] = nums[i];  // 将当前的数存入tmp中
                i++;
                res += (j - (mid + 1));  // 统计右侧比nums[i]小的数，加入res
            }else {  // nums[i] > nums[j],j++，表示右侧中当前j及以前的数都比nums[i]小
                tmp[loc] = nums[j];  
                j++;
            }
            loc++;  // tmp向右移一个位置
        }
        // 其实下面两个for只会执行一个
        for (int k = i; k <= mid; ++k) {  // 遍历i，若i没到头说明当前i到mid之间的数比右侧的数都大
            tmp[loc++] = nums[k];
            res += (j - (mid+1));
        }
        for (int k = j; k <= r; ++k) {  // 遍历j，若j没到头说明当前j到r之间的数比左侧的数都大
            tmp[loc++] = nums[k];
        }
        // 将tmp中排好序的l到r复制到原数组中，为了之后排序用
        copy(tmp.begin() + l, tmp.begin() + r + 1, nums.begin() + l);  
        return res;
    }

    int reversePairs(vector<int>& nums) {
        vector<int> tmp(nums.size());
        return mergeSort(nums, tmp, 0, nums.size()-1);
    }
};
```

## 剑指 Offer 52. 两个链表的第一个公共节点

入两个链表，找出它们的第一个公共节点。

如下面的两个链表：

<img src="C:\Users\zhuzhibo\AppData\Roaming\Typora\typora-user-images\image-20230303101421264.png" alt="image-20230303101421264" style="zoom:67%;" />

在节点 c1 开始相交。

示例 1：

<img src="C:\Users\zhuzhibo\AppData\Roaming\Typora\typora-user-images\image-20230303101434597.png" alt="image-20230303101434597" style="zoom:67%;" />

输入：intersectVal = 8, listA = [4,1,8,4,5], listB = [5,0,1,8,4,5], skipA = 2, skipB = 3
输出：Reference of the node with value = 8
输入解释：相交节点的值为 8 （注意，如果两个列表相交则不能为 0）。从各自的表头开始算起，链表 A 为 [4,1,8,4,5]，链表 B 为 [5,0,1,8,4,5]。在 A 中，相交节点前有 2 个节点；在 B 中，相交节点前有 3 个节点。

​    **遍历链表**

```c++
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        if (headB == NULL || headA == NULL) return NULL;
        ListNode *a = headA, *b = headB;
        while (a != b) {
            a = a == nullptr ? headB : a->next;
            b = b == nullptr ? headA : b->next;
        }
        return a;
    }
};
```

## 剑指 Offer 53 - I. 在排序数组中查找数字 I

统计一个数字在排序数组中出现的次数。

示例 1:

输入: nums = [5,7,7,8,8,10], target = 8
输出: 2
示例 2:

输入: nums = [5,7,7,8,8,10], target = 6
输出: 0


提示：

0 <= nums.length <= 105
-109 <= nums[i] <= 109
nums 是一个非递减数组
-109 <= target <= 109

```c++
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int ret = 0;
        for (int i = 0; i < nums.size(); ++i) {
            if (nums[i] > target) return 0;  // 优化，如果找到大于target的数说明后面都不可能等于target，返回0
            if (nums[i] == target) {
                while (i < nums.size() && nums[i] == target) {
                    ret++; i++;
                }
                return ret;
            } 
        }
        // 能走完循环说明target比数组的数都大
        return 0; 
    }
};
```

## 剑指 Offer 53 - II. 0～n-1中缺失的数字

一个长度为n-1的递增排序数组中的所有数字都是唯一的，并且每个数字都在范围0～n-1之内。在范围0～n-1内的n个数字中有且只有一个数字不在该数组中，请找出这个数字。

示例 1:

输入: [0,1,3]
输出: 2
示例 2:

输入: [0,1,2,3,4,5,6,7,9]
输出: 8


限制：

1 <= 数组长度 <= 10000

​    **二分法**

```c++
class Solution {
public:
    int missingNumber(vector<int>& nums) {
        // 如果数组长度与最后一个数相等，说明缺少的就是最后一个数
        if (nums[nums.size()-1] == nums.size()-1) return nums.size();
        int l = 0, r = nums.size()-1;
        // 二分寻找缺少的数
        while (l < r) {
            int mid = (l+r)/2;
            if (nums[mid] != mid) r = mid;
            else l = mid+1;
        }
        return l;
    }
};
```

## 剑指 Offer 54. 二叉搜索树的第k大节点

给定一棵二叉搜索树，请找出其中第 k 大的节点的值。

示例 1:

输入: root = [3,1,4,null,2], k = 1
   3
  / \
 1   4
  \
   2
输出: 4
示例 2:

输入: root = [5,3,6,2,4,null,null,1], k = 3
       5
      / \
     3   6
    / \
   2   4
  /
 1
输出: 4


限制：

1 ≤ k ≤ 二叉搜索树元素个数

​    **正向中序遍历**

```c++
class Solution {
public:
    int kthLargest(TreeNode* root, int k) {
        vector<int> ret;
        midOrder(root, ret);  // 中序遍历BST
        return ret[ret.size()-k];  // 返回第k大的数
    }
    void midOrder(TreeNode* root, vector<int>& ret) {
        if (root == nullptr) return;
        midOrder(root->left, ret);
        ret.push_back(root->val);  // 存储值
        midOrder(root->right, ret);
    }
};
```

​    **反向中序遍历**

```c++
class Solution {
public:
    int kthLargest(TreeNode* root, int k) {
        vector<int> ret;
        reverseMidOrder(root, ret, k);  // 反向中序遍历BST
        return ret[ret.size()-1];  // 返回第k大的数，即数组中最后的数
    }
    void reverseMidOrder(TreeNode* root, vector<int>& ret, int k) {
        if (root == nullptr) return;
        reverseMidOrder(root->right, ret, k);  // 右子树
        if (ret.size() == k) return;  // 如果等于k，退出遍历，一定要在根节点之前完成
        ret.push_back(root->val);  // 存储根节点
        reverseMidOrder(root->left, ret, k);  // 左子树
    }
};
```

## 剑指 Offer 55 - I. 二叉树的深度

输入一棵二叉树的根节点，求该树的深度。从根节点到叶节点依次经过的节点（含根、叶节点）形成树的一条路径，最长路径的长度为树的深度。

例如：

给定二叉树 [3,9,20,null,null,15,7]，

​    3

   / \
  9  20
    /  \
   15   7
返回它的最大深度 3 。

提示：

节点总数 <= 10000

```c++
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if (root == nullptr) return 0;
        return max(maxDepth(root->left), maxDepth(root->right))+1;
    }
};
```

## 剑指 Offer 55 - II. 平衡二叉树

输入一棵二叉树的根节点，判断该树是不是平衡二叉树。如果某二叉树中任意节点的左右子树的深度相差不超过1，那么它就是一棵平衡二叉树。

 

示例 1:

给定二叉树 [3,9,20,null,null,15,7]

   3

   / \
  9  20
    /  \
   15   7
返回 true 。

示例 2:

给定二叉树 [1,2,2,3,3,null,null,4,4]

​        1
​       / \
​     2   2
​    / \

   3   3
  / \
 4   4
返回 false 。

限制：

0 <= 树的结点个数 <= 10000

```c++
class Solution {
public:
    bool ret = true;
    bool isBalanced(TreeNode* root) {
        depth(root);
        return ret;
    }
    // 根据左右子树高度差判断
    int depth(TreeNode* root) {
        if (root == nullptr) return 0;
        int lmax = depth(root->left);
        int rmax = depth(root->right);
        if (lmax-rmax > 1 || rmax - lmax > 1) ret = false;
        return max(lmax, rmax)+1;
    }
};
```

## 剑指 Offer 56 - I. 数组中数字出现的次数

一个整型数组 nums 里除两个数字之外，其他数字都出现了两次。请写程序找出这两个只出现一次的数字。要求时间复杂度是O(n)，空间复杂度是O(1)。

示例 1：

输入：nums = [4,1,4,6]
输出：[1,6] 或 [6,1]
示例 2：

输入：nums = [1,2,10,4,1,4,3,3]
输出：[2,10] 或 [10,2]


限制：

2 <= nums.length <= 10000

​    **运用异或的原理：**

```c++
class Solution {
public:
    vector<int> singleNumbers(vector<int>& nums) {
        // 位数相同为0，不同为1；数字0与任何数字异或等于它本身，两个相同的数字异或为0
        int a = 0;
        int b = 0;
        // 所以可以得出：数组中所有的数异或结果=两个不同的数字异或的结果
        // (a^c^c^d^d^b) = (a^b)
        for(auto x : nums) b ^= x;
        int k = 0;
        // 找到满足第k位为1的位数，因为第一个数的k位为1
        while(!(b >> k & 1)) k++;
        // 循环找出第一个数，并且是两个数中的最小数
        for(auto x : nums){
            // k位为1的数有很多，但是两个相同的数异或等于0，所以最后异或得出的一定是那个只出现一次的数
            if(x >> k & 1) a ^= x;
        }
        // 第二个数就是异或和与第一个数做异或操作，相当于(a^b)^a=b
        return {a, a ^ b};
    }
};
```

## 剑指 Offer 56 - II. 数组中数字出现的次数 II

在一个数组 nums 中除一个数字只出现一次之外，其他数字都出现了三次。请找出那个只出现一次的数字。

示例 1：

输入：nums = [3,4,3,3]
输出：4
示例 2：

输入：nums = [9,1,7,9,7,9,7]
输出：1


限制：

1 <= nums.length <= 10000
1 <= nums[i] < 2^31

​    **位运算，因为除了一个数字外所有数字都出现3次，遍历所有书，统计32位每一位的数字和，求余位3的那位就是所求数字包含的位：**

```c++
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        vector<int> vec(32);  // 位数组
        int ret=0;
        for (int i = 0; i < nums.size(); ++i)  // 统计每个数
            for (int j = 0; j < 32; ++j)  // 统计每一位
                vec[j] += (nums[i]>>j&1)==1 ? 1 : 0;
        for (int i = 0; i < 32; ++i) 
            if (vec[i]%3 == 1) ret |= (1<<i);  // 如果求余为1，ret向左移i位并与ret做或运算           
        return ret;
    }
};
```

## 剑指 Offer 57. 和为s的两个数字

输入一个递增排序的数组和一个数字s，在数组中查找两个数，使得它们的和正好是s。如果有多对数字的和等于s，则输出任意一对即可。

示例 1：

输入：nums = [2,7,11,15], target = 9
输出：[2,7] 或者 [7,2]
示例 2：

输入：nums = [10,26,30,31,47,60], target = 40
输出：[10,30] 或者 [30,10]


限制：

1 <= nums.length <= 10^5
1 <= nums[i] <= 10^6

​    **双指针**

```c++
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        int left=0, right=nums.size()-1;  // 双指针
        while (left < right) {
            if (nums[left] + nums[right] == target) return {nums[left], nums[right]}; 
            else if (nums[left] + nums[right] > target) right--;  // 大于 右指针--
            else left++;  // 小于 左指针++
        }
        return {};
    }
};
```

## 剑指 Offer 57 - II. 和为s的连续正数序列

输入一个正整数 target ，输出所有和为 target 的连续正整数序列（至少含有两个数）。

序列内的数字由小到大排列，不同序列按照首个数字从小到大排列。

示例 1：

输入：target = 9
输出：[[2,3,4],[4,5]]
示例 2：

输入：target = 15
输出：[[1,2,3,4,5],[4,5,6],[7,8]]


限制：

1 <= target <= 10^5

​    **依然双指针**

```c++
class Solution {
public:
    vector<vector<int>> findContinuousSequence(int target) {
        vector<vector<int>> ret;
        vector<int> vec;
        for (int l = 1, r = 2; l < r;) {  // l，r双指针
            int sum = (l+r)*(r-l+1)/2;
            if (sum == target) {
                vec.clear();
                for (int i = l; i <= r; ++i) vec.emplace_back(i);
                ret.push_back(vec);
                l++;
            } else if (sum < target){
                r++;
            } else l++;
        }
        return ret;
    }
};
```

## 剑指 Offer 58 - I. 翻转单词顺序

输入一个英文句子，翻转句子中单词的顺序，但单词内字符的顺序不变。为简单起见，标点符号和普通字母一样处理。例如输入字符串"I am a student. "，则输出"student. a am I"。

示例 1：

输入: "the sky is blue"
输出: "blue is sky the"
示例 2：

输入: "  hello world!  "
输出: "world! hello"
解释: 输入字符串可以在前面或者后面包含多余的空格，但是反转后的字符不能包括。
示例 3：

输入: "a good   example"
输出: "example good a"
解释: 如果两个单词间有多余的空格，将反转后单词间的空格减少到只含一个。


说明：

无空格字符构成一个单词。
输入字符串可以在前面或者后面包含多余的空格，但是反转后的字符不能包括。
如果两个单词间有多余的空格，将反转后单词间的空格减少到只含一个。

​    **常规方法，用统计单词，然后出栈**

```c++
class Solution {
public:
    string reverseWords(string s) {
        stack<string> stk;  // 字符串栈
        string res;
        string temp;
        for (int i = 0; i < s.size(); ++i){
            if (s[i] != ' ') temp += s[i];  // 没找到空格
            else{  // 找完了整个单词
                if (temp != ""){
                    stk.push(temp);
                    temp = "";
                }
            }
        }
        if (temp != "") stk.push(temp);  // 看一下temp中还有没有  
        while (!stk.empty()){  // 放入res
            res += stk.top();
            stk.pop();
            if (!stk.empty()) res += ' ';  // 非最后的字符串，加空格
        }
        return res;
    }
};

```

​    **魔幻方法（由于string执行效率低，所以不推荐）：运用字符串流：**

```c++
class Solution {
public:
    string reverseWords(string s) {
        stringstream oss(s);  // 字符串流
        string ret = "", tmp;
        while (oss>>tmp) {  // 输出到ret中
            ret = tmp+' '+ ret;
        }
        ret.pop_back();  // 把最后一个空格删掉
        return ret;
    }
};
```

## 剑指 Offer 58 - II. 左旋转字符串

字符串的左旋转操作是把字符串前面的若干个字符转移到字符串的尾部。请定义一个函数实现字符串左旋转操作的功能。比如，输入字符串"abcdefg"和数字2，该函数将返回左旋转两位得到的结果"cdefgab"。

示例 1：

输入: s = "abcdefg", k = 2
输出: "cdefgab"
示例 2：

输入: s = "lrloseumgh", k = 6
输出: "umghlrlose"


限制：

1 <= k < s.length <= 10000

```c++
class Solution {
public:
    string reverseLeftWords(string s, int n) {
        return s.substr(n, s.size()-n) + s.substr(0, n);
    }
};
```

## 剑指 Offer 59 - I. 滑动窗口的最大值

给定一个数组 nums 和滑动窗口的大小 k，请找出所有滑动窗口里的最大值。

示例:

输入: nums = [1,3,-1,-3,5,3,6,7], 和 k = 3
输出: [3,3,5,5,6,7] 
解释: 

  滑动窗口的位置          最大值

 [1  3  -1] -3  5  3  6  7      3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7


​    **采用双端队列**

```c++
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        vector<int> ret; 
        deque<int> q;  // 双端队列
        for (int i = 0; i < k; ++i) {
            // 如果nums[i]的值大于等于nums[栈尾元素]，证明新加入的元素比这个元素大
            // 循环直到找到比这个数大的一个数或者空栈
            while (!q.empty() && nums[i] >= nums[q.back()]) 
                q.pop_back(); 
            q.push_back(i);  // 入栈
        }
        ret.push_back(nums[q.front()]);
        for (int i = k; i < nums.size(); ++i) {
            // 同上
            while (!q.empty() && nums[i] >= nums[q.back()]) q.pop_back();
            q.push_back(i);
            // q.front() <= i-k表示nums[i]加入后新的k+1大小滑动窗口的最大值与原来k大小的相同，q.front()在新元素加入滑动窗口后位置离开了窗口区间
            while (q.front() <= i-k) q.pop_front();
            // nums[队头元素]就是当前滑动窗口最大值
            ret.push_back(nums[q.front()]);
        }
        return ret;
    }
};
```

## 剑指 Offer 60. n个骰子的点数

把n个骰子扔在地上，所有骰子朝上一面的点数之和为s。输入n，打印出s的所有可能的值出现的概率。

你需要用一个浮点数数组返回答案，其中第 i 个元素代表这 n 个骰子所能掷出的点数集合中第 i 小的那个的概率。

示例 1:

输入: 1
输出: [0.16667,0.16667,0.16667,0.16667,0.16667,0.16667]
示例 2:

输入: 2
输出: [0.02778,0.05556,0.08333,0.11111,0.13889,0.16667,0.13889,0.11111,0.08333,0.05556,0.02778]


限制：

1 <= n <= 11

​     **动态规划**

```c++
class Solution {
public:
    vector<double> dicesProbability(int n) {
        vector<double> dp(6, 1.0 / 6.0);
        for (int i = 2; i <= n; i++) {  // 扔第i个筛子
            vector<double> tmp(5 * i + 1, 0);  // 临时数组大小为5*i-1
            for (int j = 0; j < dp.size(); j++) {  // 从第一个数开始算
                for (int k = 0; k < 6; k++) {  // k表示点数
                    tmp[j + k] += dp[j] / 6.0;
                }
            }
            dp = tmp;  // 赋值回给dp
        }
        return dp;
    }
};
```

## 剑指 Offer 62. 圆圈中最后剩下的数字

0,1,···,n-1这n个数字排成一个圆圈，从数字0开始，每次从这个圆圈里删除第m个数字（删除后从下一个数字开始计数）。求出这个圆圈里剩下的最后一个数字。

例如，0、1、2、3、4这5个数字组成一个圆圈，从数字0开始每次删除第3个数字，则删除的前4个数字依次是2、0、4、1，因此最后剩下的数字是3。

示例 1：

输入: n = 5, m = 3
输出: 3
示例 2：

输入: n = 10, m = 17
输出: 2


限制：

1 <= n <= 10^5
1 <= m <= 10^6

​    **约瑟夫环**

```c++
class Solution {
public:
    int lastRemaining(int n, int m) {
        int ret = 0;  // 最后一个元素索引一定为0
        for (int i = 2; i <= n; ++i)  // 最后一轮剩下两个人，反推
            ret = (ret+m)%i;
        return ret;
    }
};
```

## 剑指 Offer 63. 股票的最大利润

假设把某股票的价格按照时间先后顺序存储在数组中，请问买卖该股票一次可能获得的最大利润是多少？

示例 1：

输入: [7,1,5,3,6,4]
输出: 5
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格。
示例 2:

输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。


限制：

0 <= 数组长度 <= 10^5

​    **维护一个随着遍历改变的最小数minprice**

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        if (prices.size() < 2) return 0;
        int ret = 0, minprice = prices[0];  // minprice最小数
        for (int i = 1; i < prices.size(); ++i) {
            if (prices[i] <= minprice){
                minprice = prices[i];
            }else {
                if (ret < prices[i]-minprice)  // 判断是否比当前差值是否比ret大
                    ret = prices[i]-minprice;
            }
        }
        return ret;
    }
};
```

## 剑指 Offer 64. 求1+2+…+n

求 1+2+...+n ，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。

示例 1：

输入: n = 3
输出: 6
示例 2：

输入: n = 9
输出: 45


限制：

1 <= n <= 10000

​    **方法1：用&&替代判断**

```c++
class Solution {
public:
    int sumNums(int n) {
        n != 1 && (n += sumNums(n-1));
        return n;
    }
};
```

​    **方法2：奇怪的方法**

```c++
class Solution {
public:
    int sumNums(int n) {
        bool arr[n][n+1];
        return sizeof(arr)>>1;
    }
};
```

## 剑指 Offer 65. 不用加减乘除做加法

写一个函数，求两个整数之和，要求在函数体内不得使用 “+”、“-”、“*”、“/” 四则运算符号。

示例:

输入: a = 1, b = 1
输出: 2


提示：

a, b 均可能是负数或 0
结果不会溢出 32 位整数

​    **位运算**

```c++
class Solution {
public:
    int add(int a, int b) {
        while (b != 0) {
            int tmp = a^b;  // 异或
            b = ((unsigned int)(a&b)<<1);  // 与运算左移一位
            a = tmp;
        }
        return a;
    }
};
```

## 剑指 Offer 66. 构建乘积数组

给定一个数组 A[0,1,…,n-1]，请构建一个数组 B[0,1,…,n-1]，其中 B[i] 的值是数组 A 中除了下标 i 以外的元素的积, 即 B[i]=A[0]×A[1]×…×A[i-1]×A[i+1]×…×A[n-1]。不能使用除法。

示例:

输入: [1,2,3,4,5]
输出: [120,60,40,30,24]


提示：

所有元素乘积之和不会溢出 32 位整数
a.length <= 100000

```c++
class Solution {
public:
    vector<int> constructArr(vector<int>& a) {
        vector<int> ret(a.size());
        for (int i = 0, tmp = 1; i < a.size(); ++i) {
            ret[i] = tmp;  // 再乘左边的数(不包括自己)
            tmp *= a[i];
        }
        for (int i = a.size() - 1, tmp = 1; i >= 0; i--) {
            ret[i] *= tmp;  // 再乘右边的数(不包括自己)
            tmp *= a[i];
        }
        return ret;
    }
};
```

## 剑指 Offer 68 - I. 二叉搜索树的最近公共祖先

给定一个二叉搜索树, 找到该树中两个指定节点的最近公共祖先。

百度百科中最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。”

例如，给定如下二叉搜索树:  root = [6,2,8,0,4,7,9,null,null,3,5]

<img src="C:\Users\zhuzhibo\AppData\Roaming\Typora\typora-user-images\image-20230308081401219.png" alt="image-20230308081401219" style="zoom:67%;" />

示例 1:

输入: root = [6,2,8,0,4,7,9,null,null,3,5], p = 2, q = 8
输出: 6 
解释: 节点 2 和节点 8 的最近公共祖先是 6。
示例 2:

输入: root = [6,2,8,0,4,7,9,null,null,3,5], p = 2, q = 4
输出: 2
解释: 节点 2 和节点 4 的最近公共祖先是 2, 因为根据定义最近公共祖先节点可以为节点本身。


说明:

所有节点的值都是唯一的。
p、q 为不同节点且均存在于给定的二叉搜索树中。

​    **因为是二叉搜索树，所以根据值就可以判断是否为祖先**

```c++
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if (root == nullptr) return root;
        // 根节点大于p和q，说明祖先节点在左子树中
        if (root->val > p->val && root->val > q->val) return lowestCommonAncestor(root->left, p, q);
        // 根节点小于p和q，说明祖先节点在右子树中
        if (root->val < p->val && root->val < q->val) return lowestCommonAncestor(root->right, p, q);
        // 不是以上情况就说明要不q和p一个左一个右，要不就是根节点就是q或p，返回根节点
        return root;
    }
};
```

## 剑指 Offer 68 - II. 二叉树的最近公共祖先

给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

百度百科中最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。”

例如，给定如下二叉树:  root = [3,5,1,6,2,0,8,null,null,7,4]

示例 1:

输入: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1
输出: 3
解释: 节点 5 和节点 1 的最近公共祖先是节点 3。
示例 2:

输入: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 4
输出: 5
解释: 节点 5 和节点 4 的最近公共祖先是节点 5。因为根据定义最近公共祖先节点可以为节点本身。


说明:

所有节点的值都是唯一的。
p、q 为不同节点且均存在于给定的二叉树中。

​    **深度优先搜索**

```c++
class Solution {
public:
    TreeNode *ret;
    bool dfs(TreeNode* root, TreeNode* p, TreeNode* q) {
        if (root == nullptr) return false;
        bool lson = dfs(root->left, p, q);
        bool rson = dfs(root->right, p, q);
        // 如果左右子树都true或root本身就是q或p且左右子树有一个为true
        if ((lson && rson) || ((root->val == p->val || root->val == q->val) && (lson || rson))) {
            // root就为公共祖先
            ret = root;
        } 
        return lson || rson || (root->val == p->val || root->val == q->val);
    }

    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        dfs(root, p, q);
        return ret;
    }
};
```

## 大数加法

​    以字符串形式读入两个数字，编写函数计算他们的和，以字符串返回

```c++
string bigAdd(string s, string t)
{
    // 长的设置为s，短的设置为t
	if (s.size() < t.size())
		s.swap(t);
	int len1 = s.size(), len2 = t.size();
	int num1, num2, flag = 0, sum;
	while (len1 > 0) {
		num1 = s[len1-1]-'0';
		if (len2 > 0)
			num2 = t[len2-1]-'0';
		else num2 = 0;
		sum = num1+num2+flag;
		s[len1-1] = sum%10 + '0';
		flag = sum/10;
		len1--;
		len2--;
	}
	if (flag == 1)
		s = '1'+s;
	return s;
}
```

