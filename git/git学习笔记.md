[TOC]

# GIT

## 1.不同版本控制方式介绍

### 1. 集中式版本控制(SVN)

​	所有的版本数据都保存在服务器上,协同开发者从服务器上同步更新或者上传自己的修改

<img src="https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7Ksu8UlITwMlbX3kMGtZ9p00V4uLaibxtZI9RLpq7tkSdlWiaF92AVeZ0ib9DicqBkS2poo5u8sEU2mCQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" />

### 2.分布式版本控制(git)

​	所有的版本信息全部同步到本地的用户上,这样就可以在本地查看所有的历史版本,也可以离线本地提交,只需要在连网时在push到对应的服务器上就行.因为每个用户都保留所有的版本数据,因此只要一个用户的设备没有问题就能恢复所有的数据,但也占用了本地存储空间,和存在一定的安全隐患.

<img src="https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7Ksu8UlITwMlbX3kMGtZ9p0ev8Q7qXjsTfeSwFexdA4tGjFAiaVEKQzAHdGcINXILKflI2cfk9BiawQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" />



## 2. GIT

### 1. LINUX下常见命令

		1. `cd`: 改变当前目录
		1. `pwd`: 显示当前所在的目录路径
		1. `ls`: 列出当前目录中的所有文件
		1. `touch`: 新建一个文件(`touch index.js 新建一个index.js文件`)
		1. `rm`: 删除一个文件
		1. `mkdir`: 新建一个目录
		1. `rm -r`: 删除一个文件夹
		1. `mv`: 移动文件.`mv [文件名] [对方目录名] [文件命]`
		1. `reset`: 清屏/初始化终端
		1. `clear`: 清屏
		1. `history`: 查看命令历史
		1. `help`: 打开帮助页面
		1. `exit`: 退出





### 2. git系统配置

```
// 查看系统config
// 文件目录: C:\Program Files\Git\etc\gitconfig
git config --system --list

// 查看当前用户(global)配置
// 文件目录: C:\Users\zZOMZz\gitconfig
git config --global --list

// 设置用户配置
git config --global user.name [value]
git config --global user.email [value]
```



### 3. git的几个分区

1. workspace: 工作区,就是平时存放代码的地方
2. Index/Stage: 暂存区,用于临时存放你的改动,事实上只是一个文件,保存即将提交到文件列表信息
3. Repository: 仓库区(或本地仓库),就是安全存放数据的位置,这里有你提交的所有版本的数据.其中HEAD指向最新放入仓库的版本
4. Remote repository: 远程仓库,托管代码的平台.

<img src="https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7Ksu8UlITwMlbX3kMGtZ9p0NJ4L9OPI9ia1MmibpvDd6cSddBdvrlbdEtyEOrh4CKnWVibyfCHa3lzXw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:80%;" />



### 4.常见git命令([git-cheat-sheet-education](https://education.github.com/git-cheat-sheet-education.pdf))

(Q, 退出END)

<img src="https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7Ksu8UlITwMlbX3kMGtZ9p0AII6YVooUzibpibzJnoOHHXUsL3f9DqA4horUibfcpEZ88Oyf2gQQNR6w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:80%;" />

1. `git init` 在当前目录创建一个本地Git代码库
2. `git clone [url]` 克隆一个远程代码库, 默认会设置好上游分支
3. `git status [filename]`  查看指定文件,或者所有文件的状态
4. `git add .` 添加所有文件到暂存区, **如果修改了文件需要重新添加**
5. `git commit -m '提交信息'` 将所有暂存区的内容提交到本地仓库
6. `git branch` 查看当前分支
7. `git branch [name]` 新建一个分支,在这个分支上修改代码不会影响到主分支
8. `git checkout [name]` 跳转到那个分支
9. `git checkout -b [name]` 新建一个分支并且切换过去
10. `git merge [name]` 合并指定分支(它的版本库)到当前分支(版本库), 在合并远程分支时, 是不能合并没有联系起来的两个分支的
11. `git branch -d [name]` 删除指定分支
12. `git branch -r` 查看远程分支
13. `git reset --hard HEAD`  将代码退回到上次提交时的状态
14. `git log` 查看所有commit提交的信息
15. `git reset --hard [commit id]` 退回到特定的commit
16. `git remote add [alias] [url]` 创建一个远程分支
17. `git push [remote] [branch]` 将分支branch发送给远程分支remote
18. `git fetch`：相当于是从远程获取最新版本到本地，不会自动合并
19. `git pull` : 相当于是从远程获取最新版本并`merge`到本地
20. `git branch --set-upstream-to=origin/master`将origin远程仓库下的master分支于当前本地分支联系起来, 设置上游分支, 在多分支的情况下能直接`git pull`
21. `git branch --track dev origin/master`创建一个dev分支, 使其自动跟踪远程仓库origin的master分支
22. `git push origin --tags` 将所有tag都push到仓库里



### 5. git文件状态

1. untracked(未跟踪): 此文件在文件夹当中,但未入库,不参与版本控制.可以通过`git add`将其状态变为Staged.如果使用`git rm`将文件移除版本库,则这个文件又是Untracked
2. Staged(暂存状态):  通过执行`git commit`将其提交到库中,这个时候库中的文件与本地文件相同,文件状态变为Unmodified
3. Unmodify(未修改): 文件已入库,且现在本地文件未被修改,版本库中的文件快照内容与文件夹中的完全一致
4. modified(已修改): 文件已修改.既可以使用`git add`使其变成staged状态,又或者使用`git checkout`丢弃修改,将其返回到unmodify状态,即将其中版本库中取出原本的来覆盖当前修改后的文件



