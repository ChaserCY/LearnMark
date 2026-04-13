# ==可能出现的问题==
1. 编辑器中创建了C++类，但没有生成，显示出来，需要点击右下角的编辑
2. VS编辑器中，没有显示完全文件夹和相关文件，需要右键.uproject重构
3. 头文件中声明的函数必须实现(写在Cpp文件中，空的也可以)
4. 使用新建文件(.h)编写代码时，无法识别关键字，需要退出去，重新构建一下，如2
5. 代码出现语法错误，在众多警告中会产生错误，需要仔细阅读“输出栏”
# 1. 制版资产

拖入Assets文件夹到Content文件下里，导入第三人称蓝图游戏模板

# 2.
## 2-1. Hard/Soft Reference （软硬引用）

### 核心区别：有没有立刻把资源加载进内存

### 实例
Hard ：
- 一个蓝图变量直接指定了某个 Blueprint Class
- 一个 Actor 组件里直接挂了某个材质、网格、动画蓝图
- C++ 里 `UPROPERTY()` 直接保存了某个 `UObject*`
- 蓝图里某个节点直接使用了某个资源类
Soft：
-  `TSoftObjectPtr<UTexture2D>`
- `TSoftClassPtr<AActor>`
- 蓝图中 Soft Object Reference / Soft Class Reference
直接存储的引用路径，用时才会调用
- ![](图片/UE/U++/GAS_/IMG-20260324211246195.png)
### 使用情况区别
硬![](图片/UE/U++/GAS_/IMG-20260324211430809.png)软![](图片/UE/U++/GAS_/IMG-20260324211448727.png)

## 2-2. TObjectPtr
### 命名：TObjectPtr\<AActor> ActorPtr;
### 特性：
- Access Tracking :更利于引擎做对象引用管理、编辑器检查、调试和一些底层优化
- Lazy Loading :  在某些场景下，`TObjectPtr` 对编辑器里的资源访问和对象解析更友好,更灵活处理对象引用
### 限制(Limitation):
- Only member variables : 一般只用于成员变量（`UCLASS` / `USTRUCT` 的字段里）
- Only works in editor：在‘’编辑器相关能力‘’上额外收益，比如对象追踪，引用处理，工具链支持等
- No impact on performance：不是拿来做性能优化的
### 总结：
`TObjectPtr` 是 UE5 推荐的一种 `UObject` 指针写法，  
它能让引擎更好地跟踪对象引用，  
但它主要用于成员变量，优势更多体现在编辑器和引擎管理层面，  
不是用来替代软引用，也不是为了直接提升性能。


# 3.游戏框架及人物，武器...
## 3-1. 创建相关C++基类
- GameMode
	- GameBaseMode
		- BP_BaseGameMode
- Character
	- BaseCharacter
		- WarriorHeroCharacter
			- BP_HeroCharacter
		- WarriorEnemyCharacter
- PlayerController
	- WarriorBaseController
		- WarriorHeroController
			- BP_HeroController

## 3-2. 创建主要角色蓝图
右键C++类--创建相关蓝图--编辑GameMode中的Pawn,Controller--世界场景设置中添加

## 3-3. 创建打印日志相关函数
- 创建头文件.h
	- 右键public
	- 位置选择在这个public下
- 其他cpp文件调用
	- 包含这个头文件（#include）
	- 