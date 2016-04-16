#Jenkins安全配置笔记
Jenkins安装好后默认是完全开放的，自己玩玩不要紧，但如果部署到公网环境，建议开启安全选项
##Security Realm（安全领域？）
头一次看到这个术语，官方中文翻译是“安全域”，个人感觉可能会和domain搞混。看了一下[这里](http://tomcat.apache.org/tomcat-6.0-doc/realm-howto.html)的解释。个人觉得Realm可以理解为web应用的用户的组属角色，配合不同用户角色的权限策略，形成基于用户/权限矩阵的访问控制。有点绕，简单说就是：

* Realm决定**谁可以访问**
* 授权策略决定**每个用户可以做什么**

##打开Security
进入系统管理，如果security没有打开，会有一行文字提示你启用

勾选“启用安全”，下面展开`安全域`（Realm）和`授权策略`两栏

安全域里最简单的应该就是第一项，`Jenkins专有用户数据库`，记得要勾上下面的`允许用户注册`，否则就得用下一节的方法关闭Security了（我就这么干了。。）然后在授权策略中选择`登录用户可以做任何事`，保存退出，就已经可以注册用户并登录

这时候任何人还是都可以注册用户，且拥有所有权限，所以还得再设置下授权策略。当然，如果只有你一个人使用，也可以这时候把“允许用户注册”去掉，就没有其他人可以注册了

如果想精确控制不同用户的权限，就在授权策略里选`安全矩阵`，只给匿名用户Overall的Read权限，然后记得添加管理员帐号并赋予所有权限，其他帐号按需要赋权。如果有必要还可以选择下面的`项目矩阵授权策略`来根据不同项目赋权。
##关闭Security
如果不小心开启了安全认证，结果没开允许注册，导致无法配置Jenkins，参考如下步骤：

1.	停止Jenkins服务
2.	编辑`$JENKINS_HOME`目录下的`config.xml`文件
3.	把`<useSecurity>true</useSecurity>`里的值改成`false`
4.	删除`authorizationStrategy`和`securityRealm`这两行
5.	启动Jenkins



##参考链接
[Disable security](https://wiki.jenkins-ci.org/display/JENKINS/Disable+security)