# BiliTamer Release notes

## v1.7.3

* **适配 6.5.0：评论区/主页 IP 属地修复 / 6.5.0 support: comment & profile IP location
  fixed**: 宿主 App 自动升级 6.5.0 后身份链混淆锚点整组漂移，评论区 IP 属地失效
  （6.3.0 的 `up1.a` 在 6.5.0 已被无关类占用，旧 hook 挂在不相关类上静默失效）。
  本版把身份改写主路径上移到真名类 `kntr.base.moss.ignet.impl.grpc.c.f`（二进制身份头
  写入存储的唯一入口，6.3.0–6.5.0 均未漂移），对 `x-bili-metadata-bin` /
  `x-bili-device-bin` 做参数替换改写；6.3.0/6.4.0 的提供者层 hook 保留为兜底并新增
  严格形状校验（无参非抽象方法 + (String, byte[]) 构造器返回形状），杜绝撞名挂错。
  实机 6.5.0 验证：评论区属地（省份标签）恢复显示，非评论请求零改写。/ After the host
  app auto-updated to 6.5.0, the obfuscated identity-chain anchors drifted again (the
  6.3.0 `up1.a` name is now held by an unrelated class, so the old hook attached to the
  wrong class and silently died). The main rewrite path now hooks the stable,
  real-named `kntr.base.moss.ignet.impl.grpc.c.f` — the single entry through which
  binary identity headers enter the request context — and swaps the
  `x-bili-metadata-bin` / `x-bili-device-bin` bytes via argument replacement. The
  6.3.0/6.4.0 provider-level hooks remain as fallbacks, now with strict shape
  validation (non-abstract no-arg method + (String, byte[]) constructor on the return
  type) so name collisions can never attach a dead hook. Verified on a 6.5.0 device:
  province tags are back in the comment section; non-comment requests stay untouched.
* **说明 / Note**: 6.5.0 评论区主服务已迁移至 `bilibili.main.community.reply.v2`，
  服务名前缀判定天然覆盖。/ The 6.5.0 comment section mainly calls
  `bilibili.main.community.reply.v2`; the service-name prefix check covers it.
* **6.5.0 全功能盘点与适配 / 6.5.0 full audit & adaptation**: 解码/音质/HDR 的 fnval
  计算类漂移（`FG1.b`→`GI1.e`→`kJ1.a`，三类同构），新候选加入后 int/long 双钩实测触发；
  听视频播完暂停的完成监听器（`RI1.l`→`CE1.f`，R8 横向合并成 (Object,int) 合成类但签名
  未变）加入候选；底栏渲染隐藏「我的」为 6.3.0/6.4.0 专属：6.5.0 容器已 lambda 化且
  存在不可靠的首帧竞态，新版上不安装该 hook（「我的」tab 保持 App 默认显示，开关其余
  行为不受影响）。6.3.0/6.4.0 的全部旧锚点保留为候选，单 APK 继续跨三版本。
  / The fnval calculator drifted (`FG1.b`→`GI1.e`→`kJ1.a`, structurally identical);
  the new candidate hooks int/long variants, verified firing on device. The
  listen-mode completion listener gained `CE1.f` as 6.5.0 candidate (R8 horizontally
  merged the old listeners into an (Object,int) synthetic class, signatures
  unchanged). The experimental bottom-bar "hide Mine" stays 6.3.0/6.4.0-only: on
  6.5.0 the container is a compose lambda with an unreliable first-frame race, so the
  hook is deliberately not installed there (the Mine tab simply stays visible as in
  the stock app). All 6.3.0/6.4.0 anchors remain in the candidate lists — one APK
  across three versions.
* 构建 / Build: versionCode 15。
