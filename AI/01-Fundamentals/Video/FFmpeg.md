FFmpeg 是一个功能极其强大、完整的**用于处理视频、音频和其他多媒体文件与流**的跨平台开源框架，能够对视频和音频进行录制、转换、编辑、流传输等几乎任何能想到的操作。

## 核心组成

1. **`ffmpeg`**：命令行工具，这是最常用的部分，用户通过输入命令来转换多媒体文件格式，或进行各种处理。
2. **`ffplay`**：简单的媒体播放器，它是一个基于 FFmpeg 库的轻量级播放器，常用于快速测试和调试媒体文件。
3. **`ffprobe`**：媒体文件分析器，它可以显示媒体文件的详细信息，如编码格式、码率、分辨率、时长、元数据（metadata）等。
4. **核心代码库**：包含了一系列用于音视频编码/解码、转换、混合、流传输的库（如 `libavcodec`, `libavformat`, `libavutil` 等）。这些库是 FFmpeg 强大功能的基石，并被许多其他软件（如 VLC、Blender、YouTube、iTunes）在背后使用。



## 应用场景

1. **格式转换**：最基础的功能，将一个视频文件（如 MKV）转换为另一种格式（如 MP4），或将音频文件（如 WAV）转换为 MP3。
```
ffmpeg -i input.mp4 output.avi
```

2. **视频剪辑**：截取视频中的一段。
```
ffmpeg -i input.mp4 -ss 00:01:00 -t 00:00:30 -c copy output.mp4   # (从1分钟开始，截取30秒)       
```

3. **视频合并**：将多个视频文件拼接在一起。

4. **提取音频**：从视频中分离出音轨。
```
ffmpeg -i input.mp4 -vn -acodec copy output.aac
```

5. **添加/替换音轨**：为视频添加新的背景音乐或配音。

6. **压缩视频/音频**：通过调整码率、分辨率等参数，减小文件大小。
```
ffmpeg -i input.mp4 -vcodec libx264 -crf 23 output_compressed.mp4
```

7. **屏幕录制**：录制电脑屏幕和系统声音。

8. **截图**：从视频的特定时间点提取一帧作为图片。
```
ffmpeg -i input.mp4 -ss 00:00:05 -vframes 1 screenshot.jpg
```

9. **改变分辨率/帧率**：例如，将 4K 视频转换为 1080p。
```
ffmpeg -i input.mp4 -vf "scale=1920:1080" output_1080p.mp4
```

10. **添加水印**：在视频上叠加图片或文字水印。

11. **直播推流/拉流**：将本地媒体文件或设备推送到直播服务器（如 RTMP），或者从直播源拉取流。
```
ffmpeg -re -i input.mp4 -c copy -f flv rtmp://live.twitch.tv/app/your-stream-key
```

 12. **硬件加速**：利用 GPU（如 NVIDIA NVENC）进行编码，大幅提升处理速度。