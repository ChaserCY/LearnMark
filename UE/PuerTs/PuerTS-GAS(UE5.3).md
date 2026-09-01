# 构建环境
- 创建项目，调整编辑器设置
- 创建Plugins，拖入整个PuerTS插件包，导入Assets资源包
- 拖入package.json，tsconfig.json 文件到项目中，添加global.json文件，限制.NET 的版本为6.0
- ![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260528202217245.png)
- ![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260528202217303.png)- ![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260528202217355.png)
- 删除Save，.sln，Intermediate，Binaries文件，还有插件的\Plugins\Puerts\Binaries，\Plugins\Puerts\Intermediate文件，然后右键.uproject重构(==插件内部也需要删文件==)
- 打开.sln文件，然后在.Build.cs文件中加入插件名“EnhancedInput”，“Gameplay Ability”，"GameplayTags",  "GameplayTasks"，“Puerts”，“JsEnv”
- 在编辑器-插件，启用GAS ,GASAttachEditor插件，编辑器偏好-源代码-启用Rider Uproject打开


# 1.项目初始化
## 开启插件调试窗口
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260616214238879.png)
cmd控制台输入：GASAttachEditorShow 可以打开窗口

## 新建C++类继承Gamemode
==一定要在项目设置中添加游戏实例==
### GAS_GameInstance.h
```cpp
#pragma once  
  
#include "CoreMinimal.h"  
#include "JsEnv.h"  
#include "Engine/GameInstance.h"  
#include "GAS_GameInstance.generated.h"  
  
/**  
 *  GAS 游戏实例  
 */UCLASS()  
class PTGAS_API UGAS_GameInstance : public UGameInstance  
{  
    GENERATED_BODY()  
    public:  
    //初始化  
    virtual void Init() override;  
    //启动  
    virtual void OnStart() override;  
    //退出  
    virtual void Shutdown() override;  
          
protected:  
    //是否为调试模式  
    UPROPERTY(EditDefaultsOnly, BlueprintReadWrite,Category="Debug")  
    uint8 bDebugMode : 1;  
    //是否等待调试  
    UPROPERTY(EditDefaultsOnly, BlueprintReadWrite,Category="Debug")  
    uint8 bWaitForDebugger : 1;  
    //调用ts函数的代理  
    DECLARE_DYNAMIC_DELEGATE_TwoParams(FCall, FString, FunctionName, UObject*, Uobject);  
    UPROPERTY()  
    FCall FCall;  
    //此处也必须与MainGame.ts中绑定FCall同名  
          
//调用ts函数  
    UFUNCTION(BlueprintCallable)  
    void CallTS(FString FunctionName, UObject* Uobject);  
    private:  
    //脚本环境  
    TSharedPtr<puerts::FJsEnv> GameScript;   
      
};  
  
//写完后，右键--生成代码--生成定义
```

### GAS_GameInstance.cpp
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
#include "GameInstance/GAS_GameInstance.h"  
  
void UGAS_GameInstance::Init()  
{  
    Super::Init();  
    if (bDebugMode)  
    {       GameScript = MakeShared<puerts::FJsEnv>(  
       std::make_unique<puerts::DefaultJSModuleLoader>(TEXT("JavaScript")),//这里表示TS文件编译后的JS代码都放在JavaSript文件下  
       std::make_shared<puerts::FDefaultLogger>(),  
       8080  
       );  
       if (bWaitForDebugger)  
       {          GameScript->WaitDebugger();  
       }    }    else  
    {  
       GameScript = MakeShared<puerts::FJsEnv>();  
    }    //这里这个TEXT("GameInstance")的GameInstance必须和MainGame.ts中传递的参数名一致，否则在MainGame.ts中无法通过puerts.getGlobal("GameInstance")获取到这个实例  
    TArray<TPair<FString, UObject*>> Arguments;  
    Arguments.Add({TEXT("GameInstance"),this});  
    //启动脚本，指定入口模块为“MainGame”，并传递参数  
    GameScript->Start(TEXT("MainGame"),Arguments);  
    //这里的MainGame必须是TypeScript/MainGame.ts文件的名字，否则会找不到入口模块，导致脚本无法启动  
}  
  
void UGAS_GameInstance::OnStart()  
{  
    Super::OnStart();  
}  
  
void UGAS_GameInstance::Shutdown()  
{  
    Super::Shutdown();  
    GameScript.Reset();  
}  
  
  
void UGAS_GameInstance::CallTS(FString FunctionName, UObject* Uobject)  
{  
    FCall.ExecuteIfBound(FunctionName, Uobject);  
}
```

## 在插件中，运行命令
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260528202217411.png)  

在项目目录里cmd命令，添加对应npm包
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260528202217480.png)
npm install --save-dev typescript   //不同版本，可能会报错，不然无法运行npm start
 npm install --save-dev typescript@4.7.4
## 添加到解决方案条目
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260528202217518.png)

## 导入内容
TypeScript
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260528202217561.png)
三个文件各自的作用
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260528202217625.png)

![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260528202217676.png)
这个位置的名字，必须要和创建的C++类GameInstance
## 编辑同步
在编辑器中，对蓝图进行了修改，就点击上方插件的按钮还有下方的编译
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260528202217732.png)
或者使用引擎内的cmd窗口输入：
==Puerts.Gen==

C++类的代码进行了修改就需要点击右下角的
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260528202217792.png)

每次修改完TS文件代码，需要运行测试时，终端运行npm start编译TS->JS
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260528202217833.png)

## 编写规则

 这个项目中所有的=="硬绑定"==（不遵守就会报错/不运行）

   【硬绑定 1】入口文件:TypeScript/MainGame.ts→Content/JavaScript/MainGame.js

  C++ 代码 (GAS_GameInstance.cpp:35)
    → GameScript->Start(TEXT(=="MainGame"==), Arguments);

  Ts 编译器 (tsconfig.json)
  TypeScript/==MainGame.ts==  ──编译──→  Content/JavaScript/MainGame.js

  - 文件名必须是 MainGame.ts / MainGame.js，不能改
  - ==文件路径必须是 TypeScript/MainGame.ts，不能改==
  - 这是最硬的绑定，一动就不运行

  【硬绑定 2】参数名：argv.getByName(=="GameInstance"==)

  C++ 传参 (GAS_GameInstance.cpp:32)
    Arguments.Add({TEXT(=="GameInstance"==), this});

  TS 收参 (MainGame.ts:18)
    const ==GameInstance== = argv.getByName("GameInstance") as UE.GAS_GameInstance;

  - 字符串 "GameInstance" 必须两边一致
  - 如果你改 C++ 的 TEXT("GameInstance") 就必须同步改 TS，反之亦然

  【硬绑定 3】TS 源文件目录：TypeScript/

  tsconfig.json: "include": \["TypeScript/\*\*/\*", ...]

  - ==所有 .ts 源文件必须放在 TypeScript/ 目录下==
  - 目录名固定，不能改

  【硬绑定 4】编译输出目录：Content/==JavaScript==/

  tsconfig.json: "outDir": "Content/JavaScript"
  C++ (GAS_GameInstance.cpp:14):
    MakeShared<puerts::FJsEnv>(
      make_unique<puerts::DefaultJSModuleLoader>(TEXT(=="JavaScript"==)), ...
    )

  - C++ 的模块加载器硬编码了 "JavaScript" 字符串
  - TS 编译器输出到 Content/JavaScript/
  - 两边的"JavaScript"必须一致
  - 运行时会去 Content/JavaScript/ 目录下找 .js 文件

  【硬绑定 5】类型声明目录：Typing/

  tsconfig.json: "typeRoots": \["Typing", "./node_modules/@types"]

  - 这些文件是 Puerts 自动生成的，你手动去写/改就会出问题
  - Typing/ 目录名固定

  【可选的】FileWatcher 里的 Blueprints/ 目录

  FileWatcher.ts: const BlueprintsDir = path.join(__dirname, './Blueprints');

  - 如果你不使用 FileWatcher（它只是个开发辅助工具），这个目录完全不用管
  - 如果你要用它：需要在 TypeScript/Blueprints/ 下放文件
  - 它会在文件新加时自动往 MainGame.ts 里写 import 语句
  - 注意：当前 TypeScript/ 下并没有 Blueprints/ 目录，说明这个工具目前没有被用起来

  ---
 
 总结：你要遵守的最少规则

  TypeScript/
  ├── MainGame.ts     ← 文件名固定（入口）
  ├── 你的其他文件.ts  ← 可以随便加，想叫什么叫什么
  └── (可选) Blueprints/
      └── ...

  Content/JavaScript/
  ├── MainGame.js     ← 编译产物，别手动改
  └── ...

  Typing/             ← Puerts 自动生成的，别碰

### 总结：
  1. MainGame.ts 这个文件名是死的，不能改
  2. 新文件 都放在 TypeScript/ 下，通过 import 引入到 MainGame.ts（或者被其他模块 import 即可）
  3. 不需要自己创建额外目录
  4. 不要改 Typing/、Content/JavaScript/ 里的东西
  5. 

  新加功能的方式： 你就直接在 TypeScript/ 下新建 .ts 文件，然后在 MainGame.ts 里加一行 import 就行了。其他的 Puerts 自动搞定。

补充：
MainGame.ts
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260528202217876.png)
GAS.GameInstance.
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260528202217927.png)


## 添加Debug
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260528202217966.png)
项目设置--插件设置--PuerTS
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260528202218006.png)

## 蓝图Actor测试
### 添加一个Actor蓝图，然后添加一个同名的.ts文件在TypeScript/Blueprints目录下
注意第2,4行的内容，必须正确。
```ts
import * as UE from "ue";  
import mixin from "../../mixin";//额外添加这一行使得10行不会报错@mixin(AssetPath)  
  
