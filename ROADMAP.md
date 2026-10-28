# ROADMAP — token-tracker

本地 AI Agent Token 消耗追踪：Claude Code / Codex 的状态栏显示 + CLI Dashboard。
本文件是项目真实进度源，记录当前阶段、已完成、待办、阻塞与最近验证。`README.md` 写介绍与用法，`CLAUDE.md` 写开发约定，三者分工不混。

## 当前阶段

**2026-10-28 22:11 上游计价与兼容修复已同步并验证**：以原作者 main `990ef700451ae57308e8d214789c05ee013af5d0` 为基线，选择性同步至工作区，源码与锁文件版本号对齐 `0.5.9`；提交目标为个人 fork 的 `origin/main`，不包含 PyPI 发布或用户级工具升级。

计价器与模型显示名已对齐上游：补齐 25 个内置报价键，兼容 LiteLLM `tiered_pricing`，更新长上下文与 MiniMax highspeed 档位、DeepSeek Flash 时间切价及已收录的 2026 年节假日谷价、Gemini 按请求时间切价。Windows Kimi 状态栏命令同步上游的独立构造与短路径兼容。Codex 信息卡继续保留本地的双行纯文本、无 ANSI、无前导空行排版及已有回归测试，未改用户级 Hook 配置。

默认与 `LANG=C LC_ALL=C TERM=dumb` 完整 pytest 各 **565 passed**；Ruff、mypy（41 个源文件）、离线锁文件检查通过。Windows 路径分支以模拟测试验证，未做 Windows 真机验收；Codex TUI 的实际显示本次未人工验收。缺少逐请求明细时的基础档估算、非标准服务档和未收录年份节假日限制仍保留。

## 已完成

- **2026-10-28 22:11**：同步上游 0.5.9 的计价、模型短名与 Windows Kimi 兼容修复，保留本地 Codex 信息卡排版；新增与更新回归测试，双环境完整 pytest 各 565 passed。源码、锁文件与中英文说明同步，不包含 PyPI 发布或用户级工具升级。

**核心能力**
- 状态栏：Claude Code 三行布局（会话时长 / Cache 命中率 / Token 增量 / 重置倒计时 / Git 分支）+ Codex 官方 `status_line`；主题系统（mocha / dracula / default）；宽度自适应 + 终端尺寸实时检测
- CLI Dashboard：交互式（备用屏 + 按键 + vim 键位 + ESC 退出），启动自动检测 setup 状态并支持脚本版本更新
- 报表：`daily` / `weekly` / `monthly` / `sessions`，多 Agent 来源分组、排序功能
- 数据源：Claude / Codex 适配器，`registry` 探测可用 agent，`rate_limits` 采集
- 成本定价：litellm 在线表（7 天缓存）+ `_fallback_pricing` 离线底座，模型识别，5 小时块 + p90
- i18n：zh / en 跟随系统语言，支持 `TT_LANG` 覆盖

**工程**
- `ruff` / `pytest` / `mypy` 配置；CI；PyPI 发布（`pip install` + curl 一键安装脚本）
- 测试基线：6 个测试文件、55 个用例

**近期版本迭代**
- `0.3.7`：修复 Opus 4.8 成本估算为 0
- `0.3.8`：识别 Claude Fable 5 定价 + 未知模型显形（不再静默归零）
- `0.4.0`：包目录迁移标准 src layout（导入名 `src` → `token_tracker`）+ CI 适配新包名 + README 双语同步（**实际未发布**，并入 0.4.1 一起出）
- `0.4.1`：大批量功能与重构合并发布——`tt` 默认走 `daily`、daily 顶部三段单 Panel 概览（Last 12 months / This Month / This Week）；老用户升级自动重新引导（SETUP_VERSION 机制）；六套主题统一系统 + `tt theme` 命令 + 首次运行交互向导；Codex 伪 statusline 接入主题、两行布局；CC statusline 四行布局 + git diff + TPS + Code 行数；`tt status` 替代旧 dashboard、过去 5h 三段实时面板；`tt sessions` 取最近 N 条再按 cost 排（修「史上高 cost 霸榜」）；柱状图全窗口 + 暗色基线；activty_minutes 维度；config 文件统一 `config.json`；Codex 不再接管 `status_line`；产物全集中放 `~/.config/token-tracker`；install.sh 多安装方式阶梯（uv / pipx / venv / pip）绕开 PEP 668；i18n 完善（月份表头、d/h 单位、Active Days）；codex-auto-review 改写为 gpt-5.5 + 补 gpt-5.5 定价。完整改动见 `git log v0.4.0..HEAD`（约 100 commit）

## 已随 `0.4.1` / `0.4.2` 发布（已实现并验证）

- **`tt daily` 重构为 GitHub 风格 token 贡献热力图**（2026-06-16）：全角 ■ 深浅绿方格（5 档）+ 月份表头 + 紧凑总览 + 图例，替换原逐日表格。
  - 新增 `ui/heatmap.py`（渲染 + 自带紧凑总览）；`ui/theme.py` 加 5 档 GitHub 绿 + 分位分档；`ui/console.py` 加 `forced_color_console()` 强制 24-bit truecolor。
  - 自适应：按终端宽度决定显示周数（最多一年），`soft_wrap` 防折行；总览自渲染（不复用 dashboard 宽 header），半屏不折。
  - 会话内彩色：在 Claude Code 里用户敲 `!tt daily` 才渲染真彩色（slash command 注入、模型工具输出都会 strip 颜色，唯用户 `!` bash 模式渲染 ANSI —— 已实测确认）。
  - **原「会话内 skill 替代 + 删除终端 daily」设想撤销**：终端命令是彩色热力图的唯一载体，删了功能即失。
  - 新增 `tests/test_heatmap.py` 6 用例。
  - **daily 跟随当前会话 agent**（环境变量识别）：CC 会话只显示 CC、Codex 会话只显示 Codex；独立终端暂保持合并（待后续处理）。
  - 修复 `■` 实为 1 列宽导致的月份/方块错位（`_CELL_W=2`）；拿不到终端宽度（Claude Code `!` 非 tty）时默认显示整年。
  - 总览改紧凑卡片（`Panel` expand=False，贴合内容不撑满）+ 数据前置标签；daily 不再打 Detected 行。
