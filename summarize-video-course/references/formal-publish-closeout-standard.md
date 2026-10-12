# 正式发布与课程收尾标准 v3.12

正式课时发布、已发布课时修订和全课程收尾时读取本文件。

## 发布身份

生成器必须显式选择：

```powershell
py tools/knowledge_builder.py ... --publication-mode pilot
py tools/knowledge_builder.py ... --publication-mode formal --course-name "<课程名>"
py tools/knowledge_builder.py ... --publication-mode recovery --course-name "<课程名>"
```

`pilot` 不得发布到正式 `课时资料/`。`formal/recovery` 不得使用“未知课程、进行中、课程、course”等通用课程名。正式批次 manifest 必须带真实 `course_name`，恢复批次另写 `publication_mode: recovery`。

## 发布前

- 只从验证通过的 staging 发布；同名正式课时已存在时拒绝覆盖。
- 核对七项结构、时间戳、逻辑稿人读、卡片覆盖、案例扫描、无媒体、无绝对路径和源指纹。
- 课件融合稿必须先通过课件映射与引用检查。

## 修订后的连锁刷新

已发布课时发生阅读层修订后，优先运行：

```powershell
py tools/course_repair_refresh.py `
  --course-dir "outputs/<课程名>" `
  --lesson "<课时名>" `
  --expected-formal-count <数量> `
  --stage "work/course-repair-refresh-stage-<时间>" `
  --backup-root "work/course-repair-refresh-backup" `
  --mirror-root "<批次副本根目录>" `
  --mirror-root "<断点副本根目录>" `
  --run-status "work/full-course/<课程名>/run_status.json"
```

工具以正式课时为权威源，同步明确存在的副本，刷新课程级入口并执行全课程终检。失败时恢复已改副本和课程入口；备份未经用户确认不删除。

## v3.12 交易式课程收尾

用户最终验收已明确记录在来源清单的 `human_acceptance.status: approved` 和 `evidence` 后，使用唯一入口：

```powershell
py tools/course_closeout.py `
  --course-dir "outputs/<课程名>" `
  --manifest "work/<课程名>/source_coverage_manifest.visual.json" `
  --run-status "work/full-course/<课程名>/run_status.json" `
  --expected-formal-count <数量> `
  --source-root videos="<视频来源目录>" `
  --source-root courseware="<课件来源目录>"
```

它在 `work/course-closeout/` 创建锁、暂存、备份和状态日志，按以下顺序推进：

```text
acceptance_recorded
→ closeout_locked
→ catalog_and_handbook_staged
→ root_navigation_staged
→ strict_audit_passed
→ final_links_reopened
→ course_complete
```

- 不得使用 `course_finalize.py`、`course_state.py` 或手工编辑 JSON 宣告 `course_complete`；它们只生成导航或状态材料。
- 收尾将 `knowledge_catalog.jsonl`、`08-课程核心知识手册.md`、`07-课件资料索引.md`、课程总览、根索引和 `course_state.json` 作为一个可恢复发布单元。
- 任何严格终检、最终链接复开或写状态失败，必须恢复上一个完整的课程级版本；失败日志保留在 `work/`，不得写课程完成。
- `07-课件资料索引.md` 必须索引必要关键画面；没有独立课件或必要画面时，明确写“本课程无独立课件或必要知识画面”。
- 返回成功后再按 [验收与协作收尾标准](acceptance-and-efficiency-closeout-standard.md) 核对最终首页、唯一状态、固定进度入口、当前批次页和用户验收口径；发现旧状态时定点修复并重跑严格终检，不能把工具成功直接当作用户可见收尾完成。

## 新课程严格终检

终检前固定执行：来源清单自动对账、强提示画面候选审查、`course_knowledge_catalog.py` 生成 AI 目录和课程知识手册。完整命令见 [自动对账、知识目录与统一状态标准](automatic-closeout-standard.md)。

```powershell
py tools/course_completion_audit.py `
  --course-dir "outputs/<课程名>" `
  --run-status "work/full-course/<课程名>/run_status.json" `
  --strict-v310 `
  --source-coverage-manifest "work/<课程名>/source_coverage_manifest.json" `
  --source-root videos="<视频来源目录>" `
  --source-root courseware="<课件来源目录>" `
  --enforce-risk-review
```

严格终检检查：

- 课程根目录名、总索引课程名和每课课程名一致。
- 每个正式课时是 `formal` 或 `recovery`，无小样/暂存/待发布残留。
- 计划正式知识课数量与实际目录一致；导览和前言另列跳过，不混入分母。
- 七项、链接、知识地图、无媒体和路径隐私通过。
- 高风险课时有人工审读留证。
- `run_status.json` 可以是收尾中间阶段（如 `acceptance_recorded` 或 `strict_audit_passed`），但必须 `active_pid` 为空、没有虚构下一批；严格终检不能循环要求先写 `course_complete`。
- `source_coverage_manifest.json` 与现场全部来源逐项一致；视频、PDF、图片及其他文件没有漏项。
- 视频画面知识完成选择性筛选；课件完成唯一映射、页面唯一分类、策展发布和必要融合。
- 完成状态按结构、来源、画面、课件融合、自动验证和人工验收六层分别报告；六层没有全部通过时不得写 `course_complete`。
- 终检写入带输入指纹的 `latest_audit.json`。目录、手册、资料索引、课程总览、课程状态、知识目录行数与卡片数、目录链接及首页固定入口均是严格终检项；输入变化导致旧审计过期时必须重跑终检。

完整规则见 [全部来源覆盖与真正完成标准](source-coverage-completion-standard.md)。历史课程可先用非严格模式读取；但只要正式目录明确写成 `pilot` 或残留小样标签，仍必须修复。v3.10 起的新生成课程一律使用严格模式；v3.12 继续沿用 `--strict-v310` 命令行参数以保持工具兼容，并增加交易式收尾、课程首页硬入口与可见进度规则。
