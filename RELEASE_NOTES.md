# BiliTamer Release notes

## v1.6.1

* **修复 / Fixed**: 分发用户反馈的「播放随机黑屏、只有声音」：模块此前在解码**自动顺位**下
  无条件向服务端请求 AV1/HEVC 流（fnval 位强制 OR），不校验设备自身的硬解能力——没有
  HEVC/AV1 硬解的设备上播放器只能软解或解码失败，音频轨正常播放而画面黑。「随机」是因为
  不同视频服务端下发的编码不同。现自动顺位按 `MediaCodecList` 硬解能力过滤**请求位**：
  设备没有硬件解码器的编码不再写入 fnval，服务端即不下发对应流；**只过滤请求、不替换
  解码**——自动顺位的选择仍完全交给原逻辑，由其在服务端实际下发的流集合上自行回退
  （AVC 恒在），锁定 HEVC/AV1 行为不变。探测异常时按支持处理（fail-open，保持旧行为），
  结果进程内缓存。
  / Fixed the reported random black-screen-with-audio playback: in auto mode the module
  unconditionally requested AV1/HEVC streams via fnval without checking the device's own
  hardware decode capability, so devices without an HEVC/AV1 hardware decoder software-
  decode (or fail) while audio keeps playing. Auto mode now filters the requested format
  bits by MediaCodecList hardware-decoder availability: codecs the device cannot
  hardware-decode are never requested, so the server stops delivering them. Filter only,
  no substitution — the app's own preference logic still chooses freely among the
  delivered streams (AVC is always the baseline); locked modes are untouched. Probing
  failures fail open; results are cached per process.
* **新开关 / New**: 设置页「按硬解能力自动过滤 HEVC/AV1」（出厂默认开）。锁定 HEVC/AV1
  是用户显式选择，不被过滤，仅在设备无对应硬解时打一条警告日志 / New default-on switch
  "HW-decode auto filter" in settings. Locked HEVC/AV1 remain explicit user overrides
  (never filtered); a warning is logged once when the locked codec has no hw decoder.
* 构建 / Build: versionCode 11。

## v1.6.0

* **适配 / Adaptation**: 适配哔哩哔哩国际版 **6.4.0**（versionCode 不变，仍为 10）。6.4.0
  大规模混淆漂移（moss 身份链、播放器核心、okretro 参数点全部换名），全部旧锚点保留为
  6.3.0 候选，新锚点以候选列表形式并存 / Adapted to Bilibili international **6.4.0**. A
  large-scale obfuscation drift (moss identity chain, player core, okretro param injection)
  was remapped; every 6.3.0 anchor is kept as a candidate and the new anchors coexist in the
  same candidate lists.
* **修复 / Fixed**: 听视频「播完暂停」在 6.4.0 的全屏音频播放器上重新生效：6.4.0 听模式
  完成事件已不在 mini-player biz 层（旧钩子保留但触发不了），改挂播放器核心完成回调——
  播完后回退 0.8 秒并暂停、吞掉自动连播转发（completed 状态下直接 pause 是无操作，
  必须先 seek）/ Pause-after-video works again on 6.4.0's fullscreen audio player: the
  completion event left the mini-player biz layer, so the player-core completion callback is
  hooked instead — seek back 0.8 s, pause, and swallow the auto-next forwarding (pause()
  alone is a no-op in the completed state).
* **修复 / Fixed**: 空间页 IP 属地在 6.4.0 重新生效：6.4.0 空间请求的身份在 REST URL 参数
  （mobi_app=android_i）里而非 moss/proto 头，hook 空间页专属拦截器的 addCommonParam 改写
  之——天然按页面定域 / Profile-page IP location works again on 6.4.0: the space request
  identity now travels as a REST URL parameter (mobi_app=android_i) instead of a moss proto
  header; the space-specific interceptor's addCommonParam rewrites it, which is scoped to the
  space page by construction.
