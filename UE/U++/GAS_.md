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
	- 右键public文件夹
	- 位置选择在这个public下
- 文件内容
	- 命名空间+静态函数
	- ![](图片/UE/U++/GAS_/IMG-20260414173301605.png)
- 其他cpp文件调用
	- 包含这个头文件（#include）
	- <命名空间>.<静态函数名>来调用
- 头文件#pragma
	- \#pragma once    同一编译单元即使被#include 多次，也只处理一次，避免重复定义，重复预处理
	- \#pragma region <片段名>------#pragma endregion  代码分组，可折叠，它是编译器，IDE指令


## 3.4 挂载摄像机组件

### 角色蓝图初始化
```cpp

```
### 头文件（角色蓝图）
- 相关组件前向声明
	- class USpringArmComponent;
	- class UCameraComponent;
- 反射命名组件指针
	- USpringArmComponen*
	- UCameraComponent*

### cpp文件（角色蓝图）
- 创建初始化组件
	- CameraBoom\===CreateDefaultSubobject==<USpringArmComponent(TEXT("CameraBoom"));
- 设置挂接点，长度，方向等等属性
	- CameraBoom = CreateDefaultSubobject\<USpringArmComponent>(TEXT("CameraBoom"));  
	- CameraBoom->SetupAttachment(GetRootComponent());  
	- CameraBoom->TargetArmLength = 200.0f;  
	- CameraBoom->SocketOffset = FVector(0.0f, 55.f, 65.f);  
	- CameraBoom->bUsePawnControlRotation = true;//是否跟随pawn的ControlRotation


## 3.5 Gameplay Input
![](../../Pasted%20image%2020260414211530.png)
### 构建模式中必须引入模块
![](图片/UE/U++/GAS_/IMG-20260613220238021.png)

### 创建的None类，没有UCLASS不会在编辑器中显示
- .h，命名了一个Namespace
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
#pragma once  
  
#include "NativeGameplayTags.h"  
  
namespace WarriorGameplayTags  
{  
    /** Input Tags**/  
    
    PERSON_GAS_API UE_DECLARE_GAMEPLAY_TAG_EXTERN(InputTag_Move)//声明外部可见的tag  
    PERSON_GAS_API UE_DECLARE_GAMEPLAY_TAG_EXTERN(InputTag_Look)  
          
      
}
```
 命名规则

  PERSON_GAS_API 中的 PERSON_GAS 来自模块名 Person_GAS，UE 的生成规则是：模块名大写 + _API。如果你的模块叫 MyModule，对应的宏就是 MY_MODULE_API。



- .cpp 
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
  
#include "WarriorGameplayTags.h"  
  
namespace WarriorGameplayTags  
{  
    /** Input Tags**/  
    UE_DEFINE_GAMEPLAY_TAG(InputTag_Move, "InputTag.Move")  
    UE_DEFINE_GAMEPLAY_TAG(InputTag_Look, "InputTag.Look")  
          
}

```

### 创建输入资产
![](图片/UE/U++/GAS_/IMG-20260614150444897.png)
创建对应的C++类

### 声明结构体，确定映射上下文，结构体数组
- .h
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
#pragma once  
  
#include "CoreMinimal.h"  
#include "GameplayTagContainer.h"  
#include "Engine/DataAsset.h"  
#include "DataAsset_InputConfig.generated.h"  
  
class UInputAction;//前置声明  
class UInputMappingContext;  
  
USTRUCT(BlueprintType)  
struct FWarriorInputActionConfig  
{  
    GENERATED_BODY()  
    public:  
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, meta=(Categories="InputTag"))  
    FGameplayTag InputTag;  
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)  
    UInputAction* InputAction;  
          
};  
  
  
/**  
 * */  
UCLASS()  
class PERSON_GAS_API UDataAsset_InputConfig : public UDataAsset  
{  
    GENERATED_BODY()  
    public:  
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)  
    UInputMappingContext* DefaultMap;  
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, meta=(TitleProperty="InputTag"))//meta 修改了编辑器中的显示文字  
    TArray<FWarriorInputActionConfig> NativeInputActions;  
    UInputAction* FindNativeInputActionByTag(const FGameplayTag& InputTag);  
};
```

- .cpp
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
  
#include "DataAssets/Input/DataAsset_InputConfig.h"  
  
  
//循环查找结构体数组，通过FGameplayTag  
UInputAction* UDataAsset_InputConfig::FindNativeInputActionByTag(const FGameplayTag& InputTag)  
{  
    for (const FWarriorInputActionConfig& InputActionConfig:NativeInputActions)  
    {       if (InputActionConfig.InputTag == InputTag && InputActionConfig.InputAction)//相等与有效性检查  
       {  
          return InputActionConfig.InputAction;  
                 }  
    }return nullptr;  
}
```

