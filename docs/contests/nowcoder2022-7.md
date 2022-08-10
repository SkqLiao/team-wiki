!!!note "[E. Ternary Search](https://ac.nowcoder.com/acm/contest/33192/E)"

	一个长度为$n$的序列，问对于每个前缀，要把这个前缀变成单峰的至少需要交换相邻的数多少次。保证所有数都互不相同。

	$n\leq 2\times 10^5,1\leq a_i\leq 10^9$


这题的单峰，上凸和下凸是本质相同的，我们只考虑上凸的情况。

首先考虑只求整个序列的最少交换次数怎么做。如果我们想要确定这个峰的位置，后面的计算会比较困难，因此要换一个角度。一个观察是，对于最小的数字，可以放到最左或最右，剩下的部分是一个子问题，与这个最小值是无关的，那么我们只需要从小往大做，每次贪心地选择放到左端或右端即可。

现在考虑原问题怎么做，不难发现，一个数$a_x$往左端放的答案是一定的，也即它左侧比它大的数的个数，我们记作$L$。但是往右放的答案$R$，即右侧比它大的数的个数，取决于当前考虑的右端点。由于这个右侧的贡献$R$是阶梯状的，我们无法直接维护，需要反过来考虑。对于一个位置$p$，它会使它左侧所有值小于$a_p$的$R$都加一，也就是说它会使前缀中$L>R$且$a_i < a_p$的所有位置贡献$+1$，我们就是要快速维护出贡献加了多少（同时不难发现刚加进来$R=0$，因此贡献一定是$0$）。我们可以用一棵权值线段树维护每个位置$L-R$的值，当一个位置的值变为了$0$，说明不再会往右移动，我们就把这个数从权值线段树中删掉。然后，所有有值的位置会使得贡献$+1$。具体来说，考虑到$p$这个位置时，权值线段树上$[1,a_p-1]$每个有值的位置会贡献$1$，然后线段树上$a_p$这个位置设置为$L$，$[1,a_p-1]-=1$，如果一个位置的值变为了$0$，我们就将它取出，表示不再会选择往右移动了。

写的时候可以用树状数组维护所有有贡献的位置，变为树状数组上的区间求和。权值线段树上可以将所有位置的初始值设置为$inf$，维护区间最小值。每次操作时就是cover一个$L$，区间$-1$，这一个值变为$0$以后可以将它重新设置为$inf$，由于一个位置只会变为$0$一次，所以复杂度是对的。

复杂度$O(n\log n)$

```cpp
#include<bits/stdc++.h>
#include <random>
#define pb push_back
#define mkp make_pair
#define fi first
#define se second
#define hvie '\n'
using namespace std;
typedef pair<int, int> pii;
typedef long long ll;
typedef unsigned long long ull;
typedef long double db;
int read() {
    int ret = 0;
    bool f = 0;
    char c = getchar();

    while (!isdigit(c)) {
        if (c == EOF)
            return -1;

        if (c == '-')
            f = 1;

        c = getchar();
    }

    while (isdigit(c))
        ret = (ret << 3) + (ret << 1) + (c ^ 48), c = getchar();

    return f ? -ret : ret;
}

const int N = 2e5 + 10,M=N<<2,inf = 0x3f3f3f3f, mod = 998244353;

vector<int>vec;

struct Segment{
	#define ls (x<<1)
	#define rs (x<<1|1)
	#define mid ((l+r)>>1)
	int tag[M],mn[M];
	void build(int x,int l,int r){
		tag[x]=0;mn[x]=inf;
		if(l==r) return;
		build(ls,l,mid);
		build(rs,mid+1,r);
	}
	void pushup(int x){
		mn[x]=min(mn[ls],mn[rs]);
	}
	void pushdown(int x){
		int &t=tag[x];
		if(!t) return;
		tag[ls]+=t;tag[rs]+=t;mn[ls]+=t;mn[rs]+=t;
		t=0;
	}
	void cover(int x,int l,int r,int p,int v){
		if(l==r){
			mn[x]=v;
			return;
		}
		pushdown(x);
		if(p<=mid) cover(ls,l,mid,p,v);
		else cover(rs,mid+1,r,p,v);
		pushup(x);
	}
	void update(int x,int l,int r,int L,int R,int v){
		if(L>R) return;
		if(L<=l && r<=R){
			mn[x]+=v;tag[x]+=v;
			return;
		}
		pushdown(x);
		if(L<=mid) update(ls,l,mid,L,R,v);
		if(R>mid) update(rs,mid+1,r,L,R,v);
		pushup(x);
	}
	void query(int x,int l,int r){
		if(mn[x]>0) return;
		if(l==r){
			vec.pb(l);mn[x]=inf;
			return;
		}
		pushdown(x);
		query(ls,l,mid);
		query(rs,mid+1,r);
		pushup(x);
	}
	#undef ls
	#undef rs
	#undef mid
}T;

struct BIT{
	int lim;
	vector<int>c;
	void clear(int n){
		c.resize(n+5);
		fill(c.begin(),c.end(),0);
		lim=n+1;
	}
	int lowbit(int x){
		return x&(-x);
	}
	void update(int x,int v){
		for(;x<lim;x+=lowbit(x)) c[x]+=v;
	}
	int query(int x){
		int ret=0;
		for(;x;x-=lowbit(x)) ret+=c[x];
		return ret;
	}
	int query(int l,int r){
		if(l>r) return 0;
		return query(r)-query(l-1);
	}
}bit0,bit1;


int n;
int a[N],b[N];
ll fans[N],ans;

void solve() {
    n=read();
    for(int i=1;i<=n;++i) a[i]=read(),b[i]=a[i];
    sort(b+1,b+n+1);
	for(int i=1;i<=n;++i) a[i]=lower_bound(b+1,b+n+1,a[i])-b;
	//for(int i=1;i<=n;++i) cout<<a[i]<<" ";cout<<hvie;

	memset(fans,0x3f,sizeof(fans));

	bit0.clear(n);
	bit1.clear(n);
	T.build(1,1,n);
	ans=0;
	for(int i=1;i<=n;++i){
		//cout<<"!!"<<i<<hvie;
		ans+=bit0.query(a[i]-1);
		fans[i]=min(fans[i],ans);

		bit0.update(a[i],1);
		bit1.update(a[i],1);

		T.update(1,1,n,1,a[i]-1,-1);
		T.cover(1,1,n,a[i],bit1.query(a[i]+1,n));

		vec.clear();
		T.query(1,1,n);
		for(auto v:vec) bit0.update(v,-1);
			
		//cout<<"!!"<<i<<hvie;
	}

	bit0.clear(n);
	bit1.clear(n);
	T.build(1,1,n);
	ans=0;
	for(int i=1;i<=n;++i){
		a[i]=n-a[i]+1;
		ans+=bit0.query(a[i]-1);
		fans[i]=min(fans[i],ans);

		bit0.update(a[i],1);
		bit1.update(a[i],1);

		T.update(1,1,n,1,a[i]-1,-1);
		T.cover(1,1,n,a[i],bit1.query(a[i]+1,n));

		vec.clear();
		T.query(1,1,n);
		for(auto v:vec) bit0.update(v,-1);
			
		//cout<<"!!"<<i<<hvie;
	}

	for(int i=1;i<=n;++i) printf("%lld\n",fans[i]);
	puts("");
}

signed main() {
    for (int cas = 1; cas--;) {
        solve();
    }

    return 0;
}

```

