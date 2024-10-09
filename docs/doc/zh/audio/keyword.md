---
title: MaixCAM MaixPy 关键词识别
update:
  - date: 2024-10-08
    author: 916BGAI
    version: 1.0.0
    content: 初版文档
---

## 简介

`MaixCAM` 移植了 `Maix-Speech` 离线语音库，实现了连续中文数字识别、关键词识别以及大词汇量语音识别功能。支持 `PCM` 和 `WAV` 格式的音频识别，且可通过板载麦克风进行输入识别。

## Maix-Speech

[`Maix-Speech`](https://github.com/sipeed/Maix-Speech) 是专为嵌入式环境设计的离线语音库，其针对语音识别算法进行了深度优化，在内存占用上达到了数量级上的领先，并且保持了优良的WER。如果想了解原理可查看该开源项目。

## 关键词识别

```python
from maix import app, nn

speech = nn.Speech("/root/models/am_3332_192_int8.mud")
speech.init(nn.SpeechDevice.DEVICE_MIC, "hw:0,0")

kw_tbl = ['xiao3 ai4 tong2 xue2',
          'ni3 hao3',
          'tian1 qi4 zen3 me yang4']
kw_gate = [0.1, 0.1, 0.1]

def callback(data:list[float], len: int):
    for i in range(len):
        print(f"\tkw{i}: {data[i]:.3f};", end=' ')
    print("\n")

speech.kws(kw_tbl, kw_gate, callback, True)

while not app.need_exit():
    frames = speech.run(1)
    if frames < 1:
        print("run out\n")
        speech.deinit()
        break
```

### 使用方法

1. 导入 `app` 和 `nn` 模块

```python
from maix import app, nn
```

2. 加载声学模型

```python
speech = nn.Speech("/root/models/am_3332_192_int8.mud")
```

- 也可以加载 `am_7332` 声学模型，模型越大精度越高但是消耗的资源也越大

3. 选择对应的音频设备

```python
speech.init(nn.SpeechDevice.DEVICE_MIC, "hw:0,0")
```

- 这里使用的是板载的麦克风，也选择 `WAV` 和 `PCM` 音频作为输入设备

```python
speech.init(nn.SpeechDevice.DEVICE_WAV, "path/audio.wav")   # 使用 WAV 音频输入
```

```python
speech.init(nn.SpeechDevice.DEVICE_PCM, "path/audio.pcm")   # 使用 PCM 音频输入
```

- 注意 `WAV` 需要是 `16KHz` 采样，`S16_LE` 存储格式，可以使用 `arecord` 工具转换

```shell
arecord -d 5 -r 16000 -c 1 -f S16_LE audio.wav
```

- 在 `PCM/WAV` 识别时，如果想要重新设置数据源，例如进行下一个WAV文件的识别可以使用 `speech.devive` 方法，内部会自动进行缓存清除操作：


```python
speech.devive(nn.SpeechDevice.DEVICE_WAV, "path/next.wav")
```

4. 设置解码器

```python
kw_tbl = ['xiao3 ai4 tong2 xue2',
          'ni3 hao3',
          'tian1 qi4 zen3 me yang4']
kw_gate = [0.1, 0.1, 0.1]

def callback(data:list[float], len: int):
    for i in range(len):
        print(f"\tkw{i}: {data[i]:.3f};", end=' ')
    print("\n")

speech.kws(kw_tbl, kw_gate, callback, True)
```
- 用户可以注册若干个解码器（也可以不注册），解码器的作用是解码声学模型的结果，并执行对应的用户回调。这里注册了一个 `kws` 解码器用于输出最近一帧所有注册的关键词的概率列表，用户可以观察概率值，自行设定阈值进行唤醒。对于其他解码器的使用可以查看语音实时识别和连续中文数字识别部分

- 设置 `kws` 解码器时需要设置 `关键词列表`，以拼音间隔空格填写，`关键词概率门限表`，按顺序排列输入即可，是否进行 `自动近音处理`，设置为 `True` 则会自动将不同声调的拼音作为近音词来合计概率。最后还要设置一个回调函数用于处理解码出的数据。

- 用户还可以使用 `speech.similar` 方法手工注册近音词，每个拼音可以注册最多 `10` 个近音词。（注意，使用该接口注册近音词会覆盖使能 `自动近音处理` 里自动生成的近音表）

```python
similar_char = ['zhen3', 'zheng3']
speech.similar('zen3', similar_char)
```

- 在注册完解码器后需要使用 `speech.deinit()` 方法清除初始化

5. 识别

```python
while not app.need_exit():
    frames = speech.run(1)
    if frames < 1:
        print("run out\n")
        speech.deinit()
        break
```

- 使用 `speech.run` 方法运行语音识别，传入的参数为每次运行的帧数，返回实际运行的帧数。用户可以选择每次运行1帧后进行其他处理，或在一个线程中持续运行，使用外部线程进行停止。

### 识别结果

如果上述程序运行正常，对板载麦克风说话，会得到关键词识别结果，如：

```shell
kws log 2.048s, len 24
decoder_kws_init get 3 kws
  00, xiao3 ai4 tong2 xue2
  01, ni3 hao3
  02, tian1 qi4 zen3 me yang4
find shared memory(491520),  saved:491520
    kw0: 0.959; 	kw1: 0.000; 	kw2: 0.000;     # 小爱同学
    kw0: 0.000; 	kw1: 0.930; 	kw2: 0.000;     # 你好
    kw0: 0.000; 	kw1: 0.000; 	kw2: 0.961;     # 天气怎么样
```