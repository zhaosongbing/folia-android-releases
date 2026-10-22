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

### 0.7.21.19

Folia for Android 0.7.21.19

本版七处改动：四颗浮层按钮改成可拖动摆放、播放页返回与主页真正一致、bravais 集合页信息条位置、播放时常亮、两颗按钮的显隐开关，以及双击图标的第四次修正。

## 一、四颗按钮改为可拖动摆放

设置路径：主页与歌单 → Bravais 墙面 →「浮层位置」→ 打开「拖动调整位置」。

开启后，左上角返回、右下角定位、右下角工具、思索按钮外面各出现一个虚线框：

| 操作 | 效果 |
| --- | --- |
| 拖框体 | 改位置 |
| 拖右下角的小圆点 | 改大小（80%–180%） |

按钮可以被拖到屏幕内任意位置，但拖不出去——靠近边缘会自动停住。下面的滑块与拖动改的是同一组值，可以先用拖动摆个大概，再用滑块微调。

实现上用了覆盖层而不是给每个按钮包一层容器：这四颗按钮分散在四个组件里，定位方式各不相同（absolute、relative、flex item），包容器会改 DOM 结构，很容易把工具按钮的弹性布局或信息条的裁切弄坏。覆盖层只读位置、不改结构，关掉开关就整体消失。

播放页那颗返回与主页那颗共用同一个偏移值和同一套 CSS 规则，拖任意一个，另一个跟着动。

## 二、播放页返回与主页一致（此前的修正不完整）

上一版给播放页那颗算了独立的 top/left/width，而主页那颗走的是 CSS 变量加媒体查询。桌面下两者差几像素看不出，窄屏下主页的基准是 14/16、播放页仍写死 20/24，于是对不齐。

现在播放页那颗改用 `.folia-player-back` 规则，与墙上那颗 `.lattice-back` 完全同一套基准、同一组 CSS 变量、同一条窄屏规则。规则定义在 `WallBackButton.css` 一处，两边都读它。

## 三、bravais 集合页信息条位置（此前只改了首页）

上一版的位移是 inline style，只加在首页缝和集合页的 `FullSeam` 上，换到海报页、书脊就没反应；而且 inline transform 与逐帧脚本写 style 的节点重叠，有被覆盖的风险。

现在改成一处 CSS 变量加一组选择器：`BravaisSeam` 把 `--seam-head-x/y`、`--seam-dock-x/y` 写在 `.bravais-seam-content` 上，`bravais.css` 里的规则对所有变体生效。上部分（标题、面包屑）和下部分（工具格、底部动作排）各一组，互不干扰。

## 四、播放时保持屏幕常亮

设置路径：播放页 → 播放页界面 →「播放时保持屏幕常亮」，默认关。

用 Screen Wake Lock API，而不是给原生窗口加 `FLAG_KEEP_SCREEN_ON`：后者一旦设置，页面隐藏、暂停播放都不会自动失效；Wake Lock 的 sentinel 在页面切到后台时由系统释放，语义正好是「只在播放中钉住，暂停或退到后台就交还系统」。切回前台时会重新获取。

系统不支持或被拒绝时（省电模式、低电量）静默失败，不影响播放。

## 五、两颗按钮的显隐开关

设置路径：主页与歌单 → Bravais 墙面 →「浮层大小」组末尾。

| 开关 | 作用 |
| --- | --- |
| 显示定位按钮 | 右下角那颗定位当前播放歌曲的按钮 |
| 显示工具按钮 | 右下角那颗打开工具面板的按钮 |

关掉后按钮不再渲染，但位置与大小的设置仍然保留，重新打开时回到原处。工具面板本身还能通过命令面板和手势打开。

## 六、双击反馈图标（第四次修正）

你确认要的是「预告动作」语义，而这正是前三版实现的方向，所以问题在读取的信号上。前三版分别用了：

- store 的 hook 值 —— 是上次渲染的快照，切换完那一帧还是旧的；
- 页面上两个 audio deck 的 DOM —— **这一版错得最关键**：automix 为了无缝切歌会预热另一个 deck，那个 deck 的 `paused` 是 `false`（音量为 0 但确实在播），于是「有任一 deck 在播就算在播」把暂停读成了播放；
- store 的 `getState()` 快照 —— 没有闭包问题，但仍是应用层推断，不是物理事实。

这一版读 `audioRef.current`。这个 ref 只指向**当前在听的那个 deck**，预热那个根本不在它里面，所以不存在误判；ref 拿不到时才退回 store。

