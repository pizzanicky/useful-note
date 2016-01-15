#Git学习
##和目前使用的Perforce简单对比
###Perforce：  
* 依赖服务器，必须在线使用  
* 获取最新或者某个指定版本的copy到本地
* 权限控制机制完善

###Git:    
* 可单机操作，联网时再push到服务器  
* 把整个远程仓库镜像到本地
* branch简单，缺点：搞不好容易分支混乱 
* 几乎没有权限控制

##为什么要学？
* 已经渐渐成为基本技能
* 流行。合作伙伴使用Git，我们完全不懂，无法有效协同工作
* 对自己的文档（不仅代码）进行跟踪的确非常方便  

##基本概念
本地三棵树：工作目录，暂存区（stage），本地仓库
HEAD：相当于一个指针，默认指向最后一次的commit

##基本操作
###初次使用Config
设置commit的用户名  

	$ git config --global user.name "[name]"
设置commit的email地址  

	$ git config --global user.email "[email address]"
###创建仓库
创建新的本地仓库  

	$ git init [project-name]
或者下载一个远程仓库  

	$ git clone [url]
###做修改
列出要提交的新文件或者修改过的文件  

	$ git status
diff未stage的文件  

	$ git diff
添加（snapshot）准备纳入版本管理的文件  
不一定新文件，原有文件修改后也要用add进行stage，才能提交

	$ git add [file]
diff stage的文件和其上一个版本  

	$ git diff --staged
unstage文件，但保留更改  

	$ git reset [file]
提交到版本库(把snapshot存入版本库)  

	$ git commit -m "[descriptive message]"
###分支、合并等
默认在master分支  
创建分支branchA并把工作目录切换到此分支  
`git checkout -b branchA`  
切换回master分支  
`git checkout master`  
push到远程仓库  
`git push origin branchA`  

`git pull`相当于`git fetch`+`git merge`，获取远程修改并merge到本地  
`git rebase`则是先撤销本地commit，重新将远程分支版本同步到本地，然后把本地分支撤销的commit作为patch打入新同步的版本。[这里](http://gitbook.liuhui998.com/4_2.html)图文并茂，比较容易理解。不过最好要自己试试
###查看历史记录
列出当前分支的版本历史  

	$ git log
###远程同步
上传所有本地分支的提交到远程服务器  

	$ git push [alias] [branch]

##工作流程
工作流程有不少，参考这篇[Git版本控制与工作流](http://www.jianshu.com/p/67afe711c731)  
目前比较适合我们自己公司的是[Gitflow Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)
##学习Tips 
* Git上手不算难，看看文档就能进行自己的首次更改和提交。但玩转整个工作流程，特别是多人协作，还是坑不少，感觉比较深的是达成同一目的的命令会有多种，看似相似的功能也不少，必须了解每个操作背后的原理才能玩转。所以一开始要自己做好代码备份，万一误操作。。。  
* 年龄大了记性不好，入门的时候打印一张[Cheatsheet](http://www.cheat-sheets.org/saved-copy/git-cheat-sheet.pdf)比较实用  
或者直接git -help（不过help里的有些说明比较晦涩）
