## chatgpt接入微信-参考教程

**1、首先下载ubuntu20.04**

[https://mirrors.tuna.tsinghua.edu.cn/](https://www.buxiaoyao.com/?golink=aHR0cHM6Ly9taXJyb3JzLnR1bmEudHNpbmdodWEuZWR1LmNuLw==)

> **不明白可参考**
>
> 把ubuntu 22安装到U盘 [https://www.bilibili.com/video/BV1aD4y1y7o5/](https://www.buxiaoyao.com/?golink=aHR0cHM6Ly93d3cuYmlsaWJpbGkuY29tL3ZpZGVvL0JWMWFENHkxeTdvNS8=)

**项目开源地址**

[https://github.com/zhayujie/chatgpt-on-wechat](https://www.buxiaoyao.com/?golink=aHR0cHM6Ly9naXRodWIuY29tL3poYXl1amllL2NoYXRncHQtb24td2VjaGF0)

**Xshell**下载地址

[https://www.xshell.com/zh/xshell/](https://www.buxiaoyao.com/?golink=aHR0cHM6Ly93d3cueHNoZWxsLmNvbS96aC94c2hlbGwv)

## 开始安装，按顺序输入如下指令

```
git clone https://github.com/zhayujie/chatgpt-on-wechat [[克隆项目代码本地]]

cd chatgpt-on-wechat/                                   [[进入到chatgpt-on-wechat目录]]

sudo apt-get update                                     [[读取软件列表]]

sudo apt-get upgrade                                    [[更新软件]] 

sudo apt install python3-pip

pip3 install itchat-uos==1.5.0.dev0

pip3 install --upgrade openai -i https://pypi.tuna.tsinghua.edu.cn/simple/
```

注：itchat-uos使用指定版本1.5.0.dev0，openai使用最新版本，需高于0.25.0

## 配置

配置文件的模板在根目录的config-template.json中，需复制该模板创建最终生效的 config.json 文件：

```
cp config-template.json config.json

vi config.json [[打开配置文件，然后按下2gg定位到该行，通过按键h，向左移动，l向右移动位置。x键，删除字符，完成后按下ESC键，输入wq]] 回车。
```

然后在config.json中填入配置，以下是对默认配置的说明，可根据需要进行自定义修改：

```
# config.json文件内容示例
{ 
  "open_ai_api_key": "YOUR API KEY"                           # 填入上面创建的 OpenAI API KEY
  "single_chat_prefix": ["bot", "@bot"],                      # 私聊时文本需要包含该前缀才能触发机器人回复
  "single_chat_reply_prefix": "[bot] ",                       # 私聊时自动回复的前缀，用于区分真人
  "group_chat_prefix": ["@bot"],                              # 群聊时包含该前缀则会触发机器人回复
  "group_name_white_list": ["ChatGPT测试群", "ChatGPT测试群2"], # 开启自动回复的群名称列表
  "image_create_prefix": ["画", "看", "找"],                   # 开启图片回复的前缀
  "conversation_max_tokens": 1000,                            # 支持上下文记忆的最多字符数
  "character_desc": "你是ChatGPT, 一个由OpenAI训练的大型语言模型, 你旨在回答并解决人们的任何问题，并且可以使用多种语言与人交流。"  # 人格描述
}
```

## 获取key

[https://platform.openai.com/example](https://www.buxiaoyao.com/?golink=aHR0cHM6Ly9wbGF0Zm9ybS5vcGVuYWkuY29tL2V4YW1wbGVz)

## 运行

运行方式，1和2任选其一，作用一样。

### 1.如果是开发机 本地运行，直接在项目根目录下执行：

```
touch nohup.out                                   # 首次运行需要新建日志文件  

python3 app.py                                    # 或在后台运行,nohup python3 app.py & tail -f nohup.out
```

终端输出二维码后，使用微信进行扫码，当输出 “Start auto replying” 时表示自动回复程序已经成功运行了（注意：用于登录的微信需要在支付处已完成实名认证）。扫码登录后你的账号就成为机器人了，可以在微信手机端通过配置的关键词触发自动回复 (任意好友发送消息给你，或是自己发消息给好友)。

### 2.如果是服务器部署，则使用nohup命令在后台运行：

```
touch nohup.out                                   # 首次运行需要新建日志文件                     

nohup python3 app.py & tail -f nohup.out          # 在后台运行程序并通过日志输出二维码
```

扫码登录后程序即可运行于服务器后台，此时可通过 ctrl+c 关闭日志，不会影响后台程序的运行。使用 ps -ef | grep app.py | grep -v grep 命令可查看运行于后台的进程，如果想要重新启动程序可以先 kill 掉对应的进程。日志关闭后如果想要再次打开只需输入 tail -f nohup.out。



注：如果 扫码后手机提示登录验证需要等待5s，而终端的二维码再次刷新并提示 Log in time out, reloading QR code，此时需参考此 issue 修改一行代码即可解决



注：如果 扫码后手机提示登录验证需要等待5s，而终端的二维码再次刷新并提示 Log in time out, reloading QR code，此时需参考此 issue 修改一行代码即可解决

## 说明

**个人聊天**

- 个人聊天中，需要以 “bot”或”@bot” 为开头的内容触发机器人，对应配置项 `single_chat_prefix` (如果不需要以前缀触发可以填写 `"single_chat_prefix": [""]`)
- 机器人回复的内容会以 “[bot] ” 作为前缀， 以区分真人，对应的配置项为 `single_chat_reply_prefix` (如果不需要前缀可以填写 `"single_chat_reply_prefix": ""`)

**群组聊天**

- 群组聊天中，群名称需配置在 `group_name_white_list ` 中才能开启群聊自动回复。如果想对所有群聊生效，可以直接填写 `"group_name_white_list": ["ALL_GROUP"]`
- 默认只要被人 @ 就会触发机器人自动回复；另外群聊天中只要检测到以 “@bot” 开头的内容，同样会自动回复（方便自己触发），这对应配置项 `group_chat_prefix`
- 可选配置: `group_name_keyword_white_list`配置项支持模糊匹配群名称，`group_chat_keyword`配置项则支持模糊匹配群消息内容，用法与上述两个配置项相同。（Contributed by [evolay](https://www.buxiaoyao.com/?golink=aHR0cHM6Ly9naXRodWIuY29tL2V2b2xheQ==))

**其他配置**

- 对于图像生成，在满足个人或群组触发条件外，还需要额外的关键词前缀来触发，对应配置 `image_create_prefix`
- 关于OpenAI对话及图片接口的参数配置（内容自由度、回复字数限制、图片大小等），可以参考 [对话接口](https://www.buxiaoyao.com/?golink=aHR0cHM6Ly9iZXRhLm9wZW5haS5jb20vZG9jcy9hcGktcmVmZXJlbmNlL2NvbXBsZXRpb25z) 和 [图像接口](https://www.buxiaoyao.com/?golink=aHR0cHM6Ly9iZXRhLm9wZW5haS5jb20vZG9jcy9hcGktcmVmZXJlbmNlL2NvbXBsZXRpb25z) 文档直接在 [代码](https://www.buxiaoyao.com/?golink=aHR0cHM6Ly9naXRodWIuY29tL3poYXl1amllL2NoYXRncHQtb24td2VjaGF0L2Jsb2IvbWFzdGVyL2JvdC9vcGVuYWkvb3Blbl9haV9ib3QucHk=) `bot/openai/open_ai_bot.py` 中进行调整。
- `conversation_max_tokens`：表示能够记忆的上下文最大字数（一问一答为一组对话，如果累积的对话字数超出限制，就会优先移除最早的一组对话）
- `character_desc` 配置中保存着你对机器人说的一段话，他会记住这段话并作为他的设定，你可以为他定制任何人格 (关于会话上下文的更多内容参考该 [issue](https://www.buxiaoyao.com/?golink=aHR0cHM6Ly9naXRodWIuY29tL3poYXl1amllL2NoYXRncHQtb24td2VjaGF0L2lzc3Vlcy80Mw==))





##### 可能出现的问题

二维码扫描时间不够

使用下面文件路径修改文件内容，加上一个停止15秒

time.sleep(15)

在wnile循环上面一行  注意缩进



```bash
cat /usr/local/lib/python3.6/site-packages/itchat/components/login.py
```

![image-20230328095859544](/Users/likaikai/Library/Application Support/typora-user-images/image-20230328095859544.png)