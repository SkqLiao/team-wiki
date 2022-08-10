!!!note "[C. Forest](https://ac.nowcoder.com/acm/contest/33191/C)"
    
    求一个图所有的生成子图的生成森林的权值的和。$n\le 16$

先将边按权值排序，考虑每一条边计入答案的方案数，即那些编号小于他的边加入后他的两个端点还没有连通的方案数。一个朴素的做法是枚举包含u和v的集合S，并枚举子集使得包含u但不包含v。然后算出使得这两个分别连通的方案数，并乘上随便选那些还没考虑到的边，和不包含在S内的边的方案数，这样复杂度是3^n的。

然后考虑利用集合幂级数优化。集合幂级数是形如$\sum_{s\subseteq U} f(s)x^s$的玩意，注意其指数为集合。考虑如何定义它的乘法，就是要用集合的运算来定义。设$*$为集合幂级数的乘法，$\cdot$为集合的运算。

$F*G=\sum_{s\subseteq U}\sum_{t\subseteq U}f(s)g(t)x^{s\cdot t}$。

如果$\cdot$是集合并运算，那么就可以做如下转换：

令$\hat F=\sum_{s\subseteq U}x^s\sum_{i\subseteq s}f(i)$
这个变换被称为莫比乌斯变换。

那么$F*G=\sum_{s\subseteq U}x^s\sum_{i \subseteq s}f(i)\sum_{j\subseteq s}g(j)=\sum_{s\subseteq U} x^s[x^s]\hat F[x^s]\hat G$

所以两个幂级数求乘积相当于进行莫比乌斯变换过后再对各个集合求点积，再逆莫比乌斯变换回去。

原题要求的是子集卷积。

那就需要记录每个集合的时候顺便记录popcount，即$F(|S|,S)$，然后对每个FMT过后的集合，相当于有一个数组记录了原集合中popcount为i的那些点值和，对于集合$S$和$T$，只取集合并卷积中popcount为$|S|+|T|$的那些值，这可以看做固定$|S|$,$|T|$后（关于集合的点积意义下），进行多项式卷积。

总结一下：如果将集合幂级数乘法定义为子集卷积，即FMT后对每个集合进行多项式卷积，IFMT后取F(|S|,S)，同理任何多项式函数也是一样。

