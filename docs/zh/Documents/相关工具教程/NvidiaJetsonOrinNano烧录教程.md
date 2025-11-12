> 这是一篇有关于小电脑烧录系统的文章，由于小电脑买来默认烧录的事Nvidia修改过的ubuntu20.04，所以没法用，我们需要烧录适配的纯净版ubuntu22.04，但是网上资料有点杂乱，所以写一篇记录一下。

> 作者：新疆大学 计算机科学与技术 黄耀增

# 一、安装SDK-Manager

> SDK网址：[https://developer.nvidia.com/sdk-manager](https://developer.nvidia.com/sdk-manager)

不管你是双系统还是虚拟机都可以配置，安装.deb版本的Nvidia SDK Manager，然后在这个文件安装目录的终端下，输入安装命令（记得修改相应的版本）：

```Shell
sudo dpkg -i sdkmanager_2.2.0-12028_amd64.deb
```

然后安装完启动后就是正常的登陆你的Nvidia账号。



# 二、烧录

前期准备就是准备一type-C线，小电脑的电源线，然后还有一根杜邦线。

首先别给小电脑上电，先用杜邦线把小电脑设置为刷机模式，如下图：

![nano_1.png](/img/Documents/nano_1.png)

用杜邦线短接第二个和第三个引脚，然后上电启动小电脑，把type-C线连接小电脑和你的主机，打开刚刚安装的SDK-Manager，进入界面后，如下图所示：

![nano_2.png](/img/Documents/nano_2.png)

然后continue即可，其他基本默认即可，但是遇到这两个记得选择这些选项，剩下的东西安装skip即可，因为现在配置一些东西有点麻烦，也可能会导致后面启动后失败。

![nano_3.png](/img/Documents/nano_3.png)

# 三、启动小电脑

按照上述方法首先有一个很大的问题就是没有网卡驱动，所以首先你会看到小电脑右上角是没有Wi-Fi这个图标的，具体是什么原因，大概就是因为这个方法安装的ubuntu不是Jetson Orin Nano最适配的版本，因为他们官方出过修改过的ubuntu发行版，但是内核不匹配不适合我们的使用，所以只能用这种办法配置ubuntu22.04了，虽然我在大一下的时候，一个人确实花了很多时间找教程，我一直以为是我安装中途出错导致有点包没有装上，所以反复重装了好几遍，花了有好几周，但是最后还是没有放弃，在日本的一个博客的一篇文章中找到了解决办法：

原文链接：[https://qiita.com/dandelion1124/items/f137daab2fd99491b4df](https://qiita.com/dandelion1124/items/f137daab2fd99491b4df)

我就大致把里面的教程简化一下：

1.首先就是去这个网站[https://backports.wiki.kernel.org/index.php/Releases](https://backports.wiki.kernel.org/index.php/Releases)中下载一个类似叫作`backports-5.15.81-1.tar.xz`，本人亲自安装找的时候没有找到一模一样的，但是就找了一个贴近的版本，5.15.153-1，安装过了没有什么问题，但是由于小电脑没法联网，所以就从另外一台设备中下载完拷贝到小电脑上。

2.然后就是在拷贝出来文件的目录下，执行如下代码。

```Shell
tar Jxfv backports-5.15.81-1.tar.xz
cd backports-5.15.81-1
make defconfig-iwlwifi
make -j8
sudo make install
```

虽然原文章还让我改`/etc/modprobe.d/iwlwifi.conf`这个文件，但是我改过了，由于权限问题不让我改，就算我在别的文件夹改完复制进来也不行，不知道为啥，反正我就没整，但是最后小电脑还是没啥问题。

3.最后就是重启小电脑了`sudo reboot`。