### 编辑器继承，手动添加
- 新建一个DataAsset继承上面自建的DataAsset_InputConfig，然后添加官方的Input内容
![](图片/UE/U++/GAS_/IMG-20260614154703632.png)

### 创建继承自UGameplayEnhanceComponent类
- ,h 文件
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
#pragma once  
  
#include "CoreMinimal.h"  
#include "EnhancedInputComponent.h"  
#include "DataAssets/Input/DataAsset_InputConfig.h"  
#include "WarriorInputComponent.generated.h"  
  
/**  
 * */  
UCLASS()  
class PERSON_GAS_API UWarriorInputComponent : public UEnhancedInputComponent  
{  
    GENERATED_BODY()  
public:  
    template<class UserObject, typename CallBackFunc>  
    void BindNativeInputAction(const UDataAsset_InputConfig* InInputConfig, const FGameplayTag& InInputTag, ETriggerEvent TriggerEvent, UserObject* ContextObject, CallBackFunc Func);  
    };  
  
  
template <class UserObject, typename CallBackFunc>  
void UWarriorInputComponent::BindNativeInputAction(const UDataAsset_InputConfig* InInputConfig,  
    const FGameplayTag& InInputTag, ETriggerEvent TriggerEvent, UserObject* ContextObject, CallBackFunc Func)  
{  
    //check(InInputConfig);  
    checkf(InInputConfig, TEXT("Input Config data asset is null,can not proceed with binding"));  
  
  
    if (UInputAction* FoundAction = InInputConfig->FindNativeInputActionByTag(InInputTag))  
    {       BindAction(FoundAction, TriggerEvent, ContextObject, Func);  
    }          
}
```
此处的模板函数==必须写在头文件中==，这是因为C++代码的编辑-链接规则决定的，因为这个函数的创建是给与外部调用的，编译时，若把实现写在.cpp中，但是本身的模板函数没有被实例化调用(若有调用，只会有对应特例的模板函数)，就不会编译(没有对应.obj内容)，自然会缺失，会导致==链接错误==
 链接错误 ≈ 编译器说："我知道你调用了这个函数，但所有 .obj 文件里都找不到它的实现，没法生成最终的程序。"


### 修改项目设置，Default Input Component Class
![](图片/UE/U++/GAS_/IMG-20260614164104744.png)

### 给WarriorHeroCharacter添加增强输入内容
- .h 文件新增
```cpp
protected:  
    //这个函数是 "给这个 Pawn/Character 绑定输入" 的地方。它在 APlayerController 成功 Possess（控制）了这个 Pawn/Character 时被调用，并且添加各自玩家的InputComponent  
    virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;//重写父类SetupPlayerInputComponent函数
    
#pragma region Input  
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "CharacterData", meta = (AllowPrivateAccess = "true"))  
    UDataAsset_InputConfig* InputConfigDataAssets;  
    //这个变量，需要在编辑器里赋值  
    void Input_Move(const FInputActionValue& InputActionValue);  
    void Input_Look(const FInputActionValue& InputActionValue);  
             
#pragma endregion
    
```

- .cpp 文件新增
```cpp
//Controller绑定成功时触发，重写系统自带函数，会自动传入编辑器绑定的UInputComponent
void AWarriorHeroCharacter::SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent)  
{  
    checkf(InputConfigDataAssets,TEXT("Forget to assign a vaild data asset as input config"))  
    //只获取当前对象Character的Controller(只能在APawn/ACharacter中)，静态函数获取的是某个游戏玩家的Controller  
    ULocalPlayer* LocalPlayer = GetController<APlayerController>()->GetLocalPlayer();  
    UEnhancedInputLocalPlayerSubsystem* Subsystem = ULocalPlayer::GetSubsystem<UEnhancedInputLocalPlayerSubsystem>(LocalPlayer);  
    check(Subsystem);  
        Subsystem->AddMappingContext(InputConfigDataAssets->DefaultMap,0);  
    UWarriorInputComponent* WarriorInputComponent = CastChecked<UWarriorInputComponent>(PlayerInputComponent);  
    
        WarriorInputComponent->BindNativeInputAction(InputConfigDataAssets, WarriorGameplayTags::InputTag_Move, ETriggerEvent::Triggered, this, &ThisClass::Input_Move);  
    WarriorInputComponent->BindNativeInputAction(InputConfigDataAssets, WarriorGameplayTags::InputTag_Look, ETriggerEvent::Triggered, this, &ThisClass::Input_Look);  
}

