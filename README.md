##  Autovisor

**Github项目主页：**[CXRunfree/Autovisor](https://github.com/CXRunfree/Autovisor)

------
#### 2026/9/15 公告

感谢大家三年多以来的喜爱与支持~

准备读研, 本项目佛系更新

#### 2026/9/24 Autovisor-3.18.4 更新

**本次更新:**

- 新增融合共享课支持: 自动跳过 AI 随堂练习弹窗、展开折叠目录、关闭弹窗后恢复播放, 并读取课时真实学习进度 ([#158](https://github.com/CXRunfree/Autovisor/pull/158)).

**近期更新:**

- 修复课程页卡死: 已读的"学前必读"弹窗是隐藏节点, 等它可见会一直等下去, 现在超时即跳过 ([#155](https://github.com/CXRunfree/Autovisor/pull/155)).
- 课时切换、目录识别的等待也加上超时, 异常页面不再长时间卡住.
- 支持 Python 3.13.

------
#### 一、程序介绍

**项目简介:**

这是一个可无人监督的自动化程序, 基于微软的 Playwright 框架, 由 Python 和 JavaScript 编写而成. 核心原理是使用浏览器模拟用户操作.

**程序功能:**

- **支持自动登录**
- **自动通过登录滑块验证(可选)**
- **自动播放和切换下一集**
- **自动跳过弹窗和弹题**
- **自动静音、设置指定倍速**
- **自动检测暂停并自动续播**
- **支持智慧共享课、翻转课、融合共享课**
- **支持刷习惯分**
- **支持隐藏窗口后台刷课**
- 启动时自动检查更新
- 后台实时更新学习进度
- macOS 支持源码运行
- 根据当前时间自动设置背景色(白天/夜晚)
- 完成章节时提示已刷课时长
- 各种自定义配置

#### 二、发行版运行

1. 请确保系统为 Windows 10 及以上.

2. 发行版文件夹内自带 **config.ini** (可能没显示 `.ini` 后缀名), 使用 **文本编辑器** 打开.

3. 根据文件内的注释, 填写配置文件:

   - 默认启动系统自带的 Edge 浏览器;
   - 文件里的 **EXE_PATH 项** 用于自定义浏览器路径, 需要精确到**浏览器可执行文件的位置**, 可以保持默认不填;
   - 不知道浏览器的安装路径? 见下方 **五、常见问题**.
   - 确认 **保存修改** 后再退出.

   <p align="left"><img src="resources/markdown/config_detail.png" width="600" alt="config.ini 配置说明"></p>

4. 注意事项

   - **`config.ini` 配置项不需要加引号.**

   - **如果需要修改下载镜像源, 请编辑 `data/mirrors.json`**

5. 运行 **Autovisor.exe**, 会自动打开浏览器, 进入网课界面后就能自动刷课了!

   (如果未设置 **enableAutoCaptcha=True**, 则需要**手动完成**登录时的滑块验证)

**发行版目录说明:**

- `resources/`: 图片、脚本等程序资源.
- `packages/`: 自动下载的 NumPy、OpenCV 等运行时依赖.
- `data/`: 登录 Cookies 和镜像源配置.

#### 三、macOS 源码运行

需要 macOS 11 (Big Sur) 及以上 (Apple Silicon / Intel 均可) 和系统 Chrome浏览器. 先复制模板并填写配置:

```bash
cp config.macos.ini.example config.macos.ini
```

打开 `config.macos.ini`, 填入课程链接 (账号密码可留空, 用浏览器手动登录), 然后运行:

```bash
./run_macos.sh
```

首次运行会自动部署环境 (安装 `uv`、依赖和必要的浏览器), 之后直接启动. 环境或配置变化时会自动重新部署, 也可用 `./run_macos.sh --setup` 强制重建.

#### 四、Linux 源码运行 (无头服务器)

适用于没有图形界面的服务器 (Ubuntu 22.04 / 24.04 实测). 与 macOS 一样**以源码方式运行, 不需要编译**, 也不需要 PyInstaller.

> **两点前置说明**
> 1. `headless` 在代码中硬编码为 `False`, 因此**必须用 Xvfb 提供虚拟显示**; 不要改成 `headless=True` —— 自动过滑块的图像匹配 (`cv2.matchTemplate`) 与鼠标轨迹 (`page.mouse`) 都依赖真实渲染.
> 2. `enableHideWindow` 是 Windows 专属实现 (`pygetwindow`), 在 Linux 上会自动跳过并打警告, 属正常现象. 用 Xvfb 时本就无可见窗口, 无需该项.

**1) 系统依赖**

```bash
apt update && apt install -y xvfb
curl -LsSf https://astral.sh/uv/install.sh | sh
export PATH="$HOME/.local/bin:$PATH"
```

**2) 拉取代码并安装依赖**

```bash
git clone https://github.com/CXRunfree/Autovisor.git /opt/autovisor
cd /opt/autovisor

# 国内服务器建议走镜像加速
export UV_DEFAULT_INDEX=https://mirrors.aliyun.com/pypi/simple/
export PLAYWRIGHT_DOWNLOAD_HOST=https://cdn.npmmirror.com/binaries/playwright

uv sync --extra captcha                      # 含 numpy + opencv, 自动过滑块必需
uv run playwright install --with-deps chromium
```

> `--extra captcha` 不能省: `enableAutoCaptcha = True` 时程序会调用 `installer.start()` 加载 numpy/opencv, 缺少会直接报错退出.

**3) 配置 `config.ini`**

```bash
cp config.ini.example config.ini
```

| 配置项 | 值 | 说明 |
|---|---|---|
| `driver` | `chromium` | Linux 没有 Edge, 用 Playwright 自带 Chromium |
| `enableAutoCaptcha` | `True` | 开启自动过滑块; 同时决定 numpy/opencv 是否加载 |
| `enableHideWindow` | `False` | Linux 上无效, 保持默认 |
| `limitMaxTime` | 每门课时限 (分钟) | 每门课独立计时 |
| `username` / `password` | 填账号密码 | 见下 |

**登录方式**: 优先填账号密码. 实测在机房 IP 上, 浏览器填表登录可**一次通过易盾滑块**; 而直接复用其他机器 (不同出口 IP) 导出的 cookie 会被判失效并跳回登录页.

**4) 运行**

```bash
xvfb-run -a --server-args="-screen 0 1920x1080x24" \
  .venv/bin/python -u Autovisor.py --config config.ini
```

建议加 `-u`: 否则 stdout 被缓冲, 日志要等进程结束才落盘 (程序自身的 `logs/LogN.txt` 始终实时).

**5) 定时任务 (systemd)**

