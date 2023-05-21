---
title: "十大排序算法Go实现"
subtitle: ""
date: 2022-03-01T17:51:01+08:00
draft: false
# page title background image
bg_image: "images/backgrounds/page-title.jpg"
# about image
image: "images/blog/golang.jpg"
# meta description
description: "十大排序算法Go实现"
# categories
categories: ["go"]
# tags
tags: ["go", "排序"]
# type
type: "post"

aliases: ""
---

**基础排序**

-   1.冒泡排序-稳定-每轮前部排序-(无序区，有序区)
-   2.选择排序-不稳定-每轮后部排序-(有序区，无序区)
-   3.插入排序-稳定-每轮前部倒序插入-(相对有序，无序)
-   4.希尔排序-不稳定(改良排序 )
-   5.归并排序-稳定-递归与非递归实现：解决快排的最坏情况-O(n^2)，随机主元
-   6.快速排序-不稳定
-   7.堆排序-不稳定
-   8.基数排序
-   9.桶排序
-   10.计数排序

归并、快排、堆排适合数据规模大的排序，渐进性的最佳排序算法
归并并行计算，性能好很多，时间 `O(n/logn)`

后 3 种基于哈希思想，时间效率到 `O(n)`，空间换时间
空间换时间，适合数据规模较小
有重复数字出现，用计数/基数排序；基数排序是对计数排序的优化，在辅助空间上的优化，基数排序所需辅助空间为：10 个该元素空间，计数排序所需辅助空间为：最大数字+1 个空间
无重复数字建议桶排序

## 1.冒泡排序

{{<callout idea "思路">}}

-   比较相邻元素，如果前者比后者大，则进行位置交换
-   从第一个开始比较到未确定位置的最后一个，每经过一轮，则代表当前最大的数到达本次最后的位置

{{</callout>}}

{{< tabs tabTotal="1">}}
{{< tab tabName="Go" >}}

```go
/*
    时间复杂度 平均 O(n^2)， 最好 O(n)，最坏 O(n^2)
    空间复杂度 O(1)
*/
func bubbleSort(nums []int) []int {
    for i := 0; i < len(nums); i++ {
        // 记录是否交换数据，交换标明数组不是有序的
        exchange := false
        for j := 0; j < len(nums)-i; j++ {
            if nums[j] > nums[j+1] {
                // swap
                nums[j], nums[j+1] = nums[j+1], nums[j]
                // 交换，更新变量值
                exchange = true
            }
        }
        // 未进行交换，证明是有序的
        if !exchange {
            return nums
        }
    }
    return nums
}
```

{{< /tab >}}
{{< /tabs >}}

## 2.选择排序

{{<callout idea "思路">}}
每一次选出最小者直接交换到左侧，省出了多余的元素交换
{{</callout>}}

{{< tabs tabTotal="1">}}
{{< tab tabName="Go" >}}

```go
/*
    时间复杂度 O(n^2)
    空间复杂度 O(1)
*/
func selectSort(nums []int) []int {
    for i := 0; i < len(nums); i++ {
        // 设置当前数组最小的值的索引
        min := i
        for j := i + 1; j < len(nums); j++ {
            if nums[j] < nums[min] {
                // 更新索引为当前最小值的索引
                min = j
            }
            // 把最小值放到本次循环的第一位
            nums[i], nums[min] = nums[min], nums[i]
        }
    }
    return nums
}
```

{{< /tab >}}
{{< /tabs >}}

## 3.插入排序

{{<callout idea "思路">}}
维护一个有序区，把元素一个一个插入到有序区的适当位置，直到所有元素有序为止
{{</callout>}}

{{< tabs tabTotal="1">}}
{{< tab tabName="Go" >}}

```go
/*
    时间复杂度 O(n^2)
    空间复杂度 O(1)
*/
func insertSort(nums []int) []int {
    for i := 1; i < len(nums)-1; i++ {
        // 插入数值
        insertValue := nums[i]
        j := i - 1
        for ; j >= 0 && nums[j] > insertValue; j-- {
            // 如果前面的值大于插入值，依次把值向后移动
            nums[j+1] = nums[j]
        }
        // 进行赋值
        nums[j+1] = insertValue
    }

    return nums
}
```

{{< /tab >}}
{{< /tabs >}}

## 4.希尔排序

{{<callout idea "思路">}}
设置希尔增量每次折半，逐步分组进行粗调，最后进行插入排序
{{</callout>}}

{{< tabs tabTotal="1">}}
{{< tab tabName="Go" >}}

