## v1.9.0

* **起播不再先跑一趟探路 / Startup no longer waits for a probe round trip**: v1.8.0 把「问 1 个字节」的探路
  串在起播请求之前，首字节成了两段往返相加，候选全黑时还要把探路预算等满；现在第一条候选立刻开流、
  探路只在旁边问「还有谁活着、文件多大」（桌面回归 631→306 ms、929→5 ms）。剩下的快慢就看节点给字节有多快。
  / The first candidate now streams immediately while probing runs alongside it, so startup pays one
  round trip instead of two.
* **新增两个开关 / Two new switches**: 「关闭大卡片」（`feed_no_large_card`，按卡片类型名移除
  占满整屏宽度的轮播卡/大封面卡/内联播放卡，双列小卡一张不动；实测它与「只展示 UGC」重叠，
  但那条会连直播卡和广告卡一起删，这条只管大卡）与「干掉云视听小电视」
  （`player_no_activity_meta`，清空弹幕回包里的 `activity_meta` 活动浮层素材，移植自 MBGA）；
  均出厂默认关。/ A big-card remover keyed on card type, and a switch clearing the dm reply's
  `activity_meta` activity-overlay payload; both off by default.
* **「干掉云视听小电视」只证到清空、未证到界面消失 / clearing proven, disappearance not**:
  挂点、命中、清空三步有实机日志，而浮层从屏上消失要在活动期拿同一稿件肉眼 A/B，本版不声称已证。
  / The hook fires and clears a real payload; the visual removal still needs an on-device A/B.

## v1.8.0

> **需要先卸载旧版 / Uninstall the old version first**: 本版换了签名密钥，从 1.7.x 及更早版本原地升级
> 会被系统拒绝（`INSTALL_FAILED_UPDATE_INCOMPATIBLE`）；请先卸载再安装，卸载会清掉模块自己保存的开关
> 设置，装好后重新勾一遍。 / This build is signed with a new key, so updating in place is rejected —
> uninstall first, then install.

* **首页推荐流清洗三项（移植自 MBGA）/ Three home-feed cleanup switches**: 只展示 UGC（官方合集、活动、
  直播等 `cardGoto≠av` 的整卡移除）、干净的视频卡片（去推荐理由与竖屏角标，UP 主入口沿用宿主自己的名字行）、
  禁止竖屏播放器（卡片自带的竖屏路由改写为横屏播放页路由）；三项都会改变首页内容，出厂默认关，
  设置里各自独立可逆。 / UGC-only feed, clean video cards (no reason row, no portrait badge) and a
  no-portrait-player switch — all three off by default.
* **起播不再卡在 0 KB / Startup no longer stalls at 0 KB**: 首字节不再等「一整块 256 KiB 先下完再吐」，
  改成先用 1 个字节问出哪个节点还活着、然后在一条连接上边收边吐（每口 ≤64 KiB）；同时并发的多条播放流
  不再互相抢光并发许可。 / The first byte no longer waits for a fully buffered piece, and concurrent
  playbacks no longer starve each other on the shared concurrency permits.
* **拖回看过的区间不再重新加载 / Seeking back no longer re-downloads**: 块清单原先只在连接关闭时落盘，
  而长视频的播放轨是一条十几分钟不关闭的连接——中途进程被杀就「字节在盘上、清单却不存在」，整条视频重下；
  现在每 8 MiB 落一次，实机重开同一条视频的读数是 `disk=1207864 net=0`。 / The block index is now
  flushed every 8 MiB instead of only at connection close, so already-cached ranges are served from disk.


## v1.7.14

* **首页直播板块解锁 / Home live channel unlocked**: 国际版服务端按身份把顶栏「直播」那一格裁掉了，
  模块在「标题」与「页面」共用的那个收口处把它补回去，追加在末尾所以既有页的下标一个都不动，
  点「直播」进的就是直播流 / The server drops the live tab from the home top bar for the intl
  identity; the module re-adds it at the single funnel that feeds both the titles and the pager
  pages, appended last so existing page indices stay untouched.
