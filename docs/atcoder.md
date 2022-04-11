## AtCoder Beginner Contest 241

!!! note "[G - Round Robin](https://atcoder.jp/contests/abc241/tasks/abc241_g)"
    $n$ 个人，任意两人之间都有一场比赛，一共 $\frac{n\times(n-1)}{2}$ 场。比赛只有胜负，没有平局。

    已知 $m$ 局比赛的胜负，问如果自由安排剩下比赛的胜负，哪些选手可能夺冠？（即胜的场数大于其他所有人）

    $n\leq 50$

若想要让 $x$ 夺冠，则会将他在所有未完成的比赛中获胜，此时可以统计出他的总胜场 $w[x]$。

若 $x$ 能成功夺冠，当且仅当剩下的比赛存在一种胜负方案，使得其他人的胜场都小于 $w[x]$。

考虑最大流模型。

原理是，由于每场比赛都有胜者，没有平局。因此将源点与它们连边，流量限制为 $1$（表示有一个胜者需要分配），然后将每场比赛向可能成为胜者的点连边。通过设置每个人到汇点的流量限制，来限制每个点胜场的上限（除了夺冠者的其他人的获胜场数不能超过 $w[x]-1$）。最终可以通过最大流是否为比赛场数来判断能否分配所有比赛的胜负。

具体来说，除了源点 $s$，汇点 $t$ 之外，还有点 $A_{i,j}(1\leq i<j\leq n)$ 和 $B_i(1\leq i\leq n)$。

$s$ 向 $A_{i,j}$ 连流量限制为 $1$ 的边。

$A_{i,j}$ 表示 $i$ 和 $j$ 之间的比赛，若 $i$ 获胜，连接 $(A_{i,j},B_i)$，若 $j$ 获胜，连接 $(A_{i,j},B_j)$，若尚未决出胜负，则同时向 $B_i,B_j$ 连边。边的流量限制都是 $1$。

$B_x$ 向 $t$ 连接流量为 $w[x]$ 的边，其他 $B_i$ 向 $t$ 连接流量为 $w[x] - 1$ 的边。

若最大流为 $\frac{n\times(n-1)}{2}$，则说明 $x$ 可以夺冠。

对于每个 $x$ 都重新建图跑最大流，复杂度 $O(n^4)$。

## AtCoder Beginner Contest 242

!!! note "[F - Black and White Rooks](https://atcoder.jp/contests/abc242/tasks/abc242_f)"
    在 $n\times m$ 的网格图上放置 $a$ 个白棋和 $b$ 个黑棋，每个格子最多放置一个棋子，且同一行/同一列中不能同时存在黑白棋子。求有多少种喝法的放置方案。

    $n,m\leq 50$

分别枚举白棋和黑棋所占的行数和列数，然后容斥原理即可得到答案。

$ans=\sum\limits_{i=0}^{n}\sum\limits_{j=0}^{m}\sum\limits_{x=0}^{n-i}\sum\limits_{y=0}^{m-j}\binom{i}{n}\binom{j}{m}\binom{x}{n-i}\binom{y}{m-j}\binom{a}{ij}\binom{b}{xy}$

## AtCoder Beginner Contest 243

!!!note "[F - Lottery](https://atcoder.jp/contests/abc243/tasks/abc243_f)"

    一共有 $n$ 个奖品，获得第 $i$ 个奖品的概率为 $p_i$ （$\sum{p_i}=1$ ）。求 $k$ 次抽奖后，获得恰好 $m$ 个不同奖品的概率。

    $k,n,m\leq 50$

多项分布的概率： $P(X_1=c_1,\cdots,X_N=c_N)=\frac{K!}{\prod\limits_{i=1}^{N}{c_i}}\times\prod\limits_{i=1}^{N}{p_i^{c_i}}$，即抽样 $K$ 次获得 $N$ 种不同的结果，其中 $X_i$ 出现 $c_i$ 次的概率。 

则 $f[x][y][z]$ 表示经过 $z$ 次抽奖，在前 $i$ 种奖品中抽中 $y$ 种不同奖品的概率。

枚举当前奖品抽中的次数，则 $f[x+1][y+[a\not=0]][z+a]\leftarrow f[x][y][z]\times \frac{p_{x+1}^a}{a!}$。

