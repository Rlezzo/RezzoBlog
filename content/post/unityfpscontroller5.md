---
title: "[Unity] ⑤第一人称视角控制器——耐力值系统、门的交互（学习笔记）"
date: 2022-02-23T00:03:52+08:00
author:     "Rezzo"
categories: [ "Tips" ] 
tags: ["Tutorials", ""]

---

[[Unity] ④第一人称视角控制器——脚步声、血量系统（学习笔记）](https://rlezzo.github.io/post/unityfpscontroller4/)
# 耐力值系统
| 需 求 | 功 能 |
| :---: | :---: |
| 耐力值系统 开/关| bool useStamina |
|耐力条|float maxStamina（最大耐力值）float currentStamina（当前耐力值）|
|每秒消耗耐力|float staminaMultiplier = 5f|
|开始回复耐力前等待时间|float timeBeforeStaminaRengeStarts = 1.5f|
|单次回复耐力值|float staminaValueIncrement = 2f|
|单次回复耐力后等待时间|float staminaTimeIncrement = 0.1f|
|回复耐力协程|Coroutine regeneratingStamina|
|在耐力值改变时 UI Text同步改变|public static Action<float> OnStaminaChange|

冲刺 -> 消耗耐力值 -> 停止冲刺 -> （等待 timeBeforeStaminaRengeStarts 秒 -> 每 staminaTimeIncrement 秒回复 staminaValueIncrement 点耐力)

（）表示回复耐力的协程做的事。

（回复耐力）时 -> 冲刺 -> 停止协程 -> 停止冲刺或耐力值归零 -> （回复耐力）

和血量系统做法几乎一模一样。
```csharp
[Header("Stamina Paramenters")]
    // 最大耐力
    [SerializeField] private float maxStamina = 100f;
    // 每秒消耗多少耐力
    [SerializeField] private float staminaMultiplier = 5f;
    // 耐力回复前，等待时间
    [SerializeField] private float timeBeforeStaminaRengeStarts = 1.5f;
    // 每次单位耐力回复量
    [SerializeField] private float staminaValueIncrement = 2f;
    // 单位耐力回复间隔时间
    [SerializeField] private float staminaTimeIncrement = 0.1f;
    // 当前耐力
    private float currentStamina;
    private Coroutine regeneratingStamina;
    // 耐力变化时，调用
    public static Action<float> OnStaminaChange;
```
```csharp
void Update()
    {
        if (CanMove)
        {
        //...
            if (useStamina)
                HandleStamina();
            ApplyFinalMovement();
        }
    }
private void HandleStamina()
    {
        // 冲刺状态，且有移动输入，处理耐力
        if (IsSprinting && moveInput != Vector2.zero) {}
        
        // 耐力值不满，且没有冲刺，且没有耐力回复协程在进行
        if (currentStamina < maxStamina && !IsSprinting && regeneratingStamina == null)
            // 开启协程
            regeneratingStamina = StartCoroutine(RegenerateStamina());
    }
```
一、冲刺时，先检查是否有耐力回复协程
```csharp
            // 如果耐力回复协程开启，中断
            if (regeneratingStamina != null)
            {
                StopCoroutine(regeneratingStamina);
                regeneratingStamina = null;
            }
```
二、如果没有，那就正常消耗耐力值，并将改变的耐力值作为参数传给UI
```csharp
            // 每秒消耗耐力
            currentStamina -= staminaMultiplier * Time.deltaTime;
            if (currentStamina < 0f)
                currentStamina = 0f;
            // 之后在UI的脚本里添加委托
            OnStaminaChange?.Invoke(currentStamina);
```
三、耐力值用完的时候，关闭冲刺功能
```csharp
            // 耐力值归零，禁止使用冲刺
            if (currentStamina <= 0f)
                canSprint = false;
```
完整：
```csharp
   private void HandleStamina()
    {
        // 冲刺状态，且有移动输入，处理耐力
        if (IsSprinting && moveInput != Vector2.zero)
        {
            // 如果耐力回复协程开启，中断
            if (regeneratingStamina != null)
            {
                StopCoroutine(regeneratingStamina);
                regeneratingStamina = null;
            }

            currentStamina -= staminaMultiplier * Time.deltaTime;
            if (currentStamina < 0f)
                currentStamina = 0f;
            OnStaminaChange?.Invoke(currentStamina);

            // 耐力值归零，禁止使用冲刺
            if (currentStamina <= 0f)
                canSprint = false;
        }
        // 耐力值不满，且没有冲刺，且耐力回复未开启
        if (currentStamina < maxStamina && !IsSprinting && regeneratingStamina == null)
            regeneratingStamina = StartCoroutine(RegenerateStamina());
    }
```
协程：
```csharp
private IEnumerator RegenerateStamina()
    {
        yield return new WaitForSeconds(timeBeforeStaminaRengeStarts);

        WaitForSeconds timeToWait = new WaitForSeconds(staminaTimeIncrement);
        while (currentStamina < maxStamina)
        {
            // 大于0，可以使用冲刺
            if (currentStamina > 0f)
                canSprint = true;

            currentStamina += staminaValueIncrement;

            if (currentStamina > maxStamina)
                currentStamina = maxStamina;
            // 改变的耐力值
            OnStaminaChange?.Invoke(currentStamina);
            yield return timeToWait;
        }
        // 耐力回复完毕，引用置空
        regeneratingStamina = null;
    }
```
# 门的交互
门的交互，主要是动画机（Animator）的相关问题。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7eef2f5aad34454b98cdd05e2e965fd9~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f506528a9245412ab5c76dfe1d1cf49e~tplv-k3u1fbpfcp-watermark.image?)

