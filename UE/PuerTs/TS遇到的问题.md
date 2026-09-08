1. .NET 版本不对，UE5.3需要搭配.NET 6.0，在项目目录加入global.json配置版本
2. 有个Nuget配置文件引用到了已删除的D盘路径，需要删除对应行内容
3. VS编辑器的共享库，SDK的位置引用到已删除的D盘，卸载重装无效，需要改==注册表==
4. Rider直接打开.sln文件卡死，退出并删除.idea文件，然后用rider打开.uproject文件
5. 编写的ts文件没有生效，是在MainGame.ts文件底部没有添加新建的ts文件路径
6. 导入文件的路径要以，/Game开头，....的要以UE开头
7. 右键，左键一个GA蓝图文件，编辑器直接崩溃，暂时没有找到好的方法，就删了，重新建一个同名的，设置好
8. 新建的ts文件的路径上有爆红，是编辑器新建的内容没有按按钮刷新，导致代码中没有相关路径，对象的提示
9. 有个ts文件写好后，在编辑器里启动游戏直接就退，是蓝图的继承函数没连接
10. ts中，一个父类多个子类，都有mixin，那么不能用super.xxx 函数直接调用父类的函数(会有调用链的重复执行，崩溃)
	1. 原型链，蓝图节点的super，==ts中super禁用(父类ts没有使用相关函数时)==
	2. 场景中旧 Actor 实例的过期序列化数据。
	3. 判定是否存在后，把相关内容写在{}内
	4.  Puerts 的 blueprint.mixin 对 ReceiveTick 这类高频调用的BlueprintImplementableEvent 的支持不稳定。
	5. 在Base.ts中新建BaseInit（）函数，包装一层
	6. 蓝图里没有的东西(==调用到父函数==)，可能在TS中无法使用
11. UI控件的ts文件，也需要import在MainGame.ts文件中，不然无法执行相关逻辑
	1. TypeScript 编译器看到 UMG_EnemyBar 只被当作类型使用（as UMG_EnemyBar、: UMG_EnemyBar），就会把这个 import 视为"仅类型导入"。虽然编译后的 JS 里 require 还在，但 Puerts 在模块加载阶段可能有自己的解析逻辑，这条嵌套BP_Enemy.ts → BP_BaseCharacter.ts 链路上的 require 不一定会触发 UMG_EnemyBar 的模块执行。
	2. 放在 MainGame.ts 里直接 import，保证在游戏逻辑跑起来之前，所有需要的 mixin 都注册完了，之后创建的任何 Widget 实例就都带着 TS 扩展。这就是最终用户经验法则：Widget/UI 类必须在 MainGame.ts 中显式 import，不要依赖间接引入。
12. Play上挂了动画蓝图，Enmey没有挂，却在Base里使用了动画蓝图
13. TS类的变量，undefined
	1.  mixin 模式下，实例是由 UE 引擎创建的，不是 JS 的 new，所以：
	  - TS 类的构造函数不会被调用
	  - 字段初始化器（= false、= null）不会执行
	  - Die、ABP_Sinbi 在 ReceiveBeginPlay 之前都是 undefined
14. AI行为树没有启动
	1. 行为树下的任务文件在ts中，导入没有加入后缀_C（AssetPath里都是_C结尾，没有检查）
	2. 
15. 出现未定义的报错
	1. 可能是定义位置问题(ts,蓝图)
	2. begin初始化时的调用顺序不对
16. 没有打开编辑器就直接崩溃了
	""真正原因""：BP_GameMode--BP_Player--BP_PlayerController 内部出现问题，打开蓝图或者打开对应关卡就是会崩溃
	==解决措施==：先删地图(Level)，逐一排查GameMode相关文件。使用Git中的历史提交，直接回溯；或者删除重建；或者不要注入蓝图文件，改C++类
	 **可能的原因：**
    - **原因一：Main_Map.umap 中放置了损坏的资产或 Blueprint**  
        地图中某个 Actor/组件在 PostLoad()、BeginPlay() 或构造函数中引用了空指针，导致加载地图时直接崩溃。
        排查：可能是在地图保存过程中，突然关闭了进程
    - **原因二：Puerts (TypeScript) 脚本/生成代码未同步**  
        你的项目路径中包含 PuerTTS，且加载了 Puerts 插件。如果 TypeScript 没有编译、生成代码丢失或 C++ 导出的绑定与 TS 代码不匹配，在构造 CDO（类默认对象）时极易引发 CoreUObject 层的空指针崩溃。
        排查TS,直接在MainGame.ts中注释掉import代码
    - **原因三：C++ 代码（尤其是 GAS 相关的构造函数）存在空指针**  
        如果你的项目有自定义 C++ 代码（项目名 PTGAS 看起来像 Gameplay Ability System 项目），在某个类的构造函数或 PostInitProperties() 中直接使用了没有判空的指针（例如未成功获取到 Asset 或 Subsystem）。
    - **原因四：缓存 (Intermediate / Saved) 损坏**  
        CachedAssetRegistry 或 DDC (Derived Data Cache) 记录了旧的/错误的类信息。
		    删- Intermediate， Saved，DerivedDataCache，.vs，Binaries
		    **Generate Visual Studio project files**重新生成
		    打开.sln文件，生成
	- **原因五：回溯无效，文件的git lfs 保存错误**
		打开ue的资产文件都是乱码就是正确的，如果里面写着类似 version /https://git-lfs.github.com/spec/v1... 这样的一行文字，说明**你的 Git LFS 丢了文件！**
	- **原因六：代码书写书写错误，字母单词，参数，库名错误**
	- **原因七：整个插件的底层Bug，需要删除重写**
		- 这个是puerts的一个bug，在用ts接管蓝图的时候会出现问题，导致蓝图无法打开，最好的方法，直接删除报错的那个蓝图，重新写
	- **原因八：在C++的构造函数，蓝图的构造脚本中使用ts代码逻辑(序列化顺序)**
	- **原因九：在运行过游戏或者ts注入后，直接保存了蓝图内容在.uasset中，下次加载，还没有运行ts时，直接引用为空**
	- **原因十：蓝图创建的函数与TS所指向的函数不同，不使用蓝图创建的函数来重写**
		- UE 蓝图编译器（Kismet）重编译资产时，无法解析“蓝图字节码”与“Puerts Native 句柄”混合的畸形结构。
	- **原因十一：给蓝图资产（.uasset）做 mixin 会引发不可避免的物理崩溃**。==处理成功==
		- 重构架构，mixin->C++，蓝图再继承
