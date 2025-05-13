# 1.whisper、whisper.cpp、faster-whisper的比较

Finger_ebic  于 2024-03-28 22:36:46 发布
原文链接：https://blog.csdn.net/KQe397773106/article/details/137126629

让我们比较一下当前的whisper、whisper.cpp和faster-whisper。

OpenAI / Whisper 自发布以来，似乎在各个方面都发生了变化，例如在 2022 年 12 月增加了 large-v2 模型和各种版本升级。

whisper.cpp是用 CPU 的 C/C++ 编写的。 它似乎是Core ML支持，所以它对于Mac用户有强烈的感觉。

而 faster-whisper是基于 CTranslate2 重写了 Whisper 模型，似乎它的速度比原版的快 4 倍，精度也相同，并且使用的内存更少。

以下是测试音频 （WAV）（48 秒）的测试示例。

## 1.1.结果

Faster-whisper 的 GPU 和传闻中的一样快。
Faster-whisper CPU运算速度更快，GPU的速度也从85.62秒→23.9秒加快。
whisper.cpp 效果反而不是很好。 可能真正的价值体现在支持Mac CoreML加速上。

Implementation | Device | Time | 结果
---------------|--------|------|---
openai/whisper | CPU | 3分1秒 | 上午和今天的东京股市，日经平均指数小幅走高，收于11.72日元，比昨天上涨22.72日元。 是088.58日元，最初价格上涨的股票数量为1146只，而下跌的股票数量为368只，保持不变的股票数量为104只股票，以下是礼物的公告： 在这个节目中，将抽签选出10人来接收每月出版的月报4月号，申请在东京通过电话03-0107-837303-0107-8373
openai/whisper | GPU | 23.9秒 | 上午和今天的东京股市，日经平均指数小幅走高，收于11.72日元，比昨天上涨22.72日元。 是088.58日元，最初价格上涨的股票数量为1146只，而下跌的股票数量为368只，保持不变的股票数量为104只股票，以下是礼物的公告： 在这个节目中，将抽签选出10人来接收每月出版的月报4月号，申请在东京通过电话03-0107-837303-0107-8373
whisper.cpp | CPU | 7分13秒 | 上午和今天的东京股市，日经平均指数小幅走高，收于11.72日元，比昨天上涨22.72日元。 是088.58日元，最初价格上涨的股票数量为1146只，但价格下跌了368只，104只股票保持不变，这是礼物公告该计划将通过抽签将月度报告4月号赠送给10人申请，请致电东京03-0107-8373,03-0107-8373
faster-whisper | CPU（Float32） | 4分 18秒 | 我是浅野智美 今天东京股市的日经平均指数略高 收盘价为11,088.58日元，比昨天上涨22.72日元 最初价格上涨的股票数量为1146只，而价格下跌的股票数量为368只股票保持不变 104只股票 以下是礼物的公告： 在本节目中，出版了月度报告4月号。 我们将通过抽签将其赠送给 10 人 申请，请致电东京 03-0107-8373 03-0107-8373 以上是该计划的公告
faster-whisper | CPU（int8） | 2分 27秒 | 我是浅野智美 今天东京股市的日经平均指数略高 收盘价为11,088.58日元，比昨天上涨22.72日元 最初价格上涨的股票数量为1146只，而价格下跌的股票数量为368只股票保持不变 104只股票 以下是礼物的公告： 在本节目中，出版了月度报告4月号。 我们将通过抽签将其赠送给 10 人 申请时，请致电东京0301078373 0301078373 以上是该计划的公告。
faster-whisper | GPU（Float16） | 5.22 秒 | 我是浅野智美 今天东京股市的日经平均指数略高 收盘价为11,088.58日元，比昨天上涨22.72日元 最初价格上涨的股票数量为1146只，而价格下跌的股票数量为368只股票保持不变 104只股票 以下是礼物的公告： 在本节目中，出版了月度报告4月号。 我们将通过抽签将其赠送给 10 人 申请，请致电东京 03-0107-8373 03-0107-8373 以上是该计划的公告
faster-whisper | GPU（int8_float16） | 4.73 秒 | 我是浅野智美 今天东京股市的日经平均指数略高 收盘价为11,088.58日元，比昨天上涨22.72日元 最初价格上涨的股票数量为1146只，而价格下跌的股票数量为368只股票保持不变 104只股票 以下是礼物的公告： 在本节目中，出版了月度报告4月号。 我们将通过抽签将其赠送给 10 人 申请时，请致电东京0301078373 0301078373 以上是该计划的公告。


## 1.2.Whisper
安装
```sh
!pip install -U openai-whisper
```

模型加载
```python
import whisper
model = whisper.load_model("large", device="cpu")
```

执行
```py
%%time
result = model.transcribe("sampleTokyo.WAV")
print(result["text"])
```

