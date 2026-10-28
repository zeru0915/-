# 此项目为研讨课作业vibecoding实验项目，下面为项目内容简介
# 开发说明
本安卓应用为课程研讨作业：
- 开发环境：VS Code
- AI 辅助工具：Vibe Coding 插件，接入 DeepSeek API
- 工作说明：AI 负责代码初稿生成、UI 代码、基础逻辑编写；本人负责需求设计、方案确认、代码审核、bug 调试、功能测试、整合修改，完成 APK 打包验证。
# 项目主体：待办提醒（TodoReminder）

一个纯本地、无联网的 Android 待办清单应用，支持**到点提醒**：高优先级通知 + 全屏提醒页 + 闹铃/震动。

| 项目 | 值 |
| --- | --- |
| 安装包 | [`TodoReminder-debug.apk`](./TodoReminder-debug.apk)（6.62 MB） |
| 包名 | `com.example.todoreminder` |
| 版本 | 1.0（versionCode 1） |
| 支持系统 | Android 8.0 及以上（minSdk 26，targetSdk 35） |
| 签名 | debug 签名（可自装自分享，**不可上架应用商店**） |
| SHA-256 | `C57037A181C5DDB98AFC23D53BBC3E814E5E16F0F37BCA5876551AE9447DCF37` |

---

## 目录结构

```
project/
├── TodoReminder-debug.apk   可直接安装的 APK
├── TodoReminder/            Android 工程源码（Kotlin + Gradle）
│   ├── app/                 应用模块（源码、资源、单元测试）
│   ├── build.gradle.kts     根构建脚本（AGP / Kotlin 插件版本）
│   ├── local.properties     SDK 路径（指向 .toolchain）
│   └── settings.gradle.kts  模块声明与仓库配置
└── .toolchain/              本机构建工具链（JDK 17 / Gradle 8.9 / Android SDK 35，约 0.96 GB）
```

---

## 快速上手

1. **传到手机**：数据线拷贝、微信「文件传输助手」或网盘均可
2. **点击安装**：若提示「未知来源」，按提示允许该文件管理器安装应用
3. **授予权限**（关键）：
   - 首次打开点「允许」**通知权限**
   - Android 12+ 会提示**闹钟和提醒**权限，必须允许，否则无法准点提醒
4. 点右下角 **+** 新建待办，打开「到点提醒」开关，选择日期和时间，保存即可

> 部分国产 ROM（小米/华为/OPPO/vivo）还需在系统设置里给「待办提醒」打开 **自启动**、**后台运行**、**通知横幅**，否则息屏后可能收不到提醒。

---

## 功能

- 新建 / 编辑 / 删除待办，勾选完成后标题加删除线并自动置底
- 可选设置提醒时间（日期 + 时间选择器）
- 到点提醒：通知横幅 + 铃声 + 震动 + **全屏提醒页**（锁屏状态也能弹出）
- 提醒页与通知内提供「完成」「稍后 10 分钟」「关闭」
- 开机、修改时间/时区、覆盖安装后自动恢复未触发的提醒
- 深色模式适配；数据全部保存在本机 SQLite，离线可用、无账号

## 权限说明

| 权限 | 用途 |
| --- | --- |
| `POST_NOTIFICATIONS` | Android 13+ 显示提醒通知 |
| `SCHEDULE_EXACT_ALARM` / `USE_EXACT_ALARM` | Android 12+ 准点触发闹钟（Doze 下也生效） |
| `USE_FULL_SCREEN_INTENT` | 提醒时弹出全屏提醒页 |
| `RECEIVE_BOOT_COMPLETED` | 重启后恢复未触发的提醒 |
| `VIBRATE` / `WAKE_LOCK` | 震动提醒、点亮屏幕 |

---

## 技术栈

Kotlin 2.0 · AGP 8.7.3 · Gradle 8.9 · JDK 17 · compileSdk 35 / minSdk 26 · Material 3（View + XML + ViewBinding）· SQLite（`SQLiteOpenHelper`）· `AlarmManager` + `BroadcastReceiver` 精确闹钟 · 无第三方 SDK、无网络权限

### 源码结构

```
app/src/main/java/com/example/todoreminder/
├── MainActivity.kt          待办列表 / 勾选 / 删除 / 通知与闹钟权限引导
├── AddEditTaskActivity.kt   新建·编辑待办（标题、备注、日期时间）
├── ReminderActivity.kt      全屏提醒页（闹铃 + 震动，锁屏可显示）
├── data/                    Task 模型、SQLite 帮助类、TaskRepository
├── reminder/                ReminderScheduler、Notifications、ReminderReceiver、
│                            TaskActionReceiver（完成/稍后）、BootReceiver、ReminderActions
└── ui/                      TaskAdapter（RecyclerView）、DateLabels（今天/明天文案）
```

---

## 构建

本机工具链已就绪（`.toolchain/`），在 PowerShell 中执行：

```powershell
$tc = 'C:\Users\LENOVO\Desktop\project\.toolchain'
$env:JAVA_HOME = "$tc\jdk\jdk-17.0.20.1+1"
$env:ANDROID_HOME = "$tc\android-sdk"
$env:ANDROID_SDK_ROOT = "$tc\android-sdk"
$env:PATH = "$env:JAVA_HOME\bin;$env:PATH"

& "$tc\gradle\gradle-8.9\bin\gradle.bat" -p "C:\Users\LENOVO\Desktop\project\TodoReminder" assembleDebug
```

产物：`TodoReminder/app/build/outputs/apk/debug/app-debug.apk`（重新打包后记得拷贝到根目录）。

用 Android Studio 打开 `TodoReminder/` 目录也可直接 Build APK（SDK 路径见 `local.properties`）。

## 测试与校验

```powershell
# 17 个 Robolectric 单元测试（数据库、闹钟调度、界面流程）
& "$tc\gradle\gradle-8.9\bin\gradle.bat" -p "...\TodoReminder" testDebugUnitTest

# 代码检查
& "$tc\gradle\gradle-8.9\bin\gradle.bat" -p "...\TodoReminder" lintDebug

# 校验 APK 签名 / 元信息
& "$tc\android-sdk\build-tools\35.0.0\apksigner.bat" verify --print-certs ".\TodoReminder-debug.apk"
& "$tc\android-sdk\build-tools\35.0.0\aapt2.exe" dump badging ".\TodoReminder-debug.apk"
```

本次交付的验证结果：

- `assembleDebug` 构建成功
- Robolectric **17/17 通过**（`TaskRepositoryTest` 5 + `ReminderSchedulerTest` 6 + `TaskScreenTest` 6）
- `lintDebug` **0 error**
- `apksigner verify` 通过；`aapt2 dump badging` 核对包名、中文应用名、权限、启动页均正确
- ⚠️ 未在真机/模拟器上运行（构建环境为虚拟机，无嵌套虚拟化），实际观感与铃声效果需装机确认

---

## 常见问题

**到点没有响铃 / 没有弹窗？**
按顺序检查：通知权限是否允许 → 系统设置里的「闹钟和提醒」权限是否允许 → 是否被系统省电策略限制（设为「无限制」并允许自启动）→ Android 14+ 是否允许「全屏通知」。即使全屏通知被拒，仍会收到带铃声的横幅通知，点开即进入提醒页。

**「稍后 10 分钟」是怎么实现的？**
把该待办的提醒时间改为 10 分钟后并重新注册闹钟，修改会同步到待办本身。

**数据存在哪里？**
本机 SQLite（`todo.db`），不联网、不上传；卸载应用会一并清空。


研讨课作业(vibecoding的安卓端app)