!!!note "[K.Great Party](https://ac.nowcoder.com/acm/contest/33192/K)"

    一个长度为$n$的序列$a_i$，表示第$i$堆有$a_i$个石子。$Q$次询问一个区间$[l,r]$的所有子区间进行博弈有多少个先手必胜。

    博弈的内容是，每次先从一堆石子中拿任意数量，然后可以选择将这堆石子合并到任意一堆上，不能取石子的人输。

    $n,Q\leq 10^5,$1\leq a_i\leq 10^6$

如果没有合并就是个朴素的nim游戏，可以往这上面去想。首先一堆的时候一定是先手胜，两堆的时候，想赢要保证自己拿完还是两堆，也就是每一堆有一个石子不能拿，那么实际上就是一个$a_i-1$的nim游戏。三堆的时候，我一定可以操作最大的那堆，然后选择合并，使得剩下两堆石子数量相同，因此三堆是先手必胜。

这样依此类推，可以发现奇数堆时先手必胜，偶数时不能改变堆数，是个$a_i-1$的nim游戏。问题转化为有多少个长度为偶数的子区间的异或和为$0$，这是一个经典的莫队问题。

复杂度$O(n\sqrt n)$


```cpp
#include <bits/stdc++.h>

using namespace std;

typedef long long ll;

const int M = 1e5 + 5;
const int N = 2e6 + 5;

int a[M], s[M],n, q, k, blo[M], bloSZ;
ll answer, ans[M];
struct query {
    int l, r, id;
} queries[M];

bool cmp(const query &x, const query &y) {
    return (blo[x.l] ^ blo[y.l]) ? blo[x.l] < blo[y.l] : ((blo[x.l] & 1) ? x.r < y.r : x.r > y.r);
}

void divide() {
    bloSZ = sqrt(n);

    for (int i = 1; i <= n; ++i) {
        blo[i] = (i - 1) / bloSZ + 1;
    }
}

ll cnt[2][N];

void add(int pos) {
    answer += cnt[pos&1][s[pos] ^ k];
    cnt[pos&1][s[pos]]++;
}

void del(int pos) {
    cnt[pos&1][s[pos]]--;
    answer -= cnt[pos&1][s[pos] ^ k];
}

void Mo() {
    sort(queries + 1, queries + 1 + q, cmp);
    int l = 0, r = -1;

    for (int i = 1; i <= q; ++i) {
        while (l > queries[i].l)
            add(--l);

        while (r < queries[i].r)
            add(++r);

        while (l < queries[i].l)
            del(l++);

        while (r > queries[i].r)
            del(r--);

        ans[queries[i].id] = answer;
    }
}

int l[M], r[M];

int main() {
    cin.tie(NULL)->ios_base::sync_with_stdio(false);
    cin >> n >> q;

    for (int i = 1; i <= n; ++i) {
        cin >> a[i];
        s[i] = s[i - 1] ^ (a[i]-1);
    }

    divide();

    for (int i = 1; i <= q; ++i) {
        cin >> l[i] >> r[i];
        queries[i] = (query) {
            l[i] - 1, r[i], i
        };
    }

    Mo();

    for (int i = 1; i <= q; ++i) {
        printf("%lld\n", 1ll * (r[i] - l[i] + 1) * (r[i] - l[i] + 2) / 2 - ans[i]);
    }

    return 0;
}
```