const AssetPath = "/Game/BluePrints/Test/BP_Test.BP_Test_C";//这里的路径必须和Content中的蓝图ACtor路径对应，并且.后面必须是BP_Test_C加个_C   
export interface BP_Test extends UE.Game.BluePrints.Test.BP_Test.BP_Test_C{  
    }  
  
@mixin(AssetPath)  
export class BP_Test implements BP_Test {  
    Fun1(){  
        UE.KismetSystemLibrary.PrintString(  
            this,  
            "我是函数1",  
            true,  
            true,  
            UE.LinearColor.Red,  
            2)  
    }      
Fun2(){  
        UE.KismetSystemLibrary.PrintString(  
            this,  
            "我是函数2",  
            true,  
            true,  
            UE.LinearColor.Green,  
            3)  
    }    Fun3(){  
        UE.KismetSystemLibrary.PrintString(  
            this,  
            "我是函数3，通过蓝图调用的",  
            true,  
            true,  
            UE.LinearColor.Green,  
            2  
        )  
            }  
    }
```
### 重写函数(Fun1,Fun2)与新增函数(Fun3)
Fun1,Fun2 是原蓝图中的自定义事件
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260528202218049.png)
Fun3是TS文件中命名的事件(需要使用时调用)
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260528202218087.png)

![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260528202218127.png)

# 2.基础角色与相机控制
## 1. 创建基础角色蓝图，创建子类(Player,Enemy)，创建输入动作，输入操作映射
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260529150604003.png)
另外还需要创建蓝图==玩家控制器==
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260529151011246.png)
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260529151130038.png)


## 2. 在TypeScript中创建对应结构，还有对应蓝图的ts文件
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260529150710449.png)


## 3. 在Player蓝图中，命名变量和函数，ts中准备进行重写
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260529150818776.png)

## 4. ts文件中，给Player添加操作映射，写操作函数
```ts
import mixin from "../../../mixin";  
  
import {BP_BaseCharacter} from "../BP_BaseCharacter";  
import * as UE from "ue";  
//继承自父类蓝图，导入父类ts  
  
  
const AssetPath = "/Game/BluePrints/Character/Player/BP_Player.BP_Player_C";  
  
//const IMC_Default = UE.Object.Load("/Game/BluePrints/Input/IMC_Default.IMC_Default") as UE.InputMappingContext;  
const IMC_Default = UE.InputMappingContext.Load("/Game/BluePrints/Input/IMC_Default.IMC_Default");  
//加载资源，注意类型  
  
export interface BP_Player extends UE.Game.BluePrints.Character.Player.BP_Player.BP_Player_C {  
  
}  
  
@mixin(AssetPath)  
export class  BP_Player extends BP_BaseCharacter implements BP_Player {  
    //相机开始位置  
    CameraStartLocation = new UE.Vector;  
    //相机结束位置  
    CameraEndLocation = new UE.Vector(0, 0, 180);  
    //相机开始的旋转  
    CameraStartRotation = new UE.Rotator;  
    //相机结束的旋转  
    CameraEndRotation = new UE.Rotator(-17, 0, 0);  
          
ReceiveBeginPlay(){  
        super.ReceiveBeginPlay();  
        this.BP_PlayerController = UE.GameplayStatics.GetPlayerController(this,0) as UE.Game.BluePrints.Character.Player.BP_PlayerController.BP_PlayerController_C;  
        this.AddMappingContext();  
        this.LookCameraLine.SetPlayRate(1/0.3);  
    }    //添加输入映射  
    AddMappingContext(){  
        if(this.BP_PlayerController){  
            let EnhanceInputSubsystem = UE.SubsystemBlueprintLibrary.GetLocalPlayerSubSystemFromPlayerController(  
                this.BP_PlayerController,  
                UE.EnhancedInputLocalPlayerSubsystem.StaticClass()  
            )as UE.EnhancedInputLocalPlayerSubsystem;  
            //as 可以强制告诉let声明变量的类型  
            if(EnhanceInputSubsystem&&IMC_Default){  
                EnhanceInputSubsystem.AddMappingContext(IMC_Default,0);  
            }            //限制相机控制的俯仰角度  
            UE.GameplayStatics.GetPlayerCameraManager(this,0).ViewPitchMin = -65;  
            UE.GameplayStatics.GetPlayerCameraManager(this,0).ViewPitchMax = 25;  
                    }  
            }  
    //鼠标移动视角  
    Look(ActionValue: UE.Vector2D) {  
        this.AddControllerYawInput(ActionValue.X);  
        this.AddControllerPitchInput(ActionValue.Y);  
            }  
  
    //移动  
    Move(ActionValue: UE.Vector2D) {  
        //前进  
        const ForwardVector = UE.KismetMathLibrary.GetForwardVector(new UE.Rotator(0,this.GetControlRotation().Yaw,0));  
        const RightVector = UE.KismetMathLibrary.GetRightVector(new UE.Rotator(0,this.GetControlRotation().Yaw,0));  
        this.AddMovementInput(ForwardVector,ActionValue.Y);  
        this.AddMovementInput(RightVector,ActionValue.X);  
            }  
    //锁定还需要改类图视图中组件的设置(代码实现)  
    //锁定镜头  
    LookCamera(OpenLook: boolean) {  
        this.bUseControllerRotationYaw = OpenLook;  
        this.SpringArm.bUsePawnControlRotation = !OpenLook;  
        this.CharacterMovement.bOrientRotationToMovement = !OpenLook;  
        if(OpenLook){  
            this.CameraStartLocation = this.Camera.RelativeLocation;  
            this.CameraStartRotation = this.Camera.RelativeRotation;  
            this.LookCameraLine.PlayFromStart();  
                    }  
        else {  
            this.CameraStartLocation = UE.Vector.ZeroVector;  
            this.CameraStartRotation = UE.Rotator.ZeroRotator;  
            this.LookCameraLine.ReverseFromEnd();  
                    }  
            }  
    //镜头缓动,千万不要打错字  
    LookCameraLine__UpdateFunc(){  
        const NewLocation = UE.KismetMathLibrary.VLerp(this.CameraStartLocation, new UE.Vector(0,0,180),this.LookCameraLine_Time_E2604BD340D43B16DB00B9849B380DCE);  
    const NewRotation = UE.KismetMathLibrary.RLerp(this.CameraStartRotation, new UE.Rotator(-17,0,0), this.LookCameraLine_Time_E2604BD340D43B16DB00B9849B380DCE, true);  
        this.Camera.K2_SetRelativeLocationAndRotation(NewLocation, NewRotation, false, null, false);  
          
      
    }  
          
      
}
```

## 5. 细节设置
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260529151457883.png)
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260529151527978.png)

## 6. 限制相机角度
UE.GameplayStatics.GetPlayerCameraManager(this,0).ViewPitchMin = -65;  
UE.GameplayStatics.GetPlayerCameraManager(this,0).ViewPitchMax = 25;

## 7. 其他
总结就是：@mixin 类中，不要依赖 field = value 这种初始化语法。这是因为 Puerts 的 @mixin 装饰器在混入类时，ES6 类字段初始化语法（= 赋值）的执行时机和标准 JS 不同——mixin
  过程中这些初始化的赋值操作可能被跳过，字段没有被真正挂到 UE 对象上。
  应该要么：

  1. 纯声明 + 懒初始化（当前方案）：
  Move_Rotator: UE.Rotator;
  // 在使用处：
  if (!this.Move_Rotator) this.Move_Rotator = new UE.Rotator(0,0,0);

  2. 在 ReceiveBeginPlay 中初始化（对 Move_Rotator 也可行）：
  Move_Rotator: UE.Rotator;

  ReceiveBeginPlay() {
      this.Move_Rotator = new UE.Rotator(0, 0, 0);
      // ...
  }

  当前懒初始化的方式最安全，因为它一定在 UE 完全就绪后才创建对象。



# 3.基础的动画以及基础属性添加
- 创建对应的动画蓝图并设置
- 创建基础属性集
- Character基础属性监听
## 1.新建BaseAttributeSet类，通过在Rider中直接创建.h,.cpp文件
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260608171303619.png)
- BaseAttributeSet.h,声明一些基础属性值
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
#pragma once  
  
#include "CoreMinimal.h"  
#include "AttributeSet.h"  
#include "AbilitySystemComponent.h"  
#include "BaseAttributeSet.generated.h"  
  
//标准属性宏,这些宏可以提供一些常用的函数  
#define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \  
    GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \  
    GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \  
    GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \  
    GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)  
  
  
/**  
 * 基础属性集  
 */UCLASS()  
class PTGAS_API UBaseAttributeSet : public UAttributeSet  
{  
    GENERATED_BODY()  
    public:  
    //生命值  
    UPROPERTY(BlueprintReadOnly, Category="AttributeSet")  
    FGameplayAttributeData HP;  
    ATTRIBUTE_ACCESSORS(UBaseAttributeSet, HP)  
    //这里提供了HP的Get,Set,Init的函数  
    //最大生命值  
    UPROPERTY(BlueprintReadOnly, Category="AttributeSet")  
    FGameplayAttributeData MaxHP;  
    ATTRIBUTE_ACCESSORS(UBaseAttributeSet, MaxHP)  
    //魔法值  
    UPROPERTY(BlueprintReadOnly, Category="AttributeSet")  
    FGameplayAttributeData MP;  
    ATTRIBUTE_ACCESSORS(UBaseAttributeSet, MP)  
    //最大魔法值  
    UPROPERTY(BlueprintReadOnly, Category="AttributeSet")  
    FGameplayAttributeData MaxMP;  
    ATTRIBUTE_ACCESSORS(UBaseAttributeSet, MaxMP)  
    //能量  
    UPROPERTY(BlueprintReadOnly, Category="AttributeSet")  
    FGameplayAttributeData SP;  
    ATTRIBUTE_ACCESSORS(UBaseAttributeSet, SP)  
  
    //最大能量  
    UPROPERTY(BlueprintReadOnly, Category="AttributeSet")  
    FGameplayAttributeData MaxSP;  
    ATTRIBUTE_ACCESSORS(UBaseAttributeSet, MaxSP)  
    virtual void PostGameplayEffectExecute(const struct FGameplayEffectModCallbackData& Data) override;  
    //GAS框架提供的虚函数重写，GameplayEffect修改后自动调用  
    };
```
- .cpp 每次GameplayEffect变化时，进行值的限制
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
#include "Abilitys/BaseAttributeSet.h"  
  
