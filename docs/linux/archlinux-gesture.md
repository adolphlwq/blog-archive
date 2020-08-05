# kde5与archlinux环境下配置libinput-gestures多手势操作
## 初衷
自己的笔记本是archlinux+kde plasma5的环境，看到同事的macbook pro支持很多`手势操作（gestures）`，自己也希望在archlinux和kde的环境下配置方便的手势操作。

查询资料后发现[touchegg](https://github.com/JoseExposito/touchegg)和[libinput-gestures](https://github.com/bulletmark/libinput-gestures)都可以满足需求，但是在实际配置过程中，touchegg存在很多问题一直都没能解决，而libinput-gestures基本没有太大的问题。所以本文主要介绍如何配置libinput-gestures。

## 预备知识
1. [计算机窗口管理器](https://en.wikipedia.org/wiki/Compositing_window_manager)
2. kde是X.Org桌面环境，[aur](https://aur.archlinux.org/)上维护了一个基于X.Org Server的驱动[xf86-input-libinput-git](https://aur.archlinux.org/packages/xf86-input-libinput-git/)，很方便。它们间的关系如下：
![](https://wayland.freedesktop.org/libinput/doc/latest/dot_libinput-stack-xorg.png)
X11 client在我这里就是kde plasma5。
3. 驱动程序libinput，linux生态提供了很多驱动，本文使用的驱动是[libinput](https://wayland.freedesktop.org/libinput/doc/latest/)，它负责分析内核从输入设备得到的数据，发送给桌面环境，桌面环境根据不同的数据进行反馈。

### 术语解释
- touchpad：通常意义上的触摸板。
- clickpad：指底部没有按钮的touchpad，参考[clickpad](https://wayland.freedesktop.org/libinput/doc/latest/clickpad_softbuttons.html)。clickpad中通过不同手指数量的点击行为来**模拟**鼠标左键、中键、右键点击。这些都可以配置或者关闭。
- click：点击，本文语境中之物理按钮的“按压”和“释放”。
- Clickpad software button behavior：clickpad上软按钮的行为，详情见[Clickpad software button behavior](https://wayland.freedesktop.org/libinput/doc/latest/clickpad_softbuttons.html)。

## libinput

### 安装
1. install basic libinput and xf86-input-libinput
```shell
sudo gpasswd -a $USER input
sudo pacman -S libinput xf86-input-libinput
```

### 配置
libinput有两种配置方式：
- 使用配置文件的永久配置方式
- 使用xinput命令行工具，针对运行时(runtime)进行实时配置，主要用来调试。

#### 配置文件
libinput安装后默认的配置文件在`/usr/share/X11/xorg.conf.d`目录下，如何你安装多个驱动，会存在多个文件：
```shell
➜  xorg.conf.d ll
total 12K
-rw-r--r-- 1 root root 1.4K Aug 14 05:40 10-quirks.conf
-rw-r--r-- 1 root root  964 May  5 20:24 40-libinput.conf
-rw-r--r-- 1 root root 1.8K Nov 18  2016 70-synaptics.conf
```
笔者这里安装了3个驱动，所以有3个配置文件，**默认情况下，kde会根据文件前缀数字的大小决定优先使用哪个配置文件，数字越大，优先级越高。**

我们需要把默认配置文件复制到`/etc/X11/xorg.conf.d/`目录下：
```shell
sudo cp /usr/share/X11/xorg.conf.d/40-libinput.conf /etc/X11/xorg.conf.d/40-libinput.conf
```

下面是文件中touchpad部分的配置：
```shell
Section "InputClass"
        Identifier "touchpad"
        MatchIsTouchpad "on"
        MatchDevicePath "/dev/input/event*"
        Driver "libinput"
        Option "Tapping" "on"
        Option "ButtonMapping" "1 3 0 4 5 6 7"
        Option "TappingButtonMap" "lmr"
        Option "DisableWhileTyping" "on"
        Option "TappingDrag" "on"
EndSection
```
详细参数和解释见[libinput man page: based on X.Org input dirver](https://www.mankier.com/4/libinput#Configuration_Details)，解释下几个重要的配置参数：
- Option "Tapping" "on"：手指点击touchpad发送鼠标点击事件
- Option "TappingButtonMap" "lmr"：1个手指点击对应`鼠标左键`，2个手指点击对应`鼠标中键`，3个鼠标点击对应`鼠标右键`。
- Option "ButtonMapping" "1 3 0 4 5 6 7"，按钮映射，详情见[libinput#Button_Mapping](https://www.mankier.com/4/libinput#Button_Mapping)，这里笔者关闭了3指对应的左键。
- Option "DisableWhileTyping" "on"：打字时不检测touchpad事件，防止用户不小心触碰touchpad引起不必要的影响。
- Option "TappingDrag" "on"：开启点击拖拽。

#### 调试
1. 确定touchpad设备
```shell
➜  ~ xinput list
⎡ Virtual core pointer                          id=2    [master pointer  (3)]
⎜   ↳ Virtual core XTEST pointer                id=4    [slave  pointer  (2)]
⎜   ↳ SynPS/2 Synaptics TouchPad                id=13   [slave  pointer  (2)]
⎜   ↳ TPPS/2 IBM TrackPoint                     id=16   [slave  pointer  (2)]
......
```
2. 查看touchpad的详细配置
```shell
➜  ~ xinput list-props "SynPS/2 Synaptics TouchPad"
Device 'SynPS/2 Synaptics TouchPad':
        Device Enabled (142):   1
        Coordinate Transformation Matrix (144): 1.000000, 0.000000, 0.000000, 0.000000, 1.000000, 0.000000, 0.000000, 0.000000, 1.000000
        libinput Tapping Enabled (295): 1
        libinput Tapping Enabled Default (296): 0
        libinput Tapping Drag Enabled (297):    1
        libinput Tapping Drag Enabled Default (298):    1
        libinput Tapping Drag Lock Enabled (299):       0
        libinput Tapping Drag Lock Enabled Default (300):       0
        libinput Tapping Button Mapping Enabled (301):  0, 1
        libinput Tapping Button Mapping Default (302):  1, 0
        libinput Accel Speed (277):     0.000000
        libinput Accel Speed Default (278):     0.000000
        libinput Natural Scrolling Enabled (282):       0
        libinput Natural Scrolling Enabled Default (283):       0
        libinput Send Events Modes Available (262):     1, 1
        libinput Send Events Mode Enabled (263):        0, 0
        libinput Send Events Mode Enabled Default (264):        0, 0
        libinput Left Handed Enabled (284):     0
        libinput Left Handed Enabled Default (285):     0
        libinput Scroll Methods Available (286):        1, 1, 0
        libinput Scroll Method Enabled (287):   1, 0, 0
        libinput Scroll Method Enabled Default (288):   1, 0, 0
        libinput Click Methods Available (303): 1, 1
        libinput Click Method Enabled (304):    1, 0
        libinput Click Method Enabled Default (305):    1, 0
        libinput Middle Emulation Enabled (291):        0
        libinput Middle Emulation Enabled Default (292):        0
        libinput Disable While Typing Enabled (306):    1
        libinput Disable While Typing Enabled Default (307):    1
        Device Node (265):      "/dev/input/event10"
        Device Product ID (266):        2, 7
        libinput Drag Lock Buttons (293):       <no items>
        libinput Horizontal Scroll Enabled (294):       1
```
这里面有一个值很重要**Device Node (265):      "/dev/input/event10"**，后面使用libinput命令行时会用到这个路径。
3. 使用libinput debug-events 监控touchpad事件
```shell
➜  ~ libinput debug-events --device /dev/input/event10
-event10  DEVICE_ADDED     SynPS/2 Synaptics TouchPad        seat0 default group1  cap:pg  size 100x56mm tap(dl off) left scroll-nat scroll-2fg-edge click-buttonareas-clickfinger dwt-on
 event10  GESTURE_SWIPE_BEGIN  +2.82s   3
 event10  GESTURE_SWIPE_UPDATE  +2.82s  3  0.00/ 0.44 ( 0.00/ 3.63 unaccelerated)
 event10  GESTURE_SWIPE_UPDATE  +2.84s  3  0.00/ 3.43 ( 0.00/19.38 unaccelerated)
 event10  GESTURE_SWIPE_UPDATE  +2.87s  3  0.00/ 5.00 ( 0.00/19.38 unaccelerated)
 event10  GESTURE_SWIPE_UPDATE  +2.89s  3  0.00/ 3.33 ( 0.00/10.90 unaccelerated)
 event10  GESTURE_SWIPE_UPDATE  +2.92s  3  0.00/ 2.74 ( 0.00/ 8.48 unaccelerated)
```
4. 使用libinput debug-gui 监控touchpad事件
```shell
➜  ~ libinput debug-gui --device /dev/input/event10
info: event10 SynPS/2 Synaptics TouchPad     added
```
这时会出现一个GUI画面帮助检测。
![kde-libinput-gestures-001](/content/images/2017/09/kde-libinput-gestures-001.png)

## libinput-gestures
配置好touchpad和手势后，下面利用`libinput-gestures`来解析touchpad的数据，然后执行相关的操作。这里主要用到[xdotool](http://www.semicomplete.com/projects/xdotool/)，xdotool是模拟键盘/鼠标输入和窗口管理等的命令行工具。libinput-gestures依赖xdotool。

libinput-gestures安装后会有默认的配置，位置在`/etc/libinput-gestures.conf`，用户可以在`～/libinput-gestures.conf`配置自己的配置。笔者根据自己的需要修改了相关配置，如下：
```shell
# Switch to next desktop
gesture swipe right 4   xdotool key ctrl+F1
# Switch to prev desktop
gesture swipe left 4 xdotool key ctrl+F2

# Present windows (current desktop)
gesture swipe down 3 xdotool key ctrl+F9
# Present windows (all desktop)
gesture swipe down 4 xdotool key ctrl+F10

# Show desktop
gesture swipe up 3 xdotool key super+d
# Show desktops grid
gesture swipe up 4 xdotool key ctrl+F8
```
**主要思想是针对不同的手势触发相关的快捷键**，快捷键的配置则可以在系统偏好设置-->快捷键中设置。下图是笔者在plasma5中的切换桌面的快捷键配置：
![kde-libinput-gestures-002](/content/images/2017/09/kde-libinput-gestures-002.png)

### Demo
1. 四指横向滑动切换桌面：
![kde-gestures-demo-1-min](/content/images/2017/09/kde-gestures-demo-1-min.gif)
2. 四指上下滑动显示所有桌面和所有活动窗口：
![kde-gestures-demo-2-min](/content/images/2017/09/kde-gestures-demo-2-min.gif)

## TODOs
### 通过捏和(pinch in/pinch out)来放大/缩小网页（对标macbook）
`$HOME/.config/libinput-gestures.conf`中添加如下内容（方向可以自己定义）：
```vim
# back history of chromium/chrome
gesture swipe right 3   xdotool key alt+Left
gesture swipe left 3   xdotool key alt+Right
```
### 网页的前进/后退（对标macbook）
`$HOME/.config/libinput-gestures.conf`中添加如下内容
```vim
# pinch 
gesture pinch in 2 xdotool key ctrl+minus # 2指捏: 缩小
gesture pinch out 2 xdotool key ctrl+plus # 2指张: 放大
```

## 总结
本文从想法到配置好前前后后花了大约1个月时间，大部分时间用在了理解输入驱动、配置驱动以及配置调试toucgegg和libinput-gestures上了。

笔者在配置好自己的archlinux 手势后，机缘巧合用了一个星期的macbook pro，体会了苹果下面的手势操作。总体感觉苹果的手势操作更流畅，识别更准确。毕竟苹果是自己的生态系统，可以针对自己的macOS系统进行封装和调试。而在Linux生态中，因为存在多个Linux发行版和桌面环境，手势操作很难兼容所有发行版，这就要求用户要有较强的动手能力和理解能力。可以说两者都能实现丰富的手势操作，而且Linux的扩展性更强一些，但是需要更专业的知识和动手能力。

## 相关参考
- [GitHub: libinput-gestures](https://github.com/bulletmark/libinput-gestures)
- [arch wiki: libinput](https://wiki.archlinux.org/index.php/Libinput)
- [libinput official doc: related pages](https://wayland.freedesktop.org/libinput/doc/latest/pages.html)
- [libinput man page](http://jlk.fjfi.cvut.cz/arch/manpages/man/libinput.4)
- [libinput man page: based on X.Org input dirver](https://www.mankier.com/4/libinput#Configuration_Details)