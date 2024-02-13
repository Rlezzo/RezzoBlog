---
title: "[Unity] ②第一人称视角控制器——开镜、头部摆动、斜面滑落（学习笔记）"
date: 2022-02-12T21:06:59+08:00
author:     "Rezzo"
categories: [ "Tips" ] 
tags: ["Tutorials", "Unity"]
---
第一部分：[ [Unity] ①第一人称视角控制器——冲刺、跳跃、蹲伏（学习笔记）](https://juejin.cn/post/7056807343916482573)
# 开镜
首先是是否能执行开镜的开关，然后是按键设定。

需要开镜前后的FOV，「defaultFOV」和「zoomFOV」，然后开镜时间「timeToZoom」,「isZooming」如果为True，就表示在开镜中，如果为False则表示在关镜中。

最后需要一个引用，方便中断控制开镜协程。
```csharp
 [Header("Functional Options")]
     [SerializeField] private bool canZoom = true;
 
 [Header("Controls")]
     [SerializeField] private KeyCode zoomKey = KeyCode.Mouse1;
     
 [Header("Zoom Parameters")]
    // 开镜时间
    [SerializeField] private float timeToZoom = 0.35f;
    // 开镜后的FOV
    [SerializeField, Range(30.0f, 120.0f)] private float zoomFOV = 30.0f;
    [SerializeField] private bool isZooming;
    // 正常状态下默认的FOV
    private float defaultFOV;
    // 执行开镜FOV变化的协程，类似前面按C蹲执行的协程
    private Coroutine zoomCoroutine;
```
主要代码：
```csharp
private void Awake()
    {
        // 默认FOV
        defaultFOV = playerCamera.fieldOfView;
    }
void Update()
    {

        GroundCheck();
        if (CanMove)
        {
            HandleMovementInput();
            HandleMouseLook();
            if (canJump)
                HandleJump();
            if (canCrouch)
                HandleCrouch();
            if (canUseHeadbob)
                HandleHeadbob();
            // 加上开镜
            if (canZoom)
                HandleZoom();
            ApplyFinalMovement();
        }
    }
private void HandleZoom()
    {
        // 按住开镜
            // 按下右键
            if (Input.GetKeyDown(zoomKey) || Input.GetKeyUp(zoomKey))
            {
                // 正在放大
                isZooming = Input.GetKeyDown(zoomKey);
                // 如果协程不为空，则表示在开关镜中，停止协程
                if (zoomCoroutine != null)
                {
                    StopCoroutine(zoomCoroutine);
                    zoomCoroutine = null;
                }
                zoomCoroutine = StartCoroutine(ToggleZoom(isZooming));
            }
     }
```
协程：

做法和下蹲的协程几乎一模一样。
```csharp
private IEnumerator ToggleZoom(bool isEnter)
    {
        float targetFOV = isEnter ? zoomFOV : defaultFOV;
        float currentFOV = playerCamera.fieldOfView;
        float timeElapsed = 0f;

        while (timeElapsed < timeToZoom)
        {
            playerCamera.fieldOfView = Mathf.Lerp(currentFOV, targetFOV, timeElapsed / timeToZoom);
            timeElapsed += Time.deltaTime;
            yield return null;
        }

        playerCamera.fieldOfView = targetFOV;
        zoomCoroutine = null;
    }
```
## 修改下蹲代码

这里顺便把前一期，下蹲的代码修改一下：
```csharp
private bool ShouldCrouch => canCrouch && 
(Input.GetKeyDown(holdToCrouchKey) || Input.GetKeyUp(holdToCrouchKey) || Input.GetKeyDown(switchCrouchKey)) 
&& isGrounded;

private void HandleCrouch()
    {
        if (ShouldCrouch)
        {
            if (Input.GetKeyDown(holdToCrouchKey)) isCrouching = true;
            else if(Input.GetKeyUp(holdToCrouchKey)) isCrouching = false;
            else isCrouching = !isCrouching;
            if (crouchCoroutine != null)
            {
                StopCoroutine(crouchCoroutine);
                crouchCoroutine = null;
            }
            StartCoroutine(CrouchStand());
        }
    }
private IEnumerator CrouchStand()
    {
        if (isCrouching && Physics.Raycast(playerCamera.transform.position, transform.up, 1f)) yield break;

        // duringCrouchAnimation = true;

        float timeElapsed = 0;
        float targetHeight = isCrouching ? crouchHeight : standingHeight;
        float currentHeight = characterController.height;
        Vector3 targetCenter = isCrouching ? crouchingCenter : standingCenter;
        Vector3 currentCenter = characterController.center;

        Vector3 targetGroundCheckPosition = isCrouching ? groundCheckCrouchPosition : groundCheckStandingPosition;
        Vector3 currentGroundCheckPosition = groundCheck.localPosition;

        while (timeElapsed < timeToCrouch)
        {
            float chrouchPercentage = timeElapsed / timeToCrouch;
            characterController.height = Mathf.Lerp(currentHeight, targetHeight, chrouchPercentage);
            characterController.center = Vector3.Lerp(currentCenter, targetCenter, chrouchPercentage);

            groundCheck.localPosition = Vector3.Lerp(currentGroundCheckPosition, targetGroundCheckPosition, chrouchPercentage);

            timeElapsed += Time.deltaTime;
            yield return null;
        }

        characterController.height = targetHeight;
        characterController.center = targetCenter;

        groundCheck.localPosition = targetGroundCheckPosition;

        // isCrouching = !isCrouching;
        // duringCrouchAnimation = false;
    }
```

# 头部摆动
```csharp
[Header("Functional Options")]
    [SerializeField] private bool canUseHeadbob = true;
    
[Header("Headbob Parameters")]
    // 行走时，头部上下摆动的频率
    [SerializeField] private float walkBobSpeed = 10f;
    // 行走时，头部上下摆动的幅度
    [SerializeField] private float walkBobAmount = 0.03f;
    [SerializeField] private float sprintBobSpeed = 15f;
    [SerializeField] private float sprintBobAmount = 0.1f;
    [SerializeField] private float crouchBobSpeed = 8f;
    [SerializeField] private float crouchBobAmount = 0.025f;
    // 初始的Y轴位置
    private float defaultYPos = 0;
    // 计时器
    private float headbobTimer;
    
    private void Awake()
    {
        // Headbob，相机初始y轴位置
        defaultYPos = playerCamera.transform.localPosition.y;
    }
    void Update()
    {

        GroundCheck();
        if (CanMove)
        {
            HandleMovementInput();
            HandleMouseLook();
            if (canJump)
                HandleJump();
            if (canCrouch)
                HandleCrouch();
            if (canUseHeadbob)
                HandleHeadbob();
            if (canZoom)
                HandleZoom();
            ApplyFinalMovement();
        }
    }
```
摆动核心代码：

Mathf.Sin(headbobTimer) * (isCrouching ? crouchBobAmount : IsSprinting ? sprintBobAmount : walkBobAmount)

用正弦函数，时间作为横轴，然后时间增加的时候，Y轴在[-1, 1]之间摆动，然后改变摄像机的Y轴坐标，就会出现上下摆动的效果。

Amount就是上下摆动的幅度，Speed就是时间向右前进的速度，速度越快，上下摆动的频率就越快。
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b53850d55b7c47eea05901af6af8e8c3~tplv-k3u1fbpfcp-watermark.image?)
```csharp
    private void HandleHeadbob()
    {
        if (!isGrounded) return;
        // 在前进或者左右移动
        if (Mathf.Abs(moveDirection.x) > 0.1f || Mathf.Abs(moveDirection.z) > 0.1f)
        {
            // 摆动的速度，蹲、冲刺、走都不一样
            // 正弦函数，时间作为x轴，前进的速度，影响频率
            headbobTimer += Time.deltaTime * (isCrouching ? crouchBobSpeed : IsSprinting ? sprintBobSpeed : walkBobSpeed);
            playerCamera.transform.localPosition = new Vector3(
                playerCamera.transform.localPosition.x,
                // 摆动的幅度上下限为1，乘以后面的系数减弱摆动幅度，正负影响方向
                defaultYPos + Mathf.Sin(headbobTimer) * (isCrouching ? crouchBobAmount : IsSprinting ? sprintBobAmount : walkBobAmount),
                playerCamera.transform.localPosition.z
                );
        }
    }
```
# 斜面滑落

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e6e7f2753cf24f27b4282d102036bca8~tplv-k3u1fbpfcp-watermark.image?)

