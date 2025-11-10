
## 批量解压缩

下载7z，命令：

```powershell
# 压缩
7z a -t[format] archive_name file_name
# example
7z a -tzip archive_name.zip *.docx

# 解压
7z x -o[output_dir] archive_name
# example
7z x -osource_file archive.zip

# 查看
7z l test.zip
```

## 批量改名

```powershell
Get-ChildItem *.mp4 | Rename-Item -NewName { $_.name -replace '.*Vol\.([0-9]{1}).*', 'Video_$1.mp4' }


ls *.mp4 | foreach{ren $_.name ($_.name -replace '\(\d+\)', '')}
```

## 批量转码

使用ffmpeg将rmvb转换为mp4格式的视频。

```powershell
# 单个文件
ffmpeg.exe -i '.\Yes, Minister S01E01.rmvb'  -c:v libx264 -strict -2 '.\Yes, Minister S01E01.mp4'

# 多个文件
ls *.rmvb | foreach{ffmpeg -i $_.name -c:v libx264 -strict -2 ($_.name -replace '.rmvb', '.mp4')}
```

