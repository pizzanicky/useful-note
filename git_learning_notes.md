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

##基本流程
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
###查看历史记录
列出当前分支的版本历史  

	$ git log
###远程同步
上传所有本地分支的提交到远程服务器  

	$ git push [alias] [branch]

##学习Tips 
年龄大了记性不好，入门的时候打印一张[Cheatsheet](http://www.cheat-sheets.org/saved-copy/git-cheat-sheet.pdf)比较实用  
或者直接git -help
