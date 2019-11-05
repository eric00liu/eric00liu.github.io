---
title: ffmpeg中时间戳调整参数setpts
date: 2018/8/15 15:07:24
tags: 
- ffmpeg
- 音画同步
- 时间戳
- pts
---
#####	PTS（Presentation Time Stamp）：即显示时间戳，这个时间戳用来告诉播放器该在什么时候显示这一帧的数据。
当视音频音画不同步，播放器进度条拖动异常，总时长显示异常，往往都是由于pts错误导致的。

## 先说如何查看视音频的pts
利用ffprobe命令查看每帧的数据, 以下命令会查看每一帧的相关参数，其中pkt_pts即该帧的pts

```
ffprobe -show_frames input.mp4    

```

## 如何设置pts
如果将每一帧的pts打印出来，你会发现这是个递增的数列（除了有B帧的情况），因为pts本身就是显示时间戳嘛，当然就是越靠后越大了。
那么修复pts，就是看如何重新构造这个递增的时间戳数列。

setpts这个参数本质上是ffmpeg中实现的功能，以filter的形式集成在ffmpeg的处理过程中，
ffmpeg会将本次命令执行过程中，涉及到的filter功能作为一个链，每一个视音频的帧，会首先由第一个filter开始处理，处理后的帧，传给下一个filter处理，等最后一个filter处理结束后，执行最后封装的过程。

那么在配置setpts这个参数时，我们主要是写出表达式，则当filter处理该帧，即根据表达式算出正确的pts，再写入到帧里。
表达式里可以用到的变量

```
FRAME_RATE，FR
帧速率，仅针对恒定帧速率视频定义

PTS
输入中的显示时间戳

N
视频输入帧的个数或消耗的采样数，不包括音频的当前帧，从0开始。

NB_CONSUMED_SAMPLES
消耗的样本数量，不包括当前帧（仅限音频）

NB_SAMPLES，S
当前帧中的样本数（仅限音频）

SAMPLE_RATE，SR
音频采样率

STARTPTS
第一帧的PTS

STARTT
第一帧的秒数

INTERLACED
说明当前帧是否是隔行扫描

T
当前帧的秒数

POS
帧的文件中的原始位置，如果当前帧未定义，则为未定义

PREV_INPTS
上一帧的输入PTS，也就是上一帧的原始PTS

PREV_INT
以秒为单位的上一帧原始时间戳

PREV_OUTPTS
上一帧的输出PTS,也就是由表达式算出来的上一帧的PTS

PREV_OUTT
以秒为单位的上一帧输出时间戳

RTCTIME
当前时间，以微秒为单位。这是不推荐使用的，而是使用time(0)。

RTCSTART
视频开始时间，以微秒为单位。

TB
输入时间戳的时基。

```

在文档中 http://ffmpeg.org/ffmpeg-all.html    搜索setpts可以找到相关介绍。
以下列出一个实例，以供参考：

```
ffmpeg -y -i input.ts -filter_complex "[0:1]asetpts='print(if(gt(PREV_INPTS, PTS), PREV_INPTS, if(gt(PREV_OUTPTS, PTS), PREV_OUTPTS, PTS)))',aresample=async=1000" -acodec libfdk_aac -vcodec copy output.ts  
```

看到这里的小伙伴，应该是真的需要用他来修复时间戳了，但这里有一个坑。    

### 坑         
用setpts转码过之后，再用ffprobe来看，发现转出来的视频pts跟你在setpts里算出来的不一样。         
是因为ffmpeg默认在filter链里，帮你加了另外一个会改动pts的filter，那就是aresample。
可以看到日志里记录下了加fliter这一步骤

```
[format_out_0_1 @ 0x2a89140]  auto-inserting filter 'auto_resampler_0' between the filter 'Parsed_asetpts_0' and the filter 'format_out_0_1'
```
所以为了降低aresample对我们pts修复的影响，我们需要设置一下aresample这个参数，上面的例子，我加入了 aresample=async=1000 便是这个作用。        
aresample怎么用，还是在文档里找吧 http://ffmpeg.org/ffmpeg-all.html   

