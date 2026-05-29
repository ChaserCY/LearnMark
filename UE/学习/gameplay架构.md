### 第一类：全局数据类（掌管整个游戏或关卡）

这类节点获取的是游戏最顶层的管理者。引擎只知道它们是管理者，不知道你具体写了什么规则，所以需要 Cast。

#### 1. Get Game Instance (获取游戏实例)

- **引擎返回类型：** GameInstance (父类)
    
- **你需要 Cast 成：** BP_MyGameInstance (你自定义的，存着全局变量的蓝图)
    
- **常用场景：**
    
    - **跨关卡传值：** 玩家从第一关走到第二关，血量和背包数据不能丢，存在这里。
        
    - **全局设置：** 比如音量大小、语言设置、分辨率，这些在游戏启动时就该加载。
        
    - 例子： Get Game Instance -> Cast to BP_MyGameInstance -> Get HighScore (读取最高分)。
        

#### 2. Get Game Mode (获取游戏模式)

- **引擎返回类型：** GameModeBase (父类)
    
- **你需要 Cast 成：** BP_MyGameMode (你写的规则蓝图)
    
- **常用场景：**
    
    - **游戏规则判定：** 比如“杀敌数达到100获胜”、“玩家复活时间是几秒”。
        
    - **重要提示：** GameMode **只存在于服务器端**。客户端调用这个节点会得到 Null（空），客户端要用下面的 GameState。
        
    - 例子： Get Game Mode -> Cast to BP_DeathMatchMode -> Respawn Player (调用你写的复活函数)。
        

#### 3. Get Game State (获取游戏状态)

- **引擎返回类型：** GameStateBase (父类)
    
- **你需要 Cast 成：** BP_MyGameState
    
- **常用场景：**
    
    - **同步的数据：** 所有玩家都能看到的数据，比如“剩余时间”、“当前比分”、“当前是第几波敌人”。
        
    - 例子： Get Game State -> Cast to BP_MyGameState -> Get TimeRemaining (获取倒计时显示在 UI 上)。

![](图片/UE/学习/gameplay架构/IMG-20260514172358027.png)

---

### 第二类：玩家与控制类（掌管当前的用户）

这是最常用的分类，用于 UI 蓝图、敌人 AI 蓝图与玩家交互。

#### 4. Get Player Character / Get Player Pawn (获取玩家角色)

- **引擎返回类型：** Character 或 Pawn (父类)
    
- **你需要 Cast 成：** BP_Player (你操作的那个具体主角)
    
- **常用场景：**
    
    - **敌人打你：** 敌人 AI 看到你 -> Get Player Character -> Cast to BP_Player -> Set HP = HP - 10。
        
    - **UI 显示血量：** 血条 UI -> Get Player Character -> Cast to BP_Player -> Get HP。
        

#### 5. Get Player Controller (获取玩家控制器)

- **引擎返回类型：** PlayerController (父类)
    
- **你需要 Cast 成：** BP_MyController (掌管输入和 UI 的大脑)
    
- **常用场景：**
    
    - **UI 交互：** 打开背包、呼出鼠标光标、按键检测。
        
    - **HUD 管理：** 很多时候我们会把创建 UI 的逻辑写在 Controller 里。
        
    - 例子： Get Player Controller -> Cast to BP_MyController -> Show Inventory (显示背包)。
        

#### 6. Get Player State (获取玩家状态)

- **引擎返回类型：** PlayerState (父类)
    
- **你需要 Cast 成：** BP_MyPlayerState
    
- **常用场景：**
    
    - **个人数据：** 比如“我的名字”、“我的击杀数”、“我的 Ping 值”。它和 Character 的区别是：Character 死了就销毁了，PlayerState 还在（只要你不退游戏）。
        
    - 例子： Get Player State -> Cast to BP_MyPlayerState -> Get KillCount。
        

#### 7. Get HUD (获取平视显示器)

- **引擎返回类型：** HUD (父类)
    
- **你需要 Cast 成：** BP_MyHUD
    
- **常用场景：**
    
    - 虽然现在大家多用 UMG（Widget），但 HUD 类依然常用于管理这些 Widget 的创建和销毁，或者做简单的射线检测、准星绘制。
        

---

### 第三类：组件与附属关系（掌管内部逻辑）

这类通常用于父子物体、组件与宿主之间的通信。

#### 8. Get Anim Instance (获取动画实例) —— 你提到过的

- **引擎返回类型：** AnimInstance (父类)
    
- **你需要 Cast 成：** ABP_MyCharacter (具体的动画蓝图)
    
- **来源：** 通常从 Mesh 组件引出。
    
- **常用场景：**
    
    - 角色蓝图告诉动画蓝图：“我开枪了” -> Cast to ABP -> Set IsShooting = True。
        

#### 9. Get Owner (获取拥有者)

- **引擎返回类型：** Actor (父类)
    
- **你需要 Cast 成：** 具体的 Actor (比如 BP_Rifle)
    
- **常用场景：**
    
    - **组件找爸爸：** 比如你写了一个“旋转组件”，你想知道你贴在谁身上。
        
    - **枪械找主人：** 枪是一个 Actor，被玩家拿在手里。枪的蓝图里调用 Get Owner -> Cast to BP_Player -> Add Ammo (给主人加子弹)。
        

#### 10. Get Controlled Pawn (获取被控制的 Pawn)

- **引擎返回类型：** Pawn (父类)
    
- **你需要 Cast 成：** BP_Enemy 或 BP_Player
    
- **来源：** 只能在 Controller 或 AIController 里使用。
    
- **常用场景：**
    
    - **AI 逻辑：** AI 控制器想让身体移动。Get Controlled Pawn -> Cast to BP_Zombie -> Attack (调用丧尸的攻击函数)。
        

---

### 总结：如何快速判断用哪个？

记住这个简单的对应关系：

1. **存活在整个游戏期间的数据** 
    
    ```
    →→
    ```
    
     Get Game Instance
    
2. **本局游戏的规则（服务器）** 
    
    ```
    →→
    ```
    
     Get Game Mode
    
3. **本局游戏的分数/时间（客户端）** 
    
    ```
    →→
    ```
    
     Get Game State
    
4. **找主角（改血量/位置）** 
    
    ```
    →→
    ```
    
     Get Player Character
    
5. **找操作者（鼠标/UI/按键）** 
    
    ```
    →→
    ```
    
     Get Player Controller
    
6. **找模型正在播放的动画** 
    
    ```
    →→
    ```
    
     Get Anim Instance