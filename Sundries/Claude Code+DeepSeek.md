# 1  配置

## 1.1 安装Claude Code

### 1.1.1 使用NPM安装（电脑安装了Node.js)

（1）打开网络代理（挂梯子)

（2）以管理员身份运行命令提示符（cmd）
	开始页搜索cmd，右键选择以管理员身份运行
	![251](assets/Claude%20Code+DeepSeek/file-20260524102233303.png)

（3）输入指令：
```
npm install -g @anthropic-ai/claude-code
```
等待安装完成

（4）检查是否安装成功，输入下面指令，如果显示版本号则成功了
```
claude -v
```

### 1.1.2 使用PowerShell走代理
	略

## 1.2 接入DeepSeek模型

### 1.2.1 申请DeepSeek API

（1）打开deepseek官网，点击右侧的API开放平台
![261](assets/Claude%20Code+DeepSeek/file-20260524104037330.png)

（2）可先充10快钱，这个月有活动，Token消费打2.5折，够用挺久了

（3）点击API keys，创建API，名称随便，然后复制key（这个可以先粘贴到自己的笔记本里，因为这个key只出现一次，忘记了只能创建一个新的key)

### 1.2.2 使用CC Switch接入

（1）打开CC Switch，选择claude模型，再点击右上角的加号
![442](assets/Claude%20Code+DeepSeek/file-20260524104749863.png)

（2）供应商选择DeepSeek
![428](assets/Claude%20Code+DeepSeek/file-20260524105207140.png)

（3）往下划找到API Key，填入自己刚生成的API Key

（4）修改模型配置，改成下图
![464](assets/Claude%20Code+DeepSeek/file-20260524105829214.png)

Haiku：
```
deepseek-v4-flash
```

其他：
```
deepseek-v4-pro[1m]
```

（5）点击右下角添加

（6）回到首页，测试一下是否配置成功
![370](assets/Claude%20Code+DeepSeek/file-20260524110457745.png)

（6）点击启用该模型
![337](assets/Claude%20Code+DeepSeek/file-20260524110259422.png)

（7）点击用量配置查询，进去打开启动用量配置查询，保存配置，就可以在首页显示自己的余额了
![502](assets/Claude%20Code+DeepSeek/file-20260524110820076.png)

### 1.2.3 使用终端指令配置（CC Swith配置不成功的情况下使用）
```
$env:ANTHROPIC_BASE_URL="https://api.deepseek.com/anthropic"
$env:ANTHROPIC_AUTH_TOKEN="你的 DeepSeek API Key"
$env:ANTHROPIC_MODEL="deepseek-v4-pro[1m]"
$env:ANTHROPIC_DEFAULT_OPUS_MODEL="deepseek-v4-pro[1m]"
$env:ANTHROPIC_DEFAULT_SONNET_MODEL="deepseek-v4-pro[1m]"
$env:ANTHROPIC_DEFAULT_HAIKU_MODEL="deepseek-v4-flash"
$env:CLAUDE_CODE_SUBAGENT_MODEL="deepseek-v4-flash"
$env:CLAUDE_CODE_EFFORT_LEVEL="max"
```

### 1.2.4 检查是否deepseek是否接入成功

（1）在任意文件夹终端下输入
```
claude
```

（2）选择yes
![311](assets/Claude%20Code+DeepSeek/file-20260524111859019.png)

（3）查看是否显示deepseek-v4-pro[1m]
![307](assets/Claude%20Code+DeepSeek/file-20260524112024777.png)


# 2  使用方法

## 2.1 cmd使用

### 2.1.1 指定文件夹启动

（1）打开文件管理器，去到需要用该模型的的地址
![459](assets/Claude%20Code+DeepSeek/file-20260524112741910.png)
如我的项目的所需要的文件存放到了指定的项目文件夹ExamScope下，我只需要打开这个文件夹

（2）在地址栏直接输入cmd，回车，就会到达指定文件夹的终端
![541](assets/Claude%20Code+DeepSeek/file-20260524113100047.png)

因为claude会读取你这个路径下的所有文件，文件夹选择就比较重要，如我的前后端代码是放到了code里面，如果我只要修改前后端代码之间的逻辑，我就再打开code文件夹，在code文件夹下打开终端，或者我只要修改我的后端代码，那我再前进一个，直接打开后端代码的文件夹，启动终端，这样就会只读取到后端代码，修改起来会在一定程度上避免ai上下文混乱

（3）终端输入claude，就可以对该文件夹下的文件进行改动了

### 2.1.2 终端使用cd指定文件夹

略

## 2.2 编译器终端启用

（1）在编译器（IDEA,VS Code等）的终端输入claude

## 2.3 VS Code插件使用

（1）下载插件

![571](assets/Claude%20Code+DeepSeek/file-20260524115745796.png)

（2）侧边栏打开后，新建一个session即可使用
