---
title: "[Unity] ①第一人称视角控制器——冲刺、跳跃、蹲伏（学习笔记）"
date: 2022-01-25T00:26:06+08:00
author:     "Rezzo"
categories: [ "Tips" ] 
tags: ["Tutorials", "Unity"]
---

在油管上找到一个很详细的Unity FPS Controller [Unity Tutorial](https://www.youtube.com/watch?v=2FTDa14nryI&list=PLfhbBaEcybmgidDH3RX_qzFM0mIxWJa21)，决定跟着这个系列重新做了。
# 第一人称视角移动
按照这个教程，修改了一部分代码。
```csharp
public class PlayerController : MonoBehaviour
{
    // 是否能够移动
    public bool CanMove { get; private set; } = true;
    
    // 人物移动相关的参数
    [Header("Movement Parameters")]
    [SerializeField] private float walkSpeed = 3.0f;
    [SerializeField] private float gravity = -9.81f;
    [SerializeField] private bool isGrounded;
    
    // 相机视角相关参数
    [Header("Look Parameters")]
    // 鼠标X轴速度，Y轴速度
    [SerializeField, Range(1, 100)] private float lookSpeedX = 2.0f;
    [SerializeField, Range(1, 100)] private float lookSpeedY = 2.0f;
    [SerializeField, Range(0, 90)] private float upperLookLimit = 89.0f;
    [SerializeField, Range(0, 90)] private float lowerLookLimit = 89.0f;
    
    // 相机和角色控制器变量
    private Camera playerCamera;
    private CharacterController characterController;

    // 储存玩家移动速度
    private Vector3 moveDirection;
    // 储存"Horizontal"和"Vertical"输入
    private Vector2 moveInput;
    
    // 鼠标向下看，相机X轴的旋转角度
    private float rotationX;
    // 存鼠标输入
    private float mouseY;
    private float mouseX;
    
    private void Awake()
    {
        // Camera作为player的子物体
        playerCamera = GetComponentInChildren<Camera>();
        characterController = GetComponent<CharacterController>();
        // 锁定电脑光标
        Cursor.lockState = CursorLockMode.Locked;
        Cursor.visible = false;
    }
}
```
教程分成了移动输入，鼠标观看，还有最终应用移动
```csharp
void Update()
    {
        GroundCheck();
        if (CanMove)
        {
            HandleMovementInput();
            HandleMouseLook();
            ApplyFinalMovement();

        }
    }
```
处理移动输入
```csharp
    private void HandleMovementInput()
    {
        // 获得WASD的输入 * speed
        moveInput = new Vector2(Input.GetAxis("Horizontal"), Input.GetAxis("Vertical")) * walkSpeed;
        // y轴，垂直速度保存下来不动
        float moveDirectionY = moveDirection.y;
        // 得到三个方向的输入
        moveDirection = transform.right * moveInput.x + transform.forward * moveInput.y;
        moveDirection.y = moveDirectionY;
    }
```
处理鼠标输入
```csharp
private void HandleMouseLook()
    {
        mouseX = Input.GetAxis("Mouse Y") * lookSpeedY * Time.deltaTime;
        mouseY = Input.GetAxis("Mouse X") * lookSpeedX * Time.deltaTime;
        rotationX -= mouseX;
        // 左手握成点赞手势，大拇指指向正方向，手指蜷曲方向为正
        // 所以最小值是往上看，负数
        rotationX = Mathf.Clamp(rotationX, -upperLookLimit, lowerLookLimit);
        playerCamera.transform.localRotation = Quaternion.Euler(rotationX, 0, 0);
        // 貌似是因为涉及到坐标变化，所以只能用乘法？
        transform.rotation *= Quaternion.Euler(0, mouseY, 0);
        // 这个也行
        // transform.Rotate(transform.up * mouseY);
    }
```
处理移动，重力实现详见[前一篇](https://juejin.cn/post/7052718946662744071)。
```csharp
private void ApplyFinalMovement()
    {
        if (!isGrounded)
        {
            moveDirection.y += gravity * Time.deltaTime;
        }
        
        characterController.Move(moveDirection * Time.deltaTime);
    }

    private void GroundCheck()
    {
        isGrounded = Physics.CheckSphere(checkGround.position, groundCheckRadius, groundLayer);
        if (isGrounded && moveDirection.y < 0.0f)
        {
            moveDirection.y = -2.0f;
        }
    }
```
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/570fd5b253484350bd94a64568349e76~tplv-k3u1fbpfcp-watermark.image?)  ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6c785e316a434a4197c1df1ccb4a9268~tplv-k3u1fbpfcp-watermark.image?)
# 冲刺
添加
```csharp
public class PlayerController : MonoBehaviour
{
    public bool CanMove { get; private set; } = true;
    // 可以冲刺，且按了冲刺键，则进入冲刺状态
    private bool IsSprinting => canSprint && Input.GetKey(sprintKey);

    [Header("Functional Options")]
    [SerializeField] private bool canSprint = true;

    [Header("Controls")]
    // 也可以在unity顶部菜单栏Edit -> Project Setting -> Input Manager 添加一个输入
    [SerializeField] private KeyCode sprintKey = KeyCode.LeftShift;

    [Header("Movement Parameters")]
    [SerializeField] private float walkSpeed = 3.0f;
    [SerializeField] private float sprintSpeed = 6.0f;
}
```
float speed = IsSprinting ? sprintSpeed : walkSpeed;
```csharp
private void HandleMovementInput()
    {
        float speed = IsSprinting ? sprintSpeed : walkSpeed;
        
        // 获得WASD的输入 * speed
        moveInput = new Vector2(Input.GetAxis("Horizontal"), Input.GetAxis("Vertical")) * speed;
    }
```
也可以加个Input，然后Input.GetButton("Run");
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2b74050b94284426a617260d46de6986~tplv-k3u1fbpfcp-watermark.image?)
# 跳跃
private bool ShouldJump => canJump && Input.GetKeyDown(jumpKey);
跳跃功能开启，且按下跳跃键，判断应该执行跳跃
```csharp
public class PlayerController : MonoBehaviour
{
    private bool ShouldJump => canJump && Input.GetKeyDown(jumpKey);
    // 功能开关
    [Header("Functional Options")]
    [SerializeField] private bool canJump = true;
    
    [Header("Controls")]
    [SerializeField] private KeyCode jumpKey = KeyCode.Space;
    
    [Header("Jumping Parameters")]
    // 最大跳跃次数
    [SerializeField] private int maxJumpCount = 3;
    // 目前跳跃次数
    [SerializeField] private int jumpCount = 0;
    // 上次跳跃时间
    [SerializeField] private float lastJumpTime;
    // 跳跃次数重置冷却时间
    [SerializeField] private float jumpCoolDownTime = 0.1f;
    // 跳跃高度
    [SerializeField] private float jumpHeight = 5.0f;
    [SerializeField] private float gravity = -9.81f;
    }
```
```csharp
    void Update()
    {
        GroundCheck();
        if (CanMove)
        {
            HandleMovementInput();
            HandleMouseLook();
            // 如果跳跃功能开启
            if (canJump)
            {
                // 执行跳跃
                HandleJump();
            }
            ApplyFinalMovement();
        }
    }
```
因为离地有一小段时间，仍会判断在地面上，所以加一小段时间，防止第一次离地跳跃，不计算使用了跳跃
```csharp
    private void HandleJump()
    {
        
        if (ShouldJump && jumpCount < maxJumpCount)
        {
            moveDirection.y = Mathf.Sqrt(-gravity * 2f * jumpHeight);
            jumpCount++;
            lastJumpTime = Time.time;
        }

        if (Time.time - lastJumpTime > jumpCoolDownTime && isGrounded) jumpCount = 0;
    }
```
# 蹲伏
## 切换下蹲
网上绝大多数的教程，要么直接不教怎么实现蹲，要么就是通过按键，改变「CharacterControllerd」的「Height」，好一点的加个平滑过度，但是很难找到按住下蹲，松开自动站立的教程。这里先写按「C」切换站立和下蹲的状态。
```csharp
public class PlayerController : MonoBehaviour
{
    // 当功能开启，且按下「C」键，且不是已经在进行蹲/起立的状态，且在地面上（这可以根据需求决定）
    private bool ShouldCrouch => canSwitchCrouch && Input.GetKeyDown(switchCrouchKey) 
    && !duringCrouchAnimation && isGrounded;
    
    [Header("Functional Options")]
    // 是否开启切换下蹲功能
    [SerializeField] private bool canSwitchCrouch = false;

    [Header("Controls")]
    // 也可以在unity顶部菜单栏Edit -> Project Setting -> Input Manager 添加一个输入
    [SerializeField] private KeyCode switchCrouchKey = KeyCode.C;
    
    [Header("Movement Parameters")]
    // 蹲下时的移动速度
    [SerializeField] private float crouchSpeed = 1.0f;
    
    [Header("Crouch Parameters")]
    // 蹲下时的高度与站立的高度
    [SerializeField] private float crouchHeight = 1.35f;
    [SerializeField] private float standingHeight = 2f;

    [SerializeField] private float timeToCrouch = 0.25f;
    [SerializeField] private bool isCrouching;
    [SerializeField] private bool duringCrouchAnimation;
```
```csharp
    void Update()
    {
        GroundCheck();
        if (CanMove)
        {
            HandleMovementInput();
            HandleMouseLook();
            if (canJump)
                HandleJump();
                // 如果开启切换下蹲过程，执行下蹲
            if (canSwitchCrouch)
                HandleCrouch();
            ApplyFinalMovement();
        }
    }
```
按下了下蹲，应该下蹲或起立，执行一个协程
```csharp
    private void HandleCrouch()
    {
        if (ShouldCrouch)
            StartCoroutine(CrouchStand());
    }
```
这个协程做了这些事：

