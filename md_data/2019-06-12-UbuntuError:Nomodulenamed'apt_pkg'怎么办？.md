---
layout:     post
title:      UbuntuError:Nomodulenamed'apt_pkg'怎么办？
subtitle:   
date:       2019-06-12
author:     P
header-img: img/post-bg-unix-linux.jpg
catalog: true
tags:
    - python
---
ubuntu经常用要添加PPA源，就是使用如下命令：

但不知什么时候开始，就出现了错误Error: No module named 'apt_pkg' 。

这个问题困扰我好久了，每次想解决，在网上忙活半天都没有找到解决办法。

**今天我找到了答案。**

** **

**第一步：sudo gedit /usr/bin/apt-add-repository**

我们会发现所谓"apt-add-repository"命令其实就是一个python脚本，而且最上面一行写着：#! /usr/bin/python3

说明这是一个python3脚本。

**第二步：sudo ls -l /usr/bin/python3**

显示：/usr/bin/python3 -> python3.5

说明在我的ubuntu上python3是链接到python3.5的 。问题就在这个python3.5上。

**第三步：**

**cd /usr/lib/python3/dist-packages/**

**ls apt_pkg***

显示: apt_pkg.cpython-34m-x86_64-linux-gnu.so

注意其中34m这个字样，这表示只有python3.4可以安全使用这个组件！而我们电脑python3是链接到python3.5的！

**不同的ubuntu版本不一定显示34m，所以一定要自己去查查看这个文件。然后修改python3链接到对应版本。**

说到这里解决办法就很简单了。

**第四步：**

**sudo rm  /usr/bin/python3**

**sudo ln -s  /usr/bin/python3.4  /usr/bin/python3   （**<em id="__mceDel"><em id="__mceDel"><em id="__mceDel"><em id="__mceDel">具体根据文件下的文件名字版本）**</em></em></em></em>**

大功告成！ 快去试试看apt-add-repository命令是不是可以用了！

最终奥义！！！

**sudo find / -name "apt_pkg.cpython-35m-x86_64-linux-gnu.so"**

**<em id="__mceDel">**</em>**<em id="__mceDel"><em id="__mceDel"><em id="__mceDel">cd /usr/lib/python3/dist-packages/**</em></em></em>

**<em id="__mceDel"><em id="__mceDel"><em id="__mceDel">**</em></em></em>**<em id="__mceDel"><em id="__mceDel"><em id="__mceDel"><em id="__mceDel">sudo cp apt_pkg.cpython-35m-x86_64-linux-gnu.so apt_pkg.cpython-36m-x86_64-linux-gnu.so  （具体根据文件下的文件名字版本）**</em></em></em></em>
