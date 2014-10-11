mjpg-streamer
======
fork：https://svn.code.sf.net/p/mjpg-streamer/code/ -r 182

How to build
---
[see more](https://github.com/meinside/rpi-mjpg-streamer)

	$ sudo apt-get install libv4l-dev libjpeg8-dev subversion imagemagick v4l-utils

	$ svn co https://svn.code.sf.net/p/mjpg-streamer/code/mjpg-streamer/ mjpg-streamer

	$ cd mjpg-streamer

	$ sudo ln -s /usr/include/linux/videodev2.h /usr/include/linux/videodev.h

	$ make USE_LIBV4L2=true clean all

源码理解
---

###### mjpg-streamer/ 主源码在目录：
	mjpg_streamer.c: main程序入口，并负债加载 N个 input/output 插件。
		- 源码定义N为最大各10个插件。
		- 插件编译为so文件，使用-i表示加载一个input插件，-o为output插件；
		- 如:-i “./input_uvc.so -d /dev/video0”,表示使用uvc插件,“-d /dev/video0”由uvc1解释。
		- mjpg_streamer [-i|-o] “./plugin-name.so -help”，都能输出插件的命令行帮助文档。

	plugins/ 目录: 各种插件源码，每个插件一个单独子目录。
		- input.h, output.h 中定义了输入/输出插件结构件。
		- input_file, output_file 为最基础插件-- 即代码，功能最少的插件。
		- 插件定义如下函数:input_init,input_stop,input_run,input_cmd;(output定义由output开头)。
		- int input_init()函数返回0表示成功，返回1表示退出，一般为call help()后返回。
		- 每个插件自己定义help函数与解释各自需要的命名行参数,init时传入参数。

	www/ 目录：静态文件(html,css,js,img),应该output_http使用，-w参数指此目录，提供http服务。

	scripts/ 目录：作者用来放shell脚本的，只有一个脚本。

	TODO文件：从中看到UDP/RTP插件并未完美
		再个插件都能运行起来，但我也未成功看到图像，可能参数不对，也没见到例子，再研究。
		并且，默认是不编译出来这两个插件的so文件的。

###### mjpeg-client/ 目录：
	不懂，不知用什么语言写的，bin/里面有exe文件，能在windows下运行。

###### mjpg-streamer-experimental/ 目录：
	顾名思义，就是实验性的功能开发代码，与主目录结构一致。

###### uvc-streamer/ 目录：
	貌似不是同一个作者写的，完全是另一个webcam，看里面的README更详细。
	CHANGELOG中提到了3个开发者参与过代码贡献。
	进入此目录，直接执行make即可编译出./uvc_stremer
	默认视频输出URL是：http://127.0.0.1:8080/, /snapshot 可以获得单帧图片。
	还有一个控制URL是：http://127.0.0.1:8081/?control --- PS测试中没能成功控制video
	uvc_stremer可以认为是mjpg-streamer的精简版，重点在节省CPU。作者希望其与motion结合使用。

###### udp_client/ 目录：
	C++的QT写的一个udp界面客户端，还没搞懂用法。

### mjpg_streamer.c 中的加载插件过程：
- 首先所有插件必需统一定义相同名称的4个函数，并编译为标准so文件。
- globals为全局变量，有input/ouput plugin两个数组与一个 stop 标识变量；插件与访问此globals。
- main先解释命令行参数-i|-o, 然后以路径加载so为输入或输出插件。
- 加载过程，先调用[dlopen函数](http://baike.baidu.com/view/2907309.htm)，保存到global.in[i].handle。
- 然后通过[dlsym函数](http://baike.baidu.com/view/1093952.htm)依次加载插件4个标准函数。
- dlsym后，调用插件的init函数初始化插件。
- 通过两个for循环依次调用input与output插件的 run函数，最后进入pause()。
- 每个插件的run函数各自用了pthread_create修建新线程；通过判断globals.stop中止线程函数。
- 插件的stop函数由main中注册的signal_handler调用，各自调用pthread_cancel释放资源。
- signal_handler函数中，global.stop 置成 1，并调用[usleep](http://baike.baidu.com/view/2785782.htm)后再调用已加载插件的stop函数。

### 视频流量问题：
	使用./uvc_stream -r 320x240 -f 3; 320x240分辨率+每秒3帧，流量到60+KB/s,家用宽带能撑？
	视频是没声音的，作为TODO项。
	