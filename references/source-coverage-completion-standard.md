# 全部来源覆盖与真正完成标准 v3.11

建立课程清单、处理视频画面、融合课件或宣布整门课程完成时读取本文件。本标准解决一个核心问题：`27/27 课时结构通过` 只能证明已发布目录整齐，不能证明源目录里的 PDF、图片和其他资料没有遗漏。

## 一、开工前建立唯一来源清单

- 每门课程必须有 `source_coverage_manifest.json`。首次扫描要覆盖用户授权范围内的**全部文件**，不能只扫描视频或只扫描已经注意到的 PDF。
- 视频、音频、字幕、PDF、PPT、Word、图片、电子书、文本、压缩包和未知格式均需进入清单；未知格式先标 `pending_classification`，不能静默忽略。
- 清单只保存 `source_root_id + 相对路径 + 大小 + 可选 SHA-256`，不保存绝对源目录。
- 多个来源目录分别命名，例如 `videos`、`courseware`。最终终检必须现场重新提供所有必需来源目录，并比较“现场文件集合”和“清单文件集合”。现场多出一个文件或清单文件消失，都阻止完成。
- 小样批准只代表小样通过。例如批准 1 份 PDF 只能记录 `courseware_sample_accepted: 1/7`，不得改写为“整门课程课件完成”。

建议先运行：

```powershell
py tools/course_source_coverage_audit.py inventory `
  --course-name "<课程名>" `
  --source-root videos="<视频目录>" `
  --source-root courseware="<课件目录>" `
  --output "work/<课程名>/source_coverage_manifest.json" `
  --sha256
```

## 二、每个来源必须有最终去向

每个来源只能处于下列一种最终状态：

- `published_integrated`：已发布，且已融合进唯一对应课时。
- `published_course_level`：没有唯一课时映射，作为课程级资料发布。
- `explicitly_excluded`：用户确认排除，并记录具体理由。
- `skipped_non_knowledge`：用户确认是导览、通知、致谢等非知识内容而跳过。

`pending_classification`、`pending_processing`、`awaiting_mapping`、`awaiting_visual_review`、`awaiting_human_review`、`missing_source` 和 `failed` 都不是完成状态。不能用“以后再看”“应该没用”或目录数量抵消这些待处理项。

排除必须记录 `decision.reason` 和 `decision.approved_by: user`。模型可以建议排除，但不能自行把来源从完成分母中删除。

已有正式资料时，先用 `course_source_coverage_audit.py reconcile` 从正式课时和课件索引继承可验证终态，避免重做；它不能自行批准排除项。具体顺序见 [自动对账、知识目录与统一状态标准](automatic-closeout-standard.md)。

## 三、课件数量、页面和融合硬闸门

- 先比较源目录课件总数、已暂存数、已筛选数、已发布数和已融合数。五个数必须分别报告，不能只报其中最大的数。
- PDF/PPT 每一页必须恰好属于 `retained_pages`、`context_pages`、`excluded_pages` 三组之一。三组不得重叠，合并后必须等于 `1..total_pages`；手工统计不一致时自动校验失败。
- 文件名、课时编号或标题可以唯一对应时，必须标 `mapping_status: unique` 并写入对应课时。把唯一可映射课件留在课程级目录，不算融合完成。
- 无法唯一映射时可保留课程级资料，但必须明确标 `ambiguous` 或 `course_level`；不得为了赶进度随意塞进某课。
- 融合不仅是复制课件原件。对应课时的总览、逻辑稿、知识点卡片、案例页、质量清单和 `media_index.json` 应按真实内容刷新，并保留页码/幻灯片来源。
- 根目录总览、知识地图和课件索引必须能导航到正式课件知识稿；AI 检索入口不能只看视频笔记而漏掉课件。

## 四、视频画面只提取真正的知识干货

- 不逐帧保存整门课程。先扫描逐字稿中的“看这张图、屏幕上、表格、流程、漏斗、这里可以看到”等画面提示，再抽取前后关键帧做本地检查。
- 每个视频必须留下画面筛选结论：
  - `completed_selected_key_frames`：发现真正知识画面，保存必要关键图并结构化解读；
  - `not_required_no_knowledge_cues`：没有知识画面信号；
  - `explicitly_excluded_no_knowledge_value`：检查后确认画面没有独立知识价值，并记录理由。
- 有画面提示词却直接标“无需检查”会被终检拦截。
- 表格、流程图、漏斗、思维导图和关系图不能只做扁平 OCR。正式笔记必须保留必要原图，并说明节点、层级、箭头、变量关系、结论和用法；看不清时标“图示需人工核对”，不猜。
- 关键图只保留承载真实知识的画面；界面导航、讲师头像、封面、Logo、广告、课程节奏页和纯提问页不保留。
- v3.11 默认先用 `course_visual_screening.py scan` 识别强知识提示；没有提示的视频可直接记录 `not_required_no_knowledge_cues`，有提示的只抽取候选时间点，不扩张成逐帧存档。

## 五、完成必须分六层报告

课程状态必须分别记录：

1. `structure_complete`：正式课时和课程入口结构完整。
2. `source_coverage_complete`：清单覆盖全部现场来源，且每个来源有最终去向。
3. `visual_coverage_complete`：所有视频完成选择性画面知识筛选。
4. `courseware_integration_complete`：课件完成映射、策展和应有的课时融合。
5. `automated_validation_complete`：链接、页码、哈希、路径隐私、无媒体等自动检查通过。
6. `human_acceptance_complete`：用户明确完成最终人工验收。

只有六层全部为 `true`，才能写 `course_complete: true` 或告诉用户“所有资料都弄完了”。任何一层为 `false` 时，必须准确说出已经完成什么、还缺什么；不得用“自动测试 0 问题”“课时目录数量正确”代替来源完整性。

严格终检示例：

```powershell
py tools/course_completion_audit.py `
  --course-dir "outputs/<课程名>" `
  --strict-v310 `
  --enforce-risk-review `
  --source-coverage-manifest "work/<课程名>/source_coverage_manifest.json" `
  --source-root videos="<视频目录>" `
  --source-root courseware="<课件目录>" `
  --run-status "work/full-course/<课程名>/run_status.json"
```

终检完成后必须运行 `course_state.py` 汇总六层状态。审计指针的输入指纹与当前课程不一致时，状态应标记旧审计为过期，不能沿用旧的“通过”。

## 六、状态文档和依赖预检

- `README.md`、`PROJECT_STATUS.md` 中的“源文件仍保留、已完成、没有下一批”等陈述必须来自当前文件系统复核；旧对话和旧报告只能作为历史证据。
- 课程名必须从当前来源和输出共同核对，P2、P3 或同名课程不得混用。
- 启动批处理或课件打包前先检查依赖和版本；不得假设 `tzdata`、LibreOffice、FFmpeg、OCR 模型或本地模型一定存在。缺依赖时在预检阶段失败，不能运行到中途才暴露。
- 依赖失败、来源目录丢失或人工决定缺失时保留断点，状态写“未完成”；不得伪造完整报告。
