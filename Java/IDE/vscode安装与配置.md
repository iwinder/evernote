##  安装
### 下载

地址 ：https://code.visualstudio.com/Download

这里使用的下载到的是code-stable-1542309103.tar.gz。

### 解压
```
tar -zxvf code-stable-1542309103.tar.gz -C /home/wind/apps/web/
```

## 配置环境变量
```
sudo vi /etc/profile
```
配置并保存：
```
export PATH=$PATH:/home/wind/apps/web/VSCode-linux-x64/bin
```
运行
```
source /etc/profile
```
使配置生效。

这样就可以在命令窗口使用 code命令，如
```
code .
```
###  打不开时的解决方案
刚开始安好很正常，后来不知安装哪个软件的缘故，导致始终打不开vscode了，重启也不管用，后来用
```
code .
```
命令启动了一下被告知是缺少libgconf-2.so.4，于是最终通过使用如下操作解决了问题：
```
sudo apt -y install libgconf2-4
```
这也可以当是配置code命令的一个优点吧。

## 创建快捷键

拷贝ico图标

```
cp /home/wind/apps/web/VSCode-linux-x64/resources/app/resources/linux/code.png /home/wind/apps/web/VSCode-linux-x64/icons/
```

使用
```
sudo vim /usr/share/applications/VSCode.desktop
```
输入：
```
[Desktop Entry]
Name=Visual Studio Code
Comment=Multi-platform code editor for Linux
Exec=/home/wind/apps/web/VSCode-linux-x64/code
Icon=/home/wind/apps/web/VSCode-linux-x64/icons/code.png
Type=Application
StartupNotify=true
Categories=TextEditor;Development;Utility;
MimeType=text/plain;
```
保存后即可在应用程序列表 找到启动快捷键，若想放到桌面，可执行如下命令
```
cp /usr/share/applications/VSCode.desktop ~/桌面/
```
## 常用插件

### Paste Image
一个可以将剪切板中图片粘贴进Markdown文件中的插件。

使用过程中若被提示需要安装xclip，可执行如下命令：
```
sudo apt-get install xclip
```
###  TSLint

用于开发angular项目时对TS代码进行格式检测。

###  Chinese (Simplified) Language Pack for Visual Studio Code

汉化编辑器的插件。

### GitLens — Git supercharged

可以在代码中查看修改者及其修改时间