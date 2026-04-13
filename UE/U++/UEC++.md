 ## 编辑器的设置
设置'Asset Editor Open Location'为'Main Window' 修改新弹出窗口的出现位置
Ctrl + Shfit +W  修改编辑器视角大小
修改.h头文件中的原始指针
Compile：关闭实时代码编写
## 代码结构
### 头文件和源文件
**头文件 (.h)** 和 **源文件 (.cpp)** 这种设计是 **C++ 语言的特性**，但虚幻引擎（UE）利用这种特性构建了它强大的 **“反射系统”（Reflection System）**，也就是让 C++ 代码能和编辑器（蓝图）交互的核心机制。
- .h 文件（头文件） = 餐厅的菜单
	只列出名字和属性，**不包含具体做法**
- .cpp 文件（源文件） = 后厨的厨师
	**干实事的地方**，这里写具体的逻辑（if/else, for循环等）

### 修改C++类文件位置
直接在编辑器中移动，对应的.h .cpp文件，且必须修改.cpp文件中的#include 引用头文件的地址

### 源文件中的内容格式(::)
:: 在 C++ 中叫做 **作用域解析运算符**

- 在 **.h 文件**（菜单/声明）里，你是在各自的家里说话：
    
    - 在玩家家里说：“我们要有个 Attack。”（大家都知道是指玩家的攻击）
        
    - 在怪物家里说：“我们要有个 Attack。”（大家都知道是指怪物的攻击）
        
- 在 **.cpp 文件**（厨房/实现）里，这是一个**公共区域**。如果你只写：
    ```cpp
    void Attack() { ... }
    ```
    
    编译器就会困惑：“这是谁的攻击？是玩家的？是怪物的？还是一个不属于任何人的全局攻击？”
    
    所以你必须加上前缀：
    ```cpp
    void AMy_Player::Attack() { ... }  // 这是玩家的攻击逻辑
    void AMy_Monster::Attack() { ... } // 这是怪物的攻击逻辑
    ```

- **在 .h 文件中**：  
    代码是被包裹在 class 的大括号里的：
    ```cpp
    class AMy_Player : public ACharacter 
    {
        // 这里面的所有东西默认都属于 AMy_Player
        virtual void BeginPlay() override; 
    };
    ```
    
    因为在括号内部，所以不需要重复强调“我是 AMy_Player 的”。
    
- **在 .cpp 文件中**：  
    代码是平铺直叙的，默认处于**全局作用域**。

## 项目结构


![[图片/UE/U++/UEC++/IMG-20260301171708682.png]]
Engine ： 引擎源码文件
Games ：编写逻辑文件
Programs ： 存放引擎运行所需要的库，文件之类的
Rules ： 所需要的规则
Visualizers：虚幻5.1以后添加的运行配置文件

开始学时的重点关注对象：
**![[图片/UE/U++/UEC++/IMG-20260301171708709.png]]**
Config：存放项目配置文件
Source：存放C++的源代码文件

![[图片/UE/U++/UEC++/IMG-20260301171708750.png]]
存放每个模块的环境依赖信息

![[图片/UE/U++/UEC++/IMG-20260301171708786.png]]
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

## 打开项目
打开项目可能出现
![](图片/UE/U++/UEC++/IMG-20260302135942814.png)
需要重复打开重新生成的步骤：
1. 进入对应项目文件夹(比如：Fanhua)
2. **删除缓存**：删掉 Binaries、Intermediate 和 Saved 这三个文件夹（如果有 .vs 也删掉）。
3. **生成项目文件**：右键点击 .uproject 文件，选择 **Generate Visual Studio project files**
4. **打开 VS**：双击生成的 .sln 文件打开 Visual Studio
5. **编译**：在 VS 里右键点击项目名称，选择 **Build（生成）** 或 **Rebuild（重新生成）**（上方需要选择Development Editor     Win64） 。 **遇到诡异报错、升级引擎、或者刚下载别人项目时**：右键项目 -> **重新生成**，以此排除缓存干扰。
___生成：增量编译，，，重新生成：全部清理+生成___
![](图片/UE/U++/UEC++/IMG-20260302140237382.png)
6. 观察底部输出报错，无误后，在文件夹中打开.uproject 文件
7. 确实一直出错，可能是==.Build.cs   和   .uproject==配置有问题，少了点插件和其他配置