#include "GameplayEffectExtension.h"  
  
void UBaseAttributeSet::PostGameplayEffectExecute(const struct FGameplayEffectModCallbackData& Data)  
{  
    Super::PostGameplayEffectExecute(Data);  
    if (Data.EvaluatedData.Attribute.GetUProperty()!=nullptr)  
    {       //获取修改的数据属性名  
       FString AttributeName = Data.EvaluatedData.Attribute.GetUProperty()->GetName();  
       //血量限制  
       if (AttributeName == "HP")  
       {          SetHP(FMath::Clamp(GetHP(), 0.f, GetMaxHP()));  
       }       //蓝量设置  
       if (AttributeName == "MP")  
       {          SetMP(FMath::Clamp(GetMP(),0.f, GetMaxMP()));  
       }       //限制能量  
       if (AttributeName == "SP")  
       {          SetSP(FMath::Clamp(GetSP(), 0.f, GetMaxSP()));  
       }           }  
}
```

## 2. 在BaseCharacter中监听==属性值==的变化
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
#pragma once  
  
#include "CoreMinimal.h"  
#include "Abilitys/BaseAttributeSet.h"  
#include "GameFramework/Character.h"  
#include "BaseCharacter.generated.h"  
  
struct FOnAttributeChangeData;  
class UAbilitySystemComponent;  
  
//监听属性变化的代理  
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnAttributeChanged, float, Value);  
  
/*  
 * 基础角色  
 */  
  
UCLASS()  
class PTGAS_API ABaseCharacter : public ACharacter  
{  
    GENERATED_BODY()  
  
public:  
    ABaseCharacter();  
  
protected:  
    virtual void BeginPlay() override;  
  
public:   
    virtual void Tick(float DeltaTime) override;  
    virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;  
          
//技能系统组件(核心组件)  
protected:  
    UPROPERTY(EditDefaultsOnly,BlueprintReadOnly, Category="AbilitySystem" )  
    TObjectPtr<UAbilitySystemComponent>  AbilitySystemComponent;  
    //监听血量变化  
    UPROPERTY(BlueprintAssignable, Category="AbbilitySystem")  
    FOnAttributeChanged HPChanged;  
    void OnHPAttributeChanged(const FOnAttributeChangeData& Data);  
    //监听血量变化  
    UPROPERTY(BlueprintAssignable, Category="AbbilitySystem")  
    FOnAttributeChanged MPChanged;  
    void OnMPAttributeChanged(const FOnAttributeChangeData& Data);  
    //监听血量变化  
    UPROPERTY(BlueprintAssignable, Category="AbbilitySystem")  
    FOnAttributeChanged SPChanged;  
    void OnSPAttributeChanged(const FOnAttributeChangeData& Data);  
    };
```
- .cpp 
```cpp
  
#include "Character/BaseCharacter.h"  
#include "Character/BaseCharacter.h"  
  
#include "AbilitySystemComponent.h"  
  
  
ABaseCharacter::ABaseCharacter()  
{  
    PrimaryActorTick.bCanEverTick = true;  
    AbilitySystemComponent = CreateDefaultSubobject<UAbilitySystemComponent>(TEXT("AbilitySystemComponent"));  
  
}  
  
void ABaseCharacter::BeginPlay()  
{  
    Super::BeginPlay();  
    }  
  
void ABaseCharacter::Tick(float DeltaTime)  
{  
    Super::Tick(DeltaTime);  
  
}  
  
void ABaseCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)  
{  
    Super::SetupPlayerInputComponent(PlayerInputComponent);  
    if (AbilitySystemComponent)  
    {       //监听属性变化  
       AbilitySystemComponent->GetGameplayAttributeValueChangeDelegate(UBaseAttributeSet::GetHPAttribute()).AddUObject(this, &ABaseCharacter::OnHPAttributeChanged);  
       AbilitySystemComponent->GetGameplayAttributeValueChangeDelegate(UBaseAttributeSet::GetMPAttribute()).AddUObject(this, &ABaseCharacter::OnMPAttributeChanged);  
       AbilitySystemComponent->GetGameplayAttributeValueChangeDelegate(UBaseAttributeSet::GetSPAttribute()).AddUObject(this, &ABaseCharacter::OnSPAttributeChanged);  
    }  
}  
  
void ABaseCharacter::OnHPAttributeChanged(const FOnAttributeChangeData& Data)  
{  
    //属性值发生变化  
    if (Data.OldValue!=Data.NewValue)  
    {       HPChanged.Broadcast(Data.NewValue);  
    }    }  
  
void ABaseCharacter::OnMPAttributeChanged(const FOnAttributeChangeData& Data)  
{  
    //属性值发生变化  
    if (Data.OldValue!=Data.NewValue)  
    {       MPChanged.Broadcast(Data.NewValue);  
    }          
}  
  
void ABaseCharacter::OnSPAttributeChanged(const FOnAttributeChangeData& Data)  
{  
    //属性值发生变化  
    if (Data.OldValue!=Data.NewValue)  
    {       SPChanged.Broadcast(Data.NewValue);  
    }}
```

## 3. 在编辑器中绑定与调用测试
- 创建数据表格
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260608170813175.png)

- 添加对应属性的信息
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260608170859908.png)

- 绑定属性集的类与属性的数据表格
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260608170948294.png)

- 测试：从组件中取出对应属性值
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260608171025225.png)

# 4. 制作GA(技能)
- 创建继承自UGameplayAbility类的类
- 编辑器中创建GameplayAbility类的蓝图(继承上面)

## 1. 以UGameplayAbility创建类
- .h (创建了技能消耗枚举+技能信息结构，获取技能信息函数(UI)，获取技能能否使用状态(逻辑))
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
#pragma once  
  
#include "CoreMinimal.h"  
#include "Abilities/GameplayAbility.h"  
#include "BaseGameplayAbility.generated.h"  
  
//技能消耗类型  
UENUM(BlueprintType)  
enum  ECostType:uint8  
{  
    //血量  
    ECT_HP UMETA(DisplayName="HP"),  
    //魔法值  
    ECT_MP UMETA(DisplayName="MP"),  
    //能量  
    ECT_SP UMETA(DisplayName="SP"),  
};  
  
//游戏技能信息  
USTRUCT(BlueprintType)  
struct FGameplayAbilityInfo  
{  
    GENERATED_BODY()  
public:  
    //技能类  
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly,Category="AbilityInfo")  
    TSubclassOf<class UBaseGameplayAbility> AbilityClass;  
    //技能图标  
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category="AbilityInfo")  
    UMaterialInstance* IconMaterial;  
    //技能CD  
    UPROPERTY(VisibleDefaultsOnly, BlueprintReadOnly, Category="AbilityInfo")  
    float CD;  
    //技能消耗类型  
    UPROPERTY(VisibleDefaultsOnly, BlueprintReadOnly, Category="AbilityInfo")  
    TEnumAsByte<ECostType> CostType;  
    //TEnumAsByte 限制<>内枚举类型的值为uint8,节省内存  
  
    //技能消耗值  
    UPROPERTY(VisibleDefaultsOnly, BlueprintReadOnly, Category="AbilityInfo")  
    float CostValue;  
    //无参构造函数  
    FGameplayAbilityInfo()  
    {       
    AbilityClass = nullptr;  
       IconMaterial = nullptr;  
       CD = 0.f;  
       CostType = ECostType::ECT_HP;  
       CostValue = 0.f;  
    }    /** 有参数构造  
     ** @param AbilityClass 技能类  
     * @param IconMaterial 技能图标  
     * @param CD 技能CD  
     * @param CostType 技能消耗类型  
     * @param CostValue 技能消耗值  
     *     */  
    FGameplayAbilityInfo(  
    TSubclassOf<class UBaseGameplayAbility> AbilityClass,  
    UMaterialInstance* IconMaterial,  
    float CD,  
    TEnumAsByte<ECostType> CostType,  
    float CostValue  
    )    {       this->AbilityClass = AbilityClass;  
       this->IconMaterial = IconMaterial;  
       this->CD = CD;  
       this->CostType = CostType;  
       this->CostValue = CostValue;  
    }          
};  
  
  
/**  
 * 游戏技能基类  
 */UCLASS()  
