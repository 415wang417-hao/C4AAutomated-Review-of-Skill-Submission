---
AIGC:
    Label: "1"
    ContentProducer: 001191440300708461136T1XGW3
    ProduceID: 60ea5d0126731e33de81577b27285889_9fe44f96bf2d11f1884b525400cd780f
    ReservedCode1: T3bQwFZjgu3vIq7olww8EnVZt+skaR//dqNVZ15YK78m0NoKsRd77TUGZzDaqI3BiSTCETiwLuGEp1ivSg3hydHfhxNZY0viFu3tjgDKmMZwEFeGhersuGnjGB3UaXCnwA+LKYuGfa6J+I1H2nDPoO1KQWDvjr4MuVKnVzGUfRw1kuL27u6bT59t+DA=
    ContentPropagator: 001191440300708461136T1XGW3
    PropagateID: 60ea5d0126731e33de81577b27285889_9fe44f96bf2d11f1884b525400cd780f
    ReservedCode2: T3bQwFZjgu3vIq7olww8EnVZt+skaR//dqNVZ15YK78m0NoKsRd77TUGZzDaqI3BiSTCETiwLuGEp1ivSg3hydHfhxNZY0viFu3tjgDKmMZwEFeGhersuGnjGB3UaXCnwA+LKYuGfa6J+I1H2nDPoO1KQWDvjr4MuVKnVzGUfRw1kuL27u6bT59t+DA=
---

# skill-evaluator 方案设计说明

> 交付物：C4A 挑战 · 交付物一（方案设计）
> 作者：lenovo ｜ 日期：2026-10-03 ｜ 版本：v1.0

---

## 一、要解决的问题

C4 挑战是"技能分享与传播"，同一道题会有 N 份提交。人工评审有三个绕不过去的痛点：

1. **看不完**：每份提交包含说明书、技能包、演示图、教学文档、AI 日志，逐字读完成本极高。
2. **说不清**：凭印象给分（"感觉还行，给 80"），无法解释为什么是 80 而不是 75。
3. **判不准红旗**：缺件、无 AI 日志、一句话直出这三类问题直接决定封顶，但靠肉眼翻拣极易漏判。

`skill-evaluator` 的目标不是"替人打分"，而是**把评审从主观印象变成可复核的流水线**：
机器负责所有可判定项（存在性、结构性、语法、格式、红线），人只负责机器判不了的语义质量。

## 二、设计原则

| 原则 | 具体做法 | 反例（本设计明确拒绝的做法） |
| --- | --- | --- |
| **证据优先** | 每条结论必须挂 `文件 + 行号 + 原文片段`，报告里可直接看到依据 | 只输出"内容表达清晰度：良好" |
| **判据在量表，不在代码** | 评分标准全部外置为 JSON/YAML 量表，换标准不用改一行代码 | 把"必须有安装说明"硬编码进 `if` 语句 |
| **可复算** | 同一输入 + 同一量表 = 同一结果，LLM 分数不参与计分 | 把大模型的主观打分直接算进总分 |
| **红线与得分分离** | 先算原始分，再按红线规则封顶，两个数字都写进报告 | 触了红线就笼统地"扣 20 分" |
| **零依赖优先** | 只用 Python 标准库；PyYAML 缺失时降级到内置 `miniyaml` | 强制 `pip install` 一堆包才能跑 |
| **诚实边界** | 不判抄袭、不做安全审计、不评价"写得好不好"，只给人工复核提示 | 让规则引擎假装自己懂语义 |

## 三、系统架构

```
输入：一个目录（含 .skill / .md / .png / .zip）
  │
  ├─ scanner.py      扫描与分组：按作者前缀聚合成"提交"，识别版本后缀（v1/v2/最终版）
  │                    └─ 输出：{作者: [文件列表]}
  │
  ├─ extract.py      解包与抽取：.skill/.zip 按 ZIP 解包（校验唯一顶层目录、SKILL.md）、
  │                   文本文件读取并保留行号、PNG 仅记元数据
  │                    └─ 输出：可定位到"文件:行"的文本单元
  │
  ├─ completeness.py 完整性核对（Level 2）：五件套逐项匹配
  │                    └─ 输出：5/5 或缺失清单
  │
  ├─ quality.py      质量评审（Level 3）：5 类检测器
  │                   ├─ text_signal     正则命中（含证据行）
  │                   ├─ text_absent     反向命中（如硬编码路径 / 明文凭据）
  │                   ├─ skill_structure ZIP 结构合规性
  │                   ├─ python_syntax   ast 语法检查
  │                   └─ no_placeholder 占位符扫描（TODO / 待补充 / XXX）
  │                    └─ 输出：逐检查项 ✅/⚠️/❌ + 证据 + fix_hint
  │
  ├─ redflags.py     三项红线判定：missing_artifacts / no_ai_log / one_shot_ai
  │                    └─ 输出：命中的红线 + 影响维度 + 封顶分
  │
  ├─ rubric.py       量表加载与适配：default / meta / starter，或任意外部量表
  │                    └─ 自动派生缺失的检查项（兼容只给正负信号的量表）
  │
  ├─ models.py       数据模型与计分口径：official（0.4/0.6 加权）与 rubric（得分率）
  │                    └─ 含红线封顶后的 effective_points
  │
  └─ report.py       四形态输出：Markdown 单人报告 / Markdown 批量汇总 / JSON / CSV / HTML
```

