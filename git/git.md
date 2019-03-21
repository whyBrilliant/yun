# git 笔记

## 账号部分(**)
在使用Git进行开发的时候，我们可以使用ssh url或者http url来进行源码的clone/push/pull.

### 区别：

1. 用ssh url需要在本地配置ssh key，这也就意味着你必须是开发者或者有一定的权限，每次的代码同步（主要是push和clone）不需要进行用户名和密码的输入操作；
2. 那么http url就相对宽松些，但是需要在每次同步操作输入用户名和密码，有时候，为了省去每次都输入密码的重复操作，我们可以在本地新建一个文件，来存储你的用户名和密码，只需要在第一次clone输入用户名和密码，这些信息就被存储起来，以后就可以自动读取，不需要你在手动输入了。 在Git官网介绍了这一实现，是通过叫做[credential helper](https://git-scm.com/docs/gitcredentials)的小玩意儿实现的。可以把它叫做证书或者凭证小助手，它帮我们存储了credential(凭证，里面包含了用户名和密码)


### 介绍
当没有任何的证书小助手(credential helpers)定义的时候，Git将会按照如下的策略检查，并请求用户输入用户名和密码

GIT_ASKPASS环境变量

core.askPass配置

SSH_ASKPASS配置

上述都没有。最终直接要求用户在terminal输入密码

为了避免重复的输入用户名和密码，git提供了两种方法来减少这种情况：

1. 在给定的认证环境中静态地配置用户名

2. 凭据（credential）帮助缓存或者存储密码，或者和系统交互获得密码

  - 第一种：config中加入下面的语句：
```
  [credential "https://example.com"] 
  username = me 
```

- 第二种：Credential helpers

Credentials helpers是一种外置的程序代码，Git可以从那里取得用户名和密码。credentials helpers通常和操作系统或者其他程序提供的安全存储交互。

为了使用helper，你必须首先选择一个使用。Git现在包含了如下的两个helper：

1. Cache 
cache 将凭据在内存中进行短时间的缓存，详情见 https://git-scm.com/docs/git-credential-cache

2. Store 
store 将凭据保存在磁盘上，详情见 https://git-scm.com/docs/git-credential-store 
当然你也可能有第三方的helper，在git help -a的输出中使用 credential-*进行搜索，并且参考每个helper的文档。一旦你选择了一个helper，你可以通过把helper的name放到credentials.helper的变量中来告诉Git使用哪个helper。

### 实践

一、cache方式

1. 设置记住密码（默认15分钟）：

    git config --global credential.helper cache

2. 自定义设置时间

    git config credential.helper 'cache --timeout=3600'


二、store方式

1. 先查看系统支持哪种helper
```
  git help -a | grep credential 
```
cache是存储在内存中，可以设定有效时间但是时间过去后，将会失效；store是存储在磁盘上，不过用户名和密码是明文存储的，要是想加密存储，可是使用gnome-keyring来存储。

2. 设置credential helper 

凭据可以设置为全局，也可以设置为仅仅在当前的项目下起作用。 
如果想使用全局，则设置为： 

git config –global credential.helper store [–file=git_credentails]

如果仅仅设置当前项目有效，则进入项目代码目录下，运行： 

git config credential.helper store [–file=.git_credentails]

上面的–file=.git-credentials表示你的用户名密码存储的文件目录结构，即存放在哪里由你指定, 也可以不指定。

3. 查看配置

    如果你的凭证位置没有指定，则去用户目录下查看，发现多了一个.git_credentials文件，打开文件可以看到里面以明文存放你的用户名和密码（进行第四部分后才会有）。 


    然后在你的项目目录下运行指令 git config –list 发现配置文件多了一项 

    credential.helper=store

4. 同步一下