# 状态压缩DP

## 基于连通性

题目类似于，给你一些奇形怪状的积木，让你求出有几种摆放方法，或者最多能摆几个积木

* 这种？1*2长方形

![蒙德里安的梦想](C:\Users\z1382\AppData\Roaming\Typora\typora-user-images\image-20221118110116683.png)

* 或者这种？3*3长方形

![小国王](C:\Users\z1382\AppData\Roaming\Typora\typora-user-images\image-20221118110237504.png)

### 做法

状态表示 

常见的是`f[i][state]`表示前i行（或列）已经摆好，第i行的状态是state

状态计算

设第`i-1`行的状态是`before_state`，我们研究如何从`before_state`转移到`state`，使用位运算，这里就根据题目具体分析吧。

#### 例题

![小国王](C:\Users\z1382\AppData\Roaming\Typora\typora-user-images\image-20221118120808631.png)

#### 代码如下

```c++
/*
状态表示：f[i][j][s]
    所有只摆在前i行，已经摆了j个国王，并且第i行摆放的状态是s的所有方案的集合
状态计算：
    只有i-1行的状态才会影响第i行的状态，当前我们第i行的状态是s，设i-1行的状态是a，a和s要符合什么规则呢？
    首先，不管是s还是a，都不能有两个1相邻，我们使用数组head先预处理出所有没有两个1相邻的数
    其次，上下两行不能相互攻击到，要满足：(a & b) == 0 && (a | b)不能有两个相邻的1
    f[i][j][s] += f[i - 1][j - count(j)][ss] count计算一个数里面有几个1
*/

#include <iostream>
#include <cstring>
#include <algorithm>
#include <vector>

using namespace std;
const int N = 12, K = 110, M = 1 << N;
int n, k;
long long f[N][K][M];
vector<int> state;  //预处理出没有相邻的1的所有数
vector<int> change[M];  //i行状态为s，i-1行状态为ss，这里存能更新s的ss

bool check(int x)
{
    for (int i = 0; i < n; i ++ )
    {
        if ((x >> i & 1) && (x >> (i + 1) & 1)) return false;
    }
    return true;
}

int count(int x)
{
    int res = 0;
    for (int i = 0; i < n; i ++ )
    {
        if (x >> i & 1) res ++ ;
    }
    return res;
}

int main()
{
    cin >> n >> k;
    //预处理1
    for (int i = 0; i < (1 << n); i ++ )
    {
        if (check(i)) state.push_back(i);
    }
    
    f[0][0][0] = 1;
    
    for (int i = 1; i <= n + 1; i ++ )
    {
        for (int j = 0; j <= k; j ++ )
        {
            for (int s : state)
            {
                for (int ss : state)
                {
                    if ((s & ss) == 0 && check(s | ss) && j - count(s) >= 0)
                    {
                        f[i][j][s] += f[i - 1][j - count(s)][ss];
                    }
                }
            }
        }
    }
    
    //这里非常巧妙，让他多了一行，这样，这里就不用循环遍历最后一行了
    cout << f[n + 1][k][0];
    return 0;
}
```



## 基于集合

### 解决重复覆盖问题

比如说我要用最少几条线，去覆盖所有的点，就是重复覆盖问题

状态表示

 `f[state]`二进制表示已经覆盖了哪些点

状态计算

选一个还未被覆盖的点，将它所在直线加入状态中，更新状态

#### 例题

![愤怒的小鸟](C:\Users\z1382\AppData\Roaming\Typora\typora-user-images\image-20221118115753219.png)

![image-20221118115835801](C:\Users\z1382\AppData\Roaming\Typora\typora-user-images\image-20221118115835801.png)

![image-20221118115902553](C:\Users\z1382\AppData\Roaming\Typora\typora-user-images\image-20221118115902553.png)

#### 代码如下

