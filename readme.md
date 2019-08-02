# Linux下通过Wine安装微信

<details>
<summary>点击打开目录</summary>
<!-- MarkdownTOC autolink="true" -->

- [安装步骤](#%E5%AE%89%E8%A3%85%E6%AD%A5%E9%AA%A4)
	- [安装Wine 4.0 \(不一定必须\)](#%E5%AE%89%E8%A3%85wine-40-%E4%B8%8D%E4%B8%80%E5%AE%9A%E5%BF%85%E9%A1%BB)
	- [安装最新版的`winetricks`](#%E5%AE%89%E8%A3%85%E6%9C%80%E6%96%B0%E7%89%88%E7%9A%84winetricks)
	- [配置Wine bottle](#%E9%85%8D%E7%BD%AEwine-bottle)
	- [修改默认的`.desktop`链接](#%E4%BF%AE%E6%94%B9%E9%BB%98%E8%AE%A4%E7%9A%84desktop%E9%93%BE%E6%8E%A5)
	- [启动](#%E5%90%AF%E5%8A%A8)
- [参考链接](#%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)

<!-- /MarkdownTOC -->
</details>

### 安装步骤

#### 安装Wine 4.0 (不一定必须)

- 卸载旧版的Wine

如果当前的系统中已经安装有wine, 但版本不是4.0, 可以考虑先卸载再安装（否则可能提示无法安装4.0版本）。  
卸载方式如下<sup>[4]</sup>: 

```
sudo apt-get --purge remove wine
```

相应若干后续的操作可以参考[4]中剩余的指令。

- 安装4.0版wine

根据系统版本选择相应的配置下载方式, [5],[6]分别给出了在Ubuntu 16.04/Linux Mint 18以及Ubuntu 18.04/Linux Mint 19的配置安装方式。基本流程一致: 

1. 启用32位架构
```
sudo dpkg --add-architecture i386
```
2. 下载并添加repository key
```
wget -qO - https://dl.winehq.org/wine-builds/winehq.key | sudo apt-key add -
```
若提示`wget`未安装, 则通过以下命令安装
```
sudo apt-get -y install wget
```
3. 添加Wine repository
```
sudo apt-add-repository 'deb https://dl.winehq.org/wine-builds/ubuntu/ xenial main'
```
此处需要根据系统版本相应修改`xenial`, 例如我的系统是Linux Mint 18, 就无需修改, 其他系统可以相应根据系统名称修改。  

4. 安装Wine
```
sudo apt-get update
sudo apt install --install-recommends winehq-stable
```
5. 确认安装成功
```
$ wine --version
wine-4.0
```

#### 安装最新版的`winetricks`

微信的正常使用需要配置相应的依赖文件, 而依赖是通过`winetricks`安装的, 但是通过`apt-get install`按照的`winetricks`版本比较老, 相应提供的依赖可能有问题, 因此最好是安装最新的版本<sup>[3]</sup>。  
安装方法如下: 
1. 卸载已安装的旧版
```
sudo apt-get remove winetricks
```
2. 获取新版
```
wget  https://raw.githubusercontent.com/Winetricks/winetricks/master/src/winetricks
```
3. 修改权限为可执行
```
chmod +x winetricks
```
4. 将可执行文件放置于terminal可调用的目录下
```
sudo mv -v winetricks /usr/local/bin
```

#### 配置Wine bottle

**这是关键一步**。通过[2]这个视频学到了Wine的使用逻辑: 不同于将所有的东西都塞到默认的`.wine`文件夹下, 这个视频详细展示了如何构建一个wine bottle, 配置它, 再安装相应的windows应用程序。我理解wine bottle就和anaconda里的env, 或者是docker的一个image, 都是一个**微型的隔离的**虚拟机。你可以在wine bottle中单独配置相应的程序所需的依赖。这里我们只需要安装微信, 为了解决聊天框无法显示输入的问题, 需要配置相应的富文本dll文件<sup>[1]</sup>: `riched20.dll`（我还添加了`riched32.dll`）下面给出配置流程:  

1. 创建并进入目录
```
mkdir Wine
cd Wine
```
2. 初始化wine bottle config
```
WINARCH=win32 WINEPREFIX=/home/frank/Wine/WeChat winecfg
```
注意其中的几个关键点: `WINARCH=win32`是将架构配置为32位以便提供更好的兼容性, `WINEPREFIX`设置了相应bottle所在的目录, 我命名为`WeChat`, 最后是`winecfg`即启动wine的初始化设置。执行后会提示若干的`err`或`fixme`, 不用管, 直至弹出`winecfg`的界面, 4.0版本下默认是windows 7的配置, 保留默认设置即可。  

3. 通过`winetricks`添加依赖项目
```
WINARCH=win32 WINEPREFIX=/home/frank/Wine/WeChat winetricks
```
与以上命令类似, 只需要将`winecfg`替换为`winetricks`即可, 然后: 

	选择默认的Wine容器 -> OK -> 安装Windows DLL组件 -> OK -> 勾选riched20.dll（以及riched32.dll） -> OK -> 等待安装即可。  

4. 下载微信安装包
直接在微信官网下载微信PC版安装包即可, 下载后将安装包置于`Wine/`下  

5. 安装微信
```
WINARCH=win32 WINEPREFIX=/home/frank/Wine/WeChat wine WeChat_C1018.exe
```
仍然与上述命令类似, 将`winecfg`改为`wine`, 然后接安装包的名称, 等待安装完毕即可, 该步骤与windows上安装没有区别。

#### 修改默认的`.desktop`链接

以上步骤执行完毕后在开始菜单, wine下将出现微信的图标, 桌面也会出现微信的快捷方式, 但是点击并没有微信窗口弹出。其原因在于链接地址有问题, 需要相应进行修改。修改方式如下:  

1. 定位到相应的`.desktop`文件
```
cd ~/.local/share/applications/wine/Programs/微信/微信.desktop
```

2. 修改`.desktop`文件
用任意编辑器打开即可, 将`Exec=`这一行修改如下:  
```
Exec=env WINEPREFIX="/home/frank/Wine/WeChat" /usr/bin/wine explorer /desktop=wechat, 1920x1080 "/home/frank/Wine/WeChat/drive_c/Program Files/Tencent/WeChat/WeChat.exe"
```
其中几个关键词解释如下:

	/usr/bin/wine: wine执行程序所在目录
	explorer: 启动窗口
	/desktop=wechat: 窗口名称
	, 1920x1080: 窗口分辨率
	"/home/frank/Wine/WeChat/drive_c/Program Files/Tencent/WeChat/WeChat.exe": 微信执行程序所在的绝对路径

修改后保存即可, 如此便可以从开始菜单, wine下的微信图标启动微信了！  
**注**:桌面快捷方式不会自动更新, 可以通过在开始菜单中右键微信添加到桌面更新之。

#### 启动

<p align="center">
	<img src="http://ww1.sinaimg.cn/large/93d8f721gy1g5l54q1p4rj20m30hgdi7.jpg" width="800" alt="screenshot">
</p>

### 参考链接

1. [Linux下的wine生活(QQ/微信/Office)](https://www.cnblogs.com/makefile/p/wine-life.html)  
2. [How to Install and Use Wine to Run Windows Applications on Linux
](https://www.youtube.com/watch?v=RmOdA5GeSqs)
3. [winetricks sha1sum mismatch rename and try again](https://askubuntu.com/a/750333)
4. [How to remove wine completely](https://askubuntu.com/a/126745)
5. [How to Install Wine 4 on Ubuntu 16.04 / 18.10 / Linux Mint 18](https://computingforgeeks.com/how-to-install-wine-4-on-ubuntu-16-04-18-10-linux-mint-18/)
6. [How to Install Wine 4 on Ubuntu 18.04 / Linux Mint 19](https://computingforgeeks.com/how-to-install-wine-4-on-ubuntu-18-04-linux-mint-19/)