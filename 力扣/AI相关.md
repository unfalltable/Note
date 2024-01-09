### 最大矩形

```java
/*
	题目：
        给定一个仅包含 0 和 1 、
        大小为 rows x cols 的二维二进制矩阵，
        找出只包含 1 的最大矩形，并返回其面积
	思路：
		- 前缀和 + 单调栈
			- 从上往下计算i到0层中列不为0的
*/
//python
class Solution(object):
    def maximalRectangle(self, mat):
		n, m, ans = len(mat), len(mat[0]), 0
        psum = [[0] * (m + 10) for _ in range(n + 10)]
        for i in range(1, n + 1):
            for j in range(1, m + 1):
                psum[i][j] = 0 if mat[i - 1][j - 1] == '0' else psum[i - 1][j] + 1
        stk = [0] * (m + 10)
        he, ta = 0, 0
        for i in range(1, n + 1):
            cur = psum[i]
            l, r = [0] * (m + 10), [m + 1] * (m + 10)
            he = ta = 0
            for j in range(1, m + 1):
                while he < ta and cur[stk[ta - 1]] > cur[j]:
                    ta -= 1
                    r[stk[ta]] = j
                stk[ta] = j
                ta += 1
            he = ta = 0
            for j in range(m, 0, -1):
                while he < ta and cur[stk[ta - 1]] > cur[j]:
                    ta -= 1
                    l[stk[ta]] = j
                stk[ta] = j
                ta += 1
            for j in range(1, m + 1):
                ans = max(ans, cur[j] * (r[j] - l[j] - 1))
        return ans
//java
class Solution {
    public int maximalRectangle(char[][] mat) {
        int n = mat.length, m = mat[0].length, ans = 0;
        int[][] sum = new int[n + 10][m + 10];
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                sum[i][j] = mat[i - 1][j - 1] == '0' ? 0 : sum[i - 1][j] + 1;
            }
        }
        int[] l = new int[m + 10], r = new int[m + 10];
        for (int i = 1; i <= n; i++) {
            int[] cur = sum[i];
            Arrays.fill(l, 0); Arrays.fill(r, m + 1);
            Deque<Integer> d = new ArrayDeque<>();
            for (int j = 1; j <= m; j++) {
                while (!d.isEmpty() && cur[d.peekLast()] > cur[j]) r[d.pollLast()] = j;
                d.addLast(j);
            }
            d.clear();
            for (int j = m; j >= 1; j--) {
                while (!d.isEmpty() && cur[d.peekLast()] > cur[j]) l[d.pollLast()] = j;
                d.addLast(j);
            }
            for (int j = 1; j <= m; j++) ans = Math.max(ans, cur[j] * (r[j] - l[j] - 1));
        }
        return ans;
    }
}                
```



