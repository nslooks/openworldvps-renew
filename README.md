# 🚀 Openworld Free IPv6 VPS 自动续期脚本

基于 GitHub Actions 的 **Openworld Free IPv6 VPS** 全自动续期工具。采用 Playwright 自动化技术 + 智能 Discord OAuth 授权 + 多帧 GIF 动态验证码解析，实现无须人工干预的永久续期。

---

## 🌟 功能特性

- 🔑 **Discord OAuth 免干预登录**：利用账号的 `DISCORD_TOKEN` 向 Discord API 提交直接授权，跳过复杂的网页交互。
- 🧩 **多帧 GIF 算式验证码识别**：
  - 自动在浏览器上下文中获取 `blob:` 类型的多帧 GIF 动态验证码。
  - **拆帧 + 分区切割**：提取 GIF 的所有帧，将画面切割为左半区（数字A）、中区（运算符）、右半区（数字B）。
  - **模糊映射与跨帧投票**：清洗字符并映射误识别符号，利用跨帧概率统计得出高准确度的算式并自动计算结果。
- ⏱️ **智能天数检测**：自动解析面板当前的剩余到期天数，仅当剩余时间 `<= 5 天` 时才触发续期，避免无谓请求。
- 📢 **Telegram 结果通知**：可选配置 Telegram Bot，续期成功或失败时自动推送最新状态。
- 📸 **自动保存验证码GIF**：自动保存验证码GIF，在 GitHub Actions 中保存为 Artifacts 便于排查。

---

## 🔐 GitHub Secrets 配置说明

在 GitHub 仓库依次点击 **Settings** ➔ **Secrets and variables** ➔ **Actions** ➔ **New repository secret** 配置以下变量：

| Secret 名称 | 是否必填 | 说明 |
| :--- | :---: | :--- |
| `DISCORD_TOKEN` | **必填** | 你的 Discord 账号授权 Token（单账号场景，获取方式见下文） |
| `EMAIL` | ❌ 可选 | 账号标识邮箱，TG 通知会显示（多账号时用来区分是哪个号） |
| `TG_BOT_TOKEN` | ❌ 可选 | Telegram Bot Token（用于接收续期结果通知） |
| `TG_CHAT_ID` | ❌ 可选 | Telegram Chat ID（接收通知的用户或群组 ID） |

