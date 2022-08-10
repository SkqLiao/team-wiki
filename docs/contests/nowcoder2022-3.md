!!!note "[B.Boss](https://ac.nowcoder.com/acm/contest/33188/B)"

    $n$个人，$K$座城市，每座城市有需要$e_i$的人，第$i$个人分配到$j$城市花费$c_{ij}$，求最小分配花费。

    $n\leq 10^5,K\leq 10,\sum e_i=n$


不考虑数据范围，这是一个简单的最小费用最大流问题。直接做不了，那么题目很明显就是要我们用模拟费用流来优化。

考虑增广过程，这是一个只有两层的图，将一个已经分配好的人转移到另一个城市只需要走一条反向边再走一条正向边，我们可以合并这两步，用一条边代替正向边+反向边。

这样图上一共有$K$个点表示城市，我们先把所有人分配到一号城市，其他城市再从这里抢人，每个人分配的代价就是上述的正向边+反向边。对每个人可以从当前城市走到任一其他城市，那就是$K-1$条边，但是这样实际上会有很多重边，而我们实际上只关心其中最短的一条。所以我们可以用$K^2$个堆维护两两之间的重边，再跑SPFA求出最小费用模拟增广即可。同时注意每转移一个人要更新一次堆。

复杂度$O(n\cdot \text{SPFA}(K^2)+nK\log n)$


```cpp
#include<bits/stdc++.h>
#define rep(i,a,b) for(int i=(a),i##ss=(b);i<=i##ss;i++)
#define dwn(i,a,b) for(int i=(a),i##ss=(b);i>=i##ss;i--)
#define rng(i,a,b) for(int i=(a);i<(b);i++)
#define deb(x) cerr<<(#x)<<":"<<(x)<<'\n'
#define pb push_back
#define mkp make_pair
#define fi first
#define se second
#define hvie '\n'
#define read yh
#define int long long
using namespace std;
typedef pair<int,int> pii;
typedef long long ll;
typedef unsigned long long ull;
typedef double db;
int yh(){
	int ret=0;bool f=0;char c=getchar();
	while(!isdigit(c)){if(c==EOF)return -1;if(c=='-')f=1;c=getchar();}
	while(isdigit(c))ret=(ret<<3)+(ret<<1)+(c^48),c=getchar();
	return f?-ret:ret;
}
const int N=1e5+5,M=12,mod=998244353,inf=0x3f3f3f3f3f3f3f3f;

int n,K,s;
int bl[N],c[N][M],e[N];
int dis[M],vis[M],las[M];
vector<int>G[N];
priority_queue<pii>q[M][M];

int spfa(int s){
    for(int i=1;i<=K;++i){
        dis[i]=inf;vis[i]=0;las[i]=0;
    }
    queue<int>que;que.push(s);
    dis[s]=0;vis[s]=1;
    while(!que.empty()){
        int x=que.front();que.pop();vis[x]=0;
        for(int v=s-1;v>=1;--v){
            if(q[x][v].empty()) continue;
            int d=-q[x][v].top().fi;
            if(dis[v]>dis[x]+d){
                dis[v]=dis[x]+d;las[v]=x;
                if(!vis[v]) vis[v]=1,que.push(v);
            }
        }
    }

    vector<int>vec;
    for(int x=1;x!=s;x=las[x]){
        int t=q[las[x]][x].top().se;
        vec.pb(t);bl[t]=las[x];
    }
    for(auto x:vec){
        for(int i=1;i<=s;++i){
            if(i!=x) q[i][bl[x]].push({c[x][bl[x]]-c[x][i],x});
        }
    }
    return dis[1];
}

signed main(){
	n=read();K=read();
	for(int i=1;i<=K;++i) e[i]=read();
    for(int i=1;i<=n;++i) for(int j=1;j<=K;++j) c[i][j]=read();
    
    int ans=0;
    for(int i=1;i<=n;++i){
        ans+=c[i][1];
        bl[i]=1;
    }
    for(int j=2;j<=K;++j){
        for(int i=1;i<=n;++i) q[j][bl[i]].push({c[i][bl[i]]-c[i][j],i}); // may change from bl[i] to j
        for(int i=1;i<=e[j];++i){
            for(int u=1;u<=j;++u) for(int v=1;v<=j;++v){
                while(!q[u][v].empty() && bl[q[u][v].top().se]!=v) q[u][v].pop();
            }
            ans+=spfa(j);
        }
        
    }
    cout<<ans<<hvie;
	return 0;
}
```

