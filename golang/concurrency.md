# go语言并发编程

- 并发模型 : Communication Sequential Process (CSP)
- Don't communicate by sharing memory; share memory by communicating 
- 不要通过共享内存来通讯，通过通讯来共享内存

### 协程

- 轻量级 "线程"
- `非抢占式` 多任务处理，由携程主动交出控制权 
- 编译器/解释器/虚拟机层面的多任务,  并非操作系统层面的多任务
- 多个 `协程` 可能在一个或者多个线程上运行,  这是由 go语言的 `调度器` 决定的的
- 协程 的切换是由调度器决定的,  一些可能切换的点: I/O , select
- 使用 `go `关键字 + 后面加 函数，启动一个协程



### Channel

#### 创建 Channel

```go
// nil channel
var c chan int
// make 
c = make(chan int)
// buffered channel , length = 3
c = make (chan int , 3)

// demo
func channelDemo() {
	var channels [10]chan int
	for i := 0; i < 10; i++ {
		channels[i] = make(chan int)
		go doWork(i, channels[i])
	}

	for i := 0; i < 10; i++ {
		channels[i] <- 'a' + i
	}

	for i := 0; i < 10; i++ {
		channels[i] <- 'A' + i
	}

}

func doWork(id int, c chan int) {
	for {
		fmt.Printf("Worker:%d received: %c\n", id, <-c)
	}
}
```



### 等待任务完成

**请求网络**

```go
func doGet(i int) {
	resp, err := http.Get("http://www.baidu.com")
	if err != nil {
		panic(err)
	}
	defer resp.Body.Close()
	contents, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		panic(err)
	}
	fmt.Printf("第%d次请求, 返回: %s\n", i, string(contents)[:10])
}
```

##### 使用channel 等待任务完成 

```go
func channelWait() {
	done := make(chan bool)
	go func() {
		doGet(1)
		done <- true
	}()
	<-done
}
```

##### 多个任务使用slice chan bool

```go
func manyChannelWait() {
	var chs []chan bool
	count := 20
	for i := 0; i < count; i++ {
		chs = append(chs, make(chan bool))
	}

	for i := range chs {
		go func(i int) {
			doGet(i)
			chs[i] <- true
		}(i)
	}

	for _, ch := range chs {
		<-ch
	}
}
```



##### 使用标准库中的 sync.WaitGroup 来实现等待

```go
func main() {
	var wg sync.WaitGroup
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func(i int) {
			doGet(i)
			wg.Done()
		}(i)
	}
	wg.Wait()

	fmt.Println("channel wait")
	channelWait()

	fmt.Println("many channel wait")
	manyChannelWait()
}
```



### 从返回的 channel 中获取数据

```go
package main

import "fmt"

type node struct {
	value       int
	left, right *node
}

func (this *node) print() {
	fmt.Printf("%d ", this.value)
}

func (this *node) traverseFunc(f func(*node)) {
	if this == nil {
		return
	}
	this.left.traverseFunc(f)
	f(this)
	this.right.traverseFunc(f)
}

func (this *node) traverseWithChannel() chan *node {
	out := make(chan *node)
	go func() {
		this.traverseFunc(func(node *node) {
			out <- node
		})
		close(out)
	}()
	return out
}

func main() {
	root := node{value: 3}
	root.left = &node{value: 0}
	root.right = &node{value: 2}
	root.right.left = &node{value: 4}
	root.right.right = &node{value: 5}

	root.traverseFunc(func(n *node) {
		n.print()
	})

	fmt.Println()
	out := root.traverseWithChannel()
	for n := range out {
		n.print()
	}
	fmt.Println()

}
```



###  Select

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func main() {
	var c1, c2 = generator(), generator()
	// 规定程序总运行时间
	total := time.After(10 * time.Second)
	var values []int
	// worker := createWorker()
	for {
		var activeChan chan int
		var activeValue int
		if len(values) > 0 {
			activeValue = values[0]
			activeChan = createWorker()
		}
		select {
		case v := <-c1:
			values = append(values, v)
			// fmt.Println("received from c1: ", v)
		case v := <-c2:
			values = append(values, v)
			// fmt.Println("received from c2: ", v)
		case <-time.After(800 * time.Millisecond): // 每两次select 之间的间隔超过 500 毫秒
			fmt.Println("timeout, len(values):", len(values))
		case <-total:
			fmt.Println("bye")
			return
		case activeChan <- activeValue:
			values = values[1:]
		}
	}
}

func generator() chan int {
	out := make(chan int)
	go func() {
		i := 0
		for {
			time.Sleep(time.Duration(rand.Intn(1500)) * time.Millisecond)
			out <- i
			i++
		}
	}()
	return out
}

func createWorker() chan int {
	out := make(chan int)
	go func() {
		fmt.Println("received", <-out)
	}()
	return out
}

```





### 传统的同步机制

```go
package main

import (
	"fmt"
	"sync"
)

type atomicInt struct {
	value int
	lock  sync.Mutex
}

func (this *atomicInt) increment() {
	fmt.Println("atomic int increment")
	// 将 lock 和 unlock 控制在匿名函数体中
	func() {
		this.lock.Lock()
		defer this.lock.Unlock()
		this.value++
	}()
}

func (this *atomicInt) get() int {
	this.lock.Lock()
	defer this.lock.Unlock()
	return this.value
}

func main() {
	done := make(chan bool)
	var at atomicInt
	at.increment()
	go func() {
		at.increment()
		done <- true
	}()
	<-done
	fmt.Println(at.get())
}

```

