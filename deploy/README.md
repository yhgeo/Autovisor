# deploy —— Linux 无头服务器部署文件

配合根目录 `README.md` 的「四、Linux 源码运行 (无头服务器)」使用。

## 文件说明

| 文件 | 部署位置 | 作用 |
|---|---|---|
| `autovisor-daily` | `/usr/local/bin/` | 调度脚本：逐课程独立进程执行，各自独立时限 |
| `autovisor-daily.service` | `/etc/systemd/system/` | 每日刷课服务 |
| `autovisor-daily.timer` | `/etc/systemd/system/` | 每天 09:00 触发，含 0-30 分钟随机延迟 |
| `autovisor-retry.service` | `/etc/systemd/system/` | 补跑服务（只重跑未达标的课程） |
| `autovisor-retry.timer` | `/etc/systemd/system/` | 每天 13:00 / 16:30 各检查一次 |

## 安装

```bash
install -m 755 autovisor-daily /usr/local/bin/autovisor-daily
install -m 644 autovisor-daily.service autovisor-daily.timer \
               autovisor-retry.service autovisor-retry.timer \
               /etc/systemd/system/

systemctl daemon-reload
systemctl enable --now autovisor-daily.timer autovisor-retry.timer
```

## 环境变量

| 变量 | 默认 | 说明 |
|---|---|---|
| `AV_MINUTES` | 30 | 每门课的时限预算（分钟） |
| `AV_GAP` | 60 | 课程之间的间隔（秒） |
| `AV_MODE` | `daily` | `daily` 跑全部课程；`retry` 只补跑当天时长不足的课程 |
| `AV_TOLERANCE` | 1 | 判定达标时允许的收尾误差（分钟） |
| `AV_DRY_RUN` | — | 设为 `1` 时只打印计划，不实际执行 |

临时改预算（只影响这一次）：

```bash
sudo AV_MINUTES=30 /usr/local/bin/autovisor-daily
```

永久改预算：编辑 `autovisor-daily.service` 的 `Environment=AV_MINUTES`，然后
`systemctl daemon-reload`。

## 补跑机制

`autovisor-retry.timer` 每天 13:00 与 16:30 各检查一次：读取当天的汇总日志，
按课程**累加当天每次运行的实际耗时**，与 `AV_MINUTES` 比较：

- 累计时长已够（或本次已跑满时限）→ 不再执行
- 时长不足 → 只补跑缺口部分（至少 5 分钟），而不是重新跑满 30 分钟

> 之所以按实际时长而不是退出码判定：课程的全部课时学完后，程序会**提前正常退出**
> （`rc=0`），但当日学习时长可能远不足 25 分钟。只看退出码会把这种情况误判为达标。

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
- 同一时刻只运行一个实例：调度脚本已用 `flock` 保证；手动直跑时请自行确认。
