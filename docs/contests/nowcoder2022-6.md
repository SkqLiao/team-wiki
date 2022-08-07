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
