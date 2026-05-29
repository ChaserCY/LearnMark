# 构建环境
- 创建项目，调整编辑器设置
- 创建Plugins，拖入整个PuerTS插件包，导入Assets资源包
- 拖入package.json，tsconfig.json 文件到项目中，添加global.json文件，限制.NET 的版本为6.0
- ![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260528202217245.png)
- ![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260528202217303.png)- ![](图片/UE/PuerTs/PuerTS-GAS(UE5.3)/IMG-20260528202217355.png)
- 删除Save，.sln，Intermediate，Binaries文件，还有插件的\Plugins\Puerts\Binaries，\Plugins\Puerts\Intermediate文件，然后右键.uproject重构(==插件内部也需要删文件==)
- 打开.sln文件，然后在.Build.cs文件中加入插件名“EnhancedInput”，“Gameplay Ability”，"GameplayTags",  "GameplayTasks"，“Puerts”，“JsEnv”
- 在编辑器-插件，启用GAS ,GASAttachEditor插件，编辑器偏好-源代码-启用Rider Uproject打开


# 项目初始化
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

# 基础角色与相机控制
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
