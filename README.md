# 第一篇：clion_carto_ws工作空间搭建方法

[TOC]

> 工欲善其事必先利其器！

## 0. 引言

### 0.1 这篇文章是想解决什么问题？

​	cartographer(以下简称carto)的代码因为有严谨的架构设计，代码量大，文件多，质量高，用到了许多c++的语法知识和技巧，要想掌握好carto的代码变得不是那么容易。

​	对于新人，还有一个造成我们carto代码阅读感觉非常吃力的原因就是：没有一种”用IDE来对cartographer和cartographer_ros两个工作目录**同时进行管理、编译、debug单步调试**“的方法。

​	这就导致了一个严重的问题：**无法在cartographer和cartographer_ros的代码之间方便又准确地进行代码跳转**，也失去了对于现代开发者而言用IDE开发的巨大效率提升优势。进一步导致我们**很难去梳理清楚carto整个的代码逻辑**，只有经验丰富的开发人员才可能会比较清楚地阅读代码。但，我相信他们在没有借助IDE来看carto代码的过程也是很难受的。

​	因为这个，有很长一段时间我都提不起劲去仔细阅读研究carto的每一行代码(我们为自己的懒惰找借口的时候，总是最勤快的，哈哈)

​	为解决这个问题，我曾经费了不少功夫，尝试过很多方法：

1）完成对carto工作空间在终端里用命令行编译后，用vscode来直接打开。我相信这应该是大家用得最多的办法。

**优点：**

- 可以同时管理cartographer和cartographer_ros的代码；


- 打开迅速；


- 具有部分IDE的功能;
- 有丰富的插件生态。

**缺点：**

- 会有很多头文件报错，即使配置了头文件准确路径也还是会报错；


- 无法实现代码的准确跳转，有时直接跳不过去，有时会列出很多的可能，不方便；


- 无法直接进行代码编译、debug单步调试；


2）ROS Qt Creator，KDevelope也尝试过，算是走过的弯路，就不讲了。

3）使用c++开发专门的IDE：clion

> clion是什么？我相信搞Java开发的朋友应该都知道 IntelliJ IDEA 吧，没错，他俩是一个妈生的，而且还是双胞胎。

**优点：**

- 这是一个很强大的IDE，有许多提升开发者效率的功能，颜值高，背后是强大的JetBrains团队在进行开发维护。

**缺点：**

- 只能对一个工作空间(含有总cmakelist.txt)进行管理。就是说，你可以用clion单独管理和编译cartographer工程或cartographer_ros工程，这样就不是那么方便了。



​	折腾这么久，可算是让我找到了最终解！今天就是要把它分享给大家。该方法的**核心思想**就是：**通过cmake来组织**cartographer和cartographer_ros两个工程，即：实现了**用一个额外的总CMakeLists.txt来统一管理cartographer和cartographer_ros**，这样clion也就可以同时管理两者，从而解决了方法3)的缺点。从此**carto代码的阅读效率直线拉升！**

​	我录了一个我们常用的查看定义的效果视频，让大家先爽一下：

以查看map_builder怎么被创建为例。你可以看到已经能够一路从 cartographer_ros代码 跳转到 cartographer 代码里了。



### 0.2 一点特别说明

​	该方法的核心思想不是我第一个想到的，也是在我偶然间看到github上有人做了这个事情，仓库名叫做

 cartographer_superbuild ：https://github.com/larics/cartographer_superbuild

​	没有大问题，有3个小点不太好：

- 步骤描述得相对简单，缺少一些图片描述，对于不熟悉carto和clion的人显得不太友好；

- 该仓库的文件组织方式我不太喜欢：没有src目录，devel放在了build里。我还是习惯了有一个单独的总src目录，build和devel分开的方式，就像我们平时开发ros工程里的那样，所以我进行了一些修改。最终呈现的效果图如下：

<img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210822192640281.png" alt="image-20210822192640281" style="zoom:33%;" />

- 除此之外，这位原作者只提供了用cmke命令行在终端里编译的方法。



​	我会仔细描述在clion中如何配置，进而可以使用clion来对工作空间进行管理、编译、debug，更易用。

