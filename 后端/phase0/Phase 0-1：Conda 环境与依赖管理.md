# Phase 0 · 第 1 讲：Anaconda 环境底座与可复现工作流 (1/3)

## 🎯 本讲目标

1. 确认你在用的 conda 工具链是正常的。
    
2. 配好 channels 与 **strict channel priority**，减少依赖冲突。
    
3. 建立“项目一个环境”的纪律，并学会导出/重建 `environment.yml`。
    

---

## 0) 打开正确的终端

建议在 Windows 上使用 **Anaconda Prompt**（它默认已初始化 conda），避免 PowerShell 里还没 `conda init` 导致命令不可用。

---

## 1) 确认 conda 正常工作（先验收工具链）

### 指令 1：查看 conda 版本

`conda --version`

**它在干什么：** 打印当前 conda 的版本号，用于确认“conda 命令本体存在且可运行”。官方 Getting Started 就用它做第一步自检。

### 指令 2：查看 conda 与环境总体信息

`conda info`

**它在干什么：** 输出 conda 的安装位置、当前激活环境、channels、平台信息等；当你遇到“装了库但找不到/冲突”时，这是排查的第一份“系统体检报告”。

---

## 2) 更新 conda（可选但推荐，避免老版本 solver 诡异行为）

### 指令 3：在 base 环境更新 conda（推荐做法）

`conda update -n base conda -y`

**逐段解释：**

- `conda update`：更新已安装的 conda 包到“与当前环境兼容的最新版本”。
    
- `-n base`：指定“在名为 base 的环境里执行更新”，因为 conda 本体通常装在 base；这比在别的项目环境里更新更合理。
    
- `conda`：你要更新的包名就是 `conda` 本身。
    
- `-y`：自动回答所有确认提示为 yes（否则 conda 会问你 “Proceed ([y]/n)?”）。
    

> 你也可能看到文档写 `conda update conda`（不带 `-n base`），这是在“当前环境”更新；工程实践上更常把 conda 本体维护集中在 base。

---

## 3) 统一渠道策略：conda-forge 置顶 + strict（减少依赖冲突）

> 你用 Anaconda 没问题，但**“不要 defaults/conda-forge 随缘混装”**。一旦混装，常见结果就是依赖求解变慢、甚至不可解。conda 官方和 conda-forge 都明确建议在可能时启用 strict。

### 指令 4：把 conda-forge 加到 channels 列表（高优先级）

`conda config --add channels conda-forge`

**它在干什么：**

- `conda config`：修改/查看 conda 的配置（`.condarc`）。
    
- `--add channels conda-forge`：把 `conda-forge` 添加到 `channels` 这个配置项里（默认是加到列表顶部/高优先级一侧，这在文档里有说明）。
    

### 指令 5：开启 strict channel priority（关键）

`conda config --set channel_priority strict`

**它在干什么：**

- 把 `channel_priority` 设置为 `strict`：当高优先级 channel 里已经有某个包名时，低优先级 channel 的同名包不会被考虑，从而减少跨渠道拼装导致的不兼容问题，同时通常还能显著加快求解。
    

### 指令 6：查看 channels 当前值（建议用官方推荐的 --get）

`conda config --get channels`

**它在干什么：** 输出“合并后的有效配置”里 `channels` 的值，用来验收你刚才的 `--add` 是否生效。

### 指令 7：查看 channel_priority 当前值

`conda config --get channel_priority`

**它在干什么：** 输出当前 `channel_priority`，确认已变成 `strict`。

> 你也可以进一步看 `conda-forge` 的官方“快速配置”示例，它给出的就是这两条命令（add forge + strict）。

---

## 4) 项目一个环境：创建你的“无菌室”（不污染 base）

### 指令 8：创建一个新环境（示例：Python 3.11）

`conda create -n fastapi-phase0 python=3.11 -y`

**逐段解释：**

- `conda create`：创建新环境，并在创建时安装指定包及其依赖。
    
