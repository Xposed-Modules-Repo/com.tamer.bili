# BiliTamer Release notes

## v1.7.2

* **修复：顶栏入口随服务器新增分区栏错位 / Fixed: top-bar entries misaligning after
  the server added a section bar**: 6.4.0 顶栏由服务器下发了新的分区栏（如「推荐/动画」
  一行，App 版本未更新即生效）。v1.7.0 的「我的」入口与消息图标是用「追加到容器末尾 +
  负 topMargin」叠进顶栏的，该写法假定顶栏容器下只有单一内容行；服务器插入分区栏后
  入口被挤到分区栏一行，与顶栏头像脱节。本版改为「内容行用 FrameLayout 包裹、入口与
  内容行同层叠放」：入口恒与内容行对齐，服务器再往下插行也不影响。实机验证：入口回到
  顶栏行，消息图标点开消息页、头像点开完整「我的」页均正常。/ The 6.4.0 top bar received
  a server-delivered section bar (e.g. a Recommended/Anime row) without an app update.
  The v1.7.0 "Mine" entry and message icon were overlaid via "append to container end +
  negative top margin", which assumed the top bar held a single content row; the
  server-inserted section bar pushed the entries into the wrong row. Entries are now
  anchored inside a FrameLayout wrapper around the content row, so they stay aligned
  with it no matter what rows the server adds below. Verified on device: entries back on
  the top bar row; the message icon opens the IM page and the avatar opens the full Mine
  page.
* **包含 v1.7.1 全部变更 / Supersedes v1.7.1**: v1.7.1 未单独发布，本版包含其全部变更
  （黑屏过滤真正生效、锁定 H264、解码/音质/HDR 关闭档）。1.7.0 及以下用户直接安装本版。
  / v1.7.1 was not published separately; this release contains all of its changes
  (effective black-screen filter, lock H.264, off/untouched modes). Install directly if
  you are on 1.7.0 or earlier.
* 构建 / Build: versionCode 14。

## v1.7.1

* **修复：黑屏（有声无画面）过滤真正生效 / Fixed: HW-decode filter now actually takes
  effect**: v1.6.1 引入的「按硬解能力过滤」此前只在服务端请求位上 OR 加位、从不清位，
  而宿主 App 自身会按自家（乐观的）能力检测预先置好 AV1/HEVC 请求位——模块的过滤
  对宿主已置的位形同虚设，设备硬解运行时失败的流照样下发，黑屏依旧。本版起 fnval 位
  改写改为「先清后设」：自动顺位下设备没有硬解的编码位会被从请求中移除，服务端不再
  下发对应流。这是黑屏反馈的核心修复。/ The hardware-decode filter added in v1.6.1
  previously only OR-ed format bits into the request without ever clearing the bits the
  host app had already set from its own (optimistic) capability detection, so streams
  the device fails to hardware-decode at runtime were still delivered and black screens
  persisted. fnval bit handling now explicitly clears and re-sets bits: in auto mode,
  codecs the device cannot hardware-decode are removed from the request so the server
  stops delivering them. This is the core black-screen fix.
* **新增：锁定 H264 / New: lock to H.264**: 视频解码新增「锁定 H264」档位——只请求
  H.264，清掉 AV1/HEVC/H266 请求位。兼容性最好的兜底档：解码异常、黑屏、卡顿的设备
  可显式切到此档。/ New "Lock H.264" codec mode — requests H.264 only (AV1/HEVC/H266
  bits cleared). Most-compatible fallback for devices that glitch on modern codecs.
* **新增：解码/音质/HDR 均可完全关闭 / New: decode, audio and HDR can each be fully
  off**: 三组新增「关闭（不干预）」档——模块完全不触碰对应的请求位与选择逻辑，纯
  App 原行为，便于逐项排查是模块哪一路改动引发的问题。/ New "Off (no intervention)"
  option for each of codec, audio quality and HDR: the module leaves the corresponding
  request bits and selection logic completely untouched (pure app behaviour), useful
  for isolating which module change causes an issue.
* **音质/HDR 布局归位 / Audio & HDR moved under Player section**（设置页原误置于首页
  布局段）。
* 构建 / Build: versionCode 13。

## v1.7.0

* **首页布局对齐国内版 / CN-style home layout (6.4.0)**:
  - 顶栏搜索栏右侧加消息入口（代码自绘信封图标，不依赖目标资源），未读时显示红点+数字
    角标（与消息页角标同源，99+ 封顶）/ A message entry on the top bar (self-drawn icon,
    no target resources); red-dot numeric badge fed by the same source as the in-app IM
    badge (cap 99+).
  - 顶栏左侧头像变为「我的」入口：经真实 tab 派发打开完整「我的」页（不再是深链的
    不完整壳页面）/ Tapping the top-left avatar now opens the full "Mine" tab page via the
    app's own tab-select action (not the incomplete deep-link shell).
  - 底栏移除「消息」tab（数据级，页面一并收敛）；「我的」tab 默认隐藏（渲染级：数据保留，
    顶栏头像入口仍可打开完整页）/ Bottom bar: the Message tab is removed at data level;
    the Mine tab is hidden at render level by default (data kept so the avatar entry keeps
    opening the full page).
* **首页推荐分区屏蔽 / Feed partition (tag) blocker**: 按推荐卡的分区标签（tname）整卡屏蔽，
  词表管理支持批量输入（中英文逗号/分号/顿号/换行均可）、搜索定位、逐词移除，**无词数上限**。
  匹配为包含关系（加「游戏」会连带「主机游戏」）。/ Block feed cards whose partition tag
  (tname) contains any blocked word. Word list editor: bulk input (commas/semicolons/
  newlines), search-locate, per-word removal, no word-count limit. Matching is substring.
* **配置通道最小权限化 / Minimal-permission config channel**: 设置保存不再依赖 root——保存时
  携带配置+代次拉起目标应用，模块在宿主进程截获并写入宿主自有副本（host-conf），重启后按
  conf_gen 代次协议选用最新配置（陈旧副本永不反盖）。root 仅作开发兜底；LSPosed 2.2.0
  已弃用的 XSharedPreferences 通道不再使用。/ Saving settings no longer needs root:
  the app delivers the config (with a generation stamp) by launching the target, the module
  intercepts it in-process and persists to the host's own copy; on every start the newest
  generation wins. Root stays a dev-only fallback; the deprecated XSharedPreferences
  channel is not used.
* **分享到 QQ（6.4.0 门控）/ Share-to-QQ gating (6.4.0)**: 6.4.0 上 QQ 客户端校验调用方签名
  （错误码 25201），重签名包无法原生卡片分享——分享面板不再注入 QQ 渠道以免误触报错；
  6.3.0 保持原生注入。/ On 6.4.0 QQ verifies the caller signature (error 25201), so native
  card share is impossible for a repackaged build — the QQ entry is not injected into the
  share panel; 6.3.0 keeps the native entry.
* 构建 / Build: versionCode 12。

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