​	喜欢在终端里用命令行编译的同学，我还**会提供一种配合ninja编译**的方法，这样你的**编译速度会更快**！(理论上clion上应该也是可以配置用cmake+ninja的，我还没摸索出来，如果有这方面经验的同学欢迎补充啊，感激不尽!)

​	同时，你还会发现有一个好处：就是**当修改了lua、launch、rviz画面布局后，都可以直接生效**，不需要再次编译carto代码才能生效了。效率又提高了一丢丢。

> 这里特别小讲一下,也是我去了解cmake，catkin_make后分析出的。
>
> 如果你还是catkin_make_isolated的编译方式,那么不要按照carto官方的编译命令，要去掉--install,即:
>
> ```
> catkin_make_isolated --use-ninja
> source devel_isolated/setup.bash
> ```
>
> 也可以达到同样的效果!
>

---

​																						**好了，正式开始搞事情吧！**

我是以 **Ubuntu 18.04 + ROS Melodic + CLion 2019.3.6** 为例的，我也推荐这种组合。ubuntu 16有点老了，在今年就会停止更新维护。

## 1. 下载管理代码

```
#进入root
sudo su
#给hosts文件最后加上 185.199.108.133 raw.githubusercontent.com
#这样之后你下载github仓库代码的速度会提升很大，以后下载carto,abseil代码,安装ros或者以后其它工程的时候遇到的问题也会少很多
echo "185.199.108.133 raw.githubusercontent.com" >> /etc/hosts
#退出root
exit
#我的这个仓库直接是一个工作空间,所以可以直接在你的~目录下运行即可,不需要你手动新建工作空间,不需要进入你的src目录
git clone https://github.com/XiaoJake/clion_carto_ws.git
```

## 2. 加载cartographer和cartographer_ros

- **方式1：用网络下载官方最新原版carto代码:**

```
cd ~/clion_carto_ws
wstool init src
wstool merge -t src https://raw.githubusercontent.com/cartographer-project/cartographer_ros/master/cartographer_ros.rosinstall
wstool update -t src
```

- **方式2：在src目录中直接放入自己的carto代码:**

如果你有自己在注释、优化的carto代码，你可以直接把你的cartographer和cartographer_ros放入~/clion_carto_ws/src目录中即可

## 3. 代码环境安装

### 3.1 ROS Melodic 安装

roswiki官方文档就是最好的:

http://wiki.ros.org/melodic/Installation/Ubuntu

其中讲到的这句命令希望你一定运行它，这样你每次Ctrl+Alt+T新开终端就会自动加载ros环境。

```
echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
```

### 3.2 cartographer环境安装

与carto官方指导文档步骤差不太多：

```
sudo apt-get update
sudo apt-get install -y python-wstool python-rosdep ninja-build stow
cd ~/clion_carto_ws/
sudo rosdep init
rosdep update
rosdep install --from-paths src --ignore-src --rosdistro=${ROS_DISTRO} -y
sudo apt-get remove ros-${ROS_DISTRO}-abseil-cpp
src/cartographer/scripts/install_abseil.sh
```

> **注意：**
>
> 如果你是ubuntu16.04还需要额外运行下面的命令：这是由于Ubuntu 16及以下系统自带的protobuf版本小于3.0，所以需要运行下面这句代码来源码安装3.4版本的protobuf。ubuntu18.04系统就可以直接略过
>
> ```
> src/cartographer/scripts/install_proto3.sh
> ```
>
> 这里还有个坑爹的地方需要注意!运行上面这句命令后，会在src同级目录下多出一个protobuf目录，在上面的命令运行成功后，要把这个protobuf目录删除！不然，你后面编译carto代码会有proto相关的报错！这个真是非常非常坑，我折腾了好久才发现是这里的问题。。。
>

## 4. 编译&运行工程代码

- **方式1：可与clion兼容的make编译方式:**

(本文推荐这种,目前只有这种方式才可以和clion无缝联动？)

```
cd ~/clion_carto_ws/
mkdir build;cd build
cmake .. -DCATKIN_DEVEL_PREFIX=../devel
make
```

- **方式2：只在终端编译，可用ninja来实现快速编译:**

