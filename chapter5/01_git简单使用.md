# 源代码管理之git简单使用

- git 是分布式管理源代码

## git 的工作原理

- 工作区：仓库文件夹里除了.git目录以外的内容
- 版本库：.git 目录，用于存储巨鹿版本信息
	- 暂缓区（staged）
	- 分支 ：git自动创建的第一个分支
	- HEAD 指针：用于指向当前分支
- git add : 把文件修改或新添加的文件添加到暂缓区
- git commit : 把暂缓区的内容提交到当前分支

## 命令行的演练

### 1. 单人开发

1. 初始化代码仓库（本地版本库）
	- git init	
- 如果使用 git，必须配置用户名和邮箱
	- git config user,name "BW"
	- git config user.email "BW@163.com"
	- 以上命令会将用户信息保存到当前代码库中
	
- 配置全局global
	- git config --global user,name "BW"
	- git config --global user.email "BW@163.com"
	- /user/username/.gitconfig
	- 以上命令会将用户信息保存到用户目录下的.gitconfig文件中
- 初始化项目
	- touch main.m  : 创建 mian.m
	- git add main.m  : 将 main.m 添加到暂缓区
	- git commit -m " 初始化项目" main.m  : 将暂缓区的 mian.m 提交到本地版本库之后清空了暂缓区的所有内容，此时一定要使用 -m参数指定备注信息
	
	-  将当前文件夹下的所有新建或修改的文件一次性添加到代码库
		- git add .
- 查看文件状态
	- git status
		- 红色：代表该文件不在暂缓区，不被 git 管理
		- 绿色：代表该文件已经添加到暂缓区，但是没有提交到本地版本库 
- 添加多个文件
	- touch per.h per.m
	- git add .
	- git commit -m "添加了Person类"
	- open Person.h
	- git add .
	- git commit -m "增加Person类属性"
	- 注意 : 使用 git 时，每一次修改都需要添加再提交
- 给命令起别名
	- 局部别名
		- git config alias.st "status" : 给 status 起别名 st
	- 全局别名
		- git config --global alias.st "status" : 给 status 起 全局别名 st
- 删除文件
	- git rm per.m :  将 per.m 删除，只是在暂缓区删除，还需要使用 commit 提交到本地版本库
- 查看版本信息
	- git log : 查看版本库信息
	- git reflog : 功能更加强大
- 版本回退
	- git reset --hard HEAD : 回到当前版本，放弃所有没有提交的修改（慎用）
	- git reset --hard HEAD^ : 回到上一个版本
	- git reset --hard HEAD~3 : 回到之前第3个版本
	- git reset --hard e695b67 : 回到指定版本号的版本(版本号最低 5 位)

### 2.多人开发（共享版本库）

1. git 服务器（创建比较麻烦，需要使用 Linux 系统）
	- 一个 U盘可以作为共享版本库
	- 一个文件夹可以作为共享版本库（本次使用文件夹）
	- 可以把代码托管到 Github/OsChina 上
- 建立代码仓库
	- 切换目录
		- cd /Users/Desktop/git演练/公司/weibo
	- 建立空白代码库(专门用于团队开发)
		- git init --bare
- 项目经理准备项目（前奏）
	- 切换目录
		- cd /Users/Desktop/git演练/经理
	- "克隆"代码库到本地
		- git clone /Users/Desktop/git演练/公司/weibo/
	- 设置忽略文件信息
		- 忽略文件: 在和.git等级目录下创建一个.gitignore 文件,在该文件中指定需要忽略的文件
		- .gitignore ：可以指定哪些文件不纳入版本库的管理
		- 参考网址：[https://github.com/github/gitignore](https://github.com/github/gitignore)查看OC需要忽略的内容,将内容填写到 .gitignore 中
		- git add .gitignore : 将忽略文件添加到暂缓区，并使用 commit 提交
- 创建项目
	- 使用 xcode 创建项目到经理的项目文件夹下
	- 提交的同时可以 push 到远程代码库
- 张三加入开发
	- git clone 共享代码库的地址 
	- 修改代码 —> git commit —> git push
- 版本备份
	- 1.0版本开发完毕 -> 打包给测试人员（QA）进行测试 -> 上传到 AppStore 进行审核 -> 对1.0版本进行备份（打标签）（必须使用命令）
		- git tag -a weibo1.0 -m "这是1.0版本" : 打标签
		- git tag : 查看所有标签
	- 把标签上传到共享版本库
		- git push origin weibo1.0
	- 开始开发2.0版本
	- 发现1.0版本有 bug -> 创建文件夹（将最新的代码下载到该文件夹，用于修复 bug）
		- git clone 文件夹路径
	- 切换到1.0版本 tag 标签的代码
		- git checkout weibo1.0 : 切换到当前标签下
	- 创建一个分支weibo1.1fixbug，切换到该分支之后修复 bug
		- git checkout -b weibo1.1fixbug :  同时已经切换到了该分支
		- git push origin weibo1.1fixbug : 将weibo1.1分支上传到共享版本库
	- 对修复好的weibo1.1fixbug进行备份（打标签）
		- git tag -a weibo1.1 -m "这是修复1.0bug 的1.1版本" : 打标签
	- 把标签上传到共享版本库
		- git push origin weibo1.1
	- 在正在开发的2.0版本中，合并weibo1.1fixbug这个版本
		- 在 xode 中点击 sourceControl -> 点击 pull -> 选择weibo1.1fixbug这个分支
	- 删除weibo1.1fixbug分支
		- 常用操作
			- git branch : 查看本地（不是本地版本库）分支，可以查看到当前处于哪个分支，如果要删除某个分支，要保证不在该分支
			- git branch -r : 查看本地版本库的分支
			- git checkout 分支名 : 切换分支
		- 删除流程
			- git branch --delete 分支名（weibo1.1fixbug） : 删除本地的分支
			- git branch -r -d 分支路径（origin/weibo1.1fixbug） :  删除本地版本库的分支
			- git push origin --delete 分支名（weibo1.1fixbug） : 删除共享版本库的分支