//移动的输入函数
void AWarriorHeroCharacter::Input_Move(const FInputActionValue& InputActionValue)  
{  
    const FVector2D MovementVector = InputActionValue.Get<FVector2D>();  
    //使人物跟随相机控制朝向  
    const FRotator MovementRotation(0.f, Controller->GetControlRotation().Yaw, 0.f);  
    if (MovementVector.Y != 0.f)  
    {       const FVector ForwardDirection = MovementRotation.RotateVector(FVector::ForwardVector);  
       AddMovementInput(ForwardDirection, MovementVector.Y);  
    }    if (MovementVector.X != 0.f)  
    {       const FVector RightDirection = MovementRotation.RotateVector(FVector::RightVector);  
       AddMovementInput(RightDirection, MovementVector.X);  
    }}  
  
  //视角控制的输入函数
void AWarriorHeroCharacter::Input_Look(const FInputActionValue& InputActionValue)  
{  
    //初次测试失灵，原因：IMC里面没有给IA_Look绑定鼠标输入  
    const FVector2D LookAxisVector = InputActionValue.Get<FVector2D>();  
    W_Debug::Prints("输入了Look------");  
    if (LookAxisVector.X != 0.f)  
    {       AddControllerYawInput(LookAxisVector.X);  
    }    if (LookAxisVector.Y != 0.f)  
    {       AddControllerPitchInput(LookAxisVector.Y);  
    }}

```

## 3.6 GameAnimation
- C++类的继承结构
![](图片/UE/U++/GAS_/IMG-20260617155256340.png)
### Character AnimInstance
![](图片/UE/U++/GAS_/IMG-20260617190856905.png)
- WarriotCharacterAnimInstance.h(1.所属角色，运动组件的引用，2.初始化，每帧函数)
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
#pragma once  
  
#include "CoreMinimal.h"  
#include "AnimInstances/WarriorBaseAnimInstance.h"  
#include "WarriorCharacterAnimInstance.generated.h"  
  
class AWarriorBaseCharacter;  
/**  
 * */  
UCLASS()  
class PERSON_GAS_API UWarriorCharacterAnimInstance : public UWarriorBaseAnimInstance  
{  
    GENERATED_BODY()   
      
public:  
    virtual void NativeInitializeAnimation() override;  
    //作用：动画实例初始化时调用，  
    // 触发时机：仅在动画实例首次初始化时调用一次（比如 Possess、BeginPlay 时）。  
    virtual void NativeThreadSafeUpdateAnimation(float DeltaSeconds) override;  
    //作用：每帧更新动画所需的数据，把逻辑计算的结果传递给动画蓝图  
    //触发时机：每一帧都会调用，由引擎的动画系统驱动。  
          
protected:  
      
    UPROPERTY()  
    ABaseCharacter* OwningCharacter;  
    UPROPERTY()  
    UCharacterMovementComponent* OwningMovementComponent;  
    //获取动画蓝图类所属的人物引用，以及人物的移动组件引用，方便后续获取人物的速度、加速度等数据。  
    UPROPERTY(VisibleDefaultsOnly, BlueprintReadOnly, Category = "AnimData|LocomotionData")  
    float GroundSpeed;  
    //获取水平速度大小平方和  
    UPROPERTY(VisibleDefaultsOnly, BlueprintReadOnly, Category = "AnimData|LocomotionData")  
    bool bHasAcceleration;  
    //判定是否具有加速度  
          
      
};
```
- WarriotCharacterAnimInstance.cpp
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
  
#include "AnimInstances/WarriorCharacterAnimInstance.h"  
  
#include "Character/BaseCharacter.h"  
#include "GameFramework/CharacterMovementComponent.h"  
  
  
  
void UWarriorCharacterAnimInstance::NativeInitializeAnimation()  
{  
    OwningCharacter = Cast<ABaseCharacter>(TryGetPawnOwner());  
    if (OwningCharacter)  
    {       OwningMovementComponent = OwningCharacter->GetCharacterMovement();  
    }}  
  
void UWarriorCharacterAnimInstance::NativeThreadSafeUpdateAnimation(float DeltaSeconds)  
{  
    if (!OwningCharacter || !OwningMovementComponent)  
    {       return;  
    }    GroundSpeed = OwningCharacter->GetVelocity().Size2D();  
    bHasAcceleration = OwningMovementComponent->GetCurrentAcceleration().SizeSquared2D()>0.f;  
}
```
- 编辑器中创建“动画蓝图”，选好骨骼，父类是WarriotCharacterAnimInstance--WarriorHeroAnimInstance

