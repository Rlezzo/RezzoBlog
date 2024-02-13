---
title: "[Unity]Mathf Clamp"
date: 2021-12-23T22:15:24+08:00
author:     "Rezzo"
categories: [ "Tech" ] 
tags: ["unity", "method"]

---
限制value的值在min和max之间， 如果value小于min，返回min。 如果value大于max，返回max，否则返回value
```
// Set the position of the transform to be that of the time
// but never less than 1 or more than 3
//随着时间设置变换位置
//但是不会小于1或大于3
function Update () {
	transform.position = Vector3(Mathf.Clamp(Time.time, 1.0, 3.0), 0, 0);
}
```
