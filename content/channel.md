# channel通道实战
> 此开源图书由[thaiq](https://github.com/ithaiq)原创，创作不易转载请注明出处

* channel搭配管道pipeline提高并发

```go
type Cmd func(list []int) chan int
type PipeCmd func(in chan int) chan int //支持管道的函数

//求偶数
func Event(list []int) chan int {
	c := make(chan int)
	go func() {
		defer close(c)
		for _, num := range list {
			if num%2 == 0 {
				c <- num
			}
		}
	}()
	return c
}

//乘以2
func M2(in chan int) chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for num := range in {
			out <- num * 2
		}
	}()
	return out
}

//乘以10
func M10(in chan int) chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for num := range in {
			out <- num * 10
		}
	}()
	return out
}

//管道
func Pipe(args []int, c1 Cmd, cs ...PipeCmd) chan int {
	ret := c1(args)
	if len(cs) == 0 {
		return ret
	}
	retlist := make([]chan int, 0)
	for index, c := range cs {
		if index == 0 {
			retlist = append(retlist, c(ret))
		} else {
			getChan := retlist[len(retlist)-1]
			retlist = append(retlist, c(getChan))
		}
	}
	return retlist[len(retlist)-1]
}

func Test(nums []int) {
	ret := Pipe(nums, Event, M2, M10)
	for r := range ret {
		fmt.Printf("%d ", r)
	}
}

func main() {
	nums := []int{2, 3, 6, 12, 22, 16, 4, 9, 23, 64, 62}
	start := time.Now().Unix()
	Test(nums)
	end := time.Now().Unix()
	fmt.Printf("测试--用时:%d秒\r\n", end-start)
}
```

* channel使用多路复用利用多核提高并发，[通用代码](https://github.com/ithaiq/go-gitbook/tree/master/code/channel/base_pipe.go)，[测试用例](https://github.com/ithaiq/go-gitbook/tree/master/code/channel/base_pipe_test.go)

```go
//多路复用
func Pipe2(args []int, c1 Cmd, cs ...PipeCmd) chan int {
	ret := c1(args)
	out := make(chan int)
	wg := sync.WaitGroup{}
	for _, c := range cs {
		getChan := c(ret)
		wg.Add(1)
		go func(input chan int) {
			defer wg.Done()
			for v := range input {
				out <- v
			}
		}(getChan)
	}
	go func() {
		defer close(out)
		wg.Wait()
	}()
	return out
}
func Test(nums []int) {
	ret := Pipe2(nums, Event, M2, M2)//相当于2个消费者
	for r := range ret {
		fmt.Printf("%d ", r)
	}
}

func main() {
	nums := []int{2, 3, 6, 12, 22, 16, 4, 9, 23, 64, 62}
	start := time.Now().Unix()
	Test(nums)
	end := time.Now().Unix()
	fmt.Printf("测试--用时:%d秒\r\n", end-start)
}
```
---
* 部分坑

  * 超时控制搭配channel
  ```go
    func test(d time.Duration, f func() error, success string, fail string) error {
        ctx, cancel := context.WithTimeout(context.Background(), d)
        defer cancel()
        for {
            select {
            case <-ctx.Done():
                return fmt.Errorf(fail)
            default:
                err := f()
                if err == nil {
                    logger.Info(success)
                    return nil
                } else {
                    return err
                }
            }
        }
    }
  ```
  如果f()里面有耗时操作context超时限制失效，修改如下即可
  ```go
    func test(d time.Duration, f func() error, success string, fail string) error {
        ctx, cancel := context.WithTimeout(context.Background(), d)
        defer cancel()
        ch := make(chan error)
        go func() {
            ch <- f()
            close(ch)
        }()
        for {
            select {
            case <-ctx.Done():
                return fmt.Errorf(fail)
            case err := <-ch:
                f err == nil {
                    logger.Info(success)
                    return nil
                } else {
                    return err
                }
            }
        }
    }
  ```