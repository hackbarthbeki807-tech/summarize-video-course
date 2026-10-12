# 双工具：完整机械证据与一次知识定稿

版本：V3.29.0 ｜ 更新于 2026-10-09

与 WorkBuddy 或其他工具协作时先读。本规则替代要求外部工具写知识摘要、知识候选覆盖表、公式/案例概述及课件知识取舍的旧开工指令；不要求单工具课程制造交接包。旧八文件只读兼容，不能作为新课生产合同。

## 责任和顺序

外部工具把完整来源变得可检索；Codex依据完整来源整理知识，只定稿一次。外部工具不写知识摘要、不写正式七项，不判课件有无知识，不验收正式资料，不修改 outputs 或唯一进度入口。Codex承担配对裁定、知识覆盖、原例、公式、逐页视觉、独立查漏、发布和收尾；作者不能自行验收。

顺序统一为：来源预检与估时 → 完整提取 → 提取质量检查 → 冻结接管 → 完整来源/课件/公式扫描 → 定稿七项 → 独立查漏 → 定向修复复查 → 发布彩排与重开 → 用户验收。替代“先写初稿、以后补画面和公式”的旧顺序。

同机只运行一个重型转写进程，small/int8、CPU 8线程、中文、VAD；不能为省时私自改 tiny。外部工具可以准备下一课，但不改 Codex 当前稿。同一批次未验收，不启动下一批正式定稿；事先授权的证据准备可连续进行。

## 最小证据包 V2

每课工作目录在 work 的 evidence/<课时ID>/，必有 mechanical_manifest.json，包含：

- schema_version 为整数 2；packet_type 为 mechanical_evidence；lesson_id 为目录名。
- sources：每项 kind（audio/pdf）、relative_path（来源根下POSIX相对路径）、sha256；音频有实测 duration_seconds，PDF有 page_count。每单元一段音频、零份或多份PDF；上下段分单元，导论没有PDF时 pages=[]。时长由本地媒体探针取得，不能由最后一个ASR段猜算。
- transcription：path 指完整原始JSON（segments含start/end/text），model=small、compute_type=int8、cpu_threads=8、language=zh、vad_filter=true；speech_intervals 为同一音频独立VAD得到的秒区间，不能照抄ASR区间制造覆盖证明。run_record_path必填，指绑定音频哈希、原ASR哈希、parameters、status=completed、log_path和vad_record_path的JSON；VAD记录绑定audio_sha256、method=independent_vad、engine、speech_intervals和log_path。日志非空并冻结。工具现场打开WAV或通过本地ffprobe复核实际媒体时长；无法探测则阻断，不信自报时长。运行记录不是防伪证明，Codex接管还要核实执行出处。
- pages：每页 source、page（1起）、extraction_status、text_path、image_path、regions、view_segments、needs_codex_visual_review=true。source对应sources中的PDF相对路径。
- index_path 指原文定位JSON；issues_path 指提取问题JSON。全部路径位于本课包内，不能引用包外活文件。按需复制已有原始提取缓存，绝不复制源音视频。

原文索引为 audio_segments=[0,1,...最后段号] 与 pdf_pages=[{source,page},...]，精确覆盖原ASR全部段及所有PDF页。索引只导航原文，不写缩略知识摘要。公式和案例的可检索摘录可保留原文，但不能代替完整上下文。

### 音频无PDF的最小形态

以下是字段示例，哈希/时长/VAD/段号必须用现场实测替换，不可照抄制造ready。原始transcript.json为{"segments":[{"start":0,"end":5,"text":"原始ASR文字"}]}；source_index.json为{"audio_segments":[0],"pdf_pages":[]}；extraction_issues.json为{"issues":[]}。日志仍以补充资产冻结。

```json
{
  "schema_version": 2, "packet_type": "mechanical_evidence", "lesson_id": "L00",
  "sources": [{"kind":"audio","relative_path":"intro.mp3","sha256":"现场SHA-256","duration_seconds":5}],
  "transcription": {"path":"transcript.json","model":"small","compute_type":"int8",
    "cpu_threads":8,"language":"zh","vad_filter":true,"speech_intervals":[[0,5]],"run_record_path":"run.json"},
  "pages": [], "index_path":"source_index.json", "issues_path":"extraction_issues.json",
  "supplemental_assets":[{"path":"run.json","source_location":"intro.mp3","transform":"local ASR run parameters and log"}]
}
```

run.json的最小结构：{"audio_sha256":"现场源哈希","original_asr_sha256":"原ASR哈希","status":"completed","parameters":{"model":"small","compute_type":"int8","cpu_threads":8,"language":"zh","vad_filter":true},"log_path":"asr.log","vad_record_path":"vad.json"}。vad.json为{"audio_sha256":"现场源哈希","method":"independent_vad","engine":"实际VAD引擎及版本","speech_intervals":[[0,5]],"log_path":"vad.log"}。参数和区间须来自真实运行；示例不是可用证据。run_record_path及它引用的日志/VAD自动递归冻结，不需要在supplemental_assets重复列。

工具写入ready的机械结论为segment_count、pdf_page_count、semantic_review_complete=false、needs_codex_visual_review；这几个字段不用于证明实际语义审读。issues的顶层固定为issues数组，空数组表示无已记录提取问题，并非无知识。

PDF区域格式为 {id,kind,bbox:[x0,y0,x1,y1]}，坐标为原PDF点；kind只描述unclassified/body/comment/watermark/contact，不写知识保留/排除结论。完整页至少一个unclassified区域，混合页可拆区域，Codex仍需独立分辨老师正文与留言。view_segments为[{image_path,y_start,y_end}]，使用原PDF点坐标覆盖页首至页尾，可重叠、不留缺口。图片尺寸/纵横比须对应原页或分段，长图必须提供可读分段，不以缩略图算检查完成。

