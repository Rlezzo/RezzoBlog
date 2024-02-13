---
title: "[Unity] ③第一人称视角控制器——交互（学习笔记）"
date: 2022-02-19T22:32:11+08:00
author:     "Rezzo"
categories: [ "Tips" ] 
tags: ["Tutorials", "Unity"]
---
前一篇[[Unity] ②第一人称视角控制器——开镜、头部摆动、斜面滑落（学习笔记）](https://rlezzo.github.io/post/unityfpscontroller2/)

想要实现的效果：从屏幕准心，发射一条射线，能获取第一个击中的物品的信息，并且在瞄准这个物品，对物品按交互键，视线离开这个物品时，能进行相应的操作。

思路：
# 可交互物品
## 创建抽象类
```csharp
public abstract class Interactable : MonoBehaviour
{
    public virtual void Awake()
    {
        // 继承Interactable类的，都划分到“Interactable”层
        gameObject.layer = LayerMask.NameToLayer("Interactable");
    }
    // 按下交互键时
    public abstract void OnInteract();
    // 视线瞄准，选中时
    public abstract void OnFocus();
    // 视线脱离后
    public abstract void OnLoseFocus();
}
```
### virtual 和 abstract
[C#中的虚函数virtual](https://www.cnblogs.com/gygg/p/11556005.html)

```
virtual和abstract都是用来修饰父类的，通过覆盖父类的定义，让子类重新定义。

-   1.virtual修饰的方法必须有实现（哪怕是仅仅添加一对大括号),而abstract修饰的方法一定不能实现。
-   2.virtual可以被子类重写，而abstract必须被子类重写。
-   3.如果类成员被abstract修饰，则该类前必须添加abstract，因为只有抽象类才可以有抽象方法。
-   4.无法创建abstract类的实例，只能被继承无法实例化。
```
```
-   1.虚方法必须有实现部分，抽象方法没有提供实现部分，抽象方法是一种强制派生类覆盖的方法，否则派生类将不能被实例化。
-   2.抽象方法只能在抽象类中声明，虚方法不是。如果类包含抽象方法，那么该类也是抽象的，也必须声明类是抽象的。
-   3.抽象方法必须在派生类中重写，这一点和接口类似，虚方法不需要再派生类中重写。
```
简单说，抽象方法是需要子类去实现的。虚方法是已经实现了的，可以被子类覆盖，也可以不覆盖，取决于需求。

抽象方法和虚方法都可以供派生类重写。

个人理解：加了 virtual 和 abstract 的方法或者类，都可以理解为，提供了一个作文标题。

而abstract，只有标题，你要交出一篇作文，必须要自己写完内容，哪怕是个空的语句{}。

而virtual，已经提供了范文内容，你可以直接交上去，也可以自己重新写内容，覆盖掉范文。

而一个检查声明类的函数，如 "A a = new D()" 的 "a.Fun()" 方法，会先检查在 A 的定义中，Fun()方法是否是虚函数，如果不是，直接运行 A 定义的 Fun() 方法，如果是，就看 D 的定义中，是否重写了，如果重写了就用 D 的 Fun() 方法。如果没有重写，就看 D 的父类 C 是否重写了，如果 C 没有就继续往上找，假如都没有重写，最后就运行 A 的 Fun() 方法。

## 添加「Interactable」层
[关于 Layer ](https://rlezzo.github.io/post/layerAndLayerMask/)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c851a2541d242cbb96a8d6acc73ddf0~tplv-k3u1fbpfcp-watermark.image?)
# PlayerController交互功能
```csharp
[Header("Functional Options")]
    [SerializeField] private bool canInteract = true;
[Header("Controls")]
    [SerializeField] private KeyCode interactKey = KeyCode.F;

[Header("Interaction")]
    // 交互射线的起始点
    [SerializeField] private Vector3 interactionRayPoint = default;
    // 长度
    [SerializeField] private float interactionDistance = default;
    // 交互哪一层的对象
    [SerializeField] private LayerMask interactionLayer = default;
    // 获得交互对象的<Interactable> 组件
    private Interactable currentInteractable;
```

```csharp
void Update()
    {
        GroundCheck();
        if (CanMove)
        {
            if (canInteract)
            {
                HandleInteractionCheck();
                HandleInteractionInput();
            }
            ApplyFinalMovement();
        }
    }
```

```csharp
    /// <summary>
    /// 判断指向哪个可交互对象
    /// </summary>
    private void HandleInteractionCheck()
    {
        // playerCamera.ViewportPointToRay(interactionRayPoint)
        // ViewportPointToRay 把你的电脑屏幕/摄像机看到的画面，看做一个坐标系，一个平面
        // 左下角为(0, 0)，右上角为(1, 1)
        // 然后从你眼睛里，比如(0.5, 0.5)，也就是屏幕中心，发射一条射线
        // 距离是interactionDistance
        Debug.DrawRay(playerCamera.ViewportPointToRay(interactionRayPoint).origin, playerCamera.ViewportPointToRay(interactionRayPoint).direction * interactionDistance, Color.blue);
        if (Physics.Raycast(playerCamera.ViewportPointToRay(interactionRayPoint), out RaycastHit hit, interactionDistance, interactionLayer))
        {
            // Debug.DrawRay(hit.transform.position, Vector3.down * 3f, Color.red);
            // 这里用collider，可交互的物品一般都是实体，或者说要求有实体
            // 射线击中可交互物品，且当前没有与其它物品交互，或者从一个可交互物品移动到另一个
            // currentInteractable.GetInstanceID() != hit.collider.gameObject.GetInstanceID()

            // if ((1 << hit.collider.gameObject.layer) == interactionLayer && (currentInteractable == null || currentInteractable.GetInstanceID() != hit.collider.gameObject.GetInstanceID()))
            if (currentInteractable == null || currentInteractable.GetInstanceID() != hit.collider.gameObject.GetInstanceID())
            {
                // 不为空，那就是换物品了，先失焦
                if (currentInteractable != null)
                    currentInteractable.OnLoseFocus();

                // 获取Component<Interactable>， 给currentInteractable变量
                hit.collider.TryGetComponent<Interactable>(out currentInteractable);

                // 如果存在<Interactable>组件
                if (currentInteractable)
                    // 执行聚焦方法
                    currentInteractable.OnFocus();
            }
        }
        // 如果没有击中可交互物品，但是仍绑定了某个可交互物品
        else if (currentInteractable)
        {
            // 执行失焦方法
            currentInteractable.OnLoseFocus();
            currentInteractable = null;
        }
    }
    /// <summary>
    /// 判断交互操作
    /// </summary>
    private void HandleInteractionInput()
    {
        // 一、按下交互键
        // 二、存在可交互对象
        // 击中某个可交互物品
        // Physics.Raycast(playerCamera.ViewportPointToRay(interactionRayPoint), out RaycastHit hit, interactionDistance, interactionLayer)
        if (Input.GetKeyDown(interactKey) && currentInteractable != null && Physics.Raycast(playerCamera.ViewportPointToRay(interactionRayPoint), out RaycastHit hit, interactionDistance, interactionLayer))
        {
            currentInteractable.OnInteract();
        }
    }
```
精简：

首先，从摄像机，也就是你屏幕的(0.5, 0.5)中心位置，发射一条射线，长度为2，检测指定层，这里是「Interactable」层。
```csharp
Physics.Raycast(playerCamera.ViewportPointToRay(interactionRayPoint), 
out RaycastHit hit, interactionDistance, interactionLayer)
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39655197b5d34cfc860549866a440d7d~tplv-k3u1fbpfcp-watermark.image?)
然后判断，当前绑定的可交互对象，是否为射线击中的对象。未绑定，或者指向了其它可交互对象，就执行下面的代码。

```csharp
if (currentInteractable == null || currentInteractable.GetInstanceID() !=
hit.collider.gameObject.GetInstanceID())
```
一般游戏里，都是聚焦可交互物品，然后物品周围一圈被高亮。我这里是指向就变红。
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/796c9240c8cc43f8b878688d24010a11~tplv-k3u1fbpfcp-watermark.image?)
```csharp
// 如果为不为空，那就执行视线脱离前一个物品，前一个物品应该执行的方法。
if (currentInteractable != null)
    currentInteractable.OnLoseFocus();
// 获取当期可交互物品的「Interactable」组件
hit.collider.TryGetComponent<Interactable>(out currentInteractable);
// 绑定成功，执行聚焦该物品时，应该执行的方法
if (currentInteractable)
    currentInteractable.OnFocus();
```
如果没有命中可交互物品，但是仍绑定有可交互物品，执行失焦方法。
```csharp
        else if (currentInteractable)
        {
            currentInteractable.OnLoseFocus();
            currentInteractable = null;
        }
```
完整：
```csharp
private void HandleInteractionCheck()
    {
        if (Physics.Raycast(playerCamera.ViewportPointToRay(interactionRayPoint), 
        out RaycastHit hit, interactionDistance, interactionLayer))
        {
            if (currentInteractable == null || 
            currentInteractable.GetInstanceID() != hit.collider.gameObject.GetInstanceID())
            {
                if (currentInteractable != null)
                    currentInteractable.OnLoseFocus();

                hit.collider.TryGetComponent<Interactable>(out currentInteractable);
                
                if (currentInteractable)
                    currentInteractable.OnFocus();
            }
        }
        else if (currentInteractable)
        {
            currentInteractable.OnLoseFocus();
            currentInteractable = null;
        }
    }
```
执行交互操作：
按下交互键，且有可交互对象，且射线命中可交互物品。
```csharp
Input.GetKeyDown(interactKey) && currentInteractable != null && Physics.Raycast()
```
```csharp
    private void HandleInteractionInput()
    {
        if (Input.GetKeyDown(interactKey) && currentInteractable != null &&
        Physics.Raycast(playerCamera.ViewportPointToRay(interactionRayPoint), 
        out RaycastHit hit, interactionDistance, interactionLayer))
        {
            currentInteractable.OnInteract();
        }
    }
```
# 实现可交互物品

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4e84d36bc03c4c1db1b5e8f01a3c212a~tplv-k3u1fbpfcp-watermark.image?)
创建一个CubeInteractable子类，继承自Interactable抽象类。填上对应的方法。
```csharp
public class CubeInteractable : Interactable
{
    public override void OnFocus()
    {
        // Debug.Log("Looking At: " + gameObject.name);
    }

    public override void OnInteract()
    {
        // Debug.Log("Interacted with: " + gameObject.name);
    }

    public override void OnLoseFocus()
    {
        // Debug.Log("Stopped Looking At: " + gameObject.name);
    }
}
```
然后创建两个CUBE：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/68c7188714b74163b911acc141237bf6~tplv-k3u1fbpfcp-watermark.image?)
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/96d571d4eec24f94a930eddf2a1b8091~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d896ad8e05854d13831b5465cd1055bb~tplv-k3u1fbpfcp-watermark.image?)

一定要有collider，因为射线需要命中实体。
然后就可以进行正常的交互了。
