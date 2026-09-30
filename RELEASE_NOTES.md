## v1.7.13

> 本版包含未单独发布的 v1.7.12 全部变更；1.7.11 及以下直接安装本版即可。
> This release supersedes the never-published v1.7.12 — install directly over 1.7.11 or below.

* **国际版 6.6.0（9130300）适配 / Adapted to host 6.6.0**: 宿主升到 6.6.0 后混淆锚点整族换包，
  本轮每个落点都按「dex 里按形状+角色反查 → 装机一行日志验收」双证重新定位：
  `fnval` 计算 `kJ1.a` → `aK1.a`（真机 `fnval int 17364 -> 84948`）、听模式完成监听器
  `CE1.f` → `lK1.j`、首页分区屏蔽的 feed 解析入口 `pegasus.request.g` → `request.h`、
  直播后台播放门 `HX.c$b/$c.q1()` → `JX.b$b/$c.m1()`、gRPC 身份描述符族 `kr1.a..n` →
  `xr1.a..n`。旧名在 6.6.0 大多已被 R8 复用成无关类，所以每条都必须过形状/角色校验，
  只把新名字加进候选表是不够的。同时移除三条在 6.5.0 就已经断死的 IP 兜底链
  （`ip1.h` 线程标记、`mq0.a/oq0.a` 身份提供者、`kr1.a/up1.a` 头提供者 —— dex 实证类不在
  或形状不符），它们只会贡献重试次数和误导性 ERROR；已消亡的 main2 底栏与 mini-player
  biz 层探针也降为 debug，ERROR 只留给真故障。
  / Every obfuscated anchor moved as a family on 6.6.0; each seam was re-located statically by
  shape and role, then confirmed by one log line on device. Three identity-rewrite fallback
  chains that had already died on 6.5.0 were removed rather than kept as noise.
* **修复：多 CDN 加速的两处「按计划在走、事实没在走」 / Fixed: two plan-vs-reality bugs in the
  multi-CDN accelerator**: ① 只镜像了半份文件的节点会把**中段**子块按它自己的短总长截短交付，
  旧代码把截短当合法的尾部 EOF，然后跳到下一个**计划**子块的起点，响应体里留下空洞 —— 真机
  表现为播到中段花屏、往后拖一下进度条才恢复；② 起播之后某个子块三轮候选全败时旧代码直接
  向上抛异常，而代理的兜底只认「响应尚未开口」，等于把视频流当场砍尾（画面定格、进度条与
  音频照走）。两处现在都改成「交付到事实边界，再从断点单连接续传到计划末尾」（等价官方单
  连接行为，不是装饰性兜底），并给 sink 写失败补上取消记账（挂断不计节点账）。另外续传通道
  在全部候选都被退避挡掉时退回地址全集 —— 兜底通道没有第二选择，退避对它只是排程提示。
  桌面回归先复现了这两处错（修复前红、修复后绿），967 项断言全过。
  / A partially mirrored CDN truncated a middle piece and the merge loop skipped the hole that
  left (mid-video artifacts); a permanently failed piece threw after the response had already
  started, which cut the stream off. Both now deliver to the real boundary and resume on one
  connection.
* **修复：首页「不自动刷新」在 6.6.0 上拦错了类型 / Fixed: no-home-auto-refresh vetoed the
  wrong flush type on 6.6.0**: 旧名单只拦 `AUTO_BACK_FROM_BACKGROUND` /
  `AUTO_BACK_FROM_OTHER_PAGE`，而 6.6.0 上真正会发出来的自动刷新是 `FLUSH_ON_BACK_PRESS`
  （停在首页 tab 按返回键时整屏换推荐流），于是钩子「装了、也调了、但永远判成不该拦」。
  名单现在按结构匹配（不认方法名）覆盖 `FLUSH_ON_BACK_PRESS` + 三个 `AUTO_BACK_*`，
  并保留「ViewModel 已有内容才拦」的空状态放行，避免页面重建后首屏空白。
  真机 A/B（国际版 6.6.0，同一触发动作在开关两侧各跑一次）：
  关闭时 14:57:48 按 BACK → `type=FLUSH_ON_BACK_PRESS` → 首页 5 条标题全换（0 重合）；
  开启时 15:05:19 同样动作 → `auto refresh blocked, type=FLUSH_ON_BACK_PRESS` →
  5/5 标题与基线完全一致；手动下拉 `type=PULL_DOWN` 不受影响（照常刷新）。
  / The old list only matched `AUTO_BACK_FROM_*`, but on 6.6.0 the refresh that actually
  fires is `FLUSH_ON_BACK_PRESS` (back-press while sitting on the home tab replaces the
  whole feed), so the hook was installed yet never matched. Verified on device with an
  A/B on the same gesture: off → feed fully replaced; on → blocked and 5/5 titles
  unchanged; manual pull-to-refresh still works.
* **取证订正（写进代码注释，避免下次照抄）/ Corrected evidence, recorded in the source**:
  本轮一度记下「`AutoRefreshComponent#w(Z)`（`AUTO_BACK_FROM_*` 的读取方）在 6.5.0/6.6.0
  都没有调用点」，那是**扫描口径错误**造成的假结论：dex 的 `invoke-virtual` 记的是**声明类**，
  而 `w(Z)` 覆写自父类 `com.bilibili.pegasus.b`，按声明类重查后这条链是活的
  （`BasePegasusFragment#am(Z)` ← `Zl(I,I)` ← `onResume/onPause/onFragmentShow/onFragmentHide`），
  首页每次可见性变化都会过它。真机本轮没触发 `AUTO_BACK_*` 的真正原因也查清了：三个分支的
  时间阈值与页面白名单全部来自服务端下发的 Pegasus 配置，当前设备/账号上这些值为 0，`w` 在
  调用加载桥之前就返回 —— 所以「关掉功能做对照也不刷」不能当作钩子起效的证据，必须找同一动作
  在开关两侧都能观察到差异的触发点。名单仍维持显式四条而不用宿主自带的 `isUserRequest()`：
  它把 `TAB_DOUBLE_CLICK`、`BOTTOM_REFRESH_BUTTON_CLICK` 也判成「非用户请求」，照它拦会把
  双击 tab 和底部刷新按钮一并弄坏。
  / An earlier "no call site for `AutoRefreshComponent#w`" reading was a scan artifact: dex
  `invoke-virtual` records the *declaring* class, and `w` overrides `com.bilibili.pegasus.b#w`,
  so the chain is in fact live from the fragment lifecycle. The reason it stayed quiet on
  device is that all `AUTO_BACK_*` thresholds come from server-delivered config and are 0 on
  this account — which is why "nothing changed with the feature off" proves nothing.
