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
