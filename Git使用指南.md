# Git 的安装与使用	

我们直接前往 Git 官网 <https://git-scm.com/install/windows> 下载最新的安装包，根据你的系统进行选择，默认都是 Windows 系统。

选择 “Git for Windows/x64 Setup” 安装包，下载解压运行安装程序即可。

Git 包含两个，分别为：

* Git GUI : Git 的图形化界面；
* [Git Bash](#Git Bash 与 Shell 程序解释说明): Git 的 CLI (Command Line Interface) 命令行界面；

我们可在任何位置右键鼠标，找到上述两个选项，通常我们都是使用 Git Bash 命令行操作界面，具体如下所示。

<img src="C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20260509164617501.png" alt="image-20260509164617501" style="zoom: 67%;" />

由于 Git 版本管理工具本身就是开发用于管理 Linux 工程的，所以很多操作与 Linux 系统一致，感兴趣的可以稍微了解一下 Linux 操作系统。

光标前面的 “**$**” 符号是提示符，会在命令行前面自动出现，在此符号后面输入对应的操作指令，就能够执行内部预置的操作，取消掉了任何冗余图形化界面以及控件，用最原始的字符反馈信息，这就是 CLI 。

> 注：在 **AI Agent** 正盛的时代下，以及不断追求返璞归真，很多的配置都是直接通过 CLI 来进行交互的，所以学习使用 CLI 仍然具备优势。 

目前仍有许多的适配 Git 的图形化操作App，例如：**SourceTree**，**TortoiseGit** ，或者是集成在 IDE 中的扩展，如：VSCode 的 SourceControl 都是非常适合新手使用的。

# Git 的文件管理机制

## Git 项目的三个阶段

git 的文件管理划分为以下三大区域，管理的文件便是此三个区域之间进行传递，从而实现版本管理的目的。

### 工作区 - Working  Directory

对项目的某个版本独立提取出来的内容，```git checkout``` 即将仓库中的内容提取到工作区。

### 暂存区 - Staging Area / Index

是一个包含了下次要提交的文件列表信息文件。执行 [```git add```](#git add)时，暂存区便更新和记录即将要提交到仓库的文件列表。但是在 Git 中的术语称为 Index 索引。

### 版本库 - Repository

即 .git 目录本身，是所有 git 版本管理相关内容的核心存储区域，不可以人为的进行修改变更，否则破坏当前的版本管理有效性。

## Git 文件的三种状态

### 已提交 - Commited

表示数据已经安全的保存在本地数据库中；

### 已修改 - Modified

表示修改了文件，但是还没有保存到数据库；

### 已暂存 - Staged

表示对一个已修改文件的当前版本做了标记，使其包含在下一次提交的快照中。

> **未跟踪** - **Untracked** ：表示未被 Git 进行版本管理的文件。上述的三种文件状态均为 **Tracked** 跟踪的状态，已经被版本管理工具进行管理。

# Git 常用指令

Git 的官方文档 <https://git-scm.com/docs> 中有对所有的 Git 指令提供了说明参考，同时还有一个官方编写的 Git 使用电子书 <https://git-scm.com/book/en/v2> ，介绍的非常全面，有简中版本。

Git 指令很多但是实际工程中最常用的指令也就不到十几个。如果对具体的 Git 指令不知道如何使用，可以使用 ```git -h```或者 ```git --help``` 指令查看帮助指南。

<img src="C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20260509140432338.png" alt="image-20260509140432338" style="zoom:67%;" />

从 Git 的帮助指南可以看到 usage 提供了 git 指令语法的格式，对于这些格式具体的符号含义如下：

* \[ ] ：表示可选项，非必须；
* \< > ：必填内容的占位符，需要填入实际值，如\<file>指需要填入具体文件路径；
* | ：表示互斥选项，两者取其一；
* … ：表示可以存在多项； 
* = ：表示该选项后续赋值，指令需要包含该符号；
* ()：表示元素分组，通常配合 "|"使用；

示例：

```bash
#添加远程仓库语法示例
git remote add [<options>] <name> <url>
```

表示：添加远程仓库指令，可以添加选项，然后必须指明远程仓库的名称，通常都命名为 "origin"，并且附带远程仓库的 url 。 

> 注：对于下面提及的所有指令，如果想要了解更多的操作，可以查看官网说说明文档，或者使用 ``` git <指令> [-h/--help] ```查看所有的用法选项。

## Git 初始化

### git config

用于配置配置文件，配置文件内容包含了关于如何响应 git 指令的相关配置；

通常在 git 安装成功后，进入 Git Bash 界面后，我们首先做的第一件事就是配置我们的用户名，即等于在本地注册一个用户。指令通常包括：

```bash
 #列出当前配置文件的变量极其值
 git config list
 
 #设置全局配置文件中的用户名变量
 git config set --globle user.name "<username>"
 
 #设置全局配置文件中的用户邮箱变量
 git config set --globle user.email <email-addrese>
 
 #查看当前的 user.name 变量值
 git config get user.name
 
 #查看当前的 user.email 变量值
 git config get user.email
```

其中的 --globle 选项表明配置的文件为全局配置文件，其作用范围在每个仓库的自带的 config 文件之上。配置后可通过输入 ``` git config list``` 指令查看是否配置成功。

### git init

用于初始化一个空的 Git 仓库，或者重新初始化现有仓库；会在当前的工作目录下产生一个 .git 隐藏文件夹。

> 如果没看到 .git 文件夹可以如下图所示，打开显示隐藏内容；
>
> <img src="C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20260516213928831.png" alt="image-20260516213928831" style="zoom: 67%;" />
>
> git 版本管理的实际内容全部存放在该目录下，可以打开该目录查看其内部文件结构，其中就包含了上述 git config 指令所引用到的 config 文件。如下图所示。
>
> <img src="C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20260516214123954.png" alt="image-20260516214123954" style="zoom:67%;" />
>
> 此 config 配置文件是跟随每一个 repository 仓库的，其作用范围也是仅限该仓库。

### git clone

用于克隆已经存在 Git 仓库到新目录中，自动实现远程仓库与本地仓库的一个连接。

``` Bash
#克隆完整的仓库到本地目录中
git clone <url>
```



## Git  仓库状态

### git status

不带选项的指令可以直接查看当前工作区的文件状态，会显示已经变更但是没有暂存，或者没有跟踪文件。

```bash
#显示当前工作区的状态
git status
```

### git log

会显示当前仓库的所有历史提交 commit 的记录。

```bash
#显示所有 commit 提交记录
git log
```

### git ls

``` git ls -files```显示所有已经跟踪的文件。

```bash
#显示所有已经跟踪的文件
git ls -files
```

### git diff

显示工作区与暂存区之间的不同。

```bash
#显示差异
git diff
```



## Git 基础快照

### git add

从工作区添加需要跟踪的文件到缓存区中进行缓存。最常用的指令为：``` git add .``` 其中点号代表该工作区所在目录的所有文件。通常在新增跟踪文件，或者确认保存文件变更时，都要先进行添加指令。

```bash
#添加工作区中所有的文件到暂存区中 working tree -> index
git add .

#添加某个文件,可以是多个文件
git add <file> [<file1> ...]

```



### git commit

提交指令，用于提交当前缓存区中已经 add 的缓存文件，将这些文件提交到仓库进行版本管理，每一个提交都有一个对应的为一的值，该值由 Git 自动生成；通常为了便于管理，我们会在提交时附上此次变更的一些内容，使内容更加容易理解。

```Bash
#将暂存区index中的文件提交到版本仓库中
git commit -m "修复了**bug"
```

我们可以通过使用 ```git log``` 指令查看历史提交，以及其对应的唯一编码 —— Hash。

> Git 用以计算校验和的机制叫做 SHA-1 散列（hash，哈希）。 这是一个由 40 个十六进制字符（0-9 和 a-f）组成的字符串，基于 Git 中文件的内容或目录结构计算出来。

<img src="C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20260509174051554.png" alt="image-20260509174051554" style="zoom: 80%;" />

如上图所示，commit 后面即是其唯一编码，后续的版本回退等操作可以通过该码进行操作。

### Commit Message Standarlize - 提交信息标准化

提交信息（commit message）的标准化，是指在团队中约定一套**统一的格式**来书写每次提交的说明。最主流的标准就是 **Conventional Commits（约定式提交）**，它脱胎于 Angular 团队的规范，现在已被众多开源项目和公司采用。

一条标准的约定式提交消息结构如下：

```markdown
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

#### 1. 头部（必填）

- **type**：提交类型，用于说明改动的性质。常见值包括：
  - `feat`：新功能
  - `fix`：修 Bug
  - `docs`：仅文档变更
  - `style`：代码格式（不影响逻辑的空格、分号等）
  - `refactor`：重构（既不是修 bug 也不是加功能）
  - `perf`：性能优化
  - `test`：添加或修改测试
  - `build`：构建系统或外部依赖变更（如 webpack、npm）
  - `ci`：持续集成配置变更（如 GitHub Actions、Jenkins）
  - `chore`：杂项，不修改源码或测试的其他任务
  - `revert`：回滚之前的提交
- **scope**（可选）：影响的范围，例如 `feat(parser):`、`fix(auth):`，通常为模块或文件名。
- **description**：简短、祈使句式的描述（**首字母小写，结尾不加句号**），不超过 72 字符。比如 `add user login function`。

#### 2. 正文 body（可选）

当描述无法说清“为什么这样改”和“怎么改”时，在空一行后补充详细说明，可以多行。

#### 3. 脚注 footer（可选）

- **BREAKING CHANGE**：不兼容的变更，必须以 `BREAKING CHANGE:` 开头，正文说明变更内容。
- **关联 Issue**：`Closes #123` 或 `Refs #456`，用于自动关闭 Issue。

**BREAKING CHANGE 的两种写法：**

- 在脚注里专门写：

  ```markdown
  feat(api): change user endpoint response format
  
  BREAKING CHANGE: the user endpoint now returns `id` as a string
  ```

  

- 或者在 type/scope 后加 `!`：

  ```markdown
  feat(api)!: change user endpoint response format
  ```

示例：

```bash
#标准化的 commit 消息
git commit -m "refactor(main):重构main函数"\
		  -m "使用有限状态机结构重构了if-else的main函数结构"
```



### git restore

从 [git checkout](#git checkout) 指令中提取出来的功能，用于丢弃当前工作区中的变更，使其恢复为上一次记录的状态，该命令默认使用暂存区的内容覆盖工作区的内容。

```bash
#撤销当前工作区变更
git restore [<file>]
```

### git stash

如果想要记录当前工作区和暂存区中的更改，并且想要退回到一个干净的工作目录/分支时，可以使用该指令。该指令会保存本地的变更，并且恢复当前的工作区为其匹配的 HEAD 指向的历史提交状态。

```bash
#暂存当前的工作区的更改
git stash [push]

#查看缓存的文件有哪些
git stash list

#应用已缓存的变更
git stash apply
```

### git reset

用于设置目前版本管理的最新一次提交状态 Head，变更为历史的状态，做版本回退的作用。例如：需要强制回退到历史版本，那么可以使用如下指令：``` git reset --hard e9f1bd8e11```，git 版本库就会回退到与 e9f1bd8e11 匹配的提交版本状态。

```Bash
#回退到 e9f1bde11 所对应的历史提交版本
git reset --hard e9f1bde11
```

> 注：该指令需要慎用！因为一旦回退，那么此后的版本都将丢失无法找回。



## Git 分支与合并

### git branch

该指令若无选项，则会显示当前的所有分支有哪些，如果生成分支可以输入指令 ```git branch <branch-name>```就可以生成名称为 develop（如果 \<branch-name> = develop） 的分支；需要删除分支可以输入指令 ``` git branch -d <branch-name>```.

```bash
#查看当前仓库所有分支
git branch

#查看远程仓库所有分支
git branch -r

#创建分支
git branch <branch>

#删除分支
git branch -d <branch>
```

### git checkout

该指令用于检出分支或者文件，主要有以下三个功能：

* 切换分支：```git checkout <branch-name>```，切换到指定的分支；
* 恢复文件：```git chekout [--] <file>```，从暂存区/历史提交中恢复当前工作区的文件；
* 创建分支：```git checkout -b <branch-name>```，则会自动创建分支且切换到该分支。

```Bash
#切换到指定分支
git checkout <branch>

#恢复文件
git checkout [--] <file>

#创建分支且切换到该分支
git checkout -b <branch>
```

### git switch

主要用于分支的切换，是从 git checkout 中提取出来的功能；

```bash
#切换到某分支
git switch <branch>

#创建分支然后切换到该分支
git switch -c <branch>
```

### git merge

该指令主要用于合并分支，``` git merge <branch>```：合并分支 branch 的提交到当前分支上，通常成功合并后会需要我们提供一个合并后的 log 消息，用于说明该合并操作，如果使用选项 ``` --no-commit``` 则不会。

```bash
#合并指定分支到当前分支上
git merge <branch>

#合并指定分支，并且不手动输入日志消息
git merge --no-commit <branch> 

#如果合并产生了冲突，可以中断合并
git merge --abort
```

如果分支之间存在分支，在这时我们可以使用 ``` git merge --abort``` 中断当前的合并操作，并且会自动重构恢复合并前的状态。

### git tag

`git tag` 的作用是**给某个特定的提交（commit）打上一个永久的、易于记忆的标记**。

它最常见的用途就是标记软件的发布版本号，比如 `v1.0.0`、`v2.3.1` 等。

你可以把它理解成里程碑式的记号。分支（branch）是不断向前移动的指针，而标签（tag）一旦打在某个 commit 上，就永远指向那个瞬间的代码快照，不会自动移动。

分类如下：

| **轻量标签** | 一个指向某个 commit 的“别名”，不包含额外信息                 | 临时标记、个人备忘               |
| ------------ | ------------------------------------------------------------ | -------------------------------- |
| **附注标签** | 一个完整的 Git 对象，包含标签作者、日期、注释信息，还可以用 GPG 签名验证 | 正式的版本发布、对外公开的里程碑 |

```bash
# 轻量标签（不推荐用于正式发布）
git tag v1.0.0-light

# 附注标签（推荐）
git tag -a v1.0.0 -m "正式发布 1.0.0 版本"
```



## Git 共享与同步

### git fetch

用于从其他仓库中下载 git 的对象或者参考，可以从指定的仓库名或者仓库的 URL 进行获取，当没有指定远程仓库名时，默认使用 origin，除非当前分支配置了一个上游分支（链接到远程仓库的分支）。

```bash
#下拉远程仓库的指定分支
git fetch origin <branch>
```

只是获取远程分支的最新状态，此时可以查看本地仓库与远程仓库之间的变化有哪些。

### git pull

该指令在调用时默认先自动调用指令 ``` git fetch``` 下载指定仓库的分支，然后自动决定集成 （```git merge```）远程分支到当前分支上。如果没有指定仓库，则默认为当前分支对应的上游分支。

```bash
#下拉远程仓库orign的main分支并合并到当前分支
git pull origin main
```

着意味着，pull 下拉仓库数据之后就会自动进行合并，对当前本地仓库做修改。

### git push

从本地仓库更新一个到多个分支，标去或者其他的参考引用（references）到远程仓库，并且发送所有远程仓库所没有的必要数据。

最简单的方式推送就是：

```bash
#推送当前的main分支到远程仓库的main分支中
git push origin main
```

该指令会推送本地的 main 分支到 origin 远程的 main 分支。

### git remote

用于管理设置或者跟踪的仓库。

```bash
# 检查当前的远程仓库
git remote -v

# 添加远程仓库并且命名为 "origin"
git remote add origin <url>

# 删除远程仓库 origin
git remote remove origin
```

# .gitigonore 文件

对于一些我们不需要进行版本管理的文件，如日志文件 .log 和便过程中临时创建的文件等，这些不需要被跟踪的文件，并且不想要在未跟踪列表中查看到这些文件。所以 git 存在一个名为 `.gitignore`的文件。

> **注：要养成一开始就为你的新仓库设置好 `.gitignore` 文件的习惯，以免将来误提交这类无用的文件。**
>
> 可以手动在与 .git 目录所在位置手动生成 txt 文件，然后更名为 .gitignore 即可生成该文件。 

文件 `.gitignore` 的格式规范如下：

- 所有空行或者以 `#` 开头的行都会被 Git 忽略。
- 可以使用标准的 glob 模式匹配，它会递归地应用在整个工作区中。
- 匹配模式可以以 `/`开头防止递归。
- 匹配模式可以以`/`结尾指定目录。
- 要忽略指定模式以外的文件或目录，可以在模式前加上叹号`!`取反。

所谓的 glob 模式是指 shell 所使用的简化了的正则表达式：

* `*`星号匹配零个或多个任意字符；
* `[abc]` 匹配任何一个列在方括号中的字符 （这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）； 
* `?`问号只匹配一个任意字符；如果在方括号中使用短划线分隔两个字符， 表示所有在这两个字符范围内的都可以匹配（比如 `[0-9]` 表示匹配所有 0 到 9 的数字）。 
* `**`两个星号表示匹配任意中间目录，比如 `a/**/z` 可以匹配 `a/z` 、 `a/b/z` 或 `a/b/c/z` 等。

 `.gitignore` 文件的例子：

```markdown
# 忽略所有的 .a或.o 文件
*.[ao]

# 但跟踪所有的 lib.a，即便你在前面忽略了 .a 文件
!lib.a

# 只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO
/TODO

# 忽略任何目录下名为 build 的文件夹
build/
# 保留跟踪指定的 build 文件夹，前提是 src/app/ 目录没有被忽略
!src/app/build/

# 忽略 doc/notes.txt，但不忽略 doc/server/arch.txt
doc/*.txt

# 忽略 doc/ 目录及其所有子目录下的 .pdf 文件
doc/**/*.pdf
```

通常我们只需要跟踪必要的工程文件即可，将提交到远程仓库的代码最简化，方便管理。

# 常用 Git 工作流程

![image-20260517194613163](C:/Users/10637/AppData/Roaming/Typora/typora-user-images/image-20260517194613163.png)

如上图所示通常开发过程存在几个主要的分支：

* `master`/`main` ：是项目的主分支，不会直接在该分支上进行代码的开发，通常会在确定稳定的发布版本之后，添加版本标签 `Tag`；

* `develop`：是从主分支中创建出的开发分支，所有开发过程基本上都在该分支上进行；

* `feature`：由 `develop` 创建的用于开发特定功能的分支，通常命名为 `feature/can` 等格式，表明其目的是开发 can 功能，一旦该功能通过测试并确定已经实现则会合并入 `develop` 分支；

  >通常 feature 分支可以存在多个，所以必须命名为具体的功能：``feature/**`` 

* `release`：由 `develop` 分支创建的发布版本分支，通常是在完成某些功能之后，该分支主要用于发布版本的测试，在该版本稳定之后即可进行软件定版，合并到 `develop` 分支与 `master` 分支，此时 `master` 添加对应的版本标签 `Tag`；

  > 注：release 分支的命名需要附上版本号，如：`release/v1.0`

* `hotfix`：由`master`or `main`分支产生，主要用于修复已知的历史版本中存在的紧急bug，修复完成之后即会合并到`master`和`develop`分支中。

不同的分支具有不同的生存周期

`master/main`&`develop`两个主分支的生存周期最长，与整个项目的开发周期一致；

`feature`&`release`分支的生存周期较短，通常在特定的功能以及版本开发完成之后即被合并入其他主分支，随即该分支便会被删除；

`hotfix`分支生存周期最短，在紧急修复某些漏洞bug之后即会被删除。

# 构建 GitLab 仓库并上传本地项目

首先登录 GitLab 创建新的工程

<img src="C:/Users/10637/AppData/Roaming/Typora/typora-user-images/image-20260517202853300.png" alt="image-20260517202853300" style="zoom: 67%;" />

![image-20260517202944245](C:/Users/10637/AppData/Roaming/Typora/typora-user-images/image-20260517202944245.png)

![image-20260517203037827](C:/Users/10637/AppData/Roaming/Typora/typora-user-images/image-20260517203037827.png)

> **注：新建空白工程，一定要取消勾选初始化仓库，否则远程仓库与本地仓库不一致。**

![image-20260517203354463](C:/Users/10637/AppData/Roaming/Typora/typora-user-images/image-20260517203354463.png)

然后，在本地 Git 仓库中右键打开 Git Bash，输入以下指令：

```bash
# 链接远程仓库并命名为origin
git remote add origin http://172.30.3.110/evs_project001/busandcomercialvehicle_group/chenjinzhong/test.git #该URL为上图中创建的工程复制而来的内网链接

#外网访问请使用以下指令，与上述指令二选一
git remote add origin http://xgit.quyohui.com/evs_project001/busandcomercialvehicle_group/chenjinzhong/test.git #替换域名为外网域名

#上传所有本地分支
git push -u origin --all

#上传所有本地标签
git push -u origin --tags
```

上传成功之后，刷新 GitLab 界面即可看到上传的项目文件。

![image-20260517205058247](C:/Users/10637/AppData/Roaming/Typora/typora-user-images/image-20260517205058247.png)

这样后续就可以在 GitLab 上查看目前项目的状态了，也可以在其他设备通过 URL 下拉当前的工程，对于团队合作或者远程办公十分便捷。GitLab 有项目相关的内容数据分析功能，感兴趣的可以多研究。

# Git Bash 与 Shell 程序解释说明


## 1. 什么是 Shell？

**Shell 就是与操作系统伴生的提供用户操作的程序，接收你输入的命令（命令为系统内部已经编写好的可以运行脚本/程序）、然后指挥操作系统干活。**

> 你敲下 `ls`（展示文件）、`cd`（切换目录），Shell 就会把这些指令调用给系统内核去执行。

Shell 有两种形式：

- **命令行 Shell（CLI）**：纯靠敲键盘，黑底白字窗口，比如你用的 Git Bash。
- **图形 Shell（GUI）**：靠鼠标点点点，比如 Windows 资源管理器。

在 Git 的语境里，Shell 就是那个等待你输入 `git add`、`git commit` 的交互环境。

------

## 2. 什么是 Bash？

**Bash** 是 **Bourne Again SHell** 的缩写，它是 Shell 家族里的一个**具体成员**，是 Linux 和 macOS 系统的 shell 。

Shell 程序的不同衍生版本：

- **Bash**：市场占有率最高，上手容易，功能强大。（Git Bash 用的就是它）
- **Zsh**：现在 macOS 的默认 Shell，界面可以搞得很炫。
- **Sh**：祖师爷级别的 Shell，功能最基础。
- **PowerShell**：微软家的，处理的是对象而不是纯文本，在 Windows 上很强大。

所以，当你打开 Git Bash 时，那个等待你输入命令的黑窗口，里面跑着的程序就是 **Bash**。它让你在 Windows 上也能用熟悉的 Linux 命令（如 `ls`, `cat`, `grep`）和语法（如 `export`, `for` 循环），来操作文件和执行 Git 命令。

------

## 3. Git Bash 里的 “Bash” 到底是谁？

在 Windows 上，原生的命令行 Shell 是 `cmd.exe` (命令提示符) 和 `PowerShell`，它们不懂 Linux 系统 Shell 的 `ls`、`rm` 这类命令。

> Git 工具本身就是由 Linux 系统创始人开发的版本控制软件，用于管理 Linux 系统文件。

**Git Bash 是一个“特供包”**，它把 **Bash** 这个 Linux 的 Shell，连同几百个常用的 Linux 小工具（比如 `ls`、`grep`、`awk`），打包移植到了 Windows 上。因此，你的 `git commit` 等命令其实是运行在一个仿真出来的 Bash 环境里。
