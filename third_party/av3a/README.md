# av3a_dec.dll

AVS3-P3（Audio Vivid，MP4 中 `av3a` 音轨）解码 DLL，供 MusicPlayer2 使用。

- DLL 外壳源码：`third_party/av3a/src/av3a_dec.c`（MP4 抽帧 / 下混 / WAV 与缓存接口）
- 解码核心：`third_party/av3a/src/codec`（AVS3A_RMV4.0 参考解码器）+ `model.bin`
- 播放器接入：`MusicPlayer2/BassCore.cpp` 中的 `TryOpenAv3a()`
- 构建：全部由 `MusicPlayer2.sln` 一次编译（`av3a_dec.vcxproj` 生成 DLL，
  输出到 `x64\<Configuration>`，`model.bin` 由生成后事件自动复制）

## 使用

构建后 `av3a_dec.dll` 会出现在 `MusicPlayer2.exe` 同目录；模型需自行放置为
`model.bin` 或 `vivid_model.bin`。

- 模型不在仓库中，获取方式见下文「模型」
- 流式播放：首次打开约 1 秒内出声，后台继续解码（整曲约 20 秒），
  完成后写入 `%LOCALAPPDATA%\av3a_cache`，之后打开接近秒开
- 首次播放时若在解码完成前直接拖到很靠后的位置，会短暂静音直到解码追上
- 原生流为 7.1.4 12 声道，DLL 内部下混为立体声输出

## 模型（需自备，不随仓库分发）

DLL 会在自身目录依次查找 `model.bin`、`vivid_model.bin`，任选其一放入即可。
获取方式二选一：

1. AudioVivid 参考包（推荐）
   - UWA 下载页：https://www.theuwa.com/download/2?page=2
   - 或镜像包：https://github.com/6hzzz/AVS3stream 的 `TS/AudioVivid源码.7z`
   - 解压后位于 `AVS3A_Codec_linux/avs3RM0Decoder/model.bin`
     （或 `AVS3A_RMV4.0/AVS3A_RMV4.0/AVS3_Codec/avs3RM0Decoder/model.bin`）
2. 已安装的网易云客户端
   - `<安装目录>\resource\vivid_model.bin`
     （常见：`C:\Program Files (x86)\NetEase\CloudMusic\resource`、`E:\CloudMusic\resource`）
   - 可直接使用原名，或改名为 `model.bin`

该模型仅供本地使用，请勿再分发。

## 导出接口

```c
int  av3a_probe(const wchar_t* path);                    // 1 = av3a m4a
const wchar_t* av3a_prepare(const wchar_t* path);        // 返回可播放 WAV 路径
void av3a_cleanup(void);                                 // 释放临时文件
int  av3a_decode_wav(const wchar_t* path, void** data, unsigned* size);
void av3a_free(void* data);
```

## 来源与许可

- **解码核心**：`AVS3A_RMV4.0` 参考解码器，由 AVS / UWA（Audio Vivid）公开发布的
  参考代码，并非本项目原创，也不是从网易云客户端提取。
- **外壳与接入**：`av3a_dec.c`（MP4 `av3a` 帧抽取、12 声道下混、WAV/缓存/流式接口）
  以及 `MusicPlayer2/BassCore.cpp` 中的 `TryOpenAv3a()` 为本项目代码。
- **model.bin**：同一参考代码包自带的标准神经网络参考模型，与网易云
  `resource/vivid_model.bin` 哈希一致。

参考代码**不是 OSI 开源许可**，其头部许可原文要点为：

> Redistribution and use in source and binary forms, with or without modification,
> are permitted only for the purpose of developing standards within AVS and for
> testing and promoting such standards.

即：允许为 AVS 标准开发、测试与推广而使用和再分发，需保留版权声明与免责声明；
不得暗示原作者为衍生品背书。商用、专利授权与内容分发需另行取得授权。

本 DLL 仅供本地研究与格式兼容性测试，不附带任何担保，也不随附音频内容。