```go
/*
    时间复杂度 平均为 O(nlog2n)，最坏为 O(n^2)
    空间复杂度 O(1)
*/
func shellSort(nums []int) []int {
    for gap := len(nums) / 2; gap > 0; gap /= 2 {
        for i := gap; i < len(nums); i++ {
            j := i
            for j-gap >= 0 && nums[j-gap] > nums[j] {
                nums[j-gap], nums[j] = nums[j], nums[j-gap]
                j -= gap
            }
        }
    }
    return nums
}
```

{{< /tab >}}
{{< /tabs >}}

## 5.快速排序

{{<callout idea "思路(分而治之)">}}

-   在待排序的元素任取一个元素作为基准（通常选第一个元素，但最好的方法是从待排序元素中随机选取一个为基准），称为基准元素（pivot）
-   将待排序的元素进行分区，比基准元素大的元素放在它的右边，比基准元素小的放在它的左边
-   对左右两个分区重复以上步骤，直到所有的元素都是有序的

{{</callout>}}

{{< tabs tabTotal="1">}}
{{< tab tabName="Go" >}}

```go
/*
    时间复杂度 平均为 O(nlogn)，最坏为 O(n^2)
    空间复杂度 平均为 O(logn)，最坏为 O(n)
*/
func quikSort(nums []int) []int {
    var quick func(nums []int, left, right int) []int
    quick = func(nums []int, left, right int) []int {
        // 递归终止条件
        if left > right {
            return nil
        }
        // 左右指针及主元
        i, j, pivot := left, right, nums[left]
        for i < j {
            // 寻找小于主元的右边元素
            for i < j && nums[j] >= pivot {
                j--
            }
            // 寻找大于主元的左边元素
            for i < j && nums[i] <= pivot {
                i++
            }
            // 交换i/j下标元素
            nums[i], nums[j] = nums[j], nums[i]
        }
        // 交换元素
        nums[i], nums[left] = nums[left], nums[i]
        quick(nums, left, i-1)
        quick(nums, i+1, right)
        return nums
    }
    return quick(nums, 0, len(nums)-1)
}
```

{{< /tab >}}
{{< /tabs >}}

## 6.归并排序

{{<callout idea "思路(递归实现)">}}

-   基于递归的归并-自上而下的合并，另有非递归法的归并(自下而上的合并)
-   都需要开辟一个大小为 n 的数组中转
-   将数组分为左右两部分，递归左右两块，最后合并，即归并
-   如在一个合并中，将两块部分的元素，遍历取较小值填入结果集
-   类似两个有序链表的合并，每次两两合并相邻的两个有序序列，直到整个序列有序

{{</callout>}}

{{< tabs tabTotal="1">}}
{{< tab tabName="Go" >}}

```go
/*
    时间复杂度 O(nlogn)
    空间复杂度 O(logn) | O(n)
*/

// 递归实现归并算法
func mergeSort(nums []int) []int {
    merge := func(left, right []int) []int {
        res := make([]int, len(left)+len(right))
        var l, r, i int
        // 通过遍历完成比较填入res中
        for l < len(left) && r < len(right) {
            if left[l] <= right[r] {
                res[i] = left[l]
                l++
            } else {
                res[i] = right[r]
                r++
            }
            i++
        }
        // 如果left或者right还有剩余元素，添加到结果集的尾部
        copy(res[i:], left[l:])
        copy(res[i+len(left)-l:], right[r:])
        return res
    }
    var sort func(nums []int) []int
    sort = func(nums []int) []int {
        if len(nums) <= 1 {
            return nums
        }
        // 拆分递归与合并
        // 分割点
        mid := len(nums) / 2
        left := sort(nums[:mid])
        right := sort(nums[mid:])
        return merge(left, right)
    }
    return sort(nums)
}
```

{{< /tab >}}
{{< /tabs >}}

{{<callout idea "思路(非递归实现)">}}
利用变量，自下而上的方式合并
{{</callout>}}

{{< tabs tabTotal="1">}}
{{< tab tabName="Go" >}}

