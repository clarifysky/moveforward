# 介绍

Laravel尽力使整个PHP开发体验愉悦，包括你的本地开发环境。[Vagrant](http://vagrantup.com/)提供了一个简单的，优雅的方式来管理和为虚拟机提供必需品。

Laravel的Homestead是一个官方的，预包装的Vagrant盒子，为你提供了强力的开发体验，无需你安装PHP，web服务器和别的其它服务端软件在你的电脑上。无需担心弄脏你的操作系统！Vagrant盒子是彻底的一次性的，如果有什么出错了，你可以销毁它并瞬间重建！

Homestead可以运行在任意的Windows，Mac或Linux系统上，包括Nginx Web服务器，PHP7.0，MySQL，Postgres，Redis，Memcached，Node，和你需要的用来构建令人惊叹的Laravel应用的所有其它的好东西。

> 如果你使用Windows，你需要开启硬件虚拟化（VT-x）。通常在BIOS里面开启这个，如果你正在一个UEFI系统上使用Hyper-V，你可能还需要禁用Hyper-V来访问VT-x。

## 包括的软件

- Ubuntu 16.04
- Git
- PHP 7.0
- Nginx
- MySQL
- MarialDB
- Sqlite3
- Postgres
- Composer
- Node(With PM2, Bower, Grunt, and Gulp)
- Redis
- Memcached
- Beanstalkd

# 安装&设置

## 第一步

在启动Homestead环境前，你必须安装[VitualBox 5.x](https://www.virtualbox.org/wiki/Downloads)或[VMWare](http://www.vmware.com/)还有[Vagrant](http://www.vagrantup.com/downloads.html)。所有的这些软件都针对流行的操作系统提供易于使用的安装器。

要使用VMware供应者，你需要同时购买VMware Fusion/Workstation和[VMware Vagrant plug-in](http://www.vagrantup.com/vmware)。尽管它不是免费的，VMware还可以提供拆箱即用的快速的共享文件夹性能。

### 安装Homestead Vagrant Box

一旦VirtualBox/VMware和Vagrant安装好了，你应当使用命令终端运行下面的命令将`laravel/homestead`添加到你的Vagrant安装中。它会花费一些时间来下载盒子，取决于你的网速：

```
vagrant box add laravel/homestead
```

如果这条命令失败了，确保你的Vagrant安装是最新的。

### 安装Homestead

你可以简单地克隆仓库来安装Homestead。考虑将仓库克隆到一个`Homestead`文件夹内，放到你的"家"目录中，因为Homestead盒子会作为你的所有的Laravel项目的宿主来提供服务：

```
cd ~

git clone https://github.com/laravel/homestead.git Homestead
```

一旦克隆好了Homestead仓库，在Homestead目录中运行`bash init.sh`来创建`Homestead.yaml`配置文件。`Homestead.yaml`文件会被放置在`~/.homestead`隐藏目录中：

```
bash init.sh
```

## 配置Homestead

### 设置你的供应商

`~/.homestead/Homestead.yaml`文件中的`provider`键表明了应当使用哪个Vagrant供应商：`virtualbox`, `vmware_fusion`, 或者`vmware_workstation`。你可以安妮所需将这个设置到供应商上面：

```
provider: virtualbox
```

### 配置共享的文件夹

`Homestead.yaml`文件中的`folders`属性列出了所有你希望与你的Homestead环境共享的文件夹。这些文件夹内的文件更改了之后，会被在你的本地机器和Homestead环境之间保持同步。你可以按需配置更多的共享文件夹：

```
folders:
    - map: ~/Code
      to: /home/vagrant/Code
```

要开启[NFS](http://docs.vagrantup.com/v2/synced-folders/nfs.html)，只需要添加一个简单的标识到你的同步文件夹配置中：

```
folders:
    - map: ~/Code
      to: /home/vagrant/Code
      type: "nfs"
```

### 配置Nginx站点

对Nginx不熟悉？没关系。`sites`属性允许你容易地将一个“域名”对应到你的Homestead环境中的一个文件夹上。`Homestead.yaml`文件中包含了一个示例站点配置。你可以按需添加更多的站点到你的Homestead环境中。Homestead可以作为一个方便的，虚拟化的环境为每一个你正在处理的Laravel项目工作：

```
sites:
    - map: homestead.app
      to: /home/vagrant/Code/Laravel/public
```

如果你在为Homestead盒子提供了必需品之后修改了`sites`属性，你应当重新运行`vagrant reload --provision`来更新虚拟机上的Nginx配置。

### Hosts文件

你必须为你的Ngxin站点在你的机器上添加"域名"到`hosts`文件中。`hosts`文件会将你的Homestead站点的请求转发到你的Homestead机器里。在Mac或Linux上，这个文件在`/etc/hosts`。在Windows上，它位于`C:\Windows\System32\drivers\etc\hosts`。你添加到文件中的一行内容和下面的类似：

```
192.168.10.10  homestead.app
```

确保列出的IP地址是你在`~/.homestead/Homestead.yaml`文件中设置的那个。一旦你在`hosts`文件中添加了域名并启动了Vagrant盒子，你就可以通过你的web浏览器访问这个站点：

```
http://homestead.app
```

## 启动Vagrant盒子

一旦你将`Homestead.yaml`编辑成了你所喜欢的，从你的Homestead目录运行`vagrant up`命令。Vagrant会启动虚拟机并自动配置你的共享文件夹和Nginx站点。

要销毁机器，你可以使用`vagrant destroy --force`命令。

## 每个项目安装

行对于全局安装Homestead并为你的所有项目共享同样的Homestead盒子，你也可以为你的每个项目配置一个Homestead实例。为每个项目安装Homestead可能在你希望将一个`Vagrantfile`和你的项目一起运营时带来福利，允许工作在这个项目上的别的简单地`vagrant up`。

要直接将Homestead安装到你的项目下，用Composer require它：

```
composer require laravel/homestead --dev
```

一旦Homestead被装好了，使用`make`命令在你的项目根目录下生成`Vagrantfile`和`Homestead.yaml`文件，`make`命令会自动配置在`Homestead.yaml`文件中指示的`sites`和`folders`。

Mac/Linux:

```
php vendor/bin/homestead make
```

Windows:

```
vendor\\bin\\homestead make
```

接下来，运行`vagrant up`命令并在浏览器中通过`http://homestead.app`访问你的项目。记住，你仍旧需要为`homestead.app`或域名添加一个`/etc/hosts`文件入口。

## 安装MariaDB

如果你希望使用MariaDB而不是MySQL，你可以将`mariadb`选项添加到你的`Homestead.yaml`文件中。这个选项会移除MySQL并安装MariaDB。MariaDB扮演着MySQL的替代者，所以你应当仍旧在你的数据库配置中使用`mysql`数据库驱动：

```
box: laravel/homestead
ip: "192.168.20.20"
memory: 2048
cpus: 4
provider: virtualbox
mariadb: true
```

# 日常使用

## 全局访问Homestead

有时你可能希望在你的文件系统的任意地方`vagrant up`你的Homestead机器。你可以通过添加一个简单的Bash方法到你的Bash profile中来达到这个目的。这个方法允许你在任何地方运行任意Vagrant命令，并会自动将命令指向你的Homesstead安装：

```
function homestead() {
    ( cd ~/Homestead && vagrant $* )
}
```

确保调整方法中的`~/Homestead`路径为你的实际的Homestead安装路径。一旦这个方法被安装了，你就可以在任意地方运行`homestead up`或`homestead ssh`命令。

## 通过SSH连接

你可以从你的Homestead目录发出`vagrant ssh`终端命令来SSH连接到你的虚拟机。

但是，由于你将可能频繁地SSH到你的Homestead机器，所以考虑将上面的“方法”添加到你的宿主机器来快速地SSH进入Homestead盒子。

## 连接到数据库

`homestead`数据库同时为MySQL和Postgres配置好了，为了更加方便，Laravel的`.env`配置好了使用这个数据库的框架。

要从你的宿主机器通过Navicat或Sequel Pro连接到你的MySQL或Postgres数据库，你应当连接到`127.0.0.1`和端口`3306`（MySQL）或`54320`(Postgres)。这两个数据库的用户名和密码都是`homestead/secret`。

> 你应当仅在从你的宿主机器连接到数据库时使用这些非标准端口。由于Laravel运行在虚拟机内，你将在你的Laravel数据库配置文件中使用默认的3306和5432端口。

## 添加额外的站点

一旦你的Homestead环境准备好并运行了，你可能希望为你的Laravel应用添加额外的Nginx站点。你可以随意地在单个Homestead环境内运行多个Laravel安装。要添加额外的站点，简单地将那个站点添加到你的`~/.homestead/Homestead.yaml`文件中，并然后从你的Homestead目录运行`vagrant provision`命令。

##  配置Cron进度

Laravel提供了一个方便的方式来[安排Cron任务](https://laravel.com/docs/5.3/scheduling)，通过安排一条单一的每分钟运行的`schedule:run`Artisan命令的方式。`schedule:run`命令会检查安排在你的`App\Console\Kernel`类中的任务来确定应当运行哪个任务。

如果你希望给一个Homestead站点运行`schedule:run`命令，你可以在定义站点的时候设置`schedule`选项：

```
sites:
    - map: homestead.app
      to: /home/vagrant/Code/Laravel/public
      schedule: true
```

这个站点的Cron任务会被定义在虚拟机的`/etc/cron.d`文件夹内。

## 端口

默认情况下，下面的端口会转发到你的Homestead环境中：

- SSH:2222->Forwards To 22
- HTTP:8000->Forwards To 80
- HTTPS:44300->Forwards To 443
- MySQL:33060->Forwards To 3306
- Postgres:54320->Forwards To 5432

### 发送额外的端口

如果你愿意，你可以发送额外的端口到Vagrant盒子中，也包括指定它们的协议：

```
ports:
    - send: 93000
      to: 9300
    - send: 7777
      to: 777
      protocol: udp
```

# 网络接口

`Homestead.yaml`的`networks`属性为你的Homestead环境配置网络接口。你可以按需配置更多的借口：

```
networks:
    - type: "private_network"
      ip: "192.168.10.20"
```

要启用[桥接](https://www.vagrantup.com/docs/networking/public_network.html)接口，配置一个`bridge`设置并将网络类型转为`public_network`：

```
networks:
    - type: "public_network"
      ip: "192.168.10.20"
      bridge: "en1: Wi-Fi (AirPort)"
```

要启用[DHCP](https://www.vagrantup.com/docs/networking/public_network.html)，仅需要从你的配置中移除`ip`选项：

```
networks:
    - type: "public_network"
      bridge: "en1: Wi-Fi (AirPort)"
```