class PTGAS_API UBaseGameplayAbility : public UGameplayAbility  
{  
    GENERATED_BODY()  
    public:  
    //技能图标  
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category="AbilityInfo")  
    UMaterialInstance* IconMaterial;  
    //获取技能信息  
    UFUNCTION(BlueprintCallable, Category="AbilityInfo")  
    FGameplayAbilityInfo GetAbilityInfo(const int Level = 0) const;  
    //判断技能消耗是否满足  
    UFUNCTION(BlueprintCallable, Category="AbilityInfo")  
    bool IsSatisfyCost() const;  
    //  
    };
```

- .cpp(对两个函数的实现，获取当前GameEffect的信息，判断对应技能消耗的属性值是否能支持这个技能的释放)
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
  
#include "Abilitys/BaseGameplayAbility.h"  
#include "AbilitySystemblueprintLibrary.h"  
#include "AbilitySystemComponent.h"  
#include "Abilitys/BaseAttributeSet.h"  
  
  
FGameplayAbilityInfo UBaseGameplayAbility::GetAbilityInfo(const int Level)const  
{  
    //获取冷却和消耗的GameplayEffect  
    const UGameplayEffect* CDEffect = GetCooldownGameplayEffect();  
    const UGameplayEffect* CostEffect = GetCostGameplayEffect();  
    //技能消耗类型  
    ECostType CostType = ECT_HP;  
    if (CDEffect && CostEffect)  
    {       //获取技能CD  
       float CDValue = 0.f;  
              CDEffect->DurationMagnitude.GetStaticMagnitudeIfPossible(Level, CDValue);  
       if (CostEffect->Modifiers.Num() > 0)  
       {          //获取技能的消耗  
                   float CostValue = 0.f;  
                   FGameplayModifierInfo CostEffectModifierInfo = CostEffect->Modifiers[0];  
                   CostEffectModifierInfo.ModifierMagnitude.GetStaticMagnitudeIfPossible(Level, CostValue);  
                   const FString CostTypeName = CostEffectModifierInfo.Attribute.AttributeName;  
                   if (CostTypeName == "HP")  
                   {                      CostType = ECT_HP;  
                   }                   else if (CostTypeName == "MP")  
                   {                      CostType = ECT_MP;  
                   }                   else if (CostTypeName == "SP")  
                   {                      CostType = ECT_SP;  
                   }          return FGameplayAbilityInfo(GetClass(), IconMaterial, CDValue, CostType, CostValue);  
       }       return FGameplayAbilityInfo(GetClass(), IconMaterial, CDValue, CostType, 0.f);  
    }    return FGameplayAbilityInfo();  
}  
  
//判断技能消耗是否满足  
bool UBaseGameplayAbility::IsSatisfyCost() const  
{  
    if (UAbilitySystemComponent* AbilitySystemComponent = UAbilitySystemBlueprintLibrary::GetAbilitySystemComponent(GetOwningActorFromActorInfo()))  
    {       const FGameplayAbilityInfo AbilityInfo = GetAbilityInfo(0);  
       float Value = 0.f;  
       bool bResult = false;  
       switch (AbilityInfo.CostType)  
       {          case ECT_HP:  
             Value = UAbilitySystemBlueprintLibrary::GetFloatAttributeFromAbilitySystemComponent(AbilitySystemComponent, UBaseAttributeSet::GetHPAttribute(),bResult);  
             break;  
          case ECT_MP:  
             Value = UAbilitySystemBlueprintLibrary::GetFloatAttributeFromAbilitySystemComponent(AbilitySystemComponent, UBaseAttributeSet::GetMPAttribute(),bResult);  
             break;  
          case ECT_SP:  
             Value = UAbilitySystemBlueprintLibrary::GetFloatAttributeFromAbilitySystemComponent(AbilitySystemComponent, UBaseAttributeSet::GetSPAttribute(),bResult);  
             break;  
       default:  
          break;  
       }       if (bResult && Value > AbilityInfo.CostValue)  
       {          return true;  
       }    }    return false;  
}
```

## 2. BaseCharacter中加入获取技能信息函数（新增）
- .h
```cpp

//获取技能信息  

UFUNCTION(BlueprintPure, Category="AbilitySystem")  
FGameplayAbilityInfo GetAbilityInfo(const TSubclassOf<UBaseGameplayAbility> AbilityClass, const int Lecel = 0)const;

```

- .cpp
```cpp
//获取技能信息  
FGameplayAbilityInfo ABaseCharacter::GetAbilityInfo(const TSubclassOf<UBaseGameplayAbility> AbilityClass,  
    const int Lecel) const  
{  
    if (const UBaseGameplayAbility* Ability = AbilityClass->GetDefaultObject<UBaseGameplayAbility>();AbilitySystemComponent)  
    {       return Ability->GetAbilityInfo(Lecel);  
    }    return FGameplayAbilityInfo();  
}
```

# 5. GAS技能相关制作
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260616211913594.png)
## 1.被动技能制作(恢复)
- 创建一个继承自BP_GameplayAbility的蓝图
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260617151441903.png)
- 创建新的Cost Gameplay Effect Class的蓝图类
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260617151551968.png)
- 新建的类中
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260617151748776.png)
- 设置被动技能带来的属性变化
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260617151834870.png)

- 新建被动技能的ts文件
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260617152009751.png)
- 添加触发内容
```ts
import * as UE from 'ue';  
import mixin from "../../../mixin";  
  
  
const AssetPath = "/Game/BluePrints/Ability/BaseAbility/GA_BaseResponse.GA_BaseResponse_C";  
  
export interface GA_BaseResponse extends UE.Game.BluePrints.Ability.BaseAbility.GA_BaseResponse.GA_BaseResponse_C {  
}  
  
@mixin(AssetPath)  
  
export class GA_BaseResponse implements GA_BaseResponse {  
    //这是 GA（Gameplay Ability） 的激活入口函数，对应 C++ 中的 ActivateAbility。  
    k2_ActivateAbility(){  
        this.K2_CommitAbilityCost(); //触发编辑器里绑定的那个Cast类(新建的)  
    }  
          
}
```
- 在BP_BaseCharacter.ts，引入蓝图文件，
```ts
import * as UE from "ue";  
  
import mixin from "../../mixin";  
  
const AssetPath = "/Game/BluePrints/Character/BP_BaseCharacter.BP_BaseCharacter_C";  
  
const GA_BaseResponseClass = UE.Class.Load("/Game/BluePrints/Ability/BaseAbility/GA_BaseResponse.GA_BaseResponse_C");  
//导入蓝图文件  
  
export interface BP_BaseCharacter extends UE.Game.BluePrints.Character.BP_BaseCharacter.BP_BaseCharacter_C{  
    }  
  
@mixin(AssetPath)  
export class BP_BaseCharacter implements BP_BaseCharacter {  
    ReceiveBeginPlay(){  
        this.InitAbility();  
        this.InitBind();  
    }    //初始化技能  
    InitAbility(){  
        if(GA_BaseResponseClass){  
            this.AbilitySystemComponent.K2_GiveAbilityAndActivateOnce(GA_BaseResponseClass);  
            //调用蓝图对象的ASC组件的"GiveAbilityAndActivateOnce"节点  
        }  
        //GAS 提供的蓝图节点方法  
        //     - 给予该角色这个能力(Gameplay类)  
        //     - 立即激活一次该能力（执行 ActivateAbility 逻辑）  
        //     - 执行完后移除这个能力（不会保留在能力列表里）  
        //   - 用途：适用于一次性效果，比如受击反馈、出生特效、被动触发等  
    }  
    InitBind(){  
        this.HPChanged.Add((...args)=> this.HPChangedEvent(...args));  
        //- this.HPChanged：一个 多播委托（Multicast Delegate），当 HP 变化时触发  
        //   - .Add(...)：向这个委托注册一个监听函数  
        //   - (...args) => this.HPChangedEvent(...args)：箭头函数，将所有参数透传给 HPChangedEvent 方法  
    }  
    HPChangedEvent(Value:number){  
        UE.KismetSystemLibrary.PrintString(this, Value.toString(),true,true,UE.LinearColor.Green);  
        // - 当 HP 变化时被调用，Value 是新的 HP 值  
        //   - UE.KismetSystemLibrary.PrintString：在屏幕上打印字符串（对应蓝图中的 Print String 节点）  
        //     - 参数依次是：WorldContext、字符串、是否打印到屏幕、是否打印到日志、颜色  
    }  
}
```

## 2.平A攻击(间隔与动画)
创建GA----创建GE_CD---创建GE_Danmage----创建表格---创建GA.ts(修改)---创建PlayerController.ts(修改)---BP_BaseCharacter.ts中添加内容----创建蒙太奇动画(添加几段动画序列+对应序号，另外动画蓝图中加入插槽)---融合上下身的动画

### 创建平A的GA与CD的Effect
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260623214348908.png)
- 绑定GA_Melee
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260625151243823.png)
- 改技能CD
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260625151437555.png)

### 平A随等级变化的csv表
- 项目目录新建一个文件夹
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260625151534113.png)
- 新建.csv 表格
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260625151816492.png)
- 新建数据表
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260621215026724.png)
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260621215159469.png)

