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