```
cd ~/clion_carto_ws/
mkdir build;cd build
cmake .. -DCATKIN_DEVEL_PREFIX=../devel -G Ninja
ninja
```

- **好了，运行一下熟悉的官方2d demo看看:**

```
cd ~/clion_carto_ws
source devel/setup.zsh
wget -P ~/Downloads https://storage.googleapis.com/cartographer-public-data/bags/backpack_2d/cartographer_paper_deutsches_museum.bag
roslaunch cartographer_ros demo_backpack_2d.launch bag_filename:=${HOME}/Downloads/cartographer_paper_deutsches_museum.bag
```

<img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210820231916202.png" alt="image-20210820231916202" style="zoom:33%;" />

## 5. 配置clion与clion_carto_ws工程

CLion下载地址：https://www.jetbrains.com/clion/download/other.html

CLion 2019.3.6 安装中 ...

等你把CLion安装好之后，准备工作就算全部做好了。

现在讲讲怎么去配合使用CLion，相信我，使用一段时间后你就会发现它的无比魅力！

### 5.1 启动CLion

```
cd ~/clion_carto_ws
#加载ros环境
source devel/setup.bash
#实现带ros环境启动clion。这行命令要根据你的clion安装目录替换，找到clion.sh文件拖动到终端窗口，回车即可。这是在我电脑上的运行例子
~/APP/clion-2019.3.6/bin/clion.sh
```

### 5.2 在clion中打开clion_carto_ws

- `File | Open...`

<img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210821001419692.png" alt="image-20210821001419692" style="zoom:33%;" />

- 选择clion_carto_ws目录下的CMakeLists.txt 

<img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210821001513399.png" alt="image-20210821001513399" style="zoom:33%;" />

- Open as Project

<img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210821001543270.png" alt="image-20210821001543270" style="zoom: 50%;" />

- 一开始会有这个报错，可以不管，后面配置好之后就会消失：

<img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210821001641414.png" alt="image-20210821001641414" style="zoom:50%;" />

### 5.3 配置好CMake环境

- `File | Settings... | Build, Execution, Deployment | Cmake`

> ! 下面需要用到~目录的绝对路径，每个人都不同，我的是/home/robot。
>
> 即，下面提到的 /home/robot 字符你都需要替换为： /home/你的用户名

- `Build type行选择 RelWithDebinfo 模式(为以后调试我们的carto代码做准备,如果不调试,平时你也可选择release)`

- `设置devel文件夹路径。在CMake options行输入：`

```
-DCATKIN_DEVEL_PREFIX:PATH=/home/robot/carto_clion_ws/devel
```

- `设置build文件夹路径。在Generation path行输入：`

```
/home/robot/carto_clion_ws/build
```

- `配置编译时需要用到的线程数，量力而行，我是配置成系统最大线程数-2=8-2=6。在Build options行输入：`

```
-- -j 6
```

<img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210822200032857.png" alt="image-20210822200032857" style="zoom: 33%;" />

- 问题解决：

很奇怪，官方的源码编译时都会遇到报错：找不到 GMOCK_LIBRARY 

<img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210822200320082.png" alt="image-20210822200320082" style="zoom: 50%;" />

只需要在carto源码里做点小修改：

1）到/home/robot/clion_carto_ws/src/cartographer/CMakeLists.txt 的大概313行，修改成这样：

```
#target_link_libraries(${TEST_LIB} PUBLIC ${GMOCK_LIBRARY})
target_link_libraries(${TEST_LIB} PUBLIC ${GMOCK_LIBRARIES})
```

<img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210822204030850.png" alt="image-20210822204030850" style="zoom: 25%;" />

2）对背后的原因感兴趣的朋友，可以看这里: https://github.com/cartographer-project/cartographer/issues/1611

<img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210822202454284.png" alt="image-20210822202454284" style="zoom:25%;" />

### 5.4 编译并配置 Run/Debug环境 (可实现代码自动补全)

1）**先在CLIon中完成编译**。这个小锤图标就是"编译"，也可以按快捷键：Ctrl+F9

<img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210822211802884.png" alt="image-20210822211802884" style="zoom: 50%;" />

2）编译成功后