* **点头像进「我的」修好 / Avatar now opens the Mine page**: 6.6.0 起底栏点击不走旧动作总线，
  模块改为直接投宿主自己那发「按深链切底栏页」的动作（匹配键从宿主下发的那格数据里现读，
  6.3.0–6.6.0 四版候选都在 apk 里逐个对读过），此前头像点开落到「动态」的那次几何错位一并修掉 /
  On 6.6.0 the bottom-bar click no longer goes through the old action bus, so the module now
  dispatches the host's own route-to-tab action; the earlier mis-tap that landed on the dynamic
  feed is fixed too.
* **评论区/主页 IP 属地 6.6.0 复核 + 自检日志改用真键名 / IP location re-verified on 6.6.0**:
  功能开/关各跑一轮实机对照（关闭时两个不同视频的属地标签都消失，开启时评论区与空间页正常显示属地），
  启动自检行现在打印 conf 的真实键名（`ip_location_enabled=` 而不是 `ip=`）/ On-device A/B on 6.6.0
  confirms the rewrite chain still delivers `location`, and the startup self-check line now prints
  the real conf key names so the log can't be copied into a conf file as a wrong key.


## v1.7.13

> 本版含未单独发布的 v1.7.12；1.7.11 及以下直接覆盖安装。
> Supersedes the never-published v1.7.12 — install directly over 1.7.11 or below.

* **适配国际版 6.6.0（9130300）/ Adapted to host 6.6.0**: 混淆锚点整族换名，每个落点都按「形状+角色」
  在 dex 里重定位、装机一行日志验收：fnval `kJ1.a`→`aK1.a`、听模式完成监听 `CE1.f`→`lK1.j`、
  feed 解析入口 `request.g`→`request.h`、直播后台门 `HX.c$b.q1()`→`JX.b$b.m1()`、
  gRPC 身份描述符族 `kr1.*`→`xr1.*`。旧名多数已被 R8 复用成无关类，故候选一律过形状校验；
  同时删掉 6.5.0 起就已断死的三条身份兜底链（`ip1.h` 标记、`mq0.a/oq0.a`、`kr1.a/up1.a`）。
  / Anchors renamed as a whole family on 6.6.0; each re-located by shape+role and confirmed on
  device, dead legacy fallbacks removed.
* **多 CDN 加速的两处修复 / Accelerator: hole-in-stream and frozen-picture fixes**: 节点把**中段**
  子块截短时，旧代码仍跳到「计划」的下一个起点，响应体留下空洞（真机：播到中段花屏，拖一下进度条才好）；
  起播后某子块彻底失败时直接向上抛异常，而代理开口后抛异常等于把流砍尾（画面定格、进度条照走）。
  两处现在都「交付到事实边界，再从断点单连接续传到末尾」；候选全被退避挡光时兜底通道退回地址全集。
  桌面回归先复现两处错再修好，967 项断言全过。
  / A truncated middle piece used to leave a gap (mid-video artifacts) and a failed piece cut the
  stream off; both now resume from the real boundary on one connection.
* **首页不自动刷新拦对类型 / Home no-auto-refresh now vetoes the right flush**: 6.6.0 真正会发的自动
  刷新是 `FLUSH_ON_BACK_PRESS`（停在首页 tab 按返回键 → 整屏换推荐流），旧名单只拦 `AUTO_BACK_*`
  等于没拦。真机同一动作两侧各跑一次：关 → 首页 5 条标题全换（0 重合）；开 → `auto refresh blocked`
  且 5/5 与基线一致；手动下拉不受影响。名单维持显式四条（宿主的 `isUserRequest()` 会把双击 tab、
  底部刷新按钮误判成非用户请求）。机制与取证订正见 PITFALLS #35。
  / On 6.6.0 the flush that actually fires is `FLUSH_ON_BACK_PRESS`; verified on device by an A/B
  on the same gesture (off: feed fully replaced — on: blocked, titles identical).

