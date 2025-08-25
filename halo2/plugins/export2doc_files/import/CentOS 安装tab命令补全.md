CentOS 安装tab命令补全

\1. 安装epel 源

```bash
yum -y install epel-release
```

\2. 加快yum速度

```bash
yum -y install yum-plugin-fastestmirror
```

\3. 安装bash-completion

```bash
yum -y install bash-completion
```

```bash
yum -y install bash-completion-extras # CentOS 7 再多安装一个
```

\4. 生效
立即生效

```bash
source /etc/profile.d/bash_completion.sh 
```

或者退出终端重新登录