可以在商店里找门的资源，也可以自己在Blender之类的软件里做出来导入。

有个注意的点：

Pivot Point就是物体Transform.position所在的点，在这个模式下，箭头表示的是“物体的原点在世界坐标系中的坐标”，可以在模型制作软件中自己设定。

而Center模式下，箭头表示的是“包围物体的最小包围盒(AABB)的中心点”，是unity算出来的模型中心位置。
[![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d82f03ac28444dca84a264a159a47808~tplv-k3u1fbpfcp-watermark.image?)](https://blog.csdn.net/LLLLL__/article/details/88223649)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed6b19f86e8949eb80191e40344ee106~tplv-k3u1fbpfcp-watermark.image?)

Center模式：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/60c374554e9b409082be64d5ef2c3918~tplv-k3u1fbpfcp-watermark.image?)

Pivot模式：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ce58712ffc8437eaed2dc51507427c6~tplv-k3u1fbpfcp-watermark.image?)

门开关的动画，要在Pivot模式下制作。

## 动画机动画制作
在「Project」 -> 右键 -> 「Create」 -> 「Animator Controller」：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa9a7590c6564ec2be2161ef089f86a8~tplv-k3u1fbpfcp-watermark.image?)
并将其拖动到物体的「Inspector」。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0acb3d206bb2421a850d4f3b8f8907c7~tplv-k3u1fbpfcp-watermark.image?)

然后在「Animation」工作区，Create 一个 Animation Clip，选择储存位置。
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/377e4081cab74ab3b8bddeaee5cc7f8b~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/490384c8e5874168a7379adc82223796~tplv-k3u1fbpfcp-watermark.image?)

新建的「Animation Clip」命名为 "OpenIn" ，表示从门内侧开启的动画。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b46b3d01028347eca1f4842e1401f541~tplv-k3u1fbpfcp-watermark.image?)开启录制，
点击Add event，在第0帧，添加一个event。
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7105c9c1e21f4436a69a9683045b5cb1~tplv-k3u1fbpfcp-watermark.image?)

