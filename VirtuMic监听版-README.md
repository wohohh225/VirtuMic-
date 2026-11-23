# VirtuMic 监听版

基于官方 [VirtuMic](https://gitlab.com/IndiDevs/VirtuMic)（Android 虚拟麦克风，MIT 许可）的修改版。

原版可以把手机里某个 App 的声音（或音频文件）注入成「麦克风」，让 QQ、游戏、语音房里的队友听到你在放什么；但声音被注入后就不再从扬声器出来了，你自己听不到。**这个版本加了「监听」：给队友放的同时，本机扬声器也放一路，你自己也能听见。**

## 和原版的区别

| | 官方版 | 本版 |
| --- | --- | --- |
| App 采集时本机扬声器 | 静音 | 跟随播放（监听） |
| 无人占用虚拟麦克风时 | 采集管线阻塞 | 非阻塞写入，不阻塞 |
| 签名 | 官方签名 | 自签名（需先卸载官方版） |

## 新增功能

**扬声器监听**
- 用「App」模式采集其他 App 的声音时，同一路 PCM 会同时写到一个 `STREAM_MUSIC` 音轨，从扬声器放出来。
- 监听这路用非阻塞写，写不进去就丢包，不会拖慢发给队友的那一路。
- 音轨的采样率、声道数跟随注入音轨自动匹配。

**管线不再卡死**
- 原版注入音轨用阻塞写：没人占用虚拟麦克风时（比如 QQ 没按住说话），缓冲写满会把整个采集循环卡住，表现为监听一卡一卡、按住说话才流畅。
- 本版把注入和监听两路都改成非阻塞写，没人听的时候直接丢包，循环继续跑。

## 安装

1. **先卸载官方版 VirtuMic**（签名不同，不能覆盖安装）。
2. 安装 `VirtuMic-monitor-v2.apk`，允许「安装未知应用」。
3. 打开 App，连上无线调试，点 `Provide privilege to stream`，红点变绿。
4. 选要采集的 App，开始串流。

> 卸载/重装后特权会失效，需要重新点一次授权。以后官方更新也要先卸载本版。

## 已知限制

- 只覆盖 **App 采集模式**；File（音频文件）模式走的是另一条 Media3 管线，暂未加监听。
- 非阻塞写意味着极端情况下可能丢一小块声音，理论上队友或自己偶尔会听到轻微断续。
- 监听音轨走媒体音量，受手机媒体音量控制。
- 未在通话/耳机/蓝牙场景下逐一验证。

## 改动位置

只动了 `virtumic-core` 里的一个类 `PcmTransferJob`（打在 `classes3.dex` 上）：

- 新增字段 `monitorTrack: AudioTrack?`
- 新增方法 `createMonitorTrack(injectionTrack)`：按注入音轨的采样率/声道建一个 `STREAM_MUSIC` 音轨并播放
- `start()` 里创建监听音轨，`cleanup()` 里停止并释放
- `runTransferLoop()` 里同一块 PCM 先写监听、再写注入，两处都用 `WRITE_NON_BLOCKING`

其余代码、资源、界面均未改动。

## 致谢与许可

- 上游项目：[IndiDevs / VirtuMic](https://gitlab.com/IndiDevs/VirtuMic)（MIT License），本修改版同样遵循 MIT。
- 原项目作者：IndiTheo / IndiDevs。
- 本版仅为个人使用向的修改，非官方发布，与上游作者无关。

## 声明

- 本仓库是**非官方修改版**，基于 [IndiDevs / VirtuMic](https://gitlab.com/IndiDevs/VirtuMic) 的 MIT 许可代码，与原作者无关联，也不代表原作者立场。
- 原始版权：`Copyright (c) 2026 IndiTheo and IndiDevs`，完整许可证见仓库内 `LICENSE`（MIT）。
- 本修改版同样以 MIT 许可发布，保留上述版权声明。
- APK 内含 AndroidX、AndroidX Media3、Kotlin 等 Apache-2.0 组件，版权归各自作者。
- 本项目提供的 APK 为自签名构建，未使用官方签名；安装前需卸载官方版本。