extraction_status只允许ok或no_text进入冻结。ok的文字不得空；failed/partial均阻断。真正无可识别文字的图页用no_text，附empty_text_evidence={checked_image:true,reason:实际检查说明}；它只表示未提取文字，不表示无知识，更不免除Codex看原页。issues=[{id,status,evidence}]；未解决项不允许冻结，resolved必须有修复依据。

补充资产用supplemental_assets=[{path,source_location,transform}]。原始ASR不校写；繁简转换保存另一个视图并记录转换方式、原稿定位及哈希。需要局部恢复时保留原稿，Codex在自己的补充证据区留窗口结果，来源和知识主张仍由Codex核实。

## 工具、冻结和接管

运行项目现有 tools/dualtool_evidence_handoff.py：snapshot、verify、claim、verify-claim。命令通用参数为 --evidence-root、--lesson-id、--source-root、--work-root；claim/verify-claim另需--claim-root。V2不需要--pdf-pages，工具实际打开PDF对页数。使用项目work/venv里的Python，不上传来源。源码运行时所有机械资产必须位于work授权目录。

snapshot只接收V2。旧schema_version字符串2.2.0可verify/claim/verify-claim，不再要求新课补八篇候选文档；旧源格式和哈希算法不迁移。无有效V2清单时不回退旧协议伪造就绪。

新快照位于revisions/<整包SHA256>/。递归资产以POSIX相对路径排序，逐行path:sha256、LF连接、无尾换行，UTF-8求SHA256；算法单独记录，不能拿旧七项顶层指纹计算器替代。所有引用的ASR、OCR、页面图、分段图、索引和补充记录一起冻结，复制前后比对哈希，最后原子切换ready.json。指针包含schema_version、packet_type、revision/package_sha256、manifest_sha256、file_sha256、source_sha256、created_at及机械检查结论。

ready.status=evidence_ready只表示机械可接管，semantic_review_complete=false。哈希、段页覆盖和图片尺寸不能证明语义正确或图字可读。Codex claim记录lesson_id、revision、package_sha256、manifest_sha256、claimed_at；更换指针使旧claim失效，旧快照不原位改。发布前再次verify-claim，修订按可验证差异复查；无法确定影响范围就重新完整复核。

## 提取质量与阅读质量是两道关

机械检查挡模型不符、时间戳非法、重复退化/低置信信号、VAD中可疑缺段、课件漏页/漏尾、空OCR和未决提取项。重复/低置信是待核实信号，不是幻觉判定；不得通过删原文、改阈值或改校验器来“全绿”。发现信号先核对原音频，仅恢复受影响窗口。自动检测不可能证明所有ASR文字正确。

合法重复或已局部恢复的信号可以在transcription.quality_resolutions记{kind,start,end,evidence_path}；kind为repetition/compression/low_confidence/speech_gap。evidence_path指包内JSON，必须绑定original_asr_sha256、audio_sha256、source_time=[start,end]、checked_by、conclusion及status=source_verified或supplemented；supplemented另有replacement_path指包内{source_time,text}窗口恢复JSON。这些依据和恢复文字一并冻结，原ASR不变；Codex写稿与独立复核必须读取每个恢复窗口并核实所宣称的回源依据，不能把这份声明当自动内容通过。无依据或过期依据仍阻断；模型/参数不符、非法时间戳不能用此机制豁免。

接管后Codex全文读转写、逐页以可读尺寸看PDF，疑点定点回听。覆盖清单从完整来源重新建立，不沿用外部摘要或机械段覆盖冒充独立知识覆盖。课件重复支持、独有知识、术语校对和冲突分别融合；纯文字页不堆图，必要图表保存图＋关系解读＋页码。

例子遵守[原例忠实与学生可理解标准](concrete-example-standard.md)。独立复核覆盖每课：全部知识点、公式、每个原例及课件去向；同时查来源细节与概念对应。不能只抽样，也不能给“模型整理”标签后放行自编人物/结局。原课仅有框架就解释框架，案例字段不硬凑。结构检查passed不替代这一步。

修复由Codex定向完成，不把正式稿回传WorkBuddy重写。独立复查受影响位置和关联项，并回扫同批同类问题；连续两轮同类错误先查根因，不重复发整包。已接受的来源疑点与AI未完成的基础工作分开。

## 接续、唯一进度和效率

共享文件变化不会自动唤醒另一工具。开工记录实际执行入口、责任方和下一可验证动作；没有活动执行者时写waiting/checkpointed，不写running。获授权机械链用阻塞式连续执行器，Codex推理仍在活动会话完成；心跳不能代替生产。缺提取证据时写共享问题记录，能本地修的由Codex补充，不让用户逐课复制粘贴。

进度在原固定入口显示本批证据就绪、定稿并独立复核通过、正式发布三个精确集合。state.collaboration中的evidence_ready_units/review_passed_units/published_units为{课时ID:{receipt_path,input_fingerprint,verified_at}}；由执行者现场verify、独立审查和实际发布重开收据填入，不能按目录存在填。tools/dualtool_progress.py只渲染这些已核验收据引用，不负责证明收据内容真实；恢复时必须重新核验文件/指纹。

全部已复核但发布未齐即进入发布自检。未知进度不捏造0%，心跳刷新不表示产物推进。启动链接只发一次，平稳阶段静默；交付、异常或必须用户决定才通知。批次仍经用户验收，整课仍走六层终检与course_closeout.py。

按[验收与协作收尾标准](acceptance-and-efficiency-closeout-standard.md)记录真实Token、耗时、返修和交接；取不到就null/unavailable，字符数不得改名Token。没有同等质量可比基线不声称节省比例。不因本规则升级重写已完成课程，不删除源文件和备份。
