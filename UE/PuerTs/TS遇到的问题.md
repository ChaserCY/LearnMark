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