答案为 $f[n][m][k]\times k!$。

复杂度 $O(nmk^2)$。

!!!note "[G - Sqrt](https://atcoder.jp/contests/abc243/tasks/abc243_g)"

    有一个无限长的序列，第一项为 $x$。构造规则为每次选择 $[1,\lfloor{\sqrt{x}}\rfloor]$ 中的一个数插到序列末尾。求最终得到多少种不同的序列。

    $x\leq 9\times 10^{18}$

当末尾为 $1$ 时，序列的后面就只能无限插入 $1$，无法再产生新的序列。

根据 $x$ 的范围，序列的第三个数将不会超过 $x^{\frac{1}{4}}<55000$，后面的数取值范围，可只会更小，可以直接暴力算。

记 $f_i(x)$ 为序列第 $i$ 位为 $x$ 的不同序列数，则 $f_{i+1}(a)=\sum\limits_{b=a^2}^{x}{f_{i}(b)}$。

而 $f_{3}(a)=\sum\limits_{b=a^2}^{x}{f_{2}(b)}=\lfloor{\sqrt{x}}\rfloor-a^2+1$，$a\in[1,\lfloor{x^{\frac{1}{4}}}\rfloor]$。

## AtCoder Beginner Contest 244

!!!note "[G - Construct Good Path](https://atcoder.jp/contests/abc244/tasks/abc244_g)"

    给定一张无向连通图，输出一个长度不超过 $4n$ 的路径，使得每个点的访问次数的奇偶性与给定的 $01$ 串相同。

    $n\leq 10^5$

想在不改变路径的前提下改变某个点 $a$ 的奇偶性，只需要将某条边 $(a,b)$ 来回走三遍即可，即 $a\rightarrow b\rightarrow (a\rightarrow b)$（虽然此时 $b$ 奇偶性也改变了，但可以继续通过该方法反转它的奇偶性）。

得到图的任意一棵生成树，记录其欧拉序，则每个点的访问次数的奇偶性已知。从叶子节点开始，若当前点的奇偶性与 $01$ 串相异，则在返回 $fa_x$ 后再重复该边，即 $x\rightarrow fa_x\rightarrow (x\rightarrow fa_x)$ ，此时 $x$ 与 $fa_x$ 的奇偶性同时反转。回到根结点时，除根结点外的所有节点的奇偶性与给定 $01$ 串一致。

若此时根结点的奇偶性与 $01$ 串相同，则已经得到要求的路径，否则只需要在序列末尾增加 $root \rightarrow \cdots\rightarrow root(\rightarrow v\rightarrow root\rightarrow v)$，此时 $v$ 的奇偶性不变，而 $root$ 的奇偶性改变，则所有点的奇偶性都与给定 $01$ 串相同。

树的欧拉序的长度为所有点的度数之和，为 $2n-1$。最劣情况下所有边都被重复，此时长度为 $4n-3$，再加上末尾额外增加的长度为 $3$ 的路径 $\rightarrow v\rightarrow root\rightarrow v$，路径长度的最大值恰好为 $4n$。

## AtCoder Beginner Contest 245

!!!note "[E - Wrapping Chocolate](https://atcoder.jp/contests/abc245/tasks/abc245_e)"

    有 $n$ 个物品，大小为 $a_i\times b_i$。有 $m$ 个盒子，大小为 $c_i\times d_i$。每个盒子至多放一件物品，且不能旋转。求是否能将这 $n$ 个物品全部放进盒子中。

    $1\leq n\leq m\leq 2\times10^5$

本着贪心的思想，显然会找尽可能小的盒子来装物品。

但是由于物品的尺度有两维，因此需要保持一维有序的情况下取另一维最小的。

将 $(a_i,b_i)$ 和 $(c_i,d_i)$ 放在一起，倒序排序。

此时，若是个盒子，将第二维 $d_i$ 放进盒子的集合中。

如果是个物品。由于第一维已经倒序排列，因此集合中的所有盒子的第一维均满足条件，而没有插入集合的所有盒子的第一维均不符合条件。本着贪心的思想，取出集合中第一个大于等于 $b_i$ 的 $d_i$ 与之匹配。若不存在，则无解。

复杂度 $O((n+m)\log(n+m))$。