- **全 CLI 配色切到 Catppuccin Mocha / Latte 双 flavor**（2026-06-18）：`ui/theme.py` 重构——`_MOCHA`/`_LATTE` 官方调色板（truecolor）映射到 `_S` 语义槽位，暗终端 Mocha、亮终端 Latte 按 `COLORFGBG` 自动选，`TT_THEME=mocha/latte` 手动覆盖（兼容旧 light/dark）；热力梯度出 `_HEAT_MOCHA`/`_HEAT_LATTE` 双版。dashboard / daily / weekly / monthly / sessions 全部统一（`panels.py`/`tables.py` 硬编码 blue → `_S.blue`），和 statusline 同源。
- **daily 概览卡片**（2026-06-18）：紧凑卡片 = 品牌行（Token Tracker 红 + agent 暗红）+ 红分割线 + `Last 12 months` 周期标题（带日期范围）+ 过去一年三行分析：橙 Tokens/Cost/Sessions/Avg-Cost/Active-Days、蓝 Peak/Current·Longest Streak、粉 Busiest/Top Model/Active Hour，与 weekly 顶部样式统一。数据筛过去一年；Active Hour = 过去一月按**北京时区（UTC+8，固定不随系统）**聚合 token、取活跃小时区间（支持跨午夜）；Current/Longest Streak = 当前连续（末尾段延续到今/昨才有效，否则归 0）·历史最长连续活跃天数（去重日期算）。配色走 Catppuccin 语义槽位、具体色值见 `theme.py`。
  - 几轮迭代后**删除** Top Project（连带 `proj_tokens`）与 Longest Session（连带 `sessions` 参数链、`_fmt_duration`/`SessionStats` import）；`Max Streak` 改为 `Current/Longest Streak`。
  - Active Days 只统计**本地日志覆盖到的活跃天**（CC `~/.claude/projects`、Codex session 文件保留有限，更早的天无数据可读），非 bug。
- **`!tt` 非 tty 宽度探测修复**（2026-06-18，已提交 73916af）：`ui/console.py` 加 `_forced_width()`，从 `_P9K_TTY`/`SSH_TTY` ioctl 取真实终端宽度（忽略 Claude Code 子进程置的占位 `COLUMNS=0`），daily 周数判定交回 Rich console。
- **`tt weekly` 重构为五区块周报**（2026-06-18）：① This Week 卡片（品牌行 Token Tracker + agent、红分割线；第二行 Tokens/Cost/Avg-Cost 橙、第三行 Sessions/Msgs/Active Days 蓝，均带环比上周；Avg/Cost=本周成本÷已过天数）② Daily Trend 每日 token 垂直柱状图（近 30 天，前三天亮、峰值标日期、底部起止）③ Weekly Trend 逐周进度条 ④ Project Trend ⑤ Model Trend（均本周口径）。四块各用主色（橙/绿/粉/蓝），标题/名称列/进度条/分割线统一该主色，进度条用同色深浅，突出项（Weekly Trend 本周 / Project·Model Trend 第一名）用亮主色、其余压暗；Weekly/Project/Model Trend 各限 8 行。
  - 数据层：`WeeklyStats`（连带 Daily/Monthly）加 `projects` 字段 + `_aggregate` 按 project 累加（同 models 机制）；新增 `test_aggregator.py::test_aggregate_fills_projects_by_token`。
  - weekly 也跟随当前会话（原仅 daily）：CC 会话只看 CC、独立终端合并；去掉 weekly 的 Detected 行。
  - daily / weekly 品牌行统一为 `Token Tracker: a + b`（冒号 + 连接、去圆点、跟随会话）。
  - 渲染走 `forced_color_console`，`!tt weekly` 非 tty 也彩色；配色沿用 Catppuccin 语义槽位。