### 激活技能在“角色蓝图.ts”中
```ts
import * as UE from "ue";  
  
import mixin from "../../mixin";  
import {$Nullable} from "puerts";  
  
const AssetPath = "/Game/BluePrints/Character/BP_BaseCharacter.BP_BaseCharacter_C";  
  
const GA_BaseResponseClass = UE.Class.Load("/Game/BluePrints/Ability/BaseAbility/GA_BaseResponse.GA_BaseResponse_C");  
//导入蓝图文件  
  
const GA_MeleeClass = UE.Class.Load("/Game/BluePrints/Ability/_00Melee/GA_Melee.GA_Melee_C")  
  
  
export interface BP_BaseCharacter extends UE.Game.BluePrints.Character.BP_BaseCharacter.BP_BaseCharacter_C{  
}  
  
@mixin(AssetPath)  
export class BP_BaseCharacter implements BP_BaseCharacter {  
    ReceiveBeginPlay(){  
        this.InitAbility();  
        this.InitBind();  
    }    //初始化技能  
    InitAbility(){  
        if(GA_BaseResponseClass){  
            this.AbilitySystemComponent.K2_GiveAbilityAndActivateOnce(GA_BaseResponseClass);  
            //调用蓝图对象的ASC组件的"GiveAbilityAndActivateOnce"节点  
        }  
        //GAS 提供的蓝图节点方法  
        //     - 给予该角色这个能力(Gameplay类)  
        //     - 立即激活一次该能力（执行 ActivateAbility 逻辑）  
        //     - 执行完后移除这个能力（不会保留在能力列表里）  
        //   - 用途：适用于一次性效果，比如受击反馈、出生特效、被动触发等  
        if(GA_MeleeClass){  
            this.AbilitySystemComponent.K2_GiveAbility(GA_MeleeClass);  
        }            }  
    //激活技能  
    ActivateAbility(AbilityTay:UE.GameplayTag){  
        this.AbilitySystemComponent.TryActivateAbilitiesByTag(this.GetAbilityTag(AbilityTay));  
            }  
    //获取技能标签  
    GetAbilityTag(AbilityTay:UE.GameplayTag):UE.GameplayTagContainer{  
        return UE.BlueprintGameplayTagLibrary.MakeGameplayTagContainerFromTag(AbilityTay as UE.GameplayTag);  
    }          
      
InitBind(){  
        this.HPChanged.Add((...args)=> this.HPChangedEvent(...args));  
        //- this.HPChanged：一个 多播委托（Multicast Delegate），当 HP 变化时触发  
        //   - .Add(...)：向这个委托注册一个监听函数  
        //   - (...args) => this.HPChangedEvent(...args)：箭头函数，将所有参数透传给 HPChangedEvent 方法  
        this.MPChanged.Add((...args)=> this.MPChangedEvent(...args));  
        this.SPChanged.Add((...args)=> this.SPChangedEvent(...args));  
        this.DamageBox.OnComponentBeginOverlap.Add((...args)=> this.WeaponOverlop(...args));  
            }  
    WeaponOverlop(OverlappedComponent: $Nullable<UE.PrimitiveComponent>, OtherActor: $Nullable<UE.Actor>, OtherComp: $Nullable<UE.PrimitiveComponent>, OtherBodyIndex: number, bFromSweep: boolean, SweepResult: UE.HitResult)  
    {        if(this == OtherActor) return;  
        UE.KismetSystemLibrary.PrintString(  
            this,   
`${this.GetName()}击中了->${OtherActor.GetName()}`,  
            true,  
            true,  
            UE.LinearColor.Green,  
            5.0  
        );  
            }  
          
HPChangedEvent(Value:number){  
        // UE.KismetSystemLibrary.PrintString(this, Value.toString(),true,true,UE.LinearColor.Green);  
        // - 当 HP 变化时被调用，Value 是新的 HP 值  
        //   - UE.KismetSystemLibrary.PrintString：在屏幕上打印字符串（对应蓝图中的 Print String 节点）  
        //     - 参数依次是：WorldContext、字符串、是否打印到屏幕、是否打印到日志、颜色  
    }  
    MPChangedEvent(Value:number){  
        // UE.KismetSystemLibrary.PrintString(this, Value.toString(),true,true,UE.LinearColor.Green);  
    }  
    SPChangedEvent(Value:number){  
        // UE.KismetSystemLibrary.PrintString(this, Value.toString(),true,true,UE.LinearColor.Green);  
    }  
    }
```

### PlayerController 绑定平A按键，ts中写平A触发逻辑
- 里面新建一个函数，在ts中重写逻辑
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260625153105439.png)
- ts逻辑，找到controller控制的角色，使用角色的函数激活技能
```ts
import * as UE from 'ue';  
import mixin from "../../../mixin";  
import {BP_BaseCharacter} from "../BP_BaseCharacter";  
  
const AssetPath = "/Game/BluePrints/Character/Player/BP_PlayerController.BP_PlayerController_C";  
  
const MeleeTag = new UE.GameplayTag("Ability.Melee");  
  
export interface BP_PlayerController extends UE.Game.BluePrints.Character.Player.BP_PlayerController.BP_PlayerController_C {  
}  
  
@mixin(AssetPath)  
export class BP_PlayerController implements BP_PlayerController {  
    //玩家  
    BP_Player : BP_BaseCharacter;  
    ReceiveBeginPlay() {  
        this.BP_Player = UE.GameplayStatics.GetPlayerCharacter(this,0) as BP_BaseCharacter;  
    }    //普通攻击(重写编辑器里的同名函数)  
    Melee(){  
        if(this.BP_Player){  
            this.BP_Player.ActivateAbility(MeleeTag);  
            //对应BP_BaseCharacter.ts里的ActivateAbility方法，传入一个GameplayTag参数  
        }  
    }}
```

### 创建平A的蒙太奇动画，分段，加入GA_Melee.ts逻辑
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260625153953100.png)
- 动画蓝图修改
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260625154402162.png)
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260625154434980.png)
- GA_Melee.ts
```ts
import * as UE from 'ue';  
import mixin from "../../../mixin";  
  
  
const AssetPath = "/Game/BluePrints/Ability/_00Melee/GA_Melee.GA_Melee_C";  
const MA_Melee = UE.AnimMontage.Load("/Game/BluePrints/Character/Animations/Montage/MA_Melee.MA_Melee");  
  
export interface GA_Melee extends UE.Game.BluePrints.Ability._00Melee.GA_Melee.GA_Melee_C {  
}  
  
 @mixin(AssetPath) //有继承：export class GA_Melee extends xxxx implements GA_Melee { }  
export class GA_Melee implements GA_Melee {  
    //当GA触发的时候执行  
    K2_ActivateAbility() {  
        console.log("普通攻击生效");  
        this.K2_CommitAbility();  
        this.PlayMeleeMontage();  
    }  
    //播放普通攻击蒙太奇  
    private PlayMeleeMontage(){  
        //加载蒙太奇资源  
        const StartSection = UE.KismetMathLibrary.RandomInteger(3).toString();  
        let MeleeMontageTask = UE.AbilityTask_PlayMontageAndWait.CreatePlayMontageAndWaitProxy(  
            this,  
            "",  
            MA_Melee,            1,  
            StartSection        );        MeleeMontageTask.OnCompleted.Add(()=>this.K2_EndAbility());//完整播放完毕  
        MeleeMontageTask.OnInterrupted.Add(()=>this.K2_EndAbility());//被其他动画打断  
        MeleeMontageTask.OnBlendOut.Add(()=>this.K2_EndAbility());//Blend out过渡完成  
        MeleeMontageTask.OnCancelled.Add(()=>this.K2_EndAbility());//任务被取消  
        //这上面四个回调都绑定了结束这个任务，无论哪种情况都会结束任务  
        MeleeMontageTask.ReadyForActivation();  //激活这个任务  
        }  
}
```


## 3.平A攻击(伤害)
改BP_BaseCharacter的碰撞预设----加个Box碰撞箱---调整碰撞预设(只与Pawn重叠)---创建通知类---在蒙太奇挥刀位置加上一对标签（一定与人物蓝图中函数同名）---在Puerts中完成函数的逻辑部分

### 武器碰撞与触发
### 在角色蓝图中的网格体下添加，一个Box碰撞，绑定在骨骼的对应插槽上
### 设置碰撞预设(避免没必要的物品触发了碰撞)
- 角色胶囊体的
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260625160527976.png)
- 角色骨骼网格体的
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260625160557183.png)
- Box的
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260625160622621.png)


### “角色”ts中写入碰撞触发事件
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260625155147052.png)

### 新建继承自AnimNotify的Unreal类(用在蒙太奇通知处，标签起什么名，对应角色蓝图的函数就起什么名，会直接触发对应函数逻辑)
```cpp
#pragma once  
  
#include "CoreMinimal.h"  
#include "Animation/AnimNotifies/AnimNotify.h"  
#include "LfAnimNotify_Script.generated.h"  
  
/**  
 * 动画脚本通知(用于动画直接调用函数，无参的)  
 */UCLASS()  
class PTGAS_API ULfAnimNotify_Script : public UAnimNotify  
{  
    GENERATED_BODY()  
    UPROPERTY(EditAnywhere, Category="Name")  
    FName FunctionName = "None";  
    //获取通知名称  
    virtual FString GetNotifyName_Implementation() const override;  
    //触发通知  
    virtual void Notify(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation, const FAnimNotifyEventReference& EventReference) override;  
};
```

