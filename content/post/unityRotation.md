---
title: "[Unity]旋转的三种实现方式"
date: 2022-01-08T18:47:57+08:00
author:     "Rezzo"
categories: [ "Tech" ] 
tags: ["Tutorials", "unity"]
---
# Transform.Rotate 欧拉角旋转
## Transform.Rotate(float xAngle, float yAngle, float zAngle)
「xAngle」以x轴为旋转轴中心每帧旋转的角度

「yAngle」以y轴为旋转轴中心每帧旋转的角度

「zAngle」以z轴为旋转轴中心每帧旋转的角度
```csharp
void Update ()
{
    // 每一帧，以y轴为旋转轴旋转1°
    transform.Rotate(0f, 1f, 0f); 
    // 如果是个人物模型，第一人称看到的效果就是一直向右转
    // 每帧旋转1°
    // 左手点赞手势，大拇指指向轴正方向，手指蜷曲方向为旋转正方向
}
```
## Transform.Rotate(Vector3 eulerAngles)
用 Vector3 eulerAngles 代替 三个旋转角
```csharp
void Update () 
{
    transform.Rotate(new Vector3(0f,1f,0f));
    // 每一帧，以y轴为旋转轴旋转1°
}
```
## Transform.Rotate(Vector3 axis, float angle, Space relativeTo)
「Vector3 axis」 旋转轴，new Vector3(0f,1f,0f) 就代表以Y轴为旋转轴，值大小不影响旋转速度

「angle」 每一帧的旋转角度

「Space relativeTo」相对于本地坐标还是世界坐标

```csharp
void Update () 
{
    transform.Rotate(new Vector3(0f, 1f, 0f), 1f, Space.Self);
    // 每一帧，以本地坐标系（自身）y轴为旋转轴，旋转1°
}
```
其它解释按方法注解类推。
# Transform.RotateAround  一个物体围绕另一个物体旋转
## Transform.RotateAround(Vector3 point, Vector3 axis, float angle)

「Vector3 point」围绕哪个点旋转

「Vector3 axis」自身哪个轴围绕这个点旋转

「angle」每一帧旋转的角度

```csharp
void Update ()
{
    transform.RotateAround(new Vector3(0, 0, 0), new Vector3(0f, 1f, 0f), 1f);
    // new Vector3(0, 0, 0) 改成自己的position，就可以实现和上面一样的效果
}
```
个人理解为，空间中一点 point 向 Vector3 axis延长线做追垂线，然后垂线抓住axis，进行旋转。
打个比方，空间中有一根棍子，记作y轴，然后某一点，向y做垂线，垂线记作z轴，那么以这一点为中心，向x方向做圆周运动。
# 直接修改旋转角度
## 修改欧拉角 eulerAngles和localEulerAngles
「eulerAngles」世界坐标系中以欧拉角表示的旋转

「localEulerAngle」本地坐标中以欧拉角表示的旋转
```csharp
void Update ()
{
    // 改变y轴旋转角度30°
    transform.eulerAngles = new Vector3(0, 30f, 0);
    // 旋转至某一角度（有过程）
    float speed = 5.0f;
    transform.eulerAngles = Vector3.MoveTowards(transform.eulerAngles, new Vector3(0, 30f, 0), Time.deltaTime * speed);
}
```
## rotation和localRotation
四元数用于表示旋转，所有的 rotation都为Quaternion类型，没有Vector3类型的 eulerAngles 直观，但其不受万向锁影响，可以轻松插值运算。

「rotation」世界坐标系中以四元数表示的旋转

「localRotation」本地坐标中以四元数表示的旋转
```csharp
void Update () 
{   
    // 改变y轴旋转角度30°
    transform.rotation = Quaternion.Euler(new Vector3(0, 30f, 0));
    // 旋转至某一角度（有过程）
    float speed=10f;        
    transform.rotation = Quaternion.RotateTowards(transform.rotation, Quaternion.Euler(new Vector3(0, 30f, 0)), Time.deltaTime * speed);
}
```


