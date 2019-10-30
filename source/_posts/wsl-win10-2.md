---
title: 如何在Win10中使用NMAP
url: 815.html
id: 815
categories:
  - 信息技术
date: 2018-05-26 09:52:34
tags:
 - NMAP
 - WSL
---
`NMAP`是一个很有用的工具，但是如何在Win10里面使用它呢？

<!-- more -->

### 安装WSL

先到控制面板里面把功能打开。
![Windows Features Turn Windows features on or off To turn a feature on, select its check box. To turn a feature off, clear its check box. A filled box means that only pat of the feature is turned on. Telnet Client TFTP Client Windows Defender Application Guard Windows Hypervisor Platform Windows Identity Foundation 3.5 Windows PowerSheII 2.0 Windows Process Activation Service Windows Projected File System (Beta) Windows Subsystem fcr Linux Windows TIFF Filter Work Folders Client Cancel O ](https://cdn.beijing2b.com/wp-content/uploads/2018/09/windows-features-turn-windows-features-on-or-off-1.png)
然后去商店挑一个喜欢的，目前就这几种，估计乌班图的人最多。
![Machine generated alternative text: Microsoft Store Home Apps Games Devices Movies & TV Books Edge Extensions Search P A U 26 Run Linux on Windows Install and run Linux distributions side-by-side on the Windows Subsystem for Linux (WSL). Ubuntu Free openSUSE Leap 42 Owned SUSE Linux Enterprise Serv... Free Debian GNU/ Linux Installed OFFENSIVE Kali Linux Free ](https://cdn.beijing2b.com/wp-content/uploads/2018/09/machine-generated-alternative-text-microsoft-stor-1.png)
装好了以后就像其他APP一样启动。
![All Apps Best match Documents Email Web More v -5 Debian GNU/Linux Trusted Microsoft Store app Apps Remote Desktop Connection Defragment and Optimize Drives Search suggestions P de - See web results Settings (3 ) Folders (2+) Documents (1+) Photos (1 ) P debian GNU/Linux Feedback Debian GNU/Linux Trusted Microsoft Store app Open Run as administrator Pin to Start Pin to taskbar App settings Rate and review Share uninstall ](https://cdn.beijing2b.com/wp-content/uploads/2018/09/all-apps-best-match-documents-email-web-more-1.png)
或者在`PowerShell`里面直接运行`bash.exe`.
![Windows PowerSheII bash. exe $ exit acPro : ogout ](https://cdn.beijing2b.com/wp-content/uploads/2018/09/windows-powersheii-bash-exe-dollar-exit-acpro-og-1.png)

这配色谁特么看得清楚

装了多个的时候可用`wslconfig.exe /s`来指定默认发行版.

```powershell
PS C:\> wslconfig.exe /l
Windows Subsystem for Linux Distributions:
Debian (Default)
Ubuntu
```

第一次运行要大概1分钟时间，输入用户名密码即可。
![Machine generated alternative text: yu@yuMacPro: — Installing, this may take a few minutes.. lease create a default I]IIX user account. The username does not need to match your Windows username. or more information visit: https://aka. ms/wslusers nter new I]IIX username: bn_l nter new I]IIX password: etype new I]IIX password: asswd: password updated successfully Installation successful OyuhacPro ](https://cdn.beijing2b.com/wp-content/uploads/2018/09/machine-generated-alternative-text-yuyumacpro-1.png)
请自行更改软件源，这里不讨论了。 将`C:\Users\username\AppData\Local\Packages\TheDebianProject.DebianGNULinux_76v4gfsz19hv4`加入你的防病毒软件排除列表，~~除非你有牛逼的硬盘~~. 具体路径依据不同的发行版而变化，自行浏览即可。
默认的配色还凑合。
![yu@yuMacPro: — Uyu_MacYro. $ Is /rrnt/c/ Is: cannot read symbolic link ' /rrnt/c/Documents and Is: cannot access ' /rrnt/c/hiberfil. sys : Permission Settings : Permission denied deni ed Recover ,Recyc1e. Bli Shadowsocks (x86) System Volume Information $1_lGM Llsers Windows BJC*AFP B r.ARor- hiberfil. sys msdia80. dll PerfLo s Program Files Program Files and Settings OyuhacPro alias Is:' Is --color=auto OyuhacPro ](https://cdn.beijing2b.com/wp-content/uploads/2018/09/yuyumacpro-uyu_macyro-dollar-is-rrnt-c-is-can-1.png)

* * *

### 安装wsl-terminal

下载并解压缩`wsl-terminal`  [wsl-terminal releases](https://github.com/goreliu/wsl-terminal/releases). 它的 `tools`子文件夹有一些脚本，可以将快捷方式集成到资源管理器右键菜单，运行需要管理员权限，`4-create-start-menu-shortcut.js`可以在开始菜单建立快捷方式。
![Share View This PC NVME (D:) Tools wsl-terminal-tabbed tools Name 1 -add-open-wsl-terminal-here-menujs I-remove-open-wsl-terminal-here-men... 2-add-wsl-terminal-dir-to-pathjs 2-remove-wsl-terminal-dir-from-pathjs 3-write-distro-guids to-config-filejs 4-create-start-menu-shortcut.js 4-create-start-menu-shortcut-login-shell 4-remove-all-start-menu-shortcuts.js 5-add-open-with-vim-menujs 5-remove-open-with-vim-menujs 6-set-default-shell.bat Date modified 4/23/2018 11:10 AM 4/23/2018 11:10 AM Sort by Group by Refresh Type JavaScript File JavaScript File Size Customize this folder... Paste Paste sh ortcut Open wsl-terminal Here Give access o Pro perties Ctrl+Z ](https://cdn.beijing2b.com/wp-content/uploads/2018/09/share-view-this-pc-nvme-d-tools-wsl-terminal-1-1.png)
看起来比微软原生的好多了。 ![VUßvuMacPrc: — Is /mnt/c/ Is: cannot read symbolic link ' /mnt/c/Documents and Settings ' • Is: cannot access ' /mnt/c/hiberfil. sys' • Permission denied acestream z-ez-fe BJCAAPP B)CAROOT Documents and Settings Feke?et}' hiberfil . sys yu@yumacPro : N$ msdia8Ø.d11 OneDriveTem•J PerfLogs DrogramDate Program Files Program Files Recovery $Recyc1e . Bin ShadowsocksR Permission denied Users Windows (x86) SocksCap64-Portab1e-3.6 System Volume Information $UGM ](https://cdn.beijing2b.com/wp-content/uploads/2018/09/vussvumacprc-is-mnt-c-is-cannot-read-symbol-1.png)
![YUCyuMacPro: — top - 18:01:00 up Tasks : total, 11 1ø.9 KiB Mem . 33493224 ø.ø ø.ø 20 s ø.ø ø.ø ø.ø ø.ø 20 s ø.ø ø.ø ø.ø ø.ø 20 s ø.ø ø.ø ø.ø ø.ø 60 s ø.ø ø.ø ø.ø ø.ø ø.ø ø.ø ø.ø ø.ø 2:15, 0 users, 1 running, ø.ø ni, 4.8 sy, total, 2ø519984 total, load average: 0.52, 0.58, 0.59 sleeping, stopped , zombie 84.2 id, ø.ø hi, ø.ø si, ø.ø ø.l st free, 12743888 used, buff/ cache 229352 KiB Swap: PR 20 20 20 20 20 20 20 20 20 20 2ø free, 96 56 used. 2ø6156ø4 avail Mem PID 1 3 4 232 233 239 240 359 360 361 368 USER root root root root root s s ø ø ø ø ø ø ø ø ø VIRT 8304 8304 13696 8304 13696 8304 13696 8304 35464 13684 1616ø SHR 68 C/OCPU C,'OMEM TIME+ ø:oø ø:øø ø:øø ø:øø ø:øø ø:øø ø:øø ø:øø ø:øø ø:øø ø:øø. . 12 .08 .04 .18 .03 .01 init init 596 56 56 596 96 992 2260 1976 428 s 412 s 872 s 2176 s 1436 R bash init bash init bash init wslbridge -+ bash 12 top ](https://cdn.beijing2b.com/wp-content/uploads/2018/09/yucyumacpro-top-180100-up-tasks-total-1.png)
作者整合了一些mintty 主题自己挑一个顺眼的吧，听说`base16-solarized-dark`  逼格很高.
![Options Text Windovv Tr anspar @Off Opa Cursor o Line Looks in Terminal reen -screen. minty base Is-grayscale-dark. minttyrc base16 ra scale-Ii ht. min base 16 reen-screen.min rc base 16-harmonic16-dark. minttyrc base 16-harmonic16-Iight. minttyrc base Is-hopscotch minttyrc base 16-ir-black.minttyrc base 16-isotope minttyrc base 16-london-tube minttyrc base 16-macintosh minttyrc base 16-materia minttyrc base Is-mocha minttyrc base 16-monokai minttyrc base minttyrc base 16-oceanicnext. minttyrc base 16-paraiso minttyrc base 16-phd minttyrc base minttyrc base minttyrc base 16-railscasts. minttyrc base 16-set-ui. minttyrc base 16-shapeshifter minttyrc base minttyrc base 16-solarized-dark. minttyrc base 16-solarized-light. minttyrc base 16-spacemacs. minttyrc base 16-summerfruit. minttyrc base 16-tomorron-night. minttyrc base minttyrc base16-tA'iC ht. min ](https://cdn.beijing2b.com/wp-content/uploads/2018/09/options-text-windovv-tr-anspar-off-opa-curs-1.png)
都看不上的话就自己设计一个主题，放到子文件夹 `theme`里面即可调用。以下是我设计的。
```bash
\# miyunda-dark.minttyrc
\# 以下颜色都是xjb选的
ForegroundColour=180,180,180
BackgroundColour=0,25,25
CursorColour=180,180,180
Black=0,36,48
BoldBlack=101,103,131
Red=210,50,47
BoldRed=230,50,47
Green=133,143,0
BoldGreen=133,163,0
Yellow=171,127,0
BoldYellow=191,141,0
Blue=38,129,200
BoldBlue=38,149,220
Magenta=98,103,186
BoldMagenta=118,123,206
Cyan=42,151,142
BoldCyan=42,171,162
White=140,156,156
BoldWhite=250,240,220
```
 `miyunda-dark`的效果如下，瞎眼文件夹的事情稍后解决。
 ![](https://cdn.beijing2b.com/wp-content/uploads/2018/09/yu-yumacpro-in-114623-dollar-is-mnt-c-is-c-1.png)   只用英语的话可以下载安装 [DejaVu Sans Mono for Powerline](https://github.com/powerline/fonts/blob/master/DejaVuSansMono/DejaVu%20Sans%20Mono%20for%20Powerline.ttf). 咱们中国人, [雅黑Mono](https://github.com/whorusq/sublime-text-3/blob/master/fonts/Microsoft-Yahei-Mono.ttf)很清爽，这是用雅黑与`Consola`~~转基因杂交~~拼装而成的,双击字体文件即可安装，之后就可以调用了。 ![Options Text Windovv Text and Font proper bes Microsoft YaHei Mono, 13pt Leading: -I, Bold: font, Underline: fon Z] Shovv bold as font Font smoo thing u Show bold as color @Default C) None C) Par bal C) Full Allon blinking Character set ](https://cdn.beijing2b.com/wp-content/uploads/2018/09/options-text-windovv-text-and-font-proper-bes-1.png)

* * *

### 安装Zsh

接下来安装`Zsh`。注：`SUSE`不用`apt`管理软件包，请自行搜索。
```bash
sudo apt-get install zsh
sudo apt-get install git-core
```
安装`oh-my-zsh`
```bash
sudo apt-get install wget
wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | zsh
```
报错不用管它——权限问题，将`zsh`设置为默认`Shell`
```bash
chsh -s /bin/zsh
zsh
```
编辑.bashrc
```bash
$ nano ~/.bashrc
```
在最开始加入以下内容。
```bash
# Added to Launch Zsh
if [ -t 1 ]; then
exec zsh
fi
```

注：`nano`编辑完了按`Ctrl+O`回车`Ctrl+X`即可。 `oh-my-zsh`提供了很多主题，还是挑一个顺眼的，[https://github.com/robbyrussell/oh-my-zsh/wiki/Themes](https://github.com/robbyrussell/oh-my-zsh/wiki/Themes) . `ys`主题看起来比较清爽，见下图。 ![Machine generated alternative text: vußyuMacPrc: —/.ch # yu @ yuMacPro $ cd . oh-my-zsh # yu @ yuMacPro $ Is -a cache -mv-zsh in in N/ .&l-my-zsh on git:master o [9:22:26] . gitignore LICENSE . txt lib N/ .#my-zsh on git:master o [9:22:33] custan . git oh -my-zsh . sh plugins README . md thetæs CONTRIBUTING. md # yu @ yuMacPro in ](https://cdn.beijing2b.com/wp-content/uploads/2018/09/machine-generated-alternative-text-vussyumacprc-1.png)
主题的调用方法是编辑 `.zshrc`.
```bash
$ nano ~/.zshrc
```
找到下面这行，把双引号里面的内容替换为主题名字
```bash
ZSH_THEME="robbyrussell"
```
用 [dircolors-solarized](https://github.com/seebi/dircolors-solarized) 解决瞎眼文件夹的事情。
```bash
$ mkdir ~/GitHub
$ cd ~/GitHub
$ git clone [https://github.com/seebi/dircolors-solarized.git](https://github.com/seebi/dircolors-solarized.git)
```
或者就用浏览器下载之后解压复制过来也行。 差不多就是:
```bash
$ cp /mnt/r/Downloads/dircolors-solarized-master/dircolors.256dark ~
```
再次编辑
```bash
$ nano ~/.zshrc
```
```bash
#added for folder colors when ls
eval \`dircolors ~/GitHub/dircolors-solarized/dircolors.256dark\`
```

保存后重启终端。

![YUCyuMacPro: — # yu @ yuMacPro in $ Is /mnt/c Is: cannot read symbolic link ' /mnt/c/Documents and Settings ' : Permission denied Is: cannot access ' /mnt/c/hiberfil.sys ' . Permission denied -3 acestream cache BJCAAPP BJCAROOT Documents and Settings FakePath hiberfil . sys msdia80. dll OneDriveTemp PerfLogs ProgramData Program Files Program Files .6 System Volume Information $UGM Users usr Windows (x86) Recovery $Recyc1e . Bin ShadowsocksR SocksCap64-Portab1e ](https://cdn.beijing2b.com/wp-content/uploads/2018/09/yucyumacpro-yu-yumacpro-in-dollar-is-mnt-c-i-1.png) 以上为最后的效果。

* * *

### 安装NMAP

当然了nmap用不了。 ![VUßyuMacPrc: — yu @ yuMacPro '$ nmap syn Problem binding in [10:52:03] Starting Nmap 7.40 ( https://nmap.org ) at 2018 to interface errno: 92 •socket bindtodevice: Protocol not available Problem binding to interface errno: 92 socket bindtodevice: Protocol not available 10:52 DST NSOCK ERROR [0.04905] Setting of SO BINDTODEVICE failed (IOD #1): NSOCK ERROR [0.04905] Setting of SO BINDTODEVICE failed (IOD #2): NSOCK ERROR CO. 04905] Setting of SO BINDTODEVICE failed (IOD #3): Protocol not available (92) Protocol not available (92) Protocol not available (92) Problem binding to interface socket bindtodevice: Protocol Problem binding to interface socket bindtodevice: Protocol errno: 92 not available errno: 92 not available ](https://cdn.beijing2b.com/wp-content/uploads/2018/09/vussyumacprc-yu-yumacpro-dollar-nmap-syn-proble-1.png) 
下载[Windows版的nmap](https://nmap.org/dist/nmap-7.70-setup.exe) 并安装。 给Windows版的nmap建个别名。
```bash
$ alias nmap='"/mnt/c/Program Files (x86)/Nmap/nmap.exe"'
```
现在可以正常运行了。
![yu @ yuMacPro in alias nmap= ' " /mnt/c/Program Files (x86)/Nmap/nmap. exe"' yu @ yuMacPro in nmap syn Starting Nmap 7.70 ( https://nmap.org ) at 2018@. 11:07 China Standard Time Nmap scan report for syn (192.168.1.28) Host is up (0.00093s latency) . rDNS record for 192.168.1.28: SMN. Ian Not shown: PORT 80/tcp 139/tcp 161/tcp 445/tcp 515/tcp 548/tcp 3260/tcp 3261/tcp 5000/tcp 49160/tcp 50001/tcp 50002/tcp 988 closed ports STATE open open open open open open open open open open open open MAC Address: Nmap done: 1 IP SERVICE http netbios-ssn snmp microsoft-ds printer afp Iscsl winshadow upnp unknown unknown iiimsf address (1 host up) scanned in 6.81 seconds ](https://cdn.beijing2b.com/wp-content/uploads/2018/09/yu-yumacpro-in-alias-nmap-mnt-c-program-f-1.png)
之后保存别名。
```bash
$ nano ~/.bash_profile
```
以下加进去。
```bash
alias nmap='"/mnt/c/Program Files (x86)/Nmap/nmap.exe"'
alias cls='clear'
```
又双叒叕是这个文件。
```bash
$ nano ~/.zshrc
```
粘贴进去保存。
```bash
#added to include bash profile
source ~/.bash_profile
```
好了，基本的设置就完成了，感谢观赏。