
## minesweeper扫雷程序
## 开发环境

1. 开发工具：Qt5.14.0

2. 开发语言：C++

## 运行
#### macOS
macOS可用Qt直接编译运行
#### Windows
- 删除sweep_minesV1.pro文件中的ICON = minesweeper.icns
- 将mainwindow.h中私有成员int distance = 12改为int distance  = 35
  
  > (此处因为macOS与Windows的主窗口上部占用像素不一致)
#### Linux
  > distance值待测试

## 操作说明
- 菜单中可选择设置难度(easy || medium || hard || customize)
- 菜单栏game-->new game开始新游戏
- 左键翻开 & 右键标雷/标疑问/取消标记
- 左键&右键同时按下----打开标记完成位置周围其他方块

## 作弊说明
扫雷游戏最终难免遇到无解的情况，此时只需要按下鼠标中键，则选中的位置如果是雷则标记为红旗，如果非雷则直接打开

## 文件含义简介
* **cellitem.h** 和 **cellitem.cpp**  类文件：每个格子元素item，包含状态等
* **configdialog.h** 和 **configdialog.cpp** 类文件：游戏配置操作
* **fielddata.h** 和 **fielddata.cpp**  类文件：底层数据
* **main.cpp** 程序入口
* **mainwindow.h** 和 **mainwindow.cpp** 类文件：主窗口操作（统一调用及设置数据，维护视图，设置场景等）
* **mainwindow.ui** 图形文件：主窗口图元描述
* **minesweepscene.h** 和 **minesweepscene.cpp**  类文件：场景布置
* **res.qrc** 资源文件：描述加载的文件（本项目是一些图片）
* **minesweeper.pro** qt项目文件，包含一些项目相关设置
* **imgs.pptx** 游戏中用到的图片原件，去除透明背景k可以利用ps处理


## 技术要点

### 1.数据支撑

* 由于数据在全局只有唯一一份，故设计为单例模式。默认布局也在这里设置（宽20，高15，雷数50）。
* 关于布雷：随机算法，注意判断边界，如果确定此位置布雷（我用-1表示），其周围8个方向的数据均加1；
* 关于游戏设置：
   - 游戏支持自己配置宽度和高度以及地雷数目，这里用到了两套数据，一个为配置数据域，一个为基本数据域，当我们设置成功后，需要转场，这时先删除旧的场景（这时用的是基本数据域），再创建新的场景（这时把配置数据传给基本数据域，就把基本数据域更新了），再把视图设置为当前场景即可；这里的数据传递很巧妙，注意体会；
   - 数据有效性检查利用QIntValidator类自动检查，省去不少功夫（我当前限制宽为1~30，高为1~19，雷数不能超过“宽*高”的一半）；

### 2.图形框架及渲染

* QGraphicsPixmapItem负责生成每个格子元素item;
* 利用格子元素生成场景Scene；
* 将Scene装进QGraphicsView视图中；
* 注意坐标处理：
   - 视图和场景的坐标由框架自动处理，保证整体视图位于中心位置；
   - 鼠标点击寻找对应的格子元素，利用itemAt函数（注意此函数参数为int，因此无法做到太高精度，这也是整体画面限制大小的一个原因，另外一个原因是视觉的美观性）；

### 3.自动寻路扩展

* 如果点击的格子周围没有地雷，需要自动扩展，需要考虑两方面因素：一个是底层数据的扩展，一个是视图的扩展；
* 两者均利用队列的思想实现，探寻周围8个方向的格子元素（底层数据）是否周围没有地雷，如果是的话就将其加入队列，不是的话翻开；需要加上标记表示其是否被翻开过，如果已经被访问过就不需要再加进队列了；
* 根据基本数据的寻找方向，同步扩展指针坐标（相当于自动鼠标点击），找到对应的格子元素，再根据底层数据做相应操作；我当初想直接基于当前格子元素获取其周围的格子元素，但是没有找到方法，然后就模拟鼠标自动扩展了，欢迎指教；
* 注意这里可能会出现画面缩放（虽然我最后限制了画面扩展减轻了负担），缩放时需要同步考虑鼠标指针的变换，必须是高精度（int不行，可以用double，但是经过测试雷数太多（大概50*30）的时候double也不够用，因为上层的itemAt函数参数为int）；

### 4.各种标记的处理

* 我们知道，游戏中有旗子，问号等标记，因此需要单独处理；自动寻路扩展时不会考虑旗子，但是会考虑问号，只要查询其状态再对应处理即可；

### 5.判赢和判输

* 输比较好判断，只要点击到地雷，直接判输；
* 判赢方式：用两个变量分别表示目前还没有被翻开的格子和目前已经标记的地雷数目，如果某时刻两个变量相等，则判赢；注意每次翻格子的时候检查一次即可（不用开多线程进行全局判断）；


## 与传统wndows扫雷程序的对比

* 没有添加应用程序图标，有兴趣可以自己添加；
* 没有加入步骤计数、时间、音乐，有兴趣可以自己添加；
* 踩到雷即结束游戏，没有加入地雷连锁爆炸动画；
* 关于随机算法与布局：
   - windows系统自带的程序：随机算法偏弱，但是布局算法更胜一筹，各种数字出现几率更大；
   - 自己开发的程序：随机算法偏强，因此布局算法偏弱，有些大的数字出现几率相对小一点。


## 晒几张运行截图
![image](https://github.com/Skr-Skr/minesweeper/blob/master/result/screen1.png)
![image](https://github.com/Skr-Skr/minesweeper/blob/master/result/screen2.png)