# PuerTS与U++的些许差别

![](图片/UE/PuerTs/PuerTS的一些知识/IMG-20260707164554756.png)
 GetController\<T>() 这种模板函数在 JS里只能退化为返回基类 + 手动类型断言。
 TS = “带类型检查的 JS 编译器”，不是“带类型的 JS 运行时”



# **TS 相比 JS 的优势是什么 → 为什么 Puerts 用 TS → “少一步编译”到底是不是优势**。

## 一、TS 相比 JS 的真实优势（不是运行时，而是开发期）

先纠正一个关键点：

> TS 的优势 **几乎全部不在运行时性能，而在开发阶段的“可控性”**

---

### 1. 最大优势：类型系统 = 防 bug

JS：

```
function add(a, b) {  return a + b;}add(1, "2"); // 变成 "12"
```

👉 运行时才炸 / 或 silently wrong

---

TS：

```
function add(a: number, b: number): number {  return a + b;}add(1, "2"); // 编译期直接报错
```

👉 错误提前到“写代码的时候”

---

### 2. 第二个优势：可维护性（大型项目核心）

JS 在大项目的问题：

- 你不知道这个函数吃什么参数
- 你不知道返回什么结构
- 改一个接口全靠“全局搜索+祈祷”

TS：

```
interface Player {  hp: number;  name: string;}
```

👉 IDE 可以：

- 自动补全
- 类型跳转
- 重构安全

---

### 3. 第三个优势：工程级重构能力

TS 能做：

- rename safe refactor
- 参数变更自动报错
- API 变更全局提示

JS 做这些基本靠人工检查

---

### 4. 第四个优势：复杂代码“可读性”

TS 让代码变成：

> “带说明书的 JS”

---

## 二、重点问题：Puerts 为什么用 TS，而不是直接 JS？

先说结论：

> Puerts 用 TS 不是为了“替代 JS”，而是为了**“给 JS 加一层 C++/引擎级类型绑定能力”**

---

### 1. Puerts 本质是什么？

Puerts =

> Unity / Unreal + JavaScript/TypeScript 绑定层

也就是说：

- C++（引擎）
- JS/TS（逻辑层）

---

### 2. 问题核心：JS 不知道 C++ 的“类型世界”

举个例子：

C++ 里：

```
class Player {    int hp;    void Attack(int damage);};
```

---

### JS 如果直接用：

```
player.Attack("100"); // 也不会报错
```

👉 JS 根本不知道 C++ 期望 int

---

### TS 的作用来了

TS 可以定义：

```
class Player {  hp: number;  Attack(damage: number): void;}
```

👉 于是：

```
player.Attack("100"); // ❌ 编译期报错
```

---

### 关键点：

> TS 在 Puerts 里 = “C++ API 的类型映射层”

---

## 三、为什么不用 JS 直接写（你最核心的问题）

你说：

> 直接 JS 写不是少一步 TS→JS 编译吗？

这个思路在“小项目”是对的，但在 Puerts 这种场景是错的。

原因有 3 个：

---

### 1. JS 没有类型 → C++ API 无法安全调用

C++ 是强类型：

- int
- float
- struct
- overload

JS 是弱类型：

- 全是 any

👉 如果不用 TS：

所有 C++ 接口 = “黑盒”

---

### 2. TS 不增加运行时成本（关键误区）

你以为：

> TS → JS 多了一步编译

但实际上：

✔ TS 编译只发生一次（开发/打包阶段）  
✔ 运行时完全没有 TS

👉 Puerts运行时：

```
TS → JS（一次性）JS → V8执行（实时）
```

没有额外 runtime cost

---

### 3. TS 带来的是“开发体验革命”，不是性能

如果不用 TS：

- 没有自动补全 C++ API
- 没有参数提示
- 写错参数只能 runtime crash

---

## 四、一个直觉对比（非常重要）

### 只用 JS：

> “你在黑暗里摸 C++ API”

---

### 用 TS：

> “C++ API 被翻译成了带说明书的接口”

---

## 五、Puerts 设计本质（一句话）

> Puerts 用 TS 是为了让 JS “看得懂 C++”，而不是为了“优化 JS 执行速度”。

---

## 六、你这个问题的关键误区总结

你隐含的假设是：

> TS = 多一步编译 = 多余开销

但实际是：

