# BiliTamer（B站国际版增强 / International Bilibili Enhancer）

> ## ⚠️ AI-generated module / 本模块由 AI 生成
> 本项目由大语言模型（AI）在人类指导下生成，包括全部 Hook 代码、设置界面、构建流水线与文档。
> 代码未经人工长期审计，请自行评估风险后使用；欢迎人工审查与 PR。
>
> This project was generated and iterated by a large language model (AI) under
> human direction, including all hook code, the settings UI, the build pipeline
> and these docs. The code has not been long-term audited by humans — evaluate
> the risk yourself; human review and PRs are welcome.

面向**国际版哔哩哔哩** `com.bilibili.app.in`（实测适配 **6.3.0**）的 LSPosed 模块。
An LSPosed module for the **international Bilibili app** (`com.bilibili.app.in`, tested against **6.3.0**).

---

## 功能特性 / Features

| 开关 Switch (默认 Default) | 说明 / Description |
| --- | --- |
| ip（开/on） | 评论/主页 IP 属地：请求身份改写为国内版客户端，服务端返回 location 字段 / IP location: request identity rewritten to the domestic client so the server returns the location field |
| ip_scope_mode（评论区限定/scoped，默认） | 仅评论与 AI 字幕 gRPC 请求声明国内版身份；心跳/播放等其它请求保持国际版 / declare the domestic identity for comment & AI-subtitle gRPC calls only; heartbeats and playback keep the international identity |
| ai_sub（关/off） | AI 字幕源：弹幕接口按国内版身份请求，播放器出现 AI 字幕轨道，字幕 URL 打进日志 / AI subtitles: DmView requests carry the domestic identity so the AI-subtitle track appears; subtitle URL logged for export |
| codec/audio/hdr（自动/auto） | 解码 AV1>HEVC>H264、音质 杜比>Hi-Res>AAC、画质 HDR Vivid>HDR>SDR 自动顺位，可锁定 / decoder, audio and HDR auto preferences, lockable |
| listen_pause（关/off） | 听视频播完当前视频即暂停，不自动连播（零监听）/ pause when the current video ends in mini-player (zero-listener) |
| hide_triple / hide_vote / hide_up（关/off） | 隐藏一键三连动画/投票面板/UP 关注引导 / hide triple-action animation, vote panel and follow-bubble hints |
| no_refresh（关/off） | 切回首页不自动重载推荐流 / skip the automatic feed reload when returning home |

所有开关独立可逆；总开关关闭后模块完全休眠。
Every switch is independently reversible; the master switch disables the whole module.

## 身份声明机制 / How the identity rewrite works

* 评论区/字幕走 KMP moss gRPC：拦截图库 moss-common-headers 拦截器取 service/method，
  `chain.proceed()` 前打 ThreadLocal 标记，身份头提供者按标记把
  `x-bili-metadata-bin`/`x-bili-device-bin` 的 mobiApp 字节 `android_i`→`android`
  （protobuf 变长长度前缀同步重建）/ scoped via the moss-common-headers interceptor:
  before proceed, service/method is read and a ThreadLocal marker set; the header provider
  then rewrites the mobiApp protobuf bytes with the length prefix rebuilt;
* 主页走 REST：okretro 公共参数注入点按 URL 作用域改 `mobi_app=android` / profile pages
  via the okretro common-param injection point;
* 每条改写行伴随同线程 armed 行可审计 / every rewrite line is paired with a same-thread
  "armed" line in the log;
* 只重写 `android_i`→`android`，心跳/播放/首页等保持国际版身份 / heartbeats and other
  services keep the international identity.

## 安装使用 / Installation

1. Magisk/KernelSU + Zygisk + LSPosed 环境 / rooted device with Zygisk + LSPosed;
2. 安装 APK / install the APK;
3. LSPosed 中启用模块，作用域勾选 **哔哩哔哩国际版(com.bilibili.app.in)** / enable and select the Bilibili scope;
4. 打开模块桌面图标调整开关 / open the launcher icon to toggle switches;
5. **强制停止 B 站后重新打开**生效 / force-stop Bilibili and reopen for changes to take effect.

日志：logcat/LSPosed 过滤 `BiliTamer`。
Logs: filter `BiliTamer`.

## 已知限制 / Known limitations

* 仅适配实测版本 6.3.0，升级后混淆锚点可能漂移（看 `hook FAILED` 日志）/ tested against 6.3.0 only;
* 国际版评论区目前没有广告；横幅等广告仅在使用全局身份声明（旧行为）时出现，默认的
  评论区限定模式无此副作用 / the international comment area currently has no ads; banner ads
  only appear under the legacy global identity declaration — the default scoped mode has no
  such side effect;
* IP 属地显示由服务端策略决定，属风控敏感功能 / the IP-location display is server-controlled.

## 鸣谢 / Acknowledgments

| 项目 / Project | 贡献 / Contribution | 链接 / Link |
| --- | --- | --- |
| **BiliFix** (com.xjw.bilifix.in) | 身份声明思路与 libxposed 打包范式的启蒙参考（无代码派生）/ inspiration for the identity-declaration approach and libxposed packaging (no code derived) | https://github.com/xiaojiuwo233/BiliFix |
| **libxposed/api** | 现代 Xposed API | https://github.com/libxposed/api |

## 许可证 / License

MIT © mengwuzhuanshou，详见 LICENSE / MIT © mengwuzhuanshou. See LICENSE.
源码 / Source: https://github.com/mengwuzhuanshou/BiliTamer