## 七、设置弹窗的滑动指示线

开关在「通用」页，名为「显示分组滑动指示线」，默认开。这一版没有改它的实现——上一版的链路（store → SettingsModal → PageHeader → Tabs）是完整的。如果你在 0.7.21.19 上发现它仍然关不掉，请确认装的是不是 0.7.21.19，并告诉我开关的确切位置，我按那个路径再查。

## 改动文件

| 文件 | 改动 |
| --- | --- |
| `src/components/wall/ChromeEditOverlay.tsx` | 新增：可拖动的虚线框与缩放手柄 |
| `src/components/visualizer/VisualizerShell.tsx` | 返回按钮改共用 CSS 规则 |
| `src/components/wall/WallBackButton.css` | 新增 `.folia-player-back` |
| `src/library/suites/bravais/bravais.css` | 信息条上下部分的位置规则 |
| `src/library/suites/bravais/BravaisSeam.tsx` | 写入信息条位置 CSS 变量 |
| `src/services/screenWakeLock.ts` | 新增：Wake Lock 封装 |
| `src/App.tsx` 双击 | 改读 `audioRef.current` |
| `src/stores/useWallChromeOffsetStore.ts` | 编辑模式、两个显隐开关、位移上限 |
| `src/stores/usePlayerChromeSettingsStore.ts` | 常亮开关 |
| `src/library/suites/bravais/BravaisStageChrome.tsx` | 按开关渲染两颗按钮 |
| `src/components/modal/settings/pages/{BravaisWallGroup,PlayerPage}.tsx` | 拖动开关、两个显隐开关、常亮开关 |
| `src/i18n/locales/{zh-CN,en,in}.ts` | 三语文案 |

单测：58 passed。类型检查：0 error。

### 0.7.21.18

Folia for Android 0.7.21.18

本次修三处：双击反馈图标（第三次修正）、返回键改为先回主页、信息条下部分可调位置；同时把四颗浮层按钮的大小也做成了可自定义。

## 一、双击反馈图标

前两次的修改分别用了「audio 元素的 paused」和「store 的 hook 值」，都不对。这次先说明错在哪：

- **读 audio 元素会反**：automix 为了无缝切歌会预热另一个 deck，那个 deck 的 `paused` 是 `false`（音量为 0，但确实在播）。「有任一 deck 在播就算在播」这个判据，会在你暂停的那一刻把状态读成「还在播放」。
- **读 store 的 hook 值也会反**：`showGestureCue` 是个 `useCallback`，读 hook 值等于读「上一次渲染时的状态」。播放/暂停刚切换完的那一帧，它手里还是旧值。

现在的做法是 `usePlaybackStore.getState().playerState === PlayerState.PLAYING`：同步读、没有闭包过期问题，也不受预热 deck 干扰。`playerState` 由 audio 的 `play` / `pause` 事件直接维护，是应用层唯一的播放语义。

语义仍是「即将变成什么」：播放时双击显示暂停图标，暂停时双击显示播放图标。

## 二、返回键改为先回主页

上一版修好了「返回键完全没反应」，但动作是直接退到桌面。现在按层级：

1. 退出确认框开着 → 关它
2. 更新弹窗 / 设置弹窗 / 命令面板 / 播放列表面板 → 逐个关
3. 其它自绘弹窗 → 派发 Escape
4. **播放页 → 回主页**
5. **bravais 墙的非根层（集合页、表单、面板）→ 退回上一层**
6. 都没有 → 按设置决定是否二次确认后才退桌面

第 5 条是直接触发缝里那颗 `‹` 返回按钮：bravais 没有把「返回」暴露成命令，那是唯一的入口，而程序触发 `click()` 不受 `visibility` 影响，缝收起时同样有效。

## 三、信息条下部分可调位置

新增「信息条下部分 · 左右 / 上下」两条滑块，与原来的「上部分」各自独立。主页作用于工具格（`.bravais-seam-dock`），bravais 集合页作用于底部那一排动作（`.bravais-seam-actions`）——两处用同一组值，切页面不会丢。

## 四、四颗浮层按钮的大小可调

设置路径：主页与歌单 → Bravais 墙面 →「浮层大小」。

| 可调项 | 基准 |
| --- | --- |
| 左上角返回按钮 | 40px |
| 右下角定位按钮 | 40px |
| 右下角工具按钮 | 48px |
| 思索按钮（触屏灯泡） | 48px |

