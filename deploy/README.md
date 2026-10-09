# deploy —— Linux 无头服务器部署文件

配合根目录 `README.md` 的「四、Linux 源码运行 (无头服务器)」使用。

## 文件说明

| 文件 | 部署位置 | 作用 |
|---|---|---|
| `autovisor-daily` | `/usr/local/bin/autovisor-daily` | 每日刷课调度脚本：逐课程独立进程调用，各自独立时限，统计验证码命中 |
| `autovisor-daily.service` | `/etc/systemd/system/` | oneshot 服务单元 |
| `autovisor-daily.timer` | `/etc/systemd/system/` | 每天 09:00 触发，含 0-30 分钟随机延迟，`Persistent=true` 支持错过后补跑 |

## 安装

```bash
install -m 755 autovisor-daily /usr/local/bin/autovisor-daily
install -m 644 autovisor-daily.service autovisor-daily.timer /etc/systemd/system/

systemctl daemon-reload
systemctl enable --now autovisor-daily.timer
```

## 环境变量

| 变量 | 默认 | 说明 |
|---|---|---|
| `AV_MINUTES` | 26 | 每门课的时限预算（分钟） |
| `AV_GAP` | 60 | 课程之间的间隔（秒） |
| `AV_DRY_RUN` | — | 设为 `1` 时只打印计划，不实际执行 |

临时改预算（只影响这一次）：

```bash
sudo AV_MINUTES=30 /usr/local/bin/autovisor-daily
```

永久改预算：编辑 `autovisor-daily.service` 的 `Environment=AV_MINUTES`，然后
`systemctl daemon-reload`。

## 为什么逐课程独立进程

`main()` 在某门课返回 `FAILED` 时会 `break` 整轮，后面所有课程都不执行。
拆成独立进程可保证「一门课卡住不拖垮其他课」，同时让每门课拿到独立的时限预算。

## 日志

- 汇总日志：`/opt/autovisor/logs/daily-YYYY-MM-DD.log`（每门课的完整输出也追加于此）
- 单次运行详细日志：`/opt/autovisor/logs/LogN.txt`

```bash
# 看今天
tail -50 /opt/autovisor/logs/daily-$(date +%F).log

# 看 systemd 层面
journalctl -u autovisor-daily -n 50 --no-pager
```

## 注意

- 脚本从 `/opt/autovisor/config.ini` 的 `[course-url]` 段提取课程 URL，并为每门课
  派生独立配置到 `/opt/autovisor/run/course-N.ini`（仅覆盖 `limitMaxTime`）。
  **课程有变动时只需改主 `config.ini`。**
- 与其它刷课工具（如 fuckZHS）**不要同时运行**：同一出口 IP 上并行两个自动化工具
  会让风控特征叠加，且会使「是否撞验证码」的观测结果无法归因。
