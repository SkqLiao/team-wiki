# ABC G题汇总

## ABC 270 G - Sequence in mod P

**题意：**序列 $f$ 满足 $f_0=s,f_{i}=a\times f_{i-1}+b(i\geq 1)\pmod p$。给定 $a,b,s,p,g$，求最小的 $n$ 满足 $f_n=g$。

**限制：** $0\leq a,b,s,p,g\leq 10^9$

**链接：**https://atcoder.jp/contests/abc270/tasks/abc270_g

**题解：**BSGS

用 $h_i(x)$ 表示当第 $0$ 项为 $x$ 时，序列第 $i$ 项的值，则 $f_{n=pT-q}=g$ 等价于 $h_{pT}(s)=h_q(g)$。

变换 $f$ 的递推式，设 $h_i(x)=a_i\times x+b_i$，则 $a_i=a_{i-1}\times a,b_{i}=b_{i-1}\times a+b$。

令 $T=\lceil\sqrt{p}\rceil$ ，预处理 $h_q(g):h_1(g),h_2(g),\cdots,h_T(g)$，然后哈希表中依次寻找 $h_{pT}(s):h_T(s),h_{2T}(s),\cdots,h_{T^2}(s)$，最终得到 $n=pT-q$。

已知 $h_{iT}(s)$，则 $h_{(i+1)T}(s)=h_{T}(h_{iT}(s))=a_T\times h_{iT}(s)+b_{T}$，而 $a_T,b_T$ 可以在预处理 $h_{q}(g)$ 时得到，因此 $h$ 的每一项都可以 $O(1)$ 计算。

单组查询的复杂度为 $O(\sqrt{p})$。

**代码**：https://atcoder.jp/contests/abc270/submissions/35189357

## ABC 239 G - Builder Takahashi

**题意：**给定 $n$ 个点 $m$ 条边的无向连通图，删除点 $i$ 的花费为 $c_i$。可以选择 $2\sim n-1$ 号点中的若干个点删除（并删除与其相连的所有边），使得 $1$ 号点和 $n$ 号点不连通。求最小花费。

**限制**：$n\leq 100$

**链接：**https://atcoder.jp/contests/abc239/tasks/abc239_g

**题解：**最大流

