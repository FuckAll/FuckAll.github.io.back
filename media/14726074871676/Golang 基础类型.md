# Golang 基础类型
## 变量声明
1. 不同基础类型的零值
	
	| 类型 | 零值 |
	| --- | --- |
	| int | 0 |
	| float | 0 |
	| bool | false |
	| string | "" |
	| struct | nil  |
	| array | nil |
	| slice | nil |
	| map | nil |
	| channel | nil |

2. var a int 和 a := 0 的哲学
	1. var a int 

		var a int 可以用来声明局部和全局变量,如果不初始化那么就是对应的零值		var a int = 1 可以声明并且初始化
	2. a := 0

		a := 0 只能用来声明局部变量，不能够声明全局变量
		a := 1 声明局部变量并且初始化
		
	3. 在局部的变量中 var 等价 : ; var 可以使用零值， : 是必须显示指明初始化值

	4. 我的使用习惯

		1. 全局变量使用 var a int = 0 显示指明初始值
		2. 局部变量使用 a := 0 显示指明初始值


	**注意：**
		var a int 也可以用做 var a = 0,效果是一样的，但是这里尽量的使用显示的指定类型和初始值
	
## 	字符串拼加性能问题
1. 字符串拼加的三种方式：
	1. +=
	2. strings.Join()
	3. bytes.Buffer
2. 比较：

	```	
	package main
	import (
		"fmt"
	
	
		"strings"
		"time"
		"bytes"
	)
	
	func main() {
		var buffer bytes.Buffer
	
		s := time.Now()
		for i :=0; i <100000; i++ {
			buffer.WriteString("test is here\n")
		}
		buffer.String() // 拼接结果
		e := time.Now()
		fmt.Println("taked time is ", e.Sub(s).Seconds())
	
	
		s = time.Now()
		str := ""
		for i :=0; i <100000; i++ {
			str += "test is here\n"
		}
		e = time.Now()
		fmt.Println("taked time is ", e.Sub(s).Seconds())
	
		s = time.Now()
		var sl []string
		for i :=0; i <100000; i++ {
			sl = append(sl, "test is here\n")
		}
		strings.Join(sl, "")
		e = time.Now()
		fmt.Println("taked time is", e.Sub(s).Seconds())
	}
	```
	**运行结果**
	
	```
	taked time is  0.0039795080000000005
	taked time is  13.403575973
	taked time is 0.017540489
	```
3. []byte 和 string 
	1. golang 中的[]byte和string可以方便的转换
		1. []byte -> string
			string([]byte) -> string 
		2. string -> []byte
			[]byte(string) -> []byte
			
		**golang中[]byte和string的方法基本上是一样的**
	2. byte和string性能问题之buffer
		1. golang 中 bytes包中有Buffer类型以及相关方法
			1. 直接声明： var buffer bytes.Buffer
			2. 方法成成：

## 数组问题
1.  声明Golang数组有两种方式

	```
		var arr1 = new([5]int)
		var arr2 [5]int
	```
	1. Golang认为数组是一种值类型，但new的时候返回的是数组的指针。
	2. 第一种返回的是一个指针，传递的时候是不进行内存拷贝的。
	3. 第二种方式返回的是数组本身，传递的时候是进行内存拷贝的。
	```
		package main
	
		import (
			"fmt"
		)
		
		func main() {
			var arr1 = new([5]int)  // 返回的是数组的地址
			var arr2 [5]int
		
			// 传地址进入函数，适用于需要修改数组或者数组较大
			arrayAddr(arr1)
			if arr1[0] != 1 {
				fmt.Printf("Want arr1[0]==1 but arr1[0]==%d", arr1[0])
			}
			// 传值进入函数，进行内存的拷贝
			arrayVal(arr2)
			if arr1[1] != 0 {
				fmt.Printf("Want arr1[1]==0 but arr1[1]==%d", arr1[1])
			}
		
		}
		
		func arrayAddr(a *[5]int) {
			a[0] = 1
		}
		
		func arrayVal(a [5]int) {
			a[1] = 2
		}

	```
	
	
