!!!note "[I 异度之刃](https://ac.nowcoder.com/acm/contest/31620/I)"

    长为 $n$ 的序列 $a$，有 $m$ 组询问。每次查询区间 $[l,r]$ 中，有多少个「本质不同」的「连续上升」的子串。

    $n,m\leq 10^6$

注意到多组查询且无修改，考虑将查询按照右端点排序后，离线操作。

由于右端点不断向右移动，因此对于本质相同的子串，我们只需要保存最右侧的即可。

记 $f[i]$ 为左端点为 $i$ 的连续上升的子串个数（本质相同的子串只在最右侧的左端点处被统计），则每次查询的答案为 $\sum\limits_{i=l}^{r}{f[i]}$。

移动右端点 $r$，记 $[l,r]$ 是以 $r$ 为右端点且最长的「连续上升」子串，则由于 $r$ 的加入，区间 $f[l\sim r]$ 都增加 $1$。

下面考虑消除左侧「本质相同」的子串的影响，显然这些「本质相同」的子串的末尾元素与 $a[r]$ 相同。

对于每个值开一个单调栈，记录三个值：

- 它的位置 $rp$
- 以 $rp$ 为右端点的最长「连续上升」子串的左端点 $la$
- 当前「连续上升」子串的右端点 $ra$

注意到 $[la,rp]$ 是最长「连续」上升子串，而 $[la,r]$ 是它的一个前缀。

栈是单调的，自底向上 $rp-la+1$ （即最大子串长度）依次递减。

单调递减的原因在于，如果右侧出现一个长度更大的「连续子串」，则靠左的那些子串都会被删去（本质相同且更靠左）。只有当更左侧的子串长度更大时，它比右侧子串长的那部分才会被保留（长度小于等于右侧的部分被删除），这也就是第三个变量 $ra$ 的意义。

对于当前子串 $[l,r,r]$，对于当前栈顶 $[la,ra,rp]$，若 $rp-la\leq r-l$ ，则弹出栈顶，同时 $f[la\sim ra]$ 减一。

否则，考虑删去 $[la,ra,rp]$ 中长度不超过 $r-l+1$ 的部分，即 $[rp-(r-l),rp]$。但由于它此前部分后缀已经被删除，因此实际是 $f[rp-(r-l)\sim ra]$ 减一，并更新 $ra$ 为 $rp-(r-l+1)$ 。

$ra$ 实际上就是当前子串中，未被删除的实际右端点。

最后将 $[l,r,r]$ 插入栈顶。

区间加法和求和用线段树维护。

复杂度 $O((n+m)\log{n})$。

```cpp
#include <bits/stdc++.h>

using namespace std;

template<typename T>
struct SegmentTree {
    vector<T> sum, tag;
    void build();
    void pushUp();
    void pushDown();
    void update();
    T querySum();
    SegmentTree(int n) {
        sum.resize(n * 4 + 100, 0);
        tag.resize(n * 4 + 100, 0);
    }
    SegmentTree(int n, vector<T> &a) {
        sum.resize(n * 4 + 100, 0);
        tag.resize(n * 4 + 100, 0);
        build(1, 1, n, a);
    }
    void build(int rt, int l, int r, vector<T> &a) {
        if (l == r) {
            sum[rt] = a[l];
            return;
        }

        int m = (l + r) / 2;
        build(rt << 1, l, m, a);
        build(rt << 1 | 1, m + 1, r, a);

        pushUp(rt);
    }
    void pushUp(int rt) {
        sum[rt] = sum[rt << 1] + sum[rt << 1 | 1];
    }
    void pushDown(int rt, int len) {
        if (tag[rt]) {
            tag[rt << 1] += tag[rt];
            tag[rt << 1 | 1] += tag[rt];
            sum[rt << 1] += tag[rt] * (len - (len >> 1));
            sum[rt << 1 | 1] += tag[rt] * (len >> 1);
            tag[rt] = 0;
        }
    }
    void add(int rt, int l, int r, int a, int b, T x) {
        if (a <= l && r <= b) {
            sum[rt] += x * (r - l + 1);
            tag[rt] += x;
            return;
        }

        pushDown(rt, r - l + 1);
        int m = (l + r) / 2;

        if (a <= m)
            add(rt << 1, l, m, a, b, x);

        if (b > m)
            add(rt << 1 | 1, m + 1, r, a, b, x);

        pushUp(rt);
    }

    T querySum(int rt, int l, int r, int a, int b) {
        if (a <= l && r <= b)
            return sum[rt];

        pushDown(rt, r - l + 1);

        int m = (l + r) / 2;
        T ret = 0;

        if (a <= m)
            ret += querySum(rt << 1, l, m, a, b);

        if (b > m)
            ret += querySum(rt << 1 | 1, m + 1, r, a, b);

        return ret;
    }
};

struct Query {
    int l, r, id;
    bool operator < (const Query &x) const {
        return r < x.r;
    }
};

int main() {
    cin.tie(NULL)->sync_with_stdio(false);
    int n, m;
    cin >> n >> m;
    vector<int> a(n + 1);

    for (int i = 1; i <= n; ++i)
        cin >> a[i];

    vector<Query> qry(m);

    for (int i = 0; i < m; ++i) {
        cin >> qry[i].l >> qry[i].r;
        qry[i].id = i;
    }

    sort(qry.begin(), qry.end());

    vector<long long> ans(m);

    int pre = 1, pos = 0;
    SegmentTree<long long> seg(n);
    unordered_map<int, int> id;

    for (auto x : a) {
        if (!id.count(x))
            id.insert({x, id.size()});
    }

    vector<deque<array<int, 3>>> stk(id.size()); // [lpos, rpos, ori_rpos]

    for (int i = 1; i <= n; ++i) {
        if (i == 1 || a[i] != a[i - 1] + 1)
            pre = i;

        seg.add(1, 1, n, pre, i, 1);
        int val = id[a[i]], len = i - pre + 1;

        while (!stk[val].empty() && stk[val].back()[2] - stk[val].back()[0] + 1 <= len) {
            auto [l, r, rr] = stk[val].back();
            stk[val].pop_back();
            seg.add(1, 1, n, l, r, -1);
        }

        if (!stk[val].empty()) {
            auto [l, r, rr] = stk[val].back();
            stk[val].pop_back();
            seg.add(1, 1, n, rr - len + 1, r, -1);
            stk[val].push_back({l, rr - len, rr});
        }

        stk[val].push_back({pre, i, i});

        while (pos < m && qry[pos].r == i) {
            ans[qry[pos].id] = seg.querySum(1, 1, n, qry[pos].l, qry[pos].r);
            ++pos;
        }
    }

    for (auto x : ans)
        cout << x << "\n";

    return 0;
}
```