`点击下拉箭头 | Edit Configurations... | 在左侧找到并点击 cartographer_node | 点击 working directory 行的文件夹图标 | 只选择到clion_carto_ws工作空间目录这里就可以了`

<img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210822211331807.png" alt="image-20210822211331807" style="zoom: 33%;" />

<img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210821002628069.png" alt="image-20210821002628069" style="zoom: 50%;" />



**至此，已经完成了所有关键工作，你已经可以在carto的代码世界里畅游了！**

## 6.CLion插件推荐、使用技巧与美化

先看看对比效果吧

优化clion之前：

<img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210822213526009.png" alt="image-20210822213526009" style="zoom: 25%;" />

优化clion之后：

<img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210821004831964.png" alt="image-20210821004831964" style="zoom: 33%;" />

### 6.1 插件安装

`File | Settings... | plugins | 搜索插件名,并点击install来安装，全部安装完之后需要重启CLion才会生效`

<img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210821004027115.png" alt="image-20210821004027115" style="zoom:33%;" />

推荐安装的插件：

​	`Dracula Theme` : 这是我喜欢的一种暗紫色的吸血鬼风格主题，会让你的界面和代码颜色都产生变化，阅读起来很醒目又护眼，不累

​	`Atom Material Icons`：给侧边栏显示的文件，根据不同文件类型匹配一个不同的图标，也是为了看起来更醒目，更好看

​	`HighlightBracketPair`：高亮代码里匹配的{}等,并用竖线连接起来

​	`Rainbow Brackets`：给代码里匹配的括号分配不同的颜色

​	`Grep Console`：在调试时，对调试内容输出窗口的 info，warning，error，fatal等信息行分配不同的颜色，让你更改的阅读内容

插件安装3-7个就够了。clion本身就很强大，集成了很多小功能，已经足够你去探索了。插件安装太多会卡，反而体验不好。

​	Dracula Theme 提供了多种风格，默认加载的不是很好看，推荐选择 Dracula Colorful 风格

`File | Settings... | Appearance | 点击Theme的下拉箭头，选择 Dracula Colorful | 点击OK后会直接生效`

<img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210821004645737.png" alt="image-20210821004645737" style="zoom:50%;" />

### 6.2 优化clion的启动方式

第一次启动clion的时候需要新开终端，并在命令行里加载ros环境，运行CLion软件启动脚本，每次启动都这样，未免有点麻烦了。

下面的方法是实现在侧边任务栏点击一下图标就实现启动：

1）在任务栏右键clion图标 | Add to Favorites ，这样你每次就可以直接侧边栏的clion图标来启动了

<img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210821003340398.png" alt="image-20210821003340398" style="zoom:25%;" />

2）编辑clion的图标启动脚本，让clion每次启动自动加载ros环境(不加载ros环境clion会无法编译，报错)

```
gedit ~/.local/share/applications/jetbrains-clion.desktop
```

Exec="/home/robot/APP/clion-2019.3.6/bin/clion.sh" %f

改为：

Exec=bash -i -c "/home/robot/APP/clion-2019.3.6/bin/clion.sh" %f

<img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210821003640876.png" alt="image-20210821003640876" style="zoom: 50%;" />

> **注意：**
>
> 在我vmware虚拟机的ubuntu 18.04系统中，这样设置后，我从侧边任务栏单机或者桌面双击CLion图标，会无法启动，只会新增一个cpu占用100%的bash线程。但是开机后的第一次点击或者去到 ~/.local/share/applications/ 目录下双击却能正常开启，这是个奇怪的问题，我目前还没找到原因和解决办法。
>
> 但是，在我的ubuntu 16.04上该方法是可行的。

### 6.3 一点clion的高效率小技巧

