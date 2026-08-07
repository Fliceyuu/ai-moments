# AI 朋友圈 - Android APK 打包工程

把「仿微信朋友圈 + OpenRouter AI 好友」网页应用打包成 **可安装的 Android 手机 App（APK）**。
整个构建完全在 **GitHub Actions 云端**完成，你不需要本机装任何东西。

---

## 📦 一次推送，自动出 APK

整个工程设计为「推送到 GitHub 仓库 → 云端自动编译 → 下载 APK 安装」。

### 第 1 步：创建 GitHub 仓库（如已有可直接用）
登录 GitHub → 右上角 `+` → `New repository` → 命名（如 `ai-moments`）→ 可勾选 Private → `Create repository`。

### 第 2 步：把本工程文件上传到仓库
把 `android-app/` 目录里的**所有文件**上传到仓库根目录（保持 `.github/workflows/build-apk.yml` 的相对位置不变）。
> 可用 GitHub 网页的 `Add file → Upload files` 直接拖拽上传（无需本机 git）。

### 第 3 步：构建 APK
上传后会**自动触发** `Build APK` 工作流。查看方法：
- 仓库顶部 **Actions** 标签 → 看到 `Build APK` 正在运行/完成

若没自动触发，可手动触发：
- Actions → `Build APK` → 右侧 `Run workflow` → 选分支 → `Run workflow`

### 第 4 步：下载 APK
- 自动触发版本：进入 Actions 运行页 → 底部 **Artifacts** → 下载 `ai-moments-debug-apk`
- 或打一个版本标签（`v1.0`）→ Actions 运行完后会生成 **Release**，里面可直接下载 `app-debug.apk`
- 也可 `Run workflow`（手动触发）后，在运行页 Artifacts 下载

### 第 5 步：安装到手机
把下载到的 **`app-debug.apk`** 传到手机（或手机浏览器直接下）→ 点击安装。
> 允许「安装未知来源应用」。这是 debug 签名，安装没问题，可重复用后续 debug 版覆盖更新。

---

## 🔧 构建原理
- `app/src/main/assets/index.html` = 你的 AI 朋友圈应用本体（复制自 WebView `index-standalone.html`，已直接把 HTML 打进 APK）
- `MainActivity.java` = Android WebView 壳，加载 `file:///android_asset/index.html`
- `AndroidManifest.xml` = 声明 **INTERNET** 权限（调 OpenRouter API）
- `.github/workflows/build-apk.yml` = GitHub Actions 云端 JDK17 + Gradle 自动编译

### 已验证的关键点
- OpenRouter API 返回 `Access-Control-Allow-Origin: *` → WebView 内 `fetch` 直连可行
- 应用完全自包含（无外部 CDN/CSS/JS，图标用内嵌 svg）→ 适合打包、无需服务器
- WebView 开启 `domStorageEnabled` → localStorage/IndexedDB 持久化朋友圈数据

---

## 💡 使用（App 内）
1. 打开 App = 打开朋友圈界面
2. 「设置」→ 填你的 **OpenRouter API Key**（第 1 次使用）
3. 创建 AI 好友（自定义身份/导入聊天记录/自定义头像背景）
4. 发朋友圈，AI 好友会点赞、评论、回复、主动发圈

---

## ⚠️ 已知说明
- 本工程产出的是 **debug 版 APK**（可安装、可覆盖更新）。若要上架应用市场/永久签名发布，需要配置 release 正式签名（上传你自己的 keystore 到 GitHub Secrets），可后续再扩展。
- 数据存在手机 App 的 WebView 本地存储里（localStorage/IndexedDB），清 App 数据会清空朋友圈内容（可先在应用内「导出数据」备份）。
