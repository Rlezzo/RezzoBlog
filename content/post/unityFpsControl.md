---
title: "[Unity] 第一人称视角及移动控制"
date: 2022-01-13T23:56:20+08:00
author:     "Rezzo"
categories: [ "Tips" ] 
tags: ["Tutorials", "Unity"]
---

# RigidBody 实现
## 第一人称鼠标视角控制
目标：

①实现上下看

②实现左右看

③将视角限制在合理的范围（上下不超过90°）

### 新建Player
①先捏个人，「Hierarchy」-> 「Create Empty」 -> 「Rename」 —> "Player"。

②选中「Player」右键 -> 「3D Object」 -> 「Capsule」 -> 「Rename」 —> "Body"。

③同理创建其它部分（可以省略，一个身子其实就够用了）。

④「Nose」 和 「Direction」方便显示“Player”的“Z轴”方向。

⑤将摄像机放进「Head」。

⑥给"Player" 加上 「Rigidbody」组件，并冻结"X"与"Z"轴的旋转。人物就只能以"Y"轴为中心旋转了。
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b2c80d946f3e42939407e296f6a9009b~tplv-k3u1fbpfcp-watermark.image?)
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bdb618416f994811a2aadb2a45717d7c~tplv-k3u1fbpfcp-watermark.image?)

「Direction」 和 「Nose」方便区分人物面朝的方向，也就是"Player"Z轴正方向。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5c5d6d973d1840919f30df9b08d4f32a~tplv-k3u1fbpfcp-watermark.image?)

### 鼠标上下移动，摄像机上下看

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/19f4f5e71cff43f5a9790ea3399cf6e9~tplv-k3u1fbpfcp-watermark.image?)

新建脚本「CameraLook」，添加至"Main Camera"。

