# 木易音乐 · 更新发布通道

本仓库只用于发布木易音乐的构建产物与 `update.json`，**不包含源码**。源码在私有仓库维护。

## 客户端如何使用

- Android：读取本仓库最新 Release 的 `update.json`，按 ABI 选择 APK，流式下载并校验 SHA-256，然后调用系统安装器。
- Windows：读取 `update.json`，下载 `kind = web` 的更新包，校验后替换安装目录下的 `wwwroot` 并重启，登录态与播放历史保留。

客户端只使用公开、免认证的 GitHub API：

```
https://api.github.com/repos/yangdejiang666/muyi-music-releases/releases/latest
```

下载资产时使用 Release API 的 `assets[].url` + `Accept: application/octet-stream`，不依赖 `browser_download_url`，避免 `github.com` 在部分网络不可达。

## 版本约定

- Git tag：`v1.0.2`
- `update.json.version_code`：Android `versionCode`，必须单调递增
- APK 文件名：`muyi-android-arm64-v1.0.2.apk`、`muyi-android-x86_64-v1.0.2.apk`、`muyi-android-universal-v1.0.2.apk`
- Windows web 包：`muyi-windows-web-v1.0.2.zip`，根目录包含 `wwwroot/`
- 所有资产必须提供小写十六进制 SHA-256

## 发布流程

```powershell
# 1. 构建 APK（源码私有仓库）
flutter build apk --release

# 2. 构建电脑版（源码私有仓库）
powershell -ExecutionPolicy Bypass -File tool/build_desktop.ps1 -Version 1.0.2 -Build 3

# 3. 生成 update.json 并发布
powershell -ExecutionPolicy Bypass -File tool/make_release.ps1 `
  -Version 1.0.2 -Build 3 `
  -AndroidUniversal "dist/android/muyi-android-universal-v1.0.2.apk" `
  -DesktopWebZip "dist/desktop/木易音乐-电脑版-1.0.2-web.zip" `
  -Notes "更新说明" -Publish
```

## `update.json`

见 `update.example.json`。关键字段：

- `version_code`：客户端与本地 build 号比较，只有更大才会提示更新
- `min_supported_code`：低于该 build 号强制更新
- `force`：强制更新开关
- `platforms.android.abi`：按 ABI 选择；未知 ABI 使用 `universal_fallback`
- `platforms.windows.kind`：`web` 只替换 `wwwroot`

## 安全

- 客户端下载后必须校验 SHA-256，校验失败会删除临时文件。
- 不要在本仓库放任何 token、keystore 或私有源码。
- Android 更新依赖同一个 release 签名密钥；keystore 离线备份，绝不入库。