`evaluate.py` 是命令行入口，负责串起全流程；`llm_judge.py` 作为**可选的人工复核提示器**，
输出"建议人工看哪几条"，不参与计分。

## 四、评分口径（两套，量量表）

| 口径 | 触发条件 | 公式 | 适用 |
| --- | --- | --- | --- |
| `official` | 量表写 `scoring: official` | 总分率 = 完整性×0.4 + 质量×0.6 | C4 官方 starter 口径 |
| `rubric` | 量表写 `scoring: rubric` | 总分率 = Σ维度得分 ÷ Σ维度满分 | C4A 自评五维 |

其中「质量得分率 = Σ各维度有效得分 ÷ Σ维度满分」。**这里踩过一个坑**：最初用"维度等级
的平均"当质量得分率，结果一份 89.64/100 的提交算出 100% 得分率——因为等级只有 ✅/⚠️/❌
三档，"过 2 项"和"过 5 项"可能同时落在 ✅。改为加权得分率后，同一份提交得 94%，与总分自洽。

三级判据（每个检查项落到一档）：

- ✅ 通过 → 拿满该检查项权重分
- ⚠️ 部分 → 拿一半（需人工复核，报告标 ⚠️）
- ❌ 未通过 → 0 分，并输出 `fix_hint` 修复建议

## 五、三项红线与封顶

| 红线 | 判定逻辑 | 影响维度 | 封顶 |
| --- | --- | --- | --- |
| `missing_artifacts` | 五件套缺件 | 交付物规范度 | 5 |
| `no_ai_log` | 无 AI 使用日志，或日志中无真实迭代/失败记录 | 迭代与复盘 | 5 |
| `one_shot_ai` | 无迭代痕迹（既无多轮迭代，也无失败记录与原始提示词） | 迭代与复盘 | 5 |

设计要点：**红线只封顶、不直接扣分**。报告同时显示"原始分"与"封顶后有效分"，
避免出现"因为触了红线所以看起来只差一点"的糊弄式评分。

## 六、拿来主义：兼容外部量表

评审器不要求所有人接受我的量表。已实现三种加载方式：

1. `--rubric default` → 内置四条件量表（C4 五件套 + 四维质量）
2. `--rubric meta` → 内置 C4A 五维量表（交付完整性/技术执行/成果完整/自评与复盘/AI 使用）
3. `--rubric starter` → **原样加载官方 starter 的 `c4_rubric.yaml`**（该 YAML 只给
   `positive_signals` / `negative_signals`，没有量化检查项，评审器会自动派生检查项并改用
   通过率判据 `ratio:0.6/0.3`）
4. `--rubric <任意路径>` → 加载自研量表

适配层还兼容 `detection` 字段的**嵌套式**与**平铺式**两种写法——真实踩坑：内置量表用平铺
写法时，完整性一度误判为 0/5。

## 七、如何验证它对不对

- **`--selftest`**：内置自检，校验 miniyaml 解析、量表加载、官方公式口径（无需任何外部数据）
- **`--accuracy --golden <json>`**：与人工金标准比对，输出一致率
- **真实实跑**：4 份真实样本，三种量表，结果见 `reports/` 目录与 `demo.png`

## 八、已知边界（不装的坦诚）

1. **语义质量不自动判**：文风、说服力、创意这类东西，规则引擎判不了也不该判，只出人工复核提示。
2. **不做抄袭比对**：跨提交的文本相似度检测不在本版范围内。
3. **不是安全审计**：仅扫表层风险（硬编码绝对路径、明文凭据、危险调用关键词），不做深度代码审计。
4. **作者分组依赖文件名前缀**：命名混乱的提交需用 `--single-author` 手动归组。
5. **LLM 只作提示**：`llm_judge.py` 的结论不计分，保证分数永远可复算。
*（内容由AI生成，仅供参考）*
