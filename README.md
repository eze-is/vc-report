# vc-report

VC 投资人调研周报/月报助手。把零散的调研素材（链接、关键词、笔记）变成结构化的周报/月报。

## Install

```bash
npx skills add eze-is/vc-report
```

或者直接告诉 Claude Code：

> 帮我安装这个 skill：https://github.com/eze-is/vc-report

## 依赖

- [web-access](https://github.com/eze-is/web-access) — 联网调研能力（需先安装）
- Python 3 + PyYAML — 运行 `scripts/build-prompt.py`

## 核心特性

- **Profile-driven**：先学习用户的报告风格和关注领域，再按其习惯输出
- **两阶段调研**：子 Agent 客观探索（不带偏见）→ 主 Agent 认知碰撞（加载 worldview）
- **worldview.md**：持续演化的认知框架，每次调研后审视更新
- **程序化 prompt 组装**：`build-prompt.py` 根据 stage 自动隔离上下文注入，避免确认偏误
- **脚注引用**：关键数据标注来源 URL，可溯源
- **append 式调研**：子 Agent 边调研边写入文件，保留探索路径，防中断丢失

## 工作流

```
1. 学习风格 — 丢入历史周报样本 → 生成 profile.yaml
2. 收素材   — 对话直接给（链接/关键词/笔记）或 inbox 批量给
3. 调研     — 子 Agent 客观探索 → 主 Agent 认知碰撞
4. 出报告   — 从 research 文件压缩生成周报/月报
```

## 工作区结构

```
<workspace>/
├── profile.yaml          # 用户画像 + 报告风格 + 调研偏好
├── worldview.md          # 持续演化的认知框架
├── samples/              # 历史周报样本（风格学习用）
├── notes.md              # 调研脉络（事实层）
├── inbox/                # 素材投递入口
│   └── inbox.md          # 粘贴链接/关键词
└── reports/
    └── 2026-W15/         # 每期报告 = 一个子项目
        ├── research/     # 调研详情（子 Agent 写入）
        └── output.md     # 最终周报
```

## 两阶段调研设计

```
阶段 1（子 Agent）：客观探索
  ├─ 不带 worldview（避免确认偏误）
  ├─ 不带 research_preferences（避免角度筛选）
  ├─ prompt 由 build-prompt.py 程序化生成
  └─ 边调研边写入 research 文件

阶段 2（主 Agent）：认知碰撞
  ├─ 加载 worldview → 与新发现碰撞
  ├─ 加载 research_preferences → 从用户视角审视
  ├─ 矛盾 = 最有价值的发现
  └─ 更新 worldview，生成 output
```

## Skill 文件

```
vc-report/
├── SKILL.md                      # 主入口
├── research-philosophy.md        # 调研哲学（两阶段、深度标准）
├── profile-learning.md           # 风格学习流程
├── material-processing.md        # 素材处理流程
├── report-generation.md          # 报告生成流程
├── scripts/
│   └── build-prompt.py           # 子 Agent prompt 程序化组装
└── references/
    ├── prompts/                  # 各阶段 prompt 模板
    └── examples/                 # profile 示例
```

## License

[CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/) — 可自由使用和修改，禁止商用。
