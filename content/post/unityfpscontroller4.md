---
title: "[Unity] ④第一人称视角控制器——脚步声、血量系统（学习笔记）"
date: 2022-02-19T22:38:49+08:00
author:     "Rezzo"
categories: [ "Tips" ] 
tags: ["Tutorials", "Unity"]

---
[[Unity] ③第一人称视角控制器——交互（学习笔记）](https://rlezzo.github.io/post/unityfpscontroller3/)
# 脚步声
| 需 求 | 功 能 |
| :---: | :---: |
| 脚步声 开/关| bool：useFootsteps |
|不同播放频率|float baseStepSpeed = 0.6f（基础速率）float crouchStepMultipler = 1.5f（倍率）|
|不同地形播放不同脚步声|AudioClip[] grassClips = default（音频列表）|

## 脚步声功能开关
```csharp
[Header("Functional Options")]
    [SerializeField] private bool useFootsteps = true;
```
## 播放频率
假设站立行走，每隔0.6S播放一次脚步声，那么蹲下行走的间隔时间，就是站立的1.5倍，间隔更长，冲刺的是站立的0.6倍，间隔更短。

用 GetCurrentOffset 存储当前间隔时间。
footstepsTimer 从时间轴 0S 开始计时。
```csharp
[Header("Footsteps Paramenters")]
    // 播放速度，看做音频播放间隔时间
    [SerializeField] private float baseStepSpeed = 0.6f;
    [SerializeField] private float crouchStepMultipler = 1.5f;
    [SerializeField] private float sprintStepMultipler = 0.6f;
    // 计时器
    private float footstepsTimer = 0f;
    // 播放间隔
    private float GetCurrentOffset => baseStepSpeed * (isCrouching ? crouchStepMultipler : IsSprinting ? sprintStepMultipler : 1.0f);
```
## 播放列表
```csharp
[Header("Footsteps Paramenters")]
    // 声源，哪个声源播放音频
    [SerializeField] private AudioSource footstepsAudioSource = default;
    // 不同地面类型，播放不同的音频列表
    [SerializeField] private AudioClip[] grassClips = default;
    [SerializeField] private AudioClip[] snowClips = default;
    [SerializeField] private AudioClip[] groundClips = default;
```
---
## 脚步声功能代码
```csharp
void Update()
    {
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
                //
            if (useFootsteps)
                HandleFootsteps();
                //
            ApplyFinalMovement();
        }
    }
```
在地面时，且有移动输入，才播放脚步声。

当timer小于等于0的时候，相当于不在CD时间内，就可以根据射线探测到的地面类型播放对应的声音。
如下图，在②往回走时，这段一直处在CD时间，所以不播放声音，如此往复。
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b0af4d5e57bf4ff6869aef8b2408e221~tplv-k3u1fbpfcp-watermark.image?)
```csharp
private void HandleFootsteps()
    {
        // 不在地面，或者没有移动输入
        if (!isGrounded) return;
        if (moveInput == Vector2.zero) return;

        footstepsTimer -= Time.deltaTime;

        if (footstepsTimer <= 0f)
        {
            // 探测脚下的地面类型，播放音频
            if (Physics.Raycast(playerCamera.transform.position, Vector3.down, out RaycastHit hit, 3f))
            {
                switch (hit.collider.tag)
                {
                    case "Footsteps/Grass":
                        // 从 grassClips 中随机选择一条播放
                        footstepsAudioSource.PlayOneShot(
                        grassClips[UnityEngine.Random.Range(0, grassClips.Length - 1)]);
                        break;
                    case "Footsteps/Snow":
                        footstepsAudioSource.PlayOneShot(
                        snowClips[UnityEngine.Random.Range(0, snowClips.Length - 1)]);
                        break;
                    default:
                        footstepsAudioSource.PlayOneShot(groundClips[UnityEngine.Random.Range(
                        0, groundClips.Length - 1)]);
                        break;
                }
            }
            
            // 播放间隔，相当于进入CD
            footstepsTimer = GetCurrentOffset;
        }
    }
```
## 脚步声、地面TAG添加
「player」加入音源。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb2d5f0d9174428d8bf564269aaa1e4f~tplv-k3u1fbpfcp-watermark.image?)

Tag里加入「Footsteps/Grass」 和 「Footsteps/Snow」。
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca5c2d0740614c8fb230bc016a8f0547~tplv-k3u1fbpfcp-watermark.image?)

可以从unity商店，下载免费的脚步声资源。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3eaadd7544ed4d92b1233bbb70ec73e3~tplv-k3u1fbpfcp-watermark.image?)

记得将「Audio Source」添加到箭头指向位置，然后将音频片段拖拽到画圈的位置。
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6c5a11f4239f4b54bb82dcbd723b5a4d~tplv-k3u1fbpfcp-watermark.image?)

