# jenkins代码发布

## git

### 安装git

- Linux

```bash
yum install git -y
```

- Windows

```bash
https://git-scm.com/download/win
```

- mac os

```bash
https://git-scm.com/download/mac
```

### 创建git目录

```bash
mkdir xiangmu					[[创建项目名称目录]]
cd xiangmu

git init						[[目录内执行初始化命令，将其设为git库]]

初始化空的 Git 版本库于 /root/xiangmu/.git/
```

### 创建项目文件

```bash
vim index.html

1.项目完成20%
```

- 设置文件颜色显示`git config --global color.ui true`
- 关闭文件颜色显示`git config --global color.ui false`
- 管理目录下文件状态`git status `
- 将该文件提交到缓存区`git add <file>`
- 撤销提交`git rm cached <file>`
- 生成版本信息提交到本地仓库`git commit -m "描述信息"`
- 查看项目`git log`

### 配置个人信息

```bash
git config --global user.email "hope202091@163.com"					[[配置个人邮箱]]
git config --global user.name "oldli"							   [[配置用户名称]]
```

### 管理指定文件

```bash
git add index.html
```

### 生成版本

```bash
git commit -m "项目20% --v1"

[master（根提交） 631773c] 项目20% --v1
 1 file changed, 1 insertion(+)
 create mode 100644 index.html
```

```bash
vim index.html 

1.项目完成20%

2.项目完成100%
```

### 添加版本

```bash
git add .

git commit -m "项目完成100% --v2"

[master 37bb9fa] 项目完成100%
 2 files changed, 5 insertions(+)
 create mode 100644 1
```

### 版本回退

```bash
git reflog 					[[查看项目ID]]
git reset -- hard ID号		[[回到ID对应版本]]

重置后撤出暂存区的变更：
D	1
```

### 创建分支

```bash
git branch 分支名称
```

- 切换到分支中

```bash
git checkout dev
```

- 创建项目文件

```bash
vim shangcheng.py

1.商城系统
```

- 合并分支

```bash
git checkout master
git merge dev

Already up-to-date.
```

### 运程仓库

- 登录gitee
- 创建仓库
- 导入仓库

![image-20220730134020967](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/image-20220730134020967.png)

```bash
git remote add origin git@gitee.com:hope202091/dou-yin.git
```

- 查看仓库

```bash
git remote -v

origin	git@gitee.com:hope202091/dou-yin.git (fetch)
origin	git@gitee.com:hope202091/dou-yin.git (push)
```

- 生成公钥

```bash
ssh-keygen
```

- 复制公钥

```bash
cat ~/.ssh/id_rsa.pub
```

- 添加到gitee的ssh公钥中

- 添加到gitee仓库

```bash
git push origin master
```

- 查看gitee仓库

![image-20220730133854766](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/image-20220730133854766.png)

#### Windows主机克隆仓库-ssh

![image-20220730133903258](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/image-20220730133903258.png)

- 创建密钥

![image-20220730133912263](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/image-20220730133912263.png)

![image-20220730133922021](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/image-20220730133922021.png)

- 添加到ssh

- 拷贝gitee仓库到本地

![image-20220730133927172](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/image-20220730133927172.png)

#### Windows主机克隆仓库-http

![image-20220730133932504](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/image-20220730133932504.png)

```bash
git clone https://gitee.com/hope202091/dou-yin.git
```

- 进入项目

```bash
cd dou-yin/
```

- 设定邮箱&用户名

```bash
git config --global user.email "hope202091@163.com"
git config --global user.name "李开开"
```

- 更新代码

```bash
vim index.html
```

- 上传本地仓库

```bash
git add .
git commit -m "在公司开发"
```

- 同步gitee仓库

```bash
git push origin master
```

- 查看gitee仓库

![image-20211023154903637](C:\Users\李开开\AppData\Roaming\Typora\typora-user-images\image-20211023154903637.png)

- Linux获取最新项目代码

```bash
git pull origin master
```

- linux上继续开发

```bash
vim index.html
```

- 推送

```bash
git add index.html
git commit -m "在家继续开发"
```

- 同步

```bash
git push origin master
```

- gitee查看

![image-20211023160046857](C:\Users\李开开\AppData\Roaming\Typora\typora-user-images\image-20211023160046857.png)

#### tags标签

- 为当前的代码打标签

```bash
git tag -a V2.0 -m "v2.0时代"				[[-m添加说明]]
```

- 同步标签到gitee

```bash
git push origin --tags
```

![image-20220730133940823](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/image-20220730133940823.png)

**为指定commitID打标签**

- 查看commitID

```bash
git reflog
```

- 打标签

```bash
git tag -a v1.0 7329904 -m "v1.0时代"
```

- 查看标签

```bash
git tag -l
```

- 推送

```bash
git push origin --tags
```

- gitee查看

![image-20220730133949695](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/image-20220730133949695.png)

---

#### git忽略文件

```bash
vim .gitignore

[[同步时忽略匹配文件]]

*.h
!a.h 
files/ 
*.py[c|a|d]
```

---

## gitlab

### 安装gitlab

- 安装gitlab所需依赖软件

```bash
yum install -y curl openssh-server postfix wget
```

- 下载gitlab

```bash
wget https://mirror.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-12.3.9-ce.0.el7.x86_64.rpm --no-check-certificate
```

- 安装gitlab

```bash
yum localinstall gitlab-ce-12.3.9-ce.0.el7.x86_64.rpm
```

- 配置gitlab域名

```bash
vim /etc/gitlab/gitlab.rb

external_url 'http://gitlab.likaikai.net'
```

- 配置Gitlab邮箱

```bash
vim /etc/gitlab/gitlab.rb
[[打开关于邮箱注释]]

gitlab_rails['gitlab_email_enabled'] = true
gitlab_rails['gitlab_email_from'] = 'hope202091@163.com'
gitlab_rails['gitlab_email_display_name'] = 'GITLAB-Admin'

gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.163.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "likaikai@163.com"
gitlab_rails['smtp_password'] = "VZLCBPZQWWJTNXVP"
gitlab_rails['smtp_domain'] = "163.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true
```

- 关闭gitlab组件

```bash
[[不需要的组件都false]]

prometheus['enable'] = false
prometheus['monitor_kubernetes'] = false alertmanager['enable'] = false node_exporter['enable'] = false redis_exporter['enable'] = false postgres_exporter['enable'] = false gitlab_exporter['enable'] = false prometheus_monitoring['enable'] = false
grafana['enable'] = false
```

- 初始化gitlab

```bash
gitlab-ctl reconfigure


gitlab-ctl status
gitlab-ctl stop
gitlab-ctl start
```

- 加入主机hosts

- 登录
- 初始用户名为root

### 汉化gitlab

- 下载汉化包

```bash
wgethttps://gitlab.com/xhang/gitlab/-/archive/v12.3.-zh/gitlab-v12.3.0-zh.zip
```

- 解压汉化包

```bash
unzip  gitlab-v12.3.0-zh.zip
```

- 停止gitlab

```bash
gitlab-ctl stop
```

- 汉化

```bash
\cp -r gitlab-v12.3.0-zh/* /opt/gitlab/embedded/service/gitlab-rails/			[[cp加]]\强制覆盖
```

- 重启gitlab

```bash
[[重新配置gitlab服务]]
gitlab-ctl reconfigure 
[[重启gitlab服务]] 
gitlab-ctl restart
```
