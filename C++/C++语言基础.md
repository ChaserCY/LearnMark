## Vector迭代器运用

```
  auto ir = find(Q.begin(),Q.end(),x);

    Q.erase(ir);//删除对应迭代器的元素

    auto it = find(Q.begin(),Q.end(),y);

    int indd = it - Q.begin();

    if(z==1){

      Q.insert(Q.begin() + indd,x);

    }

    else{Q.insert(Q.begin() + indd+1,x);}
```
函数，方法：
Q.erase(auto)  //删除对应迭代器位置的元素
Q.find(Q.begin(),Q.end(),查找元素值)      //返回Q中对应元素的迭代器位置，==索引=返回值-Q.begin()==
Q.insert(auto,插入元素值)       //在迭代器位置的元素==前==插入元素
Q.begin()          //指向首元素。即0索引处
Q.end()             // 指向==最后元素后一位==，即最后一位空白处
Q.back()           // 直接返回数组中的最后一位的值
Q.resize()          //对未规定空间的数组定义重新分配空间，常见与                         在全局变量中命名，输入数组之前重新定义大小
reverse(Q.begin(),Q.end())           //反转数组
另外：迭代器之间不能相加，只能相减来得出距离，相加得换成int和迭代器来加（算偏移）



## 迭代器定义
- 迭代器（Iterator）是 C++ 中一种**抽象的 “遍历工具”**，本质上是一个**行为类似指针的对象**（或指针本身），用于**按顺序访问容器（如 vector、list、map 等）中的元素**，而无需暴露容器的内部存储结构。
- 迭代器可以是==**指针**==（比如 vector 的迭代器本质就是指向数组元素的指针），也可以是**自定义类对象**（比如 list 的迭代器，内部封装了链表节点的指针，通过重载`++`、`*`等运算符模拟指针行为）。
- 无论内部实现如何，迭代器必须支持一些**基础操作**（不同类型的迭代器支持的操作不同），例如：
- `++it`：移动到下一个元素（所有迭代器都支持）；
- `*it`：获取当前元素的值（所有迭代器都支持）；
- `it--`：移动到上一个元素（双向及以上迭代器支持）；
- `it + n`：移动到第 n 个元素（仅随机访问迭代器支持）；
- `it1 - it2`：计算两个迭代器的距离（仅随机访问迭代器支持）。



## 字符串输入
- cin.ignore(); //吞掉回车，一般用于输入数字后需要输入字符或者字符串的时候
- getline(cin,string s)  //读取整行字符串，不包括回车（以回车为止)
- cin.peek()  //查看当前输入的值
	-并且不会影响下一次读取，只是偷窥下一个==字符==
- cin>>string s;  //默认以空格，换行，制表符为间隔


## queue容器
**queue容器不支持find操作和迭代器**
- 添加：Q.push()
- 查询：Q.front()    Q.back()
- 删除队头：Q.pop()
- `std::queue` 是一个适配器容器，不提供迭代器功能
- 不能使用 `Q.find()` 或 `Q.begin()` / `Q.end()`
- 查找元素方法：
 ```
	 bool flag = false;
     queue<int> temp = Q;
     while(!temp.empty()){
        if(temp.front()== f){
            flag = true;
            break;
        }
        else{
            temp.pop();
        }
    }
 ```


## pair
定义：pair<T1,T2> Q
读取：Q.first    Q.second
比较：先比first，再比second，先由前决定，相等后比较后一个


## Set容器
	定义: set<T1> Q(自动升序排序)
	
	插入：Q.insert（元素）；返回pair<iterator,bool> bool表示是否插入成功
	
	删除：erase()  
      (1)按值：erase(val) 返回删除元素个数（0/1）
      (2)按迭代器：erase(it) 删除迭代器指向元素（无返回值）
      (3)按范围：erase(first，last)  删除[frist, last)区间的元素(迭代器),erase(Q.begin(),Q.end())
      
	元素个数(是否存在):  Q.count(val)
	
	查找：Q.find(val)  返回迭代器，没有就是Q.end()




## 范围类查找
### 1. 查找最大 / 最小元素相关
#### `min_element`：查找范围内的最小元素
和 `max_element` 对称，返回指向范围内最小元素的==迭代器==（需解引用获取值）。