①判断如果是在蹲的状态下，那么下面就要执行站立，判断头顶有无障碍物，有障碍物就不执行站立

②根据「isCrouching」判断，如果「isCrouching」为「True」，说明此时是在蹲的状态，那么「targetHeight」则为「standingHeight」站立时的高度

③获取当前的高度，而「timeElapsed」是一个计时器，后面Lerp平滑站立蹲起的时候用

④下面就循环执行改变高度的操作，时间从 0 -> timeToCrouch， 按时间的占比，用Mathf.Lerp算出某时刻的高度

⑤完成高度的改变后，因为会有些误差，所以将高度设定为目标高度

⑥蹲的状态改变，蹲的动画结束
```csharp
private IEnumerator CrouchStand()
    {
        if (isCrouching && Physics.Raycast(playerCamera.transform.position, transform.up, 1f)) yield break;

        duringCrouchAnimation = true;

        float timeElapsed = 0;
        float targetHeight = isCrouching ? standingHeight : crouchHeight;
        float currentHeight = characterController.height;

        while (timeElapsed < timeToCrouch)
        {
            float chrouchPercentage = timeElapsed / timeToCrouch;
            characterController.height = Mathf.Lerp(currentHeight, targetHeight, chrouchPercentage);
            timeElapsed += Time.deltaTime;
            yield return null;
        }

        characterController.height = targetHeight;
        isCrouching = !isCrouching;
        duringCrouchAnimation = false;
    }
```
到此为止，高度可以在设定的「timeToCrouch」时间内平滑匀速过渡。但是摄像头高度不会变化，所以需要改变「Center」的位置，将碰撞体上移。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/49b7e3e5877143e0b901c17cfcea1832~tplv-k3u1fbpfcp-watermark.image?) ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/35eca8512f8a484585dc48e4d1ad14f3~tplv-k3u1fbpfcp-watermark.image?)

