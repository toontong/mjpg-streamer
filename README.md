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