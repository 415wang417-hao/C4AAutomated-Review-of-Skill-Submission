---
AIGC:
    Label: "1"
    ContentProducer: 001191440300708461136T1XGW3
    ProduceID: 60ea5d0126731e33de81577b27285889_a14de1ebbf2d11f1887c525400de85a5
    ReservedCode1: D6BxL7cKYohIF4Az6JY+LXkKrgIKUkgJAnT34lda0apm0hNItm7iKNeb9kXsqdLvN65enGt+0HLvcPqwDCtreBvhq22ePsf/5z4ZOPwWKOT7GcRBVXi838oZUkm5U4cQJHMi3jgd+yCtSu9bsI0lSISJ7DHvbPDiWgmMkJ0IPKH3FPipHWO+SjtHzok=
    ContentPropagator: 001191440300708461136T1XGW3
    PropagateID: 60ea5d0126731e33de81577b27285889_a14de1ebbf2d11f1887c525400de85a5
    ReservedCode2: D6BxL7cKYohIF4Az6JY+LXkKrgIKUkgJAnT34lda0apm0hNItm7iKNeb9kXsqdLvN65enGt+0HLvcPqwDCtreBvhq22ePsf/5z4ZOPwWKOT7GcRBVXi838oZUkm5U4cQJHMi3jgd+yCtSu9bsI0lSISJ7DHvbPDiWgmMkJ0IPKH3FPipHWO+SjtHzok=
---

# skill-evaluator 教学说明

> 交付物：C4A 挑战 · 交付物二（教学说明）
> 目标读者：第一次使用评审器的同学 / 助教 / 评委
> 阅读时间：8 分钟 ｜ 动手时间：3 分钟

---

## 一、先跑通，再理解

在你读完任何解释之前，先执行这一条命令，验证环境没问题：

```bash
cd skill-evaluator/scripts
python evaluate.py --selftest
```

**预期输出**（四行，最后一行必须是 PASSED）：

```
[miniyaml] SELFTEST PASSED
[selftest] rubric 加载 OK：c4_submission_rubric.json（维度 4）
[selftest] models OK（官方口径 composite=0.00）
=== selftest PASSED ===
```

如果这里就报错，说明 Python 版本低于 3.8 或目录不完整，先解决这一步再往下。

## 二、评审第一份提交（3 分钟）

```bash
# 用 C4 官方四条件口径，评审一个交付物目录
python evaluate.py "C:\Users\lenovo\Desktop\C4 Skill Sharing and Dissemination交付物" \
    --out ".\reports\01_lenovo"
```

**看终端最后几行**，你会看到一行结论：

```
· lenovo    总分  89.64 / 100.0 | 得分率  94% | 完整性 5/5 | 红线 无
```

这一行就是全部关键信息：**总分、得分率、五件套齐不齐、有没有触红线**。

然后打开 `--out` 指定的目录，得到 5 个文件：

| 文件 | 什么时候看 |
| --- | --- |
| `reports\lenovo_report.md` | 看单人明细，**先看这个** |
| `batch_summary.md` | 一次评多人时，看排名与共性短板 |
| `results.json` | 想把结果接进别的系统 |
| `results.csv` | 想用 Excel 排序、做表 |
| `dashboard.html` | 双击用浏览器打开，给不装环境的人看 |

## 三、怎么看懂单人报告

报告有五个固定章节，按顺序读即可：

**① 结论速览** — 一张表看完总分、得分率、完整性、红线、置信度。
注意「置信度」：标 **低** 表示规则引擎的证据偏薄（比如只匹配到文件名没匹配到正文），建议人工扫一眼。

**② 交付完整性检查** — 五件套逐项对照，齐备的显示匹配到的具体文件名。

**③ 质量评审** — 核心章节。每个检查项一行，四列：

| # | 检查项 | 结果 | 检测器 | 证据（文件:行 片段） |
| --- | --- | --- | --- | --- |
| 2 | 无硬编码绝对路径 | ❌ | `text_absent` | ✗ 教学说明.md:184 — `python scripts/organize.py "C:/Users/me/Desktop/我的文件"` |

