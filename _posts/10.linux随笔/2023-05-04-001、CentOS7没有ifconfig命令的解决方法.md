今天安装了CentOS7，选择的是最小化模式，安装完成后，输入ifconfig提示command
not found（未找到命令），吃了一惊。

应该是最小化安装模式的问题，ifconfig是最基本的命令，我认为再省也不能省这个。

肯定是相关的软件包没有安装，开始解决问题吧。

## 1、找到ifconfig命令所在的软件包

[yum search ifconfig]{.mark}

![](/images/31/media/image1.png){width="7.268055555555556in"
height="1.8in"}

提示找到了，软件包名是net-tools（基本的网络工具）。

## 2、安装net-tools软件包

以下两个命令都可以安装net-tools软件包，效果相同。

[yum -y install net-tools]{.mark}

或

[yum -y install net-tools.x86_64]{.mark}

![](/images/31/media/image2.png){width="7.2652777777777775in"
height="3.4090277777777778in"}

## 3、验证

输入ifconfig，得到以下结果，图中标记出来的就是CentOS7的ip地址。

![](/images/31/media/image3.png){width="7.268055555555556in"
height="2.0229166666666667in"}

## 4、试试ip addr命令

如果只是查看IP地址，也可以用ip
addr命令，即使最小化安装的Linux系统也有这个命令，如下：

![](/images/31/media/image4.png){width="7.268055555555556in"
height="2.76875in"}

## 5、文章版权

C语言技术网原创文章，转载请说明文章的来源、作者和原文的链接。

来源：C语言技术网（[www.freecplus.net](http://www.freecplus.net)）

作者：码农有道

如果这篇文章对您有帮助，请点赞支持，或在您的博客中转发此文，让更多的人可以看到它，谢谢！！！
