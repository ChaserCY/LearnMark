# Unreal Engine C++ Doxygen 注释规范指南

本规范旨在为 UE 开发提供一套统一、易读且兼容 Doxygen 工具的注释标准。

## 1. 基础原则
- **头文件优先**：所有接口说明必须写在 `.h` 文件中。
- **宏关联**：注释块必须紧挨在 `UCLASS`, `USTRUCT`, `UFUNCTION`, `UPROPERTY` 等宏的上方。
- **简洁明了**：第一行 `@brief` 尽量在一句内交代核心功能。

---

## 2. 常用标签 (Tags)

| 标签           | 说明                       | 适用范围   |
| :----------- | :----------------------- | :----- |
| **@brief**   | 简要描述（必填）                 | 所有位置   |
| **@details** | 详细设计说明、算法解释（选填）          | 类、复杂函数 |
| **@param**   | 参数说明，格式：`@param Name 描述` | 函数     |
| **@return**  | 返回值说明                    | 函数     |
| **@note**    | 特别注意（如：仅在服务器运行，或性能开销大）   | 所有位置   |
| **@see**     | 参考其他类或函数                 | 所有位置   |
| **@todo**    | 待办事项                     | 所有位置   |

---

## 3. 结构化示例

### 3.1 文件头部 (File Header)
```cpp
/**
 * @file MyCharacter.h
 * @brief 处理玩家角色的核心逻辑，包括移动与战斗系统
 * @author YourName
 * @date 2023-10-27
 */
```
 
### 3.2 类与结构体 (UCLASS / USTRUCT)

```
/**
 * @class AMyProjectCharacter
 * @brief 游戏主角色类
 * @details 继承自 ACharacter，集成了库存系统和技能状态机
 */
UCLASS(config=Game)
class AMyProjectCharacter : public ACharacter
{ ... };
```

### 3.3 属性 (UPROPERTY)

建议在描述中注明单位。
```
/** @brief 角色的当前生命值。取值范围 [0, MaxHealth] */
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Stats")
float Health;

/** @brief 角色移动速度（单位：cm/s） */
UPROPERTY(EditAnywhere, Category = "Movement")
float WalkSpeed;
```

### 3.4 函数 (UFUNCTION)

包含参数、返回值及蓝图说明。

```
/**
 * @brief 执行攻击动作
 * @details 根据当前装备的武器触发不同的蒙太奇动画
 * 
 * @param WeaponType 攻击所使用的武器类型
 * @param bIsCrit 是否为暴击（由随机算法预先计算）
 * @return bool 攻击指令是否成功发出
 * 
 * @note 该函数仅在服务器执行 (Server Only)
 * @see EWeaponType
 */
UFUNCTION(BlueprintCallable, Category = "Combat")
bool ExecuteAttack(EWeaponType WeaponType, bool bIsCrit);
```

### 3.5 委托/分发器 (Delegates)

```
/**
 * @brief 当角色死亡时触发的签名
 * @param Killer 击杀该角色的 AController 指针
 */
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnCharacterDeath, AController*, Killer);
```

---
## 4. 针对 UE 的特别建议

1. **RPC 说明**：对于 Server/Client/NetMulticast 函数，请在注释中明确注明网络所有权要求。
    
2. **异步处理**：如果是 Latent 函数（异步蓝图节点），请说明完成后的回调行为。
    
3. **性能警告**：在 Tick 函数或高频调用的函数中使用 @warning 提示性能消耗。