---
title: 数位DP学习整理
date: 2021-05-12
top: true
tags: [数位DP,C++]
categories: 算法
mathjax: true
---
# 数位DP

## 1 数位DP介绍

数位DP往往都是这样的题型，给定一个闭区间$[l,r]$，让你求这个区间中满足某种条件的数的总数。而这个区间可能很大，简单的暴力代码如下：

```cpp
int ans=0;
for(int i=l;i<=r;i++){
    if(check(i))ans++;
}
```

我们发现，若区间长度超过$1e8$，我们暴力枚举就会超时了，而数位$DP$则可以解决这样的题型。数位$DP$实际上就是在数位上进行$DP$。

## 2 数位DP解法

数位$DP$就是换一种暴力枚举的方式，使得新的枚举方式符合$DP$的性质，然后预处理好即可。我们来看：我们可以用$f(n)$表示$[0,n]$的所有满足条件的个数，那么对于$[l,r]$我们就可以用$[l,r]\iff f(r)-f(l-1)$，相当于前缀和思想。那么也就是说我们只要求出$f(n)$即可。那么数位$DP$关键的思想就是从树的角度来考虑。将数拆分成位，从高位到低位开始枚举。我们可以视$N$为$n$位数，那么我们拆分$N:a_{n}、a_{n-1}...a_1$。那么我们就可以开始分解建树，如下。之后我们就可以预处理再求解$f(n)$了，个人认为求解$f(n)$是最难的一步。