```cpp

#include "AnimNotify/LfAnimNotify_Script.h"  
  
FString ULfAnimNotify_Script::GetNotifyName_Implementation() const  
{  
    return FunctionName.ToString();  
}  
  
void ULfAnimNotify_Script::Notify(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation,  
    const FAnimNotifyEventReference& EventReference)  
{  
    Super::Notify(MeshComp, Animation, EventReference);  
    if (!MeshComp) return;  
    if (AActor* TempActor = MeshComp->GetOwner())  
    {       if (UFunction* TempFunction = TempActor->FindFunction(FunctionName))  
       {          UE_LOG(LogTemp, Warning, TEXT("AnimNotify_Script: %s"), *FunctionName.ToString());  
                    TempActor->ProcessEvent(TempFunction, nullptr);  
       }    }    }
```

### 更新武器触发逻辑，每次触发只造成一次伤害(Actor数组)
```ts
WeaponOverlop(OverlappedComponent: $Nullable<UE.PrimitiveComponent>, OtherActor: $Nullable<UE.Actor>, OtherComp: $Nullable<UE.PrimitiveComponent>, OtherBodyIndex: number, bFromSweep: boolean, SweepResult: UE.HitResult)  
{  
    if(this == OtherActor) return;  
    if(!this.HitActor.Contains(OtherActor)){  
        this.HitActor.Add(OtherActor);  
        //这个数组存起来，保证每次平A只会命中一次  
        UE.KismetSystemLibrary.PrintString(  
            this,  
            `${this.GetName()}击中了->${OtherActor.GetName()}`,  
            true,  
            true,  
            UE.LinearColor.Green,  
            5.0  
        );  
            }  
    }  
  
//开始伤害(蒙太奇通知)  
BeginDamage() {  
    this.HitActor.Empty();  
    this.DamageBox.SetCollisionEnabled(UE.ECollisionEnabled.QueryOnly);  
    }  
  
EndDamage() {  
    this.HitActor.Empty();  
    this.DamageBox.SetCollisionEnabled(UE.ECollisionEnabled.NoCollision);  
}
```

### 伤害扣血逻辑，发送标签，使用一个GAS的函数回调
- 项目设置里加标签
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260625161152137.png)
- “基础角色”.ts中进行触发时，发送事件
```ts
//命中标签  
const MeleeHitTag = new UE.GameplayTag("Ability.Melee.HitEvent");

InitBind(){  
    。。。
    
    
    this.DamageBox.OnComponentBeginOverlap.Add((...args)=> this.WeaponOverlop(...args));  
    }


WeaponOverlop(OverlappedComponent: $Nullable<UE.PrimitiveComponent>, OtherActor: $Nullable<UE.Actor>, OtherComp: $Nullable<UE.PrimitiveComponent>, OtherBodyIndex: number, bFromSweep: boolean, SweepResult: UE.HitResult)  
{  
    if(this == OtherActor) return;  
    if(!this.HitActor.Contains(OtherActor)){  
        this.HitActor.Add(OtherActor);  
        //这个数组存起来，保证每次平A只会命中一次  
        UE.KismetSystemLibrary.PrintString(  
            this,  
            `${this.GetName()}击中了->${OtherActor.GetName()}`,  
            true,  
            true,  
            UE.LinearColor.Green,  
            5.0  
        );  
        const GameplayEventData = new UE.GameplayEventData();  
        GameplayEventData.EventTag = MeleeHitTag;  
        GameplayEventData.Instigator = this;  
        GameplayEventData.Target = OtherActor;  
        UE.AbilitySystemBlueprintLibrary.SendGameplayEventToActor(this,MeleeHitTag,GameplayEventData);  
            }  
    }
```
- GA_Melee.ts 中进行绑定事件
```ts
const MeleeHitTag = new UE.GameplayTag("Ability.Melee.HitEvent");  
  
//伤害类  
const MeleeDamageClass = UE.Class.Load("/Game/BluePrints/Ability/_00Melee/GE_Melee_Damage.GE_Melee_Damage_C");

//绑定命中事件  
private BindHitEvent(){  
    const GameplayEvent = UE.AbilityTask_WaitGameplayEvent.WaitGameplayEvent(this, MeleeHitTag,null,false,true);  
  
    GameplayEvent.EventReceived.Add((...arge)=> this.HitEvent(...arge));  
  
    GameplayEvent.ReadyForActivation();  
}  
  
//命中事件触发,扣血  
private HitEvent(Payload: UE.GameplayEventData){  
    // 测试打印 UE.KismetSystemLibrary.PrintString(this, "命中事件触发", true, true, new UE.LinearColor(1, 0, 0, 1), 5);  
    
    this.BP_ApplyGameplayEffectToTarget
    (          UE.AbilitySystemBlueprintLibrary.AbilityTargetDataFromActor(Payload.Target),   
        MeleeDamageClass,  
        UE.KismetMathLibrary.RandomIntegerInRange(0,4)  
        )   
        
 }


```


## 4.伤害UI更新显示
新建UMG蓝图，添加进度条----敌人添加Widget组件--创建UMG蓝图.ts---绑定血条显示的是属性值--- UI旋转面向玩家

- 创建简单的敌人血条显示widget，对数字与进度条的数值创建绑定函数
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260703161809544.png)
- 敌人蓝图中添加组件，一定是world显示(这样UI的显示随距离有大小变化)
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260703161903733.png)
- 创建控件蓝图的ts文件，重写显示文字与数值的绑定函数
```ts
import * as UE from 'ue';  
import mixin from "../../../../mixin";

const AssetPath = "/Game/BluePrints/Character/Enemy/UMG/UMG_EnemyBar.UMG_EnemyBar_C";  
  
export interface UMG_EnemyBar extends UE.Game.BluePrints.Character.Enemy.UMG.UMG_EnemyBar.UMG_EnemyBar_C {  
}  
  
@mixin(AssetPath)  
//有继承：export class UMG_EnemyBar extends xxxx implements UMG_EnemyBar { }  
export class UMG_EnemyBar implements UMG_EnemyBar {  
    //血量  
    HP:number;  
    //最大血量  
    Max_HP:number;  
    
    GetPercent():number
    {  
        return UE.KismetMathLibrary.FClamp(this.HP/this.Max_HP,0,1);  
    }    
    
    Get_BarText():string
    {  
        return `${this.HP}/${this.Max_HP}`;  
        //给文本框的绑定函数返回显示字符串  
    }  
    }
```

- 在BP_Enemy.ts中写好UI更新时机以及控制UI面向人屏幕旋转
```ts
import * as UE from 'ue';  
import mixin from "../../../mixin";  
import {BP_BaseCharacter} from "../BP_BaseCharacter";  
import {UMG_EnemyBar} from "./UMG/UMG_EnemyBar";  
import {$ref, $Ref} from "puerts";  
  
const AssetPath = "/Game/BluePrints/Character/Enemy/BP_Enemy.BP_Enemy_C";  
//创建属性  
const AttributeSetHP = new UE.GameplayAttribute("HP","/Script/PTGAS.BaseAttributeSet:HP",null);  
const AttributeSetMaxHP = new UE.GameplayAttribute("MaxHP","/Script/PTGAS.BaseAttributeSet:MaxHP",null);  
  
  
export interface BP_Enemy extends UE.Game.BluePrints.Character.Enemy.BP_Enemy.BP_Enemy_C {  
}  
  
@mixin(AssetPath)  
export class BP_Enemy extends BP_BaseCharacter implements BP_Enemy {  
    UMG_Bar: UMG_EnemyBar;  
    bSuccess: $Ref<boolean> = $ref(false);  
    NewRotation: UE.Rotator;  
    _rotationIntervalId: ReturnType<typeof setInterval> | null = null;  
  
    ReceiveBeginPlay()
    {  
        // super.ReceiveBeginPlay();  
        this.InitAbility();  
        this.InitBind();  
  
        this.UMG_Bar = this.Bar.GetUserWidgetObject() as UMG_EnemyBar;  
        this.SetBarValue();  
  
        // 在 BeginPlay 中初始化，不能放类字段初始化器里（mixin 模式下不会执行）  
        this.NewRotation = new UE.Rotator();  
  
        // 用 setInterval 代替 Tick        this._rotationIntervalId = setInterval(() => {  
            this.SetBarRotation();  
        }, 50);  
        //50ms执行一次  
    }  
    
    
    //监听血量变化  
 protected HPChangedEvent(Value: number) {  
     super.HPChangedEvent(Value);  
     this.SetBarValue();  
 }  
 
//ReceiveTick(DeltaSeconds: number) {  
    //super.ReceiveTick(DeltaSeconds);这段导致了崩溃  
    //this.SetBarRotation();  
//}  

    SetBarValue(){  
        if(this.UMG_Bar){  
            this.UMG_Bar.HP = UE.AbilitySystemBlueprintLibrary.GetFloatAttributeFromAbilitySystemComponent  
            (  
                this.AbilitySystemComponent,  
                AttributeSetHP,                this.bSuccess  
            );  
            this.UMG_Bar.Max_HP = UE.AbilitySystemBlueprintLibrary.GetFloatAttributeFromAbilitySystemComponent  
            (  
                this.AbilitySystemComponent,  
                AttributeSetMaxHP,                this.bSuccess  
            );  
            console.log(this.UMG_Bar.HP,this.UMG_Bar.Max_HP);  
        }   
         }  
        
    SetBarRotation(){  
        const CameraRotation = UE.GameplayStatics.GetPlayerCameraManager(this,0).K2_GetActorRotation();  
        this.NewRotation.Pitch = CameraRotation.Pitch * -1;  
        this.NewRotation.Yaw = CameraRotation.Yaw + 180;  
        this.NewRotation.Roll = CameraRotation.Roll * -1;  
        this.Bar.K2_SetWorldRotation(this.NewRotation, false, null, false);  
    }}
```