|误解|真相|
|---|---|
|TS 有运行成本|❌ 没有|
|TS 会拖慢运行|❌ 不会|
|JS 更快|❌ 运行一样（都是 V8）|
|TS 只是多余层|❌ 是类型桥梁|

---

## 七、最重要的一句话总结

> TS 的价值不在“替代 JS”，而在“让 JS 能安全地使用大型复杂系统（如 C++ 引擎）”。

---


# TS 与 U++的一些不同
```cpp
1. 模板函数 → 返回基类 + 手动 Downcast       

  // C++: 编译期静态转型                                                                                 
  GetController<APlayerController>();                                                
  Cast<AWeapon>(Actor);       

  NewObject<UMyObject>();
  CreateWidget<UMyWidget>(this);

  // TS: 运行时返回基类，需要 as 断言
  GetController() as APlayerController;
  Actor as AWeapon;  // 或使用 Puerts 的 $cast

  2. 函数重载 → 重命名为不同函数

  // C++: 同名不同参，编译期决议
  AActor::K2_GetActorLocation();  // 蓝图版
  AActor::GetActorLocation();     // C++ 原生版（可能返回引用）

  // TS: 重载被展开或只暴露一个版本
  // 有些重载会被重命名为 Func_N, Func_1 等
  // 或者只暴露反射可访问的那个版本

  3. 运算符重载 → 函数调用

  // C++: 运算符
  FVector A = B + C;
  FRotator R = R1 + R2;

  // TS: 必须调函数
  let A = B.Add(C);
  let R = R1.Add(R2);

  // C++ 的计算效率优势在 TS 里也没了
  // 因为每次操作都产生临时对象

  4. 引用/指针语义 → 全引用类型

  // C++: 值类型 vs 引用类型
  FVector V;            // 栈上值，直接拷贝
  AActor* Ptr;          // 指针，传引用

  void Func(FVector& Out);  // 输出参数

  // TS: USTRUCT 是值拷贝，但写法像引用
  let V = new FVector();      // 堆分配
  let V2 = V;                 // 这拷贝引用还是值？要看绑定实现
  // $ref / $unref 处理输出参数

  5. 枚举 → 值类型（枚举语义丢失）

  // C++: 强类型枚举
  EPhysicalSurface SurfaceType;
  if (SurfaceType == EPhysicalSurface::SurfaceType1) { }

  // TS: 生成成数字常量或字符串
  // 常写成 SurfaceType_1 这样的命名
  // 没有编译期类型检查，传错值不会报错

  6. 委托/事件 → 函数回调

  // C++: 委托绑定
  OnActorHit.AddDynamic(this, &AMyActor::OnHit);

  // TS: 函数赋值
  this.OnActorHit.Add(this.OnHit);
  // 注意：不能传 lambda（无法 GC），必须传成员函数
  // 这经常踩坑

  7. 所有权/生命周期

  - C++: UPROPERTY() 标注控制 GC 根；NewObject 时指定 Outer
  - TS: 所有通过反射创建的 UE 对象由 UE GC 管理，但 TS 侧的引用不会阻止 UE GC
  - 坑: TS 里持有一个 Actor 引用，UE 端该 Actor 被 Destroy 后，TS 侧引用变悬空，访问会崩溃

  8. FORCEINLINE / 性能敏感函数

  // C++: 引擎大量内联函数直接调用
  FVector::operator+ 可能就是几条 SIMD 指令

  // TS: 每个属性访问都是跨语言调用（C++ → V8/QuickJS）
  // 每帧循环里大量调用 UE API 会非常慢
  // 性能关键逻辑建议：
  //   1. batch 操作放 C++ 侧
  //   2. 用 UE 的 Tick 代替 JS 循环
  //   3. 避免在 TS 里每帧频繁操作 FVector/FTransform

  9. 宏/属性修饰符

  // C++: 各种 UE 宏
  UFUNCTION(BlueprintCallable, Category = "xxx")
  UPROPERTY(EditAnywhere, BlueprintReadWrite)

  // TS: 看不到这些元信息
  // 只能用 Puerts 的装饰器或声明文件
  // 有些 EditorOnly 的函数在打包后可能不存在

  10. WITH_EDITOR / 平台宏

  // C++: 预处理器条件编译
  #if WITH_EDITOR
  void DebugFunc();
  #endif

  // TS: 所有函数都有，没有条件编译
  // 编辑器的函数在 shipping 包调用会崩溃

  

 

```
![](图片/UE/PuerTs/PuerTS的一些知识/IMG-20260707164554852.png)
 简单原则：写 Puerts TS 时，把它当作一种"用 JS 语法调用 UE 反射 API"的工具，而不是"用 JS 写 UE C++"。碰到奇怪行为时，先想想这个功能在 C++ 里是不是用了模板/宏/运算符重载/引用参数——多半就是差异来源。

