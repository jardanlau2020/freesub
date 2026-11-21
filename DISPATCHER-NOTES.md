# 调度器说明 —— 别再翻 Worker 和 workflow 源码了

**结论一句话：本仓的订阅更新由外部主机在打 `workflow_dispatch`，不在 Cloudflare，也不在 GitHub 任何仓库里。**

## 证据（2026-11-21 核）

| 项 | 数据 |
|---|---|
| 全部 run 的 event | 77 次全是 `workflow_dispatch`，`schedule` 命中 **0 次** |
| 触发间隔 | 稳定 ~7 小时 6 分（05:32 / 11:34 / 18:36 / 01:41 / 08:46 / 15:51 / 22:57 / 06:02），不是人手点的节奏 |
| 最后一次 | `2026-10-08T20:18:46Z`，之后空转 43 天 |
| Cloudflare 侧 | 账号下只有 2 个 Worker 带 cron：`nodewarden`（`*/5`，Bitwarden 备份，全文无 `api.github.com` / `dispatches` / `freesub`）、`weather-push`（`0 * * * *`）。都不是 dispatcher |
| GitHub 侧 | 全仓 code search `freesub`、`workflows/update.yml/dispatches`、`workflow run` 全部 **0 命中** |

## 排查方向

找外部主机：VPS 的 crontab、第三方 cron 服务（cron-job.org / EasyCron / UptimeRobot）、或本地机器的计划任务。查 `2026-10-08` 前后有没有被改过、停过。

## 附带结论（重要，别搞反）

fork 的 workflow 默认被标 `disabled_fork`，**手动 enable 之后自带 `schedule` 正常触发**。
同账号 `ACLClouds`（fork，`event=schedule` 49 次）、`hidencloud_renew`（fork，`event=schedule` 13 次）都是实证。

所以本仓自带的 `0 */6 * * *` 在 workflow 处于 `active` 之后是可用的，不必长期依赖外部调度器。

## 注意

`README.md` 由 `scripts/main_v2.py` 自动生成（`update.yml` 里 `git add -A output/ README.md`），本文件不会被自动提交，也不会被覆盖。