* **杂项 / Misc**: 解码/音质/HDR 顺位、首页不自动刷新、隐藏互动提示、分享到 QQ 均在 6.4.0
  复测通过 / codec/audio/HDR preference, no-home-auto-refresh, interaction-hint hiding and
  Share-to-QQ re-verified on 6.4.0.
* 构建 / Build: versionCode 10。

## v1.5.0

* **撤回 / Withdrawn**: 倍速解锁功能应要求撤回，不随本版本发布。该功能已完成开发与
  实机验证（菜单注入 + 内核放行 + 时钟探针），技术结论存档于 PITFALLS #15：**native
  层存在 3.0x 硬钳制**（`ffp_set_playback_rate` 从内部配置系统读上限覆盖请求值，
  `FFP_PROP_FLOAT_MAX_SPEED` 为只读遥测），>3 档位为观感 placebo，突破需 Zygisk
  原生 hook。下次若重启该功能，按 PITFALLS #14/#15 与 git 历史直接重建，无需重新逆向。
  / The speed-unlock feature is withdrawn before release at the user's request. It was
  fully developed and device-verified; findings are archived in PITFALLS #15: the native
  layer clamps playback rate at 3.0x regardless of the requested value, entries above 3x
  are placebo, and breaking it would require a Zygisk native hook.
* **移除 / Removed**: AI 字幕源功能（含其绑定的 dm.v1 评论区限定分支）——实验性价值
  有限且与 IP 属地共享的身份链路已由 scoped 模式覆盖 / The AI-subtitle feature is
  removed (including its dm.v1 scoped-identity branch).
* 构建 / Build: versionCode 9。

## v1.4.0

* **新功能 / New**: 分享面板补回「分享到 QQ」入口（默认开）：向服务端下发的渠道列表注入
  share_channel="QQ" 条目（与微信同排），点击复用 B 站自带 QQ 互联链路
  （tauth + share_config.json 的 qq.appId），弹出 QQ 分享面板选好友/群——国内版同款效果。
  实机验证：QQ 项与微信同行显示，点击拉起 com.tencent.mobileqq 的
  QPublicTransFragmentActivity（QQ 分享确认页）。未安装 QQ 时面板自动隐藏该渠道。
  / New Share-to-QQ entry in the share panel (on by default): a share_channel="QQ" item is
  injected into the server-driven channel list (same row as WeChat), and tapping it reuses
  the app's native QQ OpenSDK flow — verified on device (QQ's share confirmation page opens).
  The entry hides automatically when QQ is not installed.

## v1.3.2

* **新功能 / New**: IP 属地「评论区限定」模式（默认）：仅评论与 AI 字幕 gRPC 请求声明国内版身份，
  心跳/播放/首页等其它请求保持国际版身份 / Scoped IP-location mode (now default): only comment
  & AI-subtitle gRPC requests declare the domestic identity; heartbeats, playback and all other
  services keep the international identity.
* **修复 / Fixed**: 评论区 IP 属地在「全局」模式下也曾不显示——改写时机必须落在
  moss-common-headers 拦截器 proceed 之前 / Comment-area IP location now actually displayed:
  the rewrite must happen before the moss-common-headers interceptor proceeds.
* **修复 / Fixed**: 听视频「播完暂停」锚点改为签名匹配，功能正式生效 / Listen-pause anchor
  switched to signature matching; the feature now works.
* **杂项 / Misc**: 详细日志关闭时保留每类改写的首条探针日志 / first-probe logging keeps
  diagnostics visible when verbose logging is off.
* 评论区限定模式（默认）下评论区无广告副作用；横幅等广告仅出现在全局声明（v1.2 旧行为）下
  / the scoped mode (default) introduces no ads into the comment area; banner ads only appear
  under the legacy global declaration.

## v1.3.1 / v1.3.0

* IP 属地架构改造与作用域模式实验（详见 PITFALLS.md #6/#7）/ IP-location rework and scoped-mode
  experiments (see PITFALLS.md).

## v1.2.0

* 探针日志模式；听视频/隐藏互动提示/首页不自动刷新等功能落地 / probe logging; listen-pause,
  interaction-hint hiding and no-home-auto-refresh features.
