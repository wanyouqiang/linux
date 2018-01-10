《Laravel Homestead》
==================================

Laravel Homestead 是一个官方的，已经打包好的Vagrant box,旨在提供一个极好的开发环境，不需要手工安装PHP,Web服务器以及其它服务端的软件，也不用担心安装多余的软件会搞乱你的操作系统！Vagrant boxes是完全可丢弃的。你可以在一分钟内重构box

### 第一步：常用软件安装
* Vitual Box/Vmware
* Vagrant

### 第二步：下载Homestead安装库
* git clone https://github.com/laravel/homestead.git
* composer require laravel/homestead

### 第三步：进入homestead目录下，执行安装
* bash init.sh
* vagrant up

### 配置Homestead.yaml
* provider:virtualbox(可选virtualbox,vmware_fusion,parallels)
* folders:该属性下可列举你想要共享到Homestead环境中的目录，如果这些文件中的内容发生了改变，将会同步到环境中的目录，你可以同时配置多个共享目录
* sites:对nginx不熟悉吗，没关系！你只需要在sites属性中配置域名加要映射的目录就OK，并且可以配置多个应用，如果改变了sites属性配置，需要运行vagrant reload --prvision 更新nginx配置信息
* 修改本地的hosts文件，保证IP与Homestead.yaml文件中的ID对应。
* 删除machine ,使用vagrant destroy --force命令
* 连接Mysql数据库，IP:192.168.10.10,username:homestead,password:secret

### 手动安装Homestead(已经下载Homestead.box文件)
```
metadata.json
{
    "name": "laravel/homestead",  # 添加后的box名称
    "versions": [{
        "version": "5.0.1", # 版本号
        "providers": [{
            "name": "virtualbox",
            "url": "file://E:/Homestead.box"  # 下载到本地的box
        }]
    }]
}

执行以下命令

vagrant box add metadata.json

```

### Homestead 设置root密码

sudo passwd root

### 常用命令
* vagrant status 获取当前虚拟机的状态信息
* vagrant global-status 查看全局所有虚拟机的状态信息
* vagrant up 启动本地环境
* vagrant ssh 通过ssh登录本地环境所在的虚拟机
* vagrant destroy 移除本地环境
* vagrant halt 关闭本地环境
* vagrant provision 修改配置文件后重置
* vagrant reload --provision 修改配置文件并重启

### Error example
> 删除homestead后重新安装报错

*使用vagrant global-status查看所有虚拟机，删除不用的，VBoxManage list vms获取虚拟机列表，然后在通过运行
VBoxManage unregistervm homestead --delete即可*