# 安装与配置
注意：必须是C++项目才妥当
需要在 `.Build.cs` 里添加对 PuerTS 模块的依赖
![](图片/UE/U++/PuerTS/IMG-20260508150846117.png)

配置TS开发环境
![](图片/UE/U++/PuerTS/IMG-20260508152144169.png)


PuerTS插件设置
![](图片/UE/U++/PuerTS/IMG-20260508152411043.png)


如何调试：
![](图片/UE/U++/PuerTS/IMG-20260508152827596.png)

编写TS逻辑
![](图片/UE/U++/PuerTS/IMG-20260508153340648.png)

TS继承引擎类
![](图片/UE/U++/PuerTS/IMG-20260508153542556.png)
![](图片/UE/U++/PuerTS/IMG-20260508160704073.png)


蓝图Mixin
![](图片/UE/U++/PuerTS/IMG-20260508153826023.png)
**要求**：
- 每个蓝图只 mixin 一次
- 必须在 JS 虚拟机初始化逻辑中或使用前执行 `mixin`
- 避免 Actor 构造后才 mixin，否则 TS 回调不会生效


实例：
![](图片/UE/U++/PuerTS/IMG-20260508153929926.png)


对比
![](图片/UE/U++/PuerTS/IMG-20260508154153458.png)
坑
![](图片/UE/U++/PuerTS/IMG-20260508154323877.png)


加载时序问题
![](图片/UE/U++/PuerTS/IMG-20260508154802215.png)
![](图片/UE/U++/PuerTS/IMG-20260508154820065.png)

拓展初始化窗口
![](图片/UE/U++/PuerTS/IMG-20260508155909863.png)
![](图片/UE/U++/PuerTS/IMG-20260508155948682.png)



![](图片/UE/U++/PuerTS/IMG-20260508155713534.png)


初始化虚拟机的地方
==先进后出==
![](../../Pasted%20image%2020260508160911.png)
![](图片/UE/U++/PuerTS/IMG-20260508161632864.png)
![](图片/UE/U++/PuerTS/IMG-20260508161824827.png)

打包注意
![](图片/UE/U++/PuerTS/IMG-20260508162034759.png)


![](图片/UE/U++/PuerTS/IMG-20260508162128141.png)
测试时可以在编辑器测试/打包后测试





# puerts::FJsEnv 启动虚拟机

这里的虚拟机功能：
语言语法  
对象模型  
函数调用  
作用域 / 闭包  
原型链  
Promise / async  
内存管理 / GC  
模块加载  
标准库能力  
和**宿主环境交互的接口**

一个puerts::FJsEnv实例代表一个虚拟机实例（可以类比一个nodejs进程）

- 在合适的地方（比如GameInstance）根据需要构造一个或者多个虚拟机
    - 如果启动多个虚拟机，这些虚拟机间是相互隔离的（可以类比多个nodejs进程间数据是隔离的）

- 通过Start函数启动一个脚本，作为脚本逻辑的入口（类似c的main函数）    
    - Start可以传入一些数据作为参数，供脚本获取使用（可以类比为nodejs命令输入的入口脚本）

- 启动：在C++的游戏实例文件中启动
- 写TS：在Content/JavaScript文件夹中写.ts文件
# 继承引擎类功能
![](图片/UE/U++/PuerTS/IMG-20260508150855053.png)

# 关系
- 一个puerts::FJsEnv就是一个虚拟机实例，虚拟机间数据是隔离的。
    
- 蓝图mixin功能是puerts::FJsEnv的一个特性，或者说任意一个puerts::FJsEnv都有蓝图mixin功能。
    
- 继承引擎类功能是基于puerts::FJsEnv之上构建的功能
    
    - 它会分析ts类，并生成相应的（代理）蓝图，代理蓝图的特点是只有数据，没有逻辑（空函数）；
        
    - 代理蓝图启动，空函数会被重定向到ts（准确来说是ts编译后的js），对于ue引擎来说，代理蓝图就是普通蓝图，并无特殊性；
        
    - 由于（代理）蓝图由ue引擎实例化，其构建就关联到某个虚拟机上的js逻辑，并需要执行构造逻辑，于是该功能需要启动一个默认的puerts::FJsEnv，这个puerts::FJsEnv和你new的puerts::FJsEnv实例没有区别，也有蓝图mixin功能，而且根据第一条，和其它地方new的puerts::FJsEnv是相互隔离的。


# Puerts文件模板缘由
 这是一个非常典型且必要的 PuerTS 模式，核心原因在于编译时类型安全和运行时连接是两个独立的问题，需要分开解决。                                                                                                              
  为什么需要这个接口                                                                                                                   
  1. 蓝图是运行时构造，TypeScript 编译时看不到它                                                                                                                                                                          
  蓝图（.uasset）是 Unreal 编辑器里的可视化资产，在 TypeScript 编译时根本不存在。当你写 this.AbilitySystemComponent 时，TypeScript 编译器不知道这个东西是什么——除非你有类型声明。   
  
  2. 接口的作用：提供编译时类型信息                                                                                                                                                                                              

  // 这个接口继承了 PuerTS 自动生成的蓝图类型定义
  export interface BP_BaseCharacter extends UE.Game.BluePrints.Character.BP_BaseCharacter.BP_BaseCharacter_C {}

  BP_BaseCharacter_C 是 PuerTS 从蓝图资产自动生成的 TypeScript 类型声明，里面包含了蓝图里定义的所有属性（如 AbilitySystemComponent）、委托（如 HPChanged）、函数等。你的空接口继承了它，就把这些类型信息带进来了。

  然后类声明 implements BP_BaseCharacter 之后，TypeScript 编译器就知道 this.AbilitySystemComponent、this.HPChanged.Add(...) 这些是合法成员，不会报红。

  3. @mixin 装饰器的作用：提供运行时连接

  @mixin(AssetPath)
  export class BP_BaseCharacter implements BP_BaseCharacter { ... }

  @mixin 在运行时做三件事：
  - 通过路径加载实际的蓝图类
  - 用 blueprint.tojs() 把它转成 JS 可调用形式
  - 用 blueprint.mixin() 把你写的 TypeScript 类方法合并到蓝图对象上

  总结：两个独立问题，两个独立解法┌───────────────────────────────────────┬─────────────────────────────────────────────────────┬────────┐
  │                 问题                  │                        解法                         │  时机  │
  ├───────────────────────────────────────┼─────────────────────────────────────────────────────┼────────┤
  │ TypeScript 编译器不认识蓝图里的成员   │ interface BP_BaseCharacter extends ... + implements │ 编译时 │
  ├───────────────────────────────────────┼─────────────────────────────────────────────────────┼────────┤
  │ TypeScript 代码需要实际连接到蓝图对象 │ @mixin(AssetPath) 装饰器                            │ 运行时 │
  └───────────────────────────────────────┴─────────────────────────────────────────────────────┴────────┘

  两者缺一不可——没有接口，编译报红；没有 @mixin，代码能编译通过但运行时根本不生效。

