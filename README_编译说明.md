# 青龙客户端 iOS 版（适配青龙 2.22）编译说明

## 本次改了什么

针对青龙 2.22 的「接口已下线」（业务码 410）问题，修复了三个文件读取接口：

| 功能 | 旧接口（已下线 410） | 新接口（已改） |
|---|---|---|
| 配置内容 | `/api/configs/{name}` | `/api/configs/detail?file=` |
| 日志详情 | `/api/logs/{name}?path=` | `/api/logs/detail?file=&path=` |
| 脚本详情 | `/api/scripts/{name}?path=` | `/api/scripts/detail?file=&path=` |

同时改了响应解析：新版 detail 接口把正文放在 `data.content` 字段，不再是纯文本，已用 `contentKey: "content"` 正确提取。

改动文件：`lib/base/http/url.dart`、`lib/base/http/api.dart`、`lib/base/http/http.dart`

登录接口（`/api/user/login`、`/open/auth/token`）在 2.22 保持不变，无需修改。

## 重要前提

**这个包是未签名 IPA，必须用你自己的证书签名后才能装到手机。** 我无法在这里编译 iOS（Linux 环境，iOS 编译必须 macOS + Xcode），编译必须在 GitHub 云端完成。

## 编译步骤（GitHub 云端，免费）

### 第 1 步：建仓库并上传源码

1. 登录 GitHub，点右上角 `+` → `New repository`
2. 仓库名随便填（如 `qinglong-ios`），选 **Public**（Private 也能编，但 Actions 免费额度对 Public 更宽松），不要勾选任何初始化选项，点 `Create repository`
3. 把本压缩包**全部内容**解压出来，上传到这个仓库（网页端 drag & drop 整个文件夹即可，或建本地 git 仓库 push）

### 第 2 步：触发编译

1. 进入仓库 → `Actions` 标签页
2. 左侧找到 `Build iOS IPA`，点进去
3. 点右侧 `Run workflow` → 再点绿色 `Run workflow` 按钮
4. 等待编译（约 5–15 分钟）

### 第 3 步：下载 IPA

1. 编译完成后，点开这次的运行记录
2. 页面底部 `Artifacts` 里找到 `qinglong-ios-ipa`，下载（是个 zip）
3. 解压得到 `qinglong-2.22-adapted.ipa`

### 第 4 步：签名并安装（用你的证书）

以手机端 **全能签 / Feather / ESign** 为例：

1. 把 IPA 传到手机（AirDrop / 文件 App / 网盘）
2. 打开签名工具，导入你的 P12 证书 + 密码
3. 选择该 IPA → 签名 → 安装
4. iOS 27 首次打开前：设置 → 通用 → VPN与设备管理 → 信任你的证书；设置 → 隐私与安全性 → 开发者模式 → 开启

用 SideStore 的话：在电脑上生成配对文件后，手机端 SideStore 导入 IPA 签名即可，7 天有效期需定期刷新。

## 编译失败怎么办

- 若报 CocoaPods 相关错误：确认 workflow 里 `pod install --repo-update` 这步已执行
- 若报 Flutter 版本错误：本项目锁定 Flutter 3.3.10，勿改成更高版本（Dart 3 有破坏性变更）
- 把 Actions 里的红色错误日志复制出来，发给我，我帮你改

## 安装后验证

打开 App → 添加面板 → 填你青龙的地址和账号 → 登录后，重点试这几个之前报「接口已下线」的功能：
- 脚本管理 → 点开任意脚本看能否显示内容
- 日志 → 点开任意日志详情
- 配置管理 → 点开配置文件

能正常显示正文，说明适配成功。