「Character Controller」在遇到超过坡度限制的斜面，会阻止继续前进。


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/91d08407d0324185b15f8ea55821c778~tplv-k3u1fbpfcp-watermark.image?)
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e1895268eaa47c292be86386ce46171~tplv-k3u1fbpfcp-watermark.image?)

但是可以跳上去，然后卡在斜面上。

所以需要实现，人物在斜面上会下滑的功能。

## 实现原理
从人物身上，垂直向下发射一条射线（红色）。
```csharp
Physics.Raycast(transform.position, Vector3.down, out RaycastHit slopeHit, rayLength)
```
「slopeHit」就是射线第一个接触点（紫色）。slopeHit.normal，就可以得到接触点所在的位置，的法线向量。
```csharp
    // 向下的射线，探测到的斜面，的法线向量
    hitPointNormal = slopeHit.normal;
```
那么，红线和蓝线的夹角①，就等于②和③，就是斜面的坡度。
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c73de15f1b4347f7ace84d85d97bf9d1~tplv-k3u1fbpfcp-watermark.image?)

将法线向量，沿Y轴对称，然后让人物沿着红色向量的方向移动，即可完成下滑。
```csharp
moveDirection += new Vector3(hitPointNormal.x, -hitPointNormal.y, hitPointNormal.z) * slopeSpeed;
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/068a576aa7444db2a82906aaef8d0716~tplv-k3u1fbpfcp-watermark.image?)

## 完整代码
```csharp
    [Header("Functional Options")]
        // 在斜面上将会下滑
        [SerializeField] private bool willSlideOnSlopes = true;
    [Header("Movement Parameters")]
        // 下滑速度
        [SerializeField] private float slopeSpeed = 8.0f;
        
    // SLIDING PARAMETERS
    // 向下探测斜面的射线的长度
    [SerializeField] private float rayLength = 2f;
    // 存射线接触点，平面的法线向量
    private Vector3 hitPointNormal;
    // 是否需要处在滑行状态
    private bool IsSliding
    {
        get
        {
            // Debug.DrawRay(transform.position, Vector3.down * rayLength, Color.red);
            // 在地面上，且射线探测到平面
            if (isGrounded && Physics.Raycast(transform.position, Vector3.down, out RaycastHit slopeHit, rayLength))
            {
                // 向下的射线，探测到的斜面，的法线向量
                hitPointNormal = slopeHit.normal;
                // 斜面的斜率大于限制
                return Vector3.Angle(Vector3.up, hitPointNormal) > characterController.slopeLimit;
                
                // 下面的可以用作测试
                // Debug.DrawRay(slopeHit.point, slopeHit.normal, Color.green);
                // Debug.DrawRay(slopeHit.point, new Vector3(hitPointNormal.x, -hitPointNormal.y, hitPointNormal.z), Color.red);
            }
            return false;
        }
    }
