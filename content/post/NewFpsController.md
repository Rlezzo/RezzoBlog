---
title: "Let's Make a First Person Game in Unity 学习笔记(一）"
date: 2023-04-19T02:26:50+08:00
author:     "Rezzo"
categories: [ "Tips" ] 
tags: ["Tutorials", "Unity3d"]
---
[原视频地址](https://www.youtube.com/watch?v=rJqP5EesxLk&list=PLGUw8UNswJEOv8c5ZcoHarbON6mIEUFBC)

（一）利用unity的input system实现基础的FPS人物控制

# 安装最新的input system

如图｢Window｣->｢Package Manager｣中找到并下载安装。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/171369be68654d5cb00bd629f654ca96~tplv-k3u1fbpfcp-watermark.image?)

# 创建player

创建一个胶囊体，并将相机作为其子物体，将胶囊体自带的collider替换为｢Character Controller｣。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8cc10dcec9d4a5ea81269a46937ec26~tplv-k3u1fbpfcp-watermark.image?)
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/005b052d55f1492db0e1c24898e3bdfa~tplv-k3u1fbpfcp-watermark.image?)

# 创建Input Actions，完成输入映射

在文件夹中，右键｢create｣->Input Actions，创建一个Input Action，Csharp脚本是之后自动生成的。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/40af702c18eb4581a752e77c5155bef1~tplv-k3u1fbpfcp-watermark.image?)
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/973e6853b0bf45b39ac8c821feba77ad~tplv-k3u1fbpfcp-watermark.image?)

如图所示进行按键的绑定：
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1b10eb44e6e4724ad8318c334a2d126~tplv-k3u1fbpfcp-watermark.image?)

## Movement

WASD移动，Action Type是Value类型，Control Type是Vector 2，也就是x和y轴的输入。当按ad的时候，人物左右移动，输入的就是x轴数据，人物也在对应的x轴左右移动。Value值[-1，1]。WASD和LeftStick并列，键盘和手柄左摇杆都能进行相同的操作。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b306994a5fb3463e9651fafe00a6e383~tplv-k3u1fbpfcp-watermark.image?)

## Jump键

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aa351e92f4134342a27cfafb973e3d8d~tplv-k3u1fbpfcp-watermark.image?)

## 鼠标第一人称视角查看

也是x,y轴的移动，所以输入也是vector 2。
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/61cf8442e2a642c8b10b57b4727769b0~tplv-k3u1fbpfcp-watermark.image?)

## 最后保存

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c45aeedcdaad4f118562224f0c78938f~tplv-k3u1fbpfcp-watermark.image?)

# 有关移动的脚本

## 实现player移动的脚本

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerMotor : MonoBehaviour
{
    // 角色控制器组件
    private CharacterController controller;
    // 角色速度向量
    private Vector3 playerVelocity;
    // 人物移动速度设定
    private float speed = 5f;

    private bool isGround;

    public float gravity = -9.8f;
    public float jumpHeight = 2f;
    // Start is called before the first frame update
    void Start()
    {
        controller = GetComponent<CharacterController>();
    }

    // Update is called once per frame
    void Update()
    {
        isGround = controller.isGrounded;
    }

    // 接收inputManager脚本传递的input数据并实现效果
    public void ProcessMove(Vector2 input)
    {
        // Input储存为移动向量
        Vector3 moveDirection = Vector3.zero;
        moveDirection.x = input.x;
        moveDirection.z = input.y;
        controller.Move(transform.TransformDirection(moveDirection) * speed * Time.deltaTime);
        //重力模拟和接地判定
        playerVelocity.y += gravity * Time.deltaTime;
        if(isGround && playerVelocity.y < 0)
        {
            playerVelocity.y = -2f;
        }
        controller.Move(playerVelocity * Time.deltaTime);
    }
// 跳跃
    public void Jump()
    {
        if (isGround)
        {
            playerVelocity.y = Mathf.Sqrt(jumpHeight * -3.0f * gravity);
        }
    }
}
```

## 处理移动的输入

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4a5c5577a8e54595b530eb246f97df3c~tplv-k3u1fbpfcp-watermark.image?)
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/945a1a02da354ff0b0b94f778fd22b3e~tplv-k3u1fbpfcp-watermark.image?)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.InputSystem;  //_______________________________命名空间导入
public class InputManager : MonoBehaviour
{
    // 对应的变量
    private PlayerInput playerInput; 
    private PlayerInput.OnFootActions onFoot; 
    private PlayerMotor playerMotor;
    void Awake() 
    {
        // 你设定的playerInput Actions
        playerInput = new PlayerInput(); 
        // 你设定的playerInput Actions 中的onFoot 
        onFoot = playerInput.OnFoot; 
        // 处理移动的脚本
        playerMotor = GetComponent<PlayerMotor>();
        // 处理jump交互，performed可以当做Input.KeyDown()
        // ctx（上下文），记录你按下跳跃键，unity检测到你按下、持续按住、松开跳跃键的一系列信息
        // 按下跳跃键的时候，调用跳跃方法
        onFoot.Jump.performed += ctx => playerMotor.Jump();
    }
    // 启/禁用脚本的操作
    private void OnEnable()
    {
        onFoot.Enable();
    }
    private void OnDisable()
    {
        onFoot.Disable();
    }
    
    void FixedUpdate()
    {
    //用fixedupdate处理移动，不受帧数影响
        playerMotor.ProcessMove(onFoot.Movement.ReadValue<Vector2>());
    }

}
```

## 处理查看操作

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerLook : MonoBehaviour
{
    public Camera mainCam;
    private float xRotation = 0f;
    public float xSensitivity = 30f;
    public float ySensitivity = 30f;

    public void ProcessLook(Vector2 input)
    {
        float mouseX = input.x;
        float mouseY = input.y;
        // 计算上下看的角度
        xRotation -= (mouseY * Time.deltaTime) * ySensitivity;
        xRotation = Mathf.Clamp(xRotation, -90f, 90f);
        // camera应用旋转
        mainCam.transform.localRotation = Quaternion.Euler(xRotation, 0, 0);
        // 左右旋转转身体
        transform.Rotate(Vector3.up * (mouseX * Time.deltaTime) * xSensitivity);
    }
}
```

InputManager处理

```csharp
//...省略...
public class InputManager : MonoBehaviour
{
//...省略...
    private PlayerLook look;
    void Awake()
    {
//...省略...
        look = GetComponent<PlayerLook>();
    }

    private void Update()
    {
        look.ProcessLook(onFoot.Look.ReadValue<Vector2>());
    }
//...省略...
}
```
# Player应用脚本，完成

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4b969c4d967b44e89977e23edc034e97~tplv-k3u1fbpfcp-watermark.image?)
# 小结
Unity的Input System将所有输入设备和输入事件表示为Action的集合。Action是一组定义了输入事件和对应响应的属性和方法的结构。您可以使用这些Action来处理游戏中的输入，例如移动、跳跃、攻击等。

Action Map是一组关联的Action的集合，它们共享相同的输入源和目标设备。例如，您可能会创建一个名为“Player”的Action Map，其中包含控制角色移动的Action，以及名为“UI”的Action Map，其中包含控制菜单导航的Action。通过使用Action Map，您可以轻松地将不同类型的输入分组并组织它们以便于管理。

