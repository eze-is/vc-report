# 报告生成

从项目的 `research/` 目录汇总生成报告。

## 触发

- "出报告" / "汇总一下" / "生成输出"
- 用户在某个项目上下文中要求产出

## 报告生成流程

1. **确定项目**：用户指定的项目（如 `projects/2026-03-录音整理/`），或当前正在操作的项目
2. **读取素材**：遍历 `projects/{项目}/research/` 所有 .md 文件
3. **读取 profile**：加载 profile.yaml 的报告结构模板和风格
4. **按模板组织**：
   - 按 profile.structure 的板块顺序排列
   - 每条素材根据内容和来源归入对应板块（板块定义来自 profile，不做预设）
   - 同一项目/公司的多条素材合并
5. **过滤自家机构信息**：检查 profile.yaml 的 `firm` 字段，确保 output 中不出现该机构的内部策略、合伙人判断、与项目方的互动细节等。即使 research 文件中已做过滤，output 阶段仍需再检查一遍
6. **压缩内容**：
   - 优先使用各 research 文件中的 "AI 摘要" 段
   - 按 profile 的 detail_level 控制信息量
   - 保留关键数据指标
7. **输出**：写入 `projects/{项目}/output.md`
8. **展示给用户**：用户可修改，触发风格学习

## 跨项目聚合

月报等需要聚合多个项目 research 的场景：

1. 创建月报项目：`projects/2026-04-月报/`
2. 聚合相关项目的 research 文件（按日期筛选或用户指定）
3. 按 profile 模板组织，侧重：
   - 趋势总结而非逐条罗列
   - 跨项目对比
   - 重点项目的进展汇总
4. 月报如有专属调研 → 写入 `projects/2026-04-月报/research/`
5. 输出到 `projects/2026-04-月报/output.md`

## 报告格式

按 profile.yaml 的 structure 字段动态生成。板块名称、顺序、层级关系、每个板块的信息粒度全部来自 profile，不做预设。

## 脚注引用

周报中的关键数据点必须用脚注标注来源，从 research 文件的脚注中继承：

```markdown
XX 公司 DAU 涨到了 160w[^1]，CLI 工具比 MCP 便宜 10-32x[^2]

---
[^1]: https://example.com/source — 数据来源
[^2]: https://example.com/benchmark — benchmark 文章
```

如果 profile 中有专门汇总链接的板块，该板块本身不需要脚注（它就是链接列表）。

## URL 有效性校验

生成报告后、展示给用户前，对所有引用的 URL（脚注 + 链接汇总板块）做一次快速检查：
1. 遵循 web-access skill，逐个 WebFetch 检查是否可访问
2. 无法访问的 URL 标注 `[链接失效]` 或替换为可用的替代来源
3. 微信公众号链接（mp.weixin.qq.com）通常有效但不可被 WebFetch 抓取，可跳过校验

## 用户修改与反馈

用户看到生成的报告后可能会：
- "这部分太简略了，补充一下" → 从 research 文件中补充
- "这条不重要，删掉" → 删除并记录偏好
- "这个板块的顺序调一下" → 更新 profile.yaml 的 structure
- "这个表述不对" → 直接修正

每次修改都是风格学习的输入（见 profile-learning.md）。

## 参考

- 报告生成 prompt：`references/prompts/generate-report.md`
