# 自动对账、知识目录与统一状态标准 v3.12

课程进入正式批次或课程收尾时读取本文件。本标准把“散落在多个报告里的事实”变成可重复执行的本地步骤；它提高效率，但不降低来源、画面、课件或人工验收门槛。

## 一、先自动继承已有正式证据

旧的 `source_coverage_manifest.json` 不能因为状态仍是 `pending_classification` 就让已发布来源重新处理。先运行：

```powershell
py tools/course_source_coverage_audit.py reconcile `
  --manifest "work/<课程名>/source_coverage_manifest.json" `
  --course-dir "outputs/<课程名>" `
  --decisions "work/<课程名>/source_decisions.json" `
  --output "work/<课程名>/source_coverage_manifest.reconciled.json"
```

自动对账只允许继承两类可验证事实：正式课时 `media_index.json` 已记录的来源，以及正式 `courseware_index.json` 已发布的课件来源。它不能根据文件名“像导览”而自行排除来源。

排除或跳过必须由 `source_decisions.json` 明确记录 `approved_by: user`、具体理由和终态 `explicitly_excluded` / `skipped_non_knowledge`。模型建议、旧推测或重复候选只产生提示，不直接修改完成分母。

## 二、画面筛选只保留知识候选

```powershell
py tools/course_visual_screening.py scan `
  --manifest "work/<课程名>/source_coverage_manifest.reconciled.json" `
  --course-dir "outputs/<课程名>" `
  --output "work/<课程名>/source_coverage_manifest.visual.json"
```

- 没有强知识画面提示的已转写视频，可进入 `not_required_no_knowledge_cues` 终态。
- 只有出现表格、图表、漏斗、思维导图、公式、屏幕文字等强提示的时间段才进入候选队列。
- 候选帧只写入 `work/`。人工或视觉模型必须逐候选写选择结论；未审查候选不能伪装成“无需画面”。
- 正式输出只保留真正承载知识、且文字无法完整替代的必要原图；封面、Logo、广告、讲师头像、纯提问页和重复画面不保留。

## 三、知识整理使用动态窗口与局部恢复

- 覆盖清单窗口根据逐字稿的知识信号密度自动选择；清单、公式和步骤密集的内容使用更小窗口。
- 输出预算按当前窗口行数计算，不使用所有课程共用的固定低上限。
- 缺少结束标记、最后一条残缺或解析失败时，绝不接受“完整前缀”；只拆分当前失败窗口，保留所有已验证窗口和正式课时。
- 环境变量 `COURSE_CARD_INVENTORY_CORE_LINES=auto` 为默认值；只有兼容旧任务时才人工指定整数。

## 四、缩小人工审读范围

风险队列只包含仍未解决的事实风险，例如“待人工核对”、来源冲突、专名/公式未确认、疑似有案例却未提取。质量清单中的通用 ASR 说明、已经修正并留证的错词、以及“已完成人工语义复核”本身不再重复触发高风险。

结构化留证推荐写入课时 `media_index.json`：

```json
{
  "quality_review": {
    "status": "completed",
    "reviewer": "user_or_human",
    "reviewed_at": "ISO-8601",
    "notes": "本课明确风险已核对"
  }
}
```

用户接受“保留标注”的资料时，`human_read_verified` 仅代表已阅读，不能推断为已回听来源。整课验收应在来源清单区分 `acceptance_mode`、`unverified_items` 和 `source_recheck_status`；字段与完成后的首页、进度一致性检查见 [验收与协作收尾标准](acceptance-and-efficiency-closeout-standard.md)。

## 五、生成 AI 目录和人类手册

```powershell
py tools/course_knowledge_catalog.py --course-dir "outputs/<课程名>"
```

- `knowledge_catalog.jsonl`：每张正式知识点卡片一行，使用稳定知识 ID、相对来源文件和原卡片字段，供 AI 检索；不二次发挥课程事实。
- `08-课程核心知识手册.md`：按课时列出“本课学会什么”、知识点标题和真实逻辑稿链接，供人快速浏览。
- 正式卡片变更后必须重新生成二者；目录行数应等于全部正式卡片数量。

## 六、只认一个课程状态

```powershell
py tools/course_state.py `
  --course-dir "outputs/<课程名>" `
  --manifest "work/<课程名>/source_coverage_manifest.visual.json" `
  --run-status "work/full-course/<课程名>/run_status.json" `
  --source-root videos="<视频来源>" `
  --source-root courseware="<课件来源>" `
  --output "outputs/<课程名>/course_state.json"
```

`course_state.json` 是当前六层状态入口。课程终检的 `latest_audit.json` 必须记录输入指纹；正式课时、来源清单或索引变化后，旧审计自动变成 `stale`，不能继续被引用为“最新通过”。

推荐收尾顺序固定为：`reconcile → visual scan/decision → 用户验收留证 → course_closeout`。`course_closeout` 内部生成目录、手册、资料索引、根导航并对实际发布结果做严格终检与状态刷新。任何一步未终态，只报告已完成层和待处理层，不宣布整课完成。