## 蓝图和C++的关系
![[画板结构图/BP_C++.canvas]]

## 创建和删除C++类
创建：在C++类文件夹中创建对应私有/公有的C++类对象，取消应用（VisualStudioTools插件）
删除：需要在项目文件夹中删除对应文件，然后删除Intermediate和Binaries文件，重新右键
![[图片/UE/U++/UEC++/IMG-20260301171708826.png]]
打开：完成上述操作后，通过VS来打开  .sln 来启动项目
![](图片/UE/U++/UEC++/IMG-20260301171708865.png)

## C++->蓝图
游戏模式重载，使用蓝图继承C++类的游戏模式，利于修改
## 正式学习
### 2.2UPROPERTY和UFUNCTION
- **UPROPERTY**是UnrealEngine中==用于声明属性的宏==，它用于标记某个属性是一个UnrealEngine托管的属性，并且可以在编辑器中进行访问和操作。
- **UPROPERTY**宏提供了一系列参数，用于定义属性的属性和行为，例如是否可编辑、是否可序列化等。
![](图片/UE/U++/UEC++/IMG-20260301171708909.png)注意事项：
1.这个属性和变量声明在.h文件中，并且括号内可以无序无数量限制，用，来继续添加属性
2.里面的public，private的作用范围是遇到下一个==访问控制说明符==为止
3.一对一标记
4.Meta作用：主要用于控制编辑器（Editor）界面的显示效果和交互行为
例如：
![](图片/UE/U++/UEC++/IMG-20260301171708945.png)



#### UFUNCTION
- **UFUNCTION**是UnrealEngine中==用于声明函数的宏==，它用于标记某个函数是一个UnrealEngine托管的函数，并且==可以在编辑器(蓝图节点)中进行访问和操作。==
- **UFUNCTION**宏提供了一系列参数，用于定义函数的属性和行为，例如是否是蓝图可调用的、是否可在网络中复制等。
参数：
![](图片/UE/U++/UEC++/IMG-20260301171708984.png)
注意事项：
1.在.h文件中声明函数属性，返回值，名称，参数，在cpp文件中重写函数内容
2.一对一标记
例如：
![](图片/UE/U++/UEC++/IMG-20260301171709022.png)

BP_函数名  和 函数名  两种节点
- “函数名”节点：
	- 普通 C++ 函数 (BlueprintCallable)
	- **图标**：一个简单的蓝色 f。
	- **程序员（你）在 C++ 里写**
- “BP_函数名”节点：
	-  蓝图实现事件 (BlueprintImplementableEvent)
	- **图标**：通常是一个菱形 ◇ 或者是带有箭头的图标（取决于具体是 Event 还是函数重载）。
	- **策划/美术（或者作为蓝图开发者的你）在蓝图里写**
![](图片/UE/U++/UEC++/IMG-20260301171709057.png)

#### Meta参数的作用 
控制编辑器界面的 UI 展示
这是 meta 最常见的用途。它可以让变量在编辑器面板中看起来更专业、更易用。
DisplayName: 修改变量在编辑器中显示的名称，而不改变代码中的变量名。
```cpp
UPROPERTY(EditAnywhere, meta = (DisplayName = "Movement Speed"))
float MyInternalVariable; // 在编辑器里显示为 "Movement Speed"
```
ToolTip: 自定义鼠标悬停时的提示（虽然注释也会自动转为提示，但 meta 可以覆盖它）。
Category: 虽然可以写在宏的第一级，但在 meta 中也可以细化分类。

2. 数值限制与约束
在编辑器中输入数值时，meta 可以防止策划输入非法数值。
ClampMin / ClampMax: 限制在编辑器中输入的数值范围。
UIMin / UIMax: 限制滑动条（Slider）的范围，但手动输入可以超过这个值。
```cpp
UPROPERTY(EditAnywhere, meta = (UIMin = "0", UIMax = "100", ClampMin = "0"))
float Health;

```