用百分比（80%–180%）而不是绝对值，因为四颗的基准不同，绝对值会让四组滑块的刻度含义不一致。组内第一行有「全部复位」。

工具按钮那颗的缩放是整套的：滑轨长度、填充宽度、图标都按同一个比例走，不会出现「圆变大了但滑轨还是老长度」。思索按钮的热区也跟着按钮的新位置和尺寸重新计算——否则位置一改，按钮就挪出了自己的热区，收起后再也点不回来。

左上角返回按钮的位置改成了 CSS 变量（`--wall-back-x/y/scale`）而不是内联 `left/top`，这样窄屏的媒体查询基准不会被内联值覆盖。播放页那颗返回按钮现在读同一组值，三处天然一致。

## 五、设置弹窗的滑动指示线可关

设置 → 通用 →「显示分组滑动指示线」，默认开。关掉后顶部分组标签下方那条跟着选中项滑的横线消失，选中态仍由文字颜色表达，不会丢状态。

## 改动文件

| 文件 | 改动 |
| --- | --- |
| `src/App.tsx` | 双击图标改同步读 playerState；返回键加播放页 / bravais 两级 |
| `src/stores/useWallChromeOffsetStore.ts` | 新增尺寸字段与 seamDock 目标 |
| `src/stores/useSettingsModalStore.ts` | 新增 showTabIndicator |
| `src/components/shared/SlideActionButton.tsx` | 支持 sizePx（容器、滑轨、填充、图标等比） |
| `src/components/wall/WallBackButton.tsx` + `.css` | 位置与大小走 CSS 变量 |
| `src/components/app/playing-song-locator/GridLocateButton.tsx` | 支持尺寸 |
| `src/components/wall/WallToolsButton.tsx` | 透传尺寸 |
| `src/components/visualizer/VisualizerShell.tsx` | 返回按钮与主页共用设置 |
| `src/components/ponder/PagePonderTouchButton.tsx` | 位置、尺寸、热区跟随 |
| `src/components/app/dialogs/AppDialogs.tsx` | 顶部提示位置 |
| `src/library/suites/bravais/BravaisSeamHomeTools.tsx` | 信息条下部分 |
| `src/library/suites/bravais/BravaisSeamContent.tsx` | 集合页上部分与下部分 |
| `src/components/modal/settings/kit/Tabs.tsx` + `PageHeader.tsx` | 指示线开关 |
| `src/components/modal/settings/pages/{BravaisWallGroup,GeneralPage}.tsx` | 位置组扩充 + 大小组 + 指示线开关 |
| `src/i18n/locales/{zh-CN,en,in}.ts` | 三语文案 |

单测：58 passed。类型检查：0 error。

## 删除

`src/utils/audiblePlayingState.ts` 及其测试已删除——那是上一版用错的方案，留着会造成混淆。

### 0.7.21.17

Folia for Android 0.7.21.17

本次改五处：双击反馈图标、去掉控制栏里放错位置的定位按钮、四处浮层位置可自定义、返回键修复。

## 一、双击播放/暂停的反馈图标

之前双击中区，反馈图标和实际动作相反。根因不在图标映射，而在状态来源：`onPause` 里有一条「程序化暂停」的分支会直接 return，不写 `setPlayerState(PAUSED)`。只要有一次暂停落到那条分支，store 就一直停在 PLAYING，下一次双击读到的就是「还在播放」，图标自然反过来。

现在改为直接读两个主音频 deck 的物理暂停态（`audio[data-folia-main-audio="1"]`）：暂停时两个 deck 都 `paused`，播放时至少有一个不是。这个信号不会滞后，也不受 store 遗留值影响。

## 二、去掉控制栏里的定位按钮

上一版把这颗准星塞进了展开态的按钮组。它不属于控制栏，已移除；定位仍由墙上原有的那颗按钮负责，且那颗现在可以单独调位置（见下文）。

## 三、四处浮层位置可自定义

设置路径：主页与歌单 → Bravais 墙面 →「浮层位置」组。

| 可调项 | 作用 |
| --- | --- |
| 左上角返回按钮 · 左右 / 上下 | 海报墙与 Lattice 共用的那颗返回 |
| 右下角定位按钮 · 左右 / 上下 | 定位当前播放歌曲的准星 |
| 右下角工具按钮 · 左右 / 上下 | 打开工具面板的那颗 |
| 信息条上部分 · 左右 / 上下 | 主页信息条的标题、账户入口与页签整块 |

