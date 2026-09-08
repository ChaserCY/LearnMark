
# 纯TS
- 文件/路径相关模块
```ts
import * as fs from 'fs';// 文件系统模块，用于读写文件  
import * as path from 'path';// 路径处理模块，用于拼接、解析路径
```

- 路径拼接
```ts
path.join(__dirname, './Blueprints')   //得到string绝对路径
//__dirname,它表示当前正在执行的脚本文件所在的目录的绝对路径。  
//path.join() 路径字符串拼接

path.relative(__dirname, filePath)  .replace(/\\/g, '/')  .replace(/\.ts$/, '');
//获取两个绝对路径的相对路径，并且替换得到的字符串中的\，.ts，为/和空

```

- 文件操作
```ts
const entries = fs.readdirSync(dir, { withFileTypes: true });  //读指定绝对路径下的一层文件/文件夹
// readdirSync 同步读取目录内容,返回特殊结构体  
// withFileTypes: true 返回 Dirent 对象，可以区分文件和文件夹

entry.isDirectory()  //返回bool，是否是文件夹

entry.name.endsWith('.ts')  //返回bool，是否是.ts后缀名

const mainGameContent = fs.readFileSync(MainGameFile, 'utf8'); 
//读取文件文字内容，为一整个string字符串

mainGameContent.includes(importStatement) 
//返回bool，string中是否包含另一串string

fs.appendFileSync(MainGameFile, '\n' + appendLines.join('\n') + '\n');
//向某文件中，添加字符串文字，
//appendLines是string数组，每个元素之间添加\n分隔，合并成一个字符串
```

- 数组操作
```ts
results.push(...getAllTsFiles(fullPath));  //递归拆分添加元素，递归读取下一层文件/夹
//getAllTsFiles 返回的是数组，用...拆分成很多个元素添加到
```

- 打印日志consloe.log()
```ts
console.log(`\nDone: all imports already exist, nothing to add.`);
```

# 工具
- 函数参数可能为空
```ts
A:$Nullable<UE.PrimitiveComponent>       //表示A这个参数可能为空，可以传nullptr，也可以是空类型
```

- ts定时器/循环定时器
```ts
//定义定时器引用，如果里面！=null，说明定时器还在运行
_rotationIntervalId: ReturnType<typeof setInterval> | null = null;

// 用 setInterval 代替 Tick，setInterval是个函数
this._rotationIntervalId = setInterval(() => {  
    this.SetBarRotation();  //循环函数
}, 50);  
//50ms执行一次

//使用完毕，清理定时器
if(this._rotationIntervalId!==null){
clearInterval(this._rotationIntervalId);
this._rotationIntervalId=null;
}

//直接用蓝图中的定时器节点，创建和清除，这里的函数名的函数必须是蓝图或者C++的
const t = UE.KismetSystemLibrary.K2_SetTimer(this,"CheckCost",0.25,true);
UE.KismetSystemLibrary.K2_ClearTimerHandle(this,t);
UE.KismetSystemLibrary.K2_PauseTimerHandle(this,t);
UE.KismetSystemLibrary.K2_UnPauseTimerHandle(this,t);

//单次定时器
setTime(执行函数(可以Lambda)，时间(ms));

```

# 函数与方法
## 数组
```ts
this.HitActor.IsEmpty();
this.HitActor.Empty();

this.HitActor.Contains(OtherActor)

this.HitActor.RemoveAt(this.HitActor.FindIndex(OtherActor));
```

## 数学
- 随机数
```ts
UE.KismetMathLibrary.RandomInteger(3)
//0,1,2 中随机一个数

```

## 自带的函数
- BeginPlay
```ts
ReceiveBeginPlay(){  
   
 }
```
![](图片/UE/_/TS_API文档/IMG-20260627161555217.png)

- Character类中获取玩家摄像机参数
```ts
UE.GameplayStatics.GetPlayerCameraManager(this,0)
```

- 通过Class在世界中找一个类的实例
```ts
UE.GameplayStatics.GetActorOfClass(this,BP_PlayerClass)
```

## 常见的蓝图函数
```ts
<Actor>.K2_GetActorRotation()   //获取Actor旋转信息

K2_SetWorldRotation(UE.Rotator,false,null,false)
```