```
ApplyFinalMovement() 在其中应用移动。
```csharp
    private void ApplyFinalMovement()
    {
        if (!isGrounded)
        {
            moveDirection.y += gravity * Time.deltaTime;
        }

        if (willSlideOnSlopes && IsSliding)
        {
            // 加上斜面方向的速度矢量
            moveDirection += new Vector3(hitPointNormal.x, -hitPointNormal.y, hitPointNormal.z) * slopeSpeed;
        }
        characterController.Move(moveDirection * Time.deltaTime);
    }
```
## 修改地面检测
之前是用球检测，遇到这种情况，卡在斜面上，会显示不在地面，就改成了胶囊检测。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f13110b1d15e4e12a59c1b7e20ec221c~tplv-k3u1fbpfcp-watermark.image?)
```csharp
private void GroundCheck()
    {
        isGrounded = Physics.CheckCapsule(transform.position, groundCheck.position, characterController.radius, groundLayer);

        if (isGrounded && moveDirection.y < 0.0f)
        {
            moveDirection.y = -2.0f;
        }
    } 
```
下图是「Character Controller」对胶囊的调整，高度指的是整体的高度，半径，是指上下半球的半径。
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c4e109956a854abfa34df75ea4e9f94f~tplv-k3u1fbpfcp-watermark.image?)
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de3445e48a4041b7b6690cc1de71a86d~tplv-k3u1fbpfcp-watermark.image?)

```csharp
Physics.CheckCapsule(A, B, 半径, groundLayer);
```
这个方法的意思是，假如以A，B点为圆心，画出胶囊的上下半球，如图。
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eee50d488ecd486f938a871f39cd1751~tplv-k3u1fbpfcp-watermark.image?)

那么黄色部分都是探测范围，卡边上不显示接地的问题就大大缓解了。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b04f49749bec45d883eeda0f1833b2cc~tplv-k3u1fbpfcp-watermark.image?)

