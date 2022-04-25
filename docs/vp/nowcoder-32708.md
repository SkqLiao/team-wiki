# The 2021 ICPC Asia Kunming Regional Programming Contest

- 比赛链接：[link](https://ac.nowcoder.com/acm/contest/32708)

- Rank 24, solved 6/13

!!!note "[B.Blocks](https://ac.nowcoder.com/acm/contest/32708/B)"

    有一个左下角为$(0,0)$，右上角为$(H,W)$的网格矩形，有$n$个子矩形左下角格点为$(x_{1,i},y_{1,i})$，右上角格点为$(x_{2,i},y_{2,i})$。现在每次从这$n$个子矩形里随机选择一个全涂黑，直到整个大矩形被涂黑，求期望步数，或输出-1表示不可能。

    $n\leq 10$

首先这个东西给的是格点，涂的是里面的格子，这里没读清楚卡了很久。简单的处理方式是变成左闭右开这样。

观察到这个数据范围很小，其实很容易往状压上想，一个直观的想法是设$f[s]$表示$s$这个状态全涂过的期望步数，然后考虑能转移到哪里，其实就是枚举每个矩形，然后它可以转移到$f[s|(1<<i)]$。

这样写出来以后，你会发现这个式子形如：

$$f[s]=\frac 1 n((f[s_1]+1)+(f[s_2]+1)+\cdots)$$

左右两边可以同时乘以$n$。然后这个东西显然是个DAG，于是按照拓扑序转移即可。当然会有自己转移到自己的可能性，不过也只需要移向即可。

但是按照上面这么做是出不来结果的，因为你会发现有多个终态（某些$s$能使整个矩形涂黑）。因此我们需要倒过来考虑，从终态出发往$f[0]$推。

然后你会发现式子更加优美了，因为现在一个点的方程一定恰好有$n$项！（考虑到如果这个点不是终态，那它能选择$n$个矩形中的一个，那么现在就会形如）

$$f[s]=1+\frac 1 n(f[s_1]+f[s_2]+\cdots)$$

下面还有一个问题是如何求某个状态是否是终态。但是其实这不是问题，因为实在是太小了，所以坐标离散化以后暴力涂色就行。

最后复杂度大概是$O(n^32^n)$或者其他啥样的。

```cpp
#include<bits/stdc++.h>
#define fi first
#define se second
#define pb push_back
#define mkp make_pair
#define rep(i,l,r) for(int i=(l),i##end=(r);i<=i##end;i++)
#define dwn(i,l,r) for(int i=(l),i##end=(r);i>=i##end;i--)
#define forn(i,n) for(int i=0;i<(n);i++)
#define hvie '\n'
#define read yh
#define int long long
using namespace std;

typedef double db;
typedef pair<int,int> pii;
typedef long long ll;

int yh(){
    int ret=0,f=0;char c=getchar();
    while(!isdigit(c)){if(c=='-')f=1;c=getchar();}
    while(isdigit(c)){ret=ret*10+(c^48);c=getchar();}
    return f?-ret:ret;
}

const int N=22,M=1055,mod=998244353;
int f[M],W,H,n,S[M],ans,cnt[M];
array<int,4>a[N];

int qpow(int x,int y){
	int ret=1;
	for(;y;y>>=1,x=x*x%mod)
		if(y&1) ret=ret*x%mod;
	return ret;
}

int mp[N][N],A[M],du[M],core[M][M];
vector<pii>G[M];
void color(int i){
	for(int x=a[i][0]+1;x<=a[i][2];++x)
		for(int y=a[i][1]+1;y<=a[i][3];++y) mp[x][y]=1;
}
int getarea(){
	int ret=0;
	for(int i=1;i<=W;++i) for(int j=1;j<=H;++j)
		if(mp[i][j]) ++ret;
	return ret;
}

queue<int>q;
void topsort(){
	for(int i=0;i<(1<<n);++i) if(S[i]==(W-1)*(H-1)){
		q.push(i),f[i]=0;
		//cout<<"ending "<<i<<endl;
	}
	while(!q.empty()){
		int x=q.front();q.pop();
		//printf("%lld %lld %lld\n",x,f[x],S[x]);
		//cout<<x<<" "<<n-__builtin_popcount(x)<<" "<<A[x]<<endl;
		for(auto v:G[x]){
			//cout<<x<<" "<<v.fi<<" val:"<<v.se<<endl;
			f[v.fi]+=v.se*f[x]%mod;
			f[v.fi]%=mod;
			--du[v.fi];
			if(!du[v.fi]){
				q.push(v.fi);
				f[v.fi]=(f[v.fi]+n)%mod;
				f[v.fi]=f[v.fi]*qpow(n-__builtin_popcount(v.fi),mod-2)%mod;
			}
		}
	}
	printf("%lld\n",f[0]);
}

void solve(){
	n=read();W=read();H=read();
	vector<int>dx,dy;
	for(int i=1;i<=n;++i){
		int x1=yh(),y1=yh(),x2=yh(),y2=yh();
		//cout<<"!!"<<x1<<" "<<y1<<" "<<x2<<" "<<y2<<endl;
		a[i]={x1,y1,x2,y2};
		dx.pb(x1);
		dx.pb(x2);
		dy.pb(y1);
		dy.pb(y2);
	}
	dx.pb(-1);dy.pb(-1);
	dx.pb(0);dy.pb(0);
	dx.pb(W);dy.pb(H);
	sort(dx.begin(),dx.end());
	sort(dy.begin(),dy.end());
	dx.resize(unique(dx.begin(),dx.end())-dx.begin());
	dy.resize(unique(dy.begin(),dy.end())-dy.begin());
	rep(i,1,n){
		auto [x,y,z,w]=a[i];
		x=lower_bound(dx.begin(),dx.end(),x)-dx.begin();
		z=lower_bound(dx.begin(),dx.end(),z)-dx.begin();
		y=lower_bound(dy.begin(),dy.end(),y)-dy.begin();
		w=lower_bound(dy.begin(),dy.end(),w)-dy.begin();
		a[i]={x,y,z,w};
	}
	W=lower_bound(dx.begin(),dx.end(),W)-dx.begin();
	H=lower_bound(dy.begin(),dy.end(),H)-dy.begin();
	int lim=(1<<n);
	for(int s=0;s<lim;++s){
		memset(mp,0,sizeof(mp));
		for(int i=0;i<n;++i) if(s&(1<<i)){
			color(i+1);
		}
		S[s]=getarea();
		//cout<<s<<" "<<S[s]<<" "<<W*H<<endl;
	}
	if(S[lim-1]!=(W-1)*(H-1)){
		puts("-1");
		return;
	}
	for(int s=0;s<lim;++s){
		if(S[s]==(W-1)*(H-1)) continue;
		for(int i=0;i<n;++i) ++core[s][s|(1<<i)];
	}
	for(int s=0;s<lim;++s){
		for(int ls=0;ls<lim;++ls){
			if(core[s][ls] && s!=ls) G[ls].pb(mkp(s,core[s][ls])),++du[s];
		}
		A[s]=n-core[s][s];
	}
	topsort();
	for(int i=0;i<lim;++i) G[i].clear();
	memset(core,0,sizeof(core));
	memset(du,0,sizeof(du));
	memset(A,0,sizeof(A));
	memset(f,0,sizeof(f));
}

signed main(){
	for(int cas=read();cas--;){
		solve();
	}
    return 0;
}
```

!!!note "[E.Easy String Problem](https://ac.nowcoder.com/acm/contest/32708/E)"

    给定一个长度为$n$的字符集大小也为$n$的字符串，$q$个独立的询问，每次询问如果将字符串删去包括$[l_i,r_i]$的任意子串，可以得到多少个本质不同的字符串。

    $n,q\leq 10^5$

这个题比赛的时候一看就不是一个传统的字符串题，于是没开。赛后想了挺久才想到的。

我们考虑如何统计这些本质不同的字符串，由于我们删除的是包含$[l,r]$的子串，那么最后得到的字符串相当于一段前缀+一段后缀。考虑如何去重，一个比较简明的方式是，我们尽量在前面匹配字符串，直到不能匹配才在后面找。当我们匹配到$i$，如果我们$s[i]$这个字符没有在前面选择，而是在后面选择了（即删除$[i,?]$这段），算重的次数实际上就是这个字符在后面出现的次数。

这样分析，不难发现总的算重次数就是左右区间每类字符出现次数的乘积。也即：

$$\sum_{i=1}^n cntl[i]\times cntr[i]$$

这是一个莫队的经典问题。

最后再用总可选区间个数减一下就行了。

复杂度$O(n\sqrt n)$


!!!note "[G.Glass Bead Game](https://ac.nowcoder.com/acm/contest/32708/G)"

    有$n$个豆子，每轮会选择一个豆子放到最前面，第$i$个豆子被选中的概率是$p_i$，满足$\sum p_i=1$。对于一次移动，若当前它是第$i$个，则代价是$i-1$。问无限轮之后，进行一轮的期望代价。

    $n\leq 100$

简单概率题，看到就秒了。

由期望的线性性，实际上我们要考虑的就是对于第$i$个豆子，它的期望排名是多少，然后求个和即可。

然后这个东西又可以转化为无限轮后每个豆子在它前面的概率和，也即：

$$E[i]=\sum_{j\neq i} p[第j个豆子在第i个豆子前面]$$

然后因为是无限轮，这个东西相当于只用考虑上一轮选了$i$还是选了$j$。我们不需要考虑其他的豆子，它们不影响$i$和$j$。而当只有它们两个的时候，显然这个概率就是$\frac {p_j} {p_i+p_j}$。

然后就没有然后了。

复杂度$O(n^2)$



!!!note "[I.Interval Mex](https://ac.nowcoder.com/acm/contest/32708/I)"

    给定一个序列$a_i$，$q$个询问，每次询问一个区间$[l_i,r_i]$的所有子区间中$\text{mex}$的第$k$小值是多少。

    $n,q\leq 10^5$，空间限制64MB

这个空间限制还是有点阴间的。

首先不妨考虑单个询问$[1,n]$怎么做。首先这个$\text{mex}$显然是可以二分的，假设我们现在二分出的值为$m$。设$p_{i,j}$为$j$这个数字在$\leq i$出现的最靠右的位置（如果没有就为0），再设$t_{i}=\min_{0\leq j<m}\{p_{i,j}\}$，根据定义，如果我们以$i$为右端点，满足$\text{mex}\geq m$的区间个数就是$t_i$。

考虑怎么计算这个$t_i$，我们枚举$j=0\dots m-1$，考虑$j$出现的所有位置$b_1,\dots b_c$，然后令$b_0=0,b_{c+1}=n+1$，设初始$t_i$均为$n+1$，那么有

$$\forall b_k\leq i\leq b_{k+1},t_i=\min\{t_i,b_k\}$$

显然$t_i$是单调递增的一个序列，那么我们要做的实际上是区间对一个数取$\min$操作，然后还要求区间和。

再考虑到区间$[L,R]$，其实和前面类似处理，只是最后统计答案的时候，满足$\text{mex}\geq m$的区间总个数就是$\sum_{i<R,t_i\geq L}(t_i-L+1)$。于是我们还要支持查询区间最大值以便找到满足条件的$t_i$从哪开始。

以上，支持对一个数取$\min$，区间和，区间最大值，是一个经典的线段树问题，可以通过额外维护次大值和最大值个数来实现，具体来说：

- 当$max\leq v$时，显然这一次修改不会对这个节点产生影响，直接退出。

- 当$smax<v<max$时，显然这一次修改只会影响到所有最大值，所以我们 把$sum$加上$cnt\cdot (v-max)$，把 $max$ 更新为$x$ ，打上标记然后退出。

- 当$smax>x$时，递归处理。

接下来考虑$q$个询问，基于前面的二分想法，我们直接考虑整体二分即可。

每一次二分后，贡献的统计与小于$mid$的所有数字出现的位置有关，我们需要保存$\log $棵线段树状态，以便完成左边的处理以后继承递归到右边保证复杂度，但是由于这个东西的空间只有64MB，显然开不下。

这里有一个和巧妙的操作是把整体二分写成非递归形式，记录下每个询问当前的二分区间$[l_i,r_i]$，然后按照$mid_i$从小到大排序，然后还是像正常一样扫一遍统计答案。

事实上对于每层二分，线段树的形态变化都一模一样，不同的只是每个询问的询问时间。这是一种通用的方法。

复杂度$O(n\log^2n)$

```cpp
#include<bits/stdc++.h>
#define fi first
#define se second
#define pb push_back
#define mkp make_pair
#define hvie '\n'
#define read yh
#define int long long
using namespace std;

typedef long double db;
typedef pair<int,int> pii;
typedef long long ll;

int yh(){
    int ret=0,f=0;char c=getchar();
    while(!isdigit(c)){if(c=='-')f=1;c=getchar();}
    while(isdigit(c)){ret=ret*10+(c^48);c=getchar();}
    return f?-ret:ret;
}

const int N=1e5+10;
const db eps=1e-8,pi=acos(-1);
int n,Q;
int ans[N];

struct Seg{
	#define ls (x<<1)
	#define rs (x<<1|1)
	#define mid ((l+r)>>1)

	int c[N<<2],mx[N<<2],smx[N<<2],sum[N<<2],cnt[N<<2];

	void pushup(int x){
		sum[x]=sum[ls]+sum[rs];
		mx[x]=max(mx[ls],mx[rs]);
		cnt[x]=0;
		if(mx[ls]==mx[rs]){
			cnt[x]=cnt[ls]+cnt[rs];smx[x]=max(smx[ls],smx[rs]);
		}
		else{
			if(mx[ls]>mx[rs]){
				cnt[x]=cnt[ls];smx[x]=max(smx[ls],mx[rs]);
			}
			else{
				cnt[x]=cnt[rs];smx[x]=max(smx[rs],mx[ls]);
			}
		}
	}
	void pushdown(int x){
		if(mx[ls]>mx[x]){
			sum[ls]-=(mx[ls]-mx[x])*cnt[ls];
			mx[ls]=mx[x];
		}
		if(mx[rs]>mx[x]){
			sum[rs]-=(mx[rs]-mx[x])*cnt[rs];
			mx[rs]=mx[x];
		}
	}
	void build(int x,int l,int r){
		mx[x]=cnt[x]=sum[x]=0;smx[x]=-1;
		if(l==r){
			sum[x]=mx[x]=n+1;cnt[x]=1;smx[x]=-1;
			return;
		}
		build(ls,l,mid);build(rs,mid+1,r);
		pushup(x);
	}
	void updatemin(int x,int l,int r,int L,int R,int v){
		if(mx[x]<=v) return;
		if(L<=l && r<=R && smx[x]<v){
			sum[x]-=(mx[x]-v)*cnt[x];mx[x]=v;
			return;
		}
		pushdown(x);
		if(L<=mid) updatemin(ls,l,mid,L,R,v);
		if(R>mid) updatemin(rs,mid+1,r,L,R,v);
		pushup(x);
	}
	int querypos(int x,int l,int r,int L,int R){
		if(mx[x]<L) return n+1;
		if(l==r) return l;
		pushdown(x);
		if(mx[ls]>=L) return querypos(ls,l,mid,L,R);
		else if(R>mid) return querypos(rs,mid+1,r,L,R);
		return n+1;
	}
	int querysum(int x,int l,int r,int L,int R){
		if(L<=l && r<=R) return sum[x];
		pushdown(x);
		int ret=0;
		if(L<=mid) ret+=querysum(ls,l,mid,L,R);
		if(R>mid) ret+=querysum(rs,mid+1,r,L,R);
		return ret; 
	}
	#undef ls
	#undef rs
	#undef mid
}T;

struct node{
	int l,r,mid,ql,qr,k,id;
}q[N];
vector<int>G[N];

int C2(int x){return x*(x+1)/2;}
void solve(vector<int>&vec){
	vector<int>res;
	sort(vec.begin(),vec.end(),[&](int x,int y){
		return q[x].mid<q[y].mid;
	});
	T.build(1,1,n);

	int now=0,m=vec.size();
	for(int i=0;i<=n && now<m;++i){
		int las=n+1;
		for(auto p:G[i]){
			T.updatemin(1,1,n,p,las-1,p);
			las=p;
		}
		if(las>1){
			T.updatemin(1,1,n,1,las-1,0);
		}
		for(;now<m && q[vec[now]].mid==i;++now){
			int t=vec[now];
			int p=T.querypos(1,1,n,q[t].ql,q[t].qr),hv=0;
			if(p<=q[t].qr) hv=T.querysum(1,1,n,p,q[t].qr)-(q[t].ql-1)*(q[t].qr-p+1);

			hv=C2(q[t].qr-q[t].ql+1)-hv;
			//printf("nowfind:%lld %lld %lld %lld\n",t,p,hv,(q[t].ql-1)*(q[t].qr-p+1));
			if(hv<q[t].k){
				q[t].l=q[t].mid+1;
				q[t].mid=(q[t].r+q[t].l)>>1;
			}
			else{
				q[t].r=q[t].mid-1;ans[t]=q[t].mid;
				q[t].mid=(q[t].r+q[t].l)>>1;
			}
			if(q[t].l<=q[t].r) res.pb(t);
		}
	}
	/*puts("\nres:");
	for(auto v:res) printf("%lld ",v);
	puts("\nvec:");
	for(auto v:vec) printf("l:%lld r:%lld %lld\n",q[v].l,q[v].r,v);
	puts("");*/
	swap(res,vec);
}

vector<int>vec;
void solve(){
	n=read();Q=read();
	for(int i=1;i<=n;++i) G[read()].pb(i);
	for(int i=0;i<=n;++i) reverse(G[i].begin(),G[i].end());
	for(int i=1;i<=Q;++i){
		q[i].ql=read();q[i].qr=read();q[i].k=read();q[i].id=i;
		q[i].l=0;q[i].r=q[i].qr-q[i].ql+1;q[i].mid=(q[i].l+q[i].r)>>1;
		vec.pb(i);
	}
	for(;vec.size();){
		solve(vec);
	}
	for(int i=1;i<=Q;++i) printf("%lld\n",ans[i]);
}

signed main(){
	for(int cas=1;cas--;){
		solve();
	}
    return 0;
}
```

!!!note "[J.Just Another String Problem](https://ac.nowcoder.com/acm/contest/32708/J)"

    给定一个字符串$s$，和一个数组$a$。对于一个字符串集合，要求没有任何一个字符串是其他字符串的后缀，其中长度为$i$的字符串可以贡献$a_i$。一个集合的价值是其中所有字符串的贡献和。$q$个询问，每次询问$s$的一个前缀子串$s_i$，在其中选择不超过$k_i$个串组成集合，问集合的最大价值。

    $|s|,q\leq 2\times10^5,a_i\leq 10^9$，多组数据。

其实这是一个大水题，现场赛没人做，vp的时候线段树写错了寄了。

观察到集合中选择的字符串有这个后缀关系的限制，联想到在SAM的parent树上，父节点的endpos是所有子节点endpos的并，而且父节点的长度一定小于子节点，因此我们实际上最优的选择就是parent树上所有的叶子节点。现在有这个$k_i$的限制，我们只需要用线段树简单维护一下每个长度有多少个以及价值和即可。

事实上我们不需要构建出SAM后再复杂地离线，在extend每个字符后父子关系也满足这个关系，我们只需要维护每个节点是否有儿子，然后顺便在线段树上修改一下就行了。

复杂度$O(n\log n)$

其实KMP树好像就能做这个问题了，不过没有写。

```cpp
/*SAM*/
#include<bits/stdc++.h>
#define fi first
#define se second
#define pb push_back
#define mkp make_pair
#define rep(i,l,r) for(int i=(l),i##end=(r);i<=i##end;i++)
#define dwn(i,l,r) for(int i=(l),i##end=(r);i>=i##end;i--)
#define forn(i,n) for(int i=0;i<(n);i++)
#define hvie '\n'
#define read yh
#define int long long
using namespace std;

typedef double db;
typedef pair<int,int> pii;
typedef long long ll;

int yh(){
    int ret=0,f=0;char c=getchar();
    while(!isdigit(c)){if(c=='-')f=1;c=getchar();}
    while(isdigit(c)){ret=ret*10+(c^48);c=getchar();}
    return f?-ret:ret;
}

const int N=4e5+10,mod=998244353;
int n,Q;
int val[N],ans[N];

struct Seg{
	#define ls (x<<1)
	#define rs (x<<1|1)
	#define mid ((l+r)>>1)
	int cnt[N<<2],sum[N<<2];
	void build(int x,int l,int r){
		cnt[x]=sum[x]=0;
		if(l==r) return;
		build(ls,l,mid);build(rs,mid+1,r);
	}
	void pushup(int x){
		cnt[x]=cnt[ls]+cnt[rs];
		sum[x]=sum[ls]+sum[rs];
	}
	void update(int x,int l,int r,int p,int v){
		if(!p) return;
		if(l==r){
			cnt[x]+=v;sum[x]=cnt[x]*val[l];
			return;
		}
		if(p<=mid) update(ls,l,mid,p,v);
		else update(rs,mid+1,r,p,v);
		pushup(x);
	}
	int query(int x,int l,int r,int K){
		if(cnt[x]<=K) return sum[x];
		if(l==r) return K*val[l];
		if(cnt[rs]>=K) return query(rs,mid+1,r,K);
		return sum[rs]+query(ls,l,mid,K-cnt[rs]);
	}
	#undef mid
	#undef ls
	#undef rs
}T;



struct SAM{
	int sz,las;
	int mx[N],fa[N],son[N],ch[N][26];
	void init(){
		for(int i=0;i<=sz;++i){
			mx[i]=fa[i]=son[i]=0;
			memset(ch[i],0,sizeof(ch[i]));
		}
		sz=las=1;
	}
	void updatefa(int x,int f){
		if(fa[x]){
			--son[fa[x]];
			if(!son[fa[x]]) T.update(1,1,n,mx[fa[x]],1);
		}
		fa[x]=f;
		if(f){
			++son[f];
			if(son[f]==1) T.update(1,1,n,mx[f],-1);
		}
	}
	void extend(int x){
		int p,q,np,nq;
		p=las;las=np=++sz;mx[np]=mx[p]+1;
		T.update(1,1,n,mx[np],1);
		for(;p && !ch[p][x];p=fa[p]) ch[p][x]=np;
		if(!p) updatefa(np,1);
		else{
			q=ch[p][x];
			if(mx[q]==mx[p]+1){
				updatefa(np,q);
			}
			else{
				nq=++sz;mx[nq]=mx[p]+1;T.update(1,1,n,mx[nq],1);
				memcpy(ch[nq],ch[q],sizeof(ch[q]));
				updatefa(nq,fa[q]);updatefa(q,nq);updatefa(np,nq);
				for(;ch[p][x]==q;p=fa[p]) ch[p][x]=nq;
			}
		}
	}
}S;

char s[N];
vector<pii>q[N];
void solve(){
	n=read();Q=read();scanf("%s",s+1);
	for(int i=1;i<=n;++i) val[i]=read();
	for(int i=1;i<=Q;++i){
		int l=read(),k=read();
		q[l].pb(mkp(i,k));
	}
	T.build(1,1,n);S.init();
	for(int i=1;i<=n;++i){
		S.extend(s[i]-'a');
		for(auto v:q[i]) ans[v.fi]=T.query(1,1,n,v.se);
	}
	for(int i=1;i<=Q;++i) printf("%lld\n",ans[i]);
	for(int i=0;i<=n;++i) q[i].clear();
}

signed main(){
	for(int cas=read();cas--;){
		solve();
	}
    return 0;
}
```
