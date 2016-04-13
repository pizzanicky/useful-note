#Ubuntu14.04安装Jenkins

在Ubuntu上安装Jenkins非常简单，照着[官网说明](https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+Ubuntu)用`apt-get`就ok：

	wget -q -O - https://jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -
	sudo sh -c 'echo deb http://pkg.jenkins-ci.org/debian binary/ > /etc/apt/sources.list.d/jenkins.list'
	sudo apt-get update
	sudo apt-get install jenkins

装好后Jenkins就能开机自启动，启动脚本在`/etc/init.d/jenkins`

当然也可以手动启停：

	sudo /etc/init.d/jenkins start
	sudo /etc/init.d/jenkins stop
	
Log文件在这里：`/var/log/jenkins/jenkins.log`

默认配置参数在这里：`/etc/default/jenkins`

Jenkins默认监听端口`8080`，如果此端口被占用，可以到`/etc/default/jenkins`里修改这一行：

	HTTP_PORT=8080
	
先这么多
##参考链接
[Installing Jenkins on Ubuntu](https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+Ubuntu)