## 1.3.whisper.cpp
安装
```sh
!git clone https://github.com/ggerganov/whisper.cpp.git
%cd whisper.cpp
!bash ./models/download-ggml-model.sh large
!make
```

转换为 16kHz 的 WAV
```sh
!ffmpeg -i ../sampleTokyo.WAV -ar 16000 sampleTokyo.wav
```
 
执行
```sh
%%time
!./main -f ./sampleTokyo.wav -nt -l ja -m models/ggml-large.bin
```

## 1.4.faster-whisper
安装
```sh
!pip install faster-whisper
```

模型加载
```py
from faster_whisper import WhisperModel
model_size = "large-v2"

model = WhisperModel(model_size, device="cpu", compute_type="float32")
#model = WhisperModel(model_size, device="cpu", compute_type="int8")
#model = WhisperModel(model_size, device="cuda", compute_type="float16")
#model = WhisperModel(model_size, device="cuda", compute_type="int8_float16")
```

```python
%%time
segments, info = model.transcribe("sampleTokyo.WAV", beam_size=5, language='ja')
results = list(segments)
print(' '.join([result.text for result in results]))
```

Finger_ebic
                        
<--------------------------------------------------------------------->
<--------------------------------------------------------------------->
# 2.whisper-cli  如何把 wav 转成 中文文字

要使用 `whisper-cli` 把 WAV 音频文件转换为中文文字，可以按照以下步骤操作：

## 2.1. 安装 `whisper.cpp`
`whisper-cli` 是 `whisper.cpp` 项目中的一个命令行工具，你得先安装 `whisper.cpp`。按照下面的步骤克隆仓库并编译项目：

```bash
# 克隆 whisper.cpp 仓库
git clone https://github.com/ggerganov/whisper.cpp.git
cd whisper.cpp

# 编译项目
make
```

## 2.2. 下载中文识别模型
你需要下载适合中文识别的模型。以 `ggml-medium.bin` 模型为例，运行下面的命令下载：

```bash
bash ./models/download-ggml-model.sh medium
```

## 2.3. 使用 `whisper-cli` 进行中文语音识别
使用 `whisper-cli` 命令把 WAV 文件转换为中文文字。要确保指定正确的模型路径和语言参数。

```bash
./main -m ./models/ggml-medium.bin -l zh -f /path/to/your/audio.wav
```

在上述命令里：
- `-m`：用于指定模型文件的路径，这里是 `./models/ggml-medium.bin`。
- `-l`：用于指定识别的语言，`zh` 代表中文。
- `-f`：用于指定要识别的 WAV 音频文件的路径，你需要把 `/path/to/your/audio.wav` 替换成实际的文件路径。

### 2.3.1 whisper-cli currently runs only with 16-bit WAV

Note that the [whisper-cli](examples/cli) example currently **runs only with 16-bit WAV** files, so make sure to convert your input before running the tool.
For example, you can use `ffmpeg` like this:

```bash
ffmpeg -i input.mp3 -ar 16000 -ac 1 -c:a pcm_s16le output.wav
```

## 2.4. 完整示例
下面是一个完整的示例，涵盖了克隆仓库、编译项目、下载模型以及进行语音识别的步骤：

```bash
# 克隆 whisper.cpp 仓库
git clone https://github.com/ggerganov/whisper.cpp.git
cd whisper.cpp

# 编译项目
make

# 下载模型
bash ./models/download-ggml-model.sh medium

# 进行中文语音识别
./main -m ./models/ggml-medium.bin -l zh -f /path/to/your/audio.wav
```

按照以上步骤操作，你就能使用 `whisper-cli` 把 WAV 音频文件转换为中文文字。 

## 2.5 my示例
```sh
#!/bin/bash
work_dir=/home/abner/abner2/dev_78/j60hkd/ 

# 从mp4中抽取 音频信息，转成mp3
# ffmpeg -i input.mp4 -vn -acodec libmp3lame output.mp3
input_mp4_path=${work_dir}/hkd126c-0402.mp4 
ffmpeg -i ${input_mp4_path}   -vn -acodec libmp3lame ${work_dir}/output.mp3

# 从mp3 转成 wav
# ffmpeg -i input.mp3 -ar 16000 -ac 1 -c:a pcm_s16le output.wav
ffmpeg -i ${work_dir}/output.mp3 -ar 16000 -ac 1 -c:a pcm_s16le ${work_dir}/output.wav

# whisper.cpp把 音频 转成 文字
# ./build/bin/whisper-cli   -l zh  -m  models/ggml-medium.bin   -f ~/abner2/zdev/ai/av/video2note/output.wav
whisper_cpp_rootDir=/mnt/disk2/abner/zdev/ai/av/whisper.cpp
${whisper_cpp_rootDir}/build/bin/whisper-cli   -l zh  -m  ${whisper_cpp_rootDir}/models/ggml-medium.bin  -f  ${work_dir}/output.wav
```

<--------------------------------------------------------------------->
<--------------------------------------------------------------------->

