---
layout: post
title: FFmpeg 3.1.4 send/receive encoding and decoding API overview
date:   2017-04-17 15:10:00 +0800
categories: FFmpeg
---

see [3.1/group__lavc__encdec](https://ffmpeg.org/doxygen/3.1/group__lavc__encdec.html)

avcodec_send_packet()/avcodec_receive_frame()/avcodec_send_frame()/avcodec_receive_packet() 函数提供了一组 encode/decode API, 该api 分离了输入输出。

这组API对于encodeing/decoding以及audio/video非常相似，工作如下:
- 跟往常一样设置并打开AVCodecContext.
- 发送有效的输入:
        - 对于decoding，调用avcodec_send_packet() 来给decoder生的压缩的数据，该数据以AVPacket形式传送。
        - 对于encoding，调用avcodec_send_frame() encoder
        一个AVFrame 数据 他包含了未压缩的视频或者音频。在两种例子中，推荐 对 AVPackets以及AVFrames惊醒重新计数，否则libavcodec可能不得不复制输入数据。(libavformat 总是返回引用计数的AVPackets,并且av_frame_get_buffer() 分配引用计数的AVFrames.)
- 在循环中接受输出。周期性的调用某一个avcode_receive_*()函数并且处理该输出
        - 对于decoding，调用avcode_receive_frame(). 成功则它返回一个AVFrame 其包含了未压缩的音频或者视频数据。
        - 对于encoding，调用avcode_receive_packet().成功则它返回一个AVPacket 包含一个压缩的frame。重复调用这个函数，直到它返回AVERROR(EAGAIN) 或者一个error。AVERROR(EAGAIN)返回值表示新的输入数据被要求来产生一个新的输出。此种情况下，继续发送输入。对于每一个输入frame/packet，编解码器通常返回一个输出的frame/packet，但是它也可能是0，或者比1更大。


在编解码开始时，codec可能接受多个输入frame/packets而不会反悔一个frame，直到内部的buffers被填满。加入你遵循上述大纲，则次情境被透明的控制。

流的尾端时。这通常需要“flushing”(也被称为"draining")编解码器,犹豫编解码器内部可能由于性能或者不必要(例如B-frames)的原因缓存了多个frame或者packets,这种情况下 应该如下处理:
- 不在输入有效的input，发送NULL给 avcode_send_packet()或者
avcodec_send_frame() 函数。这会触发进入 draining mode.
- 调用avcodec_receive_frame() 或者avcodec_receive_packet() 在一个循环中 直到 AVERROR_EOF 返回.函数将不会返回 AVERROR(EAGAIN), 除非你忘记进入 draining mode.
- 在解码可以再次启动前，编解码器必须使用avcode_flush_buffers()重置。

按照上述大纲使用api是高度推荐的。但是不按照这个严格的模式调用api也是被允许的。例如： 你可以 重复的调用avcodec_send_packet()却不调用avcodec_receive_frame()。在这个例子中，avcodec_send_packet() 将会一直成功，直到 编解码器内部的缓存被填满为止(其通常在初始输入后大小是一倍的输出frame)，接着返回AVERROR(EAGAIN)来拒收输入. 一旦它开始拒绝输入，你将没有选择，只能至少读取一些数据.

不是所有的编解码器遵守一个严格的可预测的数据流；唯一可保证的是AVERROR(EAGAIN)返回值 在send/receive端调用产生，那么就暗示了，在另外一端的receive／send 调用一定能成功.通常来讲没有codec允许不受限制的缓存输入或者输出数据.

本API 用来替换老的函数:
- avcodec_decode_video2() 以及 avcodec_decode_audio4(): 使用avcodec_send_packet() 来投喂输入数据给decoder,接着使用avcodec_receive_frame()在每一个包后来接受解码后的frames。
不像旧的解码API，多个frame可能来自一个packet。对于audio,通过部分解码包来切开输入的包到多个frame对于用户来说变得透明了。你不在需要投喂一个AVpacket给api两次。此外，发送一个 flusing／draining 包仅仅被要求一次。
- avcodec_encode_video2()/avcodec_encode_audio2():使用avcode_send_frame() 来投喂输入给encoder，接着avcodec_receive_packet() 来接受编码的packets。为avcodec_receive_packet()提供用户分配的缓存不再可能.
- 新的api不在处理subtitles。

在同一个AVCodecContext 混合使用新旧api是不允许的，这将造成未定义的结果。

某些codecs 可能要求使用新的api，使用旧的api将在调用时返回错误。