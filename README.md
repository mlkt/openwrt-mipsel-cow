# openwrt-mipsel-cow

这是 https://github.com/cyfdecyf/cow 的mipsel编译版本，用于openwrt上运行，使用方法与普通Linux下相同。

经测试可于极路由2上运行，其他路由器待确认，理论上mipsel的路由器应该都可运行。

运行效果：
https://github.com/cyfdecyf/cow/issues/400

# mipsel的gccgo工具链的使用

mipsel-linux-gcc-5.3.0.tar.gz是使用gcc 5.3.0源代码编译的含有gccgo和gotool的toolchain，可以用来编译go的程序使其可运行于mipsel平台（也可编译C与C++代码），我在Ubuntu Server 14.10（32位）下编译出的这个工具链，使用方法：

首先解压mipsel-linux-gcc-5.3.0.tar.gz，比如我在/root中使用：

```
cd /root
tar zxf mipsel-linux-gcc-5.3.0.tar.gz
```

将工具链加入环境变量：

```
export PATH=/root/mipsel-linux-gcc-5.3.0/bin:$PATH
```

这样就可以使用mipsel-linux-gccgo和mipsel-linux-go了，

可以先写一个helloWorld用gccgo编译试试：

```
package main

import "fmt"

func main() {
    fmt.Println("Hello world!")
}
```

```
mipsel-linux-gccgo test.go -static -o test
```

一定要使用-static静态链接，编译完成后传到路由器上，加上执行权限（chmod +x test），就可以运行了。

编译一个go项目与编译一般的go项目类似，只是把go build换成mipsel-linux-go build而已，这里用cow项目举例，首先设置GOPATH（我用的是/root/golang）：

```
export GOPATH=/root/golang
```

下载cow的代码（如果你已经有了当然不用下）：

```
mipsel-linux-go get github.com/cyfdecyf/cow
```

或者（如果你已经安装了普通的go编译器）：

```
go get github.com/cyfdecyf/cow
```

注意golang.org被墙，可能下载失败，怎么翻墙自行解决。

下载完成后，进行编译，编译有两种方法：

①像编译C代码那样，自行编写Makefile，运行mipsel-linux-gccgo，编译每一个.go源码文件，gccgo的参数和gcc基本一样。

②进入项目目录，与编译一般的go项目类似，运行mipsel-linux-go build即可。mipsel-linux-go会自行分析GOPATH下的代码依赖，自动调用mipsel-linux-gccgo进行编译并链接。

显然第二种方法最简单：

```
cd /root/golang/src/github.com/cyfdecyf/cow
mipsel-linux-go build -gccgoflags '-O3 -static'
```

编译成功的话/root/golang/src/github.com/cyfdecyf/cow目录下就会出现一个叫cow的可执行文件，上传至路由器添加执行权限即可运行。

要注意的几个点：

使用mipsel-linux-go build编译时，-gccgoflags是指传递给gccgo的参数，一定要加上-static使用静态链接，也可以加上-O0、-O1、-O2、-O3等优化参数，总之和gcc有什么参数gccgo也基本有什么参数。

使用静态链接编译出来的程序很大，但动态链接我试了很久都不能运行，我明明把动态库也复制到了路由器上，可运行时死活找不到，原因不明。

编译出来的go程序必须是not stripped的（也就是编译时不能加-s参数），否则也不能运行，原因也不明。
