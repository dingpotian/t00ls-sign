# T00ls 自动签到

自用，用于 T00ls 论坛每日签到，防止账号变僵尸号。

已稳定运行：2026-01-15 ~ 2026-10-28（286 天）。

签到脚本来源：[HanZeYu-momo/Automatic-check-in-script-for-t00ls](https://github.com/HanZeYu-momo/Automatic-check-in-script-for-t00ls)

## 功能

- 自动登录并签到
- 支持钉钉机器人通知（可选）
- 支持 GitHub Actions 定时运行

## GitHub Actions 配置

### 1. Fork 本仓库

### 2. 配置 Secrets

1. 进入你的 GitHub 仓库页面
2. 点击顶部的 `Settings` 标签
3. 左侧菜单找到 `Secrets and variables` → 点击 `Actions`
4. 点击绿色按钮 `New repository secret`
5. 依次添加以下变量：

| Secret 名称 | 必填 | 说明 |
|------------|------|------|
| `T00LS_USERNAME` | ✅ | T00ls 用户名 |
| `T00LS_PASSWORD` | ✅ | T00ls 密码 |
| `T00LS_QUESTIONID` | ❌ | 安全问题 ID，默认 `0` |
| `T00LS_ANSWER` | ❌ | 安全问题答案 |
| `DD_ACCESS_TOKEN` | ❌ | 钉钉机器人 access_token |
| `DD_SECRET` | ❌ | 钉钉机器人加签密钥 |

### 3. 配置钉钉通知（可选）

1. 打开钉钉，进入一个群（或新建一个群）
2. 点击群设置 → 智能群助手 → 添加机器人
3. 选择「自定义」机器人，设置名称
4. 安全设置选择「加签」，复制生成的 `SEC...` 开头的密钥（即 `DD_SECRET`）
5. 点击完成，复制 Webhook 地址，格式如下：
   ```
   https://oapi.dingtalk.com/robot/send?access_token=xxxxxx
   ```
   `access_token=` 后面那串即为 `DD_ACCESS_TOKEN`
6. 将这两个值添加到 GitHub Secrets

### 4. 启用 Actions

Fork 后需手动启用 Actions，进入 `Actions` 页面点击启用即可。

> **注意**：若 workflow 已被 GitHub 因仓库长期无活动而自动禁用（`disabled_inactivity`），需进入 `Actions` 页面点击 `Enable workflow` 手动恢复一次。仓库内置的 `keepalive` 任务只能**预防**自动禁用，无法在已被禁用后自行恢复（禁用状态下定时任务不再运行）。

## Fork 用户须知

1. **先启用 Actions**：Fork 出来的仓库 Actions 默认是关闭的（页面顶部会有 `Workflows aren't being run on this forked repository` 提示），必须先点 `I understand my workflows, go ahead and enable them`，否则定时任务不会运行。
2. **配好 Secrets**：至少要有 `T00LS_USERNAME` 和 `T00LS_PASSWORD`，否则签到会失败（见上文第 2 步）。`DD_*` 不配也能跑，只是收不到钉钉通知。
3. **`keepalive` 什么时候跑**：每月只有 13 号（UTC）会真正调用 GitHub API 续命，其余日期自动跳过。同步上游（`Sync fork`）本身就是一次仓库活动，同样会重置 60 天计时器，所以错过当月 13 号也不会中断。
4. **万一 `keepalive` 报权限错误（403）**：说明账号/仓库的默认 workflow 权限被设成了只读，到 `Settings` → `Actions` → `General` → `Workflow permissions` 选 `Read and write permissions` 即可。正常情况下不需要改——workflow 里已显式声明 `permissions: actions: write`，会覆盖默认值。

## 定时执行

cron 表达式配置为每天 **UTC 0:00**（北京时间 8:00）触发，可在 `.github/workflows/checkin.yml` 中修改。

> 实际启动时间会晚于这个点：GitHub 公共 runner 需要排队，实测中位延迟约 2.6 小时，近期稳定在**北京时间 13:30 左右**。日期判断用的是 UTC 日期，而延迟只有小时级、跨不过 24 小时，因此不影响每月 13 号的续命。

上面「已稳定运行」一行是**动态**的：每次签到成功后，脚本按日历天数重算该行并 commit 回仓库（提交信息为 `自动更新 README 稳定运行时间`），天数没变化时不会产生提交。起算日默认 `2026-01-15`，可用 `T00LS_START_DATE` 覆盖。

## 手动触发

在 Actions 页面可手动触发 `workflow_dispatch`。

## 免责声明

仅供学习交流，请勿滥用。

## CHANGELOG

### 2026-10-12

- 新增 `keepalive` 任务：与每日签到共用同一次定时运行，仅在每月 13 号调用 GitHub API 重新启用本 workflow，防止公共仓库因 60 天无活动导致定时任务被自动禁用（不依赖第三方 action，不产生空提交）。
- 续命成功时由 `keepalive` 单独发送一条钉钉通知，其余日期不发；13 号当天共两条通知（续命 + 签到），两者相互独立。
- README 补充 workflow 被自动禁用后的手动恢复说明。
- README 新增「Fork 用户须知」，汇总 Fork 后的启用与配置要点。
- README 顶部「已稳定运行」改为动态计算：每个签到日成功后自动按日历天数重算并提交，天数未变化时不产生提交。
- 修正 README 与 workflow 注释里「北京时间 8:00」的说法：那只是排队时间，受公共 runner 积压影响，近期实际多在**北京时间 13:30 左右**启动。

### 2026-09-18

- 更新 README 稳定运行时间。

### 2026-06-09

- 更新 README 运行时间。

### 2026-05-05

- 日志输出对 Cookie、formhash 等敏感字段做脱敏处理（PR #3）。
- 保活定时 workflow。

### 2026-01-19

- 签到脚本日志脱敏，避免敏感信息泄露。

### 2026-01-15

- 首个可用版本：自动登录 T00ls 论坛并每日签到。
- 支持钉钉机器人通知（加签 + access_token）。
- 支持 GitHub Actions 定时运行。
- 支持安全问题的登录校验（`T00LS_QUESTIONID` / `T00LS_ANSWER`）。
- 补充 README 详细配置教程。
