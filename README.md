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

面向**国际版哔哩哔哩** `com.bilibili.app.in`（实测适配 **6.3.0 / 6.4.0**）的 LSPosed 模块。
An LSPosed module for the **international Bilibili app** (`com.bilibili.app.in`, tested against **6.3.0 / 6.4.0**).

---

## 功能一览 / Features

| 功能 Feature | 说明 Description | 默认 Default |
| --- | --- | --- |
| 评论/主页 IP 属地 IP location | 把请求身份改写为国内版客户端，服务端返回 location 字段，评论区「IP属地：」与主页 IP 标签随之显示 / Rewrite request identity to the domestic client so the server returns the location field (comment-area "IP location" and profile IP tag) | 开 on |
| 身份声明范围 Identity scope | 评论区限定：仅评论/字幕请求声明国内版身份，其余请求保持国际版；或全局（旧行为）/ Scoped: declare the domestic identity for comment & subtitle requests only; or global (legacy behavior) | 评论区限定 scoped |
| 解码顺位 Decoder preference | AV1 > HEVC > H264 自动顺位，或锁定某一种 / AV1 > HEVC > H264 auto preference, or lock one | 自动 auto |
| 音质顺位 Audio preference | 杜比全景声 > Hi-Res > AAC 自动顺位，或锁定 / Dolby > Hi-Res > AAC auto preference, or lock | 自动 auto |
| HDR 画质顺位 HDR preference | HDR Vivid > HDR > SDR 自动顺位，或锁定/关闭 / HDR Vivid > HDR > SDR auto preference, or lock/disable | 自动 auto |
| 听视频听完暂停 Pause after video | 听视频（全屏音频播放器）播完当前视频即暂停，不自动连播（零监听实现）/ Pause when the current video ends in the listen-mode fullscreen audio player instead of auto-advancing (zero-listener implementation) | 关 off |
| 隐藏互动提示 Hide interaction hints | 一键三连动画/文案、投票面板、UP 关注引导气泡 / Hide triple-action animation, vote panel and follow-bubble hints | 关 off |
| 首页不自动刷新 No home auto-refresh | 从后台/其它页面切回首页时不自动重载推荐流；下拉/点 tab/首次进入不受影响 / Skip the automatic feed reload when returning to the home page; manual refresh unaffected | 关 off |
| 分享到 QQ Share to QQ | 分享面板补回 QQ 渠道，点击走 B 站自带的 QQ 互联链路，弹出 QQ 分享面板选好友/群 / Restore the QQ channel in the share panel; tapping opens QQ's native share sheet (pick friends/groups) via the app's built-in QQ OpenSDK config | 开 on |

所有开关独立可逆；总开关关闭后模块完全休眠。
Every switch is independently reversible; the master switch disables the whole module.

## 环境要求 / Requirements

* 已 root 的 Android 设备：Magisk 或 KernelSU + Zygisk + LSPosed / rooted device with Zygisk + LSPosed;
* 国际版哔哩哔哩 6.3.0 / 6.4.0（com.bilibili.app.in）/ international Bilibili 6.3.0 / 6.4.0.

## 使用方法 / Installation

1. 安装 APK，LSPosed 中启用模块，作用域勾选「哔哩哔哩国际版」/ install the APK, enable the module and select the Bilibili scope;
2. 强制停止哔哩哔哩后重新打开 / force-stop Bilibili and reopen;
3. 设置入口：LSPosed 模块详情页，或桌面「B站国际版增强」图标 / open settings from the LSPosed module page or the launcher icon;
4. 改开关后需再次强停 B 站重开生效 / force-stop & reopen the app after changing switches.

### 实现要点 / How the identity rewrite works

* 评论/字幕走 KMP moss gRPC：拦截图库「moss-common-headers」拦截器取 service/method，
  proceed 前打 ThreadLocal 标记，身份头提供者按标记把
  `x-bili-metadata-bin`/`x-bili-device-bin` 里 mobiApp 字节从 `android_i` 改为 `android`
  （protobuf 变长长度前缀同步重建）。6.3.0 锚点 `up1.a.a()`、6.4.0 锚点 `kr1.a.a()`
  / Comment & subtitle RPCs are scoped via the moss-common-headers interceptor: before
  `chain.proceed()` the service/method is read and a ThreadLocal marker set; the header
  provider then rewrites the mobiApp protobuf bytes (`android_i` → `android`, rebuilding
  the varint length prefix). 6.3.0 anchor `up1.a.a()`, 6.4.0 anchor `kr1.a.a()`;
