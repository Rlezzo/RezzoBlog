---
title: "C# 常用排序算法实现"
date: 2021-12-23T22:11:28+08:00
author:     "Rezzo"
categories: [ "Tips" ] 
tags: ["Csharp", "Alogrithm"]

---
---
theme: vuepress
---

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bdd259bbac5b483ea2953bae1187e6eb~tplv-k3u1fbpfcp-watermark.image?)
图出处：[Leetcode图解算法数据结构](https://leetcode-cn.com/leetbook/read/illustration-of-algorithm/pxal47/)
# 平均时间复杂度O(n^2)
下面排序算法中用到的Swap函数
```csharp
public void Swap(int[] arr, int a, int b)
{
    // 不能同一个变量，异或自身
    // 不同的变量，相同的值可以异或
    if(a == b) reutrn;
    arr[a] ^= arr[b];
    arr[b] ^= arr[a];
    arr[a] ^= arr[b];
}
```
## 冒泡排序
时间复杂度：一趟排序比较n个数，然后是n-1个数，然后是n-2个数，等差数列 n * (n + 1) / 2。O(n^2)

稳定性：很规律的左右交换，具备稳定性
```csharp
// arr: 需要排序的数组
// [left，right] 需要排序的数组范围，数组下标
public void BubbleSort(int[] arr, int left, int right)
{
    // 如果数组元素个数小于2，或者给定范围不合法，直接返回
    if ( arr.Length < 2 || left < 0 || right > arr.Length - 1 || right <= left ) return;	
    // 冒泡的一点优化，提升不了什么性能，聊胜无于
    // 默认数组已经有序，假如下面一趟排序，没有发生交换，说明数组已经有序，就可以直接跳出
    bool isOrderly = true; 
    
    while(right > left) 
        {
            // 从最左到右，冒泡，把最大的数一直换到最右边
            for (int i = left; i < right; i++)  
                {
                    if (arr[i].CompareTo(arr[i + 1]) > 0)
                        {
                            Swap(arr, i, i + 1);
                            // 发生了交换，说明还未完全排好序
                            isOrderly = false; 
                        }
                }
            // 数组已经有序，跳出循环		
            if (isOrderly)  break;	
            //  最大的数已经移动到最右端了，右范围缩小
            right--; 
        }
}
```
## 插入排序
时间复杂度：假设最差情况，逆序。第2个数往前换1个，第3个数往前换2个... 1 + 2 + 3 + ... + n - 1。也是累加和，n^2

稳定性：很规律向左一直交换，具备稳定性
```csharp
public void InsertionSort(int[] arr, int left, int right)
{
    // 至少两个数的数组,左右范围也是合法且至少俩数
    if (arr.Length < 2 || left < 0 || right > arr.Length - 1 || right <= left) return false;
    // 从有序数组右边第一个，未排序的数开始
    // 往前交换，直到前面的数比自己小，停止，然后换后面一个数
    for(int i = left + 1; i <= right; i++) 
        {
            // 这个数比左边的数小，交换，j--
            // j 大于左边界，且小于前一个数，往前换
            for(int j = i; left < j; j--) 
            {
                if(arr[j].CompareTo(arr[j - 1]) < 0) Swap(arr, j, j - 1);
            }
            
        }
}
```
## 选择排序
时间复杂度：选一个位置，后面全部扫一遍，重复所有位置，也是n^2

稳定性：因为会把前面的数换到后面，所以顺序可能会乱，不具备稳定性
```csharp
public void SelectionSort(int[] arr, int left, int right)
{
    // 至少两个数的数组,左右范围也是合法且至少俩数
    if (arr.Length < 2 || left < 0 || right > arr.Length - 1 || right <= left) return false;
    
    
    int minIndex;
    
    for(int i = left; i <= right; i++)
    {
        // minIndex 记录最小值的下标
        int minIndex = i;
        // 从初始位置i开始
        // 比如从0开始，然后另一个指针j ，从1开始到最后一个数
        // 扫描 1 ~ end 末尾的所有数，记录最小的数的下标
        for(j = i + 1; j <= right; j++)						
        {
            if(arr[j].CompareTo(arr[minIndex]) < 0)   minIndex = j;                 
        }
        // 将最小的数换到第一位
        Swap(arr, i, minIndex);
    }
}
```
- - -
## 快速排序
时间复杂度： 有划分，一分二，二分四，一层一层的分解下去。随数据量，划分的复杂度为 log n, 每次扫描一次区域，复杂度为n。所以平均时间复杂度为O(n log n)，最坏的情况，会退化为冒泡，比如不随机找数，或者交换中间的数，然后每次取最右的数为基准，而取的数又是最小或者最大的。但是概率极低，通常按照O(n log n)看待

稳定性：因为有不确定的前后交换，不具备稳定性
### 不带聚集：
```csharp
public void QuickSort(int[] arr, int left, int right)
{
    if(right - left < 10)
        {
            // 5~20个元素
            // 可以调用插入排序
            // 提升效率
            // return;
        }
    if (left < right)
        {
            // 取中间的数，作为基准
            // 也可以随机取数
            // 选则最左边作为基准就与左交换，反正与右边交换
            Swap(arr, left, left + ((right - left) >> 1));
            // 获得基准的下标，基准左边都是小于等基准的数，右边是大于等于基准的数
            int pivotIndex = Partition(arr, left, right);
            // 然后递归调用，排序基准左边的范围，右边的范围
            QuickSort(arr, left, pivotIndex - 1);
            QuickSort(arr, pivotIndex + 1, right);
        }

}
```
Partition(): 将数组划分成 [ 小于等于基准的数 ]  [ 基准 ] [ 大于等于基准的数 ] 这样三块区域，左右两块区域还是乱序
#### 填坑法：
以最左数为划分基准
```csharp
// 按基准划分
// 一、填坑法
public int Partition(int[] arr, int left, int right)
{
    // 以最左为基准，记录基准的值，这个坑的位置就出来了 
    // 先动右边指针，填坑
    int pivot = arr[left];
    while (left < right)
   {
       // 当没有越界，且右指针指向的数大于等于基准时，说明这数在大于等于基准的范围，是正确的不需要动
       // 右指针前进（左移）
       while (left < right && arr[right] >= pivot) right--;
       // 当右指针遇到的数小于基准，就把他换到左边区域的坑里
       arr[left] = arr[right];
       // 左指针重复同样的操作
       while (left < right && arr[left] <= pivot) left++;
            
       arr[right] = arr[left];
   }

    //当循环结束时，l两个指针会重合，停在坑上
    //补坑
    arr[left] = pivot;
    return left;
}
```
以最右数为划分基准
```csharp
// 唯一区别就是先动左，还是先动右
public int Partition(int[] arr, int left, int right)
{
    // 右基准 先动左边指针，填坑
    int pivot = arr[right];
    while (left < right)
        {
            while (left < right && arr[left] <= pivot) left++;
            arr[right] = arr[left];
            while (left < right && arr[right] >= pivot) right--;
            arr[left] = arr[right];
        }

    //当循环结束时，l两个指针会重合，停在坑上
    //补坑
    arr[right] = pivot;
    return right;
}
```
#### 交换法：
以最左数为划分基准
```csharp
public int Partition(int[] arr, int left, int right)
{
    // 左基准
    // 记住左基准的下标，不操作做基准的数
    int pivot = left;
    while(left < right)
        {
            // 一直向左移动，直到遇到比基准小的数
            while (left < right && arr[right] >= arr[pivot]) right--;
            // 一直向右移动，直到遇到比基准大的数
            while (left < right && arr[left] <= arr[pivot]) left++;
            // 互换
            Swap(arr, left, right);
        }
    // 将基准换到正中间
    Swap(arr, pivot, left);
    // 左右指针，停止位置永远重合，返回哪个都行
    return left;
}
```
以最右数为划分基准
```csharp
// 交换法
public int Partition(int[] arr, int left, int right)
{
    // 右基准
    int pivot = right;
    while(left < right)
        {
            while (left < right && arr[left] <= arr[pivot]) left++;
            while (left < right && arr[right] >= arr[pivot]) right--;
            Swap(arr, left, right);
        }
    Swap(arr, pivot, right);
    // 左右指针，停止位置永远重合，返回哪个都行
    return right;
}
```
### 带聚集 + 尾递归：
划分过程中：[小于p的区域] [等于p的区域] [  未扫描区域  ] [等于p的区域] [大于p的区域] [p]

划分为：[小于p的区域] [等于p的区域][大于p的区域]，返回等于p的区域的左右边界
```csharp
public void QuickSort(int[] arr, int left, int right)
{
    //if (right - left < 10)
    //{
    //    //用插排
    //    // return
    //}
    // 这里用while，迭代，右半区间
    while (left < right) 
        {
            Swap(arr, left, left + ((right - left) >> 1) );

            int pivotIndex = Partition(arr, left, right);
            // 这里是递归
            QuickSort(arr, left, pivotIndex - 1);
            // 差别
            left = pivotIndex + 1;
        }
}
/*
	横向，不停的迭代右半区间
	(0, 100)  →while→→→ 迭代 →→→→→→→→→→→  （50,100）→→→→→→→→→(76, 100)
  	  ↓		                         ↓
         递归                                  (50, 75) →→→ (64,75)
          ↓                                      ↓
        (0,49)  →→→→→→(26,49)	                 ↓ 
          ↓                                     (50, 63)
        (0,25)
	  纵向，不停的递归左半区间
*/
```
带聚集的划分，返回左右边界
```csharp
public int[] Partition(int[] arr, int left, int right)
{
    // 右基准
    int pivot = right;
    int leftLen = 0;
    // 因为右基准不动，右指针又从right开始，所以不算上右基准那一动的长度
    int rightLen = -1;
    while (left < right)
        {
            while (left < right && arr[left] <= arr[pivot])
                {
                    // 相同，小于等于长度加一
                    if(arr[left] == arr[pivot]) leftLen++;
                    // 小于pivot，加入小于区域
                    else Swap(arr, left, left - leftLen);
                    // 左指针前进
                    left++;
                }

            while (left < right && arr[right] >= arr[pivot])
                {
                    if(arr[right] == arr[pivot]) rightLen++;
                    else Swap(arr, right, right + rightLen);
                    right--;
                }
            Swap(arr, left, right);
        }
		
		
    // 停止位置永远重合
    // 因为左指针先动
    // 如果左指针处跳出循环，就是左指针向右碰到右指针，右指针因为在上一轮交换过，则这个位置是大于P的数
    // 如果右指针处跳出循环，就是右指针向左碰到左指针，左指针停住的位置，一定是大于P的数
    // 所以此位置与右基准交换，不会出问题
    Swap(arr, left, pivot);
    // 如果没有进第二个循环，pivot 就是最大的
    // [6,4,2,4,3][9]
    // 第一个循环，左指针一直移动到9，与右指针重合，跳出循环，rightLen没动
    if(rightLen == -1) rightLen = 0;
    // 返回左右边界位置
    return new int[] { left - leftLen, right + rightLen};
}
```
- - -
## 归并排序
时间复杂度： 类似快排的分解，但是更均匀，稳定。时间复杂度是比较稳定的O(n log n)

稳定性：优先放左侧的数，从小到大，从左往右排序合并，具备稳定性
```csharp
public void MergeSort(int[] arr, int left, int right)
    {
        // 如果只有一个数，即left == right
        // 或者越界，不进行合并
        if (left >= right) return;
        // 中点下标
        int mid = left + ((right - left) >> 1);
        // 归并排序左半部分
        MergeSort(arr, left, mid);
        // 右半部分
        MergeSort(arr, mid + 1, right);
        // 将排好序的左右两部分，合并成有序的一个部分
        Merge(arr, left, mid, right);
    }
```
Merge:
```csharp
public void Merge(int[] arr, int left, int mid, int right)
{
    // 一个临时数组，存放排好序后的结果
    int[] tmpArr = new int[right - left + 1];
    // i用来扫描tmpArr
    int i = 0;
    // left ~ mid 区域的指针
    int leftIndex = left;
    // mid + 1 ~ right 区域的指针
    int rightIndex = mid + 1;
    // 先把两者重合区域比较完，放进数组
    while (leftIndex <= mid && rightIndex <= right)
    // 先放左边的能保证稳定性
    // 如果左边的数小于等于右边的数，就放右边的数进临时数组，左边的指针前进，右边的不动
    // 反正，放右边的数，左边的指针不动
    // 临时数组的指针每次都前进
    // 比如 [1, 3] [2, 4] -->  [1, 2, 3, 4] 会按这个顺序放入临时数组
    // 因为递归会分解到只有两个数的时候开始进行合并
    // 所以从最底层的数组开始，左右都是有序的，因此不会出现大数排到小数前的情况
    tmpArr[i++] = arr[leftIndex] <= arr[rightIndex] ? arr[leftIndex++] : arr[rightIndex++];
    
    // 左右两边如果有未比较过的数，那一方一定都大于前面的数
    // 直接加入
    // left ~ mid
    while (leftIndex <= mid) tmpArr[i++] = arr[leftIndex++];
    // 右边同理
    // mid + 1 ~ right
    while (rightIndex <= right) tmpArr[i++] = arr[rightIndex++];
    
    // 将临时数组中排好序的数据，存回原数组
    // 注意 left + i 下标
    for (i = 0; i < tmpArr.Length; i++) arr[left + i] = tmpArr[i];

}
```
## 堆排序
时间复杂度 O（N * log N）
空间复杂度 O(1) 
不稳定

```csharp
public void HeapSort(int[] arr)
{
    if (arr.Length < 2) return;

    // 从第一个数字开始，到最后一个数字，一个个加入堆，改成大顶堆
    // n * log n
    for (int i = 0; i < arr.Length; i++) HeapInsert(arr, i); 

    //  堆数组，初始个数即为堆大小
    int heapSzie = arr.Length;

    // 把堆最后一个数，和第一个数交换
    // 把这个堆数组中，最大的数移动到最后，heapSzie减一，当做移出堆
    Swap(arr, 0, --heapSzie);

    while(heapSzie > 0) // 堆还有数的时候就重复进行上述操作  O（N）
        {
            // 堆化，把换上来数下沉到它应该在的位置
            Heapify(arr, 0, heapSzie);  // O（log N）

            //  再把最大的数放回末尾，移出堆
            Swap(arr, 0, --heapSzie);
        }

}
```
```csharp
public void HeapInsert(int[] arr, int index)
{
    // （index - 1）/ 2，代表父节点的位置
    //  如果这个数的值，比父节点的值大
    while (arr[index] > arr[(index - 1) / 2])
        {
            // 交换上去
            Swap(arr, index, (index - 1) / 2);
            // 交换上去以后，这个数的index变成父节点的
            index = (index - 1) / 2;
        }
}
```
```csharp
public void Heapify(int[] arr, int index, int heapSize)
{
    // 左孩子的数组下标
    int leftChild = index * 2 + 1;

    while(leftChild < heapSize)  
    // 是否有左孩子（因为堆是完全二叉树结构），没有左孩子一定也没有右孩子，到底了
        {
            // 两个左右孩子比较，谁大，把下标记录下来
            // 如果，右孩子存在，然后右孩子比左孩子大，记录右孩子，反之，只有左孩子或者左孩子大，记录左孩子
            int largest = leftChild + 1 < heapSize && arr[leftChild + 1] > arr[leftChild] ? leftChild + 1 : leftChild;

            // 然后左右孩子中大的那个，和index比较，记录最大的那个数的下标
            largest = arr[largest] > arr[index] ? largest : index;

            // index 大于两个孩子，或者到底了，退出循环
            if (largest == index) break;

            // 如果下面有孩子比他大，把孩子换上来，index换下去
            Swap(arr, largest, index);

            // index的值，换到了孩子的下标上，从这个下标开始进行下一轮的比较
            index = largest;
            leftChild = index * 2 + 1;
        }
}
```

## 基数排序
桶排序的一种，不基于比较的排序

时间复杂度O(N * d)  每个数，都按最大位数操作 d次，d这里是几位数

空间复杂度O(N + k)  一个和N一样长度的桶，加一个基数长度的数组，k这里是K进制，十进制比较这里是10（0~9）

有稳定性
```csharp
/// <summary>
/// 基数排序
/// </summary>
/// <param name="arr">数组引用</param>
/// <param name="left">数组左范围下标</param>
/// <param name="right">数组右范围下标</param>
/// <param name="digit">最长几位数</param>
public void RadixSort(int[] arr, int left, int right, int digit)
{
    // 0 ~ 9  10个基数
    int radixLen = 10;
    int i = 0, j =  0;
    // 桶，每轮比较时临时存数
    int[] bucket = new int[right - left + 1];

    // d代表这一轮，是比较哪一位，1是个位，2是十位...
    for(int d = 1; d <= digit; d++)
        {
            // 统计这一位，每个数的词频，用累加和记录
            // 比如[0, 1, 1, 2, 2, 2, 3, 3, 3, 3]  → count[1, (1 + 2), (3 + 3), (6 + 4)]
            // 表示该位是0的数，有小于等于1个，该位是3的小于等于10个，分别对应数组下标存放计数
            int[] count = new int[radixLen]; // 0~9 十个位置，记0~9的数

            for (i = left; i <= right; i++)
                {
                    j = GetDigit(arr[i], d);
                    count[j]++;
                }

            // 转换成累加和形式
            for (i = 1; i < radixLen; i++)
                {
                    count[i] += count[i - 1];
                }

            // 按照统计结果，按该位，从大到小的顺序，放进桶里
            // 从后开始读数（因为上一轮排好后，大数在后，小数在前）
            for (i = right; i >= left; i--)
                {
                    // 得到这个要入桶的该位数的值，比如7
                    j = GetDigit(arr[i], d);
                    // 这个值,7在count[7]中
                    // count[7]的值表示小于等于7的数有多少个
                    // 比如3个，则 3 - 1 = 2，就是说这个7应该在2号下标的位置
                    bucket[count[j] - 1] = arr[i];
                    // 统计的数减一
                    count[j]--;
                }

            // 从桶里放回去
            for (i = left, j = 0; i <= right; i++, j++)
                {
                    arr[i] = bucket[j];
                }

        }
}
```
```csharp
/// <summary>
/// 得到从右往左第d位的数字
/// </summary>
/// <param name="num">数</param>
/// <param name="d">从右往左第几位，个位就是1，十位是2，以此类推</param>
/// <returns></returns>
public int GetDigit(int num, int d)
{
    if(d < 1) return 0;
    // 将要取的位置的数移动到个位
    int remove = (int)Math.Pow(10, d - 1);
    // 除10取余
    return (num / remove) % 10;
}
```




