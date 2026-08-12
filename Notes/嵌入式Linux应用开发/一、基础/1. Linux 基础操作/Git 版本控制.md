### 1. 初始化与配置 (Setup & Init)

在开始写任何项目代码之前，通常需要进行基础配置或拉取仓库。

- **配置全局用户名和邮箱**（每次提交都会记录这个信息）：
    ```
    git config --global user.name "Your Name"
    git config --global user.email "your.email@example.com"
    ```
    
- **初始化新仓库**（在当前目录创建一个全新的 Git 仓库）：
    ```
    git init
    ```
    
- **克隆远程仓库**（将云端的项目完整下载到本地）：
    ```
    git clone [url]
    ```
    
### 2. 日常开发流 (Daily Workflow)

这是每天写代码时循环执行的高频操作：写代码 -> 暂存 -> 提交。

- **查看当前状态**（随时查看哪些文件被修改了、哪些在暂存区）：
    ```
    git status
    ```
    
- **将修改添加到暂存区**：
    ```
    git add [file-name]   # 添加指定文件
    git add .             # 添加当前目录下的所有更改（最常用）
    ```
    
- **提交到本地仓库**（给这次更改写一句清晰的描述）：
    ```
    git commit -m "feat: 增加用户登录页面的 UI"
    ```
    
### 3. 分支管理 (Branching)

在开发新功能（比如为小程序增加一个新模块）或修复 Bug 时，强烈建议在独立的分支上进行，避免破坏主分支的稳定性。

- **查看本地分支**（带 * 的是当前所在分支）：
    ```
    git branch
    ```
    
- **创建并切换到新分支**：
    ```
    git switch -c [branch-name]  # 推荐的新版命令
    # 或者老版命令：git checkout -b [branch-name]
    ```
    
- **切换回已存在的分支**（例如切回主分支 main）：
    ```
    git switch main
    ```
    
- **合并分支**（将开发分支的代码合并到当前所在的 main 分支）：
    ```
    git merge [branch-name]
    ```
    
### 4. 远程同步 (Remote Sync)

本地代码写好并 Commit 后，需要推送到远程仓库备份或与队友共享。

- **拉取远程最新代码**（写代码前建议先拉取，避免冲突）：
    ```
    git pull origin [branch-name]
    ```
    
- **推送到远程仓库**：
    ```
    git push origin [branch-name]
    ```
    
- **查看关联的远程仓库地址**：
    ```
    git remote -v
    ```
    
### 5. 历史查看与撤销 (History & Undo)

当代码写错、或者提交了不该提交的内容时，可执行下面命令回退。

- **查看提交历史**：
    ```
    git log --oneline  # 紧凑模式查看历史记录
    ```
    
- **撤销工作区的修改**（让文件恢复到最后一次提交的状态，**危险操作，未提交的代码会丢失**）：
    ```
    git restore [file-name]
    ```
    
- **撤销最新的 commit，但保留代码修改**（通常用于改错 commit 信息或漏交了文件）：
    ```
    git reset --soft HEAD~1
    ```

### 6. Angular 规范的基本格式

每次提交的信息通常包含三个部分：Header（标题行）、Body（主体内容）和 Footer（页脚）。在实际日常开发中，**最常用且必填的只有 Header。**

```
<type>(<scope>): <subject>
// 空一行
<body>
// 空一行
<footer>
```
### 常用的类型前缀（Type）

`type` 是规范的核心，它用一个英文单词直接说明了这次提交的目的。以下是最常用的几种前缀：

- **`feat` (Feature):** 引入了新功能。
    
    - _示例:_ `feat: 增加用户微信授权登录功能`
        
- **`fix` (Bug Fix):** 修复了 Bug。
    
    - _示例:_ `fix: 修复购物车金额计算偶尔出现负数的问题`
        
- **`docs` (Documentation):** 仅修改了文档（如 README、注释等）。
    
    - _示例:_ `docs: 更新 README 中的项目启动步骤`
        
- **`style` (Style):** 仅修改了代码格式（空格、缩进、逗号等），不影响代码逻辑。
    
    - _示例:_ `style: 格式化全局 CSS 缩进`
        
- **`refactor` (Refactoring):** 代码重构（既不是新增功能，也不是修复 Bug 的代码变动）。
    
    - _示例:_ `refactor: 提取公共的 API 请求错误处理逻辑`
        
- **`perf` (Performance):** 优化了性能的代码变动。
    
    - _示例:_ `perf: 优化首页图片加载速度`
        
- **`test` (Test):** 增加或修改了测试用例。
    
    - _示例:_ `test: 添加商品详情页的单元测试`
        
- **`chore` (Chore):** 对构建过程、辅助工具（如 webpack 配置、依赖包更新）的变动。
    
    - _示例:_ `chore: 升级 React 版本至 v18`
        

### 作用域与简短描述（Scope & Subject）

- **`scope` (作用域 - 可选):** 用于说明本次 Commit 影响的范围。比如可以是你负责的具体的模块名称：`auth`、`dashboard`、`components` 等。
    
    - _示例:_ `feat(auth): 支持手机号验证码登录`
        
- **`subject` (简短描述 - 必填):** 对本次修改的简要总结。
    
    - _要求：_ 尽量使用祈使句（动词开头），首字母小写（如果是英文），句末不要加句号。