* 空间页走 REST：6.4.0 身份在 URL 参数里（`mobi_app=android_i`），hook 空间页 API 专属
  拦截器的 `addCommonParam` 改写之——天然按页面定域。6.3.0 锚点为 okretro 公共参数注入点
  `XA0.a` / Profile pages go through REST: on 6.4.0 the identity is a URL parameter
  (`mobi_app=android_i`), rewritten via the space-API-specific interceptor's
  `addCommonParam` — scoped to the space page by construction. The 6.3.0 anchor is the
  okretro common-param injection point `XA0.a`;
* 分享到 QQ：国际版分享面板的渠道列表由服务端下发（不含 QQ），而客户端白名单、图标文案
  与 QQ 互联配置（`assets/share_config.json` 的 qq.appId + tauth SDK + QQAssistActivity）
  原生齐全——向渠道 bean 的 getter 注入 share_channel="QQ" 条目（与微信同排）即可复用
  完整原生链路 / The intl share panel's channels are server-driven (no QQ), yet the client
  whitelist, icons and the QQ OpenSDK config are all present natively — injecting a
  share_channel="QQ" item into the channel bean's getter (same row as WeChat) restores the
  full native flow;
* 只重写 `android_i`→`android`，不触碰 android_hd；心跳/播放等其它服务保持国际版身份
  （日志可验证：每条改写行伴随同线程 armed 行）/ Heartbeats and other services keep the
  international identity — every rewrite line is paired with a same-thread "armed" line in the log;
* 机制、坑与校准方法详见 [PITFALLS.md](PITFALLS.md) / See PITFALLS.md for the full mechanism notes.

## 为什么用 libxposed API / Why libxposed

经典 `IXposedHookLoadPackage` 在 B 站主进程可能以 webview 名义触发（classLoader 错误），
或被厂商进程管理冻结导致不触发；libxposed 的 `onPackageReady` 直接给出正确 classLoader，
一次注入成功。打包声明用 `META-INF/xposed/java_init.list` + `module.prop` + `scope.list`，
manifest 不写 classic xposedmodule metadata。
The classic API is unreliable on the Bilibili main process (webview-provider triggers, vendor
process freezing). libxposed's `onPackageReady` delivers the right classLoader in one shot.

## 从源码构建 / Build from source

    python tools/build_module.py

* 无 Gradle / Android SDK 依赖：javac(--release 8, 编译桩) → dalvik-dx → 手写 AXML → zip(arsc 对齐) → apksig v1+v2+v3 签名，约 30 秒
  / No Gradle or Android SDK: javac(--release 8 with compile stubs) → dalvik-dx → hand-written AXML → aligned zip → apksig v1+v2+v3, ~30 s;
* 构建引擎完整内置于 `tools/engine/`（含 AOSP dx 与 apksig 两个 Apache-2.0 jar，见其 NOTICE.md），
  libxposed API jar 在 `tools/libxposed/`（io.github.libxposed:api 102.0.0，Apache-2.0，见其 PROVENANCE.md）
  / The engine is fully vendored in tools/engine/ (AOSP dx + apksig jars, Apache-2.0 — see NOTICE.md);
  the libxposed API jar is in tools/libxposed/ (io.github.libxposed:api 102.0.0, Apache-2.0 — see PROVENANCE.md);
* 需要 JDK（`BILITAMER_JDK` 或 `JAVA_HOME`，或 PATH 上有 javac/java）
  / Requires a JDK (BILITAMER_JDK or JAVA_HOME, or javac/java on PATH);
* 签名密钥经 gitignored 的 `tools/signing.local` 提供（KS_PATH=/KS_PASS=/KS_ALIAS=），仓库不含任何密钥；
  自行生成：`keytool -genkeypair -v -keystore my.jks -alias mymod -keyalg RSA -keysize 2048 -validity 10000`
  / The keystore is supplied via gitignored tools/signing.local — the repo contains no keys;
* 产物：`dist/BiliTamer-v<版本>.apk` / output at dist/BiliTamer-v<ver>.apk.

