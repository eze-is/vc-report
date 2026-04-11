# 素材处理

将用户提供的素材（链接、关键词、笔记、录音）转化为结构化的调研详情文件。

## 输入入口

### 对话直接给（主路径）

用户在对话中直接提供素材，AI 即时处理：

**A. 链接**
```
用户："帮我看下这个 https://finance.yahoo.com/news/traini-..."
```
1. 遵循 web-access skill 抓取页面内容
2. 按 `references/prompts/process-material.md` + profile.yaml 的字段规范结构化
3. 写入 `projects/{当前项目}/research/topic.md`
4. 展示摘要供用户确认/补充

**B. 关键词**
```
用户："帮我查一下 XX 公司最近的数据"
```
1. 遵循 web-access skill 搜索相关信息
2. 筛选有价值的来源，逐个抓取
3. 综合多源信息，按 profile 字段结构化
4. 写入 `projects/{当前项目}/research/topic.md`
5. 展示调研结果，标注信息来源

**C. 笔记**
```
用户："今天跟 XX 创始人聊了，要点是..."
```
1. 按 profile.yaml 的字段规范结构化（如按产品/模型/运营/融资拆分）
2. 写入 `projects/{当前项目}/research/topic.md`
3. 展示结构化结果供用户确认

**D. 录音**
```
用户："帮我处理这些录音" + 文件路径
用户："这是上周路演的录音，整理一下"
```
1. 遵循 audio-transcribe skill 检查环境依赖
2. **压缩并替换原件**：先将录音压缩（audio-transcribe skill 的压缩流程），压缩完成后即可删除原件，后续全部使用压缩版
3. 调用 `transcribe.py` 转录压缩版，**输出到 `projects/{当前项目}/transcripts/`**（持久化保存）
   - **优先启用说话人分离**（需要 HF_TOKEN 已配置），不加 `--no-diarization`
   - 说话人标识（SPEAKER_00 等）是后续"谁说了什么"的关键归属依据
   - 如果发现已有转录但缺少说话人分离，用 `--diarize-only` 补跑（复用 ASR 结果，只跑 pyannote）
   - **不要输出到 /tmp**——转录文本是不可再生的中间产物（重跑需要 ASR 费用和时间），必须持久化
3. 读取 `.transcript.md` 转录文本
4. **按 vc-report 的视角加工**（不是照搬转录文本）：
   - 按 profile.yaml 的字段规范结构化（产品/模式/运营/融资等）
   - 提炼投资洞察，而非逐句记录
   - 识别说话人角色（创始人/投资人/行业专家），标注关键判断的归属
   - 保留关键原话作为引用（有判断价值的观点、数据承诺等）
   - **过滤自家机构信息**：检查 profile.yaml 的 `firm` 字段，该机构方的内部表态、策略、需求等不写入 research 文件
5. 写入 `projects/{当前项目}/research/topic.md`（格式同其他素材类型）
6. 展示摘要供用户确认/补充

**批量录音处理**：用户一次性给多个录音文件时，每个文件派一个子 Agent 并行转录 + 加工：
```
主 Agent
  ├─ 子 Agent 1: 压缩 file1.m4a → 删除原件 → 转录压缩版 → 加工为 research/topic1.md
  ├─ 子 Agent 2: 压缩 file2.m4a → 删除原件 → 转录压缩版 → 加工为 research/topic2.md
  └─ ...
```
子 Agent prompt 中必须写：`必须加载 audio-transcribe skill 并遵循指引` + `压缩完成后立即删除录音原件`。

### inbox 批量给（补充路径）

用户提前把素材放到 `inbox/` 目录，然后告诉 AI "处理一下 inbox"。

**inbox.md**：用户在这里粘贴链接和关键词，每行一条。格式自由：
```markdown
https://finance.yahoo.com/news/traini-...
https://arxiv.org/pdf/2001.08361
查一下 XX 公司最近的 DAU 数据
某篇论文，跟某个技术方向相关
```

**其他 .md 文件**：用户直接放进 inbox 的笔记文件，当作"笔记"类型处理。

**音频文件**：用户放进 inbox 的 .mp3/.m4a/.wav/.aac 等文件，当作"录音"类型处理。

**处理流程**：
1. 读取 `inbox/inbox.md`，逐行判断类型（URL → 链接模式，其他 → 关键词模式）
2. 读取 inbox/ 中其他 .md 文件 → 笔记模式
3. 读取 inbox/ 中音频文件 → 录音模式（遵循 audio-transcribe skill）
4. 逐条处理，写入 `projects/{当前项目}/research/`
5. 处理完成后清空 inbox.md 对应行，移动已处理文件到 `inbox/_processed/`

## research 文件格式

每个话题一个 .md 文件，文件名为「公司/话题名 + 核心关键词」，中英文混合，可读性优先（如 `AcmeCorp-语音助手.md`、`YC-S25批次观察.md`、`端侧模型-性能对比.md`）。避免纯英文缩写（如 `acme.md`、`yc-s25.md`），打开文件夹时应该一眼知道每个文件是什么。

```markdown
# {话题名称}

> 来源：{link/search/note/audio} | {日期}
> URL：{如有}
> 录音：{原始文件名，仅 audio 类型}
> 转录：{transcripts/xxx.transcript.md，仅 audio 类型}

## {板块1}（按 profile 字段规范）
- 要点1
- 要点2
- ...

## {板块2}
- ...

## 关键数据
- {数据指标}[^1]
- {数据指标}[^2]

## AI 摘要
{2-3 句话的压缩摘要，供周报直接引用或改写}

---
[^1]: https://xxx.com — 来源描述
[^2]: https://yyy.com — 来源描述
```

**脚注引用**：关键数据点必须用 `[^N]` 标注来源 URL，文末集中列出脚注。让每个数据都可溯源。

**文件中的 "AI 摘要" 段**是关键：它是调研详情到周报之间的桥梁。用户审阅时可以看到完整详情，同时也能快速看到 AI 打算怎么压缩。

## 合并逻辑

同一话题多次输入时（如先有笔记，后有链接补充）：
- 如果 `projects/{当前项目}/research/` 已存在同话题文件 → 追加/合并内容，而非覆盖
- 更新 AI 摘要
- 标注信息来源的追加时间

## 并行调研

当用户要求同时调研多个方向时，派出子 Agent 并行处理：
- 每个子 Agent 负责一个独立话题
- prompt 中必须写：`必须加载 web-access skill 并遵循指引`
- 用"获取"而非"搜索"描述任务，让子 Agent 自主选择工具
- 每个子 Agent 的结果汇总后分别写入对应的 research 文件

## 参考

- 素材结构化 prompt：`references/prompts/process-material.md`
- 录音加工 prompt：`references/prompts/process-audio.md`