- 设置控件的可视化
```ts
this.CD.SetVisibility(UE.ESlateVisibility.Visible);
//this.CD代表控件引用
```
## 材质
- 动态材质与变量修改(运行中动态控制材质变化)
```ts
this.Image_Bar.GetDynamicMaterial().SetVectorParameterValue("Color",this.Color);
	1.this.Image_Bar - 这是一个 UMG Image 控件（UImage 类型）
	2. GetDynamicMaterial() - 获取该 Image 控件的动态材质实例
    - 动态材质允许在运行时修改参数，而不需要创建新的材质资产
    3. SetVectorParameterValue("Color", this.Color) - 设置材质中的向量参数
    - "Color" - 材质参数的名称（在材质编辑器中定义的参数名）
    - this.Color - 一个 FLinearColor 类型的值，表示要设置的颜色

this.Image_Bar.GetDynamicMaterial().SetScalarParameterValue("Pre",UE.KismetMathLibrary.FClamp(Progress,0,1));

Scalar = 1个数字（如：进度、强度、开关）
Vector = 多个数字（如：颜色RGBA、坐标XYZ）
```

## 特效
- Niagara 之前的旧粒子，**Cascade 粒子系统**(仅限 Cascade (旧粒子))
```ts
const HPRegenFX = UE.ParticleSystem.Load("/Game/Assets/Abilities/HealthRegen/P_HealthRegen.P_HealthRegen");         //加载类

UE.GameplayStatics.SpawnEmitterAtLocation(  
    this,  
    HPRegenFX,    //加载的粒子
    MyTarget.K2_GetActorLocation(),  
    MyTarget.K2_GetActorRotation(),  
    MyTarget.GetActorScale3D(),  
    true,             //自动销毁粒子？
    UE.EPSCPoolMethod.ManualRelease,  //使用粒子组件对象池（PSC Pool）模式，
    true              //自动开始播放粒子？
)                //生成特效

```
![](图片/UE/_/TS_API文档/IMG-20260808003654074.png)

## 生成物
```ts
this.LaserActor = UE.GameplayStatics.BeginDeferredActorSpawnFromClass(this,LaserActorClass,UE.Transform.Identity);  
UE.GameplayStatics.FinishSpawningActor(this.LaserActor,UE.Transform.Identity);

this.LaserActor.K2_AttachToComponent(  
    this.Character.LaserPoint,   
"",              //普通组件没有SocketName，有骨骼时才填  
    UE.EAttachmentRule.SnapToTarget,  
    UE.EAttachmentRule.SnapToTarget,  
    UE.EAttachmentRule.KeepRelative,  
    false   //是否需要物理焊接  
    )
```


# GAS 
## GameplayAbility
### 异步蒙太奇播放任务
```ts
**只能在 GameplayAbility 蓝图 / C++ 内使用**

动画和**技能生命周期绑定**，技能取消 / 结束自动停止动画；自带网络预测、多端回调，适合**技能主流程动画（攻击、施法、翻滚、换弹）**

//启动任务
this.PlayMeleeMontage();//是用函数，相当于启动任务


//播放普通攻击蒙太奇  
private PlayMeleeMontage(){  
    const StartSection = UE.KismetMathLibrary.RandomInteger(3).toString();  
    let MeleeMontageTask = UE.AbilityTask_PlayMontageAndWait.CreatePlayMontageAndWaitProxy(  
        this,  
        "",  
        MA_Melee,        
        1,  
        StartSection    
        );    
    MeleeMontageTask.OnCompleted.Add(()=>this.K2_EndAbility());//完整播放完毕  
    MeleeMontageTask.OnInterrupted.Add(()=>this.K2_EndAbility());//被其他动画打断  
    MeleeMontageTask.OnBlendOut.Add(()=>this.K2_EndAbility());//Blend out过渡完成  
    MeleeMontageTask.OnCancelled.Add(()=>this.K2_EndAbility());//任务被取消  
    //这上面四个回调都绑定了结束这个任务，无论哪种情况都会结束任务  
    MeleeMontageTask.ReadyForActivation();  //激活这个任务，异步任务启动开关  
    }
```