单课程跑失败时 `main()` 会 `break` 整轮, 因此推荐**逐课程独立进程**调用:

```bash
#!/bin/bash
set -uo pipefail
BASE=/opt/autovisor; PY="$BASE/.venv/bin/python"; MINUTES="${AV_MINUTES:-26}"
mapfile -t URLS < <(grep -E '^[[:space:]]*URL[0-9]+[[:space:]]*=' "$BASE/config.ini" \
                    | sed -E 's/^[^=]*=[[:space:]]*//' | grep -v '^[[:space:]]*$')
for i in "${!URLS[@]}"; do
    xvfb-run -a --server-args="-screen 0 1920x1080x24" \
      "$PY" Autovisor.py --config "$BASE/run/course-$((i+1)).ini"
done
```

完整的调度脚本、systemd 单元与 timer 见仓库 `deploy/` 目录.

**6) 常见坑**

| 现象 | 原因 | 处理 |
|---|---|---|
| 启动即 `NameError: name 'packages' is not defined` | `installer.start()` 非 Windows 分支引用了未定义变量; 因 macOS 模板默认 `enableAutoCaptcha=False`, 该分支长期未被触发 | 本仓库已修复 |
| 单个课时点不中, 整轮直接终止 | `run_course()` 原为 `return CourseOutcome.FAILED` | 本仓库已改为跳过并计数 |
| 课中弹出安全验证后永久卡住 | `wait_for_verify` 只做弹窗 + 死等真人 | 本仓库已接入 `slider_verify` 自动过滑块 |
| 内存吃紧 / 偶发崩溃 | Chromium 约 400 MB | 2 GB 内存可跑单实例, 不要并发多个 |