然后添加一些地面，将TAG修改为对应的TAG，音频就能正确播放了。
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a68d4a8ff3c34581be9a942429debb68~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e75f6058beb64f6da064cd9e1376c63e~tplv-k3u1fbpfcp-watermark.image?)
# 血量系统
这里用的是经典呼吸回血大法。
| 需 求 | 功 能 |
| :---: | :---: |
| 血条| float currentHealth（当前血量）float maxHealth（血量上限）|
|呼吸回血协程|Coroutine regeneratingHealth|
|协程单次回血量|healthValueIncrement = 3f|
|协程单次回血后等待时间|healthTimeIncrement = 0.1f|
|受伤害开始回血前等待时间|timeBeforeHealthRengeStarts = 3f|

掉血 -> 启动呼吸回血协程 -> 等待3S -> 每回血3点，等待0.1S。
```csharp
[Header("Health Paramenters")]
    [SerializeField] private float maxHealth = 100f;
    // 开始回血前，等待3秒
    [SerializeField] private float timeBeforeHealthRengeStarts = 3f;
    // 每次单位回血量
    [SerializeField] private float healthValueIncrement = 3f;
    // 单位回血等待间隔
    [SerializeField] private float healthTimeIncrement = 0.1f;
    private float currentHealth;
    // 回血的协程
    private Coroutine regeneratingHealth;
```

```csharp
    private void ApplyDamage(float damage)
    {
        // 降低生命值，收到伤害掉血

        // 启动协程呼吸回血
    }
    
    
    private void KillPlayer()
    {
        // 血量归零
        // 停止回血协程
        // 死亡后执行的操作
    }
    
    private IEnumerator RegenerateHealth()
    {
        // 回血前等待CD
        // 回血
    }
```
收到伤害调用：
```csharp
    private void ApplyDamage(float damage)
    {
        // 降低生命值
        currentHealth -= damage;

        if (currentHealth <= 0)
            KillPlayer();
        else if (regeneratingHealth != null)
            // 回血过程中受伤，重新等待三秒呼吸回血
            StopCoroutine(regeneratingHealth);
       
        regeneratingHealth = StartCoroutine(RegenerateHealth());
    }
```
player死亡执行：
```csharp
    private void KillPlayer()
    {
        // 玩家死亡状态
        currentHealth = 0;

        if (regeneratingHealth != null)
            StopCoroutine(regeneratingHealth);

        // 死亡后的处理,比如enable一些东西
        print("DEAD");
    }
```
回血协程：
```csharp
private IEnumerator RegenerateHealth()
    {
        yield return new WaitForSeconds(timeBeforeHealthRengeStarts);

        WaitForSeconds timeToWait = new WaitForSeconds(healthTimeIncrement);
        while (currentHealth < maxHealth)
        {
            currentHealth += healthValueIncrement;

            if (currentHealth > maxHealth)
                currentHealth = maxHealth;

            yield return timeToWait;
        }
        // 回血协程执行完毕，变量设为空
        regeneratingHealth = null;
    }
```
## 显示血量
在「Inspector」中，新建一个空「Object」，右键新建「UI」-> 「Text - TextMeshPro」。
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d338c77e0a0946d89958fd931896cbe1~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2e7d5c891f6e4ac4b006e4f67e1d5b2d~tplv-k3u1fbpfcp-watermark.image?)

新建一个script。
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c4e2fa3da8f94aa8b9e5c67367a031d2~tplv-k3u1fbpfcp-watermark.image?)
```csharp
using TMPro;

public class UI : MonoBehaviour
{
    [SerializeField] private TextMeshProUGUI healthText = default;

    // 显示血量，传什么血量文本框显示什么血量
    private void UpdateHealth(float currentHealth)
    {
        healthText.text = "Health: " + currentHealth.ToString("00");
    }
}
```
创建一个伤害方块：玩家的碰撞体进入到伤害方块的碰撞体时，调用「PlayerController」的 ApplyDamage() 方法。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4944672a80944393959883cd09dbde46~tplv-k3u1fbpfcp-watermark.image?)
红色箭头记得开启。
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cd1525d7237549f0a45ba322f709f379~tplv-k3u1fbpfcp-watermark.image?)
```csharp
public class LavaDamage : MonoBehaviour
{
    [SerializeField] private float damage = 15f;
    private void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Player")){
            PlayerController.OnTakeDamage(damage);
        }
    }
}
```
### 其它script要怎么调用「PlayerController」的private方法呢？
```csharp
    public static Action<float> OnTakeDamage;
    // PlayerController 脚本启用时
    private void OnEnable()
    {
        // 绑定委托，方法指针
        OnTakeDamage += ApplyDamage;
    }

    private void OnDisable()
    {
        // 取消委托
        OnTakeDamage -= ApplyDamage;
    }
```
这样，当 「LavaDamage」调用「PlayerController」的 public static Action<float> 变量的时候，PlayerController.OnTakeDamage(damage)， 相当于把伤害作为参赛，传给了「PlayerController」的 ApplyDamage() 方法。可以理解为C++方法指针。

Action 委托 ：封装一个方法，该方法不具有参数并且不返回值。
详细可以去了解委托相关知识。

- - -

同理，再添加两个委托：

