# 重科小助手（校园小助手）

一款面向学生的 Android 校园综合服务工具，基于 Material Design 3 设计，全部数据本地存储，无广告、无统计 SDK、隐私优先。

- 包名：`com.tool.campusassistant`
- 当前版本：`2.0.12.d`（versionCode 57）
- 运行要求：Android 8.0（API 26）及以上，仅支持 arm64-v8a 设备

## 功能特性

### 校园服务
- **课程表**：课程增删改查、周视图 / 日视图、学期设置、OCR 截图导入（可编辑预览后入库）、上课提醒（AlarmManager 精确定时，开机自动恢复）、系统日历同步
- **校园地图**：高德地图定位、地点标记（支持照片附件）、地点搜索、首页地图速览卡片
- **校园网直达**：校园门户 WebView 入口

### 信息查询
- **OCR 助手**：ML Kit 中文离线识别，支持相册图片识别与 CameraX 实时取景识别
- **取件码**：实时识别提取取件码 / 柜址，震动确认入库，到期前自动提醒
- **快递跳转**：快递公司清单，复制单号后跳转对应平台
- **每日一言**：108 条内置语录，按日期固定展示，完全离线可用

### 日常工具
- **海拔仪**：海拔、经纬度、气压实时展示
- **音乐播放器**：本地音频扫描索引、Media3 后台播放、通知栏 / 锁屏控制、MiniPlayer、系统均衡器、洛雪音源（默认关闭）
- **清风工具箱**：轻量工具聚合入口

### 个性化
- **首页磁贴**：增删、排序、单双列尺寸自定义
- **WebApp 容器**：导入本地 HTML 文件或 ZIP 目录，生成桌面磁贴
- **主题**：Material You 动态取色（Android 12+）、深浅色模式、多组预设主题色板、四种界面风格
- **桌面小组件**：课表等信息一屏速览

## 下载安装

前往对应平台的 **Releases** 页面下载最新 APK（附件名 `app-release.apk`）：

- GitHub：<https://github.com/Lee-Ja7/ExpressAssistant_ii/releases>
- Gitee：<https://gitee.com/qingfeng_studio/ExpressAssistant_ii/releases>

> 仅提供 arm64-v8a 架构，覆盖绝大多数近五年机型；2015 年前的老旧 32 位设备无法安装。

## 应用内更新

设置页「检查更新」流程：检查版本 → 自动下载 → APK 校验 → 调用系统安装器。

- **双源策略**：Gitee 为主源，失败或无有效信息时自动回退 GitHub
- 更新元数据 `update.json` 与 APK 作为**同一次 Release 的附件**发布，保证版本一致
- 下载完成后校验 APK 包名、versionCode、versionName 与文件完整性，校验失败自动删除
- Android 8.0+ 会引导用户完成「安装未知应用」授权

## 技术栈

| 分类 | 选型 |
|:---|:---|
| 语言 | Kotlin 2.4 |
| UI | Jetpack Compose + Material Design 3（Material You 动态取色） |
| 导航 | Navigation Compose（`@Serializable` 类型安全路由） |
| 依赖注入 | Koin |
| 数据库 | Room（单库多 DAO，Flow 响应式） |
| 偏好存储 | DataStore Preferences |
| 网络 | Ktor Client（OkHttp 引擎） |
| 后台任务 | WorkManager / AlarmManager |
| 地图 | 高德 3D 地图合包（定位 + 搜索） |
| OCR | ML Kit 中文文字识别（离线） |
| 相机 | CameraX |
| 音频 | Media3 ExoPlayer + MediaSession |
| 小组件 | Glance |
| 构建 | AGP 9.3 / KSP / JDK 21 |

## 架构概览

MVVM + Repository + Observer，严格单向依赖：

```
UI (Compose Screen + ViewModel)
  → Repository
    → Room / DataStore / Ktor
      → System Bridge（AlarmManager / Notification / MediaSession / AMap / CameraX）
```

- `feature → core` 允许，`core → feature` 禁止，`feature ↔ feature` 禁止直接依赖
- 跨模块协作经 `core/navigation` 路由契约或 core 层共享接口完成
- 高德、ML Kit、Media3 等第三方 SDK 仅存在于所属 feature 包内
- `ReminderObserver` / `PickupExpireObserver` 订阅数据流变更，自动重排系统闹钟

## 自行构建

环境要求：JDK 21、Android SDK 37。

1. 克隆仓库：

```bash
git clone https://github.com/Lee-Ja7/ExpressAssistant_ii.git
# 或
git clone https://gitee.com/qingfeng_studio/ExpressAssistant_ii.git
```

2. 在项目根目录创建 `local.properties`，配置 SDK 路径与高德 Key（地图功能需要，缺失时地图页会显示明确提示而非空白）：

```properties
sdk.dir=D\:\\Android-studio\\Sdk
AMAP_KEY=你的高德Key
```

3. 构建：

```bash
./gradlew assembleDebug      # 调试包
./gradlew assembleRelease    # 正式包（未配置签名，需自行签名）
./gradlew lintRelease        # 代码检查
```

> `local.properties` 已在 `.gitignore` 中，不会提交到仓库。

## 隐私说明

- 课程、地点、取件码、曲库等全部数据仅存储在本机 Room / DataStore 中
- 无广告、无统计、无埋点 SDK
- 权限最小化申请，用途如下：

| 权限 | 用途 |
|:---|:---|
| 定位 | 校园地图定位、海拔仪 |
| 相机 | OCR 实时识别取件码 |
| 通知 | 上课提醒、取件码到期提醒、音乐播放控制 |
| 精确闹钟 | 课前定时提醒 |
| 开机自启 | 重启后恢复全部提醒 |
| 读取音频文件 | 本地曲库扫描 |
| 日历（可选） | 课程表同步到系统日历 |
| 安装未知应用 | 应用内更新时调用系统安装器 |

## 参考与致谢

- [Material Design 3](https://m3.material.io/)
- 高德开放平台、Google ML Kit、AndroidX Media3
