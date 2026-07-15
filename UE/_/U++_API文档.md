# 声明与定义
## 宏
- 标准生成属性宏,快速生成对象成员的Set，Get方法
```cpp
	#define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \  
    GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \  
    GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \  
    GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \  
    GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)
    
    例如：ATTRIBUTE_ACCESSORS(UBaseAttributeSet, HP)
```

- 常用宏
```cpp
UCLASS()                 + GENERATED_BODY()
UENUM(BlueprintType) 
USTRUCT(BlueprintType)    + GENERATED_BODY()
UPROPERTY()
UFUNCTION()

```
## 类
- 声明UCLASS==类==的指针,TSubclassOf
```cpp
TSubclassOf<class UBaseGameplayAbility> XXX;
```

## 结构体
- 声明结构体与结构体数组
```cpp

```


## 组件
- 声明组件指针
```cpp
TObjectPtr<UAbilitySystemComponent>  AbilitySystemComponent;//这是AbilitySystemComponent（ACS）组件
```

- 声明与创建==组件==
```cpp
<组件类>*  XXX; //声明组件指针

XXX = CreateDefaultSubobject<组件类>(TEXT("展示名"));//创建组件对象


USpringArmComponent* CameraBoom;
CreateDefaultSubobject<USpringArmComponent>(TEXT("CameraBoom"));

//对组件Detail进行设置
CameraBoom->SetupAttachment(GetRootComponent());  //挂载点
CameraBoom->TargetArmLength = 200.0f;             //长度
CameraBoom->SocketOffset = FVector(0.0f, 55.f, 65.f);  //旋转
CameraBoom->bUsePawnControlRotation = true;          //是否控制旋转


```


- 角色==属性数据==的数据类型声明
```cpp
FGameplayAttributeData HP;
```

- ==材质实例==声明
```cpp
UMaterialInstance* IconMaterial;
```

- 声明一种枚举的==枚举值==
```cpp
EMyEnum  XXX; //正常枚举声明
TEnumAsByte<EMyEnum> XXX;//强制限制声明的枚举是一个字节的，即数量不超过254
```

## GameplayTag
- 声明
```cpp
.h
PERSON_GAS_API UE_DECLARE_GAMEPLAY_TAG_EXTERN(InputTag_Move)//声明外部可见的tag
项目名全大写+_API UE_DECLARE_GAMEPLAY_TAG_EXTERN(Tag名)

.cpp
UE_DEFINE_GAMEPLAY_TAG(InputTag_Move, "InputTag.Move")
UE_DEFINE_GAMEPLAY_TAG(Tag名,"在编辑器展示的Tag名")



```

# 方法与函数
- 数学限制(Clamp)
```cpp
FMath::Clamp(当前值，最小值，最大值);//把当前值限制在[最小值，最大值]之中
```

- GAS框架自带的重写函数(GameplayEffect修改后自动调用)，在头文件声明
```cpp
virtual void PostGameplayEffectExecute(const struct FGameplayEffectModCallbackData& Data) override;


//对应cpp文件
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

## 向量
- 速度的向量方法
```cpp
.Size()                //计算三维长度 √(X²+Y²+Z²)，涉及开平方，性能稍慢
.SizeSquared()         //计算三维长度平方 X²+Y²+Z²，避免了开平方  
.SizeSquared2D()       //只计算二维长度平方 X²+Y²，忽略 Z 轴 
```

# 特殊功能
- 委托
```cpp
DECLARE_[动态?]_[多播?]_DELEGATE_[参数数?]Params(名, 类型1, 名1, ...)   

// ========== 普通委托（1个监听者，C++ 专用）==========
  DECLARE_DELEGATE(FDelegate)                          // 0参
  DECLARE_DELEGATE_OneParam(FDelegate, float, Val)     // 1参
  DECLARE_DELEGATE_TwoParams(FDelegate, float, A, int, B)  // 2参
  // ... 一直到 NineParams

  // ========== 多播委托（N个监听者，C++ 专用）==========
  DECLARE_MULTICAST_DELEGATE(FMulticast)
  DECLARE_MULTICAST_DELEGATE_OneParam(FMulticast, float, Val)

  // ========== 动态委托（1个监听者，蓝图可见）==========
  DECLARE_DYNAMIC_DELEGATE(FDynDelegate)
  DECLARE_DYNAMIC_DELEGATE_OneParam(FDynDelegate, float, Val)

  // ========== 动态多播委托（N个监听者，蓝图可见）[你项目里用的]==========
  DECLARE_DYNAMIC_MULTICAST_DELEGATE(FDynMulti)                     // 0参
  DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FDynMulti, float, V)  // 1参 ← 你的
  DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(FDynMulti, float, A, int32, B)

```

- 检查(开发期使用)
```cpp
check(InInputConfig);  
checkf(InInputConfig, TEXT("Input Config data asset is null,can not proceed with binding"));// 这个能写入报错信息


```
![](图片/UE/_/U++_API文档/IMG-20260622193347737.png)
![](图片/UE/_/U++_API文档/IMG-20260622193347811.png)