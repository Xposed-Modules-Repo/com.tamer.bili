# BiliTamer — 哔哩哔哩国际版增强模块 / Enhancement module for the international Bilibili app

> ## ⚠️ AI-generated module / 本模块由 AI 生成
> 本项目由大语言模型（AI）在人类指导下生成，包括全部 Hook 代码、设置界面、构建流水线与文档。
> 代码未经人工长期审计，请自行评估风险后使用；欢迎人工审查与 PR。
>
> This project was generated and iterated by a large language model (AI) under
> human direction, including all hook code, the settings UI, the build pipeline
> and these docs. The code has not been long-term audited by humans — evaluate
> the risk yourself; human review and PRs are welcome.

> 与哔哩哔哩公司无任何关联；B 站相关商标与版权归原厂所有。仅供学习与研究 Android Hook 技术。
> Not affiliated with Bilibili Inc.; trademarks and copyrights belong to their owners.
> For learning and research on Android hooking techniques only.

面向**国际版哔哩哔哩** `com.bilibili.app.in`（实测适配 **6.3.0 / 6.4.0 / 6.5.0 / 6.6.0 / 6.7.1**）的 LSPosed 模块。
An LSPosed module for the **international Bilibili app** (`com.bilibili.app.in`, tested against **6.3.0–6.7.1**).

源码、完整说明与逐版锚点校准见源仓：<https://github.com/mengwuzhuanshou/BiliTamer>

---

## 功能一览 / Features