添加 mouseSensitivity, xRotation 获得"Mouse Y"鼠标输入，进行摄像机旋转。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5afabe30bc8a468f94c982332c9cc64a~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f03aebe6f0db4c28bdd9e5aa5f934b2b~tplv-k3u1fbpfcp-watermark.image?)
```csharp
public class CameraLook : MonoBehaviour
{
    [Header("Control Settings")]
    public float mouseSensitivity = 100.0f;

    // 记录摄像机沿X轴旋转的变化量
    // 也就是鼠标上下时旋转的角度
    private float xRotation = 0f;

    // Update is called once per frame
    void Update()
    {
        // Mouse input
        // 鼠标在桌面上，Y轴的移动，对应上下
        float mouseY = Input.GetAxis("Mouse Y");
        // Look up/down
        // Δx = Δv * Δt 单位变化量 = 单位变化速度 * 单位时间
        // 负号，则对应鼠标上下等于视角上下，反之鼠标反转
        xRotation -= mouseY * mouseSensitivity * Time.deltaTime;
        // 本地坐标（摄像机以人为参考系）绕x轴旋转
        transform.localRotation = Quaternion.Euler(xRotation, 0, 0);
    }
}
```
这里旋转摄像头的方式不止一种，详见[旋转的实现方式](https://juejin.cn/post/7050784202207264805)。

### 鼠标左右移动，摄像机左右看
这里改动一下脚本的位置。
将「CameraLook」脚本从"Main Camera"中移除。添加到"Player"。

顺便做一下视角限制，将上下最大角度限制为90°。修改或添加的部分如下：
```csharp
public class CameraLook : MonoBehaviour
{
    public Transform cameraTransform;
    void Update()
    {
        xRotation = Mathf.Clamp(xRotation, -90, 90);
        // 本地坐标（摄像机以人为参考系）绕x轴旋转
        cameraTransform.localRotation = Quaternion.Euler(xRotation, 0, 0);
    }
}
```
```csharp
public class CameraLook : MonoBehaviour
{
    void Update()
    {
        // 左右看，用身体旋转实现
        float mouseX = Input.GetAxis("Mouse X");
        // 以自身Y轴为旋转轴旋转
        transform.Rotate(Vector3.up * mouseX * mouseSensitivity * Time.deltaTime);
    }
}
```
整理一下：
```csharp
    public Transform cameraTransform;

    [Header("Control Settings")]
    public float mouseSensitivity = 100.0f;

    private float xRotation = 0f;

    void Update()
    {
        // 鼠标输入
        float mouseY = Input.GetAxis("Mouse Y") * mouseSensitivity * Time.deltaTime;
        float mouseX = Input.GetAxis("Mouse X") * mouseSensitivity * Time.deltaTime;
        // 上下看，以X轴旋转
        xRotation -= mouseY;
        xRotation = Mathf.Clamp(xRotation, -90, 90);
        cameraTransform.localRotation = Quaternion.Euler(xRotation, 0, 0);
        // 左右看，用身体旋转实现
        // 以自身Y轴为旋转轴旋转
        transform.Rotate(Vector3.up * mouseX);
    }
```
## 移动控制
Transform Translate实现：
与物体发生碰撞时，因为使用了刚体组件，可能出现不受控的左右旋转，可以在「Inspector」-> 「Rigidbody」 -> 「Constraints」中，勾选"Freeze Rotation"中的y轴。
```csharp
public class PlayerMovement : MonoBehaviour
{
    
    public float speed = 20.0f;
    // Update is called once per frame
    void Update()
    {
        float horizontal = Input.GetAxis("Horizontal");
        float vertical = Input.GetAxis("Vertical");
        // transform.right获取的是player在世界坐标中的方向
        Vector3 move = transform.right * horizontal + transform.forward * vertical;
        // Space.World如果省略，则默认是Space.Self以自身坐标系为准
        transform.Translate(move * speed * Time.deltaTime, Space.World);
    }
}
```
Rigidbody Addforce实现：
```csharp
void FixedUpdate()
    {
        float horizontal = Input.GetAxis("Horizontal");
        float vertical = Input.GetAxis("Vertical");
        Vector3 move = transform.right * horizontal + transform.forward * vertical;
        // 直接设置刚体的速度矢量
        playerRigidbody.velocity = move * speed;
    }
```
物理运算一般都在FixedUpdate()中实现，可参考[FixedUpdate是在固定的时间间隔执行吗？](https://www.cnblogs.com/murongxiaopifu/p/7683140.html)

FixedUpdate的deltaTime是可以人为设定的一个常数，用于保证物理运算，而不是按人为设定的时间间隔（deltaTime），去执行FixedUpdate()

引用一段上文链接的话「这里还拿Unity引擎来举例子，默认情况下项目的Fixed Timestep的值为0.02s。也就是说物理模拟的频率是50FPS，假设我们的游戏的更新频率是25FPS，那么会发生什么呢？没错，游戏每1次Update时，物理模拟都要推进2次，也就是之前我们看到的在Update之前多次调用了FixedUpdate。那么如果我们的游戏更新频率是100FPS呢？这次就变成了每2次Update调用1次FixedUpdate。」

## 实现跳跃
多次跳跃的问题，见下文「Charactor Controller实现」。
```csharp
float jumpForce = 10.0f;
if (Input.GetButtonDown("Jump"))
    {
        // 玩家按跳跃键，生成一个向上的力
        playerRigidbody.AddForce(jumpForce * Vector3.up, ForceMode.Impulse);
    }
```
# Charactor Controller实现
Rigidbody实现，需要处理很多问题，不是需要大量物理交互的情况下，一般使用Charactor Controller实现。

①实现前后左右移动。

②实现有重力的效果。

③实现跳跃。

## 「Character Controller」组件
在「Player」 -> 「Inspector」 中添加 「Character Controller」组件。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3a25179c4974e1ab7cefe0a840fa8a7~tplv-k3u1fbpfcp-watermark.image?)
```
    // 坡度限制
    Slope Limit: 遇到小于等于这个角度的斜坡，能自动爬上去，否则无法前进
    
    // 台阶偏移量
    Step Offset: 小于等于这个高度的障碍，如台阶的一截，能自动走上去，否则会像撞墙一样停住
    该值不应该大于角色控制器的高度，否则会产生错误。
    
    // 皮肤宽度
    Skin width：两个碰撞体可以穿透彼此且穿透深度最多为皮肤宽度 (Skin Width)。
    较大的皮肤宽度可减少抖动。较小的皮肤宽度可能导致角色卡住。
    合理设置是至少大于 0.01 并且是Radius的10%。
```
未运行时，将皮肤宽度设置为8，此时Radius为0.5。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d4d9afc9409d47559bc9d5fca8f84615~tplv-k3u1fbpfcp-watermark.image?)

