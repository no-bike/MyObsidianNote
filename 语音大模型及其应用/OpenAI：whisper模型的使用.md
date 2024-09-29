		2022112479 曲本磊
### 环境配置

1. 使用==Anaconda==创建虚拟环境
```shell
conda create --name whisper_env python=3.9
conda activate whisper_env
```

2. 安装whisper（需要提前配好git）
```shell
pip install git+https://github.com/openai/whisper.git
```

3. 安装依赖FFmpeg
```shell
conda install -c conda-forge ffmpeg
```

如果有NVIDIA GPU可以使用cuda进行加速，如果没有可以略（有也可以）


### whisper使用

可以使用如下命令：
```shell
whisper "path/to/your/audiofile.mp3" --model small --output_format srt --output_dir "path/to/output"
```
- **–model small**：选择模型大小，模型越大，精度越高，但速度较慢（可选模型：`tiny`, `base`, `small`, `medium`, `large`）。
- **–output_format srt**：指定输出格式为 `.srt` 字幕文件。
- **–output_dir**：指定生成文件的保存路径。

### 使用结果

因为使用的是歌曲，吐字不是特别清晰。（也可能和使用的是small模型有关）
但是还是较为精准。

![[Pasted image 20240929211056.png]]

![[Pasted image 20240929211159.png]]