### 通过Tag发送事件和监听事件
```ts
//发送方，在Controller中
const GameplayEventData = new UE.GameplayEventData();  
GameplayEventData.EventTag = LaserEndTag;  
GameplayEventData.Instigator = this;  
GameplayEventData.Target = this;  
UE.AbilitySystemBlueprintLibrary.SendGameplayEventToActor(this.BP_Player, LaserEndTag, GameplayEventData);
//第一个参数必须是Actor，不能是Controller，标签类型是GameplayTag

//接收方
const GameplayEvent = UE.AbilityTask_WaitGameplayEvent.WaitGameplayEvent(this,LaserEndTag,null,true,true);  
GameplayEvent.EventReceived.Add((...args)=>this.EndMontage(...args));  
GameplayEvent.ReadyForActivation();
// 需要自定义一个回调函数，参数是UE.GameplayEventData

EndMontage(Payload: UE.GameplayEventData){  
    this.MontageJumpToSection("End");    //当前蒙太奇跳至片段
    this.K2_EndAbility();  
}


```

###  GA.ts中的固定的触发函数和结束回调函数（需要在蓝图激活）
```ts
//GA 触发时自动执行此函数内容
k2_ActivateAbility(){  
    
}

k2_OnEndAbility(){

}
```

### 移除GA
```ts
//获取GameplayTag
const BaseResponseTag = new UE.GameplayTag("Ability.BaseResponse");

//转换成AbilityTag
GetAbilityTag(AbilityTay:UE.GameplayTag):UE.GameplayTagContainer{  
    return UE.BlueprintGameplayTagLibrary.MakeGameplayTagContainerFromTag(AbilityTay as UE.GameplayTag);  
}

//直接在对应位置执行函数(通过标签移除技能)
this.AbilitySystemComponent.RemoveActiveEffectsWithTags(this.GetAbilityTag(BaseResponseTag));
```

### 其他Character.ts文件中获取自身属性值
```ts
UE.AbilitySystemBlueprintLibrary.GetFloatAttributeFromAbilitySystemComponent  
(  
    this.AbilitySystemComponent,  //ASC组件
    new UE.GameplayAttribute("HP","/Script/PTGAS.BaseAttributeSet:HP",null),           //属性位置
	this.bSuccess   //引用传递的布尔变量
);
//这一串返回float属性值
```

### GA中获取施法者对象引用
```ts
this.GetAvatarActorFromActorInfo()
//可以用as 转换类型，但需要判断是否有效
```

- 获取所拥有者
```ts
this.GetOwningActorFromActorInfo()
```

### 施加/移除GE伤害
```ts
//单个目标
this.BP_ApplyGameplayEffectToTarget(Payload.TargetData,LaserDamageClass);
//群体数组目标
this.BP_ApplyGameplayEffectToTarget(UE.AbilitySystemBlueprintLibrary.AbilityTargetDataFromActorArray(this.HitActors,true),BlastDamage);

/*通过标签Tag移除GE_Cost*/
this.BP_RemoveGameplayEffectFromOwnerWithAssetTags(this.Character.GetAbilityTag(LaserCostTag));


```

## 输入操作的绑定
- .h，类需要继承UExtensionMethods这个库，然后命名静态函数
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
#pragma once  
  
#include "CoreMinimal.h"  
#include "EnhancedInputComponent.h"  
#include "ExtensionMethods.h"  
#include "LF_ExtensionMethods.generated.h"  
  
//与下面函数一些参数有关  
/**  
 * @class UExtensionMethods  插件中编写的类  
 */UCLASS()  
