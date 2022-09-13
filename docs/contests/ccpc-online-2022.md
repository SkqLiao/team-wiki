!!!note "[G Name the Puppy]()"

    给定$n$个字符串，现在可以将这些字符串以任意顺序拼接，每个字符串可以使用任意多次。问最长的anti-border有多长，无限长输出INF。

    anti-border的定义是前缀reverse后能匹配后缀。

    $n\leq 5000,\sum|s|\leq 5000$

首先我们考虑border长度没有跨越答案串的中心，这种情况实际上是比较好办的，我们要做的实际上是把所有串reverse以后进行匹配。

对正串和反串分别建出Trie树，然后设$f[x][y]$表示正串匹配到节点$x$，反串匹配到节点$y$的最长长度，然后记忆化搜索匹配即可，注意如果走到了一个结尾的位置，可以转移到初始节点。如果重复走到一个节点，实际上答案就是INF了（只需要一直循环这个路线即可）。

接下来考虑border长度跨越了答案串的中心，看上去是一个很特殊的情况，实际上稍微想一下就发现并不存在这样的情况。因为这个答案串必然是一个回文串，那么我们不断重复这个串就可以得到INF了，而在前面的DP过程中，我们事实上在原本答案串中间的部分不断重复就可以让匹配无限延长，这样就是INF了。（大概就是一定能遍历到一个对应的情况）

复杂度$O((\sum|s|)^2\times \sum)$

```cpp
#include<bits/stdc++.h>
#include <random>
#define pb push_back
#define mkp make_pair
#define fi first
#define se second
#define int long long
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

const int N=5010,inf=0x3f3f3f3f,mod=998244353;

int n,ans,f[N][N];
char st[N];

struct Trie{
    int cnt=1;
    int ch[N][26],ed[N];
    void insert(char *s,int m){
        int now=1;
        for(int i=1;i<=m;++i){
            int x=s[i]-'a';
            if(!ch[now][x]) ch[now][x]=++cnt;
            now=ch[now][x];
        }
        ed[now]=1;
    }
}T1,T2;

void hvfun(){
    puts("INF");
    exit(0);
}

void dfs(int x,int y){
    //cout<<x<<" "<<y<<" "<<f[x][y]<<hvie;
    ans=max(ans,f[x][y]);
    for(int i=0;i<26;++i){
        if(!T1.ch[x][i] || !T2.ch[y][i]) continue;
        if(f[T1.ch[x][i]][T2.ch[y][i]]) hvfun();
        f[T1.ch[x][i]][T2.ch[y][i]]=f[x][y]+1;
        dfs(T1.ch[x][i],T2.ch[y][i]);
    }
    if(T1.ed[x]){
        if(f[1][y]) hvfun();
        f[1][y]=f[x][y];
        dfs(1,y);
    }
    if(T2.ed[y]){
        if(f[x][1]) hvfun();
        f[x][1]=f[x][y];
        dfs(x,1);
    }
    if(T1.ed[x] && T2.ed[y]) hvfun();
}

void solve(){
    n=read();
    for(int i=1,m;i<=n;++i){
        scanf("%s",st+1);
        m=strlen(st+1);
        T1.insert(st,m);
        reverse(st+1,st+m+1);
        T2.insert(st,m);
    }
    dfs(1,1);
    cout<<ans<<hvie;
}

signed main() {
    for (int cas = 1; cas--;) {
        solve();
    }
    return 0;
}

/*
4
ab
cbba
bccd
eddc
3
abcdefg
gfe
dcba

*/
```