拆点并建图，连边 $(i, i', c_i)$。对于每条边 $(u,v)$，连边 $(u',v,\infty),(v',u,\infty)$。

最小花费为 $1'\rightarrow n$ 的最大流，被删除的点为最小割上的割边（显然这些边都形如 $i\rightarrow i'$）对应的点。

需要注意的是，最小割上的边并不等价于流量流满（flow=capcity）的边。

复杂度 $O(\text{maxflow}(n, m))$。

**代码：**https://atcoder.jp/contests/abc239/submissions/35192622

## ABC 237 G - Range Sort Query

**题意：**给定一个长为 $n$ 的排列，有 $q$ 次操作。第 $i$ 次操作为将子区间 $[l_i,r_i]$ 的元素升序/降序排序。求经过 $q$ 次操作后，元素 $x$ 的下标是多少。

**限制：**$n,q\leq 10^5$

**链接：**https://atcoder.jp/contests/abc237/tasks/abc237_g

**题解：**线段树

区间排序时，只需要知道 $\geq x$（或者 $\leq x$）的元素个数后将左右区间分别覆盖成 $0/1$ ，并更新 $x$ 的位置。

维护一棵区间求和、区间覆盖的线段树，复杂度 $O(q\log{n})$。

**代码：**https://atcoder.jp/contests/abc237/submissions/35194796

## ABC 247 G - Dream Team

**题意：**有 $n$ 个人，第 $i$ 个人属于 $a_i$ 大学，擅长学科为 $b_i$，能力为 $c_i$。求恰好选择 $1,2,\cdots$ 个人时，在满足任意两个人的大学不同且擅长学科不同的条件下，能力值之和的最大值。

**限制：**$n\leq 3\times 10^4,1\leq a_i,b_i\leq 150$

**链接：**https://atcoder.jp/contests/abc247/tasks/abc247_g

**题解：**费用流

每所大学和学科都只能被选择一次，这是经典的网络流模型。

$s\rightarrow a_i$，流量为 $1$，费用为 $0$；$b_i\rightarrow t$，流量为 $1$ ，费用为 $0$；$a_i\rightarrow b_i$，流量为 $1$，费用为 $-c_i$。

跑最小费用最大流。每增广一次，流量增加 $1$，即多选一个人，累计的总费用即为此时的能力值之和。

注意要用增广的方式跑费用流，而不是每次从 $s$ 增加一条边容量为 $1$ 的边再重新跑pushRelabel。

**代码：**https://atcoder.jp/contests/abc247/submissions/35195856

## ABC 238 G - Cubic?

**题意：**给定长为 $n$ 的序列 $a$，有 $q$ 个询问。第 $i$ 次询问给定 $l_i,r_i$，查询 $\prod\limits_{j=l_i}^{r_i}{a_j}$ 是否是立方数。

**限制：**$n,q\leq 2\times 10^5,1\leq a_i\leq 10^6$

**链接：**https://atcoder.jp/contests/abc238/tasks/abc238_g

**题解1：**莫队

对 $a_i$ 做质因数分解，不难发现至多 $8$ 个质因数（$2\times 3\times 5\times 7\times 11\times 13\times 17\times 19=9,699,690>1,000,000$）。而立方数只需要满足每个质因子的幂指数都为 $3$ 的倍数。因此可以用莫队维护区间中每个质因子的幂指数。复杂度 $O(q\sqrt{n})$。

其实莫队的部分跑的并不慢，但如果在预处理的时候暴力分解质因数，那就寄了，需要用埃氏筛预处理每个数的最小质因数，即可实现 $O(\log{a_i})$ 分解。

**题解2：**哈希

分析同上，我们需要维护的是每个质因数的幂指数。不难发现不同质因子之间互不干扰。因此可以通过哈希的方式维护每个前缀区间的 $\sum\limits_{p\in \text{prime}}{p\times (cnt[p]\bmod 3)}$ ，其中 $cnt[p]$ 为质因子 $p$ 的出现次数。若两前缀值相等，则说明该区间中所有质因子的幂指数都是 $3$ 的倍数。复杂度 $O(n\log{a_i})$。

由于本方法复杂度低，因此即使你暴力分解质因数，代码跑的也不慢。。

**代码1：**https://atcoder.jp/contests/abc238/submissions/35207997

**代码2：**https://atcoder.jp/contests/abc238/submissions/35208067

## ABC 267 G - Increasing K Times

**题意：**给定长为 $n$ 的序列 $a$，求有多少个 $1\sim n$ 的排列 $p$ ，满足恰好存在 $k$ 个 $i$ 满足 $a_{p_i}<a_{p_{i+1}}$。答案对 $998244353$ 取模。

**限制：**$n\leq 5000$

**链接：**https://atcoder.jp/contests/abc267/tasks/abc267_g

**题解：**DP

问题等价于求 $a$ 有多少种排列方式 $b$，满足存在 $k$ 个「正序对」（ $b_i<b_{i+1}$）。考虑将 $a$ 升序排序后，逐个插入空序列中，则每插入一个数，新序列的「正序对」个数要么不变，要么增加 $1$。

将 $a_i$ 插入在 $b_j,b_{j+1}$ 之间（令 $b_{-1}=b_{|b|+1}=0$），根据大小关系有两种情况：

- $a_i>b_j,b_j\geq b_{j+1}$：个数加一
- 其他：个数不变

记 $f(i,j)$  为前 $i$ 个数，组成 $j$ 个「正序对」的方案数。（记 $cnt[a_i]$ 为 $a_i$ 的当前出现次数）

- $f(i,j)\rightarrow f(i+1,j)$：插在与 $a_i$ 相等的数 / $b_j<b_{j+1}$ 之间，共 $cnt[a_i]+j+1$ 种方式
- $f(i,j)\rightarrow f(i+1,j+1)$：共 $i-j-cnt[a_i]$ 种方式

答案为 $f(n,k)$，复杂度 $O(nk)$。

**代码：**https://atcoder.jp/contests/abc267/submissions/35214949

## ABC 256 G - Black and White Stones

**题意：**将若干个黑白石子摆成每条边上有 $d+1$ 个石子的 $n$ 边形，要求每条边上的白色石子个数相同，求方案数。答案对 $998244353$ 取模。

**限制：**$n\leq 10^{14}, d\leq 10^4$

**链接：**https://atcoder.jp/contests/abc256/tasks/abc256_g

**题解：**矩阵快速幂

枚举每条边的白色石子数 $m$，设 $f^m_{0/1}(i,0/1)$，表示已经放置了 $i$ 条边，其中第 $1$ 条边的首和第 $i$ 边的尾分别为黑/白石子的方案数，则答案为 $\sum\limits_{m=0}^{d+1}{f^m_1(n,1)+f^m_0(n,0)}$。

则转移方程 $f^m_*(i)=a^m\times f^m_*(i-1)$ 为：
$$
\begin{bmatrix}
f(i, 0)\\
f(i, 1)\\
\end{bmatrix}
=
\begin{bmatrix}
{d-1\choose m-2}\ {d-1\choose m-1}\\
{d-1\choose m-1}\ {d-1\choose m}\\
\end{bmatrix}\times
\begin{bmatrix}
f(i-1, 0)\\
f(i-1, 1)\\
\end{bmatrix}
$$
这个方程可以用矩阵快速幂转移，总复杂度 $O(d\times \log{n})$。

**代码：**https://atcoder.jp/contests/abc256/submissions/35216315

## ABC 246 G - Game on Tree 3

**题意：**给定一棵树，除根节点外的任意点 $i$ 都有权值 $a_i$。Alice和Bob轮流操作，从根节点出发。在每个回合中，Alice首先选择任意一个点，将其权值置为 $0$，然后Bob从当前点移动到它的任意一个子节点中，并可以选择立刻结束游戏（若到达叶子节点，则游戏自动结束）。游戏得分是结束游戏前他所在点的权值，Alice想要最小化，Bob想要最大化。如果双方均采取最优策略，求最终得分。

**限制：**$n\leq 2\times 10^5,1\leq a_i\leq 10^9$

**链接：**https://atcoder.jp/contests/abc246/tasks/abc246_g

**题解1：**二分答案+树形DP

首先二分答案 $x$，将权值小于 $x$ 的点的权值 $b_i$ 置为 $0$，否则置为 $1$。问题变成Bob能否达到一个权值为 $1$ 的点。

设 $f(x)$ 表示如果下一步走到 $x$ 点，在此之前，Alice需要至少多少次置 $0$ 才能阻止未来Bob到达权值为 $1$ 的点。转移方程为 $f(x)=\max(-1+\sum\limits_{y\in son[y]}{f(v)},0)+b_x$，最后只需要判断 $f(1)$ 是否为 $0$。

复杂度 $O(n\log{a_i})$。

**题解2：**线段树

同样是自底向上考虑。贪心地想，当前在 $x$ 点，Alice下一回合会置 $0$ 的点应该是 $x$ 子树中（不包括 $x$）剩余点中的最大值。因此可以用线段树维护子树最大值，自底向上转移。

复杂度 $O(n\log{n})$。

**代码1：**https://atcoder.jp/contests/abc246/submissions/35227917

**代码2：**https://atcoder.jp/contests/abc246/submissions/me

## ABC 234 G - Divide a Sequence

**题意：**给定长为 $n$ 的序列 $a$，共有 $2^n$ 种方式将其划分成长度为 $m=1,2,\cdots,n$ 的非空连续子序列 $b_1,b_2,\cdots,b_m$。对于任意划分，定义其权值为 $\prod{\max(b_i)-\min(b_i)}$。求这 $2^n$ 个划分的权值之和，答案对 $998244353$ 取模。

**限制：**$n\leq 3\times 10^5$

**链接：**https://atcoder.jp/contests/abc234/tasks/abc234_g

**题解：**单调栈

$f(i)$ 表示前 $i$ 个数的所有划分的权值之和，则 $f(i)=\sum\limits_{j<i}{f(j)\times (\max\limits_{k=j+1}^{i}{a_k}-\min\limits_{k=j+1}^{i}{a_k})}$。

将后式的 $\max$ 和 $\min$ 拆开维护，显然 $[1,i-1]$ 可以划分出若干段连续区间，其区间 $\max$ / $\min$ 是相等的，且区间的从左到右 $\max/\min$ 值是递减/递增的。

因此可以用单调栈维护这些 $\max/\min$ 的区间 $f$ 值之和，弹出栈顶的同时计算出它的贡献增量。复杂度为 $O(n)$。

**代码：**https://atcoder.jp/contests/abc234/submissions/35230774

## ABC 223 G - Vertex Deletion

**题意：**给定一棵树，求有多少个点满足在删除它之后，最大匹配数不变。

**限制：**$n\leq 2\times 10^5$

**链接：**https://atcoder.jp/contests/abc223/tasks/abc223_g

**题解：**换根DP

求树的最大匹配有 $O(n)$ 的树形DP，即 $f[x][0/1]$ 表示 $x$ 子树中的最大匹配数，且 $x$ 是否与它父亲连边。

转移方程为 $f[x][1]=\sum\limits_{y\in son[x]}{f[y][0]},f[x][0]=\sum\limits_{y\in son[x]}{f[y][0]}+\max\limits_{y\in son[x]}{f[y][1]+1-f[y][0]}$。

同时记录 $x$ 的子节点中，$f[y][1]+1-f[y][0]$ 的最大值和次大值来源，记为 $from[x][0/1]$。

换根DP， $x\rightarrow y$ 时维护 $f[x][*],f[y][*],from[x][*],from[y][*]$ 的变化值即可，其他点的值不变。

复杂度 $O(n)$。

**代码：**https://atcoder.jp/contests/abc223/submissions/35242470

## ABC 230 G - GCD Permutation

**题意：**给定一个长为 $n$ 的排列 $p$，求有多少个数对 $(i,j)(i\leq j)$ 满足 $\gcd(i,j)\not=1$ 且 $\gcd(p_i,p_j)\not=1$。

**限制：**$n\leq 2\times 10^5$

**链接：**https://atcoder.jp/contests/abc230/tasks/abc230_g

**题解：**莫比乌斯反演

设 $f(a,b;i,j)$ 表示【 $i,j$ 是 $a$ 的倍数，且 $p_i,p_j$ 是 $b$ 的倍数】。

由 $\sum\limits_{d|n}\tilde\mu(d)=[n\geq 2]$，枚举位置 $i,j$ ，并枚举公因数 $a,b$，则答案为 $\sum_\limits{1\leq i\leq j\leq n}\sum\limits_{a=1}^{n}\sum\limits_{b=1}^{n}{\tilde\mu(a)\tilde\mu(b)f(a,b;i,j)}$，其中 $\mu(a)$ 为莫比乌斯函数。

记 $h(a,b)$ 为满足 $i|a$ 且 $p_i|b$ 的 $i$ 的个数，则 $\sum\limits_{1\leq i\leq j\leq n}{f(a,b;i,j)}=\frac{h(a,b)(h(a,b)+1)}{2}$，答案化简为 $\sum\limits_{a=1}^{n}\sum\limits_{b=1}^{n}\tilde\mu(a)\tilde\mu(b)\frac{h(a,b)(h(a,b)+1)}{2}$。

先枚举 $a$，无需枚举 $b$。此时满足 $i|a$ 的数为 $p_{a},p_{2a},\cdots,p_{ka}$，则 $b$ 需要是他们的因数之一且 $\tilde\mu(b)\not=0$。因此对于 $p_{ia}$ ，遍历它的所有 $\tilde\mu(d)\not=0$ 的因子 $d$，并统计出现次数。

枚举的数总个数 $\{p_{ia}\}=\sum\limits_{i=1}^{n}{\lfloor\frac{n}{i}\rfloor}=O(n\log{n})$ ，而 $\leq 2\times 10^5$ 的数的质因子个数不会超过 $7$ 个，因而 $\tilde\mu\not=0$ 的因数个数不超过 $2^7$ 个，总复杂度 $O(63n\log{n})$。

**代码：**https://atcoder.jp/contests/abc230/submissions/35244167

## ABC 236 G - Good Vertices

**题意：**$n$ 个点的有向图，在第 $0$ 秒时没有边。在第 $i(1\leq i\leq t)$ 秒，加入有向边 $u_i\rightarrow v_i$。对于任意点 $t$，求长度恰好为 $L$ 条边的路径 $1\to t$ 的最早出现时间。

**限制：**$n\leq 100, t\leq n^2,L\leq 10^9$

**链接：**https://atcoder.jp/contests/abc236/tasks/abc236_g

**题解：**矩阵快速幂

设 $f(i,j)$ 表示到达经过恰好 $i$ 条边到达 $j$ 点的所有路径中编号最大的边的最小值，答案为 $f(L,1),\cdots,f(L,n)$。

转移方程：$f(i,j)=\min\limits_{1\leq j\leq n}\{\max(f(i-1,j),W(j,i))\}$，其中 $W(j,i)$ 为边 $j\rightarrow i$ 出现的时间，若不存在置为 $\infty$。

初值 $f(0,1)=0,f(0,i)=\infty(2\leq i\leq n)$。

将 $\max$ 定义为 $+$，$\min$ 定义为 $\times$，则上式可以写成矩阵乘法的形式 $f_{n\times 1}(i)=W_{n\times n}\times f_{n\times 1}(i-1)$。

由于 $\max,\min$ 满足分配律、交换律和结合律，因此可以改变运算顺序，对这个式子做矩阵快速幂，得到 $f(L,i)=W^L(i,1)$。

复杂度 $O(n^3\log{L})$。

**代码：**https://atcoder.jp/contests/abc236/submissions/35326063

## ABC 249 G - Xor Cards

**题意：**有 $n$ 张卡片，第 $i$ 张卡片正面写着 $a_i$，背面写着 $b_i$。现在选择若干张卡片，求在满足正面的异或和不超过 $k$ 的条件下，背面的异或和的最大值。

**限制：** $n\leq 1000,0\leq a_i,b_i,k\leq 2^{30}$

**链接：**https://atcoder.jp/contests/abc249/tasks/abc249_g

**题解：**线性基

根据线性基定理，$n$ 个数的任意子集的异或和只需要 $O(\log{V})$ 个数的线性基即可表示。

对于两张卡片 $(a,b),(c,d)$，它们等价于 $(a\oplus c,b\oplus d),(c,d)$，因此可以通过这样的方式，一边构造 $a_i$ 的线性基 $la$，并把所有能用线性基表示的卡片 $(a_i,b_i)$ 转换成 $(0,b_i')$。

此时剩下若干个 $(0,b_i')$ 和一个线性基 $la$，前者可以再构造出一个对 $k$ 的限制无影响的线性基 $lb$。

此时考虑如何满足 $k$ 的限制。

枚举 $i$，用 $la$ 构造出前 $i-1$ 位与 $k$ 相同，且当前位为 $0$ 而 $k_{(i)}=1$ 的状态，此时任意取后面的位都满足限制。那么就可以将后面位的 $b_i$ 插入 $lb$ 中，并查询异或最大值。

复杂度 $O(n\log{V}+\log^3{V})$。

**代码：**https://atcoder.jp/contests/abc249/submissions/35388300

## ABC 248 G - GCD cost on the tree

**题意：**给定一棵树，点 $i$ 有点权 $a_i$。定义 $f(u,v)$ 为 $u,v$ 之间的最短路径上的点数与最短路径上所有点权值的 $\gcd$ 的乘积。求 $\sum\limits_{i=1}^{n}\sum\limits_{j=i+1}^{n}{f(i,j)}$。

**限制：**$1\leq n,a_i\leq 10^5,$

**链接：**https://atcoder.jp/contests/abc248/tasks/abc248_g

**题解：**树形DP

应该算是个暴力，也不知道复杂度对不对，不过跑的飞快。

维护 $f(i)=\{(x,y)\}$ ，表示 $i$ 的子树中的所有点，从 $i$ 出发，路径 $\gcd=x$ 的所有点的深度之和为 $y$。

那么合并 $f(i)$ 和其子节点 $f(j)$ 时，只需要分别枚举 $(x_1,y_1)\in f(i)$ 和 $(x_2,y_2)\in f(j)$，贡献为 $[y_1\times x_2+x_1\times y_2-(2\times dep(i) - 1)\times (y_1\times y_2)] \times \gcd(x_1,x_2)$。

注意到，虽然看起来 $x$ 的取值有很多，但是实际上至多只有 $d(a_i)$ 个。而根据经典老图，$\max\{d(10^5)\}=128$，因此复杂度上界是 $O(128^2\times n)$，但实际上远远跑不满。

![IMG_2672](https://raw.githubusercontent.com/SkqLiiiao/image/main/202210041448144.jpeg)

**代码：**https://atcoder.jp/contests/abc248/submissions/35390296

## ABC 225 G - X

**题意：**给定一个 $n\times m$ 的网格，每个格子有权值 $w(i,j)$。现在可以选择若干个格子画"X"，即两条对角线。最后的得分为所有画"X"的格子的权值之和- $C\times$ 需要画的对角线数量（每次画对角线可以穿越多个格子）。要求每个格子要么画"X"，要么什么都没画。求得分的最大值。

**限制：**$n,m\leq 100$

**链接：**https://atcoder.jp/contests/abc225/tasks/abc225_g

**题解：**最大流

现在的得分是两部分的差，不太好算。先假设所有格子都被画"X"获得 $\sum{w(i,j)}$ 的分数，再减去付出代价的最小值，如此得到的结果应与答案相同。

先考虑右下方向的对角线，它的代价应该是 $C\times$ 【 $(i,j)$ 画"X"，但是 $(i-1,j-1)$ 不画的数量】，同理左下方向的。

因此构图方法如下：

- $s\rightarrow (i,j)$，流量为 $w(i,j)$
- $(i,j)\rightarrow (i-1,j-1)$，流量为 $c$（存在 $(i-1,j-1)$）
- $(i,j)\rightarrow (i-1,j+1)$，流量为 $c$（存在 $(i-1,j+1)$）
- $(0,j)/(i,0)\rightarrow t$，流量为 $c$
- $(0,j)/(i,m)\rightarrow t$，流量为 $c$ 

答案为 $\sum{w(i,j)}-\text{maxflow}(s,t)$。

**代码：**https://atcoder.jp/contests/abc225/submissions/35391979

## ABC 271 G - Access Counter

**题意：**给定长为 $24$ 的字符串，$s[i]=a$ 表示每天 $i$ 点钟Alice有 $x$ 的概率访问网页，$s[i]=b$ 表示每天 $i$ 点钟Bob有 $y$ 的概率访问网页。从某天的 $0$ 点开始游戏，求第 $n$ 次访问网页恰好是Alice完成的概率。

**限制：**$n\leq 10^{18}$

**链接：**https://atcoder.jp/contests/abc271/submissions/35417575

**题解：**矩阵快速幂

设 $f(i,j,k)$ 表示第 $2^i$ 次访问网页在 $k$ 点钟，上一次访问在 $j$ 点钟的概率，转移方程为 $f(i,j,k)=\sum\limits_{l=0}^{23}{f(i-1,j,l)\times f(i-1,l,k)}$。

计算初值，假设第 $1$ 次访问网页是在 $i$ 点钟，则它需要经过 $n(n\geq 0)$天的失败，以及 $[0,i-1]$ 点钟的失败和 $i$ 点钟的成功，概率为 $g(i)=\sum_\limits{n=0}^{\infty}{q^n\times \prod\limits_{j=0}^{i-1}{(1-p_j)}\times p_i}$，其中 $q=\prod{(1-p_i)}$，$p_i$ 为 $i$ 点钟访问网页的概率。

根据等比数列求和公式，$g(i)=\frac{1}{1-q}\times\prod\limits_{j=0}^{i-1}{(1-p_j)}\times p_i$。

计算出 $g(i)$ 后，通过枚举第 $1$ 次和第 $2$ 次访问时间可以得到初值 $f(1,*,*)$。

不难发现转移方程矩阵乘法的形式，因此可以通过矩阵快速幂求第 $n$ 次访问的概率 $h(*,*)$，答案为 $\sum h(23,i)$，其中 $i$ 满足 $s[i]=a$。

复杂度 $O(24^3\log{n})$。

**代码：**https://atcoder.jp/contests/abc271/tasks/abc271_g

## ABC 235 G - Gardens

**题意：**有三种石子，分别有 $a,b,c$ 个放置在 $n$ 个有标号的盒子中。要求每个盒子至少放置一个石子，每个盒子中至多一个同种石子，而且不必放完所有石子。求方案数。

**限制：**$n \leq 5\times 10^6$

**链接：**https://atcoder.jp/contests/abc235/tasks/abc235_g

**题解：**反演

先考虑至少 $i$ 个盒子为空的方案数，$\binom{n}{i}\sum\limits_{x=0}^{\min(a,n-i)}\binom{a}{i}\sum\limits_{y=0}^{\min(b,n-i)}\binom{b}{y}\sum\limits_{z=0}^{\min(c,n-i)}\binom{c}{z}$。

则答案为恰好 $0$ 个盒子为空的方案数，容斥一下，得到 $\sum\limits_{i=0}^{n}(-1)^{i}\binom{n}{i}\sum\limits_{x=0}^{\min(a,n-i)}\binom{a}{i}\sum\limits_{y=0}^{\min(b,n-i)}\binom{b}{y}\sum\limits_{z=0}^{\min(c,n-i)}\binom{c}{z}$

用 $f_M(N)$ 表示 $\sum\limits_{i=0}^{\min(N,M)}\binom{N}{i}$，则答案化简为 $\sum\limits_{i=0}^{n}(-1)^{n-i}\binom{n}{i}f_a(i)f_b(i)f_c(i)$。

现在考虑如何从 $f_M(N)$ 快速计算出到 $f_M(N+1)$。

如果 $N+1\leq M$，根据二项式定理， $f_M(N)=\sum\limits_{i=0}^{N}{\binom{N}{i}}=2^N$，$f_M(N+1)=2^{N+1}$，得 $f_M(N+1)=2f_M(N)$。

否则 $f_M(N)=\sum\limits_{i=0}^{M}\binom{N}{i}$，$f_M(N+1)=\sum\limits_{i=0}^{M}\binom{N+1}{i}$。

根据 $\binom{N}{M}=\binom{N-1}{M}+\binom{N-1}{M-1}$，则 $2\sum\limits_{i=0}^{M}\binom{N}{i}=\sum\limits_{i=0}^{M}\binom{N+1}{i}-\binom{N}{M}$。

也可以通过下图考虑它的意义：

<img src="https://img.atcoder.jp/ghi/d93c643497867d310c6255fb673d9682.png" alt="image" style="zoom:50%;" />

从左下角出发，每次向右/上移动一格，则到达黄色点的方案数之和为 $f_M(N)$，到达蓝色点的方案数为 $f_M(N+1)$。

不难发现除了右下角的黄点无法向右移动以外，其他点都有向上/右移动两种方案，因此 $f_M(N+1)=2f_M(N)-\binom{N}{M}$。

如此每次转移就是 $O(1)$ 的了，总复杂度 $O(n)$。

**代码：**https://atcoder.jp/contests/abc235/submissions/35426630

## ABC 232 G - Modulo Shortest Path

**题意：**有 $n$ 个点的有向完全图，每个点有权值 $a_i$ 和 $b_i$。有向边 $i\rightarrow j$ 的边权为 $(a_i+b_j)\bmod m$ ，求 $1\to n$ 的最短路。

**限制：**$n\leq 2\times 10^5, 0\leq a_i,b_i\leq m,2\leq m\leq 10^9$

**链接：**https://atcoder.jp/contests/abc232/tasks/abc232_g

**题解：**建图+最短路

显然需要把边的数量降下来才能用dijkstra求最短路，因此考虑构建与原图等价的新图。

对于权值 $0\sim m-1$ 分别建一个新点，$[i]\rightarrow [(i+1)\bmod m]$ 边权为 $1$，可以想象将这 $m$ 个点排列成一个顺指针的圆环。

对于原图上的 $n$ 个点，$i\rightarrow j$ 的边权为 $(a_i+b_j)\bmod m$，考虑用这个圆环上的边替代，则可以用 $i\rightarrow [m-a_i]$ 连边权为 $0$ 的边，再连一条 $[b_j]\rightarrow j$ 边权为 $0$ 的边。此时 $i\rightarrow j$ 就可以沿着圆环走，路上的边权和仍然为 $(a_i+b_j)\bmod m$。

显然不是圆环上的所有点都有用，那些除圆环外没有入度的点可以被删除合并，这样圆环上至多留下 $2n$ 个点。

现在一共 $3n$ 个点，$4n$ 条边，可以用dijkstra $O(n\log{n})$ 求出 $1\to n$ 的最短路。

**代码：**https://atcoder.jp/contests/abc232/submissions/35428237