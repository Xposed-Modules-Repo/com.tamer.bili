## v1.7.11

> 本版包含未单独发布的 v1.7.10 全部变更；1.7.9 及以下用户直接安装本版。
> This release supersedes the never-published v1.7.10 — install directly over 1.7.9 or below.

* **修复：听完自动暂停在 6.5.0 听视频上不生效 / Fixed: pause-after-listen now works on
  6.5.0 listen mode**: 旧实现挂在播放器的完成回调上吞事件——6.5.0 的听视频框架**根本不用
  这个回调决定切集**（真机取证：完成事件后 138ms 框架独立发起下一集加载，循环/顺序模式
  路径相同）。本版改为两层：完成事件到达时把播放器停在片尾前 0.8s 并暂停（用户可见的
  「听完暂停」），随后 6 秒守卫窗内拦截播放器实例上的一切加载/起播调用。真机验证：完成后
  138ms 框架试图起播下一集，被守卫精确拦下——**下一集不会自动播放**；注：框架自己的列表
  指针仍会前进（UI 显示切到下一集），守卫拦的是播放而非列表状态，这是当前语义。普通视频
  行为不受影响（播完原生即暂停）。零监听、零轮询。
  / The old hook swallowed the player's completion callback — but 6.5.0's listen mode
  decides episode switching independently (device evidence: the framework fired the
  next-episode load 138 ms after the completion event). Now: on completion the player is
  parked 0.8 s before the end and paused, and a 6-second guard window blocks every
  load/start call on the player instance, so the next episode never auto-plays. Note: the
  framework's own playlist pointer still advances (the UI jumps to the next episode) — the
  guard blocks playback, not the list state. Verified on device; normal (non-listen)
  playback is untouched. No listeners, no polling.
* **修复：首页「不自动刷新」在 6.5.0 失效 / Fixed: no-home-auto-refresh ineffective on 6.5.0**:
  6.5.0 把 feed 状态对象改成了嵌套结构（列表在 state.a.a 两层深），「已有内容」判定只扫
  直接字段，永远判成空状态 → 每次自动刷新都被放行。现改为深度受限的递归搜索，并加探针
  （fired / allowed-empty / blocked 三态首触日志）。真机验证：切后台返回时
  `auto refresh blocked, type=AUTO_BACK_FROM_OTHER_PAGE`。/ 6.5.0 nested the feed state
  object, so the has-content guard always saw "empty" and let every auto-refresh through.
  The guard now searches nested fields, with first-fire probes for tri-state diagnosis;
  verified on device (background-return refreshes are blocked).
* **新增：直播后台播放入口（6.5.0）/ New: live background-play entry on 6.5.0**:
  国际版 6.5.0 直播间的播放器设置面板里，「后台播放」（应用退至后台，可继续播放）一项被
  房间 specialType 判定跳过而不创建。本版把该判定强制放行——设置面板恢复显示「后台播放」
  开关，打开后退出到后台直播声音继续。真机验证：开关出现、后台播放正常（MediaSession
  PLAYING 持续）。注意这与「仅播声音」（观看中切纯音频）是两个特性，后者仍受限于播放器
  元数据检查，未在本版处理。/ On 6.5.0 the live-room settings panel skipped creating the
  "Background play" entry behind a room specialType check. The check is now forced open —
  the entry is back, and toggling it keeps live audio playing after leaving the app.
  Verified on device (MediaSession stays PLAYING in background). Note this is distinct
  from audio-only switching while watching, which remains gated and is not addressed here.
* **修复：头像 →「我的」打开完整页面（6.5.0）/ Fixed: avatar → full Mine page on 6.5.0**:
  6.5.0 上底栏 tab 选中动作类从 `FC1.c` 漂移为 `jD1.b/jD1.c`（`HomeFrameViewModel.w0` 参数
  接口同组换名，方法名未漂移）、`tab_host` 资源 id 从 0x7f0938b4 漂到 0x7f0938d3——真实
  派发、合成点击、tab 服务三级入口全部失守，头像降级为深链，打开的「我的」页面不完整。
  现在动作类按候选列表（jD1.c/FC1.c）+ 形状校验解析，全失败时把 `w0` 参数接口名打进日志
  （下次漂移一行日志定位）；合成点击的 `tab_host` 查找失败时按资源名运行时解析兜底。
  实机验证：头像点击走真实派发，页面含离线缓存/历史/收藏/创作中心等完整功能区。
  / On 6.5.0 the bottom-bar tab-select action class drifted (FC1.c → jD1.c) and the
  tab_host resource id moved, so all three entry paths failed and the avatar fell back to
  the incomplete deep-link shell. The action class is now resolved from a candidate list
  with shape validation (falling back to logging the dispatch interface for the next
  drift), and the tab-host lookup falls back to resolving by resource name.
* **移除：分享面板「分享到 QQ」/ Removed: Share-to-QQ entry**: QQ 侧对重签名包的
  「非官方应用 25201」校验已覆盖全部宿主版本（6.3.0 也失效），该 hook 无存在意义，连同
  设置项一并删除。/ QQ now rejects repackaged builds on every supported host version, so
  the injection hook and its settings entry are removed.
* **IP 属地改写点加活体计数 / Liveness counters on the identity-rewrite point**: 排查
  「评论区属地失效」时发现 once-per-process 探针会掩盖长会话中的钩子失效。改写点现在带
  滚动计数（每 200 次写打一条 alive，每 200 次实际改打一条心跳），与 moss RPC 计数对照
  即可分层定位：钩子死 / 服务端行为变 / 传输路径变。/ The rewrite hook now logs rolling
  counters so a mid-session hook death is distinguishable from a server-side change —
  the previous once-per-process probe could mask exactly that.
* 构建 / Build: versionCode 23。
