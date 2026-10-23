# 法衡 LexAlign｜法答网 Benchmark 110 条初步任务池

## 打开方式

直接双击 `index.html` 即可使用，不需要服务器。

首次打开后，任务下拉框中应包含 **110 条**法答网 Benchmark，范围：

- `FDW-2026-0031`
- …
- `FDW-2026-0140`

## 本次接入做了什么

1. 从原 Week4 压缩包中发现 `160` 份 `benchmark_sample.jsonl`。
2. 其中 91–140 存在 50 份内外嵌套重复副本。
3. 按 `task_id` 去重后得到 **110 条唯一任务**。
4. 兼容三代数据结构：
   - v1：`gold_laws`
   - v2：`gold_laws` + 更完整的双源核验字段
   - v3：`gold_norms`
5. 原始记录不被覆盖或“自动修正”；统一映射只用于前端展示和人工复核。

## 页面工作流

`Source QA → Query → Gold → Verify → Review`

每条任务可查看：

- 法答网原始问题 / 回答摘要
- 生活化 Benchmark Query
- 法律关系标签与争点标签
- Gold Norms / 旧版 Gold Laws 的统一展示
- 黄金答案与 Gold Path
- 核验状态和原 reviewer
- 人工评分、问题标签、复核人、`draft / reviewed / locked`

## 数据文件

### `data/benchmark_110_unique.jsonl`

110 条唯一的原始 Benchmark 记录。  
这是最接近原项目“机器事实源”的汇总文件；没有把三代 Schema 强行改成一种。

### `data/task_pool_normalized.json`

为法衡前端统一展示生成的任务池数据。  
旧版 `gold_laws` 只做字段映射，不改变原始 JSONL。

### `data/task_pool_manifest.csv`

110 条任务清单，便于快速检查编号、领域、批次、状态、规范数量、reviewer 数和数据代际。

### `data/pool_summary.json`

任务池统计摘要。

## 重要说明

- 当前 110 条原始记录的 `annotation_status` 均仍是原文件中的状态；本打包不会擅自把 draft 改成 locked。
- 人工复核结果默认保存在浏览器 `localStorage`。
- 顶部导出按钮可导出带 `human_review` 的 JSONL。
- 页面内嵌了 110 条任务，因此即使浏览器禁止 `file://` 下 fetch 本地 JSON，仍可直接使用。
- 原包中的 Word / Excel / Skill 等材料没有复制进这个轻量任务池 ZIP，避免重复和体积膨胀；本任务池专注于网页复核与机器数据。


## V1.2 双人复核门禁

本版本将原来的 `draft / reviewed / locked` 升级为：

`draft → review_a_done → review_b_done → disputed → locked`

推荐操作：

1. Reviewer A 独立完成第一轮审查，填写姓名、评分与备注，结论设为 `approved` 或 `changes_requested`。
2. Reviewer B 独立复核，必须使用不同姓名；若同意 A，则设为 `approved`。
3. 如果 B 与 A 对核心结论、规范、条号或题面存在实质分歧，B 选择 `disagree`，任务自动进入 `disputed`。
4. `disputed` 任务必须填写“分歧裁决 / 关闭说明”。
5. 只有以下门禁全部满足时才允许锁定：
   - 四个维度评分全部完成且均 ≥ 4；
   - Reviewer A 已填写并 approved；
   - Reviewer B 已填写并 approved；
   - A / B 不是同一人；
   - 不存在未关闭的阻塞性问题标签；
   - 分歧已经说明并完成裁决；
   - 锁定人已填写。
6. `locked` 后导出 JSONL 会写入 `human_review.lock_owner` 与 `human_review.locked_at`。

阻塞性问题标签包括：

- 引用漏项
- 条号错配
- 双源不一致
- 题面泄露
- 事实不足
- 黄金答案不完整

`版本时点风险` 与 `复核人数不足` 仍会被记录，但不会单独作为永久阻塞；是否关闭需由复核意见说明。


---

# v1.6 预装标注版说明

本版已将 week5 + week6 的 Benchmark 内容直接写入系统任务池，无需再通过“导入 JSON/JSONL”手动添加。

- week5 读取：169 条
- week6 读取：170 条
- 去重并修正冲突后：170 条唯一任务
- 其中 FDW：169 条
- RMSF：1 条
- week6 的 RMSF 银行卡任务内部 task_id 冲突已按文件夹 ID 修正为 RMSF-2026-0001

## 直接标注

1. 解压整个文件夹。
2. 双击 `index.html`。
3. 顶部任务选择器中已经有全部 170 条任务。
4. 点开任一任务后，可直接完成：
   - 四个复核维度评分；
   - 问题标签；
   - 备注；
   - Reviewer A / Reviewer B；
   - A/B 审核状态与说明；
   - 分歧裁决/关闭说明；
   - 提交。
5. 点击“保存草稿”会写入浏览器 localStorage。
6. 点击“导出”可导出整批 JSONL 复核结果。

## 文件职责

- `index.html`：标注/双人复核界面。
- `data/frontend_pool.js`：供页面加载的预装任务池。
- `data/benchmark_170_merged.jsonl`：170 条完整原始 Benchmark 数据。
- `data/import_report_v1.6.json`：本次导入、去重、ID 修复记录。

注意：不要只单独移动 `index.html`。`index.html` 与 `data/frontend_pool.js` 必须保持当前相对目录结构，否则预装任务不会显示。
