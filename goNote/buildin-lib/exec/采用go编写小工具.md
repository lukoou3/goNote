
# 采用go编写小工具
http://www.manongjc.com/detail/51-lwnxtlxmiadrifi.html

前段时间换了个环境写代码，但公司由于信息安全限制，要求大部分人员使用windows系统，于是我只能硬着头皮用windows。用windows的过程中发现一个很不便的地方，以前用类Unix系统，可以很方便写脚本完成一些小任务，但在windows里就变得很麻烦。解决方案有好几种：

* 使用cygwin之类的bash环境模拟器。但涉及windows命令与cygwin里的命令互操作时，会出现一些问题，解决起来很麻烦。
* 使用微软的powershell写脚本。不太想学一门新的类bash脚本语言。
* 装上python，写python脚本。这似乎是个好办法，可对python语言不太熟。


最后想了下，之前用过Go，可以用它来写小工具，试了试还挺好使的，下面举几个小例子。

自动生成hexo博客静态文件:
```go
# 定义变量
val1 = tf.placeholder(tf.float32, [None, 784])

...

# 初始化所有变量
init = tf.global_variables_initializer()

with tf.Session() as sess:
    # 在图中初始化所有变量
    sess.run(init)
    ...
```

部署hexo博客
```go
package main

import (
	"path/filepath"
	"os/exec"
	"os"
	"log"
)

// 此命令工具用于将hexo部署至服务器
func main() {
	blogDir := filepath.Join(`W:`, `gits`, `blog`)

	deployBlog(blogDir)
}
func deployBlog(blogDir string) {
	cmd := exec.Command(`hexo`, `deploy`)
	cmd.Dir = blogDir
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr
	err := cmd.Run()
	if err != nil {
		log.Fatal(err)
	}
}
```

杀Tomcat
```go
package main

import (
	"os/exec"
	"log"
	"bufio"
	"strings"
	"io"
)

// 此命令工具杀掉意外未死的tomcat进程
// idea有时候意外退出，此时开发用的tomcat服务器还在运行
func main() {
	cmd := exec.Command(`netstat`, `-ano`)
	output, err := cmd.StdoutPipe()
	if err != nil {
		log.Fatal(err)
	}
	go func() {
		cmd.Run()
	}()

	processOutput(output)
}

func processOutput(output io.ReadCloser) {
	scanner := bufio.NewScanner(output)
	for scanner.Scan() {
		line := scanner.Text()
		if hasTomcatPort(line) {
			parts := strings.Fields(line)
			tomcatPid := parts[len(parts) -1]
			log.Println(tomcatPid)
			killCmd := exec.Command(`taskkill`, `/F`, `/PID`, tomcatPid)
			killCmd.Run()
		}
	}
}

func hasTomcatPort(line string) bool {
	return strings.Contains(line, `LISTENING`) && (strings.Contains(line, `8080`) || strings.Contains(line, `1099`))
}
```

总结:
Go语言很精练，用来写这些小工具很合适。同时也可以经常用Go写点小东西练练手，以免长时间不用忘记了Go的玩法。