### 6. .gitignore 文件

规则:

1.  在这个文件中可以使用Linux通配符.例如: (*) 代表任意多个字符, (?) 代表一个字符, 方括号([ abc]) 代表可选字符范围, ({ sting1, sting2,})大括号代表可选的字符串
2. 如果文件名称前加上( ! ),代表它不被忽略,不受这个gitignore文件影响

```javascript

#为注释
*.txt        #忽略所有 .txt结尾的文件,这样的话上传就不会被选中！
!lib.txt     #但lib.txt除外
/temp        #仅忽略项目根目录下的TODO文件,不包括其它目录temp,往上走
build/       #忽略build/目录下的所有文件,往下走
doc/*.txt    #会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
```





### 7. 远程仓库验证

- 方式一： 基于HTTP的凭证存储(无状态)
- 方式二： 基于SSH的密钥

git的品质存储(crediential)有几个选项,这里介绍一部分:

- "cache"模式, 把凭证(一般为用户名和密码)放在内存中一段时间. 密码永远不会被放到磁盘里, 并且会在15分钟后从内存中清除
- "store"模式, 会把凭证以明文的形式放到磁盘中, 并且永不过期

####  SSH密钥

- SSH( Secure Shell 安全外壳协议 )密钥: 以非对称加密实现身份验证
  - 一种方式: 利用自动生成的公钥-私钥对来简单的加密网络连接, 随后使用密码认证进行登录
  - 一种方式: 人工生成一对公钥和私钥, 通过生成的密钥进行认证, 这样就能在不输入密码的情况下登录
  - 公钥放在待访问的电脑里, 对应的私钥由用户自行保管

生成密钥步骤:

1. `ssh-keygen -t(公钥的类型) ras (后面是使用一种加密算法) -C(注释) [email]` 在本地生成一个密钥
2. 也可以是`ssh-keygen -t(公钥的类型) ed25519 (后面是使用一种加密算法) -C(注释) [email]`
   1. 生成的密钥在`C:\\用于\\DELL\\.ssh`里
3. 到github或者gitee上将这个密钥中的**公钥**添加到账号设置中的SSH公钥上,然后建立仓库



### 8. git提交对象

- 在每次提交操作时, GIT会保存一个提交对象(一个文件中)
  - 该对象会包含一个指向暂存区内容快照的指针(tree)
  - 该对象还会包含作者的姓名和邮箱,提交时输入的信息,以及指向它的父对象的指针



### 9. Git 分支

- Git的分支, 本质上时指向提交对象的可变指针
  - 默认分支master, 在多次提交之后, 指向最后一次的提交对象, 会自动移动
  - tag也是一个指针, 会指向打了tag的提交里
- git创建一个新的分支只是创建一个新的指针
- HEAD指针指向当前分支, master指向最后一次提交的提交对象
- 可以在修复BUG时,使用分支, 在过往提交上创建的新分支上修复BUG, 然后和现有开发的代码进行合并



出现冲突:

- 当前分支和合并分支的不一样的代码会出现冲突
  - 手动解决
  - 利用VScode解决



#### 远程分支:

- main分支为主分支
- 以<remote>/<branch>的形式命名
- `git fetch`将远程分支的主分支拿到本地
- `git branch --set-upstream-to=origin/main` 让本地分支跟踪
- `git config push.default upstream`修改默认push到上游分支,而不是默认的同名分支
- `git checkout --track origin/main`创建一个跟踪远程分支的main分支,并转换过去



- **`git checkout develop`在本地clone下代码后得到的是主分支,需要在别的分支上进行创作时,需要切换到该分支上,如果本地找不到该分支,则会到远程分支上找,创建一个新的分支,并跟踪它**
- `git push origin -d [name]` 删除远程仓库origin的一个分支
- `git branch -d [name]`删除本地命令



### 10. git工作流

- 在项目开发周期中, 可以同时拥有多个开放工作流, 定期的将某些主题的分支合并到其他分支中

- 经典的工作流:

  - master分支作为主分支

  - develop作为开发分支, 并且有稳定版本时, 会合并到master分支中

  - topic分支作为某一个主题或者功能或者特性的分支进行开发, 开发完成后合并到develop分支中





### 11. Git rebase

与`git merge`相似, 将不同分支的提交整合到线性的提交结构里

- merge用于记录git的所有历史, 所有错综复杂的历史都会被记录
- rebase会简化历史记录(`git log`),将两个分支的历史简化, 使整个历史更加简洁
- **永远不要在主分支上使用rebase**



- 不同的是,`git rebase [name]` 直接将子分支的提交对象与主分支(目前所在的分支与[name]分支)的提交对象合并,并直接将新的提交对象连接到[name]分支上, 是一种线性结构
  - 将当前子分支所指的提交对象的上一级指向的对象变为[name]分支当前指向的对象
  - 要在子分支上进行操作
  - rebase后master还停留在前一个提交对象上
  - 需要`git merge feature`
- `git merge`会保留图形结构

![Snipaste_2022-12-22_16-40-30](.\图片\Snipaste_2022-12-22_16-40-30.png)



### 12. git tag

给具体的某个版本打上标签, 一般用于标记版本更新, 不会发生移动, 会标记打标签时的提交对象, 在github中可以下载特定版本的内容

```shell
# 打标签
git tag v1.0.0
# 查看所有标签
git tag

# 将本地的标签传入到远程仓库
git push origin --tags
```