```csharp
    public static Action<float> OnDamage;
    public static Action<float> OnHeal;
```
需要「PlayerController」向UI传递血量参数，并修改文本内容，就在UI脚本里，委托绑定UI的UpdateHealth方法。
```csharp
public class UI : MonoBehaviour
{
    [SerializeField] private TextMeshProUGUI healthText = default;

    private void OnEnable()
    {
        PlayerController.OnDamage += UpdateHealth;
        PlayerController.OnHeal += UpdateHealth;
    }

    private void OnDisable()
    {
        PlayerController.OnDamage -= UpdateHealth;
        PlayerController.OnHeal -= UpdateHealth;
    }

    private void UpdateHealth(float currentHealth)
    {
        healthText.text = "Health: " + currentHealth.ToString("00");
    }
}
```
OnDamage?.Invoke，？是检测是否为空，OnDamage是否绑定了 currentHealth 委托，如果未绑定就currentHealth，没绑定不调用。
```csharp
    private void ApplyDamage(float damage)
    {
        currentHealth -= damage;

        // 传递受伤害后血量，显示血量
        OnDamage?.Invoke(currentHealth);
        
        // ...以下省略
    }
```
OnHeal?.Invoke(currentHealth)同理，传递回血后被改变的血量。
```csharp
    private IEnumerator RegenerateHealth()
    {
        yield return new WaitForSeconds(timeBeforeHealthRengeStarts);
         
        WaitForSeconds timeToWait = new WaitForSeconds(healthTimeIncrement);

        while (currentHealth < maxHealth)
        {
            currentHealth += healthValueIncrement;
            if (currentHealth > maxHealth)
                currentHealth = maxHealth;
        //
            OnHeal?.Invoke(currentHealth);
        //    
            yield return timeToWait;
        }
        regeneratingHealth = null;
    }
```
# 完整代码
用 Action 要加上 using System, 所以脚步声 Random.Range 前需要变成 UnityEngine.Random.Range。
```csharp
using System;
public class PlayerController : MonoBehaviour
{    
    [Header("Health Paramenters")]
        [SerializeField] private float maxHealth = 100f;
        // 开始回血前，等待3秒
        [SerializeField] private float timeBeforeHealthRengeStarts = 3f;
        // 每次单位回血量
        [SerializeField] private float healthValueIncrement = 3f;
        // 单位回血等待间隔
        [SerializeField] private float healthTimeIncrement = 0.1f;
        private float currentHealth;
        // 回血的协程
        private Coroutine regeneratingHealth;
        public static Action<float> OnTakeDamage;
        public static Action<float> OnDamage;
        public static Action<float> OnHeal;
        // Health
    private void OnEnable()
    {
        // 绑定委托，方法指针
        OnTakeDamage += ApplyDamage;
    }

    private void OnDisable()
    {
        // 取消委托
        OnTakeDamage -= ApplyDamage;
    }
    
     private void Awake()
    {
        // Health
        currentHealth = maxHealth;
    }
    
    private void ApplyDamage(float damage)
    {
        currentHealth -= damage;

        OnDamage?.Invoke(currentHealth);

        if (currentHealth <= 0)
            KillPlayer();
        else if (regeneratingHealth != null)
            // 恢复过程中受伤，重新等待三秒呼吸回血
            StopCoroutine(regeneratingHealth);
            
        regeneratingHealth = StartCoroutine(RegenerateHealth());
    }

    private void KillPlayer()
    {
        // 玩家死亡状态
        currentHealth = 0;

        if (regeneratingHealth != null)
            StopCoroutine(regeneratingHealth);

        // 死亡后的处理,比如enable一些东西
        print("DEAD");
    }

    private IEnumerator RegenerateHealth()
    {
        yield return new WaitForSeconds(timeBeforeHealthRengeStarts);

        WaitForSeconds timeToWait = new WaitForSeconds(healthTimeIncrement);

        while (currentHealth < maxHealth)
        {
            currentHealth += healthValueIncrement;

            if (currentHealth > maxHealth)
                currentHealth = maxHealth;

            OnHeal?.Invoke(currentHealth);

            yield return timeToWait;
        }
        // 回血协程执行完毕，变量设为空
        regeneratingHealth = null;
    }
}
```

```csharp
public class LavaDamage : MonoBehaviour
{
    [SerializeField] private float damage = 15f;
    private void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Player")){
            PlayerController.OnTakeDamage(damage);
        }
    }
}
```
```csharp
using TMPro;

public class UI : MonoBehaviour
{
    [SerializeField] private TextMeshProUGUI healthText = default;

    private void OnEnable()
    {
        PlayerController.OnDamage += UpdateHealth;
        PlayerController.OnHeal += UpdateHealth;
    }

    private void OnDisable()
    {
        PlayerController.OnDamage -= UpdateHealth;
        PlayerController.OnHeal -= UpdateHealth;
    }

    private void Start()
    {
        UpdateHealth(100f);
    }

    private void UpdateHealth(float currentHealth)
    {
        healthText.text = "Health: " + currentHealth.ToString("00");
    }
}

```