运行游戏，但是不进行移动。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c66acfce82884988a456b084319d40e5~tplv-k3u1fbpfcp-watermark.image?)

开始移动，立刻被抬升，如图。

个人理解是，有些游戏中，被卡住的时候会被弹出或者挤出来，比如两个人重叠到一起，而且这两个人之间是有碰撞关系的。然后会被弹开。这里应该是类似的效果，皮肤宽度就是能贴在一起的最小距离，小于这个距离就会被弹开。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6677619d4ef24ef4800709609867d4b4~tplv-k3u1fbpfcp-watermark.image?)

```
    // 最小移动距离
    Min Move Distance: 如果角色移动的距离小于该值，那角色就不会移动。这可以避免颤抖现象。大部分情况下该值被设为0。
    
    // 下面三个都是对胶囊碰撞体的调节，实际动手调节碰撞体的时候就能明白。
    Center
    Radius
    Height
```

## 角色移动
前后左右移动，和上面用transform.Translate() 几乎一模一样，只不过是改用了CharacterController.Move()方法。
```csharp
public class PlayerController : MonoBehaviour
{
    public CharacterController controller;
    public float speed = 5f;
    
    void Start()
    {
        // 得到character组件
        controller = GetComponent<CharacterController>();
    }
    
    void Update()
    {
        // 前进后退的输入
        float horizontal = Input.GetAxis("Horizontal");
        float vertical = Input.GetAxis("Vertical");
        // 移动的方向矢量
        Vector3 move = transform.right * horizontal + transform.forward * vertical;
        // 速度矢量 * 速度大小 * 单位时间 = 单位距离
        // Move()里的就是每帧要移动的距离
        controller.Move(move * speed * Time.deltaTime);
    }
}
```

## 伪重力效果
因为没有使用Rigidbody组件，所以在走阶梯被抬升后，会停留在这个高度。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e1cb4ab281548aebda1a535bb358b3c~tplv-k3u1fbpfcp-watermark.image?)


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/93ab388248514112959c254475564dd6~tplv-k3u1fbpfcp-watermark.image?)


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2062324da1cb4ae6bbd453dad68f0dee~tplv-k3u1fbpfcp-watermark.image?)

下落：
```csharp
public class PlayerController : MonoBehaviour
{
    // 重力加速度
    private float gravity = -9.81f;
    // 玩家速度
    private Vector3 playerVelocity;

    private void PlayerPhysics()
    {
        // 计算下落速度
        // Δv = a * Δt
        playerVelocity.y += gravity * Time.deltaTime;
        // 下落，等于往下移动
        controller.Move(playerVelocity * Time.deltaTime);
    }
}
```
当落到地面的时候，下落速度依然会继续计算，假如从楼上跳下来，腾空的瞬间，下落速度会非常大，人物会直接瞬移到地面，再次被地面挡住。所以需要在地面上时，限制下落速度。

```csharp
public class PlayerController : MonoBehaviour
{
    public bool isGround;
    // 地面检测基准点
    public Transform checkGround;
    // 检测半径
    public float groundCheckRadius;
    // 检测对象
    public LayerMask groundLayer;
    
    private void PlayerPhysics()
    {
        // 计算下落速度
        playerVelocity.y += gravity * Time.deltaTime;
        
        
        // 地面检测
        isGround = Physics.CheckSphere(checkGround.position, groundCheckRadius, groundLayer);
        // 在地面时，下落速度不再增加
        if (isGround && playerVelocity.y < 0)
        {
            // 如果设置为0，从高处走出来时，有一种猫和老鼠里那种滞空的效果
            // 保持一个很小的下落速度比较真实
            playerVelocity.y = -2f;
        }
        
        
        // 下落
        controller.Move(playerVelocity * Time.deltaTime);
    }
    
}
```
添加一个检测点。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4710c64399a5437ab570cf4171aa64be~tplv-k3u1fbpfcp-watermark.image?)