| 功能 Feature | 说明 Description | 默认 Default |
| --- | --- | --- |
| 评论/主页 IP 属地 IP location | 把请求身份改写为国内版客户端，服务端返回 location 字段，评论区「IP属地：」与主页 IP 标签随之显示 / Rewrite request identity to the domestic client so the server returns the location field (comment-area "IP location" and profile IP tag) | 开 on |
| 身份声明范围 Identity scope | 评论区限定：仅评论/字幕请求声明国内版身份，其余请求保持国际版；或全局（旧行为）/ Scoped: declare the domestic identity for comment & subtitle requests only; or global (legacy behavior) | 评论区限定 scoped |
| 解码顺位 Decoder preference | AV1 > HEVC > H264 自动顺位，或锁定某一种（含锁定 H264 兜底档），或关闭（不干预）；自动顺位按设备硬解能力过滤请求位 / AV1 > HEVC > H264 auto preference, lock one (incl. H264 fallback) or off; auto mode filters request bits by hardware decode capability | 自动 auto |
| 音质顺位 Audio preference | 杜比全景声 > Hi-Res > AAC 自动顺位，或锁定，或关闭（不干预）/ Dolby > Hi-Res > AAC auto preference, lock or off | 自动 auto |
| HDR 画质顺位 HDR preference | HDR Vivid > HDR > SDR 自动顺位，或锁定/强制关闭，或关闭（不干预）/ HDR Vivid > HDR > SDR auto preference, lock/force-off or off | 自动 auto |
| 听视频听完暂停 Pause after video | 听视频（全屏音频播放器）播完当前视频即暂停，不自动连播 / Pause when the current video ends in the listen-mode fullscreen audio player instead of auto-advancing | 关 off |
| 隐藏互动提示 Hide interaction hints | 一键三连动画/文案、投票面板、UP 关注引导气泡 / Hide triple-action animation, vote panel and follow-bubble hints | 关 off |
| 首页不自动刷新 No home auto-refresh | 从后台/其它页面切回首页时不自动重载推荐流；下拉/点 tab/首次进入不受影响 / Skip the automatic feed reload when returning to the home page; manual refresh unaffected | 关 off |
| 首页顶栏消息入口 Top-bar message entry | 顶栏搜索栏右侧加消息图标，未读红点带数字 / A message icon on the top bar with an unread numeric badge | 开 on（6.4.0） |
| 首页头像→我的 Avatar as Mine entry | 顶栏头像点击投发宿主自己的「按路由切页」动作，打开完整「我的」页 / Avatar tap dispatches the host's own route action to open the full Mine page | 开 on（6.4.0） |
| 首页直播板块 Live channel unlock | 把 `bilibili://live/home` 追加回顶栏页列表（国际版服务端按身份裁掉了它）/ Append the live channel back into the top-bar page list (trimmed by the server for intl identities) | 开 on |
| 底栏删 tab Bottom-bar tabs | 移除「消息」tab、隐藏「我的」tab / Remove Message tab, hide Mine tab | 开 on（6.4.0） |
| 首页推荐分区屏蔽 Feed partition blocker | 按推荐卡分区标签（tname）整卡屏蔽；词表批量输入/检索/逐词移除 / Block feed cards by partition tag; bulk-edit/search/remove word list | 空词表不生效 |
| 首页只展示 UGC UGC-only feed | 移除官方合集/活动/直播等非用户上传卡（判据 `cardGoto=av`）；整批都不匹配时不过滤并告警 / Drop non-UGC feed cards; a batch with no match is left untouched instead of blanking the feed | 关 off |
| 干净的视频卡片 Clean video cards | 去掉卡片上的「竖屏」「1万点赞」类角标与推荐理由；UP 入口沿用宿主自己的名字行 / Strip portrait/like-count badges and recommendation reasons; the UP entry stays the host's own name line | 关 off |
| 禁止竖屏播放器 No portrait player | 竖屏卡路由 `bilibili://story/<id>` 改写为 `bilibili://video/<id>`，落进横屏播放器（丢弃卡片自带预载参数段）/ Rewrite story routes on feed cards to the landscape player's minimal route (preload query dropped) | 关 off |
| 关闭大卡片 No large cards | 按 `card_type` 移除占满整屏宽度的卡（轮播/大封面/内联播放），双列小卡与直播/广告卡不受影响 / Drop full-width feed cards by `card_type`; two-column, live and ad cards untouched | 关 off |
| 干掉云视听小电视 No activity overlay | 清空弹幕回包 `DmViewReply` 里的 `activity_meta` 活动浮层素材 / Clear the `activity_meta` activity-overlay material from the dm reply | 关 off |
| 干掉详情页广告位 No detail-page ads | 关掉播放器下方商品卡、简介区楼层广告与「视频提及」里的游戏推广卡（只摘 `MENTION_TYPE_GAME`，UP 自己的课程／充电链接不动）；**出厂默认开**，因为这些槽位是身份改写让服务端把客户端当国内版才排期下发的 / Strip the under-player product card, the description-area floor ad and game-promotion mentions (`MENTION_TYPE_GAME` only — the uploader's own course and purchase links stay); **on by default**, since these slots only appear because the identity rewrite makes the server schedule them as domestic | 开 on |
| 直播后台播放入口 Live background play | 解锁直播页的后台播放入口（无设置项，随模块生效）/ Unlock the background-play entry on the live page (no settings switch; active with the module) | 开 on |
| 下载加速 Download acceleration | 在线播放与离线下载改接本机回环代理，多 CDN 节点多区间并发取数、按序回吐；离线取到的字节前台播放直接复用（实验特性，缓存默认配额 2 GB、LRU，DRM/免流地址不接管）/ Route playback and offline downloads through a loopback proxy that fetches ranges concurrently from multiple CDN nodes; offline bytes are reused by foreground playback (beta; 2 GB LRU cache default; DRM/zero-rating URLs untouched) | 关 off |
| 配置同步 Config sync | 设置保存经启动投递+host-conf 代次协议生效，不依赖 root / Settings delivered at launch with a host-conf generation protocol — no root needed | — |

所有开关独立可逆；总开关关闭后模块完全休眠。
Every switch is independently reversible; the master switch disables the whole module.

## 环境要求 / Requirements

* 已 root 的 Android 设备：Magisk 或 KernelSU + Zygisk + LSPosed / rooted device with Zygisk + LSPosed;
* 国际版哔哩哔哩 6.3.0–6.7.1（com.bilibili.app.in，6.6.0 = 9130300）/ international Bilibili 6.3.0–6.7.1.

## 使用方法 / Installation

1. 安装本仓 Releases 里的最新 APK，LSPosed 中启用模块，作用域勾选「哔哩哔哩国际版」/ install the latest release APK, enable the module and select the Bilibili scope;
2. 强制停止哔哩哔哩后重新打开 / force-stop Bilibili and reopen;
3. 设置入口：LSPosed 模块详情页，或桌面「B站国际版增强」图标 / open settings from the LSPosed module page or the launcher icon;
4. 保存设置会自动拉起/投递到 B 站，无需 root；改开关后若未即时生效，强停重开一次 / saving settings auto-delivers the config without root; if a change does not take effect, force-stop and reopen.

### 实现要点 / How the identity rewrite works

* 评论/字幕走 KMP moss gRPC：在 moss-common-headers 拦截器里按 service/method 打 ThreadLocal
  标记，身份头提供者据此把 `x-bili-metadata-bin`/`x-bili-device-bin` 里的 mobiApp 从
  `android_i` 改为 `android_hd`（protobuf 变长长度前缀同步重建）。描述符族随版本整族换名，
  按「名字提示 + 形状校验」定位 / Comment & subtitle RPCs are scoped via the moss-common-headers
  interceptor: before `chain.proceed()` the service/method is read and a ThreadLocal marker set;
  the header provider then rewrites the mobiApp protobuf bytes (`android_i` → `android_hd`,
  rebuilding the varint length prefix). Obfuscated descriptor families are resolved by name
  hints plus shape validation;
* 空间页走 REST：改写空间页 API 专属拦截器 `addCommonParam` 里的 `mobi_app=android_i`，
  天然按页面定域 / Profile pages go through REST: the space-API-specific interceptor's
  `addCommonParam` rewrites `mobi_app` — scoped to the space page by construction;
* 详情页那一族只武装 `View` 一个方法：整族换身份时服务端会在回包里按国内端把弹幕池裁掉（视频内弹幕整片消失），
  而取流那族（`playerunite`/`playurl`）必须照常全武装，否则锁区稿件详情页能开、播放器却停在「加载失败」
  / Within the detail-page family only `View` is armed — arming the whole family makes the server trim the
  danmaku pool for the domestic client (in-video danmaku disappears), while the playurl family must stay
  fully armed or a region-locked video opens but the player stops at 加载失败;
* moss 二进制头取 `android_hd`（国内 HD 端）、空间页 REST 取 `android`（国内普通端）：两档不等价，纯 `android` 会顺带排下国内详情页广告位；IP 属地与锁区解锁两项两档实测等价。心跳/播放等其它服务保持国际版身份
  / The moss binary header carries `android_hd` while space-page REST params carry `android` — not equivalent: plain `android` also gets domestic detail-page ad slots scheduled in, while IP location and region-locked visibility measure the same on both; heartbeats and playback keep the international identity;
* 完整的机制、坑与逐版校准方法见源仓 [PITFALLS.md](https://github.com/mengwuzhuanshou/BiliTamer/blob/main/PITFALLS.md).

## 排查 / Troubleshooting

* LSPosed 日志过滤 `BiliTamer`：安装成功有 `hooks installed` 与各 hook `ok` 行
  / filter `BiliTamer` in LSPosed logs; look for `hooks installed` and per-hook `ok` lines;
* 详细日志开关打开后可见 `kmp header value rewritten` / `rest params rewritten` 等改写细节
  / enable verbose logging for rewrite details;
* 功能不生效：确认开关已开、作用域勾选、强停重开；升级 B 站后混淆锚点可能漂移导致静默失效，
  以日志为准 / after app upgrades the obfuscated anchors may drift silently — trust the logs.

## 已知限制 / Known limitations

* 仅适配实测版本 6.3.0–6.7.1；其它版本需自行校准混淆锚点 / tested against 6.3.0–6.7.1 only;
* 首页四项（只展示 UGC／干净卡片／禁止竖屏／关闭大卡片）挂在推荐流的协议解析出口上：只影响
  之后发出的流请求，已渲染的卡片要下拉刷新/切页/重进首页才会变 / The four home-feed switches
  act at the protocol parse exit, so they only affect subsequent feed requests — already
  rendered cards change after a pull-to-refresh, a tab switch or a relaunch;
* 「关闭大卡片」按 `card_type` 子串匹配，值域只在 6.6.0 上取过样本；服务端换卡型命名后该项
  退化为「什么都不删」，不会误删 / The large-card removal matches `card_type` substrings
  sampled on 6.6.0 only; if the server renames card types it removes nothing instead of guessing;
* 「干掉云视听小电视」只证到清空字段：挂点命中与条目清空有实机日志，浮层从屏幕消失未做
  同稿件肉眼 A/B / The activity-overlay switch is verified down to clearing the field; visual
  disappearance was not A/B'd, so it is not claimed;
* 「禁止竖屏播放器」实测只在 6.6.0 上生效；它丢弃卡片自带的预载参数段（这是不崩的前提），
  因此竖屏卡在横屏播放页起播时不走那条预载、分 P 固定从第一 P 解析 / The no-portrait rewrite
  is verified on 6.6.0 only; dropping the preload query (required to keep the target page from
  crashing) means those videos resolve from part 1;
* IP 属地依赖服务端策略，属风控敏感功能，是否显示由服务端决定；主页 IP 标签依赖账号与服务端
  返回，个别页面可能无该字段 / IP-location display is server-controlled and risk-control
  sensitive; the profile IP tag depends on the server response.

## 鸣谢 / Acknowledgments

| 项目 / Project | 贡献 / Contribution | 链接 / Link |
| --- | --- | --- |
| **BiliFix** (com.xjw.bilifix.in) | 身份声明思路与 libxposed 打包范式的启蒙参考（本模块与其无代码派生关系）/ the inspiration for the identity-declaration approach and libxposed packaging (no code derived from it) | https://github.com/xiaojiuwo233/BiliFix |
| **libxposed/api** | 现代 Xposed API / the modern Xposed API | https://github.com/libxposed/api |
| **MBGA** (top.trangle.mbga) | 首页三项与「干掉云视听小电视」的语义来源；本版按 6.6.0 的新接缝与新城载体重写，未复用代码 / the origin of the three home-feed features and the activity-overlay switch's semantics — reimplemented, no code reused | https://github.com/cledwynl/mbga |
| **AOSP dx / apksig** | 构建链组件（Apache-2.0）/ build-chain components | https://android.googlesource.com |

## 许可证 / License

MIT © mengwuzhuanshou，详见 LICENSE / MIT © mengwuzhuanshou. See LICENSE.