```go
/*
    时间复杂度 O(nlogn)
    空间复杂度 O(n)
*/

// 非递归实现归并算法
func mergeSort(nums []int) []int {
    if len(nums) <= 1 {
        return nums
    }
    merge := func(left, right []int) []int {
        res := make([]int, len(left)+len(right))
        var l, r, i int
        // 通过遍历完成比较填入res中
        for l < len(left) && r < len(right) {
            if left[l] <= right[r] {
                res[i] = left[l]
                l++
            } else {
                res[i] = right[r]
                r++
            }
            i++
        }
        // 如果left或者right还有剩余元素，添加到结果集的尾部
        copy(res[i:], left[l:])
        copy(res[i+len(left)-l:], right[r:])
        return res
    }
    i := 1 //子序列大小初始1
    res := make([]int, 0)
    // i控制每次划分的序列长度
    for i < len(nums) {
        // j根据i值执行具体的合并
        j := 0
        // 按顺序两两合并，j用来定位起始点
        // 随着序列翻倍，每次两两合并的数组大小也翻倍
        for j < len(nums) {
            if j+2*i > len(nums) {
                res = merge(nums[j:j+i], nums[j+i:])
            } else {
                res = merge(nums[j:j+i], nums[j+i:j+2*i])
            }
            // 通过index控制每次将合并的数据填入nums中
            // 重填入的次数和合并及二叉树的高度相关
            index := j
            for _, v := range res {
                nums[index] = v
                index++
            }
            j = j + 2*i
        }
        i *= 2
    }
    return nums
}
```

{{< /tab >}}
{{< /tabs >}}

## 7.堆排序

{{<callout idea "思路(本质是完全二叉树，根节点为最大值或者最小值)">}}

-   1.建堆，从非叶子节点开始依次堆化，注意逆序，从下往上堆化
    -   建堆流程：父节点与子节点比较，子节点大则交换父子节点，父节点索引更新为子节点，循环操作
-   2.尾部遍历操作，弹出元素，再次堆化
    -   弹出元素排序流程：从最后节点开始，交换头尾元素，由于弹出，end--，再次对剩余数组元素建堆，循环操作
        建堆函数，堆化

{{</callout>}}

{{< tabs tabTotal="1">}}
{{< tab tabName="Go" >}}

```go
/*
    时间复杂度 O(nlogn)
    空间复杂度 O(1)
*/
func heapSort(nums []int) []int {
    // 遍历元素时间O(n)，堆化时间O(logn)，开始建堆次数多些，后面次数少
    var heapify func(nums []int, root, end int)
    heapify = func(nums []int, root, end int) {
        // 大顶堆堆化，堆顶值小一直下沉
        for {
            // 左孩子节点索引
            child := root*2 + 1
            // 越界跳出
            if child > end {
                return
            }
            // 比较左右孩子，取大值，否则child不用++
            if child < end && nums[child] <= nums[child+1] {
                child++
            }
            // 如果父节点已经大于左右孩子大值，已堆化
            if nums[root] > nums[child] {
                return
            }
            // 孩子节点大值上冒
            nums[root], nums[child] = nums[child], nums[root]
            // 更新父节点到子节点，继续往下比较，不断下沉
            root = child
        }
    }
    end := len(nums) - 1
    // 从最后一个非叶子节点开始堆化
    for i := end / 2; i >= 0; i-- {
        heapify(nums, i, end)
    }
    // 依次弹出元素，然后再堆化，相当于依次把最大值放入尾部
    for i := end; i >= 0; i-- {
        nums[0], nums[i] = nums[i], nums[0]
        end--
        heapify(nums, 0, end)
    }
    return nums
}
```

{{< /tab >}}
{{< /tabs >}}

## 8.基数排序

{{<callout idea "思路">}}

-   将整数按位数切割成不同的数字，然后按每个位数分别比较。
-   具体做法是：将所有待比较数值统一为同样的数位长度，数位较短的数前面补零。然后，从最低位开始，依次进行一次排序。这样从最低位排序一直到最高位排序完成以后, 数列就变成一个有序序列。

{{</callout>}}

{{< tabs tabTotal="1">}}
{{< tab tabName="Go" >}}

```go
/*
    时间复杂度 O(nk)
    空间复杂度 O(n+k)
*/
func radixSort(li []int) {
    // 一次遍历获取最大值
    max_num := li[0]
    for i := 0; i < len(li); i++ {
        if max_num < li[i] {
            max_num = li[i]
        }
    }
    // 根据最大值的位数确定分几轮基数排序，如234，需要3轮，9仅需要一轮排序
    for j := 0; j < len(strconv.Itoa(max_num)); j++ {
        // 1.每轮排序，先分桶，数据装桶
        bin := make([][]int, 10)
        for k := 0; k < len(li); k++ {
            n := li[k] / int(math.Pow(10, float64(j))) % 10
            bin[n] = append(bin[n], li[k])
        }
        // 2.每轮排序，装完桶后，依次遍历桶，重排数组
        m := 0
        for p := 0; p < len(bin); p++ {
            for q := 0; q < len(bin[p]); q++ {
                li[m] = bin[p][q]
                m++
            }
        }
    }
}
```