## 5. 角色死亡(玩家/敌人)后的处理

死亡制作以及移除被动

### 动画蓝图动画
1. 动画蓝图中创建一个Dead的bool变量
2. ![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260707172403329.png)

### Base与Enmey的添加
1. 在Base蓝图中添加Dead的bool变量，表示死亡状态
2. 在Base.ts 的HP监听函数中，设置死亡后需要改的设置
	1. 设置动画蓝图Dead=true;
	2. 自己蓝图变量Dead=true;
	3. 移除自动回血的技能；
	4. 关闭物理碰撞
3. Enmey.ts中需要关闭Tick(定时器)和Bar(血条UI)

- Base.ts
```ts
const BaseResponseTag = new UE.GameplayTag("Ability.BaseResponse");

ABP_Sinbi:UE.Game.BluePrints.Character.Animations.ABP_Sinbi.ABP_Sinbi_C = null;

protected BaseInit(){  
    this.ABP_Sinbi = this.Mesh.GetAnimInstance() as UE.Game.BluePrints.Character.Animations.ABP_Sinbi.ABP_Sinbi_C;  
    this.InitAbility();  
    this.InitBind();  
}

protected HPChangedEvent(Value:number){  
    if(Value <= 0 && !this.Dead){  
        this.Dead = true;  
        this.ABP_Sinbi.Dead = true;//设置动画蓝图的死亡状态  
        //移除被动回复效果  
        this.AbilitySystemComponent.RemoveActiveEffectsWithTags(this.GetAbilityTag(BaseResponseTag));  
        this.CapsuleComponent.SetCollisionEnabled(UE.ECollisionEnabled.NoCollision);  
    }      
}

```

- Enmey.ts
```ts
protected HPChangedEvent(Value: number) {  
    super.HPChangedEvent(Value);  
    this.SetBarValue();  
    if(this.Dead){  
        if(this.Bar){  
            this.Bar.K2_DestroyComponent(this);  
        }        if(this._rotationIntervalId!==null){  
            clearInterval(this._rotationIntervalId);  
            this._rotationIntervalId = null;  
        }    }}
        
SelfTick(){  
    if(!this.Dead){  
        this.SetBarRotation();  
    }   
    }
        
```

## 6. 敌人简单行为树
创建AI控制器/黑板---绑定敌人蓝图的pawn---创建导航体积---创建任务(寻找玩家，普通攻击)---新建AI控制。ts(启动行为树)---新建Find，Melee.ts---禁用玩家死亡后的控制
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260712172139970.png)

## 7. UI
### UI面板搭建(Widget)
创建三个UMG面板，分别是技能图标，属性图标，还有主体框
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260717194956611.png)
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260715001752956.png)

### UI面板功能实现
激活控件蓝图中的红色事件(还需创建一个变量Linear Color)---控件图标设置为变量---创建3个控件的ts文件---创建BP_GameplayAbility.ts---在Controller里创建Widget添加到窗口
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260718004239426.png)
- UMG_Main.ts
```ts
import * as UE from 'ue';  
import mixin from "../../../../mixin";  
import {TArray} from "ue";  
  
const AssetPath = "/Game/BluePrints/Character/Player/UMG/UMG_MainUI.UMG_MainUI_C";  
  
export interface UMG_MainUI extends UE.Game.BluePrints.Character.Player.UMG.UMG_MainUI.UMG_MainUI_C {  
}  
  
@mixin(AssetPath)  
export class UMG_MainUI implements UMG_MainUI {  
    AllAbilitySlot:TArray<UE.Game.BluePrints.Character.Player.UMG.UMG_AbilitySlot.UMG_AbilitySlot_C>  
    OnInitialized() {  
            }  
    PreConstruct(IsDesignTime: boolean) {  
        this.AllAbilitySlot.Add(this.AbilitySlot_1);  
        this.AllAbilitySlot.Add(this.AbilitySlot_2);  
        this.AllAbilitySlot.Add(this.AbilitySlot_3);  
        this.AllAbilitySlot.Add(this.AbilitySlot_4);  
        this.AllAbilitySlot.Add(this.AbilitySlot_5);  
  
  
    }}
```
- UMG_AttributeBar.ts
```ts
import * as UE from 'ue';  
import mixin from "../../../../mixin";  
  
const AssetPath = "/Game/BluePrints/Character/Player/UMG/UMG_AttributeBar.UMG_AttributeBar_C";  
  
export interface UMG_AttributeBar extends UE.Game.BluePrints.Character.Player.UMG.UMG_AttributeBar.UMG_AttributeBar_C {  
}  
  
@mixin(AssetPath)  
//有继承：export class UMG_AttributeBar extends xxxx implements UMG_AttributeBar { }  
export class UMG_AttributeBar implements UMG_AttributeBar {  
    PreConstruct(IsDesignTime: boolean) {  
        this.SetColor();  
    }    protected SetColor(){  
        this.Image_Bar.GetDynamicMaterial().SetVectorParameterValue("Color",this.Color);  
    }    SetProgress(Progress:number){  
        this.Image_Bar.GetDynamicMaterial().SetScalarParameterValue("Pre",UE.KismetMathLibrary.FClamp(Progress,0,1));  
    }          
}
```
- UMG_AbilitySlot.ts
```ts
import * as UE from 'ue';  
import mixin from "../../../../mixin";  
import{BP_GameplayAbility} from "../../../Ability/BP_GameplayAbility";  
  
  
const AssetPath = "/Game/BluePrints/Character/Player/UMG/UMG_AbilitySlot.UMG_AbilitySlot_C";  
  
export interface UMG_AbilitySlot extends UE.Game.BluePrints.Character.Player.UMG.UMG_AbilitySlot.UMG_AbilitySlot_C {  
}  
  
@mixin(AssetPath)  
export class UMG_AbilitySlot implements UMG_AbilitySlot {  
    //总CD  
    CD_Intel:number;  
    //当前CD  
    CD_Current:number;  
    //在技能CD区间  
    IsDuringCD:boolean;  
    //技能类  
    AbilityClass:UE.Class;  
    PreConstruct(IsDesignTime: boolean) {  
        this.Key.SetText(this.KeyText);  
    }  
    Tick(MyGeometry: UE.Geometry, InDeltaTime: number) {  
        this.UpdateCD(InDeltaTime);  
    }  
    //初始化信息  
    InitInfo(AbilityInfo:UE.GameplayAbilityInfo){  
        this.CD_Intel = AbilityInfo.CD;  
        this.AbilityClass=AbilityInfo.AbilityClass;  
        this.AbilityImage.SetBrushFromMaterial(AbilityInfo.IconMaterial);  
    }    StartUI_CD(){  
        this.IsDuringCD = true;  
        this.CD.SetVisibility(UE.ESlateVisibility.Visible);  
        this.CD_Current = this.CD_Intel;  
    }    UpdateCD(DeltaTime:number) {  
        //处于CD中...  
        if(this.IsDuringCD){  
            this.CD_Current = UE.KismetMathLibrary.FClamp(this.CD_Current-DeltaTime,0,this.CD_Intel);  
            if(this.CD_Current>0){  
                this.CD.SetText(  
                    UE.KismetTextLibrary.Conv_DoubleToText(  
                        this.CD_Current,UE.ERoundingMode.HalfToEven,false,true, 1,324,0,1  
                    )  
                )                this.AbilityImage.GetDynamicMaterial().SetScalarParameterValue("Pre",UE.KismetMathLibrary.FClamp((1-this.CD_Current/this.CD_Intel),0,1))  
            }            else{  
                this.IsDuringCD = false;  
                this.CD.SetVisibility(UE.ESlateVisibility.Hidden);  
                this.AbilityImage.GetDynamicMaterial().SetScalarParameterValue("Pre",1);  
            }        }    }}
```

### UI 属性初始化及绑定
在Main面板中修改三个圆的颜色---player.ts中添加血量变化时的逻辑(蓝图函数，ts重写，圆圈)---在BP_Player中新建技能数组(开启类）---创建测试GA，添加到Player中
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260721174953591.png)
Player.ts
```ts
InitAbility()
{  
    super.InitAbility();  
    
    for(let i=0;i<this.GAS.Num();i++)
    {  
        if(this.GAS.GetRef(i))
        {  
            this.AbilitySystemComponent.K2_GiveAbility(this.GAS.GetRef(i));  
            this.BP_PlayerController.MainUI.AbilitySlots.GetRef(i).InitInfo(this.GetAbilityInfo(this.GAS.GetRef(i),0));  
        }    
    }
}
        //重写了InitAbility()初始化技能函数，循环激活技能

//添加了血量变化时的控制UI变化
protected HPChangedEvent(Value: number){  
    super.HPChangedEvent(Value);  
    const Pre = Value/UE.AbilitySystemBlueprintLibrary.GetFloatAttributeFromAbilitySystemComponent(this.AbilitySystemComponent,AttributeSetMaxHP,null);  
    this.BP_PlayerController.MainUI.HPAttributeBar.SetProgress(Pre);  
          
//玩家死亡时  
    if(this.Dead){  
       //禁用输入  
        this.DisableInput(this.BP_PlayerController);  
           }  
    }

```

![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260721003641002.png)
- 给人物添加技能
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260721174927842.png)
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260721182131517.png)

