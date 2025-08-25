# 修改Linux命令提示符

## 修改Linux命令提示符教程

**[Linux](https://m.haicoder.net/linux/linux-tutorial.html)** 的命令提示符是可以修改的，要想修改 Linux 的命令提示符，我们可以通过设置 PS1 **[环境变量](https://m.haicoder.net/linux/linux-env-var.html)** 的值来达到修改 Linux 命令提示符的目的。

同时，Linux 的系统命令提示符还是可以设置颜色的， 如果我们仅仅修改环境变量，那么修改只是临时生效，重启就会丢失，要永久修改，我们需要写入配置文件。

## Linux修改命令提示符参数

| 参数 | 描述                                                     |
| ---- | -------------------------------------------------------- |
| `\d` | 代表日期，格式为 weekday month date，例如：”Mon Aug 1”   |
| `\H` | 完整的主机名称                                           |
| `\h` | 仅取主机的第一个名字                                     |
| `\T` | 显示时间为 24 小时格式，如：HH：MM：SS                   |
| `\t` | 显示时间为 12 小时格式 , 如：HH：MM：SS                  |
| `\A` | 显示时间为 12 小时格式：HH：MM                           |
| `\u` | 当前用户的账号名称                                       |
| `\v` | BASH 的版本信息                                          |
| `\w` | 完整的工作目录名称                                       |
| `\W` | 利用 basename 取得工作目录名称，所以只会列出最后一个目录 |
| `#`  | 下达的第几个命令                                         |
| `$`  | 提示字符，如果是 root 时，提示符为：# ，普通用户则为：$  |

## Linux命令提示符颜色信息表

| Font | background | color  |
| ---- | ---------- | ------ |
| 30   | 40         | 黑色   |
| 31   | 41         | 红色   |
| 32   | 42         | 绿色   |
| 33   | 43         | ×××    |
| 34   | 44         | 蓝色   |
| 35   | 45         | 紫红色 |
| 36   | 46         | 青蓝色 |
| 37   | 47         | 白色   |

## 临时修改Linux命令提示符详解

### 说明

我们可以通过系统环境变量 PS1 来修改命令提示符。

### 案例

Linux 系统默认的系统提示符的格式为 `[username@host 工作目录]# `，我们可以使用 echo 命令，查看当前的系统提示符，具体命令如下：

```
echo $PS1
```

运行后，终端输出如下：

![15_Linux修改命令提示符.png](https://m.haicoder.net/uploads/pic/server/linux/linux-file-dir/15_Linux%E4%BF%AE%E6%94%B9%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6.png)

此时，我们看到了系统默认的命令提示符，现在，我们使用如下命令，修改系统命令提示符，具体命令如下：

```
PS1="[\u@\h \W\t]\$ "
```

运行后，终端输出如下：

![16_Linux修改命令提示符.png](https://m.haicoder.net/uploads/pic/server/linux/linux-file-dir/16_Linux%E4%BF%AE%E6%94%B9%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6.png)

此时，我们看到了系统命令提示符已经被我们修改了，但这只是临时的修改，我们重启系统之后，该命令提示符就会丢失了，会再次变成系统默认的。

## 永久修改Linux命令提示符详解

### 说明

PS1 命令可以设置当前 shell 的命令提示符，是 shell 中的一个功能，但是 shell 也是一个程序，有进程的生命周期，他会随着进程生命周期结束而将保存在内存中的数据丢失，如果想保存配置，需要将赋值保存在文件中。

### 案例

我们要想命令提示符永久生效，我们只需要将其写入 `~/.bashrc` 配置文件即可，我们使用 vim 打开该文件，具体命令如下：

```
vim ~/.bashrc
```

运行后，终端输出如下：

![17_Linux修改命令提示符.png](https://m.haicoder.net/uploads/pic/server/linux/linux-file-dir/17_Linux%E4%BF%AE%E6%94%B9%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6.png)

现在，我们在该文件中加入如下内容：

```
PS1="haicoder(www.haicoder.net)# "
```

加入完成后，如下图所示：

![18_Linux修改命令提示符.png](https://m.haicoder.net/uploads/pic/server/linux/linux-file-dir/18_Linux%E4%BF%AE%E6%94%B9%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6.png)

现在，我们保存并退出 vim，然后使用 source 命令，重新加载配置文件，具体命令如下：

```
source ~/.bashrc
```

执行完成后，如下图所示：

![19_Linux修改命令提示符.png](https://m.haicoder.net/uploads/pic/server/linux/linux-file-dir/19_Linux%E4%BF%AE%E6%94%B9%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6.png)

我们看到，命令提示符已经被修改了，而且这是永久生效的，哪怕重启机器，还是存在的。

## 修改Linux命令提示符颜色

### 说明

我们可以通过上述表格的颜色，来修改命令提示符颜色。

### 案例

我们输入以下命令，来临时修改 Linux 命令提示符的颜色，具体命令如下：

```
PS1='[\[\e[32;40m\]\u@\w]\$ '
```

运行后，终端输出如下：

![20_Linux修改命令提示符.png](https://m.haicoder.net/uploads/pic/server/linux/linux-file-dir/20_Linux%E4%BF%AE%E6%94%B9%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6.png)

我们看到，命令提示符的颜色已经被我们修改了。

## 修改Linux命令提示符总结

要想修改 Linux 的命令提示符，我们可以通过设置 PS1 环境变量的值来达到修改 Linux 命令提示符的目的。

同时，Linux 的系统命令提示符还是可以设置颜色的， 如果我们仅仅修改环境变量，那么修改只是临时生效，重启就会丢失，要永久修改，我们需要写入配置文件。