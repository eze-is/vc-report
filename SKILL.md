---
name: vc-report
description: |
  VC 投资人调研周报/月报助手。处理调研素材（链接、关键词、笔记），生成结构化调研报告。
  触发场景：周报、月报、调研汇报、VC report、投资调研、帮我调研、收素材、生成报告。
metadata:
  author: eze
  version: "1.0.0"
---

# VC Report — 调研报告助手

把零散的调研素材（链接、关键词、笔记、录音）变成结构化的调研报告。Profile-driven，先学用户风格，再按其习惯输出。

## 工作区结构

全局资产放根目录（跨项目共享），每个项目是一次独立的整理/调研任务。

```
<workspace>/
├── profile.yaml              # 全局：用户画像 + 报告风格 + 模板结构
├── worldview.md              # 全局：持续演化的认知框架（赛道判断、趋势 thesis）
├── samples/                  # 全局：历史报告样本（风格学习用）
│
└── projects/                 # 项目集合
    ├── 2026-03-录音整理/       # 一个项目 = 一次整理任务
    │   ├── inbox/            # 项目素材入口（录音、链接、笔记）
    │   ├── transcripts/      # 录音转录原文（持久化，追问时回溯）
    │   ├── research/         # 调研详情（用户可审阅的中间态）
    │   └── output.md         # 最终产出
    │
    ├── 2026-W15-周报/
    │   ├── inbox/
    │   ├── research/
    │   └── output.md
    │
    └── 2026-04-月报/           # 可引用其他项目的 research
        ├── inbox/
        ├── research/
        └── output.md
```

## 数据流

```
素材投入项目 inbox
  → projects/{项目}/transcripts/（录音转录原文，持久化）
  → projects/{项目}/research/topic.md（调研详情，用户可审阅）
    → projects/{项目}/output.md（报告压缩版）
```

用户审阅路径：
- 报告觉得某话题压缩太狠 → 打开 research/ 详情 → 告诉 AI 补充
- research 稿需要增删内容 → AI 回溯 transcripts/ 中的转录原文重新提炼

## 三个核心动作

### 1. 学习风格

首次使用或更新样本时执行。从 `samples/` 中提取风格模板，生成 `profile.yaml`。

→ 详见 [profile-learning.md](profile-learning.md)

### 2. 收素材 & 调研

日常使用。两个入口：

**对话直接给**（主路径，零门槛）：
- 甩链接："帮我看下这个 https://..."
- 说关键词："帮我查一下 XX 公司最近的数据"
- 发笔记："今天跟 XX 创始人聊了..."
- 丢录音："帮我处理这些录音" + 文件路径

**inbox 批量给**（补充路径）：
- `inbox.md` 里粘贴多个链接/关键词
- 直接往 inbox/ 放 .md 文件或音频文件
- 告诉 AI "处理一下 inbox"

**关键：调研不是信息搬运。** 收到素材后不要浅尝辄止，要主动发散探索。

→ **[research-philosophy.md](research-philosophy.md)**（调研哲学）
→ [material-processing.md](material-processing.md)（素材处理流程）

**子 Agent prompt 必须通过脚本生成**，不要手动拼：

```bash
# 阶段 1：客观探索（脚本自动隔离 worldview 和 research_preferences）
python3 ~/.claude/skills/vc-report/scripts/build-prompt.py \
  --stage 1 \
  --topic "话题名" \
  --initial-info "用户提供的初始信息" \
  --output-path "projects/{项目}/research/topic.md" \
  --workspace "<workspace路径>" \
  --depth medium  # shallow/medium/deep

# 阶段 2：认知碰撞（脚本自动注入 worldview 和 research_preferences）
python3 ~/.claude/skills/vc-report/scripts/build-prompt.py \
  --stage 2 \
  --topic "交叉分析主题" \
  --workspace "<workspace路径>" \
  --research-files "file1.md" "file2.md"
```

脚本根据 stage 参数程序化决定注入什么上下文，避免主 Agent 手动拼接出错。

### 3. 出报告

汇总产出。读取 `projects/{项目}/research/` 所有文件，按 profile 模板压缩成报告。

→ 详见 [report-generation.md](report-generation.md)

## 主动评估

每次收到用户消息后，响应前先判断：

1. **profile.yaml 存在吗？** → 不存在则引导首次设置
2. **用户在做什么？** → 丢素材 / 要报告 / 讨论调研方向
3. **需要联网吗？** → 有链接或关键词 → 遵循 web-access skill
4. **有新判断？** → 主动更新 notes.md（事实层）和 worldview.md（认知层）

## 首次操作前加载

| 场景 | 先读 |
|------|------|
| **任何调研开始前** | **[research-philosophy.md](research-philosophy.md)** |
| 学习风格 | [profile-learning.md](profile-learning.md) |
| 处理素材 | [material-processing.md](material-processing.md) |
| 生成报告 | [report-generation.md](report-generation.md) |
| 联网操作 | web-access skill |
| 录音处理 | audio-transcribe skill |

## 依赖

- **web-access skill**：所有联网操作遵循 web-access skill
- **audio-transcribe skill**：所有录音转录操作遵循 audio-transcribe skill
- **Python 3 + PyYAML**：运行 scripts/build-prompt.py
- 子 Agent prompt **必须由 build-prompt.py 生成**，不要手动拼接