![](https://img-blog.csdnimg.cn/img_convert/cfc37954b97e29aa1d848925105fec1b.png)

听完是不是有点绕，我们可以来点题目练习一下，做完就会发现了数位$DP$的套路了。

## 3 数位DP经典例题

### [3.1 度的数量](https://www.acwing.com/activity/content/11/)

* **题面**

  >求给定区间$ [X,Y]$ 中满足下列条件的整数个数：这个数恰好等于 $K$ 个互不相等的 $B$ 的整数次幂之和。例如，设 $X=15,Y=20,K=2,B=2$，则有且仅有下列三个数满足题意：
  >$17=2^4+2^0$
  >$18=2^4+2^1$
  >$20=2^4+2^2$
  >输入格式
  >第一行包含两个整数 X 和 Y，接下来两行包含整数 $K$ 和 $B$。
  >输出格式
  >只包含一个整数，表示满足条件的数的个数。
  >数据范围
  >$1≤X≤Y≤2^{31}−1,$
  >$1≤K≤20,$
  >$2≤B≤10$
  >输入样例：
  >15 20
  >2
  >2
  >输出样例：
  >3

* **解题思路**

  此题实际上就是将十进制数转化为$B$进制数，判断位数上的值是否为$1$。那么我们可以视$N$为$n$位数，那么我们拆分$N:a_{n}、a_{n-1}...a_1$。从树的角度考虑：我们设$N=76543210,B=10$，那么我们从高位往最低位开始枚举如下；枚举$a_n$时，我们有两种选择：

  1. 走右边分支，那么我们填$7(a_n)$，而题目要求每一位只能填$1$或者$0$,而$a_n>1$，所以不是合法方案，我们直接剔除。
  2. 走左边分支，那么我们可以填$0$~$6$，即$0-{a_n}-1$，那么由于每一位只能填$1$或者$0$，所以我们累加这两种选择的方案。
  
  记住，走到了左边分支是可以直接累加的。
  
  所以我们实际上还是要做一个预处理的，我们用$f[i][j]$表示还剩下$i$位没有填，且需要填写$j$个$1$的方案数。那么在$(i,j)$这个状态，我们可以选择填$1$，那么接下来的状态就是$f[i-1][j-1]$，而如果填$0$，那么接下来的状态就是$f[i-1][j]$，那么状态转移方程就是$f[i][j]=f[i-1][j]+f[i][j-1]$。而初始状态即是当$j=0$时，$f[i][0]=1$。这样我们就可以预处理$f$数组了。
  
  处理完之后我们就可以直接模拟做了。

* **代码**

```cpp
/**
  *@filename:度的数量
  *@author: pursuit
  *@csdn:unique_pursuit
  *@email: 2825841950@qq.com
  *@created: 2021-05-12 11:23
**/
#include <bits/stdc++.h>

using namespace std;

typedef long long ll;
const int maxn = 100000 + 5;
const int mod = 1e9+7;

int l,r,k,b;
int f[35][35];
//首先我们先预处理f数组。其中f[i][j]表示剩下还有i个没填，需要填写j个1的方案数。
void init(){
    for(int i=0;i<35;i++){
        for(int j=0;j<=i;j++){
            if(!j)f[i][j]=1;
            else{
                f[i][j]=f[i-1][j]+f[i-1][j-1];
            }
        }
    }
}
int dp(int n){
    //求解f(n)。我们需要避免n为0的情况，这里需要特判。
    if(!n)return 0;
    vector<int> nums;//将n分割，存储位数。
    while(n){
        nums.push_back(n%b);
        n/=b;
    }
    int ans=0;//答案。
    int last=0;//前面的信息，这里代表的是前面分支选取了多少个1.
    for(int i=nums.size()-1;i>=0;i--){
        int x=nums[i];
        if(x){
            //说明x>0，我们可以选择左边分支填0.
            ans+=f[i][k-last];
            if(x>1){
                //当x>1我们才可以枚举左边分支填1.
                if(k-last-1>=0){
                    //如果还可以填1的话。
                    ans+=f[i][k-last-1];
                }
                break;//因为右边分支只能为0或1，所以不符合条件。break。
            }
            else{
                //当x=1就可以进入右边的分支继续讨论。
                last++;
                if(last>k)break;
            }
        }
        //考虑到最后一位，如果符合条件那么末位填0也算一种方案。
        if(!i&&last==k)ans++;
    }
    return ans;
}
void solve(){
}
int main(){
    cin>>l>>r>>k>>b;
    init();
    cout<<dp(r)-dp(l-1)<<endl;
    solve();
    return 0;
}
```


### [3.2 计数问题](https://www.acwing.com/problem/content/340/)

* **题面**

  >给定两个整数 a 和 b，求 a 和 b 之间的所有数字中 0∼90∼9 的出现次数。
  >
  >例如，$a=1024，b=1032$，则 a 和 b 之间共有 99 个数如下：
  >
  >```
  >1024 1025 1026 1027 1028 1029 1030 1031 1032
  >```
  >
  >其中 `0` 出现 10次，`1` 出现 10 次，`2` 出现 7 次，`3` 出现 3 次等等…
  >
  >输入格式
  >
  >输入包含多组测试数据。
  >
  >每组测试数据占一行，包含两个整数 a 和 b。
  >
  >当读入一行为 `0 0` 时，表示输入终止，且该行不作处理。
  >
  >输出格式
  >
  >每组数据输出一个结果，每个结果占一行。
  >
  >每个结果包含十个用空格隔开的数字，第一个数字表示 `0` 出现的次数，第二个数字表示 `1` 出现的次数，以此类推。
  >
  >数据范围
  >
  >$0<a,b<1000000000<a,b<100000000$
  >
  >输入样例：
  >
  >```c++
  >1 10
  >44 497
  >346 542
  >1199 1748
  >1496 1403
  >1004 503
  >1714 190
  >1317 854
  >1976 494
  >1001 1960
  >0 0
  >```
  >
  >输出样例：
  >
  >```c++
  >1 2 1 1 1 1 1 1 1 1
  >85 185 185 185 190 96 96 96 95 93
  >40 40 40 93 136 82 40 40 40 40
  >115 666 215 215 214 205 205 154 105 106
  >16 113 19 20 114 20 20 19 19 16
  >107 105 100 101 101 197 200 200 200 200
  >413 1133 503 503 503 502 502 417 402 412
  >196 512 186 104 87 93 97 97 142 196
  >398 1375 398 398 405 499 499 495 488 471
  >294 1256 296 296 296 296 287 286 286 247
  >```

* **解题思路**

  ![](https://img-blog.csdnimg.cn/img_convert/d9cff4c573361692ac8f6cef1b47a916.png)

  我们需要预处理$f$数组，那么我们可以用$f[i,j,u]$表示$i$位，最高位为$j$的数拥有$u$的个数。那么如果$j$不等于$u$时，则$f[i][j][u]+=f[i-1][k][u],0\leq k \leq 9$。这个应该不难理解，因为这个状态就是由之前的状态得到的。 而当$j$等于$u$时，那么同样也可以由之前的$9$个状态得到。为$f[i][j][u]+=f[i-1][k][u],0\leq k \leq 9$。记住，我们是还没有计算最高位的$u$个数的，因为最高位本身就为$u$，也是一种可能，所以我们需要加上。那么总共有$10^{i-1}$多的数，所以增加的u的数量为$10^{i-1}$。初始状态就是$f[1][i][i]=1$，到这，我们的$f$数组就初始化完了，那么接下来。就是拆位分支的数位$DP$套路讨论了，这里不在叙述，代码附详细注释。

* **代码**

```cpp
/**
  *@filename:计数问题
  *@author: pursuit
  *@csdn:unique_pursuit
  *@email: 2825841950@qq.com
  *@created: 2021-05-12 13:12
**/
#include <bits/stdc++.h>

using namespace std;

typedef long long ll;
const int maxn = 100000 + 5;
const int mod = 1e9+7;

int l,r;
int f[11][10][10];//预处理f数组。其中f[i][j][u]表示i位最高位为j的数拥有u的个数。
void init(){
    for(int i=0;i<10;i++)f[1][i][i]=1;
    for(int i=2;i<11;i++){
        for(int j=0;j<10;j++){
            for(int u=0;u<10;u++){
                //判断j是否等于u。
                if(j==u)f[i][j][u]+=pow(10,i-1);
                for(int k=0;k<10;k++){
                    f[i][j][u]+=f[i-1][k][u];
                }
            }
        }
    }
}
ll dp(int n,int u){
    //1~n,求x的出现次数。
    if(!n)return u?0:1;//特判n是否为0.根据u的值确定返回值。
    vector<int> nums;//存储分割后的位数。
    while(n)nums.push_back(n%10),n/=10;
    int last=0;//last记录前面u出现的次数。
    ll ans=0;//答案。
    for(int i=nums.size()-1;i>=0;i--){
        int x=nums[i];
        //左边分支，0~x。
        for(int j=(i==nums.size()-1);j<x;j++){
            //由于此题不能有前导0.
            ans+=f[i+1][j][u];//注意这里i需要+1，因为我们i下标从0开始。而位数从1开始。
        }
        //走左边分支，那么我们需要加上前面的个数。注意这里需要乘上x，因为左边分支有x中选择。
        ans+=x*last*pow(10,i);
        if(x==u)last++;//记录last。
        if(!i)ans+=last;//加上这个数本身含有的。
    }
    //由于我们前面都是枚举n位数的，我们还需要统计所有0~n-1位数的方案数量。
    //例如000011是不合法的，但11是合法的。
    //这一步确实很容易忽略，没办法，数位DP就是这么难。
    for(int i=1;i<nums.size();i++){
        for(int j=(i!=1);j<=9;j++){
            ans+=f[i][j][u];
        }
    }
    return ans;
}
void solve(){
}
int main(){
    init();
    while(cin>>l>>r&&(l||r)){
        if(l>r)swap(l,r);
        for(int i=0;i<=9;i++){
            cout<<dp(r,i)-dp(l-1,i);
            i==9?cout<<endl:cout<<" ";
        }
    }
    solve();
    return 0;
}
```

### [3.3 数字游戏](https://www.acwing.com/activity/content/11/)

* **题面**

  >科协里最近很流行数字游戏。
  >
  >某人命名了一种不降数，这种数字必须满足从左到右各位数字呈非下降关系，如 123，446。
  >
  >现在大家决定玩一个游戏，指定一个整数闭区间 [a,b]，问这个区间内有多少个不降数。
  >
  >**输入格式**
  >输入包含多组测试数据。
  >
  >每组数据占一行，包含两个整数 a 和 b。
  >
  >**输出格式**
  >每行给出一组测试数据的答案，即 [a,b] 之间有多少不降数。
  >
  >**数据范围**
  >$1≤ a ≤ b ≤2^{31}−1$
  >
  >**样例输入**
  >1 9 
  >1 19
  >**样例输出**
  >9
  >18

* **解题思路**

  同样的套路， 先预处理$f$数组，我们用$f[i][j]$表示$i$位数，且最高位为$j$的不降数方案数。那么我们来列写一下状态转移方程，对于$f[i][j]$，要满足不降数的要求，则$f[i-1][k]$，$k$需满足$j \leq k \leq 9$，那么自然$f[i][j]=\sum_{k=j}^9 f[i-1][k]$。而初始状态自然是$f[1][j]=1$。预处理完之后，我们就好做了，直接按数位$DP$的思想处理即可。代码附详细注释。

* **代码**

```cpp
/**
  *@filename:数字游戏
  *@author: pursuit
  *@csdn:unique_pursuit
  *@email: 2825841950@qq.com
  *@created: 2021-05-12 14:57
**/
#include <bits/stdc++.h>

using namespace std;

typedef long long ll;
const int maxn = 100000 + 5;
const int mod = 1e9+7;

int l,r;
int f[11][11];//预处理f数组。其中f[i][j]表示i位数，且最高位为j的不降数方案数。
void init(){
    for(int i=1;i<10;i++)f[1][i]=1;
    for(int i=2;i<11;i++){
        for(int j=0;j<10;j++){
            for(int k=j;k<10;k++){
                f[i][j]+=f[i-1][k];
            }
        }
    }
}
int dp(int n){
    //1~n，这里我们需要特判n=0。
    if(!n)return 0;
    vector<int> nums;//存储分割位数。
    while(n)nums.push_back(n%10),n/=10;
    int last=0;//last存储上一位的最大值。
    int ans=0;//答案。
    for(int i=nums.size()-1;i>=0;i--){
        int x=nums[i];
        //走左边的分支。因为要保持不降序，所以我们j>=last。
        for(int j=last;j<x;j++){
            ans+=f[i+1][j];//注意是i+1位。
        }
        if(last>x)break;//说明上一位比x大，不能构成降序了，直接退出。
        last=x;//走右分支了，更新last。
        if(!i)ans++;//全部枚举完了，自身也同样构成了一种方案。
    }
    return ans;
}
int main(){
    init();
    while(cin>>l>>r){
        cout<<dp(r)-dp(l-1)<<endl;
    }
    return 0;
}
```

  

### [3.4 windy数](https://www.luogu.com.cn/problem/P2657)

* **题面**

  >windy 定义了一种 windy 数。
  >
  >不含前导零且相邻两个数字之差至少为 2 的正整数被称为 windy 数。windy 想知道，在 a 和 b 之间，包括 *a* 和 *b* ，总共有多少个 windy 数？
  >
  >**输入格式**
  >
  >输入只有一行两个整数，分别表示 *a* 和 *b*。
  >
  >**输出格式**
  >
  >输出一行一个整数表示答案。
  >
  >**输入输出样例**
  >
  >**输入 #1**
  >
  >```
  >1 10
  >```
  >
  >**输出 #1**
  >
  >```
  >9
  >```
  >
  >**输入 #2**
  >
  >```
  >25 50
  >```
  >
  >**输出 #2**
  >
  >```
  >20
  >```
  >
  >**说明/提示**
  >
  >**数据规模与约定**
  >
  >对于全部的测试点，保证 $1 \leq a \leq b \leq 2 \times 10^9$。

* **解题思路**

  同样，我们先进行预处理$f$数组，其中$f[i][j]$表示$i$位，其中最高位为$j$的方案数。那么根据题意，状态转移方程即为$f[i][j]=\sum_{}f[i-1][k]$，其中$0 \leq k \leq 9 \space and \space abs(k-j)>=2$。而初始状态即为$dp[1][i]=1$。预处理完之后就好处理了，这里不再提供思路，请大家自己画出树结构并完成此题。

* **代码**

```cpp
/**
  *@filename:windy数
  *@author: pursuit
  *@csdn:unique_pursuit
  *@email: 2825841950@qq.com
  *@created: 2021-05-12 15:43
**/
#include <bits/stdc++.h>

using namespace std;

typedef long long ll;
const int maxn = 100000 + 5;
const int mod = 1e9+7;

int l,r;
int f[11][10];//f数组。其中f[i][j]表示i位，其中最高位为j的方案数。
void init(){
    for(int i=0;i<10;i++)f[1][i]=1;
    for(int i=2;i<11;i++){
        for(int j=0;j<10;j++){
            for(int k=0;k<10;k++){
                if(abs(k-j)>=2){
                    f[i][j]+=f[i-1][k];
                }
            }
        }
    }
}
int dp(int n){
    if(!n){
        //特判n为0的情况，避免对之后操作造成影响。
        return 0;
    }
    vector<int> nums;//存储分割位数。
    int last=-2;//存储上一位的值。这里初值为-2,是因为我们需要确定1可以。
    int ans=0;//答案。
    while(n)nums.push_back(n%10),n/=10;
    for(int i=nums.size()-1;i>=0;i--){
        int x=nums[i];
        //左分支。
        for(int j=(i==nums.size()-1);j<x;j++){
            if(abs(j-last)>=2){
                //说明符合要求。
                ans+=f[i+1][j];
            }
        }
        if(abs(x-last)<2)break;//不满足要去。
        last=x;
        if(!i)ans++;//枚举到最后一位，自身也形成了一种方案。
    }
    //特殊枚举有前导0的数。
    for(int i=1;i<nums.size();i++){
        for(int j=1;j<=9;j++){
            ans+=f[i][j];
        }
    }
    return ans;
}
void solve(){
}
int main(){
    init();
    cin>>l>>r;
    cout<<dp(r)-dp(l-1)<<endl;
    solve();
    return 0;
}
```

  ### [3.5 数字游戏Ⅱ](https://www.acwing.com/activity/content/11/)

* **题面**

  >由于科协里最近真的很流行数字游戏。
  >
  >某人又命名了一种取模数，这种数字必须满足各位数字之和$modN$为 0。
  >
  >现在大家又要玩游戏了，指定一个整数闭区间 [a.b]，问这个区间内有多少个取模数。
  >
  >数据范围
  >$1≤a,b≤2^{31}−1$,
  >$1≤N<100$
  >样例
  >输入
  >1 19 9
  >
  >输出
  >2

* **解题思路**

  虽然这道题看起来很复杂，但是本质还是还是数位DP的套路，只不过现在性质是满足各位数字之和$mod N$为0。那么此题实际上困难点在于预处理，我们发现预处理这其实就是一个$dp$，我们用闫式$DP$分析法分析如下：
![](https://img-blog.csdnimg.cn/img_convert/d9a440fdaf5a847c7cdc37227b76acf5.png)
  
  我们得到这个$f$数组有什么用呢？我们发现，如果我现在已知前面的位数相加为$last$，在左分支处，由于后面的数可以随便枚举，所以我们利用这个性质直接累加$f[i+1][j][mod(-last,p)]$即可得到种类数。故此按照数位$DP$步骤易解。

* **代码**

```cpp
/**
  *@filename:数字游戏Ⅱ
  *@author: pursuit
  *@csdn:unique_pursuit
  *@email: 2825841950@qq.com
  *@created: 2021-05-12 18:23
**/
#include <bits/stdc++.h>

using namespace std;

typedef long long ll;
const int maxn = 100000 + 5;

int l,r,p;
int f[12][12][110];//f[i][j][k]表示i位数，最高位是j，其模n的余数是k的方案数。
//预处理也是一个dp过程。
int mod(int x,int y){
    //由于c++中的%负数会得到负数，所以我们需要做一个偏移。
    return (x%y+y)%y;
}
void init(){
    memset(f,0,sizeof(f));
    //预处理f数组。
    for(int i=0;i<10;i++)f[1][i][i%p]++;
    for(int i=2;i<12;i++){
        for(int j=0;j<10;j++){
            for(int k=0;k<p;k++){
                for(int x=0;x<10;x++){
                    f[i][j][k]+=f[i-1][x][mod(k-j,p)];
                }
            }
        }
    }
}
int dp(int n){
    if(!n)return 1;
    vector<int> a;//存储切出来的位数。
    while(n)a.push_back(n%10),n/=10;
    int last=0;//last存储前面数字之和。
    int ans=0;
    for(int i=a.size()-1;i>=0;i--){
        int x=a[i];
        for(int j=0;j<x;j++){
            //走左边分支。为了凑成模n余0，则接下来的所有位数相加+last模n为0，所以我们来个-last即可。
            ans+=f[i+1][j][mod(-last,p)];
        }
        last+=x;
        if(!i&&last%p==0)ans++;//判断本身是否符合条件。
    }
    return ans;
}
void solve(){
}
int main(){
    while(cin>>l>>r>>p){
        init();
        cout<<dp(r)-dp(l-1)<<endl;
    }
    solve();
    return 0;
}
```

### [3.6 不要62](http://acm.hdu.edu.cn/showproblem.php?pid=2089)

* **题面**

  >杭州人称那些傻乎乎粘嗒嗒的人为62（音：laoer）。
  >杭州交通管理局经常会扩充一些的士车牌照，新近出来一个好消息，以后上牌照，不再含有不吉利的数字了，这样一来，就可以消除个别的士司机和乘客的心理障碍，更安全地服务大众。
  >不吉利的数字为所有含有4或62的号码。例如：
  >62315 73418 88914
  >都属于不吉利号码。但是，61152虽然含有6和2，但不是62连号，所以不属于不吉利数字之列。
  >你的任务是，对于每次给出的一个牌照区间号，推断出交管局今次又要实际上给多少辆新的士车上牌照了。
  >
  >Input
  >
  >输入的都是整数对n、m（0<n≤m<1000000），如果遇到都是0的整数对，则输入结束。
  >
  >Output
  >
  >对于每个整数对，输出一个不含有不吉利数字的统计个数，该数值占一行位置。
  >
  > Sample Input
  >
  >```
  >1 100
  >0 0
  >```
  >
  > Sample Output
  >
  >```c++
  >80
  >```

* **解题思路**

  这道题相对来说比较简单，因为预处理这一部分我们很容易想到。用$f[i][j]$表示$i$位数字且最高位为$j$的方案数。那么我们排除掉特殊情况进行状态转移即可。代码附详细注释。

* **代码**

```cpp
/**
  *@filename:不要62
  *@author: pursuit
  *@csdn:unique_pursuit
  *@email: 2825841950@qq.com
  *@created: 2021-05-12 19:56
**/
#include <bits/stdc++.h>

using namespace std;

typedef long long ll;
const int maxn = 100000 + 5;
const int mod = 1e9+7;

int l,r;
int f[11][11];//f[i][j]表示i位数且最高位为j的方案数。
//那么我们来对这个进行分析，对于f[i][j]这个状态，我们根据题意我们转移的f[i-1][k]必须满足k!=4,j!=4.
//并且jk!=62.
void init(){
    for(int i=0;i<10;i++)f[1][i]=1;
    //排除4的情况。
    f[1][4]=0;
    for(int i=2;i<11;i++){
        for(int j=0;j<10;j++){
            if(j==4)continue;
            for(int k=0;k<10;k++){
                if((j==6&&k==2)||k==4)continue;
                f[i][j]+=f[i-1][k];
            }
        }
    }
}
int dp(int n){
    if(!n)return 1;
    vector<int> a;//存储分割位数。
    int ans=0,last=0;//last保存上一位的值。
    while(n)a.push_back(n%10),n/=10;
    for(int i=a.size()-1;i>=0;i--){
        int x=a[i];
        for(int j=0;j<x;j++){
            //走左边分支，我们需要判断。
            if(j==4||(j==2&&last==6))continue;
            ans+=f[i+1][j];
        }
        if(x==4||(last==6&&x==2))break;
        last=x;
        if(!i)ans++;
    }
    return ans;
}
void solve(){
}
int main(){
    init();
    while(cin>>l>>r&&(l||r)){
        cout<<dp(r)-dp(l-1)<<endl;
        solve();
    }
    return 0;
}
```

### [3.7 恨7不成妻](http://acm.hdu.edu.cn/showproblem.php?pid=4507)
由于此题太过变态，已单开一篇blog讲解： [点这里](https://blog.csdn.net/hzf0701/article/details/116766379?spm=1001.2014.3001.5501)。

## 4 数位DP总结

做了这么多的题，我们发现数位$DP$确实是有套路的，难点就在于预处理，通常就是要用$DP$来预处理，这里推荐大家学一下闫式$DP$分析法。预处理完之后，就可以套路做题了。当然，学$DP$一定要多刷题，所以请各位一定要多多刷题哦！
