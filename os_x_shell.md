#OS X shell相关
OS X默认使用的是bash shell，如果不确定，可以用下面命令查看

	echo $SHELL

##环境变量
打印环境变量列表：

	printenv
	
列出完整的shell变量：

	set

注意这个的输出有可能会很长。

环境变量可以设在`~/.bash_profile`里，修改后要重启shell或者新开一个shell生效，或者在当前shell中用下面的命令：

	source ~/.bash_profile

`source`是shell的内置命令，它会执行后面文件的内容，等同于`.`(点号)

临时的环境变量（只在当前shell有效）用`export`，比如：

	export VAR_A=/foo/bar