**用法**：

```cpp
#include <algorithm>
#include <vector>
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {3, 1, 4, 2};
    // 找到最小元素的迭代器
    auto min_it = min_element(v.begin(), v.end()); 
    cout << "最小元素：" << *min_it; // 输出：1
    return 0;
}
```

### 2. 元素查找相关

#### `find`：查找指定值的元素

在范围内查找等于目标值的第一个元素，返回其==迭代器==；若未找到，返回 `end()`。

**用法**：

```cpp
vector<string> v = {"apple", "banana", "cherry"};
auto it = find(v.begin(), v.end(), "banana"); 
if (it != v.end()) {
    cout << "找到：" << *it; // 输出：banana
}
```

#### `find_if`：按条件查找元素

在范围内查找第一个满足**自定义条件**的元素（通过 lambda 或函数指定条件）。

**用法**：

```cpp
vector<int> v = {1, 2, 3, 4, 5};
// 查找第一个偶数（条件：x % 2 == 0）
auto it = find_if(v.begin(), v.end(), [](int x) { return x % 2 == 0; });
cout << "第一个偶数：" << *it; // 输出：2
```

### 3. 元素计数相关

#### `count`：统计指定值的出现次数

返回范围内等于目标值的元素个数。

**用法**：

```cpp
vector<int> v = {1, 2, 2, 3, 2};
int cnt = count(v.begin(), v.end(), 2); 
cout << "2出现的次数：" << cnt; // 输出：3
```

#### `count_if`：按条件统计元素个数

返回范围内满足**自定义条件**的元素个数。

**用法**：

```cpp
vector<int> v = {1, 3, 5, 7, 8};
// 统计奇数的个数（条件：x % 2 != 0）
int cnt = count_if(v.begin(), v.end(), [](int x) { return x % 2 != 0; });
cout << "奇数个数：" << cnt; // 输出：4
```

### 4. 条件判断相关

#### `all_of`：判断所有元素是否满足条件

若范围内**所有元素**都满足条件，返回 `true`；否则返回 `false`。

**用法**：

```cpp
vector<int> v = {2, 4, 6, 8};
// 判断是否全是偶数
bool all_even = all_of(v.begin(), v.end(), [](int x) { return x % 2 == 0; });
cout << (all_even ? "全是偶数" : "有奇数"); // 输出：全是偶数
```

#### `any_of`：判断是否存在元素满足条件

若范围内**至少有一个元素**满足条件，返回 `true`；否则返回 `false`。

**用法**：


```cpp
vector<int> v = {1, 3, 5, 8};
// 判断是否存在偶数
bool has_even = any_of(v.begin(), v.end(), [](int x) { return x % 2 == 0; });
cout << (has_even ? "存在偶数" : "无偶数"); // 输出：存在偶数
```

#### `none_of`：判断所有元素是否都不满足条件

若范围内**所有元素都不满足条件**，返回 `true`；否则返回 `false`（与 `all_of` 逻辑相反）。

**用法**：

```cpp
vector<int> v = {1, 3, 5, 7};
// 判断是否没有偶数
bool no_even = none_of(v.begin(), v.end(), [](int x) { return x % 2 == 0; });
cout << (no_even ? "没有偶数" : "有偶数"); // 输出：没有偶数
```

### 5. 元素累加相关（`<numeric>` 头文件）

#### `accumulate`：计算范围内元素的累加和（或自定义累加）

需要包含 `<numeric>` 头文件，默认计算总和，也可通过自定义函数实现其他累加逻辑（如乘积）。

**用法**：

```cpp
#include <numeric> // accumulate所在头文件
vector<int> v = {1, 2, 3, 4};
// 计算总和（初始值为0）
int sum = accumulate(v.begin(), v.end(), 0); 
cout << "总和：" << sum; // 输出：10

// 计算乘积（初始值为1，自定义乘法逻辑）
int product = accumulate(v.begin(), v.end(), 1, [](int a, int b) { return a * b; });
cout << "乘积：" << product; // 输出：24
```

### 6. 排序与反转相关

#### `sort`：对范围内元素排序

