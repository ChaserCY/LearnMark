# 概念讲解
## 是什么
- 启动js运行环境 V8 Quickjs NodeJs 三引擎选择
- 实现js运行环境和引擎的交互
- 生成ue引擎的api的ts声明
### 打开方式一
- 自己新建一个或者多个Js环境
- JS环境中的TS/JS可以调用引擎API
	- 宏标记后的可以直接访问
	- 其他API可以通过拓展函数，生成代码访问
- 引擎通过==动态多播==访问TS/JS
### 打开方式二
- TS里直接继承UE的一个类，类似原生UEC++开发
### 两种方式对比
- UI、网络，Tick等事件驱动TS/JS逻辑，TS/JS调用服务器或者调用引I擎展示
- 脚本和非脚本界限分明
- 支持多JS环境，SDK开发首选
- 需要从项目初期介入
- 需要做框架设计
---沿用UE的Gameplay框架---
- UE程序猿平滑过渡
- 沿用UE编辑器的各种能力，体验上接近引擎原生支持
- 可在项目任意阶段介入
- 目前还未完全覆盖蓝图的能力
- 不支持多JS环境
## 介绍
### 工具链
- 主流IDE都有TS的支持
- 实现了websocket+v8 inspector协议
	- 主流IDE上断点调试TS
	- CPU Profiler(VSCode, Chrome dev tool)
	- 内存快照(Chrome dev tool)
	- Hot-reload(Chrome dev tool)
- Web常用工具链
	- Webpack
	- Gulp
### 代码健壮性
- 静态类型检查
	- 能==在编辑器检查参数==错误
- 引用对象生命周期跟踪
	- 背景：UE下对象即使声明了引用，仍可能被释放
	- 尝试==访问/传递无效对象==，抛==js==异常

### 性能
- 脚本执行效率
	- 除ios均可开启jit
		- 对android 低端机有帮助
		- 对服务器ds有帮助
	- 开启jit后高于lua，不开启和lua差不多
- 脚本和引擎交互
	- 反射高于xlua的静态生成代码
![](图片/UE/PuerTs/PuerTS的一些知识/IMG-20260908143812104.png)
### 对象存储效率
- 多份同结构数据最为常见
	- 面向对象：类->对象
	- 策划陪表数据
	- 消息
- 同样对象结构数量，优于Lua

## 整体实现
### 涉及技术
- 实现了什么
	- 启动js引擎
	- 支持互访
	- 对象持有
- 涉及技术
	- v8拓展API
	- UE反射API
	- UE GC API
		- 解决 UE 和 V8 两套垃圾回收互相不知情导致的崩溃 / 内存泄露
	- Libclang
		- 静态代码生成工具（生成 Wrapper 胶水代码）底层
		- 静态绑定代码生成器的源码解析内核，扫描 C++ 头文件自动产出胶水代码
### 反射调用
- UE函数反射调用关键点
	- 参数和返回值组成结构体
- 流程
	- 栈分配/初始化结构体
	- 从JS取出参数设置到结构体
	- UObject::ProcessEvent
	- 将返回值/输出参数返回给JS
	- 结构体解析
**UObject::ProcessEvent：UE 反射系统的函数执行入口，用来动态执行 UFunction。Puerts 动态调用 UE 函数底层依赖该接口；优点是灵活无需预生成绑定代码，缺点是存在反射开销。**

### 对象持有
- 对象池
	- FGCObject::AddReferencedObjects
- UE对象入池，构造JS影子对象，JS影子对象持有UE对象指针
- JS影子对象被JS虚拟机gc释放，触发回调，UE对象出池
- 监听UE对象强制释放，标记JS影子对象为无效对象
	- FObiectDeletelistener:NotifvUObiectDeleted

### Dynamic_Delegate访问TS/JS
- UDynamicDelegateProxy（普通的UCLASS），成员记录了绑定
的虚拟机，JS函数
- 覆盖UObject的ProcessEvent方法
	- void ProcessEvent(UFunction*, void* Params)
- Delegate触发，ProcessEvent被调用
	- 从Params调用信息解出参数
	- 调用js函数
	- 如果有返回值，回写返回值到Params