## 项目结构 / Layout

    BiliTamer/
    ├── module_conf.py                  # 构建配置（包名/版本/入口/图标）/ build config
    ├── apk/                            # 随仓库提交的发布 APK（CI 取此打包）/ committed release APK for CI
    ├── tools/
    │   ├── build_module.py             # 一键构建入口 / one-shot build entry
    │   ├── engine/                     # 自包含构建引擎（builder/axml/arsc/SignApk/dx/apksig/编译桩）/ vendored engine
    │   ├── libxposed/                  # libxposed API jar（Apache-2.0）+ PROVENANCE.md
    │   ├── signing.local               # 签名密钥（gitignored）/ keystore pointer (gitignored)
    │   └── icon/ic_launcher.png
    ├── app/src/main/
    │   ├── assets/xposed_init          # classic 入口占位（libxposed 模式下构建时不打包）/ classic placeholder
    │   └── java/com/tamer/bili/
    │       ├── MainHook.java           # libxposed 入口 / entry (XposedModule.onPackageReady)
    │       ├── BiliConfig.java         # 配置加载 / config loading
    │       ├── hooks/
    │       │   ├── HookApi.java            # hook 统一封装 / libxposed wrapper
    │       │   ├── IpLocationHooks.java    # 评论/主页 IP 属地 / scoped IP location
    │       │   ├── PlayerCodecHooks.java   # 解码/音质/HDR 顺位 / codec & audio & HDR preference
    │       │   ├── ListenPauseHooks.java   # 听视频听完暂停 / pause after video
    │       │   ├── InteractHintHooks.java  # 隐藏互动提示 / hide interaction hints
    │       │   └── HomeNoAutoRefreshHooks.java # 首页不自动刷新 / no home auto-refresh
    │       └── ui/SettingsActivity.java    # 纯代码设置界面 / code-only settings UI
    └── PITFALLS.md                     # 实现笔记与坑 / implementation notes & pitfalls

## 排查 / Troubleshooting

* LSPosed 日志过滤 `BiliTamer`：安装成功有 `hooks installed` 与各 hook `ok` 行
  / filter `BiliTamer` in LSPosed logs; look for `hooks installed` and per-hook `ok` lines;
* 详细日志开关打开后可见 `kmp header value rewritten` / `rest params rewritten` 等改写细节
  / enable verbose logging for rewrite details;
* 功能不生效：确认开关已开、作用域勾选、强停重开；升级 B 站后混淆锚点（`up1.a`/`XA0.a` 等）可能
  漂移导致静默失效，以日志为准 / after app upgrades the obfuscated anchors may drift silently —
  trust the logs, and see PITFALLS.md for calibration.

## 已知限制 / Known limitations

* 仅适配实测版本 6.3.0 / 6.4.0；其它版本需自行校准混淆锚点 / tested against 6.3.0 and
  6.4.0 only;
* 国际版评论区目前没有广告；横幅等广告仅在使用全局身份声明（v1.2 旧行为）时出现，
  默认的评论区限定模式无此副作用 / The international comment area currently has no ads;
  banner ads only appear when the legacy global identity declaration is used — the default
  scoped mode has no such side effect;
* IP 属地依赖服务端策略，属风控敏感功能，是否显示由服务端决定 / the IP-location display is
  server-controlled and risk-control sensitive;
* 主页 IP 标签依赖账号与服务端返回，个别页面可能无该字段 / the profile IP tag depends on the server response.

## 鸣谢 / Acknowledgments

| 项目 / Project | 贡献 / Contribution | 链接 / Link |
| --- | --- | --- |
| **BiliFix** (com.xjw.bilifix.in) | 身份声明思路与 libxposed 打包范式的启蒙参考（本模块与其无代码派生关系）/ the inspiration for the identity-declaration approach and libxposed packaging (no code derived from it) | https://github.com/xiaojiuwo233/BiliFix |
| **libxposed/api** | 现代 Xposed API / the modern Xposed API | https://github.com/libxposed/api |
| **AOSP dx / apksig** | 构建链组件（Apache-2.0）/ build-chain components | https://android.googlesource.com |

## 许可证 / License

MIT © mengwuzhuanshou，详见 LICENSE / MIT © mengwuzhuanshou. See LICENSE.