默认升序，可通过自定义比较器指定排序规则（如降序）。

**用法**：

```cpp
vector<int> v = {3, 1, 4, 2};
sort(v.begin(), v.end()); // 默认升序：{1,2,3,4}
// 降序排序（用greater<int>()）
sort(v.begin(), v.end(), greater<int>()); // {4,3,2,1}
```

#### `reverse`：反转范围内的元素顺序

**用法**：

```cpp
vector<int> v = {1, 2, 3, 4};
reverse(v.begin(), v.end()); // 反转后：{4,3,2,1}
```

## 固定模板(每道题都可加入)
```cpp
#define endl ‘\n’  \\原endl慢一些

int main(){
ios::sync_with_stdio(false); // false可以是0
	cin.tie(nullptr);  // nullptr可以是0
	 cout.tie(nullptr);
}
```

### 奇偶数的判定
#### 一个不知奇偶的整数(s)/一个不知奇偶的数(d)所得值，不用多处判定，直接是（s + d -1)/d  - 1

## sort排序操作
### Vector系列
1.基本操作

```cpp
	vector<int> Q;
	 sort(Q.begin(), Q.end());  //(单位元素)全升序，从小到大
```

2.自定义排序规则

```cpp
	 vector<int> T;
	 sort(T.begin(), T.end(), cmp(或者lamada函数))；
	 bool cmp(const int *a, const int *b)
	 {
	    return a < b;  //小的排前面（升序）
	 }
 ```

lamada版(格式：[](const 两个Vector中&基本比较元素){bool 函数内部})
```cpp
	 vector<int> T;
	 sort(T.begin(), T.end(), [](const int *a, const int *b)
	 {
	    return a < b;  //小的排前面（升序）
	 })；
	 
 ```

3.特殊情况的结构
```cpp
	vector<pair<int,int>> Q;
	 sort(Q.begin(), Q.end(), cmp); //默认是从first进行升序
	 bool cmp (const pair<int,int> &a, const pair<int,int> &b){
	 if(a.first!=b.first){
	 return a.first > b.first;
	 }
	 else {
	 return a.second < b.second;
	 }
	 }
```

## tuple 数据类型 (简便型结构体)

#### 特性：支持**任意数量的不同元素类型**

#### 定义：tuple<int, double, std::string> T;

#### 自动推导类型：auto f = make_tuple(100, true, 2.718);

#### 结合Vector的使用

```cpp
#include <vector>  
using namespace std;        
vector<tuple<int, string>> vec;   
vec.emplace_back(1, "A"); // 直接在容器中构造 tuple
vec.push_back(make_tuple(2, "B")); // 先构造再插入,也可以直接push_back
```

#### 元素访问：`get<索引>(tuple对象)`  //索引表示在定义tuple时的元素类型顺序从0开始



## 向下，向上取整(正整数版)
#### 向上取整
		(s + d - 1) / d
		函数：ceil(s/d); //返回浮点数double
#### 向下取整
		 s/d
		 函数: floor(s/d); //返回浮点数double


## map和unordered_map(哈希表解法平替，字典，键自动去重)
**一维象征数组效率>>map系列**(字母表)
**`map`**：基于**红黑树**，pair构成的集合，按照key的值==升序排序==，查找删除效率(O(logn))
**`unordered_map`**：基于**哈希表**，没有顺序，查找删除效率(平均O(1)--最坏O(n))，追求极致的查找删除用这个

	1.访问元素
- auto p.first    auto p.second   Q[key]

	1.插入元素
- **`insert();`**
```cpp
map<int, string> mp;
mp.insert({1, "a"}); // 插入{1:"a"}
mp.insert(pair<int, string>(2, "b")); // 另一种插入方式

unordered_map<int, string> ump;
ump.insert({1, "a"});
```
	[]运算符
```cpp
mp[3] = "c"; // 若key=3不存在则插入，存在则覆盖value
ump[3] = "c";
```
==若key=3不存在则插入，存在则覆盖value==

	2.查找元素
