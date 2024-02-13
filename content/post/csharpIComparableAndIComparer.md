---
title: "C# IComparable 和 IComparer"
date: 2021-12-23T21:46:58+08:00
author:     "Rezzo"
categories: [ "Tips" ] 
tags: ["Tutorials", "Blog", "C#"]
---
```CSharp
using System;
using System.Collections.Generic;
namespace HelloWorldApplication
{
    class HelloWorld
    {
        static void Main(string[] args)
        {
		Student stu1 = new Student(8, "c");
		Student stu2 = new Student(10, "a");
		Student stu3 = new Student(24, "b");
                
		List<Student> list = new List<Student>();
		list.Add(stu3);
		list.Add(stu2);
		list.Add(stu1);
                
		foreach(Student stu in list)
		{
			Console.WriteLine($"name: {stu.name}; age: {stu.age}");
		}
                
		Console.WriteLine("按age排序");
		list.Sort();
		foreach(Student stu in list)
		{
			Console.WriteLine($"name: {stu.name}; age: {stu.age}");
		}
                
		Console.WriteLine("按name排序");
		list.Sort(new StudentComparator());
		foreach(Student stu in list)
		{
			Console.WriteLine($"name: {stu.name}; age: {stu.age}");
		}
                Console.ReadKey();
        }
    }
	// 创建了自己的实体类，如Student
        // 默认想要对其按照年龄进行排序，则需要为实体类实现IComparable接口
	class Student :IComparable<Student>
	{
		public int age;
		public string name;
		
		public Student(int age, string name)
		{
			this.age = age;
			this.name = name;
		}
		
		public int CompareTo(Student other)
		{
			return age - other.age;
		}
	}
	// 如果不想使用年龄作为比较器了
        // 这个时候IComparer的作用就来了，可使用IComparer来实现一个自定义的比较器
	class StudentComparator : IComparer<Student>
	{
                public int Compare(Student a, Student b)
                {
                    return a.name.CompareTo(b.name);
                }
	}
}

```

#### 代码可直接复制粘贴到菜鸟教程[C# 在线工具](https://www.runoob.com/try/showcs.php?filename=HelloWorld)查看效果

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d8b1e741baee496ca2b6ca9b37c57782~tplv-k3u1fbpfcp-watermark.image?)
##### 个人理解： 创建一个新的实体类，想要一个默认的比较方法，就实现“IComparable < T >” 接口，使这个类具有可比性。

##### 如果需要更多的比较方式，比如按年龄，按学号，就新建一个比较器的类，如“class StudentComparator : IComparer < Student >”, 实现其它的比较方式。使用的时候如“list.Sort( new StudentComparator() );”

---
补充：如果想将int[]数组按自己的方式进行排序
```CSharp
    Array.Sort(nums, Compare);
    // 按绝对值，从大到小排序
    public int Compare(int a, int b)
    {
        return Math.Abs(b) - Math.Abs(a);
    }
```