```csharp
    [Header("Crouch Parameters")]
    [SerializeField] private float crouchHeight = 1.35f;
    [SerializeField] private float standingHeight = 2f;
    // 下蹲时，碰撞体的中心位置变化
    [SerializeField] private float timeToCrouch = 0.25f;
    [SerializeField] private Vector3 crouchingCenter = Vector3.zero;
    [SerializeField] private Vector3 standingCenter = Vector3.zero;
    [SerializeField] private bool isCrouching;
    [SerializeField] private bool duringCrouchAnimation;
    // 地面检测物体和摄像头位置变化
    // changedLength = (standingHeight - crouchHeight) / 2
    [SerializeField] private float changedDistanceY;
```
这个crouchingCenter高度，可以实现设定好，我这里在Start()里计算。
```csharp
void Start()
    {
        changedDistanceY = (standingHeight - crouchHeight) / 2f;
        crouchingCenter.Set(standingCenter.x, standingCenter.y + changedDistanceY, standingCenter.z);
changedDistanceY + crouchingCenter.y, 0);
    }
```
协程里，也和height一样的操作。
```csharp
private IEnumerator CrouchStand()
    {
        if (isCrouching && Physics.Raycast(playerCamera.transform.position, transform.up, 1f)) yield break;

        duringCrouchAnimation = true;

        float timeElapsed = 0;
        float targetHeight = isCrouching ? standingHeight : crouchHeight;
        float currentHeight = characterController.height;
        
        Vector3 targetCenter = isCrouching ? standingCenter : crouchingCenter;
        Vector3 currentCenter = characterController.center;

        while (timeElapsed < timeToCrouch)
        {
            float chrouchPercentage = timeElapsed / timeToCrouch;
            characterController.height = Mathf.Lerp(currentHeight, targetHeight, chrouchPercentage);
            characterController.center = Vector3.Lerp(currentCenter, targetCenter, chrouchPercentage);

            timeElapsed += Time.deltaTime;
            yield return null;
        }

        characterController.height = targetHeight;
        characterController.center = targetCenter;

        isCrouching = !isCrouching;
        duringCrouchAnimation = false;
    }
```
这里测试的时候，如果设置成center下移，可能会在站起时穿墙。

