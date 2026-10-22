# Folia for Android — 在线更新源

这个仓库只用来放 **APK 与更新日志**，应用内的「在线更新」就是从这里读版本号。

- 安装包：见右侧 [Releases](../../releases)
- 上游仓库：[Timingyyds/folia-android](https://github.com/Timingyyds/folia-android)
- 代码仓库：[zhaosongbing/folia-android](https://github.com/zhaosongbing/folia-android)
- 应用内的检查地址：`https://api.github.com/repos/zhaosongbing/folia-android-releases/releases/latest`

每次发布都会上传一个 `.apk` 资产；应用发现新版本会弹窗显示版本对比和更新日志，
点「下载安装包」用系统浏览器打开 APK 直链，由系统安装器接管安装。

> 这是个人移植的非官方 Android 版本，与原项目无关。

## 更新日志

### 0.7.21.13


### 修复

- **长按滑轨滑不动、一滑就消失**：长按亮起滑轨后，手指一横移滑轨就没了，左右滑也触发不了。
  根因有两个：一是长按待命期间动态改 `touch-action` 会打断正在进行的指针流；二是手指移动后
  事件目标变成滑轨下面的其他元素，`pointermove` / `pointerup` 被改派出去，判定逻辑收不到。
  现在指针在按下时就被捕获到播放页根节点上，事件不再中途丢失；同时去掉了运行中改
  `touch-action` 的做法。

- **上一首按钮改为常显**：队列第一首时按钮不再消失，而是变灰不可点，播放到第二首自动恢复。
  这样播放按钮不会因为左右按钮的增减而左右跳位。

- **设置弹窗改为自适应高度**：弹窗高度不再写死（原来是 90dvh / 桌面 86dvh），而是按当前页
  内容撑开，最高不超过 90dvh（桌面 86dvh）。内容少的分组不用再拖着一个大空壳，
  内容多的照旧在弹窗内部滚动。

### 说明

- 应用内的更新检查、弹窗与更新日志显示沿用上一版，本版把更新日志同步到仓库 README，
  网页上也能直接看到发布历史。
