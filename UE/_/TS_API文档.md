
# 工具
- 函数参数可能为空
```ts
A:$Nullable<UE.PrimitiveComponent>       //表示A这个参数可能为空，可以传nullptr，也可以是空类型
```

- ts定时器
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

```

# 函数与方法
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


# GAS 
## GameplayAbility
- 蒙太奇播放任务
```ts
//启动任务
this.PlayMeleeMontage();


//播放普通攻击蒙太奇  
private PlayMeleeMontage(){  
    const StartSection = UE.KismetMathLibrary.RandomInteger(3).toString();  
    let MeleeMontageTask = UE.AbilityTask_PlayMontageAndWait.CreatePlayMontageAndWaitProxy(  
        this,  
        "",  
        MA_Melee,        1,  
        StartSection    )    MeleeMontageTask.OnCompleted.Add(()=>this.K2_EndAbility());//完整播放完毕  
    MeleeMontageTask.OnInterrupted.Add(()=>this.K2_EndAbility());//被其他动画打断  
    MeleeMontageTask.OnBlendOut.Add(()=>this.K2_EndAbility());//Blend out过渡完成  
    MeleeMontageTask.OnCancelled.Add(()=>this.K2_EndAbility());//任务被取消  
    //这上面四个回调都绑定了结束这个任务，无论哪种情况都会结束任务  
    MeleeMontageTask.ReadyForActivation();  //激活这个任务  
    }
```

- GA.ts中的触发函数
```ts
//GA 触发时自动执行此函数内容
k2_ActivateAbility(){  
    
    
}
```

- 移除GA
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

- 其他Character.ts文件中获取自身属性值
```ts
UE.AbilitySystemBlueprintLibrary.GetFloatAttributeFromAbilitySystemComponent  
(  
    this.AbilitySystemComponent,  //ASC组件
    new UE.GameplayAttribute("HP","/Script/PTGAS.BaseAttributeSet:HP",null),           //属性位置
	this.bSuccess   //引用传递的布尔变量
);
//这一串返回float属性值
```