#### 五、发行版下载

- Github: [Releases · CXRunfree/Autovisor](https://github.com/CXRunfree/Autovisor/releases)
- 网盘备用: [蓝奏云 · Autovisor-for-windows](https://wwk.lanzouj.com/b05evsxif) 密码: 492l

这是已经打包好的程序, 若需要**源代码**请于 Github 项目主页下载.

#### 六、常见问题

1. 解压后没有 `Autovisor.exe` / 提示缺少依赖文件?

   - 多为杀毒软件误杀: 加入信任区后重新解压; 也请确认下载的是 Releases 里的发行包, 而不是源码.

2. 第一次启动很久没反应 / 依赖下载失败?

   - 首次会检查并下载运行时依赖 (numpy、opencv-python), 等后台日志跑完即可; 下载失败会自动换用 `data/mirrors.json` 里的下一个镜像.
   - 报 `ModuleNotFoundError` / `ImportError: numpy...` 时, 删掉 `packages/` 目录重新运行.

3. 提示 **未检测到有效网址** / 一直**等待登录完成** ?

   - `[course-url]` 要填**课程播放页**的地址 (能直接看到课程视频的页面), 不要填课程首页.
   - 登录页改版导致的等待已在最新版修复, 请先升级到最新发行版.

4. 登录时还是要手动过滑块?

   - 只有 `enableAutoCaptcha = True` 才会自动过滑块, 否则需要手动完成.

5. 只出现命令行窗口, 没有浏览器界面?

   - 命令行是程序后台, 可以查看运行状态; 浏览器加载需要时间, 后台没异常退出就不必担心. 报错通常是浏览器路径配置有误.
   - 找浏览器路径: 打开浏览器, 地址栏输入 `chrome://version`, 其中的"可执行文件目录"就是.

   <p align="left"><img src="resources/markdown/broswer_version.png" width="720" alt="Chrome 可执行文件目录"></p>

6. 看不到浏览器窗口 / 想后台刷课 / 报 `MoveWindow 无效的窗口句柄`?

   - 最小化会让浏览器暂停渲染, 进度不增加, 也会影响弹题检测. 想后台刷课请用 `enableHideWindow = True`, 程序会把窗口移出屏幕.
   - 若报窗口句柄错误, 改回 `enableHideWindow = False`.

7. 卡在加载播放页 / 不会自动跳下一课 / 新版页面刷不了课?

   - 这类基本都是**页面结构还没适配**, 不是配置问题. 目前支持: 智慧共享课、翻转课、融合共享课.

   - 智慧树新版 **AI 课程** (带 AI 角标的课程) 尚未适配, 表现就是识别不到课时、停在加载中或单个视频无限循环; 需要等待适配

     欢迎附上课程链接和日志提 issue.

8. 弹题关不掉 / 程序卡住?

   - 弹题随时可能出现, 而检测不是实时的, 所以无法完全消除; 页面长时间无响应时, 先确认浏览器没有被最小化.

9. 遇到 PPT、PDF 等没有视频的章节会卡住?

   - 目前只处理视频课时, 非视频章节需要手动切到有视频的课时.

10. 不想再弹赞赏码?

    - 感谢您对本项目的支持~ 只需要设置 `showDonateCode = False` 就好了!

------
#### 已知问题

- **长时间挂机**有概率弹出人机验证, 程序检测到后会暂停操作, 直到手动验证完成;
- 浏览器窗口若**最小化**可能导致视频播放进度不增加;
- 若出现其他异常崩溃, 请提交 issue 并附上报错信息.

#### 写在最后

觉得体验还不错? 请留下你宝贵的 Star ⭐, 并分享给更多有需要的人!

或者为项目发电支持一下~

<p align="left"><img src="resources/markdown/donate.png" width="200" alt="赞赏码"></p>

**作者的 CSDN:** [欢迎关注~](https://blog.csdn.net/Runfreeone)

**声明：本程序只可用于学习和研究计算机原理, 请于 24h 内删除所有存档！**
