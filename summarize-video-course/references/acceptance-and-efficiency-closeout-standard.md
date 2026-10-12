# 验收与协作收尾标准 V3.28.0

仅在条件验收、整课收尾或 Codex 与外部工具协作时读取。本标准补充现有六层终检，不降低知识与来源复核要求，也不要求为单工具课程制造协作报告。

## 一、把“接受资料”和“核实来源”分开

用户的“通过”可能有不同含义，不能统一改写成已核实。来源清单的 `human_acceptance` 保留 `status: approved` 以兼容现有收尾工具，同时增加：

```json
{
  "status": "approved",
  "acceptance_mode": "accepted_with_labeled_uncertainties",
  "evidence": "用户原话及可追溯日期或消息位置",
  "unverified_items": ["课时 ID、疑点及仍可查看的质量清单位置"],
  "source_recheck_status": "not_performed"
}
```

- `accepted_with_labeled_uncertainties`：用户接受疑点继续标注的可用资料，不代表疑点正确、音频已回听、图表已重新识别。`unverified_items` 不得为空，疑点必须仍出现在相应质量清单和交付说明中。
- `fully_verified`：只有逐条回到音频、视频画面或课件核实并留下时间戳/页码证据时使用；用户表示“看着可以”不足以写此值。
- 若用户只通过某批或某课，不得写整门课的 `human_acceptance.status: approved`。验收范围和版本必须对应实际交付文件。
- 历史包中的 `human_read_verified: true` 只表示读过文件，不作为 `source_recheck_status: verified` 的证据；保持旧字段兼容，不批量重写已验收课时。
- 质量清单无未解决的基础漏项仍是前置条件。“允许保留的来源疑点”和“AI 尚未做完的工作”不能混同；后者不能通过条件验收绕过。

## 二、收尾一致性是完成闸门

`course_closeout.py` 返回成功后，仍须对**最终磁盘文件**执行一次自检；不以工具退出码或旧审计报告代替。至少核对：

1. `course_state.json` 六层均完成；`run_status.json` 为 `course_complete`、无活动进程、无下一批；最终审计指纹仍对应当前文件。
2. `00-课程总览.md` 使用最终状态文案，不残留“等待最终验收”“收尾进行中”等旧状态；课程手册、课件索引、推荐逻辑稿均能从首页实际打开。
3. 课程固定 `99-处理记录/00-实时进度.md` 显示完成、真实课时数、无下一批；当前批次页也应明确已通过验收或指向整课完成。历史批次页保留历史状态时须标明是历史记录，不能作为当前入口。
4. 首页、状态、进度和对用户的完成消息都使用同一验收口径。若接受方式是 `accepted_with_labeled_uncertainties`，必须明确列出仍待核实的疑点及入口，不能写“全部来源内容已证实”。
5. 任何一项不一致，状态记为 `closeout_needs_repair`，在 `work/` 留下差异与待修复文件；只修受影响的课程级入口，重新运行严格终检并实际重开，不重转写或重做已验证课时。修复前不得向用户说“全部完成”。

进度页可能包含用户有用的旧批次链接和来源说明，不得为改一个状态整页粗暴覆盖；先备份，在 `work/` 生成差异并核对链接，再安全更新。备份未经用户确认不清理。

## 三、双工具只记录能证明的效率

只有使用外部工具分担任务时，随批次记录一份简短的 `work/<课程>/<批次>/collaboration_metrics.json`，在接管时和发布后各更新一次：

```json
{
  "scope": "批次 ID 与课时集合",
  "evidence_version": "冻结版本与清单指纹",
  "division": {"external_tool": "来源、转写、课件证据", "codex": "定稿、独立复核、发布"},
  "codex_input_tokens": null,
  "external_input_tokens": null,
  "token_measurement": "unavailable",
  "elapsed_minutes": null,
  "handoff_rounds": 0,
  "repair_rounds": 0,
  "omissions_found": 0,
  "comparable_baseline": null,
  "savings_claim": "not_measurable"
}
```

- 能从工具账单或运行记录读到 Token 时填实际值与证据位置；读不到就保留 `null/unavailable`。文件字节、字符数、模型输出字数只是代理量，不得命名为 Token。
- 记录总历时、交接轮数、返修轮数和遗漏项，计算口径注明是否包括等待用户验收、人工复查和重复工作；没有同课程类型、相近时长与同等质量闸门的基线，就只报告“投入结构已记录，节省额度无法定量”。
- 证据准备复用、已验证转写不重做、只交冻结差异、Codex 集中定稿与独立复核，是默认省重复工作的做法；不能为了少用 Token 省略逐页视觉、公式、案例、来源核查或最终重开。
- 这份效率记录不属于正式七项资料，缺少 Token 统计不会阻断课程知识交付；但不得在未完成记录时对外声称“节省了 X%”。
