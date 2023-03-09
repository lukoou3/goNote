
# exec.Command 执行命令用法实例

https://blog.csdn.net/whatday/article/details/109277998

### cmd字段介绍


```go
type Cmd struct {  
    Path         string　　　//运行命令的路径，绝对路径或者相对路径  
    Args         []string　　 // 命令参数  
    Env          []string         //进程环境，如果环境为空，则使用当前进程的环境  
    Dir          string　　　//指定command的工作目录，如果dir为空，则comman在调用进程所在当前目录中运行  
    Stdin        io.Reader　　//标准输入，如果stdin是nil的话，进程从null device中读取（os.DevNull），stdin也可以时一个文件，否则的话则在运行过程中再开一个goroutine去  
　　　　　　　　　　　　　//读取标准输入  
    Stdout       io.Writer       //标准输出  
    Stderr       io.Writer　　//错误输出，如果这两个（Stdout和Stderr）为空的话，则command运行时将响应的文件描述符连接到os.DevNull  
    ExtraFiles   []*os.File 　　  
    SysProcAttr  *syscall.SysProcAttr  
    Process      *os.Process    //Process是底层进程，只启动一次  
    ProcessState *os.ProcessState　　//ProcessState包含一个退出进程的信息，当进程调用Wait或者Run时便会产生该信息．  
}  
```


### 用法一:直接在当前目录使用并返回结果


```go
func Cmd(commandName string, params []string) (string, error) {
    cmd := exec.Command(commandName, params...)
    fmt.Println("Cmd", cmd.Args)
    var out bytes.Buffer
    cmd.Stdout = &out
    cmd.Stderr = os.Stderr
    err := cmd.Start()
    if err != nil {
        return "", err
    }
    err = cmd.Wait()
    return out.String(), err
}
```


### 用法二：在命令位置使用并返回结果


```go
func CmdAndChangeDir(dir string, commandName string, params []string) (string, error) {
    cmd := exec.Command(commandName, params...)
    fmt.Println("CmdAndChangeDir", dir, cmd.Args)
    var out bytes.Buffer
    cmd.Stdout = &out
    cmd.Stderr = os.Stderr
    cmd.Dir = dir
    err := cmd.Start()
    if err != nil {
        return "", err
    }
    err = cmd.Wait()
    return out.String(), err
}
```


### 用法三：在命令位置使用并实时输出每行结果


```go
func CmdAndChangeDirToShow(dir string, commandName string, params []string) error {
  cmd := exec.Command(commandName, params...)
  fmt.Println("CmdAndChangeDirToFile", dir, cmd.Args)
  //StdoutPipe方法返回一个在命令Start后与命令标准输出关联的管道。Wait方法获知命令结束后会关闭这个管道，一般不需要显式的关闭该管道。
  stdout, err := cmd.StdoutPipe()
  if err != nil {
    fmt.Println("cmd.StdoutPipe: ", err)
    return err
  }
  cmd.Stderr = os.Stderr
  cmd.Dir = dir
  err = cmd.Start()
  if err != nil {
    return err
  }
  //创建一个流来读取管道内内容，这里逻辑是通过一行一行的读取的
  reader := bufio.NewReader(stdout)
  //实时循环读取输出流中的一行内容
  for {
    line, err2 := reader.ReadString('\n')
    if err2 != nil || io.EOF == err2 {
      break
    }
    fmt.Println(line)
  }
  err = cmd.Wait()
  return err
}
```


### 用法四：在命令位置使用并实时写入每行结果到文件


```go
func CmdAndChangeDirToFile(fileName, dir, commandName string, params []string) error {
    Path, err := GetCurrentPath()
    if err != nil {
        return err
    }
    Path += "logs/cmdDir/"
    EnsureDir(Path)
    f, err := os.Create(Path + fileName) //创建文件
    defer f.Close()
    cmd := exec.Command(commandName, params...)
    fmt.Println("CmdAndChangeDirToFile", dir, cmd.Args)
    //StdoutPipe方法返回一个在命令Start后与命令标准输出关联的管道。Wait方法获知命令结束后会关闭这个管道，一般不需要显式的关闭该管道。
    stdout, err := cmd.StdoutPipe()
    if err != nil {
        fmt.Println("cmd.StdoutPipe: ", err)
        return err
    }
    cmd.Stderr = os.Stderr
    cmd.Dir = dir
    err = cmd.Start()
    if err != nil {
        return err
    }
    //创建一个流来读取管道内内容，这里逻辑是通过一行一行的读取的
    reader := bufio.NewReader(stdout)
    //实时循环读取输出流中的一行内容
    for {
        line, err2 := reader.ReadString('\n')
        if err2 != nil || io.EOF == err2 {
            break
        }
        _, err = f.WriteString(line) //写入文件(字节数组)
        f.Sync()
    }
    _, err = f.WriteString("=================处理完毕========================") //写入文件(字节数组)
    f.Sync()
    err = cmd.Wait()
    return err
}
```


### 用法五：bash -c 方式执行

```go
// 后台运行一个命令 bash -c 方式
func CmdBash(commandName string) *exec.Cmd {
	cmd := exec.Command("/bin/bash", "-c", commandName)
	fmt.Println(line)(cmd.Args)
 
	go func() {
		var out bytes.Buffer
		cmd.Stdout = &out
		cmd.Stderr = os.Stderr
		_ = cmd.Run()
	}()
 
	return cmd
}
```