- **`find(key)`**：查找 `key`，返回指向该元素的迭代器；若不存在则返回 `end()`
```cpp
auto it = mp.find(1);
if (it != mp.end()) {
    cout << it->second; // 输出"a"
}

auto it2 = ump.find(2);
if (it2 != ump.end()) {
    cout << it2->second; // 输出"b"
}
```

- **`count(key)`**：返回 `key` 存在的数量（关联容器中 `key` 唯一，故==返回 0 或 1）==。
```cpp
if (mp.count(3)) {
    cout << "key=3存在";
}
```

	 3. 删除元素

- **`erase(key)`**：按 `key` 删除元素，返回删除的数量（0 或 1）。
- **`erase(iterator)`**：按迭代器删除元素。
    
    cpp
    
    运行
    
 ```cpp
   mp.erase(1); // 删除key=1的元素
   ump.erase(ump.find(2)); // 按迭代器删除
   ```

	 4. 清空与大小

- **`clear()`**：清空所有元素。
- **`size()`**：返回元素个数。
- **`empty()`**：判断是否为空。

  ```cpp
   if (mp.empty()) cout << "空";
   cout << mp.size(); // 输出当前元素数
   ```
	 5. 遍历元素

通过迭代器遍历（`begin()` 到 `end()`）：
```cpp
// map遍历（有序）
for (auto& p : mp) {
    cout << p.first << ":" << p.second << " "; // 按key升序输出
}

// unordered_map遍历（无序）
for (auto& p : ump) {
    cout << p.first << ":" << p.second << " "; // 顺序由哈希值决定
}
```
反向遍历(`rbegin()` 到`rend()`):
```cpp
 for (auto it = numCount.rbegin(); it != numCount.rend(); ++it) {
            sumCount += it->second; // 累加当前元素的出现次数

            // 3. 累加和 ≥ k 时，当前元素就是第k大元素
            if (sumCount >= k) {
                return it->first;
            }
        }
```

## set和unordered_set（自动去重序列)
- **`set`**：基于**红黑树**（自平衡二叉搜索树）实现，天然维护元素的**有序性**（默认按升序排列）。查找插入删除效率(O(logn))；
- **`unordered_set`**：基于**哈希表**（开链法解决冲突）实现，元素存储顺序由哈希值决定，**无序** 。查找删除效率(平均O(1)--最坏O(n))；

#### 1. 基础操作（共性）

两者均支持集合的核心操作（==去重、插入、删除、查找==）：

```cpp
set<int> s;
unordered_set<int> us;

// 插入元素（自动去重）
s.insert(3); s.insert(1); s.insert(3); // 最终{1,3}
us.insert(3); us.insert(1); us.insert(3); // 最终{1,3}（顺序不定）

// 查找元素
if (s.count(1)) cout << "set存在1";
if (us.find(3) != us.end()) cout << "unordered_set存在3";

// 删除元素
s.erase(1);
us.erase(3);
```

#### 2. 特性操作（差异）

- `set` 支持有序操作：

```cpp
    set<int> s{1,3,5,7};
    auto it = s.lower_bound(4); // 指向第一个≥4的元素（5）
    auto it2 = s.upper_bound(5); // 指向第一个>5的元素（7）
 ```

- `unordered_set` 支持哈希表参数调整：

 ```cpp
    unordered_set<int> us;
    us.reserve(100); // 预留桶空间，减少扩容开销
    cout << us.bucket_count(); // 查看当前桶数量
 ```


#### 3.查找操作

```cpp
set<int> s{1,2,3};
unordered_set<int> us{1,2,3};

// set的find
auto it1 = s.find(2);
if (it1 != s.end()) cout << "set找到：" << *it1; // 输出2

// unordered_set的find
auto it2 = us.find(3);
if (it2 != us.end()) cout << "unordered_set找到：" << *it2; // 输出3
```

#### 4.判断元素是否存在操作
```cpp
num_set.count(数字)   //只会返回0或1
```


#### 5.集合初始化

```cpp
unordered_set<int> Q(nums.begin(),nums.end());

set<int> S(nums.begin(), nums.end());
```




## 位运算
### 1. 基础位运算总览

