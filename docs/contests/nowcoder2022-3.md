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