| 快捷键               | 功能                                                         | 效果图                                                       |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Ctrl+鼠标左键        | 查看声明/定义                                                |                                                              |
| Ctrl+Alt+7           | 查看引用(可以精确地知道某个变量/函数<br />在代码中的调用地方，对研究代码逻辑很有帮助) | <img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210827114057129.png" alt="image-20210827114057129" style="zoom:25%;" /> |
| Ctrl+Shift+F         | 全局搜索                                                     |                                                              |
| Ctrl+Tab             | 回到上一个文件                                               |                                                              |
| Ctrl+E               | 查看文件浏览历史                                             | <img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210827112600551.png" alt="image-20210827112600551" style="zoom:25%;" /> |
| Ctrl+Ctrl+E          | 查看代码行浏览历史                                           | <img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210827112703017.png" alt="image-20210827112703017" style="zoom:25%;" /> |
| Alt+Ctrl+左/右方向键 | 后退/前进到上一次浏览的代码行                                |                                                              |
| Ctrl+H               | 查看当前类的 继承关系                                        | <img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210827115648128.png" alt="image-20210827115648128" style="zoom:25%;" /> |
| Ctrl+Alt+L           | 对选中代码行的风格规范化                                     |                                                              |
| Ctrl+Alt+Home        | 源文件和头文件之间反复横跳                                   |                                                              |
| 双击Shift            | 弹出类、文件、变量符等的搜索窗                               | <img src="https://raw.githubusercontent.com/XiaoJake/img_personal/master/img/image-20210827154759480.png" alt="image-20210827154759480" style="zoom:25%;" /> |

- 关于 Ctrl+H：查看当前类的 继承关系。这个我特别讲一下它能解决的问题：

在carto中有基类的设计，**代码中许多指针,引用,函数参数都是以基类定义的**。这导致你**查看函数定义时只会跳转到对应基类的声明中**。基类只是接口声明，没有代码实现，这样你就**无法直接知道代码运行时实际调用的函数实现是什么**。

以 src/cartographer_ros/cartographer_ros/cartographer_ros/map_builder_bridge.cc 文件约118行为例:

>   map_builder_->LoadState(&stream, load_frozen_state);

1）Ctrl+鼠标左键点击 LoadState 函数会跳转到基类 map_builder_interfere 中的声明处

2）往上翻，鼠标左键点击map_builder_interfere类声明后，Ctrl+H

3）右侧会出现类的基础关系栏，可以看到它只被MapBuilder类继承，那么我们想查看的 LoadState 函数实现肯定就在这个类里！

4）我们点击MapBuilder类之后，Ctrl+F 搜索找到 LoadState ，再次Ctrl+鼠标左键查看定义，就看到了最终的代码实现了

以上方法主要是针对第一次看carto代码，还不熟悉它的代码、文件结构，逻辑关系时。后面熟悉了，可以直接找到相应文件去可能会快一点。



关于CLion的高效率技巧真的是很多，要讲完篇幅就太长了，我这里只讲了快捷键部分，这些都是我感觉比较实用的分享给大家。

剩下的大家可以自己去探索，我后面可能会单独出一篇讲CLion实用技巧的。

## 7.写在最后

​	这个方法也是最近才研究出来，刚好自己又需要搭建一个新的Ubuntu 18.04环境，于是觉得正好可以写一篇文章分享出来。自己平时也很少写公开博客之类的，对MarkDown还不是很熟悉。本以为应该写不了多久，第一版写出来觉得太简单，又补充补充，没想到这前前后后写了好几天 ... 

​	我是在本地typora编辑的，想放到知乎上还遇到了一些问题，如果图片、文字格式要重新整理的话，可能还得晚几天才能发了。(我其实是有写目录的，但知乎上好像还不支持这个功能。)

​	难以忍受这样的低效率，幸好找到了这位知乎大神@Adapting的方法：在vscode上安装 Zhihu On VSCode 插件。不仅解决了问题，还有提前预览的功能，在此表示感谢！

> ​	为什么知乎不支持 MarkDown ？ - Adapting的回答 - 知乎 https://www.zhihu.com/question/21560499/answer/2071395353



​	本着负责的态度，我是从搭环境一步一步记录过来的，步骤都有经过我的验证，所以**文中的代码块应该都是可以直接整体复制，放在终端里粘贴回车运行即可**（不用担心#号的注释内容，会被自动忽略），不需要一行一行的粘贴，这样大家会方便一点。如果还有其它问题，欢迎在评论里与我交流，我再进行更改。



​	对了，还有**“怎么在CLion里去 Debug单步调试 cartographer代码”**没有讲到，我就放在下次吧。

​	

​	希望这些内容能切实的给大家带来一点帮助！


