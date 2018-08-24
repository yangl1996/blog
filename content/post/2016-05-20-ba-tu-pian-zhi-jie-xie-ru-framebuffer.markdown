---
date: "2016-05-20T11:52:53Z"
title: 把图片直接写入Framebuffer
---

Linux frame buffer 是 Linux 对显示设备的一种抽象，它就像内存空间和显示设备每个像素的一一映射。修改这个设备文件的内容时，显卡会把这个修改反映到屏幕上，这样就修改了屏幕上的显示。

使用窗口管理器的现代 Linux 发行版上，直接与 frame buffer 进行 I/O 很可能会失败。所以往往在嵌入式设备上会比较有用，可以不依赖窗口管理，创建带显示功能的程序。事实上，在程序里把 frame buffer 的设备文件 mmap 一下，就可以很方便地控制显示。如果需要稍稍复杂的图形处理，还有 DirectFB 等库帮助。

frame buffer 用的是一种位图格式。要看具体是哪种位图，可以用 `fbset` 命令。比如在我的 Cubieboard 2 上，它返回的结果是：

```
mode "1280x720-50"
        # D: 74.250 MHz, H: 37.500 KHz, V: 50.000 Hz
        geometry 1280 720 1280 1440 32
        timings 13468 220 440 20 5 40 5
        accel false
        rgba 8/16,8/8,8/0,8/24
endmode
```

其中和这个位图格式相关的有 `rbga` 表示是 RGB 加 Alpha 的四通道位图，而 `geometry` 中数据表示这是 1280*720 大小，而且每个像素 32 位。

要简单的试试用 frame buffer 操纵显示，可以试试把 urandom 文件里的数据重定向到 frame buffer 设备，相当于往里输随机数据。

```
cat /dev/urandom > /dev/fb0
```

这样的效果应该是屏幕花了。

要显示有意义的图片，我们需要把图片转换成相应的位图格式，然后也直接重定向到 frame buffer 设备就行。这里可以用 ImageMagick 这个工具。它可以胜任这类转换。

```
convert -resize 1280x720 -depth 8 ./input.png bgra:./output.bitmap
```

其中 `resize` 选项会把图片改变大小，第二个 `depth` 选项设定了每个通道的深度（就是每个通道的位数，像这里就是 32/4 得到 8 位）。输出路径前有个 `bgra` 是用于确定输出位图的格式的。位图的三个颜色通道往往按 BGR 顺序存，再加上 Aplha 通道所以格式用 `bgra` 即可。

这样得出来的文件，再重定向到 frame buffer 就显示好了。

```
cat ./output.bitmap > /dev/fb0
```