| 符号       | 名称                   | 规则               | 助记                 |
| -------- | -------------------- | ---------------- | ------------------ |
| **`&`**  | **按位与 (AND)**        | 两个位都为 1 时，结果才为 1 | **全真才真** (用于提取/清零) |
| **`      | `**                  | **按位或 (OR)**     | 只要有一个位为 1，结果就为 1   |
| **`~`**  | **按位取反 (NOT)**       | 0 变 1，1 变 0      | **黑白颠倒**           |
| **`<<`** | **左移 (Left Shift)**  | 二进制位全部左移，右补 0    | **乘 2 的 N 次方**     |
| **`>>`** | **右移 (Right Shift)** | 二进制位全部右移         | **除 2 的 N 次方**     |
| `^`      | 异或 (XOR)             | 相同为 0，不同为 1      | 无进位加法 (你刚用的那个)     |

---

### 2. 详细用法与技巧

#### (1) 按位与 (`&`)

- **逻辑**：只有对应的两个二进制位都为 1 时，结果位才为 1。
    
- **用途**：**清零** 或 **判断某一位是否为 1**。
    

**常用技巧：**

- **判断奇偶性**：`n & 1`
    
    - 如果结果是 1，说明二进制最后一位是 1，是奇数。
        
    - 如果结果是 0，是偶数。
        
    - _比 `n % 2` 更快。_
        
- **消去数字二进制表示中最后一个 1**（Brian Kernighan 算法）：`n & (n - 1)`
    
    - 常用于计算一个数中有多少个 1（汉明重量）。
        

#### (2) 按位或 (`|`)

- **逻辑**：只要对应的两个二进制位有一个为 1，结果位就为 1。
    
- **用途**：**将某一位设置为 1**。
    

**示例：**

- `x | (1 << n)`：将 x 的第 n 位设置为 1，其他位不变。
    

#### (3) 按位取反 (`~`)

- **逻辑**：单目运算符，将 1 变成 0，0 变成 1。
    
- **注意**：在计算机中数值通常用“补码”表示。
    
    - `~x` 的结果通常等于 `-x - 1`。
        
    - 例如：`~0` 是 `-1`，`~1` 是 `-2`。
        

#### (4) 左移 (`<<`)

- **逻辑**：各二进位全部左移若干位，高位丢弃，低位补 0。
    
- **数学意义**：`x << n` 等价于 $x \times 2^n$。
    

**示例：**

- `1 << 10` 等于 $2^{10} = 1024$。这是生成 2 的幂次的最快方法。
    

#### (5) 右移 (`>>`)

- **逻辑**：各二进位全部右移若干位。
    
- **数学意义**：`x >> n` 等价于 $x / 2^n$ (向下取整)。
    
- **注意**：
    
    - **算术右移**（针对有符号数）：高位补符号位（正数补0，负数补1）。C++ 中的 `>>` 也就是这个。
        
    - **逻辑右移**（针对无符号数）：高位直接补 0。在 Java/JS 中有专门的 `>>>` 运算符，但 C++ 主要看数据类型。
        

---

### 3. 算法题中的“骚操作”总结

这些组合拳在 LeetCode 中非常常见：

1. **判断是不是 2 的幂**：
    
    cpp
    
    ```cpp
    // 如果 n 是 2 的幂，它的二进制只有一个 1
    // n & (n - 1) 会把这唯一的 1 消除变成 0
    bool isPowerOfTwo = (n > 0) && (n & (n - 1)) == 0;
    ```
    
2. **获取二进制中最右边的 1 (Lowbit)**：  
    树状数组（Fenwick Tree）的核心操作。
    
    cpp
    
    ```cpp
    int lowbit = x & -x;
    // 例如 x = 6 (110)，-x (补码) = ...010，结果是 2 (10)
    ```
    
3. **不使用临时变量交换两个数**（基于异或）：
    
    cpp
    
    ```cpp
    a ^= b;
    b ^= a;
    a ^= b;
    ```
    
4. **构造掩码 (Mask)**：
    
    - 获取后 n 位全是 1 的数：`(1 << n) - 1`
        
    - 例如 `n=3`，`(1<<3) - 1` 就是 `1000 - 1 = 111` (十进制 7)。
        

掌握这些，很多涉及二进制处理、状态压缩 DP 的题目会变得很简单。