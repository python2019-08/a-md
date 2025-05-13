# 1.python edge-tts库实现文字转语音
菲宇  编辑于 2023-11-29 16:13・IP 属地广东
https://zhuanlan.zhihu.com/p/669473903

## 1.1关于edge-tts库

Edge-TTS是一个Python库，允许您从 Python 代码中使用 Microsoft Edge 的在线文本转语音服务或使用提供的 or 命令。它使用微软的Azure Cognitive Services来实现文本到语音转换（TTS）。该库提供了一个简单的API，可以将文本转换为语音，并且支持多种语言和声音，输出的音频文件更接近正常人的发音。

## 1.2安装 edge-tts库
```sh
pip install edge-tts
```

## 1.3查看edge-tts的命令

> usage: edge-tts [-h] [-t TEXT] [-f FILE] [-v VOICE] [-l]
>                 [--rate RATE] [--volume VOLUME] [--pitch PITCH]
>                 [--words-in-cue WORDS_IN_CUE] [--write-media WRITE_MEDIA] 
>                 [--write-subtitles WRITE_SUBTITLES][--proxy PROXY]
> edge-tts: error: one of the arguments -t/--text 
>                 -f/--file -l/--list-voices is required

参数说明
> --text 【参数】 指明要保存的mp3的文本。
> --write-media 【参数】 指明保存的mp3文件路径。
> --voice 【参数】指明了使用哪种语音和风格的发音人，例如zh-CN-YunjianOnlineNatural。系统中可以通过edge-tts --list-voices 查看更多的语音和风格。
> --rate 【参数】调整语速，例如-50%，慢速了50%。
> --volume 【参数】 调整音量 例如+10% ，声音提高了10%。
> -f 【参数】指明要转换语音的文本文件，例如一个txt文件。

## 1.4语音列表
查看edge-tts支持的语音列表命令：edge-tts --list-voices，使用edge-tts --voice 语音Name即可以使用。

以下是支持中文的语音：

> Name: zh-CN-XiaoxiaoNeural
> Gender: Female
> 
> Name: zh-CN-XiaoyiNeural
> Gender: Female
> 
> Name: zh-CN-YunjianNeural
> Gender: Male
> 
> Name: zh-CN-YunxiNeural
> Gender: Male
> 
> Name: zh-CN-YunxiaNeural
> Gender: Male
> 
> Name: zh-CN-YunyangNeural
> Gender: Male
> 
> Name: zh-CN-liaoning-XiaobeiNeural
> Gender: Female
> 
> Name: zh-CN-shaanxi-XiaoniNeural
> Gender: Female
> 
> Name: zh-HK-HiuGaaiNeural
> Gender: Female
> 
> Name: zh-HK-HiuMaanNeural
> Gender: Female

## 1.5 常用命令实例

```
#使用命令行Edge-TTS来产生语音
edge-tts --text "Hello, world!" --write-media hello.mp3

#edge-tts查看支持的语音。
edge-tts --list-voices

#添加--voice命令，指定输出的语音。
edge-tts --voice zh-CN-YunxiNeural --text "hello 大家好" --write-media hello.mp3

#添加rate与volume指令调整语速与音量
edge-tts --voice zh-CN-YunxiNeural --rate=-4% --text "hello 大家好" --write-media hello1.mp3
edge-tts --voice zh-CN-YunxiNeural --volume=-4% --text "hello 大家好" --write-media hello1.mp3
rate=-4% 意思是语速降低4%，volume=-4%意思是音量降低4%，当然若是加号，便是增加多少。
```

## 1.6使用示例

```
edge-tts --voice zh-CN-YunxiNeural --rate=-4% --text "hello 大家好,我是Python与
Django学习的小编，欢迎大家阅读我的文章，如果喜欢请点赞收藏转发！谢谢您！" --write-media hello1.mp3
#输出结果
WEBVTT

00:00:00.102 --> 00:00:03.592
hello 大家 好 我 是 Python 与 Django 学习 的

00:00:03.605 --> 00:00:07.563
小编 欢迎 大 家阅读 我 的 文章 如果 喜欢 请

00:00:07.563 --> 00:00:10.039
点赞 收藏 转发 谢谢 您

```
输出的mp3,大家请试听

## 1.7批量实现文本转换为语音

通过打开本地 demo.txt 文本，将其传递传递给edge-tts进行语音转化。通过decode函数转换一下，是为了字符类型报错。
```python
import edge_tts
import asyncio
TEXT = ""
with open ('demo.txt','rb') as f:
    data = f.read()
    TEXT = data.decode('utf-8')
print(TEXT)
voice = 'zh-CN-YunxiNeural'
output = 'demo.mp3'
rate = '-4%'
volume = '+0%'
async def my_function():
    tts = edge_tts.Communicate(text = TEXT,voice = voice,rate = rate,volume=volume)
    await tts.save(output)
if __name__ == '__main__':
    asyncio.run(my_function())
```    
使用async进行异步请求，若是直接运行，会提示如下错误

RuntimeWarning: coroutine 'Communicate.save' was never awaited
  tts.save(output)
RuntimeWarning: Enable tracemalloc to get the object allocation traceback
将本文第一段转换成语音。

项目地址：https://github.com/rany2/edge-tts
 