!!!note "[D.Directed](https://ac.nowcoder.com/acm/contest/33188/D)"

    有一棵树，可以随机选$k$条边定向至根方向，问随机定向k条边，从s出发随机游走到根的期望步数。

首先有个结论：从一个点随机游走走到父亲的期望步数是$2siz(x)-1$
然后如图

![image-20220810153215417](https://s1.328888.xyz/2022/08/10/4InSg.png)

```cpp
#include<bits/stdc++.h>
#define rep(i,a,b) for(int i=(a),i##ss=(b);i<=i##ss;i++)
#define dwn(i,a,b) for(int i=(a),i##ss=(b);i>=i##ss;i--)
#define rng(i,a,b) for(int i=(a);i<(b);i++)
#define deb(x) cerr<<(#x)<<":"<<(x)<<'\n'
#define pb push_back
#define mkp make_pair
#define fi first
#define se second
#define hvie '\n'
using namespace std;
typedef pair<int,int> pii;
typedef long long ll;
typedef unsigned long long ull;
typedef double db;
int yh(){
	int ret=0;bool f=0;char c=getchar();
	while(!isdigit(c)){if(c==EOF)return -1;if(c=='-')f=1;c=getchar();}
	while(isdigit(c))ret=(ret<<3)+(ret<<1)+(c^48),c=getchar();
	return f?-ret:ret;
}
const int maxn=1e6+5,mod=998244353;
/*
g(x)=1+\frac1{d(x)}\sum_{y\in son(x)}(g(y)+g(x))\\
d\cdot g(x)=d+\sum g(y)+(d-1)g(x)\\
g(x)=d+\sum g(y)=1+\sum (g(y)+1)
*/
/*
sz(x)=1+\sum sz(y)\\
2sz(x)-1=1+\sum((2sz(y)-1)+1)
g(x)=2sz(x)-1
*/
/*
定向一个边，对于其祖先链上的边来说，相当于删去一个子树，贡献为-2siz
要算的是从s到1的贡献
如果定向的边在链上，那对任意长度都会-2siz
可以把所有长度的贡献都算出来

然后从n-1中选k个，选出长度为i的边的方案数，考虑选中最下方的边，而且不选中上面的i-1条边，然后方案数是
C(n-1-i,k-1)
*/
vector<int>adj[maxn];
int n,k,s;
int sz[maxn],dep[maxn];
int nxt[maxn];
bool onChain[maxn];
ll sum[maxn];
ll fac[maxn],ifac[maxn],inv[maxn];
void dfs(int x,int fa){
	sz[x]=1;
	onChain[x]=(x==s);
	for(int y:adj[x])if(y!=fa){
		dep[y]=dep[x]+1;
		dfs(y,x);
		sz[x]+=sz[y];
		onChain[x]|=onChain[y];
	}
}
void dfs2(int x,int fa){
	if(onChain[x]) nxt[x]=x;
	else nxt[x]=nxt[fa];
	for(int y:adj[x])if(y!=fa){
		dfs2(y,x);
	}
	int l=dep[x]-dep[nxt[x]],r=dep[x];
	// cout<<x<<' '<<l<<' '<<r<<hvie;
	(sum[l]+=mod-2ll*sz[x]%mod)%=mod;
	(sum[r]+=2ll*sz[x]%mod)%=mod;
}
ll binom(int x,int y){
	if(x<y) return 0;
	return fac[x]*ifac[y]%mod*ifac[x-y]%mod;
}

signed main(){
	n=yh(),k=yh(),s=yh();
	fac[0]=ifac[0]=fac[1]=ifac[1]=inv[1]=1;
	rep(i,2,n){
		fac[i]=(ll)fac[i-1]*i%mod;
		inv[i]=(mod-(mod/i)*inv[mod%i]%mod)%mod;
		ifac[i]=ifac[i-1]*inv[i]%mod;
	}
	rep(i,1,n-1){
		int x=yh(),y=yh();adj[x].pb(y);adj[y].pb(x);
	}
	dfs(1,0);
	dfs2(1,0);
	rep(i,1,n) sum[i]=(sum[i-1]+sum[i])%mod;
	ll g=ifac[n-1]*fac[k]%mod*fac[n-1-k]%mod;
	ll ans=0;
	rep(i,1,n){
		if(onChain[i]&&i!=1) ans=(ans+2ll*sz[i]%mod+mod-1)%mod;
	}
	// cout<<ans<<hvie;
	rep(i,1,n-1){
		ll p=binom(n-1-i,k-1)*g%mod;
		ans=(ans+sum[i]*p%mod)%mod;
	}
	cout<<ans<<hvie;
	return 0;
}

```

!!!note "[G.Geometry](https://ac.nowcoder.com/acm/contest/33188/G)"

    给定两个凸包和初速度，问何时碰撞。

    凸包大小$\le 5e4$

设$P_1$相对$P_2$的速度是$v$,要求的实际上是$\min t,s.t. U\in P_1, V\in P_2, U+vt\in V, vt\in V-U$
即求$P_2$和$-P_1$的闵科夫斯基和。然后二分时间看是否在这个和的凸包里，注意初始就在和无解的情况。


```cpp
#include<bits/stdc++.h>
#define rep(i,a,b) for(int i=(a),i##ss=(b);i<=i##ss;i++)
#define dwn(i,a,b) for(int i=(a),i##ss=(b);i>=i##ss;i--)
#define rng(i,a,b) for(int i=(a);i<(b);i++)
#define deb(x) cerr<<(#x)<<":"<<(x)<<'\n'
#define pb push_back
#define mkp make_pair
#define fi first
#define se second
#define hvie '\n'
using namespace std;
typedef pair<int,int> pii;
typedef long long ll;
typedef unsigned long long ull;
typedef long double db;
int yh(){
	int ret=0;bool f=0;char c=getchar();
	while(!isdigit(c)){if(c==EOF)return -1;if(c=='-')f=1;c=getchar();}
	while(isdigit(c))ret=(ret<<3)+(ret<<1)+(c^48),c=getchar();
	return f?-ret:ret;
}
const int maxn=3e5+5;
const db eps=1e-10,inf=1e18;
int sgn(db x){
	if(x>eps) return 1;
	if(x<-eps) return -1;
	return 0;
}
struct vec{
	db x,y;
	vec(){}
	vec(db x,db y):x(x),y(y){}
};
ostream&operator<<(ostream&out,const vec&A){out<<'('<<A.x<<','<<A.y<<')';return out;}
vec operator+(const vec&A,const vec&B){return vec(A.x+B.x,A.y+B.y);}
vec operator-(const vec&A,const vec&B){return vec(A.x-B.x,A.y-B.y);}
vec operator*(const db&B,const vec&A){return vec(A.x*B,A.y*B);}
vec operator*(const vec&A,const db&B){return vec(A.x*B,A.y*B);}
vec operator/(const vec&A,const db&B){return vec(A.x/B,A.y/B);}
db cross(const vec&a,const vec&b){
	return a.x*b.y-a.y*b.x;
}
db dot(const vec&a,const vec&b){
	return a.x*b.x+a.y*b.y;
}
vec trunc(const vec &v){return vec(-v.y,v.x);}
db len(const vec &v){return hypot(v.x,v.y);}
vec normalize(const vec&v){
	return v/(len(v));
}
typedef vec point;
db xmult(point a,point b,point o){
	return cross(a-o,b-o);
}
struct line{
	vec s,e,v,n;
	db angle;
	line(){}
	line(vec a,vec b):s(a),e(b){
		v=b-a;
		angle=atan2(v.y,v.x);
		n=normalize(trunc(v));
	}
	bool operator<(const line &a )const{
		if(sgn(angle-a.angle)==0){
			return cross(v,a.e-s)>=0;
		}
		return angle<a.angle;
	}
};
bool onLeft(line L,point P){
	return cross(L.v,P-L.s)>0;
}
int onSegment(point P,point A,point B){
	if(sgn(xmult(A,B,P))) return 0;
	int d=sgn(dot(A-P,B-P));
	if(d==0) return 1;
	if(d<0) return 2;
	return 0;  
}
int segcrossseg(line a,line b){
	int d1=sgn(cross(a.e-a.s,b.s-a.s));
	int d2=sgn(cross(a.e-a.s,b.e-a.s));
	int d3=sgn(cross(b.e-b.s,a.s-b.s));
	int d4=sgn(cross(b.e-b.s,a.e-b.s));
	if((d1^d2)==-2&&(d3^d4)==-2) return 2;
	return (d1==0&&sgn(dot(b.s-a.s,b.s-a.e))<=0)||
		(d2==0&&sgn(dot(b.e-a.s,b.e-a.e))<=0)||
		(d3==0&&sgn(dot(a.s-b.s,a.s-b.e))<=0)||
		(d4==0&&sgn(dot(a.e-b.s,a.e-b.e))<=0);
}
point intersection(line A,line B){
	vec u=B.s-A.s;
	db dt=cross(A.v,B.v);
	if(sgn(dt)==0) return point(-inf,-inf);
	db t=cross(u,B.v)/dt;
	return A.s+A.v*t;
}

typedef vector<point>Polygon;

const point O(0,0);

line ls[maxn],hpi[maxn];
bool onRight(line a,line b,line c){
	point i=intersection(b,c);
	return sgn(cross(a.v,i-a.s))<0;
}
void HalfPlaneIntersection(vector<line>&L){
	sort(L.begin(),L.end());
	int head=0,tail=0,cnt=0;
	int n=L.size();
	for(int i=0;i<n-1;i++){
		if(sgn(L[i].angle-L[i+1].angle)==0) continue;
		ls[cnt++]=L[i];
	}
	ls[cnt++]=L[n-1];
	for(int i=0;i<cnt;i++){
		while(tail-head>1&&onRight(ls[i],hpi[tail-1],hpi[tail-2])) tail--;
		while(tail-head>1&&onRight(ls[i],hpi[head],hpi[head+1])) head++;
		hpi[tail++]=ls[i];
	}
	while(tail-head>1&&onRight(hpi[head],hpi[tail-1],hpi[tail-2])) tail--;
	while(tail-head>1&&onRight(hpi[tail-1],hpi[head],hpi[head+1])) head++;
	L.clear();
	if(tail-head<3) return;
	for(int i=head;i<tail;i++){
		L.pb(hpi[i]);
		// cout<<hpi[i].s<<"--->"<<hpi[i].e<<endl;
	}
	// cout<<L.size()<<hvie;
}

db Section(const Polygon &P){
	db ans=0;
	int n=P.size();
	for(int i=0;i<n;i++){
		ans+=cross(P[i],P[(i+1)%n]);
	}
	return fabs(ans)/2.0;
}

void input(int &n,Polygon&A){
	n=yh();
	db x,y;
	rep(i,0,n-1){
		x=yh(),y=yh();
		A.pb(point(x,y));
	}
}
Polygon Mincowski(const Polygon&a,const Polygon&b){
	Polygon c,u,v;
	int n=a.size(),m=b.size();
	for(int i=0;i<n;i++){
		u.pb(a[(i+1)%n]-a[i]);
	}
	for(int i=0;i<m;i++){
		v.pb(b[(i+1)%m]-b[i]);
	}
	c.pb(a[0]+b[0]);
	int i=0,j=0;
	while(i<n&&j<m){
		if(i<n&&sgn(cross(u[i],v[j]))>=0) c.pb(c.back()+u[i++]);
		else c.pb(c.back()+v[j++]);
	}
	while(i<n) c.pb(c.back()+u[i++]);
	while(j<m) c.pb(c.back()+v[j++]);
	return c;
}
void MakeConvex(Polygon &C){
	static int st[maxn];
	int tot=0;
	sort(C.begin(),C.end(),[&](const point &a,const point&b){return a.y<b.y||(a.y==b.y&&a.x<b.x);});
	point base=C[0];
	st[++tot]=0;
	for(auto &p:C) p=p-base;
	sort(C.begin()+1,C.end(),[&](const point&a,const point&b){return cross(a,b)>0||(cross(a,b)==0&&dot(a,a)<dot(b,b));});
	for(int i=1;i<(int)C.size();st[++tot]=i++){
		while(tot>1&&xmult(C[st[tot]],C[i],C[st[tot-1]])<=0){
			tot--;
		}
	}
	for(int i=0;i<tot;i++){
		C[i]=C[st[i+1]]+base;
	}
	C.resize(tot);
}
int n,m,K;
Polygon A,B,C;
point va,vb;

bool in(point p){
	rep(i,0,K-1){
		if(sgn(xmult(C[i],C[(i+1)%K],p))<0) return 0;
	}
	return 1;
}
bool check(line l){
	rep(i,0,K-1){
		if(segcrossseg(l,line(C[(i+1)%K],C[i]))) return true;
	}
	return false;
}

signed main(){
	input(n,A);
	input(m,B);
	MakeConvex(A);
	rep(i,0,m-1) B[i]=B[i]*(-1);
	MakeConvex(B);
	C=Mincowski(A,B);
	MakeConvex(C);
	K=C.size();
	va.x=yh(),va.y=yh();
	vb.x=yh(),vb.y=yh();
	vb=vb-va;
	if(in(point(0,0))){
		cout<<"0\n";return 0;
	}
	if(!check(line(point(0,0),vb*2e9))){
		cout<<"-1\n";return 0;
	}
	db l=0,r=2e9;
	rep(_,1,66){
		db mid=(l+r)/2.0;
		if(check(line(point(0,0),vb*mid))){
			r=mid;
		}
		else l=mid;
	}
	cout<<fixed<<setprecision(10)<<l<<hvie;
	return 0;
}
```
