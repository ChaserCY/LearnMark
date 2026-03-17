![](图片/做过的题/数学/LCM和GCD(lcm,%20gcd)/IMG-20260315155914935.png)

![](图片/做过的题/数学/LCM和GCD(lcm,%20gcd)/IMG-20260310204929308.png)
![](图片/做过的题/数学/LCM和GCD(lcm,%20gcd)/IMG-20260310204937943.png)

题解：(明确知道LCM与GCD的转换式，)
```cpp
#include <bits/stdc++.h> 
using namespace std;    
  
const int MAXH = 100000 + 5; 
  
int main()
{
    ios::sync_with_stdio(false); 
    cin.tie(nullptr);         
       
    int N;
    cin>>N;
    
    vector<int> Q(N);
    vector<int> cnt(MAXH+1);//哈希表，存数的出现次数
    
    int bast_g;    //记录最大公约数(3个数组的数的)
    for(int i = 0;i < N;i++){
      cin>>Q[i];
      cnt[Q[i]]++;   //记录出现次数
    }
    for(int g = MAXH-1;g>=1;g--){   //从最大的数倒着
      int c=0;                       //统计以g为约数的次数
      
        for(int j = g;j<MAXH;j+=g){  //列举g的倍数，即找是以g为约数的数
          c += cnt[j];
          if(c>=3){
            bast_g = g;          //找到就是最大的g
            break; 
          }
        }
        if(c>=3){
          break;
        }
    }
  vector<int> ant;
  for(int x:Q){
    if(x%bast_g==0){          //能被bast_g整除就是前面找到的数
      ant.push_back(x);
    }
  }
  sort(ant.begin(),ant.end()); //题目要求S相同时，找最小的
  cout<<ant[0]<<" "<<ant[1]<<" "<<ant[2];
}
```