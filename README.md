# Simple-Status Switch大气层插件-简化状态栏
当初开发这个插件的目的纯粹是我想要一个能显示系统时间的插件，在玩游戏的时候避免过头错过时间，而在插件库里能找到的都是监控性能类的插件了，因此为了实现这个功能，决定用AI辅助自己写了一个插件。

# Simple Status 0.1.0

Tesla 常驻状态条：`6.8w 2h35m 85% 21:08`。四项分别开关，电量颜色为 >80% 绿色、20%–80% 黄色、<20% 红色。

## 安装 / 更新

1. 先退出正在运行的旧版插件。
2. 解压 `Simple-Status-0.1.0-All-in-One.zip`，把其中的 **switch** 和 **config** 文件夹复制到 TF 卡根目录，覆盖同名 `.ovl`。
3. 从 Tesla 打开 **Simple Status**，确认版本号为 **0.1.0**。
4. 调整设置后选择 **显示悬浮条 / Start HUD**。

已有四项开关设置会保留；新增外观选项自动使用默认值。不必删除旧配置。压缩包中的 source 文件夹是源码备份，不需要复制到 TF 卡。

默认 Tesla 快捷键为 **L + 十字键↓ + 右摇杆按下**；插件读取 `config/tesla/config.ini` 中的自定义快捷键。显示状态条时再按该组合返回设置。重启主机后需重新启动一次；开启其他 Tesla Overlay 前需退出本插件。

## 0.1.0 更新

- 修正中文字体覆盖：增加简体、扩展简体、繁体共享字体回退，汉字优先使用中文字体；显式指定 UTF-8 编译。
- **外观与位置 / Appearance**：背景透明度、文字透明度分别调节；字号 12–40 px；水平和垂直位置 0–100%。
- 接电充电时，剩余时间显示为绿色的预计充满时间；确认满电时显示绿色 **Full**。
- 增加 **充电说明 / Charging info**，以中英文解释充电时间含义。

## 外观设置

在“外观与位置”中，选中一项后按十字键左/右减少或增加，A 也可增加，B 返回。透明度、位置每次变化 5%，字号每次变化 1 px，修改后自动保存。

| 设置 | 含义 | 默认 |
|---|---|---|
| 背景透明度 | 0% 不透明，100% 背景完全透明 | 40% |
| 文字透明度 | 0% 不透明，100% 文字完全透明 | 0% |
| 文字大小 | 1280×720 逻辑坐标中的字号；底座模式按比例缩放 | 20 px |
| 水平位置 | 0% 最左侧，100% 最右侧 | 100% |
| 垂直位置 | 0% 顶部，100% 底部 | 0% |

上下左右移动由水平/垂直位置控制，自动限制边界避免状态条超出屏幕。渲染格式的 Alpha 为 4 位，因此透明度实际呈现约 16 个等级；菜单仍以百分比调节。

“恢复默认外观”仅恢复外观与位置，保留四项内容开关。即使文字和背景都完全透明，也能通过 Tesla 快捷键重新打开设置。

## 充电时间说明 / Charging time

**充电时，绿色时间表示预计充满电所需的剩余时间；充满电后显示绿色 Full。** 拔掉充电器后恢复显示剩余可用时长。暂停充电、净充电电流过小或数据不可用时显示绿色 `--h--m`，不显示虚假的充满时间。

**While charging, the GREEN time is the estimated time remaining until fully charged. When full, GREEN Full is displayed.** Unplugging restores the remaining battery-runtime estimate. Paused charging or unavailable data is shown as `--h--m`.

MAX17050 没有可用的 Time-To-Full 寄存器。本插件使用 `(FullCAP - RepCap) × 0.5 mAh / 平均净充电电流 × 60` 估算分钟数并平滑显示；充电不足 1 分钟时至少显示 1 分钟，直至确认满电。估算会随游戏负载、温度及充电末期电流下降而变化，并非固定倒计时。[MAX17050 芯片资料](https://www.analog.com/media/en/technical-documentation/data-sheets/MAX17047-MAX17050.pdf)

Full 的判定要求：接入电源、系统电量为 100%，并且电流接近零或系统报告已停止充电；明显放电时不显示 Full。仅开启电量保护、暂停充电但电量未满不会误显示 Full。

## 其他显示规则

- 功率是电池端功率，放电 `6.8w`、充电 `+6.8w`，不是外接电源输入功率。
- 系统时间使用 Switch 用户系统时间及其时区，格式 `HH:MM`，24 小时制。
- 数据每秒更新；状态条约 10 FPS，游戏按键独立轮询。
- 读取失败时显示对应的占位符，不继续使用陈旧读数。

配置保存在 `sdmc:/config/simple-status/config.ini`。安装包只提供模板，避免覆盖用户设置。新增字段：

```ini
background_transparency=40
text_transparency=0
font_size=20
position_x=100
position_y=0
```

以上百分比字段可在配置中以整数精调。无效内容使用默认值或限制到有效范围。若显示保存失败，请检查 TF 卡可写状态；Start HUD 会重新尝试保存，仍失败则留在设置页。

## 验证与源码

已通过：AArch64 `.ovl` 编译；16 种开关组合；充电时间与 Full 判定边界；电量颜色边界；透明度与位置边界；旧配置兼容、配置保存与恢复测试；NRO 头、元数据及 ZIP 内容检查。

**本次 0.1.0 更新已进行 Switch 实机验证。** 目Tesla + Atmosphère 1.12.1 + HOS 23.0.1。其余系统版本的中文显示、各位置/字号组合、充电完成状态以及休眠/底座切换仍需设备验证。

完整源码在包内 `source/Simple-Status-0.1.0-Source.zip`。本地构建使用 PowerShell 7：

```powershell
./scripts/test.ps1
./scripts/build.ps1
./scripts/package.ps1
```

需要项目内的 devkitA64 / newlib / switch-tools 和主机测试 GCC；下载来源在 THIRD_PARTY.md。源码遵循 GPL-2.0-or-later，第三方组件保留原许可证。