groundCheck的位置跟上移，一样的操作。
```csharp
    [SerializeField] private Vector3 groundCheckStandingPosition;
    [SerializeField] private Vector3 groundCheckCrouchPosition;
       void Start()
    {
        // 蹲相关数据
        changedDistanceY = (standingHeight - crouchHeight) / 2f;
        crouchingCenter.Set(standingCenter.x, standingCenter.y + changedDistanceY, standingCenter.z);
        groundCheckStandingPosition = new Vector3(0, groundCheck.localPosition.y, 0);
        groundCheckCrouchPosition = new Vector3(0, groundCheck.localPosition.y + changedDistanceY + crouchingCenter.y, 0);
    }
        private IEnumerator CrouchStand()
    {
        Vector3 targetGroundCheckPosition = isCrouching ? groundCheckStandingPosition : groundCheckCrouchPosition;
        Vector3 currentGroundCheckPosition = groundCheck.localPosition;

        while (timeElapsed < timeToCrouch)
        {
            groundCheck.localPosition = Vector3.Lerp(currentGroundCheckPosition, targetGroundCheckPosition, chrouchPercentage);

            timeElapsed += Time.deltaTime;
            yield return null;
        }
        groundCheck.localPosition = targetGroundCheckPosition;

        isCrouching = !isCrouching;
        duringCrouchAnimation = false;
    }
```
## 按住「LeftCtrl」下蹲