## 蓝图代理自动生成实现剖析
### 背景
- 已经成型的项目较难以使用
- unlua实现了lua“覆写”功能
- TypeScript有静态类型
- TypeScript编译器开放API
![](../../Pasted%20image%2020260908151639.png)
### 编辑器下代理蓝图生成
![](图片/UE/PuerTs/PuerTS的一些知识/IMG-20260908151022681.png)
![](图片/UE/PuerTs/PuerTS的一些知识/IMG-20260908151648287.png)
### 运行时绑定方案
- TS构造函数实现
	- 指明生成的蓝图的GeneratedClass
		- UTypeScriptGeneratedClass
		- 该类会记录TS类构造函数调用信息
	- 设置UClass的ClassConstructor，实现为调用TS的构造函数
		- 该方式支持CreateDefaultSubobject
		- 该方式支持CDO的初始化
- 代理蓝图空函数绑定
	- 替换空函数的UFuntion为UJSGeneratedFunction
	- UJSGeneratedFunction记录了绑定的TS函数调用信息
	- SetNativeFunc修改UFunction的逻辑，改为调用特定TS函数
	- 把UJSGeneratedFunction的属性设置为Native，阻止通过UFuntion.Script执行

### Hot-reload实现
- 蓝图重绑定
	- 如果有对应的代理蓝图，设置对应UTypeScriptGeneratedClass的脏标记
	- NotifyUObjectCreated，如果检测该对象对应的UTypeScriptGeneratedClass为脏，重新绑定
- JS代码Hot-reload
	- 目前用自己实现的代码重加载
	- 准备改为使用V8内置的hot-reload
- 效果：做到运行中代码修改马上生效

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



# PuerTS新东西(值与引用修改)
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


# 资源加载
加载蓝图类/实例区别
![](图片/UE/PuerTs/PuerTS的一些知识/IMG-20260721222524127.png)

# TS可否完全替代蓝图？
## 可行分类
==完全可行==：游戏逻辑，数据处理，网络同步 
==不可行==：   动画蓝图，行为树，材质，时间轴，UI   

## 优劣
==TS优势：==
  - 代码复用、类型安全
  - 更容易进行单元测试
  - 版本控制友好（文本 diff）
  - 跨平台一致性强

  ==劣势：==
  - 调试不如蓝图直观（断点、可视化）
  - 团队学习曲线（如果习惯蓝图）
  - 某些实时编辑场景不如蓝图灵活
  - 性能：蓝图在某些场景有优化（如 Tick）

## 混合使用
![](图片/UE/PuerTs/PuerTS的一些知识/IMG-20260721223244991.png)

# Mixin完整混入流程实例
**常驻池里的对象，绝对==不能引用==后来动态创建、需要被 GC 回收的普通对象**
1. **C++ 原生类（Native Class）与蓝图类的区别：**
    
    - **C++ 类（如 ABasePlayerController）**：在引擎启动和模块加载的最早期由 Native 代码生成。在进入游戏主循环前，所有 C++ 类的 UClass 以及它的类结构（包括挂在类上的函数定义）都会被引擎视为**永久常驻内存**，进入 **Disregard for GC 池**。
        
    - **蓝图类（BlueprintGeneratedClass）**：是作为普通资产加载的，它们通常**不属于** Disregard for GC 池，参与正常的 GC 引用计数，因此怎么 mixin 都不会破坏这个假设。
	实在想要C++类，只有 在 C++ 中手动把该 UFunction 设为 Root
2. **Puerts mixin 在底层干了什么：**
    
    - 当你在 TypeScript 里执行 mixin(BasePlayerController, ...) 时，Puerts 底层会为被混入的方法（比如 ReceiveBeginPlay）**动态生成或修改一个 UFunction 对象**，并将其挂入 BasePlayerController 的==函数列表/反射表==中。
        
    - 这个动态生成的 UFunction 是普通创建的对象，**没有被打上常驻根集合（Root Set）标记**。
        
    - 结果就是：**常驻池里的 Native Class 引用了一个普通需要 GC 的动态 UFunction**，在启动阶段的 VerifyGCAssumptions() 检测中被引擎直接当作致命异常截获。
3. **Root**
	在虚幻引擎中，**“非 Root 的 UObject” 指的就是：没有被加入到“根集合（Root Set）”中、随时可能因为没人引用而被垃圾回收（GC）销毁的普通 UObject 对象。**
	
	其实，你在游戏中 **99% 动态创建出来的 UObject，默认全都是“非 Root”的**。
	 它的命运完全取决于**有没有一条由 UPROPERTY() 构成的引用链条把它连到某个 Root 对象上**。