## 8. 制作技能
### 1. 回血技能
创建ts变量的报错问题(ts中命名-加载-new-卸载)-----添加一个GA(新建对应标签)+三个GE----1.Cost-及时消耗MP数值，2. CD-持续消耗5s，添加阻塞组件，Gameplay Cues（有新标签）-----添加gameplaycue的静态蓝图---在controll蓝图中添加测试，并添加技能同名函数---改controll,gameplayAbility--添加GA.ts(激活)---添加GC_HPRegen.ts(生成粒子)-----GA中播放蒙太奇动画，加载GE_Value，做恢复数值


![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260729170048095.png)
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260808010511552.png)
- GA_HPRegen
```ts
import * as UE from 'ue';  
import mixin from "../../../mixin";  
import {BP_GameplayAbility} from "../BP_GameplayAbility";  
//需要导入mixin 模块  
  
//如果该蓝图继承自别的蓝图，还需要导入对应ts模块  
//import {xxxx} from "../xxxx";  
  
const AssetPath = "/Game/BluePrints/Ability/_01HPRegen/GA_HPRegen.GA_HPRegen_C";  
//恢复动作  
const MA_HPRegen = UE.AnimMontage.Load("/Game/BluePrints/Character/Animations/Montage/MA_HPRegen.MA_HPRegen");  
//恢复数值（Class类需要加 _C）  
const GE_HPRegenValueClass = UE.Class.Load("/Game/BluePrints/Ability/_01HPRegen/GE_HPRegen_Value.GE_HPRegen_Value_C");  
  
export interface GA_HPRegen extends UE.Game.BluePrints.Ability._01HPRegen.GA_HPRegen.GA_HPRegen_C {  
}  
  
@mixin(AssetPath)  
export class GA_HPRegen extends BP_GameplayAbility implements GA_HPRegen {  
    K2_ActivateAbility(){  
        this.K2_CommitAbility();  
        this.StartUI_CD();  
        this.BP_ApplyGameplayEffectToOwner(GE_HPRegenValueClass);  
        this.PlayHPRegenMontage();  
    }    PlayHPRegenMontage(){  
        const MontageTask = UE.AbilityTask_PlayMontageAndWait.CreatePlayMontageAndWaitProxy  
        (  
            this,  
            "HPRegen",       //名称  
            MA_HPRegen                       //动画  
        );  
        MontageTask.OnCompleted.Add(()=>this.K2_EndAbility());  
        MontageTask.OnInterrupted.Add(()=>this.K2_EndAbility());  
        MontageTask.OnBlendOut.Add(()=>this.K2_EndAbility());  
        MontageTask.OnCancelled.Add(()=>this.K2_EndAbility());  
        MontageTask.ReadyForActivation();  
    }}
```

- GC_HPRegen
```ts
import * as UE from 'ue';  
import mixin from "../../../mixin";  
import {$Nullable} from "puerts";  
  
//需要导入mixin 模块  
  
//如果该蓝图继承自别的蓝图，还需要导入对应ts模块  
//import {xxxx} from "../xxxx";  
  
const AssetPath = "/Game/BluePrints/Ability/_01HPRegen/GC_HPRegen.GC_HPRegen_C";  
  
//特效  
const HPRegenFX = UE.ParticleSystem.Load("/Game/Assets/Abilities/HealthRegen/P_HealthRegen.P_HealthRegen");  
  
  
export interface GC_HPRegen extends UE.Game.BluePrints.Ability._01HPRegen.GC_HPRegen.GC_HPRegen_C {  
}  
  
@mixin(AssetPath)  
//有继承：export class GC_HPRegen extends xxxx implements GC_HPRegen { }  
export class GC_HPRegen implements GC_HPRegen {  
    WhileActive(MyTarget: $Nullable<UE.Actor>, Parameters: UE.GameplayCueParameters): boolean {  
        if(HPRegenFX){  
            UE.GameplayStatics.SpawnEmitterAtLocation(  
                this,  
                HPRegenFX,                MyTarget.K2_GetActorLocation(),  
                MyTarget.K2_GetActorRotation(),  
                MyTarget.GetActorScale3D(),  
                true,  
                UE.EPSCPoolMethod.AutoRelease,   //这里原来用的是ManualRelease  
                true  
            )  
        }        return true;  
    }  
}
``` 

### 2.1 冲刺技能(上)
创建GA,GE_Cost,GE_CD 三件套---新的Tag---新建GA_Dash.ts---在BP_Play中添加新GA---ControolerC++中创建新的Dash函数BlueprintNativeEvent(函数中写Player激活技能)---创建绑定输入----添加动画蒙太奇，在GA.ts里面激活这个任务---在BaseCharacter.ts中，新建两个函数，一个用来实现冲刺，另一个用来设置摩擦力---在GA.ts中激活时，使用冲刺函数(注意在结束函数中加入还原摩擦力)

### 2.2 冲刺技能(下)
在BP_Player视图中创建一个碰撞体球(设置初始碰撞)---在BP_Player.ts中添加球的事件，绑定触发回调的函数(添加到HitActor数组)---重载父类的函数SetFrictionToZero，添加设置摄像机杆的碰撞，球的物理检测 查询/无，球的大小SetSphereRadius，清空HitActor---添加新标签(Dash.Hit)---Player.ts中创建GameplayEvent，在技能中监听它，媒介是那个新标签(类似事件分发器)---在技能中创建两个函数，一个监听(启动等待任务)，一个回调(写主要技能逻辑，如添加GE伤害，造成冲刺撞开，造成眩晕剥夺AI控制器控制权)---眩晕内容：Base写好函数，Enmey重写，在GA_Dash.ts中激活眩晕---标签设置，Dash时不可打断，1,BP_GameplayAbility添加标签 2. 给GA_Dash加入激活过程标签(技能激活时有，结束后没有)
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260831165418347.png)
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260831202142350.png)
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260901170350910.png)

### 3.1 射线技能(上)
创建GA,2个GE，1个伤害GE(持续性消耗，添加标签方便移除。CD老样子。瞬时伤害)---一个特殊标签的蒙太奇动画(后段循环)---创建绑定好输入激活那一块(Controller)---在GA_Laser.ts中处理好激活GE，移除对应GE(BaseCharacter中新加了一个是否在激光的bool)，还有镜头锁定----需要做再次点击按键，取消技能。在Controller的函数中添加事件，然后GA中监听
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260901213926952.png)
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260901214049768.png)
![](../../Pasted%20image%2020260901214307.png)
## 9. 输出操作的改写
新建C++类，继承自UExtensionMethods(写一个函数，然后实现一下，是否存在，然后使用绑定函数)---C++类中新加入动作函数---在Controller.ts中去添加输入动作和对应函数(前提创建好IA，添加至上下文)----修改Look，Move函数(有传参数)的输入值Input Action Value---把视野锁定的也添加上
![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260825213934654.png)
- ExtensionMethods类
```cpp
.h

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
```cpp
.cpp
#include "Libary/LF_ExtensionMethods.h"  
  
void ULF_ExtensionMethods::BindAction(UEnhancedInputComponent* InputComponent, const UInputAction* InputAction,  
    const ETriggerEvent TriggerEvent, UObject* Object, const FName& FunctionName)  
{  
    if (InputComponent)  
    {       InputComponent->BindAction(InputAction, TriggerEvent, Object, FunctionName);  
    }}
```
- Controller和Player中都要有对应技能的绑定
```ts
const JumpAction = UE.InputAction.Load("/Game/BluePrints/Input/Action/IA_Jump.IA_Jump")  
const MoveAction = UE.InputAction.Load("/Game/BluePrints/Input/Action/IA_Move.IA_Move")  
const LookAction = UE.InputAction.Load("/Game/BluePrints/Input/Action/IA_Look.IA_Look")

BindKey(){  
    const InputComponent = this.GetComponentByClass(UE.EnhancedInputComponent.StaticClass()) as UE.EnhancedInputComponent;  
    if(InputComponent){  
        InputComponent.BindAction(JumpAction,UE.ETriggerEvent.Started,this,"Jumpp"); //这里的第一个函数必须是蓝图函数，或者C++蓝图可以调用的函数  
        InputComponent.BindAction(MoveAction,UE.ETriggerEvent.Triggered, this,"Move");  
        InputComponent.BindAction(LookAction,UE.ETriggerEvent.Triggered, this,"Look");  
        InputComponent.BindAction(LockCameraAction,UE.ETriggerEvent.Started, this,"IA_LockCamera");  
    }
    }
    
    IA_LockCamera() {  
    this.LookCamera(!this.InLock);  
    this.InLock = !this.InLock;  
}  
  
//鼠标移动视角  
Look(ActionValue: UE.InputActionValue) {  
    const Value2D = UE.EnhancedInputLibrary.Conv_InputActionValueToAxis2D(ActionValue);  
    this.AddControllerYawInput(Value2D.X);  
    this.AddControllerPitchInput(Value2D.Y);  
    }  
  
//移动  
Move(ActionValue: UE.InputActionValue) {  
    //前进  
    const Value2D = UE.EnhancedInputLibrary.Conv_InputActionValueToAxis2D(ActionValue);  
    if (!this.Move_Rotator) {  
        this.Move_Rotator = new UE.Rotator(0, 0, 0);  
    }    this.Move_Rotator.Yaw =this.GetControlRotation().Yaw;  
  
    const ForwardVector = UE.KismetMathLibrary.GetForwardVector(this.Move_Rotator);  
    const RightVector = UE.KismetMathLibrary.GetRightVector(this.Move_Rotator);  
    this.AddMovementInput(ForwardVector,Value2D.Y);  
    this.AddMovementInput(RightVector,Value2D.X);  
    }
```