# 基于Yolov5实现的Bilibili蒙版弹幕

前言：该项目没啥技术含量，只适用于yolov5的入门了解，通过该项目可以初步了解yolov5代码的一些特性与使用方法，从而能更好的继续往下学习。

------

一：准备工作

1. B站视频以及弹幕的下载：

   通过浏览器（我用的是Chrome）安装Tampermonkey(油猴)插件。然后在油猴插件里搜索"B站视频下载"脚本，安装后打开B站，即可下载视频，以及相关弹幕。具体安装方法，网上搜一搜就能搜到，在这里就不展开具体介绍了。

2. Yolov5的安装及环境配置

   Yolov5的下载链接：https://github.com/ultralytics/yolov5

   环境配置：

   ​	下载好yolov5后里面有个requirements.txt文件，在Terminal中输入

   ```python
   pip install -r requirements.txt
   ```

   即可完成大部分安装，但是不出意外的话，会有两个包安装失败。一个pillow,一个

   ```python
   ERROR: tensorflow 2.0.0 has requirement tensorboard<2.1.0,>=2.0.0, but you'll have tensorboard 2.4.1 which is incompatible.
   ERROR: matplotlib 3.3.4 has requirement pillow>=6.2.0, but you'll have pillow 6.1.0 which is incompatible.
   ```

   pillow只需要卸载，再重新安装即成为最新版本了，另外一个这个项目不需要，我就忽略了（偷偷懒！）

   ```python
   pip uninstall pillow #卸载命令
   pip install pillow #安装命令
   ```

------

二：detect.py介绍

这个文件是yolov5自带的，可以实行80种物体的识别与标注。具体的一些功能，可以去网上直接搜"YOLOV5检测代码detect.py注释与解析",从而可以快速了解里面的类和函数的用法。

------

三：detect2.py介绍

这个文件是在detect.py的基础上改了一些参数，可以对照两个文件仔细对比一下，就能发现。现在这个文件可以实现，只对"人"进行检测，画出图中"人"的Box,并且可以对获取的"人"的坐标保存成一个txt文件。这里要着重说明一下这个txt文件，里面都是小于1的数，和下面获取的目标坐标不一样，这是因为，在保存之前，源代码，做了一些处理（貌似是归一化，我猜的，应该和归一化的操作类似），如果你想直接调用txt文件的坐标，需要看看原代码，从而还原回去。

------

四：detect3.py介绍

这个文件又是在detect2.py的基础上进行了修改，这个文件实现了蒙版弹幕功能。具体如何实现请看代码。

------

五：Voice.py

视频声音处理

相关依赖库的安装：

```python
pip install moviepy
```

这个东西难搞哦，只有放在把两个要处理的文件放在当前目录下，直接传入文件名，能保证成功。传入路径之后，就会出现一些莫名其妙的问题。如果有读者解决了，希望可以私信告诉我，非常感谢。

------

六：其他

1. 检测图片和检测视频是一样的，只需要把文件放入images文件夹里就行了，蒙版弹幕功能，新创建了一个文件夹images2，这个文件夹是用来方带弹幕的视频或者图片的，因为images文件夹是用来抠图的。
2. Opencv处理过的视频是没有声音的，所以需要对视频进行另外的处理。
3. Yolov5的模型参数，已经下好了yolov5s.pt，如果需要其他的模型参数，请读者自行去网上下载。注意！调用其他模型，需要仔细阅读detect.py里面的东西。
4. 处理视频和处理图片是一样的，不需要更改什么，视频就是很多图片连在一起。

------

七：实现的思路

1. 用Yolov5进行检测，获取检测目标的坐标

   ```python
   (tx,ty,bx,by) = (int(xyxy[0].item()),int(xyxy[1].item()),int(xyxy[2].item()),int(xyxy[3].item()))
   ```

   这里面的坑，我替你们趟了。这里面包含了pytorch的一些细节问题，搞完又有类型问题，很是麻烦。

2. 把目标区域裁剪下来

   ```python
   img_dm[ty:by,tx:bx]  = im0[ty:by,tx:bx]  # 裁剪图像
   ```

   这里又有很多细节问题，要好好搞清楚这个detect.py的代码逻辑。im0保存的就是加载进来的图片。这里还有个小细节，一定要，y在前，x放后面，否则裁剪的就会不准确。t:top,b:bottom，这个不难理解。

3. 把带有弹幕的图片或者视频加载进来

   这里需要注意的是，因为这个项目是基于pytorch的，所以加载数据的方法是用的pytorch的加载方法（具体如何使用，感兴趣可以深入了解一下）。如果你看到dataset放在for后面，这个for的次数是你加载的文件夹里的文件个数，注意是文件个数。

4. 把弹幕文件的相应位置的坐标参数换成刚裁剪下来的坐标参数即可。

5. 视频需要声音的话，再进行声音处理即可。

6. 大致逻辑就是这样，还有很多细节问题，你们可以自己捋一遍，思考思考，也欢迎私信和我讨论。

------

八：效果图

 <img src="https://github.com/Learninglogs/yolov5/blob/master/%E5%9B%BE%E5%BA%93/yolov5%E5%A4%84%E7%90%86%E7%89%88%2000_00_00-00_00_30.gif" alt="效果图片" width="200" height="300" align="bottom" />

九：自我说明

1. 我很菜，对于这个专业，只能算是入门级别。我们可以一起交流，一起学习，一起进步，一起变得更强。
2. 我的微信：wx1277590830（数字部分是qq,加我请备注一下来意，谢谢！）

