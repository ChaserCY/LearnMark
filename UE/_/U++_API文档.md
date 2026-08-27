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

ULFSampleHttpTimeAsyncAction* AsyncAction = NewObject<ULFSampleHttpTimeAsyncAction>();
//ULFSampleHttpTimeAsyncAction是自己创建的类
```

## 数组
```cpp
TArray<类> Arr名;
TArray<FString> StrArr

Arr.Add();                  //手上已有现成对象实例
Arr.Emplace();              //要新建对象塞数组，还没实例，会在数组中直接创建对象
Arr.Insert(<元素>，<位置>);  //对应索引位置插入
StrArr.Append(Arr<元素...>); //拼接字符串数组，在后面加一坨
StrArr.AddUnique(元素)       //添加唯一元素
StrArr.EmplaceUnique(元素)  //同上
StrArr.SetNum(数字)          //设置数组长度，多的不要，少时填默认值
StrArr.Remove(元素)          //移除对应元素
StrArr.RemoveAt(索引)        //移除对应索引元素
StrArr.GetData()                   //返回数组的起始指针

for((const不修改迭代值) auto(&) Str:StrArr)     //引用与创建临时对象的迭代
{

}

for(int32 Index = 0;Index<StrArr.Num();Index++)
{

}

//排序,有Sort,HeapSort,StableSort 基于不同原理的排序
StrArr.Sort([](const FString& A,const FString& B){
return A.Len()<B.Len();
})

//Slack元素：数组没有元素时内存上所占的当前未使用的有效预分配元素储存槽
TArray<int32> SlackArray;
//SlackArray.GetSlack()=0
//SlackArray.Num()=0
//SlackArray.Max()=0

SlackArray.Add(1);
//SlackArray.GetSlack()=3;容量不够时，按倍数增长（默认后续约翻倍：4→8→16→32…）
//SlackArray.Num()=1;
//SlackArray.Max()=4

SlackArray.Empty()
//全部清零,释放所有内存

SlackArray.Empty(3)
//给Slack分配数量
//SlackArray.GetSlack()=3;
//SlackArray.Num()=0;
//SlackArray.Max()=3;

SlackArray.Reset(0)//分配Slack空间
//0小于上面的3，就不再分配Slack
//SlackArray.GetSlack()=3;
//SlackArray.Num()=0;
//SlackArray.Max()=3;

SlackArray.Reset(10)
//10大于上面的3，就不再分配Slack
//SlackArray.GetSlack()=10;
//SlackArray.Num()=0;
//SlackArray.Max()=10;

SlackArray.Shrink()
//释放所有Slack，Num=Max

//为了**性能优化**：
//普通 Insert 会先创建临时对象，再移动 / 拷贝进数组。
//`InsertUninitialized + Placement New`直接在数组内存上原地构造对象，省去临时对象拷贝开销。
TArray<FString> UninitStrs;
UninitStrs.Emplace(TEXT("A"));
UninitStrs.Emplace(TEXT("D"));
UninitStrs.InsertUninitialized(1,2);    //分配单纯的内存空间，1号索引，2个单位
new ((void*)(UninitStrs.GetData() +·1)) FString(TEXT("B"));//new了索引1的对象
new ((void*)(UninitStrs.GetData() +·2)) FString(TEXT("C"));

//与上面类似
char* buffer =new char[4];
FXGAddStructInfo* Struct =new (buffer)FXGAddStructInfo;
delete []buffer;
// 普通 `new` 是「向系统申请新内存 + 在新内存上构造对象」两步一起完成；
// 定位 `new` **不申请新内存**，它直接在你指定的 `buffer` 这块已有内存地址上，调用 `FXGAddStructInfo` 的构造函数，原地造出一个对象。
// 最终 `Struct` 指针和 `buffer` 指向的是**同一块内存地址**，只是类型从 `char*` 变成了结构体指针。
//不会触发析构函数

//数组追加内存空间
StrArr.AddZeroed(N)  //数组末尾追加N个单位内存空间
StrArr.InsertZeroed(i,N)   //数组i索引后追加N个单位内存空间

SArr.SetNumUninitialized(3); //设置内存大小
new ((void*)(SArr.GetData() + 1)) S(5,(void*)0x12345678,3.14);//对应索引1new对象
new ((void*) (SArr.GetData() + 2)) S(2,(void*)0x87654321,2.72);
// SArr ==[
// {Int:0, Ptr: nullptr,Flt:0.0f} ,
// {Int:5,Ptr:0x12345678,Flt:3.14f} ,
//{Int:2,Ptr:0x87654321,Flt:2.72f }
//]

SArr.Swap(i,j)   //交换两个索引的位置,但会检查数组越界
StrArr.SwapMemory(i,j)  //只有交换位置功能

StrArr.AddDefaulted(N);  //尾部追加N个无参构造的对象
Arr.AddDefaulted_GetRef();   //直接返回添加对象的引用
//伪代码
//AddDefaulted(1); return GetData()[Num()-1]; 
// 返回新元素引用

```

## 字符串
```cpp
FString s1 = "测试"; // ❌ Windows下乱码！窄字符 
FString s2 = TEXT("测试"); // ✅ 正确，宽字符
//括号中只能弄字面量，不能包变量


```
## 结构体
- 声明结构体与结构体数组
```cpp
.h文件

USTRUCT()
struct FXGEquStructInfo
{
GENERATED_BODY()

public:
FXGEquStructInfo()
{
}

FXGEquStructInfo(int32 InID):ID(InID)
{
}

~FXGEquStructInfo()
{
}

int32 ID=0;

bool operator==(const FXGEquStructInfo& other)const
{
	retrun ID==other.ID?true:false;
}

}

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


# 特殊框架
## 异步蓝图函数
```cpp


```

```cpp

```