第二期有更好的解决方法:[②第一人称视角控制器——开镜、头部摆动、斜面滑落](https://juejin.cn/post/7063807920944709640)

这个是真找不到好的教程，感觉多少都有点问题，下面是我自己写的，算是没有太大问题吧。
```csharp
public class PlayerController : MonoBehaviour
{
    // 功能开关
    [Header("Functional Options")]
    [SerializeField] private bool canSwitchCrouch = false;
    // 按住下蹲开关
    [SerializeField] private bool canHoldToCrouch = true;
    [Header("Controls")]
    // 也可以在unity顶部菜单栏Edit -> Project Setting -> Input Manager 添加一个输入
    [SerializeField] private KeyCode holdToCrouchKey = KeyCode.LeftControl;
    [SerializeField] private KeyCode switchCrouchKey = KeyCode.C;
    
    [Header("Crouch Parameters")]
    // 下蹲速度
    [SerializeField, Range(50f, 100f)]  private float crouchStandSpeed = 50f;
    // 蹲姿锁
    [SerializeField] private bool crouchLock;
}   
```
我分成了两种开关，一种是上面，按C切换蹲和站立的。
一种是按住下蹲的，也可以同时实现蹲和站立。
```csharp
    void Update()
    {
        GroundCheck();
        if (CanMove)
        {
            HandleMovementInput();
            HandleMouseLook();
            if (canJump)
                HandleJump();
            if (canSwitchCrouch || canHoldToCrouch)
                HandleCrouch();
            ApplyFinalMovement();
        }
    }
```
① 如果在开始蹲姿锁的情况下，按下了「C」，切换蹲姿，则开关蹲姿锁

② 如果蹲姿锁开启，但是按下了「leftcontrol」，则关闭蹲姿锁

③ isCrouching，下蹲状态，由是否按住下蹲，或者 开启了蹲姿锁决定

```csharp
private void HandleCrouch()
    {
        // 如果开启按住蹲下，优先按住蹲下的功能
        if (canHoldToCrouch)
        {
            // 如果按下蹲姿切换，则锁开关切换
            if (Input.GetKeyDown(switchCrouchKey)) crouchLock = !crouchLock;
            // 如果在锁定状态下，按下了保持蹲下键，则锁为关闭状态
            if (crouchLock && Input.GetKeyDown(holdToCrouchKey)) crouchLock = false;
            // 按住保持蹲下键，或者锁定为蹲下，则显示正在蹲伏状态
            isCrouching = Input.GetKey(holdToCrouchKey) || crouchLock;
            // 调整身高
            AdjustHeight();
        }

        // 否则，通过协程进行蹲起
        else if (ShouldCrouch)
            StartCoroutine(CrouchStand());

    }
```
在update里，实时调整身高。

① 如果目标高度已经是当期高度，则直接返回。

② 如果当前身高和目标身高误差很小，则直接设定为目标身高

③ 如果是其它情况，则需要调整身高

因为是在update里实现，所以计时器不是那么好用，并且还有蹲一半松开「leftcontrol」等情况。

我尝试了计时器，或者Mathf.SmoothDamp()等方法，都不尽如人意。

第三个参赛的时刻，使用固定参数或者计时器，或者身高比，都有卡顿或者其他麻烦的情况。

测试下来，目前设定一个蹲起速度，然后乘上Time.deltaTime还算比较良好。

Mathf.Lerp(currentHeight, targetHeight, crouchStandSpeed * Time.deltaTime);

想知道有没有更成熟的实现方法，网上找遍了是在是找不到。

关于[Lerp和SmoothDamp](https://www.jianshu.com/p/8a5341c6d5a6)相关细节可以看看这个。

简单来说，Lerp就是Mathf.Lerp(a, b, t(0.5)) 假设t取0.5，那么在这一帧，就取(b - a)这段，然后乘以0.5，相当于截取一半，然后新的坐标位置就是 a + 这一半长度，然后，如果拿a + 这一半长度的坐标，作为新的起始点，就会继续取一半，相当于最初的3/4的位置。所以到最后会越来越接近b，但是接近的越来越慢。

但是取时间的话，就能实现协程里，匀速的变化，而不是这样的减速效果。

而SmoothDamp，有点类似汽车加速启动，然后减速，最终停止那种平滑感。
```csharp
private void AdjustHeight()
    {
        float targetHeight = isCrouching ? crouchHeight : standingHeight;
        float currentHeight = characterController.height;

        // 如果已经一致，返回
        if (targetHeight == currentHeight) return;


        Vector3 targetCenter = isCrouching ? crouchingCenter : standingCenter;
        Vector3 currentCenter = characterController.center;

        Vector3 targetGroundCheckPosition = isCrouching ? groundCheckCrouchPosition : groundCheckStandingPosition;
        Vector3 currentGroundCheckPosition = groundCheck.localPosition;

        if (Mathf.Abs(targetHeight - currentHeight) < 0.01f)
        {
            characterController.height = targetHeight;
            characterController.center = targetCenter;
            groundCheck.localPosition = targetGroundCheckPosition;
        }
        else
        {
            // 蹲 -> 站立 过程，如果头上有阻挡，则不能改变身高
            if (Physics.Raycast(playerCamera.transform.position, transform.up, 1f) && !isCrouching) return;

            characterController.height = Mathf.Lerp(currentHeight, targetHeight, crouchStandSpeed * Time.deltaTime);
            characterController.center = Vector3.Lerp(currentCenter, targetCenter, crouchStandSpeed * Time.deltaTime);
            groundCheck.localPosition = Vector3.Lerp(currentGroundCheckPosition, targetGroundCheckPosition, crouchStandSpeed * Time.deltaTime);
        }
    }
```
还有蹲下时的移动速度，在这里改动。
```csharp
private void HandleMovementInput()
    {
        float speed = isCrouching ? crouchSpeed : IsSprinting ? sprintSpeed : walkSpeed;
        // Debug.Log("speed: " + speed);
        // 获得WASD的输入 * speed
        moveInput = new Vector2(Input.GetAxis("Horizontal"), Input.GetAxis("Vertical")) * speed;
        // y轴，垂直速度保存下来不动
        float moveDirectionY = moveDirection.y;
        // 得到三个方向的输入
        moveDirection = transform.right * moveInput.x + transform.forward * moveInput.y;
        moveDirection.y = moveDirectionY;

    }
```
