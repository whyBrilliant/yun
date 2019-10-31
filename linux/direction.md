# direction

## yum

> Yum（全称为 Yellow dog Updater, Modified）是一个在Fedora和RedHat以及CentOS中的[Shell](https://baike.baidu.com/item/Shell)前端软件包管理器。基于[RPM](https://baike.baidu.com/item/RPM)包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装。 



### 常用命令

1. yum install X
2. yum remove x 或 yum erase x
3. yum update x 或 yum upgrade x
4. yum info x 查看包信息
5. yum search x 搜索包
6. yum deplist x 显示软件包依赖关系



### 参数

| 参数名        | 说明                 |
| ------------- | -------------------- |
| -q            | 静默执行             |
| -t            | 忽略错误             |
| -R            | 设置等待时间（分钟） |
| -y            | 自动应答yes          |
| --skip-broken | 忽略依赖问题         |
| --nogpgcheck  | 忽略GPG验证          |

check-update 检查可更新的包
clean all 清除全部
clean packages 清除临时包文件（/var/cache/yum 下文件）
clean headers 清除rpm头文件
clean oldheaders 清除旧的rpm头文件
deplist 列出包的依赖
list 可安装和可更新的RPM包
list installed 已安装的包
list extras 已安装且不在资源库的包
info 可安装和可更新的RPM包 信息
info installed 已安装包的信息(-qa 参数相似)
install[RPM包] 安装包
localinstall 安装本地的 RPM包
update[RPM包] 更新包
upgrade 升级系统
search[关键词] 搜索包
provides[关键词] 搜索特定包文件名
reinstall[RPM包] 重新安装包
repolist 显示资源库的配置
resolvedep 指定依赖
remove[RPM包] 卸载包



## wget

 wget是一个从网络上自动下载文件的工具，它支持http，https，ftp协议，可以使用http代理 . 所谓的自动下载是指，wget可以在用户退出系统的之后在后台执行。这意味这你可以登录系统，启动一个wget下载任务，然后退出系统，wget将在后台执行直到任务完成 

 wget 可以跟踪HTML页面上的链接依次下载来创建远程服务器的本地版本，完全重建原始站点的目录结构。这又常被称作”递归下载”。 



| 选项 | 选项(全)          | 说明                               |
| ---- | ----------------- | ---------------------------------- |
| -V   | --version         | `显示 ``Wget` `的版本信息并退出。` |
| -h   | --help            | 打印此帮助。                       |
| -b   | --background      | 启动后转入后台。                   |
| -e   | --execute=COMMAND | 运行一个‘.wgetrc’风格的命令。      |

[wget命令详解]: https://www.cnblogs.com/ftl1012/p/9265699.html



eg:

​	1.  使用wget -O下载并以不同的文件名保存(-O：下载文件到对应目录，并且修改文件名称) 

​	 `wget -O wordpress.zip http:``//www``.minjieren.com``/download``.aspx?``id``=1080` 



## tar

 tar支持将多个文件包成一个文件，同时还可通过gzip/bzip将该文件压缩。目前window的winRAR也支持.tar.gz文件解压。 

 说明： -c/-x/-t/u 不可同时出现 

| 选项           | 参数说明                                                     |      |
| -------------- | ------------------------------------------------------------ | ---- |
| -c             | 新建打包文件，同 -v 一起使用 查看过程中打包文件名            |      |
| -x             | 解压文件， -C 解压到对应的文件目录。                         |      |
| -f             | 后面接要处理的文件                                           |      |
| -j             | 通过bzip2方式压缩或解压，最后以.tar.br2 为后缀。压缩后大小小于.tar.gz |      |
| -z             | 通过gzip方式压缩或解压，最后以.tar.gz 为后缀                 |      |
| -v             | 压缩或解压过程中，显示出来过程                               |      |
| -t             | 查看打包文件中内容，重点文件名                               |      |
| -u             | 更新压缩文件中的内容。                                       |      |
| -p             | 保留绝对路径，即允许备份数据中含有根目录                     |      |
| -P             | 保留数据原来权限及属性。                                     |      |
| --explode=FILE | 压缩过程中，不要讲FILE打包                                   |      |
| man tar        | 查看更多参数                                                 |      |



常用命令

| 命令                                             | 说明           |
| ------------------------------------------------ | -------------- |
| tar -jcv -f 压缩文件名称.tar.br2                 | 压缩方式一     |
| tar -jxv -f 压缩文件名称.tar.br2 -C 指定文件目录 | 解压文件方式一 |
| tar -zcv -f 压缩文件名称.tar.gz                  | 压缩方式二     |
| tar -zxv -f 压缩文件名称.tar.gz                  | 解压文件方式二 |



eg: tar  -zxvf  xxxx.tar.gz  -C  /etc/local/src/xxxx-dir 将压缩文件解压到目标目录