在第60帧，也就是动画最后一帧，设置为门完全开启的状态：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7cfdafc0e88c472883afeafc7bfdabc3~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e9260c36d374fa18b1a9106f86cb428~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d65fd42320f549bba5cb709e8067bb54~tplv-k3u1fbpfcp-watermark.image?)
### 又一个要注意的点：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/775eb81640ec4901b1c4b8941cbc2768~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4b4c097c5e234d6eabe66e0dd5ca0616~tplv-k3u1fbpfcp-watermark.image?)

假如，我录制了一段移动的动画。

并且我把这个动画，复用在其他物体上，比如下面那个方块。当我播放下面那个方块的动画时，会发生什么?

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1fffe99c0edd453caebb86a1678c03ae~tplv-k3u1fbpfcp-watermark.image?)

没错，瞬移到上面的位置，播放了一段完全相同的动画。


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f1f034922e9844c0a61fb1fc41acd52f~tplv-k3u1fbpfcp-watermark.image?)

所以，需要创建一个父物体，将门作为他的子物体，然后录制本地坐标的动画，如果录制的时候，是相对于世界坐标，当一个动画应用到多个物体上的时候，就会出问题。

---

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7d04c2f8120f40cda8413ddfb968d8ab~tplv-k3u1fbpfcp-watermark.image?)

录制五种状态的动画：

动画的循环关掉，开关门动画不需要循环。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/298e273bc5bb4be6a712dfa0d80650d8~tplv-k3u1fbpfcp-watermark.image?)

从内部面朝门时的开关动画。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/04f8e8a56b974735953407361d83dc9d~tplv-k3u1fbpfcp-watermark.image?)
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e1f52e2dd784e058a898b29aacd9947~tplv-k3u1fbpfcp-watermark.image?)

从外部面朝门时的开关动画。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97d1f342600b4005a012325ab6cf214b~tplv-k3u1fbpfcp-watermark.image?)
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f13e3d08804a4390ad6375bb84a8dbde~tplv-k3u1fbpfcp-watermark.image?)

默认关闭状态的动画，什么都不用录，状态是正常关闭状态就行。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c6e18fd2a49d4f67b68a1cc81d27eb46~tplv-k3u1fbpfcp-watermark.image?)

## 门的交互代码
|变量|含义|
|:--:|:--:|
|bool isOpen|门的开关状态|
|Animator animator|门的动画机|

```csharp
public class Door : Interactable
{
    private bool isOpen = false;
    private bool canBeInteractedWith = true;
    private Animator animator;

    public override void OnFocus()
    {
        animator = GetComponent<Animator>();
    }

    public override void OnInteract()
    {
        if (canBeInteractedWith)
        {
            isOpen = !isOpen;

            // 门面朝方向的世界坐标
            Vector3 doorTransformDirection = transform.TransformDirection(Vector3.down);
            // 玩家面朝方向
            Vector3 playerTransformDirection = 
            PlayerController.instance.transform.TransformDirection(Vector3.forward);
            
            // 向量点积，cosθ 为正，方向基本相同在(0, 90)，反之相反
            float dot = Vector3.Dot(doorTransformDirection, playerTransformDirection);

            // 动画设置
            animator.SetFloat("dot", dot);
            animator.SetBool("isOpen", isOpen);

        }
    }
}
```
动画机中添加两个变量：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1dcb58917483464e8a9b6b1ce3a44562~tplv-k3u1fbpfcp-watermark.image?)

dot: 是两个向量的点集，值为正，则两个向量方向基本相同，在(0, 90)度间，为0，向量垂直，为负，向量方向反向，在(90, 180)。

doorTransformDirection 就是将门本地正方向，转换为世界坐标。playerTransformDirection 将玩家的正方向，转换为世界坐标，计算在世界坐标中的方向是否一致。