3. 增强 Blueprint（蓝图）的功能
有些 meta 标签直接改变了蓝图节点的呈现方式。
BlueprintThreadSafe: 标记函数为线程安全，常用于动画蓝图。
WorldContext: 标记哪个参数是世界上下文（通常在静态函数库中使用，会自动隐藏该输入引脚）。
DeterminesOutputType: 根据输入参数的类型自动决定输出引脚的类型（常用于 GetActorOfClass 这种泛型操作）。
```cpp
UFUNCTION(BlueprintCallable, meta = (DeterminesOutputType = "Class"))
UObject* SpawnSomething(TSubclassOf<UObject> Class);

```

4. 逻辑控制与可见性
AllowPrivateAccess: 允许在蓝图中访问类的 private 变量。
```cpp
private:
    UPROPERTY(VisibleAnywhere, meta = (AllowPrivateAccess = "true"))
    class USphereComponent* SphereComponent;

```

EditCondition: 只有当某个布尔值为真时，该属性才在编辑器中可编辑。

```cpp
UPROPERTY(EditAnywhere)
bool bEnableSpeed;

UPROPERTY(EditAnywhere, meta = (EditCondition = "bEnableSpeed"))
float Speed; // 只有勾选了 bEnableSpeed，这里才能改

```

5. 资源筛选
AllowedClasses: 限制 FSoftObjectPath 或 UObject* 只能选择特定的类。
```cpp
UPROPERTY(EditAnywhere, meta = (AllowedClasses = "Texture2D, StaticMesh"))
FSoftObjectPath AssetLink;

```

### 2.3\#include 引入头文件与不完全声明
#### 前置声明
![](图片/UE/U++/UEC++/IMG-20260301171709100.png)

#### 前置声明的两种写法
1. 类前前置声明
2. 用时前置声明

#### 演示示例(创建组件)
![](图片/UE/U++/UEC++/IMG-20260301171709139.png)
创建步骤：
- **.h头文件中**
1. 引入组件头文件(类前前置声明)：
class USpringArmComponent;
class UCameraComponent;
USpringArmComponent   摄像机弹簧臂组件
UCameraComponent         摄像机视角组件
2. 增加反射
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Carmera, meta=(AllowPrivateAccess="true"))
3. 命名组件指针
TObjectPtr\<USpringArmComponent> ==CameraBoom==;
TObjectPtr\<UCameraComponent>    ==PlayerCamera==;
- **.cpp文件中**
1. 组件头文件包含
\#include "Camera/CameraComponent.h"
\#include "GameFramework/SpringArmComponent.h"
2. 在实例化函数中初始化
==CameraBoom== = CreateDefaultSubobject<\USpringArmComponent>("CameraBoom");            创建组件
==CameraBoom==->SetAttachment(RootComponent);           设置组件的指向(附着)
==CameraBoom==->TargetArmLength = 300.f;          设置组件长度
==CameraBoom==->SetRelativeRotation(FRotator(70.f，0.f，0.f));    设置组件角度

==PlayerCamera== = CreatDefaultSubObject<\UCameraComponent>("Player Camera");     创建摄像机
==PlayerCamera== = SetupAttachment(CameraBoom);          相机绑定在杆组件上

### 2.4 UE的日志打印两种方式
- 在输出日志中的打印：
![](图片/UE/U++/UEC++/IMG-20260304215317381.png)
```cpp
UE_LOG(LogTemp, Warning, TEXT("CameraBoom : TargetArmLength: %f"), CameraBoom->TargetArmLength);
```
![](图片/UE/U++/UEC++/IMG-20260304215844123.png)

- 在游戏屏幕上打印
```cpp
GEngine->AddOnScreenDebugMessage(-1, 10, FColor::Blue, FString::Printf(TEXT("CameraBoom->TargerArmLength: | %f"), CameraBoom->TargetArmLength));       //第一个参数是-1，表示每次打印加一行，其他数字表示编号，直接覆盖；第二个参数是显示时长(s)
```


![](图片/UE/U++/UEC++/IMG-20260304215327258.png)



### 3.1 创建Inputmapping Content 和 Input Action
![](图片/UE/U++/UEC++/IMG-20260307202320353.png)