范围 ±120px，步长 1，**默认 0 = 与之前完全一致**。组内第一行有「全部复位」，8 个滑块调乱了一键回到默认。

实现上刻意避开 `transform`：返回按钮的悬停缩放与隐藏动画、定位按钮的滑入位移都写在 transform 上，用 transform 做偏移会把它们盖掉。所以返回与定位改用 `left/top`，工具按钮用负 margin（它同时还要叠加底栏抬升那个 MotionValue，覆盖掉就等于把底栏避让一起废了），只有信息条上部分用 `translate`（那一块没有别的 transform）。

## 四、返回键拦截修复

上一版完全没生效，根因是 `targetSdk` 已经是 36：Android 13（API 33）起系统走预测式返回，**不再调用 `Activity.onBackPressed()`**，写在里面的拦截逻辑是死代码。按下返回键时系统直接把任务退到后台，所以表现为「弹窗不关、也不弹确认，直接回桌面」。

修法是挂两条通道：API 33+ 注册 `OnBackInvokedDispatcher` 回调，低版本仍由 `onBackPressed` 兜底，两者都走同一个处理函数，并加了防重入（连按不会叠出两个确认框）。

JS 侧也一并改了处理顺序。原来只派发一个 Escape 事件指望弹窗自己关，但更新弹窗和退出确认框都不监听 Escape，等于没反应。现在按「离用户最近的一层」逐级关闭：

1. 退出确认框开着 → 关它
2. 更新弹窗 → 关它
3. 设置弹窗 → 关它
4. 命令面板 → 关它
5. 播放列表 / 面板 → 收起
6. 其它自绘弹窗 → 派发 Escape 兜底
7. 都没开 → 按设置决定是否二次确认

顺带给更新弹窗补上 `role="dialog"` 与 `aria-modal="true"`（此前只有设置弹窗和退出确认框有，第 6 条兜底认不出它）。

## 改动文件

| 文件 | 改动 |
| --- | --- |
| `src/utils/audiblePlayingState.ts` | 新增：按 deck 物理状态判断是否在出声 |
| `src/components/app/playback/usePlayerDoubleTapGesture.ts` | 未改（反馈仍在动作之前发出） |
| `src/App.tsx` | 双击反馈改用物理状态；返回键逐级关闭重写 |
| `src/components/FloatingPlayerControls.tsx` | 移除定位按钮及相关 props |
| `src/components/app/overlays/buildAppOverlaysModel.ts` | 移除定位按钮的传值 |
| `src/stores/useWallChromeOffsetStore.ts` | 新增：四组浮层偏移 |
| `src/components/wall/WallBackButton.tsx` | 支持 left/top 偏移 |
| `src/components/wall/WallToolsButton.tsx` | 支持负 margin 偏移 |
| `src/components/app/playing-song-locator/GridLocateButton.tsx` | 支持相对定位偏移 |
| `src/library/suites/bravais/BravaisSeamHome.tsx` | 信息条上部分 translate |
| `src/components/modal/settings/pages/BravaisWallGroup.tsx` | 新增「浮层位置」组（8 滑块 + 复位） |
| `src/components/modal/AppUpdateDialog.tsx` | 补 role/aria-modal |
| `android/.../MainActivity.java` | 注册 OnBackInvokedDispatcher 回调 |
| `test/unit/audiblePlayingState.test.ts` | 新增 7 个用例 |

单测：`npx vitest run -c vitest.config.ts test/unit/`。

## 已知限制

- 浮层偏移是绝对值微调，不跟随屏宽换算；换到很宽的屏幕上可能需要重新调一次。
- 信息条上部分是整块位移，不改变内部标题与页签的相对排布。

### 0.7.21.16


### 在线更新改为优先走 Gitee

国内直连 GitHub 下载太慢，实测同一份 14.8MB 的包：

| 线路 | 速度 | 下载完约需 |
| --- | --- | --- |
| GitHub 直链 | 13.7 KB/s | 18 分钟 |
| Gitee 附件 | 1.63 MB/s | 9 秒 |

所以更新检查与下载默认改成 Gitee，GitHub 作为回退：Gitee 查询失败时才去问 GitHub，
两边的接口字段基本一致，不需要额外配置。

### 修复 Gitee 源的版本识别

Gitee 上这个仓库的 Release 标签叫 `folia`（不是版本号），原来的逻辑从标签解析版本，
会得到 0.0.0，永远判定「已是最新」。现在标签里没有数字时，改为从 APK 资产名
（`Folia-0.7.21.16-release.apk`）里提取版本号。