- `-n fastapi-phase0`：给环境起名（`-n` 是 `--name` 的缩写）。
    
- `python=3.11`：在该环境里安装并固定 Python 主版本（后面装包会围绕这个 Python 解析兼容性）。
    
- `-y`：自动确认创建与下载/安装过程中的提示。
    

> Anaconda 官方环境文档也强调：创建环境时把需要的包尽量一次性写进同一条命令，可降低冲突风险（这是“减少 solver 反复回溯”的工程经验）。

### 指令 9：激活环境（进入“无菌室”）

`conda activate fastapi-phase0`

**它在干什么：**

- 把当前 shell 的 PATH 等环境变量切换到 `fastapi-phase0`，之后你运行的 `python/pip` 都来自这个环境，而不是 base。
    

### 指令 10：确认 Python 版本（验收你真的在正确环境里）

`python --version`

**它在干什么：** 打印当前 `python` 的版本号，验证激活是否成功、Python 是否如你创建时指定的版本。conda 官方“管理 Python”文档就用它做验收。

### 指令 11：列出所有环境，并确认当前激活的是哪个

`conda info --envs`

**它在干什么：** 列出所有环境，当前激活的环境会带 `*` 号。官方环境管理文档明确给出 `conda info --envs`/`conda env list` 两种等价方式。

---

## 5) 可复现：导出 environment.yml（让别人/未来的你一键重建）

### 指令 12：导出“从历史记录推导的”环境清单（推荐）

`conda env export --from-history > environment.yml`

**逐段解释：**

- `conda env export`：导出环境描述到 YAML，用于分享与重建。
    
- `--from-history`：只导出你“显式安装/声明”的包（更干净、更适合跨机器/跨平台迁移）。
    
- `> environment.yml`：把命令输出重定向保存为文件 `environment.yml`（这是标准 shell 行为）。
    

> 这一步的工程意义：你后面做 FastAPI、数据库、Docker 时，团队协作与部署都会要求“环境可复现”，否则你会陷入“我机器能跑你机器不能跑”的无限沟通成本。

### 指令 13：从 environment.yml 重建环境（验证可复现能力）

`conda env create -f environment.yml`

**逐段解释：**

- `conda env create`：根据 YAML 文件创建新环境。
    
- `-f environment.yml`：指定要读取的文件（`-f` 即 file）。Anaconda 环境文档也把“从 .yml 创建环境”作为标准用法。
    

> 注意：这条命令会再创建一个新环境（名字取决于 yml 里写的 name 字段）。你现在阶段可以先“知道它存在”，不一定要立刻跑。

---

# ✅ 本讲作业（Checklist）

在 **Anaconda Prompt** 里依次执行，确保输出正常、无红字报错：

`conda --version`

看点：能打印版本号。

`conda update -n base conda -y`

看点：能完成更新且不需要你手动输入 y（因为 `-y`）。

`conda config --add channels conda-forge conda config --set channel_priority strict conda config --get channels conda config --get channel_priority`

看点：channels 里有 `conda-forge`，priority 是 `strict`。

`conda create -n fastapi-phase0 python=3.11 -y conda activate fastapi-phase0 python --version conda info --envs`

看点：`python --version` 是你指定的版本；`conda info --envs` 里 `fastapi-phase0` 带 `*`。

`conda env export --from-history > environment.yml`

看点：当前目录生成 `environment.yml` 文件。

---

## ⏭️ 下一讲预告（Phase 0 · 第 2 讲）

**Phase 0 · 第 2 讲：安装 FastAPI 的“最小依赖集”与依赖清单策略（conda + pip 边界）**  
你会学到：

- `conda install` 与 `pip install` 怎么分工（避免装错环境/装出冲突）
    
- 如何把 pip 依赖正确写进 `environment.yml`（工程团队常用做法）
    
- 以及一个适合 Obsidian 的“命令块排版模板”