# Vagrant: Great for Distributed System Developers"

Vagrnat是一个工具，方便的配置和访问虚拟机。我们应该会经常用来配置统一的开发和测试环境。

只要机器上安装了VirtualBox和Vagrant，那么下面步骤可以帮助我们配置一个基本开发环境：

  1. `vagrant box add p32 http://files.vagrantup.com/precise32.box` 下载一个标准VirtualBox虚拟机镜像，放到`~/.vagrant.d/boxes/p32`目录下，以备使用。`precise32.box`是一个32bit Ubuntu机器。

  1. `mkdir learn; cd learn; vagrant init p32` 创建了一个目录 `learn`，在其中加入了一个配置文件 Vagrantfile，这个文件中指定使用`p32`这个镜像作为虚拟机基本镜像。Vagrantfile通常连同`learn`这个目录中其他源码文件一起加入到版本管理系统里。

  1. 修改Vagrantfile，指定一个配置脚本文件`bootstrap.sh`:

        Vagrant.configure("2") do |config|
          config.vm.box = "precise32"
          config.vm.provision :shell, :path => "bootstrap.sh"
        end

      并且在当前目录`learn`下编辑一个配置脚本`bootstrap.sh`，用来安装Apache

        #!/usr/bin/env bash
        apt-get update
        apt-get install -y apache2
        rm -rf /var/www
        ln -fs /vagrant /var/www

   1. `vagrant up` 启动一个虚拟机，执行镜像`p32`；启动之后，运行`bootstrap.sh`，安装Apache。

   1. `vagrant ssh` 通过ssh登录进入启动的虚拟机。虚拟机里有一个目录`/vagrant`是Host机器上`learn`目录（Vagrantfile所在目录的）镜像目录。所以虚拟机里可以方便的访问`learn`里所有的源码。

   1. `vagrant suspend` 休眠；`vagrant resume`唤醒；`vagrant halt`关机；`vagrant up`启动并且重新执行配置脚本；`vagrant reload --provision`重启一个已经启动着的虚拟机，并且执行配置脚本。


通过编辑Vagrantfile，我们可以让`vagrant up`命令启动好几个虚拟机，每个可以有不同的配置。然后把一个并行系统（比如一个在线广告系统）部署在这几台虚拟机上，并且执行。机群操作系统Skynet就是用这种方法做自动化测试的。