同时 Gitee 的 API 不返回发布页地址，改为从接口地址推导，弹窗里的「打开发布页」不会再是空链接。

### 说明

- 两个源都发布才能保证所有用户收到更新提示。这一版同时发在 GitHub 与 Gitee 上。
- Gitee 的附件下载不支持断点续传（服务端忽略 Range 请求，总是返回完整文件），
  但九秒就能下完，实际用不上续传。

### 已知限制

- 检查更新时先问 Gitee，成功就不再问 GitHub。若某次只发了 GitHub 没发 Gitee，
  装了旧版本的用户不会收到提示。发版时两边都要传。

### 0.7.21.15


### 修复

- **浅色模式下更新弹窗看不清字**：弹窗用的是设置界面那套 `--st-*` 变量，而它只在设置
  弹窗的 Provider 内定义；更新弹窗画在应用层，取不到这些值，于是浅色模式下成了浅底浅字。
  现在改用全站主题变量（`--bg-color` / `--text-primary`），两套主题都清晰。

- **双击播放/暂停的反馈图标**：让反馈和播放按钮读同一个状态源
  （`selectDisplayPlayerState`，工程约定给「对外展示」用的那个）。此前两边各读一处，
  尾奏或加载中的状态切换时会短暂分叉，出现「按钮显示在播放、反馈却说暂停」。

### 新增

- **系统返回键**：有弹窗时按返回只关弹窗，不再直接退到桌面。
  没有弹窗时按设置决定 —— 设置 → 通用 → 启动与窗口 →「按返回键时二次确认」，默认开启；
  确认框里点「回到桌面」才真的退到桌面（保留任务栈，再次进入时状态还在）。

- **主页控制栏点击**：点一下展开，展开后再点歌名进入播放页。
  播放页上再点则是收起 —— 同一颗按钮在不同页面承担各自最自然的动作。

- **定位按钮放进控制栏**：展开状态下，切歌按钮那一行的末尾多了一颗准星，
  点它定位到当前播放歌曲，不用再去墙上找。

### 说明

- 本版首次带上新的返回键处理，属于原生改动；如果某些机型返回键无响应，请反馈具体机型。

### 后续

- 「右侧工具按钮移进信息条、面板仍以浮窗打开」与「左上角返回按钮位置自定义」两项尚未完成，
  会在下一版处理。

### 0.7.21.14


### 修复

- **长按滑轨滑不动、一滑就消失**：这次找到的是浏览器手势层面的原因。播放页根节点的
  `touch-action` 是默认的 `auto`，手指横向一动，浏览器就判定为滚动手势，直接发出
  `pointercancel` 把整条指针事件流掐断 —— 滑轨刚亮就被收走，后面的 move 也再不会到达。
  现在播放页根节点设为 `touch-action: pan-y`：纵向滚动照旧，横向手势留给应用；同时按下时
  对根节点做了指针捕获，手指移到别的元素上也不会丢事件。

- **双击播放/暂停的反馈图标**：之前读的是 `audio` 元素的 `paused`，而播放按钮读的是应用
  状态，automix 双 deck 切换时两者会短暂分叉，于是出现「按钮说在播放、反馈却显示暂停」。
  现在反馈与播放按钮使用同一个状态源，两者永远同步：播放时双击显示暂停图标，
  暂停时双击显示播放图标。

### 新增

- **系统状态栏与导航栏开关**：设置 → 通用 → 启动与窗口 → 「显示系统状态栏与导航栏」。
  应用默认仍是全屏沉浸；开启后把系统的状态栏（时间、电量）和底部导航栏露出来。
  这个开关走的是新写的原生插件，切换后窗口重新获得焦点也会保持选择。

- **主页信息条标题**：顶部那行小字从「书库」改为「Folia」。设置 → 主页与歌单 → Bravais 墙面
  → 「信息条标题」可以改成任意文字（最多 24 字符），留空即用默认的 Folia。

- **在线更新改为应用内下载**：点「下载安装包」不再跳到浏览器，而是直接在应用内下载，
  按钮上显示百分比与进度条，下载完成后自动拉起系统安装器，只需在安装器上确认一次。
  下载失败会在弹窗里给出提示，可以再点一次重试。

### 说明

- 应用内安装需要系统允许「安装未知来源应用」。首次使用若被系统拦下，按提示给 Folia 授权即可。

- 应用内的更新检查、更新日志显示与仓库 README 的发布历史仍然由同一次发布同步。

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
