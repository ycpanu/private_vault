# 1  配置
## 1.1 安装Claude Code
### 1.1.1 使用NPM安装（电脑安装了Node.js)

（1）打开网络代理（挂梯子）
（2）以管理员身份运行命令提示符（cmd）
	开始页搜索cmd，右键选择以管理员身份运行![461](assets/Claude%20Code+DeepSeek/file-20260524102233303.png)
（3）输入指令：
```
npm install -g @anthropic-ai/claude-code
```
	等待安装完成
（4）检查是否安装成功，输入下面指令，如果显示版本号则成功了
```
claude --version
```
### 1.1.2 使用PowerShell走代理
	略
## 1.2 接入DeepSeek模型
### 1.2.1 申请DeepSeek API
（1）打开deepseek官网，点击右侧的API开放平台![379](assets/Claude%20Code+DeepSeek/file-20260524104037330.png)
（2）可先充10快钱，这个月有活动，Token消费打2.5折，够用挺久了
（3）点击API keys，创建API，名称随便，然后复制key（这个可以先粘贴到自己的笔记本里，因为这个key只出现一次，忘记了只能创建一个新的key)
### 1.2.2 使用CC Switch接入
（1）打开CC Switch，选择claude模型，再点击右上角的加号![](assets/Claude%20Code+DeepSeek/file-20260524104749863.png)
（2）供应商选择DeepSeek![](assets/Claude%20Code+DeepSeek/file-20260524105207140.png)
（3）往下划找到API Key，填入自己刚生成的API Key
（4）再往下划，填入指定的模型