在「Player Controller」 脚本中，将自己的实例创建一个公共静态变量。
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/763ae38f141b41a6ace601ad38fe5ffd~tplv-k3u1fbpfcp-watermark.image?)
## 动画状态机
从默认关闭状态到开启：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/40f18406e15e4159b0b1f7052969cbe8~tplv-k3u1fbpfcp-watermark.image?)

Reset设置，取消勾选 退出时间，添加两个转换条件：

当门被玩家变为 isOpen = True 开启状态，且当 dot > 0（玩家方向和门方向基本一致），判断为玩家在门内，所以向外开，播放「OpenIn」，从内向外打开的动画。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/afddc683a51a44d8871b25eda70d949b~tplv-k3u1fbpfcp-watermark.image?)

而从内打开的状态，回到对应的关闭状态，条件只需要是 false 就够了，不用判断方向，因为已经打开了，关闭动画唯一确定。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/61b7648ffd844b6e988e88bb74501734~tplv-k3u1fbpfcp-watermark.image?)
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0b35ee2afe0f4d3dacce9d47f4c77348~tplv-k3u1fbpfcp-watermark.image?)

从关闭动画，到默认关闭状态，不需要设置，默认即可。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/689cac5012ce4100bb4aa5172a72d53f~tplv-k3u1fbpfcp-watermark.image?)
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39fa2ecac27345088e80b3ecf8d5a010~tplv-k3u1fbpfcp-watermark.image?)

另一边，从外开关门的动画同理，只是dot的判断改为 dot < 0。

## 自动关门
通过添加一个协程，可以完成自动关门的操作。

玩家与门交互后，开启一个协程，每隔一段时间，检查玩家和门的距离。
```csharp
Vector3.Distance(transform.position,
            PlayerController.instance.transform.position) > 3f
```
```csharp
public class Door : Interactable
{
    private bool isOpen = false;
    // 动画过程中，禁止交互
    private bool canBeInteractedWith = true;
    private Animator animator;
    private WaitForSeconds detectionInterval = new WaitForSeconds(3);

    public override void OnFocus()
    {
        animator = GetComponent<Animator>();
    }

    public override void OnInteract()
    {
        if (canBeInteractedWith)
        {
            isOpen = !isOpen;
            Vector3 doorTransformDirection = transform.TransformDirection(Vector3.down);
            Vector3 playerTransformDirection = PlayerController.instance.transform.TransformDirection(Vector3.forward);
            float dot = Vector3.Dot(doorTransformDirection, playerTransformDirection);
            animator.SetFloat("dot", dot);
            animator.SetBool("isOpen", isOpen);
        //
            StartCoroutine(AutoClose());
        //
        }
    }
    private IEnumerator AutoClose()
    {
        // Debug.Log("开启协程");
        while (isOpen)
        {
            yield return detectionInterval;

            if(Vector3.Distance(transform.position,
            PlayerController.instance.transform.position) > 3f)
            {
                isOpen = false;
                animator.SetFloat("dot", 0f);
                animator.SetBool("isOpen", isOpen);
            }
        }
        // Debug.Log("退出协程");
    }

    public override void OnLoseFocus()
    {

    }
    private void Animator_LockInteraction()
    {
        // Debug.Log("锁定交互");
        canBeInteractedWith = false;
    }
    private void Animator_UnlockInteraction()
    {
        // Debug.Log("开启交互");
        canBeInteractedWith = true;
    }
}
```
记得前面添加的事件吗？

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/487ee81712694ffe9ab083dc10c220d3~tplv-k3u1fbpfcp-watermark.image?)
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/99283ae4e7cf48018e571dfcb572341b~tplv-k3u1fbpfcp-watermark.image?)
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2c1103a978914e25860294a7cf17ce0c~tplv-k3u1fbpfcp-watermark.image?)

通过播放动画时，执行事件，来禁止玩家在动画过程中再次与门交互。

又因为协程的循环条件是(isOpen = true)，所以，在玩家主动关门后，协程就会直接运行完毕。玩家反复开关门不会运行超过2个以上的协程。