```c++
/*
好难呀，我滴妈呀，就是那种好像能理解，但是我想不出来啊啊啊……
谁能想到用二进制去存点啊……

主要是难想，但其实它的dp部分，我觉得还是可以理解的
目前可知的是 两点确定一条抛物线，有n个点，我们可以先预处理出n^2条抛物线，和每条抛物线能覆盖那些点
用数组path[i][j]来存
公式推导一下下：(x1, y1) (x2, y2) 求a和b
a * x1 * x1 + b * x1 = y1
a * x2 * x2 + b * x2 = y2


我们用状压dp解决重复覆盖问题（不会什么Dancing Links）
状态表示：f[state]
    表示达到状态state最少用了几条抛物线
状态计算：
    设前一个状态为before_state
    两者怎么转移呢？before_state加多一条抛物线l变成了state
    state = before_state | l 是不是！！！！感觉还可以理解
*/

#include <iostream>
#include <cstring>
#include <algorithm>
#include <cmath>

using namespace std;
typedef pair<double, double> PDD;
const int N = 20, M = 1 << N;
int n, m;
int t;
PDD p[N];
int path[N][N];
int f[M];

int cmp(double x, double y)
{
    if (fabs(x - y) < 1e-9) return 0;
    if (x < y) return -1;
    return 1;
}

int main()
{
    cin >> t;
    while (t -- )
    {
        memset(path, 0, sizeof path);
        cin >> n >> m;
        for (int i = 0; i < n; i ++ ) cin >> p[i].first >> p[i].second;
        
        //预处理一下
        for (int i = 0; i < n; i ++ )
        {
            path[i][i] = 1 << i;
            for (int j = 0; j < n; j ++ )
            {
                int state = 0;
                double x1 = p[i].first, y1 = p[i].second;
                double x2 = p[j].first, y2 = p[j].second;
                if (!cmp(x1, x2)) continue;
                double a = (y1 / x1 - y2 / x2) / (x1 - x2);
                double b = y1 / x1 - a * x1;
                
                if (cmp(a, 0) >= 0) continue;
                for (int k = 0; k < n; k ++ )
                {
                    double x = p[k].first, y = p[k].second;
                    if (!cmp(a * x * x + b * x, y)) state += 1 << k;
                }
                path[i][j] = state;
            }
        }
        
        memset(f, 0x3f, sizeof f);
        f[0] = 0;
        //开始dp
        for (int i = 0; i + 1 < 1 << n; i ++ )
        {
            //我们这里的i是before_state 这样好写一点点
            //我们先找到一个新的抛物线l，让l | before_state 得到新的state
            int x = 0;
            for (int j = 0; j < n; j ++ )
                if (!(i >> j & 1))
                {
                    x = j;
                    break;
                }
            
            for (int j = 0; j < n; j ++ )
                f[i | path[x][j]] = min(f[i | path[x][j]], f[i] + 1);
        }
        
        cout << f[(1 << n) - 1] << endl;
    }
    return 0;
}
```



### 解决跟dfs相关的路径问题

状态表示

`f[i][state]`目前在点i，state表示经过了哪些点

状态计算

根据题意，用二进制进行转移

#### 例题

![最短Hamilton路径](C:\Users\z1382\AppData\Roaming\Typora\typora-user-images\image-20221118120610003.png)

#### 代码如下

```c++
/*
又是一道好难的题啊

0->1->2->3
0->2->1->3
0->2->3->1
……

总共有n-1的阶乘次
20的阶乘非常大，我们要想办法优化它

然后你会发现
0->1->2->3 18
0->2->1->3 20
同样的，我走过123，我肯定走第一条路，那么第二条路这个状态我就不用表示啦，直接并入第一个状态就行
这样你就会发现，影响我的选择的，就只有两个因素
1. 哪些点被用过
2. 目前停在哪个点上

如何记录哪些点被用过呢？使用二进制表示

状态表示：
    f[i][j] i表示哪些点被用过（0010..二进制），j表示当前处在第几个点上
状态计算：
    我现在在k这个点上，我下一步走到j上去
    f[i][j] = w[k][j] + f[state_k][k]
    state_k和i有什么关系呢？i减去j就是state_k
    你这里会发现state_k是固定的，当i和j固定下来的话，但是k是不固定的，我们枚举k，找到最小值赋予f[i][j]
*/

#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;
const int N = 20, M = 1 << N;
int g[N][N];
int n;
int f[M][N];

int main()
{
    cin >> n;
    for (int i = 0; i < n; i ++ )
    {
        for (int j = 0; j < n; j ++ )
        {
            cin >> g[i][j];
        }
    }
    
    memset(f, 0x3f, sizeof f);
    f[1][0] = 0;
    for (int i = 0; i < (1 << n); i ++ )
    {
        for (int j = 0; j < n; j ++ )
        {
            if (i >> j & 1)
            {
                int state_k = i - (1 << j);
                for (int k = 0; k < n; k ++ )
                {
                    if (state_k >> k & 1)
                    {
                        f[i][j] = min(f[i][j], g[k][j] + f[state_k][k]);
                    }
                }
            }
        }
    }
    
    cout << f[(1 << n) - 1][n - 1];
    return 0;
}
```