{{< /tab >}}
{{< /tabs >}}

## 9.桶排序

{{<callout idea "思路">}}

-   基于哈希思想的外排稳定算法
-   相当于计数排序的改进版，服从均匀分布，先将数据分到有限数量的桶中，
-   每个桶分别排序，最后将非空桶的数据拼接起来

{{</callout>}}

{{< tabs tabTotal="1">}}
{{< tab tabName="Go" >}}

```go
/*
    时间复杂度 O(n+k)
    空间复杂度 O(n+k)
*/
func bucketSort(nums []int) []int {
    //
    var bucket func(nums []int, bucketSize int) []int
    bucket = func(nums []int, bucketSize int) []int {
        if len(nums) < 2 {
            return nums
        }
        // 获取最大最小值
        minAndMax := func(nums []int) (min, max int) {
            minNum := math.MaxInt32
            maxNum := math.MinInt32
            for i := 0; i < len(nums); i++ {
                if nums[i] < minNum {
                    minNum = nums[i]
                }
                if nums[i] > maxNum {
                    maxNum = nums[i]
                }
            }
            return minNum, maxNum
        }
        min_, max_ := minAndMax(nums)
        // 定义桶
        // 构建计数桶
        bucketCount := (max_-min_)/bucketSize + 1
        buckets := make([][]int, bucketCount)
        for i := 0; i < bucketCount; i++ {
            buckets[i] = make([]int, 0)
        }
        // 装桶-排序过程
        for i := 0; i < len(nums); i++ {
            // 桶序号
            bucketNum := (nums[i] - min_) / bucketSize
            buckets[bucketNum] = append(buckets[bucketNum], nums[i])
        }
        // 桶中排序
        // 上述装桶完成，出桶填入元素组
        index := 0
        for _, bucket := range buckets {
            sort.Slice(bucket, func(i, j int) bool {
                return bucket[i] < bucket[j]
            })
            for _, num := range bucket {
                nums[index] = num
                index++
            }
        }
        return nums
    }
    // 定义桶中的数量
    var bucketSize int = 2
    return bucket(nums, bucketSize)
}
```

{{< /tab >}}
{{< /tabs >}}

## 10.计数排序

{{<callout idea "思路">}}

-   找出待排序的数组的最大值和最小值
-   创建数组存放各元素的出现次数，先于[min, max]之间
-   统计数组值的个数
-   反向填充数组，填充时注意,num[i]=j+min，

{{</callout>}}

j-前面需要略过的数的个数，两个维度，依次递增的数 j++，一个是重复的数的计数 j-不变

{{< tabs tabTotal="1">}}
{{< tab tabName="Go" >}}

```go
/*
    时间复杂度 平均为 O(n)
    空间复杂度 平均为 O(n)
*/
func counterSort(nums []int) []int {
    // 获取最大最小值
    minAndMax := func(nums []int) (min, max int) {
        minNum := math.MaxInt32
        maxNum := math.MinInt32
        for i := 0; i < len(nums); i++ {
            if nums[i] < minNum {
                minNum = nums[i]
            }
            if nums[i] > maxNum {
                maxNum = nums[i]
            }
        }
        return minNum, maxNum
    }
    min_, max_ := minAndMax(nums)
    // 中转数组存放遍历元素
    // 空间只需要min-max
    tmpNums := make([]int, max_-min_+1)
    // 遍历原数组
    for i := 0; i < len(nums); i++ {
        tmpNums[nums[i]-min_]++
    }
    // 遍历中转数组填入原数组
    j := 0
    for i := 0; i < len(nums); i++ {
        // 如果对应数字cnt=0，说明可以计入下一位数字
        for tmpNums[j] == 0 {
            j++
        }
        // 填入数字
        nums[i] = j + min_
        // 填一个数字，对应数字cnt--
        tmpNums[j]--
    }
    return nums
}
```

{{< /tab >}}
{{< /tabs >}}

## 参考

-   [八大排序算法的 Golang 实现](https://juejin.cn/post/7066728890575618078)
-   [十大排序算法-golang 实现](https://leetcode-cn.com/problems/sort-an-array/solution/shi-da-pai-xu-suan-fa-golangshi-xian-by-8p1bl/)
