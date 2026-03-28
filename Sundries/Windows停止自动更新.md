### 一键开启 gpedit.msc 的步骤：

**1. 新建一个文本文档** 在电脑桌面空白处，点击鼠标右键 -> 选择“新建” -> “文本文档”（也就是记事本）。

**2. 复制并粘贴代码** 打开刚刚新建的文本文档，将下面这段代码完整地复制粘贴进去：

DOS

```
@echo off
pushd "%~dp0"
dir /b %SystemRoot%\servicing\Packages\Microsoft-Windows-GroupPolicy-ClientExtensions-Package~3*.mum >List.txt
dir /b %SystemRoot%\servicing\Packages\Microsoft-Windows-GroupPolicy-ClientTools-Package~3*.mum >>List.txt
for /f %%i in ('findstr /i . List.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i"
pause
```

**3. 另存为批处理文件**

- 点击记事本左上角的“文件” -> 选择“另存为...”。
    
- 在弹出的窗口底部，将“保存类型”改为 **“所有文件 (_._)”**。
    
- 将“文件名”修改为：**`开启组策略.bat`** （注意后缀必须是 `.bat`）。
    
- 点击“保存”，然后关掉记事本。
    

**4. 以管理员身份运行**

- 回到桌面，你会看到刚才保存的 `开启组策略.bat` 文件（图标通常是一个带有齿轮的窗口）。
    
- **关键一步**：在这个文件上点击**右键**，选择 **“以管理员身份运行”**。
    
- 此时会弹出一个黑色的命令提示符窗口，系统会自动开始配置进度条（可能会跑到 100% 几次），这个过程大概需要一两分钟。
    
- 等待直到窗口提示“请按任意键继续...”，按下回车键关闭窗口即可。

### 