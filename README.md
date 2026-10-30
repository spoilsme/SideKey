# SideKey — 一加 / ColorOS 侧键增强模块

纯 C++ 实现的 KernelSU / Magisk / APatch 模块：把电源键、音量键变成可编程快捷键，
支持多击、长按、组合键、三段式静音开关，内置 40+ 个常用动作脚本，带 WebUI 配置界面。

- 作者：spoilsme
- 平台：arm64，一加 / OPPO / realme（ColorOS 15 / 16），其他机型理论可用（自动识别按键）
- 当前版本：**v1.7.1**
- 本仓库只提供**安装包**与说明文档，不包含源代码

## 下载

| 版本 | 安装包 | 说明 |
| --- | --- | --- |
| v1.7.1（最新） | [sidekey-v1.7.1.zip](https://github.com/spoilsme/SideKey/releases/latest) | 手电筒改用官方服务开关，控制中心不再失灵 |

到 [Releases](https://github.com/spoilsme/SideKey/releases) 页面下载最新 `sidekey-vX.Y.Z.zip`，直接刷入即可。

## 它能做什么

- **多击手势**：单击 / 双击 / 三击 / 四击 / 五击 / 长按，每个手势绑定一个动作
- **多击拦截**：双击只跑你的脚本，不会顺带加减音量（需要 uinput 支持，自动检测）
- **电源键叠加**：默认保留原功能（息屏 / 关机菜单），可选拦截
- **保留系统截图组合键**（v1.6.7 新增）：音量下 / 电源键配了拦截后，「电源键 + 音量减」截图仍可用（设置页可开关）
- **组合键**：电源键 + 音量键等组合绑定动作（可选键自动识别真机实体键，虚拟设备的假键不显示）
- **三段式静音开关**：拨到上 / 中 / 下各执行一个动作，可屏蔽系统原行为
- **多动作序列**：可视化编排「打开应用 → 等待 → 回主屏」等步骤，保存成脚本
- **锁屏自动化**：锁屏拉起微信付款码并自动解锁到密码键盘（ColorOS 实测配方）
- **WebUI**：5 个页签（按键 / 脚本 / 设置 / 日志 / 关于），改完一键保存重启

## 安装

1. 把 `sidekey-v1.7.1.zip` 刷进 KernelSU（或 Magisk / APatch），**重启**。
   - 兼容 KernelSU / KSU Next、Magisk 稳定版 / Beta / Canary、Magisk Delta / Alpha、
     APatch、狐狸面具（Fox Magisk Module Manager）等所有主流 root 管理器。
   - v1.7.0 起包内补齐了标准 `META-INF/com/google/android/update-binary`（Magisk
     官方安装脚本）：之前缺失它会导致 Magisk 系安装器报「Unzip error」刷入失败
     （KernelSU 不依赖它所以之前能装）。
2. 打开模块的 WebUI，按键页会自动识别本机的按键和三段式开关。
3. 点任意手势行挑一个动作，底部「保存并重启」生效（守护进程重启约 2 秒）。

升级 = 管理器里刷入新 zip 再重启，配置和脚本都会保留。

## 内置动作库

WebUI「脚本」页可查看全部内置脚本（只读）和试运行；自己写的脚本放
`/storage/emulated/0/Android/sidekey/scripts/`，改完即时生效不用重启。

**ColorOS 快捷**：

| 脚本 | 功能 | 说明 |
| --- | --- | --- |
| `sys_home.sh` / `sys_back.sh` | 主屏 / 返回 | |
| `sys_recents.sh` | 最近任务 | |
| `sys_screen_off.sh` | 息屏 | |
| `sys_power_menu.sh` | 电源菜单 | |
| `sys_notification.sh` | 下拉通知栏 | |
| `sys_qs.sh` | 展开快捷设置 | |
| `sys_split.sh` | 分屏 | |
| `sys_screenshot.sh` | 截图 | |
| `cos_ocr.sh` | 文字识别 | |
| `cos_screenshot_area.sh` | 区域截图 | |
| `cos_torch.sh` | 手电筒（最大亮度） | **v1.7.1**：开关完全沿用 OPlus 官方服务（判断 `flashlight_enabled` → ACTION_OPEN/CLOSE + 同步两个 settings 值，**全程不碰 `switch_*` 电源闸**），点亮后再把 `led:torch_*` 拉到 `max_brightness` 得到满功率。实测：开灯约 1.7 秒（其中 1 秒是官方服务点亮硬件的固有延迟）、关灯约 0.5 秒，**控制中心手电筒开关始终正常**。若哪天失灵，WebUI「设置」页点【修复手电筒】（或 `TORCH_REPAIR=1 cos_torch.sh`）约 10 秒恢复，不用重启手机。日志 `torch.log` |
| `cos_torch_sys.sh` | 手电筒（系统亮度·原版） | 走 OPlus 官方手势服务，与相机 HAL 完全兼容 |
| `cos_bypass.sh` | 游戏免打扰 | |
| `cos_nfc.sh` | NFC 开关 | |
| `cos_ringer.sh` | 铃声 响铃→震动→静音 | 读当前模式 → 切下一个 → 回读确认；日志 `ringer.log` |

**媒体**：`media_play.sh`（播放 / 暂停）、`media_next.sh` / `media_prev.sh`（切曲）、
`slider_volume_up.sh` / `slider_volume_down.sh`（音量加减）。

**微信 / 支付宝**：

| 脚本 | 功能 | 说明 |
| --- | --- | --- |
| `wx_scan.sh` | 微信扫一扫 | |
| `wx_pay.sh` | 微信付款码（亮屏版） | 直接拉起付款码界面 |
| `wx_pay_lock.sh` | 付款码（锁屏自动解锁） | 亮屏 → 菜单键抬起锁屏 → 底部上滑出密码键盘（ColorOS 实测配方）；conf 写 `chain=1` 可自动输密码（v1.6.9 全程约 3 秒） |
| `wx_password.sh` | 自动输入密码 | 自动定位键盘逐位点按；v1.6.9 起键盘坐标缓存（首次校准后免抓屏）、密码加密存储。`WX_KEYPAD_REFRESH=1` 强制重新校准 |
| `ali_scan.sh` / `ali_pay.sh` | 支付宝扫一扫 / 付款码 | |
| `ali_trip.sh` | 支付宝出行码 | |

**已移除**：`slider_torch.sh`、`tool_flashlight_toggle.sh`、`wx_trip.sh`
（微信已拦截外部跳小程序的 scheme）。

## 锁屏密码自动输入（WebUI 配置）

「设置」页 →「锁屏密码自动输入」卡：填密码数字、可选打开「解锁后自动输」，
点保存立即生效。

**隐私与安全（v1.6.9 起）：**

- 密码**加密存储**：conf 里只有密文 `enc=`，采用一次一密（每次保存重新生成随机密钥）；
- 密钥单独放在 `/data/adb/sidekey/.pkey`（权限 600，普通应用无法读取）；
- 密码**仅保存在本机**，模块**不联网、不上传、不收集**任何密码信息，卸载模块即全部清除。

## 故障排查

- 守护进程没起来：看 WebUI「日志」页 → daemon.out 和 sidekey.log
- 手电筒点不亮（控制中心）：WebUI「设置」页点【修复手电筒】（= `TORCH_REPAIR=1 cos_torch.sh`），约 10 秒恢复，**不用重启手机**
- 铃声切换不对：看 `ringer.log`，把日志发到 Issues
- 多击拦截不生效：设置页「按键扫描」确认设备和 uinput 状态
- 刷入失败：确认 root 管理器版本，附上安装日志提到 Issues

## 目录说明（安装后）

```
/data/adb/modules/sidekey/            模块本体（勿手改）
/data/adb/sidekey/                    运行数据：日志、密钥
/storage/emulated/0/Android/sidekey/  用户脚本（改完即时生效）
```

## 反馈

问题与建议请开 [Issues](https://github.com/spoilsme/SideKey/issues)，附机型、系统版本、复现步骤和相关日志。

## 许可

仅供学习交流，禁止倒卖。使用本模块产生的任何后果由使用者自行承担。
