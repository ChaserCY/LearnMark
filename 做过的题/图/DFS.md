### 问题描述

小蓝最近迷上了一款名为《数字接龙》的迷宫游戏，游戏在一个大小为 N×NN×N 的格子棋盘上展开，其中每一个格子处都有着一个 0…K−10…K−1 之间的整数。游戏规则如下：

1. 从左上角 (0,0)(0,0) 处出发，目标是到达右下角 (N−1,N−1)(N−1,N−1) 处的格子，每一步可以选择沿着水平/垂直/对角线方向移动到下一个格子。
    
2. 对于路径经过的棋盘格子，按照经过的格子顺序，上面的数字组成的序列要满足：0,1,2,…,K−1,0,1,2,…,K−1,0,1,2…0,1,2,…,K−1,0,1,2,…,K−1,0,1,2… 。
    
3. 途中需要对棋盘上的每个格子恰好都经过一次（仅一次）。
    
4. 路径中不可以出现交叉的线路。例如之前有从 (0,0)(0,0) 移动到 (1,1)(1,1) ，那么再从 (1,0)(1,0) 移动到 (0,1)(0,1) 线路就会交叉。
    

为了方便表示，我们对可以行进的所有八个方向进行了数字编号，如下图 2 所示；因此行进路径可以用一个包含 0…70…7 之间的数字字符串表示，如下图 1 是一个迷宫示例，它所对应的答案就是：4125521441255214。

![图片描述](https://dn-simplecloud.shiyanlou.com/questions/uid1331660-20240701-1719822098425)

现在请你帮小蓝规划出一条行进路径并将其输出。如果有多条路径，输出字典序最小的那一个；如果不存在任何一条路径，则输出 −1−1。

### 输入格式

第一行包含两个整数 N,KN,K 。

接下来输入 NN 行，每行 NN 个整数表示棋盘格子上的数字。

### 输出格式

输出一行表示答案。如果存在答案输出路径，否则输出 −1−1。

### 样例输入

`3 3` 
`0 2 0 1 1 1 2 0 2`

 ### 样例输出

```text
41255214
```

### 样例说明

行进路径如图 11 所示。

### 评测用例规模与约定

对于 80%80% 的评测用例：1≤N≤51≤N≤5 。

对于 100%100% 的评测用例：1≤N≤10,1≤K≤101≤N≤10,1≤K≤10 。

**题解**
```cpp
#include <bits/stdc++.h>
using namespace std;
  
int dir[8][2] ={{-1,0},{-1,1},{0,1},{1,1},{1,0},{1,-1},{0,-1},{-1,-1}}; //循环周围八个方向
int N;
int K;
vector<int> path;
  
void dfs(const vector<vector<int>> &Q,int x,int y,vector<vector<bool>> &V){
    if(path.size()==N*N-1&&x==N-1&&y==N-1){
      for(int num:path)cout<<num;
      exit(0);
    }
  for(int i = 0;i < 8;i++){
    int tx = x+dir[i][0];
    int ty = y+dir[i][1];
    if(tx<0||ty<0||tx>=N||ty>=N||V[tx][ty]){
      continue;
    }
  
      int ex = (Q[x][y]+1)%K;
      if(Q[tx][ty]==ex){
        path.push_back(i);
        V[tx][ty] = true;
        dfs(Q,tx,ty,V);
        V[tx][ty] = false;
        path.pop_back();
      }
  }
}
  
int main()
{
  ios::sync_with_stdio(false);

  
  cin>>N>>K;
 if (N == 1) {
        cout << endl; // 或根据题目要求输出空/0，需看题目描述
        return 0;
    }
  
  vector<vector<int>> Q(N,vector<int>(N));
  vector<vector<bool>> V(N,vector<bool>(N,false));
  for(int i = 0;i<N;i++){
    for(int j = 0;j < N;j++){
      cin>>Q[i][j];
    }
  }
  V[0][0] = true;
  dfs(Q,0,0,V);
cout << -1 << endl;
  return 0;
}
```