class PTGAS_API ULF_ExtensionMethods : public UExtensionMethods  
{  
    GENERATED_BODY()  
    public:  
    /**  
     * 绑定输入动作  
     * @param  ImputComponent 输入组件  
     * @param InputAction 输入动作  
     * @param TriggerEvent 触发事件  
     * @param Object 绑定的对象  
     * @param FunctionName 绑定的函数  
     */    UFUNCTION(BlueprintCallable, Category = "LFExtensionMethods")  
    static void BindAction(UEnhancedInputComponent* InputComponent, const UInputAction* InputAction, const ETriggerEvent TriggerEvent, UObject* Object, const FName& FunctionName);  
          
};
```
- .cpp
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
  
#include "Libary/LF_ExtensionMethods.h"  
  
void ULF_ExtensionMethods::BindAction(UEnhancedInputComponent* InputComponent, const UInputAction* InputAction,  
    const ETriggerEvent TriggerEvent, UObject* Object, const FName& FunctionName)  
{  
    if (InputComponent)  
    {       InputComponent->BindAction(InputAction, TriggerEvent, Object, FunctionName);  
    }}
```
- 。ts
```ts
const TestAction = UE.InputAction.Load("/Game/BluePrints/Input/Action/IA_Test.IA_Test")  
const MeleeAction = UE.InputAction.Load("/Game/BluePrints/Input/Action/IA_Melee.IA_Melee")  
const HPRegenAction = UE.InputAction.Load("/Game/BluePrints/Input/Action/IA_HPRegen.IA_HPRegen")  
const DashAction = UE.InputAction.Load("/Game/BluePrints/Input/Action/IA_Dash.IA_Dash")  
const LaserAction = UE.InputAction.Load("/Game/BluePrints/Input/Action/IA_Laser.IA_Laser")  
const GroundBlastAction = UE.InputAction.Load("/Game/BluePrints/Input/Action/IA_GroundBlast.IA_GroundBlast")



const InputComponent = this.GetComponentByClass(UE.EnhancedInputComponent.StaticClass()) as UE.EnhancedInputComponent;  
if(InputComponent){  
    InputComponent.BindAction(TestAction,UE.ETriggerEvent.Started,this,"TestAction"); //这里的第一个函数必须是蓝图函数，或者C++蓝图可以调用的函数  
    InputComponent.BindAction(MeleeAction,UE.ETriggerEvent.Started, this,"Melee");  
    InputComponent.BindAction(HPRegenAction,UE.ETriggerEvent.Started, this,"HPRegen");  
    InputComponent.BindAction(DashAction,UE.ETriggerEvent.Started, this,"Dash");  
    InputComponent.BindAction(LaserAction,UE.ETriggerEvent.Started, this,"Laser");  
    InputComponent.BindAction(GroundBlastAction,UE.ETriggerEvent.Started, this,"GroundBlast");  
}
```
### 传参数的按键动作函数
```ts
Look(ActionValue: UE.InputActionValue) {  
    const Value2D = UE.EnhancedInputLibrary.Conv_InputActionValueToAxis2D(ActionValue);  
    this.AddControllerYawInput(Value2D.X);  
    this.AddControllerPitchInput(Value2D.Y);  
    }
```

## GAS内置类
### 目标选择设计基类AGameplayAbilityTargetActor
![](图片/UE/_/TS_API文档/IMG-20260907203921203.png)
- .h
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
#pragma once  
  
#include "CoreMinimal.h"  
#include "Abilities/GameplayAbilityTargetActor.h"  
#include "GroundSelectTargetActor.generated.h"  
  
/**  
 * 地面选择目标Actor  
 */UCLASS()  