- **结果**：✅ 通过 / ⚠️ 部分（需人工复核）/ ❌ 未通过
- **证据**：直接给出文件、行号、原文片段——**这是本评审器最重要的设计**，任何结论都可点开原文核对
- ❌ 项下面会跟一条「改进建议」，告诉你怎么改能拿到这几分

**④ 红线检查** — 三项红旗的判定与封顶说明。显示"— 未命中"就是好事。

**⑤ 附录** — 文件清单（含大小、版本、可读性）与版本轨迹（识别 v1/v2/最终版）。

## 四、换一套评分标准（不用改代码）

评审器自带三套量表，用 `--rubric` 切换：

```bash
# ① C4 官方四条件：完整性×0.4 + 质量×0.6
python evaluate.py ".\submissions" --rubric default

# ② C4A 五维自评（交付完整性/技术执行/成果完整/自评与复盘/AI 使用）
python evaluate.py ".\submissions" --rubric meta

# ③ 直接吃官方 starter 的 c4_rubric.yaml
#    （该文件只给正负信号，没有量化检查项，评审器会自动派生检查项）
python evaluate.py ".\submissions" --rubric starter

# ④ 你自己的量表
python evaluate.py ".\submissions" --rubric "D:\my_rubric.json"
```

## 五、常见场景与坑

**场景 1：一个目录里有多个人的提交**
按文件名前缀自动分组。文件名写成 `张三_C4_技能说明.md` 就能正确聚合。

**场景 2：目录里的文件命名不规范**
用 `--single-author 张三`，把整个目录当成张三一个人的提交。

**场景 3：文件名不含 "C4"，被全部跳过**
加 `--include-non-matching`。

**场景 4：只想先看看结果，不生成文件**
加 `--dry-run`。

**场景 5：想只比对人工评分的一致性**
```bash
python evaluate.py ".\submissions" --accuracy --golden ".\golden.json"
```

### 踩坑速查表

| 现象 | 原因 | 解决 |
| --- | --- | --- |
| 完整性显示 0/5，但文件明明都在 | 量表里 `filename_patterns` 写成了平铺式（没放在 `detection` 下） | v1.0 已兼容两种写法；如仍异常，检查 `--rubric` 指向的文件 |
| 得分率 100%，但总分只有 89.64 | 旧版用"维度等级平均"算质量得分率导致虚高 | v1.0 已改为加权得分率，两数自洽 |
| 终端输出中文乱码 | Windows 控制台默认 GBK | `set PYTHONIOENCODING=utf-8`（PowerShell：`$env:PYTHONIOENCODING="utf-8"`） |
| 提示 `No module named yaml` | 没装 PyYAML | **不影响运行**，会自动降级到内置 `miniyaml` |
| 路径含空格或中文报错 | 未加引号 | 用双引号包住路径：`python evaluate.py "C:\我的 目录"` |

## 六、自己写一份量表（最小可用模板）

量表就是一个 JSON，核心是「维度 → 检查项 → 判据」：

```json
{
  "id": "my_rubric",
  "version": "1.0",
  "scoring": "rubric",
  "deliverables": [
    { "key": "说明文档", "weight": 1,
      "detection": { "filename_patterns": ["说明"], "preferred_extensions": [".md"] } }
  ],
  "dimensions": [
    { "key": "可实现性", "weight": 25,
      "checks": [
        { "name": "含安装步骤", "weight": 10, "detector": "text_signal",
          "params": { "patterns": ["安装", "install"] },
          "fix_hint": "补一节安装步骤，写清命令与依赖" }
      ] }
  ],
  "red_flags": [
    { "key": "missing_artifacts", "cap": 5, "target": "artifact" }
  ]
}
```

写完直接 `--rubric "路径"` 就能用。检测器可选：`text_signal`、`text_absent`、
`skill_structure`、`python_syntax`、`no_placeholder`。

## 七、给评委的三条使用建议

1. **先看红线，再看分数**：红线命中意味着该维度封顶 5 分，分数本身已经失去区分度。
2. **抽查证据行**：随机抽 2~3 条 ✅，打开原文件核对行号。证据对不上，说明量表匹配规则需要收紧。
3. **把 `fix_hint` 反馈给作者**：这份工具的价值一半在评分，一半在"告诉他怎么改能拿分"。
*（内容由AI生成，仅供参考）*
