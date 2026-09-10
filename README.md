作者:抖音 萌猪过河。用本机 MATLAB 执行数值建模、自动控制作图及 Simulink 建模仿真，验证并交付结果；提供可见窗口测试例程，以及 Codex、豆包工作电脑版的 MATLAB MCP 接入指南。
---

# MATLAB Automatic WWJ

把用户的建模描述变成可复现的 `.m` 脚本，实际执行，检查数值结果后交付。执行层首选 MathWorks 官方 MATLAB MCP Server；模型选择、参数解释和验证由本 skill 指导。

## 按用户请求选择入口

- “跑一下例子 / 测试 skill / 打开 MATLAB 演示”：使用下方两套测试例程，执行方法见 [references/test-routines.md](references/test-routines.md)。
- “豆包配置 MCP / 添加 MATLAB 连接器”：读 [references/doubao-mcp.md](references/doubao-mcp.md)，按豆包工作电脑版的本地 STDIO 入口配置。不要替换成扣子，也不要将配置教程说成已经在用户豆包账号中测试通过。
- “用 MATLAB / Simulink 做具体模型”：按下方建模过程执行，不能用固定例程代替用户的实际模型。

## 接入与选择

- 先发现当前可用 MATLAB MCP 工具，并读取工具的实时参数 schema。首次使用调用 `detect_matlab_toolboxes`，确认 MATLAB/Simulink 和任务所需工具箱；安装列表不等于许可证一定可用，实际运行错误才是最终证据。
- 小规模环境查询使用 `evaluate_matlab_code`。可复现任务先写 `.m` 文件，再用 `run_matlab_file` 执行，使用宿主 MATLAB 可访问的绝对路径。
- 若当前会话尚未加载已配置的 MCP，可用本机 `matlab -batch` 执行同一脚本；明确说明实际采用的通道。不要声称命令行成功就意味着当前会话已加载 MCP。
- 用户要求真实窗口时，使用有桌面的 MATLAB 会话，或以 `matlab -r` 启动桌面例程。`-batch` 和 `nodesktop` 不满足真实窗口展示要求；演示结束保留本次模型和图窗。窗口演示不需要修改用户全局 MCP 配置。
- 本机位置与客户端配置见 [references/connection.md](references/connection.md)。更换电脑时发现实际 MATLAB 安装路径，不直接照抄机器路径。
- MCP 会话中不要调用 `restoredefaultpath`；它会移除服务函数所在路径，破坏当前连接。

## 建模过程

1. 从用户描述中确定输入、输出、方程、参数及单位、初始条件、仿真区间与验收标准。可合理假设的演示参数直接选择并说明；会实质改变研究结论的缺失参数才需要澄清。
2. 每次运行建立独立输出目录。已有模型先另存副本再改；在共享会话中只关闭本次创建的模型和图窗，避免 `clear all`、`close all`、`bdclose all`。
3. MATLAB 数值建模选择适合方程的求解器，输出结构化数据。Simulink 使用 `new_system`、`add_block`、`add_line`、`set_param`、`save_system` 和 `sim` 创建、连接、配置、保存并仿真真实模型。先读已有模型再修改；不用截图点击搭建框图。
4. 设置并记录求解器、时间步长/误差容限、停止时间和信号记录。通过 `Simulink.SimulationOutput` 中的实际日志读取结果，不把公式曲线当成仿真输出。
5. 用解析解、独立数值方法、守恒量、边界条件或用户验收指标验证。检查时间覆盖、样本数量、NaN/Inf 和合理量级。只有运行和验证均通过才能报告成功。
6. 交付 `.m`、实际 `.slx`（Simulink任务）、原始 `.mat` 或 CSV、结果图和简短结果说明。逐项核对用户明确要求的产物；脚本的数值校验通过不代表额外要求也已完成。总结参数、关键指标、输出路径和仍未验证的部分。

执行失败后保留日志，定位具体错误再修复；避免无变化重复重跑。长时间无返回时先检查本次进程、日志和超时状态，不启动重叠计算。模型生成成功与模型物理有效性要分别判断。

## 两套用户测试例程

| 用户想看什么 | 例程 | 验收重点 |
|---|---|---|
| 真实 Simulink 模型、Scope 与运行曲线 | [assets/codex_desktop_demo.m](assets/codex_desktop_demo.m) | 自动搭建一阶系统，10 秒仿真，1001 个样本，对照解析解；GUI 模式窗口保持打开 |
| 普通 MATLAB 代码作图 | [assets/control_theory_plots.m](assets/control_theory_plots.m) | 五种阻尼比的阶跃响应、Bode 图和独立三阶根轨迹，数值验证后保存图与数据 |

默认给用户展示可见窗口。仅在用户要求后台验证或接入测试时选择无界面模式。Windows 一次运行两例使用 [scripts/run-test.ps1](scripts/run-test.ps1) 的 `-Test all`；单例用 `-Test simulink` 或 `-Test control`，加 `-Headless` 做后台验证。启动器将脚本复制到指定输出目录，避免把结果写进 skill 安装目录。

其他系统或通过 MCP 运行时，先在调用方设置 `codex_demo_config.output_root` 为用户项目内的绝对路径，再设置 `codex_demo_config.visible` 并调用例程；具体 MATLAB 命令、依赖、输出清单与成功条件见测试说明。`run` 会切换当前目录，因此不能依赖它调用前的 `pwd` 自动传入脚本。

读实际 `summary.json` 和全部要求的产物后判断通过；进程启动成功、MCP 已连接或图窗已出现均不能单独算测试通过。可选的旧后台连通示例 [assets/codex_first_order_demo.m](assets/codex_first_order_demo.m) 仍可用，但它不是默认桌面展示入口。

## 跨智能体使用

本机 Codex 配置见 [references/connection.md](references/connection.md)；豆包工作电脑版的实际菜单、字段与逐步测试见 [references/doubao-mcp.md](references/doubao-mcp.md)。支持本地 STDIO MCP 的客户端可复用同一官方服务；HTTP MCP 或函数调用客户端需要匹配的执行适配层。不同智能体各自配置连接器与技能，不能把 Codex 的 TOML、技能安装路径或 `$技能名` 语法当成所有产品都支持的格式。