2. 切片问题
		**切片在进行传递的时候也是指针传递**
		
		package main

		import (
			"fmt"
		)
		
		func main() {
			sliceAppend()
		
		}
		func sliceAppend() {
			// 声明一个数组,这个并不是Slice, len == cap
			var array = [...]int32{1, 2, 3, 4, 5, 6}

			// 不能使用append，append只能用给slice
			//array = append(array, 6)

			//struct	Slice
			//{				// must not move anything
			//	byte*	array;		// actual data
			//	uintgo	len;		// number of elements
			//	uintgo	cap;		// allocated number of elements
			//};
			//Slice 是指向一定长度的byte指针
			var slice = array[2:4]
			if &array[2] == &slice[0] {
				fmt.Printf("array[2]:%p  slice[0]:%p\n", &array[2], &slice[0])
			} else {
				fmt.Printf("Want array[2](%d)==slice[0](%d) but array[2]=%d,slice[0]%d\n", &array[2], &slice[0], &array[2], &slice[0])
			}
			//Slice 初始长度为len实际存数据的长度，cap为从切割位置开始到数组后的长度
			l := len(slice)
			if len(array)-l != cap(slice) {
				fmt.Printf("Want len+1==cap but len(slice):%d, cap(slice):%d\n", len(array)-l, cap(slice))
			}

			//append扩展Slice，有两种变化
			// 1. 如果append增加之后的长度比array本身短，那么将覆盖掉array中的元素，slice不重新申请空间
			// 2. 如果append增加之后超过了array的长度，那么也会覆盖掉array中的元素，slice重新分配空间，空间扩张的方式为,基数的整数倍，这里的基数是最开始的cap大小
			fmt.Printf("第一次改变slice之前: array=%+v, slice=%+v\n", array, slice)
			a:= []int32{100,102}
			sA := slice
			slice = append(slice,a...)
			if fmt.Sprintf("%p",sA) != fmt.Sprintf("%p",slice) {
				fmt.Printf("Want &sA(%p)==&slice(%p) but sA=%p,&slice=%p\n", sA, slice, sA, slice)
			}
			fmt.Printf("第一次改变slice之后: array=%+v, slice=%+v\n", array, slice)


			fmt.Printf("第二次改变slice之前: array=%+v, slice=%+v\n", array, slice)
			slice=array[2:4]
			a = []int32{100,102,103,104,105,106,107}
			sA = slice
			slice = append(slice,a...)
			if fmt.Sprintf("%p",sA) == fmt.Sprintf("%p",slice) {
				fmt.Printf("Want &sA(%p)!=&slice(%p) but sA=%p,&slice=%p\n", sA, slice, sA, slice)
			}
			fmt.Printf("第二次改变slice之后: array=%+v, slice=%+v\n", array, slice)

			// 总结： slice是一个结构体，结构体的第一个变量是一个指针，所以对slice的操作都会改掉原始的数据，如果append增加新的数据之后
			//可能会大于原来slice的长度，会重新分配内存，这样新的slice就和旧的slice不再有任何的关系。


			// 如果不想改变原始的数据，那么就需要将原来的数据拷贝出来
			slice = array[2:4]
			fmt.Printf("Copy slice之前: slice=%+v\n", slice)
			// copy 命令copy的长度是两个参数中len比较小的
			a = make([]int32, len(slice), (cap(slice)+1)*2)
			n := copy(a, slice)
			if n != 2 {
				fmt.Printf("Copy slice 长度有问题！n==%d", n)
			}
			// 改变a对应的slice
			a = append(a, []int32{12,13}...)
			fmt.Printf("Copy slice之后: slice=%+v, a=%+v\n", slice, a)
		}

			
#总结：除了数组本身，切片和new一个都是指针的形式，数组本身是值类型，其余两种情况得到的嗾使指针，如果想要在传递的时候不进行拷贝操作，那么两种方式，一种就是直接传指针，另外一种的直接传切片。

    1. new() 和 make() 的关系：
        1. new() 是为变量分配内存，然后就将分配好的地址的首地址返回，并且也是进行初始化
        2. make() 这个是用来直接初始化，然后将变量直接返回。
    2. 关于效率问题：
        1. 字符串的拼凑应该直接使用bytes的buffer进行操作，会快一些。
    3. 数组和切片问题：
        1. 数组一旦定义好，就固定长度即len == cap
        2. 数组想要扩容或者是缩容可以转换成切片进行使用,即先要make一个切片，然后将数组中的数据放入其中。
		
	



