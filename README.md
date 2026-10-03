# C4A · Automated Review of Skill Submission

EduSeed 挑战 **C4A「技能提交自动评审」** 交付物仓库。

| 项 | 值 |
| --- | --- |
| 挑战 ID | `ch-20260717031432-5jvqje` |
| 作者 | lenovo（学生 2025105400102） |
| 提交日期 | 2026-10-03 |
| 运行环境 | Windows + Python 3.12.13（零第三方依赖，可离线运行） |

## 一、技能是什么

`skill-evaluator`：读取一份技能提交目录，按**外部量表**（YAML / JSON）自动完成
**完整性检查 → 质量评分 → 红线扫描 → 报告渲染**，每条结论都挂「文件:行 + 原文片段」的证据链。
量表与代码解耦：换一份量表即可评审另一类提交，无需改代码。

```bash
python scripts/evaluate.py --selftest                 # 自检：量表加载 / miniyaml / 官方口径公式
python scripts/evaluate.py "<待评审目录>" --out "<输出目录>"   # 评审一份提交
```

## 二、交付物清单

| 文件 | 说明 |
| --- | --- |
| `lenovo_C4A_skill-evaluator.skill` | 可安装技能包（ZIP；顶层目录 `skill-evaluator/`，含 `SKILL.md` + 脚本 + 3 份量表） |
| `lenovo_C4A_方案设计.md` | 方案设计说明 |
| `lenovo_C4A_教学说明.md` | 教学说明 |
| `lenovo_C4A_评审报告.md` | 评审报告：3 份真实 C4 提交的实跑结果、口径与证据链 |
| `lenovo_C4A_demo.png` | 演示图 |
| `lenovo_C4A_AI日志.md` | AI 使用日志（含真实失败与修正记录） |
| `lenovo_C4A_拿来说明.md` | 使用说明 |
| `reports/` | 4 份真实评审产物（C4 五件套 ×1、官方样板 ×2、C4A 自评 ×1） |

## 三、实测结果（可复现）

| 评审对象 | 口径 | 总分 | 得分率 | 完整性 | 红线 |
| --- | --- | --- | --- | --- | --- |
| lenovo 本人 C4 五件套 | 官方 default | **89.64 / 100** | 94% | 5/5 | 无 |
| skill-explainer | starter | **40.83 / 100** | 41% | 3/5 | 2 项 |
| wechat-doc-mapper | starter | **38.75 / 100** | 39% | 3/5 | 1 项 |
| lenovo C4A 交付物自评 | C4A meta | **91.07 / 100** | 95% | 5/5 | 无 |

三档差距明显，说明评审器**具备区分度**，不是「谁交都一样分」。

## 四、目录

```
.
├── lenovo_C4A_*.md / .png / .skill   # 六件交付物
└── reports/                          # 4 份真实评审报告产物
    ├── 01_lenovo/reports/
    ├── 02_official_samples/b1_skill-explainer/reports/
    ├── 02_official_samples/b2_wechat-doc-mapper/reports/
    └── 03_lenovo_meta/reports/
```

## 五、诚实边界

- 评审器给出的是**结构化证据与分数**，不替代人工终审；教师保留最终裁量权。
- 量表口径与代码分离，分数随量表变化是**预期行为**，不是缺陷。
- 当前盲区：跨提交相似度检测（互相复制）尚未实现，已列为 P1 改进项。