class PTGAS_API AGroundSelectTargetActor : public AGameplayAbilityTargetActor  
{  
    GENERATED_BODY()  
public:  
    AGroundSelectTargetActor();  
    protected:  
    virtual void BeginPlay() override;  
    public:  
    virtual void Tick(float DeltaTime) override;  
    //启用目标选择  
    virtual void StartTargeting(UGameplayAbility* Ability) override;  
    //确认目标选择并发送数据  
    virtual void ConfirmTargetingAndContinue() override;  
    //获取玩家当前朝向的点，纯函数  
    UFUNCTION(BlueprintPure)  
    FVector GetPlayerLookAtPoint();  
    //SpawnActor时可以作为参数传入  
    UPROPERTY(EditAnywhere, BlueprintReadWrite,meta = (ExposeOnSpawn = true), Category="GroundSelect")  
    float SelectRadius = 10.f;  
};
```
- .cpp
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
  
#include "Abilitys/GroundSelectTargetActor.h"  
  
#include "Abilities/GameplayAbility.h"  
  
AGroundSelectTargetActor::AGroundSelectTargetActor()  
{  
}  
  
void AGroundSelectTargetActor::BeginPlay()  
{  
    Super::BeginPlay();  
}  
  
void AGroundSelectTargetActor::Tick(float DeltaTime)  
{  
    Super::Tick(DeltaTime);  
}  
  
void AGroundSelectTargetActor::StartTargeting(UGameplayAbility* Ability)  
{  
    Super::StartTargeting(Ability);  
    //基类成员变量，不会自动初始化，必须要在这个函数手动赋值  
    PrimaryPC = Cast<APlayerController>(Ability->GetOwningActorFromActorInfo()->GetInstigatorController());  
}  
  
void AGroundSelectTargetActor::ConfirmTargetingAndContinue()  
{  
    FVector LookPoint = GetPlayerLookAtPoint();  //获取玩家当前朝向的点  
    if (LookPoint.Size()!=0)  
    {       //重叠检测结果  
       TArray<FOverlapResult> OverlapResults;  
       //弱指针存储重叠的Actor为什么用弱指针？—目标选择是一个异步过程，从玩家确认到技能真正释放有时间差。如果敌人在这  
       //期间被别的技能杀死，裸指针会变成悬空指针。弱指针会自动感知对象销毁，安全处理。  
       TArray<TWeakObjectPtr<AActor>> OverlapActors;  
       FCollisionQueryParams CollisionParams;  
       CollisionParams.AddIgnoredActor(PrimaryPC->GetOwner());  
       //在看到的点进行球体重叠检测  
       bool IsHit = GetWorld()->OverlapMultiByChannel(  
       OverlapResults,       LookPoint,       FQuat::Identity,             //无旋转  
       ECC_Pawn,                   //检测Pawn  
       FCollisionShape::MakeSphere(SelectRadius), //球体半径  
       CollisionParams  
       );       //创建一个数据句柄，用来封装目标信息，GAS的“目标数据容器”，多态数据数组  
       FGameplayAbilityTargetDataHandle TargetDataHandle;  
       //创建一个目标位置信息，并且将它加入到目标数据句柄中  
       FGameplayAbilityTargetData_LocationInfo* CenterLocation = new FGameplayAbilityTargetData_LocationInfo();  
       CenterLocation->TargetLocation.LiteralTransform = FTransform(LookPoint); //目标位置  
       CenterLocation->TargetLocation.LocationType = EGameplayAbilityTargetingLocationType::LiteralTransform; //目标位置类型，字面值变换  
          TargetDataHandle.Add(CenterLocation); //0  
              //创建Actor数组数据  
       FGameplayAbilityTargetData_ActorArray* ActorArray = new FGameplayAbilityTargetData_ActorArray();  
       if (IsHit)  
       {          for (int i = 0;i < OverlapResults.Num();i++)  
          {             APawn* HitPawn = Cast<APawn>(OverlapResults[i].GetActor());  
             if (HitPawn && !OverlapActors.Contains(HitPawn))  
             {                OverlapActors.AddUnique(HitPawn);  
             }          }          if(!OverlapActors.IsEmpty())  
          {             ActorArray->SetActors(OverlapActors);  
             TargetDataHandle.Add(ActorArray); //1  
          }  
       }       //检查并且应用目标数据  
       check(ShouldProduceTargetData())  
       //如果目标数据有效，就广播目标数据  
       if (IsConfirmTargetingAllowed())  
       {          TargetDataReadyDelegate.Broadcast(TargetDataHandle);  
       }    }    return;  
}  
  
FVector AGroundSelectTargetActor::GetPlayerLookAtPoint()  
{  
    FVector ViewLocation;//玩家视口位置  
    FRotator ViewRotation;//玩家视口旋转  
    //获取玩家当前朝向的点  
    PrimaryPC->GetPlayerViewPoint(ViewLocation, ViewRotation);  
          
FHitResult HitResult;  
    FCollisionQueryParams CollisionParams;  
    CollisionParams.AddIgnoredActor(PrimaryPC->GetOwner());  
    //单条射线检测  
    GetWorld()->LineTraceSingleByChannel(  
       HitResult,       ViewLocation,       ViewLocation + ViewRotation.Vector() * 5000.f,   //沿摄像机朝向延伸5000单位  
       ECC_Visibility,  
       CollisionParams                           //附加参数，忽略玩家自己  
    );  
    //如果有阻碍  
    if (HitResult.bBlockingHit)  
    {       return HitResult.ImpactPoint;   //返回阻碍点  
    }  
    return FVector::Zero();  
}
```
![](图片/UE/_/TS_API文档/IMG-20260907220925302.png)