> 💡 **多账号场景**：登录凭据改用 `ACC1_DISCORD_TOKEN` / `ACC1_EMAIL` 这类带分支前缀的命名（分支 `acc1` 对应 `ACC1_*`），详见下方 [多账号方案](#-多账号方案5-个分支--cloudflare-worker-定时触发)。

> [!NOTE]
> 无需手动配置 VPS 地址，脚本登录后会**自动从面板检测**账号下所有 VPS 实例并逐一续期。

---

## 🛠️ GitHub Actions 部署指南

1. **Fork 本仓库** 到你自己的 GitHub 账号下。
2. **开启 Actions 权限**：在仓库的 **Actions** 标签页中点击按钮允许运行工作流。
3. **添加 Secrets**：在 **Settings ➔ Secrets and variables ➔ Actions** 中添加 `DISCORD_TOKEN`（如需 Telegram 通知，同时添加 `TG_BOT_TOKEN` 和 `TG_CHAT_ID`）。
4. **手动测试运行**：
   - 进入 **Actions** 标签页。
   - 选择左侧的 **Auto Renew Openworld VPS** 工作流。
   - 点击 **Run workflow** 按钮启动测试。
5. **定时自动运行**：可配置 Cloudflare Worker 定时触发（见下方 [多账号方案](#-多账号方案5-个分支--cloudflare-worker-定时触发)），不依赖 GitHub 自带定时器。

---

## 🔍 如何获取 Discord Token

1. 使用电脑浏览器打开 [Discord 网页版](https://discord.com/app) 并登录你的账号。
2. 按 `F12`（或 `Ctrl + Shift + I`）打开开发者工具。
3. 切换到 **网络 (Network)** 标签页。
4. 在 Discord 中点击任意频道或点击Openworld Inc.频道，触发 API 请求。
5. 在网络请求列表中点击任意 `discord.com/api/science` 开头的请求science。
6. 在右侧 **请求标头 (Request Headers)** 中找到 `Authorization` 字段，该字段对应的长字符串即为 **DISCORD_TOKEN**。

> ⚠️ **安全提示**：请妥善保管你的 Discord Token，切勿泄露给他人。

---

## 🧩 多账号方案（5 个分支 + Cloudflare Worker 定时触发）

支持 5 个账号，每个账号对应一个分支（`acc1` ~ `acc5`），脚本会根据分支名自动读取对应前缀的 Secret，互不干扰。定时触发由 Cloudflare Worker 完成，不依赖 GitHub 自带定时器。

### 1. 创建 5 个分支

分支上不需要做任何文件改动，直接从最新 `main` 拉出来即可：

```bash
git checkout main && git pull
git push origin main
git branch acc1
git branch acc2
git branch acc3
git branch acc4
git branch acc5
git push origin acc1 acc2 acc3 acc4 acc5
```

> 也可以直接在 GitHub 网页端操作：仓库页面 ➡ 分支下拉框输入 `acc1` 回车，依次创建 5 个分支。

### 2. Secrets 配置

每个账号的登录凭据按分支前缀命名（分支 `acc1` 对应 `ACC1_*`，依此类推）：

| Secret 名称 | 是否必填 | 说明 |
|---|---|---|
| ACC1_DISCORD_TOKEN ~ ACC5_DISCORD_TOKEN | ✅ 必填 | 对应账号的 Discord 授权 Token |
| ACC1_EMAIL ~ ACC5_EMAIL | ❌ 可选 | 对应账号的标识邮箱（TG 通知会显示，方便分辨是哪个号） |

共享 Secret（所有账号共用一份）：

| Secret 名称 | 说明 |
|---|---|
| TG_BOT_TOKEN / TG_CHAT_ID | TG 通知（所有账号共用） |

> Secret 名不区分大小写，统一用大写即可。
> 💡 `main` 分支兼容单账号模式：自动回退读取不带前缀的 `DISCORD_TOKEN` / `EMAIL`，多账号只走 `acc1`~`acc5` 分支。

### 3. Cloudflare Worker 定时触发

Worker 每天定时向 GitHub Actions API 发起 5 次手动触发，每次指定一个分支：

```js
// wrangler.jsonc 中的 cron 触发器（按自己服务的到期时间调整）
// { "crons": [ "0 6 */2 * *" ] }

const REPO = "nslooks/openworldvps-renew";
const BRANCHES = ["acc1", "acc2", "acc3", "acc4", "acc5"];

export default {
  async scheduled(event, env, ctx) {
    const token = env.GITHUB_TOKEN; // Worker 的 Secret 中存放的触发 token
    for (const ref of BRANCHES) {
      const res = await fetch(
        `https://api.github.com/repos/${REPO}/actions/workflows/renew-openworld.yml/dispatches`,
        {
          method: "POST",
          headers: {
            "Authorization": `Bearer ${token}`,
            "Accept": "application/vnd.github+json",
            "X-GitHub-Api-Version": "2022-11-28",
          },
          body: JSON.stringify({ ref }),
        }
      );
      console.log(ref, res.status); // 成功返回 204
    }
  },
};
```

> 触发用的 token 需要 GitHub(classic) `repo` + `workflow` 权限（或 fine-grained token 勾选该仓库 Actions 读写），放在 Worker 的 Secret 里，不要明文写死。

## ⚠️ 免责声明

- 本脚本仅供个人自动化运维及 Python 自动化学习交流使用。
- 请遵守 Openworld 平台的服务条款 (Terms of Service)，作者不对任何使用不当导致的账号问题负责。