放置于人物脚底。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f1c2f9e86f8c478ab65ae5d999e07c41~tplv-k3u1fbpfcp-watermark.image?)

记得拖拽到「Inspector」。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/931c0d67620d4808aeb9a881c2b1daa1~tplv-k3u1fbpfcp-watermark.image?)

在需要设置的地面或其它建筑物的「Layer」。点击下拉菜单，“Add layer..”，然后进行更改。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/74a8f8f994b64701885aa552c1f9a1f3~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/882673bd2dcb4edabe196baf22bd0bde~tplv-k3u1fbpfcp-watermark.image?)

记得在“player”里，将要检测的对象也选择为“Ground”。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f4ceffbaf02c43cd969caaa4c63ad234~tplv-k3u1fbpfcp-watermark.image?)

## 跳跃
跳跃就直接将人物y轴方向的速度，改为向上的一个速度。

由 h = 1/2*g*t^2 和 v = g*t推导出来，因为这里g是一个负数，所以下面开方的时候再加一个负号。
```csharp
public class PlayerController : MonoBehaviour
{
    public float speed = 5f;
    // 跳跃的高度
    public float jumpHeight;
    
    public int jumpCount = 2;
    
    void Update()
    {
        // 上文地面检测，下落
        PlayerPhysics();

        // 多段跳
        if(Input.GetButtonDown("Jump") && jumpCount > 0)
        {
            playerVelocity.y = Mathf.Sqrt(-gravity * 2f * jumpHeight);
            jumpCount--;
        }
        
        // 只有在地面上才能跳
        if(Input.GetButtonDown("Jump") && isGround)
        {
            playerVelocity.y = Mathf.Sqrt(-gravity * 2f * jumpHeight);
        }
    }
}
```

# 完整代码
```csharp
public class PlayerController : MonoBehaviour
{
    [Header("Gravity")]
    private float gravity = -9.81f;
    private Vector3 playerVelocity;
    [Header("OnGroundCheck")]
    public bool isGround;
    public float groundCheckRadius;
    public Transform checkGround;
    public LayerMask groundLayer;
    [Header("PlayerControl")]
    public CharacterController controller;
    public float speed = 5f;
    public float jumpHeight;
    public int jumpCount = 2;
    
    void Start()
    {
        controller = GetComponent<CharacterController>();
    }

    // Update is called once per frame
    void Update()
    {
        PlayerPhysics();
        float horizontal = Input.GetAxis("Horizontal");
        float vertical = Input.GetAxis("Vertical");

        Vector3 move = transform.right * horizontal + transform.forward * vertical;
        controller.Move(move * speed * Time.deltaTime);
        // 跳跃
        //if(Input.GetButtonDown("Jump") && jumpCount > 0)
        //{
        //    playerVelocity.y = Mathf.Sqrt(-gravity * 2f * jumpHeight);
        //    jumpCount--;
        //}
        if(Input.GetButtonDown("Jump") && isGround)
        {
            playerVelocity.y = Mathf.Sqrt(-gravity * 2f * jumpHeight);
        }
    }

    private void PlayerPhysics()
    {
        // 计算下落速度
        playerVelocity.y += gravity * Time.deltaTime;
        // 地面检测
        isGround = Physics.CheckSphere(checkGround.position, groundCheckRadius, groundLayer);
        // 在地面时，下落速度不再增加
        if (isGround && playerVelocity.y < 0)
        {
            playerVelocity.y = -2f;
            jumpCount = 2;
        }
        // 下落
        controller.Move(playerVelocity * Time.deltaTime);
    }
}
```

