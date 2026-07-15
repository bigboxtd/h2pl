# Host2Play 自动续期脚本

# ⭐ **觉得有用？给个 Star 支持一下！**
> 注册地址：[https://host2play.gratis](https://host2play.gratis)

自动续期 Host2Play 免费游戏服务器的 GitHub Actions 脚本。基于 Chrome 自动化，自动处理 reCAPTCHA 音频验证，支持 IP 封锁自动切换 WARP，多服务器批量续期，并通过 WxPusher（微信）和 Telegram 推送运行结果。

## ✨ 功能特性

- ✅ 多服务器支持 —— 一次配置多个续期链接，顺序执行
- ✅ 全自动 reCAPTCHA 破解 —— 音频识别 + 自动填写，无需人工干预
- ✅ 智能 IP 切换 —— 检测到 Google 封锁后自动通过 WARP 更换出口 IP
- ✅ WxPusher 通知 —— 续期完成后汇总推送一条报告到微信
- ✅ Telegram 通知 —— 续期成功 / 失败均推送消息，附带页面截图（可选）
- ✅ 定时 + 手动运行 —— 默认每天北京时间 9:30 自动执行，也支持手动触发
- ✅ 自动清理运行记录 —— 只保留最近 2 条 Actions 记录，仓库保持清爽
- ✅ 虚拟显示无头运行 —— 使用 Xvfb，不依赖图形界面

## 📋 配置步骤

### 1. 配置续期链接（必须）

编辑仓库根目录下的 `main.py`，找到第 27 行 `RENEW_URLS` 列表，填入你的续期链接：

```python
RENEW_URLS = [
    "https://host2play.gratis/server/renew?i=你的服务器ID",
    # 多个服务器继续往下加
]
```

**如何获取链接：** 打开 [Host2Play Minecraft 面板](https://host2play.gratis/panel/minecraft)，找到你的服务器，点击 **Copy public renew link** 复制即可。

### 2. 配置 Secrets

进入仓库 `Settings` → `Secrets and variables` → `Actions`，添加以下 Secrets：

| Secret 名称 | 必填 | 说明 |
|------------|------|------|
| `H2P_SERVER_ID` | ✅ | 服务器 ID（链接中 `?i=` 后面的部分） |
| `WXPUSHER_TOKEN` | ❌ | WxPusher 应用的 AppToken |
| `WXPUSHER_UID` | ❌ | WxPusher 接收消息的用户 UID |
| `TG_BOT_TOKEN` | ❌ | Telegram Bot Token |
| `TG_CHAT_ID` | ❌ | Telegram 接收消息的 Chat ID |

> WxPusher 和 Telegram 均为可选，不配置时跳过对应推送，脚本正常运行。

### 3. WxPusher 配置说明

1. 前往 [WxPusher 开发者后台](https://wxpusher.zjiecode.com/admin/) 注册并创建应用，获取 **AppToken**。
2. 扫码关注该应用的服务号，获取你的 **UID**。
3. 将 AppToken 填入 Secret `WXPUSHER_TOKEN`，UID 填入 `WXPUSHER_UID`。

WxPusher 推送格式示例：
```
🔄 Host2Play 自动续期报告
✅ bof5006: 续期成功
   23:42:59 → 23:59:50
❌ server2: 续期失败（未通过 reCAPTCHA 验证）
```

### 4. Telegram 配置说明（可选）

- **Bot Token**：向 [@BotFather](https://t.me/BotFather) 发送 `/newbot` 创建机器人获取。
- **Chat ID**：向 [@userinfobot](https://t.me/userinfobot) 发送任意消息获取。
- Telegram 通知会附带页面截图，每个服务器单独推送一条。

## 🚀 使用方法

### 方法 1：定时自动运行（默认）

Fork 本仓库并完成配置后，工作流每天**北京时间 9:30**自动执行（对应 UTC 1:30）。

如需修改时间，编辑 `.github/workflows/Host2Play_Renew.yml` 中的 `schedule` 部分：

```yaml
schedule:
  - cron: '30 1 * * *'  # 北京时间 9:30，可按需修改
```

常用参考：
- `0 1 * * *` → 北京时间每天 9:00
- `0 */6 * * *` → 每 6 小时一次

### 方法 2：手动触发

1. 进入仓库的 `Actions` 页面
2. 选择 **Host2Play 续期** 工作流
3. 点击 **Run workflow** → 绿色 **Run workflow** 按钮

### 方法 3：API 调用

```bash
curl -X POST \
  -H "Authorization: Bearer ghp_你的Token" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/你的用户名/你的仓库名/actions/workflows/Host2Play_Renew.yml/dispatches \
  -d '{"ref":"main"}'
```

## 🐛 常见问题

### 1. WxPusher 显示"未配置，跳过推送"？

检查 Secret 名称是否与代码一致：代码读取的是 `WXPUSHER_TOKEN`（不是 `WXPUSHER_APP_TOKEN`）和 `WXPUSHER_UID`。在 GitHub Secrets 页面确认名称完全一致。

### 2. 验证码识别失败？

- **网络问题**：Google 语音识别 API 访问不稳定，脚本会自动重试（最多 3 次，每次尝试最多 50 轮）。
- **IP 被标记**：脚本检测到封锁后会自动通过 WARP 更换出口 IP 并重试。
- 建议查看 Actions 日志中的 `[INFO] 识别结果: [xxxx]` 行确认识别内容。

### 3. IP 被封锁无法继续？

脚本检测到 `try again later` 等错误提示时，会自动调用 WARP 断开重连以更换出口 IP，然后重新开始续期。每个链接最多尝试 **50 次**，可应对大多数情况。

### 4. 为什么需要 WARP？

Host2Play 续期页面嵌入了 Google reCAPTCHA，短时间内多次尝试会导致 IP 被封。WARP 能快速更换 Cloudflare 出口 IP，绕过封锁。

### 5. 截图在哪看？

每次运行结束后，进入 Actions 运行详情页，底部 **Artifacts** 区域下载 `screenshots-运行编号` 压缩包，包含每次续期的成功/失败截图。

### 6. 运行时间很长正常吗？

正常。启动 Chrome、加载页面、处理验证码、切换 IP 重试，单个链接通常需要 2～5 分钟，多次 IP 封锁可能更久。GitHub Actions 最长允许 6 小时，完全够用。

### 7. 能否同时续期多个服务器？

可以。在 `RENEW_URLS` 列表中依次填入多个链接，脚本会顺序处理，互不影响。WxPusher 最终汇总推送一条，Telegram 每个服务器单独推送一条附截图。

## 🔒 安全建议

- 所有敏感信息（Token、UID 等）请存放在 GitHub Secrets 中，不要写入代码。
- 如果 Host2Play 页面结构更新导致脚本失效，请关注仓库更新。

## 📄 许可证

MIT License

---

**⚠️ 免责声明**：本脚本仅供学习交流使用，请遵守 Host2Play 的服务条款。因使用本脚本造成的任何问题，作者不承担任何责任。