- **statusline 标签微调**（2026-06-18）：`Context` 标签缩写为 `Ctx`（缩短状态栏宽度，`HOOK_VERSION` → 1.8，老用户升级 pip 后自动同步 `~/.claude/tt-statusline.py`）；进度条维持原样（█ 三段色填充 + ░ 空槽，期间试过 ▄/满格双色/灰底/蓝紫灰/浅蓝紫均回退）。
- **daily / weekly 署名 footer + 缩进对齐 + `tt -v`**（2026-06-18）：四个 Trend 区块（Daily/Weekly/Project/Model Trend）+ daily 热力图整面板统一左缩进 2 格（对齐卡片内容；daily grid 宽度判定预留 2，`_display_weeks` 用 `width-6`，防折行）；weekly 底部、daily 图例 `More` 右侧（空 4 格）加 dim 署名 `tt · by stormzhang`（左对齐缩进 2）；`tt -v` 输出全名 `token-tracker <版本>` + `by stormzhang · GitHub 链接`。
- **UI 去重重构**（2026-06-18）：提取 `brand_line` / `append_metric` 到 `format.py`，daily（heatmap.py）与 weekly（tables.py）品牌行 + 指标渲染共用、消除重复；渲染输出逐字不变。
- **daily/weekly 稀疏 + 窄终端自适应**（2026-06-18）：① Weekly Trend 取消本周高亮、逐周统一亮绿（本周恒在最上无需另标）；② daily 概览三行指标按终端可用宽贪心折行（`emit_metrics`，窄终端不溢出卡片）；③ Daily Trend 稀疏自适应——去前导空白天（保底 7 天）+ 中间 > 5 天连续空压成 3 列「·」gap + 标题显实际跨度；④ daily 图例署名窄终端自动另起一行、对齐 Less 列、空行隔开（避免终端硬折导致折行掉色），宽终端仍接图例右侧。
- **会话内彩色报表 hook 产品化（终端 CLI）**（2026-06-18）：`tt setup` 自动注册会话内彩色命令——CC `/tt-daily`·`/tt-weekly`（`UserPromptExpansion`）、Codex `ttdaily`·`ttweekly`（`UserPromptSubmit`），`block`+`reason` 直接渲染真彩色报表、不经模型、不污染上下文。`hooks.py` 加 `CC_REPORT_HOOK_SCRIPT`/`CODEX_REPORT_HOOK_SCRIPT` 内嵌模板（`sys.executable -m token_tracker.cli` 调用、独立 `REPORT_HOOK_VERSION`、宽度探测 -14、Windows 跳 ioctl）+ 扩展 `_setup_*`/`_unsetup_*`（特征码 `tt-report-hook` 幂等合并、不覆盖用户 hook、CC 写 commands/*.md + 合并 UserPromptExpansion 数组、Codex 末尾追加 `[[hooks.UserPromptSubmit]]` 零依赖）+ 4 个测试 + README 双语（含「仅终端」限制）。
- **统一多主题系统**（已完成 7 阶段，方案见 `~/.claude/plans/modular-chasing-pine.md`）：把现在割裂的两套配色（CLI `ui/theme.py` Catppuccin Mocha/Latte vs statusline `hooks.py` 内嵌 256-color + 写死 mocha）统一成一套主题源，主题集扩到 Catppuccin 全家（Mocha/Latte/Frappe/Macchiato）+ Nord + Dracula + default，支持预览、`tt theme` 切换、首次运行交互向导。
  - **阶段 1 ✅ 地基（不接线）**（2026-06-18）：新增 `ui/themes.py`（7 主题 × 9 基色 + 5 档热力 + `is_light`；mocha/latte 迁现值、frappe/macchiato 官方 hex、Nord/Dracula 按色相就近映射 9 槽、default 用 3-bit 兼容色名；`derive_slots` 9→17 语义槽、`_STATUSLINE_SLOTS` 10 key 子集、`theme_to_statusline_ansi` hex→truecolor/default→3-bit）+ `config.py`（`~/.config/token-tracker/theme.json` 读写 + `resolve_theme` 优先级链 `TT_THEME`>配置>`COLORFGBG`>mocha、兼容旧 light/dark）+ `tests/test_theme.py` 12 用例。官方 hex 逐字段核对（catppuccin/palette、nordtheme、dracula README）。
  - **阶段 2 ✅ `_S` 运行时化（接线 CLI）**（2026-06-18）：`ui/theme.py` 删旧 `_MOCHA`/`_LATTE`/`_is_light_theme`/`_HEAT_*`，`_S` 改成 `_SProxy` 运行时代理（`__getattr__` 取当前激活主题的 17 槽，60+ 处 `_S.token` 调用零改、KeyError→AttributeError）；加 `get_active_theme(_name)`/`set_active_theme`/`preview_theme`（上下文管理器）/`heat_greens()`（取代模块级 `HEAT_GREENS`）；`heatmap.py` 引用随之改。CLI 报表配色现在走 `config.resolve_theme()`，`TT_THEME=dracula/nord/...` 即时生效。新增 5 用例（代理跟随/未知属性/preview 还原/heat 跟随）。
  - **阶段 3 ✅ statusline 同源烘焙**（2026-06-18）：`hooks.py` 把 statusline 脚本里写死的 `THEME="mocha"`/256-color `THEMES` 换成占位符 `__STATUSLINE_THEME_COLORS__`（当前主题 truecolor）+ `__STATUSLINE_DEFAULT_COLORS__`（3-bit 兜底）；`_render_hook_script` 注入 `themes.theme_to_statusline_ansi(config.resolve_theme())`；脚本内 `C = THEME_COLORS if _supports_color() else DEFAULT_COLORS`。`HOOK_VERSION` 1.8→1.9（老用户 pip 升级后自动重烘焙）。statusline 改 truecolor 烘焙（后按主人审美把 mocha 状态栏配色调回旧观感，`_STATUSLINE_SLOTS` 与 CLI `_S` 不完全同源，见最近验证 2026-06-19）。
  - **阶段 4-5 ✅ `tt theme` 命令 + 预览**（2026-06-18）：`cli.py` 加 `theme` 子命令（自管、不触发 auto-update/agent 检测）——`show`（当前主题 + 来源 env/config/auto）、`list`（7 主题 × 8 色块 + ● 当前标记）、`set <name>`（`config.save_theme` + `set_active_theme` + 已配则 `update_hook` 重烘焙 statusline + env 覆盖警告）、`preview <name>`（`preview_theme` 渲染 CLI 语义色/三段进度条/热力阶 + statusline 示例行）、`tt theme <name>` 简写 = set。i18n 加 10 条文案、`available_cmds` 加 theme。+5 测试。
  - **阶段 6 ✅ 交互向导**（2026-06-18）：新增 `wizard.py`（`run_wizard`：欢迎 → 列主题 → Prompt 选 → 渲染所选主题预览 → `save_theme` + `setup()` 落地）；`cli.py` 首次运行钩子改为 `_should_run_wizard()`（双 tty 且非会话内才进，否则降级静默 `setup(auto=True)`）；延迟 import 破 cli↔wizard 循环。向导聚焦主题选择，agent 自动检测、彩色命令随 setup 默认装、语言跟随系统（对原 plan 四问的合理收敛，避免改 setup 签名链）。+2 测试（判定三态 + happy path）。
  - **阶段 7 ✅ 文档收尾**（2026-06-18）：README 双语加「配色主题」节（主题表 + `tt theme` 用法 + 向导 / `TT_THEME` 说明）+ 功能列表改「多主题统一配色」+ 使用表加 `tt theme`；CLAUDE.md 加「主题系统约定」节 + 项目结构表（themes/config/wizard）+ 命令行；ROADMAP 同步。
- **Codex 虚拟 model `codex-auto-review` 改写 + gpt-5.5 定价**（2026-06-22，已提交 `1ceb26a`）：`adapters/codex.py` 加 `_rewrite_virtual_model`，把 Codex stop-time auto-review gate 的虚拟 model name `codex-auto-review` 改写成背后真实模型 `gpt-5.5`，避免它在 Model Trend / sessions 等报表独占一行、同时让 cost 走真实定价。`cost.py` 补 `gpt-5.5` 内置价（input 5 / output 30，是 gpt-5 的 4 倍，必须专属、不能走系列兜底）+ `_FAMILY_FALLBACK` 加 `codex-` 前缀兜底做双保险。`tables.py` Model Trend 加 `min_pct=2` 长尾过滤、与 Project Trend 对齐。补 `_rewrite_virtual_model` / gpt-5.5 定价 4× / auto-review 兜底三组用例。
- **Codex 伪 statusline 接入主题系统**（2026-06-22）：原本配色硬编码 mocha、不跟随主题；现改为**烘焙注入**——`_render_codex_statusline_hook()` 注入 `themes.theme_to_statusline_ansi(resolve_theme())` 的 truecolor 字典，脚本内统一走 `C["..."]` 槽位（项目 `project` / 分支 `branch` / 增删 `added`·`deleted` / 进度条 `bar_ok·warn·danger` / Limit·Ctx 标签 `label` / **Total → `tokens`(橙) / Model → `total`(红)**）。Codex TUI 实测支持 truecolor，只注入一套（不像 CC statusline 还需 256 兜底）。`tt theme set` → `is_setup()` → `update_hook()` 现成链路自动重烘焙 Codex 脚本，无需新增触发。至此三套配色（CLI 报表 / CC statusline / Codex 伪 statusline）全部同源跟随主题。`STATUSLINE_HOOK_VERSION` 1.4→1.6。
- **报表柱状图全窗口显示 + 暗色基线**（2026-06-22，已提交 `b3f4e8a`）：daily Daily Trend 改固定最近 30 天全显示（去前导裁剪 + 中段空段「·」压缩），monthly Weekly Trend 改固定最近 30 个日历周（原 20、且只显示有数据周 → 现全窗口，缺失/没用过的周补 0）。无可见高度的天/周（空白、或量极小不足一格）最底行画一格更暗基线 ▁（`_darken(_S.peach,0.4)`+dim），与真实矮柱（dim peach）拉开、一眼可辨、时间轴连续。Model Trend 与 Project Trend 一致过滤占比 ≤2% 长尾项。`test_tables.py` 补 frozen `now`（固定 30 周窗口测试稳定）、mock `week` 改 monday ISO 日期对齐 `aggregate_weekly` 真实契约。
- **`tt sessions` 取数纠错 + codex 补真实 duration**（2026-06-22，已提交 `25fbfb7`）：原「全量按 cost 倒序取前 20」致史上高 cost 会话恒久霸榜、新会话和低成本 agent（codex）永远进不了榜 → 改为**先按时间取最近 N 条、再按 cost（或 `--sort`）展示**。过滤口径从活跃时长 `active_minutes` 改为会话跨度 `duration_minutes`（cc + codex 统一）。codex 每会话仅 1 条汇总 entry、原 duration 恒为 0 被全滤；`UsageEntry` 加 `session_end` 字段、`adapters/codex.py` 取末事件 timestamp 填入、`aggregate_sessions` 据此算真实跨度（claude 留 None 走多条 entry 原逻辑）。补 `session_end` 解析 + 单条 entry 真实跨度两组用例。
- **国产模型识别 + cost（七家）**（2026-06-24）：`cost.py` `_fallback_pricing` 补 30 个国产 model 内置价 + `_FAMILY_FALLBACK` 加 10 条系列前缀兜底，`format.py` `MODEL_SHORT` 配套短名，覆盖 Kimi/Moonshot、智谱 GLM、阿里 Qwen、火山 Doubao、DeepSeek、MiniMax、小米 MiMo。**定价口径**：国产按各家中国站人民币价 ÷7.1 折 USD（与 CC/Codex 同口径）；**GLM 例外**用 z.ai 国际站官方 USD（中国站含缓存完整价是 SPA、官方页抓不到）；阶梯定价模型（Qwen3-Coder / Doubao）统一取 0-32K 基础档。七家官方页 2026-06 核实并修正草案（`kimi-k2-instruct`/`turbo` 已 EOL → 改 K2.7/K2.6/K2.5；DeepSeek 进 V4、chat/reasoner 映射 v4-flash；小米 MiMo 与 DeepSeek 同价、Pro 主攻 agentic 编程）；已下线旧 id / 未来新版本走系列兜底不归零。新增 `_cny`/`_usd` 汇率 helper（汇率常量 `_CNY_PER_USD=7.1` 集中）。**另补国外**：xAI Grok 官方定价（`grok-4.3` / `grok-build-0.1` / `grok-code-fast-1`，litellm bare 零收录会归 $0；2026-05-15 退役 slug 靠系列兜底按官方路由对齐 grok-4.3 / build-0.1）+ Gemini 8 个短名（litellm 价已对、只补显示名不入兜底）。`test_cost.py` 共 +10 用例。

## 待办 / 计划

- **Codex 非标准服务档计价（待确认日志字段）**：Astra 官方 Fast 为适用标准价的 2 倍，Flex 为 0.5 倍；当前抽查的 3 个真实 Astra 会话无可核实的 `service_tier`，继续按标准价估算。后续以逐请求实际生效档位为准，不从当前用户配置或模型名推断整段历史的倍率。
- **`tt sidebar` 点击跳转补 Ghostty（未启动）**：自动分屏已支持 Ghostty（2026-08-03），但总览「点头行跳窗格」仍只有 tmux（`TMUX_PANE` 映射）/ iTerm2（`ITERM_SESSION_ID` 映射）——三套 statusline 模板只采集这两类，`ui/sidebar_app._jump_argvs` 也只认这两类，Ghostty 会话头行当前不可点。Ghostty 无 per-pane 环境变量，statusline 渲染期不宜调 osascript（300ms 预算）；可行路径是点击时懒算：AppleScript `every terminal whose working directory is <会话 cwd>` 匹配后 `focus`（多窗格同项目时有歧义，需定优先级，如取 front window 首个匹配），或推动 Ghostty 暴露 surface id 环境变量后再走精确映射。README 对点击跳转的「iTerm2 / tmux」描述在补齐前保持现状（是准确描述，非遗漏）。
- **自动 1/3 分屏跟随原会话退出（独立后续，未实现）**：不监听 `/quit` 文本，改由 `$tt-sidebar` launcher 沿父进程树定位原生 Codex PID 并传给 split；macOS 用 `kqueue` `EVFILT_PROC + NOTE_EXIT`、Linux 用 `pidfd` 阻塞等待真实进程退出，收到事件后退出 Textual 并关闭配对 pane。该路径应覆盖 `/quit`、`/exit`、崩溃和原 pane 关闭，不增加 transcript / SQLite 监听或周期 timer；iTerm2 `jobName` 变量监听只作终端专属备用，shell wrapper 需改变启动方式，均不作为主实现。Claude Code 继续优先使用官方 `SessionEnd`，强杀再走同类进程兜底。
- **GitHub issue / PR 状态重新核对**：本地确认 #16 / #17 / #19 对应功能已经落地；外部 open/closed 状态在实际处理前重新查询，不沿用 2026-07-04 的旧快照。
- 桌面版（Tauri）规划：图表可视化、数据钻取、实时监控、多 Agent 多模型监控（仅规划，未启动）
- **会话内彩色报表 hook — 桌面多形态适配**（终端部分已落地，见「进行中」）：桌面 app / web GUI 不吃 ANSI（实测乱码），需按形态输出 HTML/markdown（`tt` 加 `--format`、hook 检测形态）。详见本地 `docs/cc-hook-tt-真彩色.md`。
- **Sidebar 状态引擎 v2（独立迭代）**：把普通 `tt sidebar` 的状态判断从当前 `_infer_state` 时间窗口 + 单个 `pending_tool` 布尔值，升级为「生命周期事件为主、registry / rollout 快照恢复、现有启发式兜底」的每会话状态机；自动 1/3 分屏继续保持纯提示词视图，不显示状态或当前动作，也不与本迭代混做。
  - **Claude Code 数据源**：前台会话优先消费 `UserPromptSubmit`、`PreToolUse`、`PermissionRequest`、`PostToolUse` / `PostToolUseFailure`、`Stop`、`StopFailure`、`SessionEnd`，用 `tool_name` / `tool_input` / `tool_use_id` 补充当前动作；启动或 sidebar 重启时读取 `~/.claude/sessions/<pid>.json` 的 `status`、`statusUpdatedAt`、`waitingFor` 恢复快照。该 registry 是本机 Claude Code 2.1.207 的内部接口，字段缺失或版本变化时必须防御式降级；`claude agents --json` 面向 Agent View / 后台会话，普通前台会话未 background 前不能作为状态真相。
  - **Codex 数据源**：当前普通 TUI 以 `UserPromptSubmit`、`PreToolUse`、`PermissionRequest`、`PostToolUse`、`Stop` hooks 为主，rollout 的 `task_started` / `task_complete` / `turn_aborted` 与带 `turn_id` / `call_id` 的调用、输出配对用于启动恢复和漏事件兜底；`PermissionRequest` 才能精确标记「等授权」，未完成工具不得继续直接等同「待确认」。Codex hooks 暂无 `SessionEnd`，`/quit` / `/exit` 也不会进入 `UserPromptSubmit`；普通总览关闭态继续用进程探活 / TTL 兜底，自动分屏的即时跟随退出按上方独立事项处理。
  - **App Server 后续选项**：协议已提供 `thread/status/changed`、`activeFlags=[waitingOnApproval|waitingOnUserInput]`、`turn/*`、`item/*` 及审批 / 用户输入请求，运行状态精度最高；但需要先启动 App Server，再用 `codex --remote ...` 承载 TUI，不能事后附着普通已运行会话。远程 TUI 执行 `/quit` 只会 `thread/unsubscribe`，最后订阅者离开后默认空闲 30 分钟才发 `thread/closed` 与 `notLoaded`，因此不能替代即时的进程退出监测；仍只作为单独启动架构升级评估，不纳入 v2 首期。
  - **统一状态与工程约束**：目标状态为 `thinking`、`tool_running`、`waiting_approval`、`waiting_input`、`idle`、`closed`、`error`，冲突优先级为「等授权 > 等输入 > 工具运行 > 思考 / 生成 > 空闲 / 关闭」；事件按 `session_id + turn_id + tool_use_id / call_id` 配对，并行工具必须用集合而非单布尔值。hooks 是边沿事件而非查询 API，状态需持久化或可由 registry / rollout 重建；展示工具参数前必须脱敏并截断。
  - **验收标准**：长命令不再误报「待确认」；`PermissionRequest` 到达后立即、准确进入等授权；并行工具完成一个时不得提前切到等输入；Claude Code `SessionEnd` 后及时移除，Codex 异常退出由兜底可靠收敛；sidebar 重启后能恢复正确状态；旧版本、hook 缺失、事件乱序或进程崩溃时不崩溃并安全降级。
- **远程下发通知通道**（📝 规划，新分支 `feature/broadcast` 做）：纯下行——读本地 `~/.claude/tt-broadcast.json`，在 **statusline 第 5 行**（未过期里优先级最高一条、按 level 着色）+ **`tt status` 头图上方横幅**（全部未过期）展示远程下发的公告/告警/进度；level 分流 `urgent`/`info`/`progress` + `expires_at` 自动过期；配 `tt broadcast` 写入命令自测。**只接收不上传、不碰隐私定位**。同步链路（远程→文件）+ 排行榜 PK（需上行、必 opt-in + 脱敏）留后续。详见 `docs/broadcast-plan.md`。

## 阻塞

- 纯 osascript 无法在 iTerm2 原生全屏下调整 pane 列宽；当前安全回滚并提示退出全屏，若以后要求原生全屏 1/3，需重新评估 Python API fallback 或 macOS Accessibility 方案。

## 最近验证

- **2026-10-28 22:11**：上游同步验证。新增定价回归在旧实现上为 72 failed / 36 passed；同步后默认完整 pytest **565 passed in 22.43s**，英文 dumb terminal 完整 pytest **565 passed in 11.40s**。Ruff 全过，mypy 41 个源文件无错误，离线 `uv lock --check` 通过。首次沙箱运行因本地端口监听、进程信息查询和 uv 系统配置读取限制受阻，获准扩大权限后串行重跑通过；未弱化断言。计价器、模型短名和定价测试按上游源码同步，Codex 模板与原有无前导空行测试保留；未做 Windows 真机或 Codex TUI 视觉验收。

- **2026-09-23 20:29**：**Codex Hook 2.0 已同步用户配置**。同步前确认安装脚本为 1.9 且含 ANSI；同步后版本为 2.0，ANSI 标记检查无匹配，使用用户级安装解释器实跑返回合法的双行纯文本 `systemMessage`。`/Users/xinyan/.codex/hooks.json` 的 SHA-256 同步前后均为 `790fbea0ef5a5ab96a8cd7f275eb64f59dd70cd77e08c973fef3e0a33e012eda`，Stop 命令和其他 Hook 均未改动，不触发新的 Hook 定义信任；真实 Codex TUI 展示待本轮结束后人工确认。代码回归仍沿用本次实现验证结果：默认与 `LANG=C LC_ALL=C TERM=dumb` 完整 pytest 各 **418 passed**，Ruff、mypy、`uv lock --check` 和 `git diff --check` 通过。

- **2026-09-10 00:09**：**GPT 命名空间定价修复**。先复现 `chatgpt/`、`openai/` 前缀导致定价归零，再补最小解析规则；新增 21 项回归覆盖两个前缀、Sol / Astra、日期后缀、272K 上下界、完整 ID 报价优先、缓存兜底后新增精确价，以及未知／嵌套／空前缀边界。完整 pytest 和英文 dumb terminal 各 416 passed，Ruff 全过、mypy 41 个源文件无错误、`git diff --check` 通过。只读加载本机现有价格缓存，确认 `chatgpt/gpt-5.6-sol` 命中 `gpt-5.6-sol`；未修改用户级安装或缓存。

- **2026-09-09 19:55**：**0.5.7 打包、发布与远端回验完成**。版本与锁文件一致；完整测试和英文 dumb terminal 各 395 passed，Ruff / mypy / `uv lock --check` / diff 检查通过。sdist / wheel 经 Twine 校验，包文件与发布提交一致，未混入本地品牌素材或临时文件；远端 main 和 tag 目标已核对。PyPI 两份产物元数据及下载字节的 SHA-256 与本地一致；`uvx --no-cache --index-url https://pypi.org/simple --from token-tracker==0.5.7 tt --version` 返回 0.5.7。构建仅有既存 setuptools license 弃用提示；GitHub CI workflow 为 active，但截至检查时未查询到本次发布提交的 Actions 运行记录，不将其标为远端 CI 通过。

- **2026-09-09 17:52**：**Astra / Fable 5.1 定价与 Codex 缓存写入适配**。完整 pytest 与 `LANG=C LC_ALL=C TERM=dumb` 各 **395 passed**（新增 28 项参数化回归），Ruff 全过、mypy 41 个源文件无错误、`git diff --check` 通过。覆盖旧缓存／断网、272K 边界、日期后缀、历史价隔离、按请求而非会话累计套档、缺失／非法缓存写入和状态栏回退。两款内置模型的全部价格字段与实时 LiteLLM 表一致；3 个真实 Astra 会话总 token 与原始日志一致、逐请求分段完整；样本缓存写入均为 0，非零写入以合成日志验证。英文测试首次受 uv 用户缓存沙箱限制，获准扩大权限后重跑通过，未修改用户配置。Fable 5.1 未做真实会话端到端验证。

- **2026-08-31 20:08**：**代码性能优化与 Kimi 裸安装检测修复完成**。真实数据基准：16,508 条 Kimi daily/monthly 聚合约 0.71s/0.68s → 0.033s/0.027s；200MB Codex 当前会话状态栏解析由 2～3 遍收敛为单遍 0.31s；三 Agent 默认 `tt sessions 20` 在约 6.5GB 历史下由全量 10.46s 降至 0.80s，前 20 条 session id 逐项一致，并补近期窗口完整会话、长会话扩窗、候选不足全量回退、状态栏单扫描和 Kimi 根目录检测回归。完整 pytest **367 全绿**、英文 dumb terminal CI 模拟 **367 全绿**、Ruff 全过、mypy 41 个源文件 0 报错、`git diff --check` 通过。

- **2026-08-30 10:55**：**模型定价全面校准与动态计价落地（已实现验证，待发版）**。官方页确认 DeepSeek 新价自 2026-08-16 16:00 UTC 生效，仅周一至周五 UTC 01:00-04:00、06:00-10:00 为峰时，周末全天谷价；同步更新 GPT-5.6、Claude Mythos／Sonnet、GLM、Qwen、Doubao、Grok 等最新价与模型识别。计价器支持按单次请求选择峰谷／长上下文档，Codex 从 `last_token_usage` 保存逐请求 segment，状态栏同口径并升 1.7；补生效时刻、峰谷边界、周六／周日、阶梯边界、provider key、旧版 DeepSeek 隔离与跨时段会话回归。完整 pytest **362 全绿**、英文 dumb terminal CI 模拟 **362 全绿**、Ruff 全过、mypy 41 个源文件 0 报错，`git diff --check` 通过。

- **2026-08-25 21:57**：**版本升至 0.5.6 并完成 GitHub / PyPI 发布与远端回验**。`pyproject.toml` / `uv.lock` 同步 0.5.6，release commit `fb27d57` 与 annotated tag `v0.5.6` 已 push。发布前完整 pytest 与英文 dumb terminal CI 模拟均 **344 全绿**，Ruff 全过、mypy 41 个源文件 0 报错；sdist / wheel 构建与 Twine check 通过。PyPI JSON 已传播，远端 wheel SHA-256 `0d90c7de…`、sdist SHA-256 `3ac94964…` 与本地产物一致；Simple 索引延迟约 1 分钟后，`uvx --no-cache --from token-tracker==0.5.6` 隔离安装正确输出 0.5.6；用户级 `tt` 已 `uv tool upgrade` 至 0.5.6。

- **2026-08-25 21:54**：**Codex sidebar 注入噪声收口：skip 前缀补三类 + `next_hint` 过滤 raw JSON（已实现验证，未发版）**。0.5.5 真机扫描（120h / 54 会话）复核发现自动审批工具（codex-auto-review 类）会话残留三类注入：`The following is the Codex agent history…` 与 `>>> TRANSCRIPT START/DELTA` 段（event 通道本身携带、旧版照收）、goal 模式 `<codex_internal_context source="goal">`；这类会话最后一条回复本身是结构化审批 JSON（event 与 response 通道一致），「下一步」栏显示 raw JSON。修复：`_CODEX_SKIP_PREFIXES` 补 `The following is the Codex agent history` / `>>> TRANSCRIPT` / `<codex_internal_context` 三个前缀；`_hint_text` 入口对 `{` / `[` 开头的回复直接返回空（JSON blob 不是给人看的「下一步」，CC / Kimi 同样受益）。新增 2 个回归（注入前缀过滤、JSON hint 置空）；真机复测 54 会话注入残留 0 条、纯自动审批会话不再显示垃圾提示词。完整 pytest **344 全绿**（含 CI 模拟）、Ruff 全过、mypy 41 个源文件 0 报错。

- **2026-08-25 20:56**：**版本升至 0.5.5 并完成 GitHub / PyPI 发布与远端回验**。`pyproject.toml` / `uv.lock` 同步 0.5.5，release commit `a7ab053` 与 annotated tag `v0.5.5` 已 push。发布前完整 pytest 与英文 dumb terminal CI 模拟均 **342 全绿**，Ruff 全过、mypy 41 个源文件 0 报错，`uv lock --check` 与 `git diff --check` 通过；sdist / wheel 构建与 Twine check 通过，wheel 含新版 `ghostty_split.py` / `sidebar.py`，METADATA 版本正确。PyPI JSON 已传播，远端 wheel SHA-256 `37614ed0…`、sdist SHA-256 `78b697a3…` 与本地产物一致；`uvx --no-cache --from token-tracker==0.5.5` 隔离安装后 `tt --version` 正确输出 0.5.5。

- **2026-08-25 20:19**：**修复 `response_item` 兼容改动的三个真机 bug（已实现验证，未 commit / 未发版）**。上一条（18:15）的兼容改动方向正确，但用本机真实 rollout 复测暴露：① 双写文件（`event_msg` 与 `response_item` 各记一份）同一提示词被收两次（实测「你好 ×2」等成对重复）；② `<recommended_plugins>` 注入前缀未过滤、混进提示词首位；③ `response_item` assistant 的结构化 JSON 输出覆盖 `next_hint` 正常回复。修复：`_CodexParseState` 加 `prompt_channels` 逐条打标与 `last_reply_response` 分通道；`_parse_codex` 收尾时「有 event 通道真实提示词则以 event 为准剔除 response 孪生副本，event 完全没有（纯新版日志）才保留 response 通道」——同文真实重复输入（实测「commit 并继续做下一步」×4）不受影响；`next_hint` 优先 `event_msg agent_message`，无才回退 response_item；skip 前缀补 `<recommended_plugins`。新增 2 个回归（双写去重 + hint 优先序、纯 response_item 文件真实重复保留）；真实 rollout 复测：双写文件 4 条无重复、hint 干净，纯新版文件真实重复保留。完整 pytest **342 全绿**（含 CI 模拟）、Ruff 全过、mypy 41 个源文件 0 报错。

- **2026-08-25 18:15**：**修复 Codex 新版 rollout 导致 `$tt-sidebar` 不显示提示词（已实现验证，待同步用户级工具）**。根因是新版 Codex 将用户与助手消息记录为 `response_item/message`，旧解析器仅识别 `event_msg/user_message`；现已兼容两种格式并新增回归测试。真实当前会话解析出 28 条提示词，最新一条为「去做哟」；`uv run --extra dev pytest`（340 passed）、`LANG=C LC_ALL=C TERM=dumb uv run --extra dev pytest`（340 passed）、`uv run --extra dev ruff check src tests`、`uv run --extra dev mypy src` 全部通过。

- **2026-08-24 21:06**：**Ghostty 分屏加前置探针，精确区分 Codex 沙箱与版本过低（已实现验证，未 commit / 未发版）**。痛点：Codex 沙箱内整段 AppleScript 在编译期死于 -2741（读不到 Ghostty 术语字典），脚本内的运行时版本检查根本没机会执行，-2741 只能把「沙箱限制」与「版本低于 1.3.0」混报成一条文案。修复只动 `ghostty_split.py`：新增 `_probe_ghostty()`，跑整段脚本前先发最小探针 `tell application "Ghostty" to get version`（探针编译同样需加载字典，沙箱下同样 -2741）——-2741 直接断定沙箱，提示「当前命令运行在 Codex 沙箱中，请以沙箱外权限（require_escalated）重跑 $tt-sidebar」；探针成功则在 Python 侧用 `_version_tuple()` 精确比版本，低于 1.3.0 报当前版本号；探针自身失败（osascript 缺失 / 超时）不阻断主流程，原 `_failure_message` 保留作运行时兜底。Codex Skill 模板的 require_escalated 执行指引此前已存在，无需改动。新增 6 用例（探针沙箱精确文案 / 精确版本号 / 达标放行 / 探针失败不阻断 / `_version_tuple` / 运行时兜底），替换原沙箱混报用例；完整 pytest **339 全绿**、CI 模拟环境全绿、Ruff 全过、mypy 41 个源文件 0 报错；本机 Ghostty 1.3.1 实跑探针返回 `(True, '')`。

- **2026-08-24 15:35**：**产品主页三版设计对比稿落盘（候选未定，`homepage.html` 仍为正式版）**。基于同一质感层（JetBrains Mono webfont + 终端网格肌理 + spotlight 卡片 + 入场编排 + reduced-motion 全降级）产出：① `homepage-live.html` 活终端首屏 -- `uvx asciinema rec`（隔离环境，未全局安装）真实录制 `tt daily/weekly --mock`（110×32，mock 脱敏数据），cast 内联进页面经 Blob URL 喂给 asciinema-player，绕开 `file://` fetch 限制，双击即播；② `homepage-kinetic.html` 巨型 `tt▌` 标识（约半屏）+ 打字机逐字价值主张，打完副文案与 CTA 入场；③ `homepage-scrolly.html` GSAP ScrollTrigger（CDN）-- statusline 区 pin 住滚动切换 CC / Codex / CLI 三张真实截图（左侧条目同步高亮，`start: top top+=80`、scrub 0.6），功能区 sticky-stack 卡片堆叠（CSS sticky + 后卡驱动前卡 scale/opacity），`min-width:901px` 以下退回流式布局。**前置修复**：mock 已腐烂 -- `mock/mockdata.py` 仍从 `ui.format` 导入 `system_tz`（已挪至 `token_tracker.tz`），改一行导入后 `tt daily/weekly --mock` 恢复可用（mock 为 gitignored 本地文件）。三页均依赖公开 CDN（fontsource / asciinema-player / GSAP），正式部署需自托管。

- **2026-08-24 14:48**：**产品主页预览落盘（`assets/brand/homepage.html`，静态预览、细节待主人单独优化）**。按定稿品牌（方案 A `tt▌` 光标 + Mauve `#cba6f7`，Catppuccin Mocha 深色锁定）实现：非对称 hero（左文案右 `screenshot-daily.png` 真实截图）、三 Agent 支持徽章条（OpenAI logo CDN 已下架 404，弃用 logo 墙改终端风 mono 徽章）、statusline 截图区、7 格 bento 功能矩阵（4+2 / 2+2+2 / 3+3 节奏，侧边栏与主题两格嵌真实截图）、终端风安装命令块（curl 一键脚本 + `tt setup` + `tt daily`）、页脚 GitHub / PyPI / MIT 链接。工程：IntersectionObserver 滚动淡入、`prefers-reduced-motion` 全降级、CTA 对比度达标、移动端单列塌陷；截图引用 `../screenshot-*.png` 相对路径已核实存在。浏览器打开验证排版与截图加载正常。

- **2026-08-24 14:38**：**品牌 logo 方案定稿落盘：方案 A（`tt▌` 光标）+ Mauve 紫**。经 `preview.html` / `preview-v2.html` 双方案页对比，方案 B（`tt[0]` 像素计数窗）归档未采用（细节多、小尺寸略复杂）；主色对比 Blue / Green / Peach / Teal 后维持 Mauve，深浅底配对 `#cba6f7` / `#8839ef`（同色相，浅底加深保对比度，修正了预览页一度浅底误用 Blue `#1e66f5` 的跨色相问题）。正式资产不变：`logo.svg` / `logo-mark.svg` / `favicon.svg` / `logo-og.svg` / `logo-og.png`；`assets/brand/README.md` 决策记录已更新为定稿状态。

- **2026-08-24 14:18**：**新增品牌 logo 资产（`assets/brand/`，纯设计资产、未改代码）**。核心标识为小写 `tt` + 终端方块光标（`tt▌`），呼应 statusline / CLI 形态；主色 Catppuccin Mauve `#cba6f7`，与 CLI 主题同源。产出 `logo.svg`（mark + 等宽字标完整组合标识，`currentColor` 随主题变色）、`logo-mark.svg`（独立标识）、`favicon.svg`（深底 Mauve，16px 可辨）、`logo-og.svg` + `logo-og.png`（1200×630 社交分享图，GitHub social preview 需手动上传）。两轮 qlmanage / magick 渲染评审：v1 横笔过高误读为大写 T、光标块过小像句号，已修正为横笔下移 + 光标加高至 x 字高；32px / 16px 小尺寸与深浅底均验证可读。字标暂为字体引用原型，生产化待办：安装 JetBrains Mono 后转曲。方向决策与使用规范见 `assets/brand/README.md`。

- **2026-08-10 12:16**：**版本升至 0.5.4 并完成 GitHub / PyPI 发布与远端回验**。本版兼容 Kimi Code 新旧 `state.json`：会话匹配优先读新版 `cwd`、回退旧版 `workDir`，`updatedAt` 同时解析 epoch 毫秒与 ISO 字符串，项目名解析同步兼容；现场失败会话已命中。`pyproject.toml` / `uv.lock` 同步 0.5.4；完整 pytest 与英文 dumb terminal CI 模拟均 **334 全绿**，Ruff 全过、mypy 41 个源文件 0 报错，锁文件与 diff 检查通过。release commit `b33fa84` 与 annotated tag `v0.5.4` 已 push；sdist / wheel 经 Twine、METADATA 与关键包内容检查通过。PyPI JSON 已传播，远端 wheel SHA-256 `96af24db…c901`、sdist SHA-256 `7a8a7505…d659` 与本地产物一致；无缓存安装正确输出 `token-tracker 0.5.4`。

- **2026-08-03**：**`$tt-sidebar` 自动分屏新增 Ghostty 后端（已实现并真机验证，未 commit / 未发版）**。本机 Ghostty 1.3.1 实测先行：1.3 起 AppleScript 字典提供 `split <terminal> direction right with configuration cfg`（cfg 支持 `command` / `initial working directory` / `environment variables`）与 `perform action "resize_split:方向,像素"`（像素级、朝该方向扩展——实测 `left` 使右栏 67→71 列变宽，缩小右栏用 `right`）。关键坑：① `command` 实测以 `shell -c "exec -l <cmd>"` 启动，只能是单条简单命令，复合 shell 语法报 `syntax error near unexpected token '('`，包装逻辑必须落盘成 /tmp 脚本文件、`command` 只传 `/bin/sh <脚本>`；② 字典不暴露窗格列宽 / bounds，1/3 收敛改为「新窗格内 `stty -f /dev/tty size` 后台回写列数文件 + AppleScript `read POSIX file` 闭环 resize」，目标列数 = 平分半宽 ×2/3（`2 * c0 div 3`），方向按回读列数自适应翻转，启动器完全不依赖 /dev/tty（沙箱 / 管道友好），读不到列数时降级保留平分并返回 `_raw` 标记；分步逼近会产生一段可见的「慢慢收成 1/3」过渡动画，经主人确认保留作为默认观感，不做像素/列比一步到位的优化；③ `before` 是 AppleScript 保留字，作变量名报 -2741 误导性语法错；④ `tell` 块内不能 `do shell script`，需移到块外 handler。新增 `ghostty_split.py`（结构对齐 `iterm_split.py`：静态脚本 + argv 传参、版本门 ≥ 1.3.0、焦点窗格 cwd 校验防误分错窗格、失败关新 pane 并回源焦点、新窗格命令带同口径列数门控防半宽首帧），`sidebar_command` 按 `TERM_PROGRAM=ghostty` 路由（优先级 tmux > iTerm2 > Ghostty）。真机端到端：split → 收敛 41 列 ≈ 1/3 → 焦点回源 pane → 临时脚本 / 列数文件自删全过。Skill 模板（Codex/Claude 共享 + Kimi）补 Ghostty 分支（同 iTerm2 沙箱外执行口径，内容比对自动同步、无需 bump SETUP_VERSION），README 双语 / CLAUDE.md / AGENTS.md 同步。完整 pytest **332 全绿**、CI 模拟环境（`LANG=C LC_ALL=C TERM=dumb`）全绿、Ruff 全过、mypy 41 个源文件 0 报错。自动分屏跟随主会话退出（kqueue/pidfd 方案）继续留在「待办 / 计划」，本次未做。

- **2026-08-01**：**Kimi 状态栏 Model 段加实际 effort（`K3/high/auto`），Out TPS 逻辑上线但暂不展示**。effort 取 wire `llm.request.thinkingEffort`（stdin 快照无此字段，跟随 `/model` 实时切换）；Out TPS = usage.record output ÷（llm.request→usage.record 时长），端到端有效值（同 CC `api_duration_ms` 口径）。关键坑：request 与其 usage.record 相隔整个生成时长（实测 3.7~56s），1s 节流增量消费下两者必落不同帧，配对 req_time 必须随 state 持久化（内存态永远配不上）。TPS 显示按用户决定先关闭（`_render` 留一行恢复点），计算与 state 持久化保留。KIMI_STATUSLINE_HOOK_VERSION 升 1.2。真实会话端到端验证：effort `high`、tps 25~38 正常结算。pytest / ruff / mypy 全绿。