对于原题，需要求一个使得$S$连通的方案数，$G(x)=e^F$即为任意选边的方案数，而$f^{\phi}=1$，$f(S)=2^{S包含的边数}$，所以需要求的是F的ln，照常进行FMT，进行点积意义下的ln(1+x)，这里由于项数只有popcount个，考虑暴力求：
$$
F'=\ln'(G+1)=\frac{G'}{G+1}\\
F'=G'-F'G\\
f_i=g'[i]-\sum_{j=1}^n f_ig[i]
$$
这样可以$n^2$求出多项式ln。

这样，每考虑进一条边，都重新算一下G，再进行FMT后ln的操作后取F[|S|][S]，即可得到$S$连通的方案，从而只需要$2^{已加入边数}$-这个结果就可以了。

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
const int maxn=3e5+5,mod=998244353,inv2=(mod+1)/2;
int add(int x,int y){x+=y;return x>=mod?x-mod:x;}
int mul(ll x,ll y){return x*y%mod;}
void FMT(int *a,int n){
	for(int mid=1;mid<n;mid<<=1){
		for(int R=mid<<1,j=0;j<n;j+=R){
			for(int k=0;k<mid;k++){
				a[j+k+mid]=add(a[j+k+mid],a[j+k]);
			}
		}
	}
}
void IFMT(int *a,int n){
	for(int mid=1;mid<n;mid<<=1){
		for(int R=mid<<1,j=0;j<n;j+=R){
			for(int k=0;k<mid;k++){
				a[j+k+mid]=add(a[j+k+mid],mod-a[j+k]);
			}
		}
	}
}
int inv[maxn];
int cnte[maxn];
int n,mask,pop[maxn],G[20][maxn];
void recalc(){
	memset(G,0,sizeof(G));
	rep(i,1,mask){
		G[pop[i]][i]=cnte[i];
	}
	rep(i,0,n) FMT(G[i],1<<n);
	static int a[20],f[20];
	rep(S,1,mask){
		rep(i,0,n) a[i]=G[i][S];
		rep(i,0,n-1){
			int res=0;
			for(int j=0;j<i;j++){
				res=add(res,mul(f[j], a[i-j]));
			}
			f[i]=add(mul(a[i+1],i+1),mod-res);
		}
		rep(i,1,n){
			G[i][S]=mul(f[i-1],inv[i]);
		}
	}
	rep(i,0,n) IFMT(G[i],1<<n);

}
signed main(){
	inv[1]=1;
	rep(i,2,100){
		inv[i]=mod-mul(mod/i,inv[mod%i]);
	}

	n=yh(),mask=(1<<n)-1;
	rep(i,0,mask) pop[i]=pop[i>>1]+(i&1),cnte[i]=1;
	vector<array<int,3>>v;
	int pwe=1;
	rep(i,0,n-1)rep(j,0,n-1){
		int x=yh();
		if(x&&i<j){
			v.pb({x,i,j});
			pwe=2ll*pwe%mod;
		}
	}
	sort(v.begin(),v.end());
	int ans=0;
	for(auto [w,x,y]:v){
		// cout<<w<<' '<<x<<' '<<y<<endl;
		pwe=mul(pwe,inv2);
		recalc();
		int res=0;
		rep(S,1,mask)if((S>>x&1)&&(S>>y&1)){
			res=add(res,mul(G[pop[S]][S],cnte[mask^S]));
		}
		res=add(cnte[mask],mod-res);
		// cout<<res<<endl;
		ans=add(ans, mul(res, mul(pwe, w)));
		rep(S,0,mask)if((S>>x&1)&&(S>>y&1)){
			cnte[S]=mul(cnte[S],2);
		}
	}
	cout<<ans<<hvie;
	return 0;
}
```


!!!note "[F. Hash](https://ac.nowcoder.com/acm/contest/33191/F)"

	定义一棵树$T$的哈希值$F(T)$为:$F(T)=\sum_{i=1}^n\sum_{j=i+1}^n X^iY^jZ^{\text{lca}(i,j)}$ 对$998244353$取模的结果。

    现在给定$F(T),X,Y,Z$，构造一棵不超过50个点的树满足这个哈希值，$X,Y,Z$均随机。

    首先这个随机就很耐人寻味，事实上，我们可以认为$X^k,Y^k,Z^k$这堆东西都是不一样的。一个朴素的想法是，我们对于每个点对$(i,j)$，有$n$个可选的$\text{lca}$，相当于每个点对在$n$个取值里选择一个，使得哈希值符合条件且能构造出这样一棵树。由于随机的特性，这样几乎一定能构造出来，但是复杂度显然是不对的。事实上，我们观察到这个方法在做背包的时候，可能产生的值远超过$998244353$个，这启发我们缩小背包的规模，用更简单的构造方式。

    我们考虑构造一个$n=49$的树，一开始所有点的父亲都是$1$。然后将$2\sim 49$每$8$个点分为一组，每组中选择$x<y<z$的三个点，将$y$和$z$的父亲改为$x$，其他点的父亲都为$1$，这样会产生一个哈希值的增量。对于每一组，方案数是$\binom {8} {3}=56$的，那么总的可能的哈希值的增量实际上就是$56^6$的，这远大于模数的范围，我们可以认为这几乎一定可以构造出来。

    事实上，这里选取$x<y<z$是为了方便计算，如果是每组中任选三个点可以让点数的规模更小。


```cpp
#include<bits/stdc++.h>
#include <random>
#define pb push_back
#define mkp make_pair
#define fi first
#define se second
#define hvie '\n'
#define int long long
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

const int N=55,inf=0x3f3f3f3f,mod=998244353;

int X[N],Y[N],Z[N];
vector<array<int,3>>ord;

int upm(int x){
    return (x%mod+mod)%mod;
}

void solve() {
    int fans=read(),n=49;
    X[0]=Y[0]=Z[0]=1;
    X[1]=read(),Y[1]=read(),Z[1]=read();
    for(int i=2;i<N;++i){
        X[i]=X[i-1]*X[1]%mod;
        Y[i]=Y[i-1]*Y[1]%mod;
        Z[i]=Z[i-1]*Z[1]%mod;
    }
    
    int now=0;
    for(int i=1;i<=n;++i){
        for(int j=i+1;j<=n;++j){
            now+=X[i]*Y[j]%mod*Z[1]%mod;
            now%=mod;
        }
    }

    vector<pii>vec[6];
    for(int i=2,c=0;i<=n;i+=8,++c){
        int id=0;
        for(auto v:ord){
            int x=v[0]+i,y=v[1]+i,z=v[2]+i;//change y,z's fa to x
            int tmp=(X[y]*Y[z]%mod*(Z[x]-Z[1])%mod+X[x]*Y[y]%mod*(Z[x]-Z[1])%mod+X[x]*Y[z]%mod*(Z[x]-Z[1])%mod)%mod;
            tmp=upm(tmp);
            vec[c].pb(mkp(tmp,id));
            ++id;
        }
    }

    unordered_map<int,array<int,3>>mp;
    int cnt=0;
    for(auto [i,idi]:vec[0]) for(auto [j,idj]:vec[1]) for(auto [k,idk]:vec[2]){
        mp[(i+j+k)%mod]={idi,idj,idk};
        ++cnt;
    }
    //cout<<cnt<<hvie;
    for(auto [i,idi]:vec[3]) for(auto [j,idj]:vec[4]) for(auto [k,idk]:vec[5]){
        int tmp=(i+j+k)%mod,hs=upm(mod-(now+tmp-fans));
        if(mp.count(hs)){
            cout<<n<<hvie;
            auto t=mp[hs];
            for(int m=0;m<3;++m){
                for(int p=0;p<8;++p){
                    if(p!=ord[t[m]][1] && p!=ord[t[m]][2]){
                        cout<<"1 "<<p+m*8+2<<hvie;
                    }
                }
                cout<<ord[t[m]][0]+m*8+2<<" "<<ord[t[m]][1]+m*8+2<<hvie;
                cout<<ord[t[m]][0]+m*8+2<<" "<<ord[t[m]][2]+m*8+2<<hvie;
            }

            for(int p=0;p<8;++p){
                if(p!=ord[idi][1] && p!=ord[idi][2]){
                    cout<<"1 "<<p+26<<hvie;
                }  
            }
            cout<<ord[idi][0]+26<<" "<<ord[idi][1]+26<<hvie;
            cout<<ord[idi][0]+26<<" "<<ord[idi][2]+26<<hvie;

            for(int p=0;p<8;++p){
                if(p!=ord[idj][1] && p!=ord[idj][2]){
                    cout<<"1 "<<p+34<<hvie;
                }
            }
            cout<<ord[idj][0]+34<<" "<<ord[idj][1]+34<<hvie;
            cout<<ord[idj][0]+34<<" "<<ord[idj][2]+34<<hvie;

            for(int p=0;p<8;++p){
                if(p!=ord[idk][1] && p!=ord[idk][2]){
                    cout<<"1 "<<p+42<<hvie;
                }
            }
            cout<<ord[idk][0]+42<<" "<<ord[idk][1]+42<<hvie;
            cout<<ord[idk][0]+42<<" "<<ord[idk][2]+42<<hvie;

            return;
        }
    }
}

void init(){
    for(int i=0;i<8;++i) for(int j=i+1;j<8;++j) for(int k=j+1;k<8;++k){
        ord.pb({i,j,k});
    }
}

signed main() {
    init();
    for (int cas = read(); cas--;) {
        solve();
    }

    return 0;
}
```
