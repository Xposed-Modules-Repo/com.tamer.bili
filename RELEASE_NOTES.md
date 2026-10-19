## v1.9.3

* **官方下发的地址不再被扔掉 / The URLs Bilibili handed out are no longer discarded**: 国内档原先按
  「国内节点表」筛选 playurl 下发的地址，真机一次播放下发的 cosov（41 条流）与 akamai（12 条流）
  两条全被丢弃，候选于是 100% 变成「拿海外签名问大陆节点」，其中三个大陆镜像在本机连 DNS 记录都没有；
  现在任何模式都保留下发原址，并让它们排在换节点派生的地址之前。/ Mainland mode no longer drops the
  delivered URLs; they stay at the head of every candidate list.
* **本机连不上的节点跨进程记账，只降级不删候选 / Nodes this machine cannot reach are remembered — demoted, never deleted**:
  基线里 19 条 `out=0 disk=0 net=0` 全来自「DNS 查不到名字 / 连接被拒」这类链路侧失败，旧写法只在进程内退避，
  每次起播重新撞一遍同样的死节点（首字节 max 2962 ms）；现在同一网络里攒够两次就把它挪到队尾，账本落在宿主
  files 目录、以本机网络身份为键（被动读一次网卡，不监听、不轮询），十分钟 TTL 自愈，
  节点交付过字节立刻销账；播放器中途挂断的那几次连不上也照样记账，不再白撞。/ Two link-side failures move a
  node to the tail, keyed by the machine's own network identity so a network change invalidates the ledger on its
  own; candidates are only re-ordered, never removed.
* **两个进程记的账现在真的会相加 / The two processes now add up to the same count**: 落盘前那趟读盘合并按「谁的
  时刻新听谁的整笔」覆盖，另一个进程较小的计数会把刚写下的降级进度顶回去，两次失败因此永远凑不齐门槛；现在
  「叫不应」和「空转」各按自己最新的一笔合并。/ Each counter merges by its own timestamp, so a later process
  continues the other's count instead of overwriting it.
* **拨通了不吭声也算节点的问题 / A node that takes the call but never answers is charged too**: 首字节超时原先
  只在进程内退避，每次起播照样重新撞一遍那个不吭声的地址；现在同一网络里三次这样的空转同样把它挪到队尾。
  4xx 与 416 不进这本账——缺一个文件不该让整台机器在所有视频里被降级。/ Three zero-byte idle tries demote a node
  as well, while HTTP errors stay out of the ledger.
* **重连不再把退避记忆清零 / A reconnect no longer wipes the backoff memory**: 解析器原先按「路径＋签名」缓存，
  而播放器每重连一次就把整套 query 重签，于是几乎每次重连都新建一个解析器，刚学到的退避与封禁跟着旧的一起被
  淘汰；现在按文件路径复用同一个解析器，只把拨号用的签名换成刚下发的。/ Resolvers are cached by file path now,
  so backoff survives reconnects while the signature used for dialling is refreshed.
* **412/429 之后三十秒内每条流只走一条连接 / A 412 or 429 drops every stream to one connection for 30 s**:
  限速信号说的是这台机器的总请求速率，不是某条地址的错，按节点退避只换掉被拒的那一条、其余七路照样撞，
  一次被拒容易放大成整批重连（界面上的 0 KB）；现在降速窗内窗口收成一块、探路与对冲都暂停，只降并发、不关加速。
  / The limit is charged globally for 30 s: the window collapses to one piece and probing/hedging pause, while
  acceleration itself stays on.
* **一批稿件在国际版又能看了 / A batch of videos is watchable again**: 「视频不见了哦」不是 IP 区域封锁
  （同一手机同一网络的其它客户端能播、PC 网页也能看），而是服务端按 `mobi_app` 裁剪这批稿件的下发；现在取稿件
  与取播放信息这两处（`bilibili.app.viewunite.v1` / `bilibili.app.playerunite.v1`）沿用 IP 属地那一份身份声明，
  定域到这两个服务，不开全局档。/ The detail and play-info RPCs now carry the same domestic identity the
  IP-label feature already uses, scoped to those two services rather than switching on global mode.

## v1.9.2

* **起播卡 0 KB/s 的两条根因都收掉了 / Both causes of the 0 KB/s startup stall are fixed**: 一是代理
  同时在流的上限只有 12，而真机播放器的并行 range 连接会冲到 13，一被打满就 503 拒客，播放器并不
  优雅回落而是整批拆连重连、重连又叠在没排干的旧连接上再次越过上限，形成「0 KB、偶尔跳 2 KB、隔几秒
  才自愈」的正反馈——上限提到 64，真正的上游并发本就由下载器的并发闸把关，这里只兜底防泄漏；二是节点
  空响应的封禁在进程内永不过期，宿主主进程连跑两天会把可用节点越缩越少直到代理被饿死（只有强停重启才
  恢复），现在给每次 strike 记时间戳、按 120 秒时效惰性淘汰，长命进程能自愈。/ The stream cap of 12
  tripped the player's own reconnect loop (raised to 64), and empty-reply node bans never expired
  in-process (now a 120 s strike TTL).
* **干净卡片偶尔仍吃掉上传者名字 / Clean cards no longer drop the uploader name**: 带角标那代小卡的
  底部名字行读的是 `desc` 而不是 `args.up_name`，服务端把名字放进 up_name、把这一行让给角标时 desc
  是空的，只清角标文字会让这行既无角标也无名字；现在清完角标把 up_name 回填进 desc，名字落回它本来
  的槽位。/ The badged card's name row reads `desc`, not `args.up_name`; after clearing the badge we
  now backfill `desc` from `up_name` so the name returns.

## v1.9.1

* **干净卡片不再把上传者名字一起清掉 / Clean cards keep the uploader name**: 「推荐理由」那族数据原先
  整对象置 null，而宿主把这个对象同时当作「名字行要不要渲染」的信号，于是标签和名字一起消失；现在
  保留对象、只清里面的文字与配色，实机开启轮 6/6 张卡名字都在、屏上「××万点赞」这类标签一条不剩。
  / The reason-tag object doubled as the host's signal to draw the name row, so nulling it removed the
  name too; now only its text and colors are cleared.
* **投递失败看得见 / A failed config delivery is now visible**: 设置页原先在拉起宿主**之前**就显示
  「已保存，正在拉起」，拉起被 ROM 拦下时用户读到的是「开关不生效」；现在这一步的失败原因直接印在
  状态行和 Toast 上，并给出一条不依赖模块拉起的手动重挂路径（彻底退出 B 站再打开一次）。同一轮
  顺手删掉了首页那组早已用完的反编译探针，它每次启动打几十行、恰好把那行供人自查的 `confSrc=`
  冲出日志。/ Save no longer claims a delivery it did not complete; the reason and a manual fallback
  are shown, and the spent home-screen RE probe that drowned the self-check line is gone.
* **配置代次单调化 / Config generation is now monotonic**: 保存代次从「取当前时刻」改为
  「取当前时刻与已有最大代次 + 1 的较大者」，系统时钟回拨（换机、恢复备份、手动改时间）不会再让此后
  每一次保存都被宿主判成陈旧配置而永久丢弃。/ Generation is `max(now, last+1)`, so a clock set
  backwards can no longer make every later save look stale to the host.

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

