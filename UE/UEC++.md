## 编辑器的设置
设置'Asset Editor Open Location'为'Main Window' 修改新弹出窗口的出现位置
Ctrl + Shfit +W  修改编辑器视角大小
修改.h头文件中的原始指针

## 项目结构


![[Pasted image 20251130135106.png]]
Engine ： 引擎源码文件
Games ：编写逻辑文件
Programs ： 存放引擎运行所需要的库，文件之类的
Rules ： 所需要的规则
Visualizers：虚幻5.1以后添加的运行配置文件

开始学时的重点关注对象：
**![[Pasted image 20251130135349.png]]**
Config：存放项目配置文件
Source：存放C++的源代码文件

![[Pasted image 20251130135738.png]]
存放每个模块的环境依赖信息

![[Pasted image 20251130140014.png]]
.vs: IDE的项目配置文件，工程设置和解决方案文件
Binaries ： 编译后的可执行文件，动态链接库
Config：存放项目配置文件
Content ：游戏内资产内容
DerivedDataCache：缓存项目的派生数据
Intermediate ： 保存临时文件和中间文件
Saved：自动保存文件，日志文件，截图，本地用户的设置
Source：Source：存放C++的源代码文件，逻辑和功能
.vsconfig: vs的配置文件
最后两个用来启动 VS和UE的

## 蓝图和C++的关系
![[BP_C++.canvas]]

## 创建和删除C++类
创建：在C++类文件夹中创建对应私有/公有的C++类对象，取消应用（VisualStudioTools插件）
删除：需要在项目文件夹中删除对应文件，然后删除Intermediate和Binaries文件，重新右键
![[Pasted image 20251130172519.png]]
打开：完成上述操作后，通过VS来打开  .sln 来启动项目

## C++->蓝图
游戏模式重载，使用蓝图继承C++类的游戏模式，利于修改
## 正式学习
### UPROPERTY
- **UPROPERTY**是UnrealEngine中==用于声明属性的宏==，它用于标记某个属性是一个UnrealEngine托管的属性，并且可以在编辑器中进行访问和操作。
- **UPROPERTY**宏提供了一系列参数，用于定义属性的属性和行为，例如是否可编辑、是否可序列化等。
![](图片/UE/UEC++/IMG-20251227202058421.png)
注意事项：
1.这个属性和变量声明在.h文件中，并且括号内可以无序无数量限制，用，来继续添加属性
2.里面的public，private的作用范围是遇到下一个==访问控制说明符==为止
3.一对一标记
例如：
![](图片/UE/UEC++/IMG-20251227215722447.png)



### UFUNCTION
- **UFUNCTION**是UnrealEngine中==用于声明函数的宏==，它用于标记某个函数是一个UnrealEngine托管的函数，并且可以在编辑器中进行访问和操作。
- **UFUNCTION**宏提供了一系列参数，用于定义函数的属性和行为，例如是否是蓝图可调用的、是否可在网络中复制等。
参数：
![](图片/UE/UEC++/IMG-20251227202246890.png)
注意事项：
1.在.h文件中声明函数属性，返回值，名称，参数，在cpp文件中重写函数内容
2.一对一标记
例如：
![](图片/UE/UEC++/IMG-20251227220527863.png)
