### 两个超强的Linux帮助命令

---

#### cheat

##### 安装

`Centos7.6上安装`

一. ##Centos上安装cheat命令
1.安装python

```bash
yum -y install python
```

2.安装pip

```bash
yum -y install python-pip 
[[如果失败的话先执行]]
yum -y install epel-release 
[[然后执行]]
yum -y install python-pip
```

3.安装git

```bash
yum -y install git
```

4.安装Python依赖的文件

```bash
pip install docopt pygments
```

5.从github克隆项目到本地、用不到

```bash
git clone https://github.com/chrisallenlane/cheat.git
```

6.安装cheat

```bash
sudo pip install cheat
```

7.查看版本号

```bash
cheat -v
```

8.使用

```bash
cheat {需要查询用法的命令}
```

---

`Ubunte20.04上安装`

1.安装python；

```bash
sudo apt-get install python
```

2.安装pip；

```bash
sudo apt-get install python-pip 
[[如果失败的话先执行]]
sudo apt-get install epel-release
[[然后执行]]
sudo apt-get install python-pip
```

3.安装git

```bash
sudo apt-get install git
```

4.安装Python依赖的文件

```bash
pip install docopt pygments
```

5.从github克隆项目到本地、用不到

```bash
git clone https://github.com/chrisallenlane/cheat.git
```

6.安装cheat

```bash
sudo pip install cheat
```

7.查看版本号

```bash
cheat -v
```

8.试一下效果吧，

```bash
cheat {需要查询用法的命令}
```

###### 常用配置

如果使用的是bash，则配置文件为.bashc
如果使用的是zsh，则配置文件为.zshrc

```bash
export EDITOR = /usr/bin/vim # 默认编辑器
export CHEATCOLOR=true   # 语法高亮
```

---



#### tldr

##### 安装

`Centos上安装`

1.安装npm

```bahs
sudo apt-get install npm
```

2.安装nodejs-legacy

```bash
sudo apt-get install nodejs-legacy
```

3.安装n模块更新

```bash
sudo npm install n -g
```

4.安装最新版的node，视情况执行

```bash
sudo n latest
```

5.安装tldr

```bash
sudo npm install -g tldr
```

---

`Ubunte20.04上安装`

1.安装python；

```bash
sudo apt-get install python
```

2.安装pip；

```bash
sudo apt-get install python-pip 
[[如果失败的话先执行]]
sudo apt-get install epel-release
[[然后执行]]
sudo apt-get install python-pip
```

3.安装git

```bash
sudo apt-get install git
```

4.安装Python依赖的文件

```bash
pip install docopt pygments
```

5.从github克隆项目到本地，用不到

```bash
git clone https://github.com/tldr-pages/tldr
```

6.安装cheat

```bash
sudo pip install tldr
```

7.查看版本号

```bash
tldr -v
```

8.试一下效果吧，

```bash
tldr {需要查询用法的命令}
```

---