# Puerts 里 @mixin 做的事情是：
  UE.Class.Load(蓝图) → blueprint.tojs() → blueprint.mixin(注册 TS 方法覆盖)
  这个注册必须在对应蓝图对象被创建之前完成。如果 UMG_EnemyBar 的 mixin 通过嵌套 import 链加载时晚了半拍， Enemy Actor 已经在场景中，它的 Bar WidgetComponent 已经创建了 UMG_EnemyBar 实例 —— 这个实例就不会带 TS 扩展的
  HP、Max_HP、GetPercent 等方法。

# 原型链
## 一个崩溃问题
```ts
// BP_Enemy.ts (崩溃时的版本)                              
  ReceiveTick(DeltaSeconds: number) {                 
      super.ReceiveTick(DeltaSeconds);                                             
      this.SetBarRotation();                                     
  }
```
- 崩溃原因分析:
	- 核心问题在于 Puerts @mixin 的原型链 + 父类未定义 ReceiveTick。
![](图片/UE/PuerTs/PuerTS的一些知识/IMG-20260707164554945.png)



# PuerTS新东西
JavaScript/TypeScript 中，原始类型（bool、number等）是按值传递的，函数内部修改不了外部的变量。
PuerTS提供了包装机制：
$ref(value) —— 创建引用包装

会在特殊函数的参数中用到这种需要传递引用的变量
bSuccess: $Ref\<boolean> = $ref(false);
![](图片/UE/PuerTs/PuerTS的一些知识/IMG-20260707164554999.png)

# 开销
 - 调用链开销
  C++ 直接调用:        C++ → C++           （最快，无桥接）
  C++ 调 TS:           C++ → V8 JS Bridge → JS  （有桥接开销）
  TS 调 UE 函数:       JS → V8 JS Bridge → C++  （有桥接开销）
  蓝图调用:            蓝图 VM → C++        （也有桥接，且蓝图 VM 是解释执行）
蓝图是解释执行的 VM，而 Puerts 用的是 V8 引擎，JS 是 JIT 编译的，纯计算逻辑（循环、数学运算、字符串处理）TS 比蓝图快很多
![](图片/UE/PuerTs/PuerTS的一些知识/IMG-20260707164753730.png)

# 类中变量的命名区别
```ts
@mixin(AssetPath)
  export class BP_BaseCharacter implements BP_BaseCharacter {
      Die: boolean = false;                    // 实例字段
      ABP_Sinbi: UE.xxx = null;               // 实例字段
      static MaxHP: number = 100;              // 静态字段
      
      ReceiveBeginPlay(){ ... }               // 方法 ← 只有这个会被 mixin 注入
  }

```

普通类 new 一个实例时，构造函数会执行所有字段初始化器（Die = false、ABP_Sinbi = null）。

  但 mixin 模式下，实例是由 UE 引擎创建的，不是 JS 的 new，所以：
  - TS 类的构造函数不会被调用
  - 字段初始化器（= false、= null）不会执行
  - Die、ABP_Sinbi 在 ReceiveBeginPlay 之前都是 undefined


# 蓝图与ts同名变量
在 PuerTS 的 mixin 机制下，同名变量会互相覆盖，具体表现：                                                           
  Blueprint 有 UPROPERTY，TS 也声明了同名字段：                                                                                                                                                                                    
  - TS 侧的声明会遮蔽（shadow） Blueprint 的属性
  - TS 代码访问的是 TS 侧存的值 
  - Blueprint 内部（蓝图逻辑）访问的仍然是自己 UPROPERTY 存的值 
  - 两边各存各的，数据不同步

  举例： 假设 Blueprint 里有个 Health = 100 的变量，你在 TS 里也写了：

  Health: number;  // 这只是类型声明，undefined

  如果你在 TS 里给 this.Health = 50，只有 TS 侧改成 50，蓝图那边仍然是 100。