17. Bar进度条控件，背景图与填充图颜色不一致
	1. 原因：外观-颜色不是纯白，背景图不受外观影响，而填充图会因外观的颜色叠加(包括父类)
	2. 解决：直接把外观弄成白色
18. 蓝图与TS拥有==同名变量==引起的接口错误的拓展蓝图类
	1. 
![](图片/UE/PuerTs/TS遇到的问题/IMG-20260825185209831.png)
19. 制作技能时，
	1. 情况1：运行时，第二次点击技能就崩溃了
	2. 情况2：运行时，点击技能1.5s后直接崩溃
	1.
```
 ** LoginId:5d19517c4f79783a713df8b392134cd2  	   
  EpicAccountId:d7721dce4cca47c69537bd4d4f0ab5ba                                                                                                                          
  Unhandled Exception: EXCEPTION_ACCESS_VIOLATION writing address 0x0000000000000001                                                                                      
  VCRUNTIME140                                    
  UnrealEditor_CoreUObject                        
  UnrealEditor_GameplayAbilities                   
  UnrealEditor_GameplayAbilities                   
  UnrealEditor_GameplayAbilities                          
  UnrealEditor_CoreUObject   
  UnrealEditor_JsEnv!puerts::FFunctionTranslator::FastCall() [K:\UE5.6.1_projects\PuerTTS-5.3\PTGAS\Plugins\Puerts\Source\JsEnv\Private\FunctionTranslator.cpp:388]            
  UnrealEditor_JsEnv!puerts::FFunctionTranslator::Call() [K:\UE5.6.1_projects\PuerTTS-5.3\PTGAS\Plugins\Puerts\Source\JsEnv\Private\FunctionTranslator.cpp:270]            
  UnrealEditor_JsEnv!puerts::FFunctionTranslator::Call() [K:\UE5.6.1_projects\PuerTTS-5.3\PTGAS\Plugins\Puerts\Source\JsEnv\Private\FunctionTranslator.cpp:226]             
  v8                                                
  v8                                               
  v8                                              
  v8                                           
  v8                           
  v8**
```

- 可能原因
	- 1. JS 侧持有了已销毁的 GAS 对象引用（最常见）
		- Ability 执行结束、Actor 被销毁、ASC 被移除时，底层 UObject 已经被 UE GC 回收或显式销毁，但 JS 侧的包装对象不会自动置空。****
	- 2. FastCall 路径下参数类型 / 数量不匹配
	- 3. GAS 对象本身的空指针(无效GAS对象)
	- 4. 一个 GameplayAbility 被激活后，必须调用 K2_EndAbility() 才会结束(无需担心K2_EndAbility函数的重复调用空指针，==情况1解决==)
	- 5. 删除蓝图文件，重建同名的不行？会造成文件损坏，并需要修改之前的绑定
	- 6. mixin到带参数的引擎回调函数，导致计算参数传递空指针。==其实是蓝图中，对应函数没有激活==(ts中谨慎填写带参数的引擎回调函数，使用蓝图，==情况2解决==）

20. 技能制作时，事件监听没有生效，注意把命名的等待事件激活ReadyForActivation()，还有标签一定要在DefaultGameplayTags.ini文件中对应好

21. 山崩地裂技能的第二次释放会出现bug,退出PIE会有bug
	1. **EXCEPTION_ACCESS_VIOLATION 内存访问违规（野指针 / 已经释放的内存又去读写）![](图片/UE/PuerTs/TS遇到的问题/IMG-20260907162215272.png)**
	2. 解决：将一些变量，在使用时，判断是否有效
	3. 解决2：double free 把添加for结果的数组，写在了for的里面，重复添加数组，销毁时，会重复销毁数组，访问到空指针![](图片/UE/PuerTs/TS遇到的问题/IMG-20260907202639631.png)