---
name: guwenzi-read
description: Read ancient-character papers with separate source, glyph and interpretation evidence; capture printed forms and photo-to-print pairs using the website detector-owned ledger or the available external CLI. Not for ordinary non-palaeographic papers.
---

# 古文字阅读与逐字采集

先遵守 guwenzi-remote 的当前环境契约。网页出现TaskState/PreparePage时走托管采集，**不用布局模型，不让语言模型写bbox或裁字**，不适用外部原子CLI的手工框流程。

## 任务范围与身份

“隶定释文／照原录／打印体／原字形／字形表／逐字对应”及此前错字的复核属于严格采集，不是概括论文。心理预期是研究罕见字、异构与无编码形；不能以普通OCR或语义补全替代。普通概括论证无需为每个字造字或验收。

以用户指定PDF物理页为范围，书内页码另记。缺少配对的一侧时说明，不猜书籍编排或擅自寻找别页释文。原始文字层、图片、作者括注和Agent推断分开保存。原生字体/path有可靠证据就保留，不为了增加工具调用重建原字。

## 网站逐字流程

TaskState → PreparePage（总览＋当前批次）→ 看这一批并立即RecordReadings → NextCandidates接续，直到全部候选初录 → SearchCandidates/glyph_lookup查证 → 结构造字与配对 → ExportCapture交付字体正文。字形比较另作按需质量诊断，不是流程结束前的必过步骤。

不要为“先全面了解”反复翻完所有局部才开始记录。照片侧先保存photo角色、text=null；未知打印体保存null和具体note，不能为了推进伪造文字。当前批次的图和最新工具结果受上下文保护；整理后接着pending_batches写回，不重新取页。原样复制返回unit_id，不能另编u001或只写c1。此处初录不是验收完成，疑字后续必须按证据修订。

坐标、文档/页面ID和逐字台账由程序拥有。模型只对已有unit_id作阅读、角色标注和配对。漏检不补框，错框写note，不能自行用Python或投影另裁。未识text=null保留原图与稳定来源标记，不留一个看不见的空格或偷换熟字。单字候选可能实际是合文、部件或噪声，检测编号不等于字符真值。

每个print/photo/annotation候选都要有角色或明确未定原因；不要只处理看起来容易的字。多个疑字批量查，同字的不同排形不自动合并。同一原图哈希可以复用检索，不重复生成整篇文档。

程序按方向和几何列给出初始字序。vertical-rl为右列先、列内从上到下；不倒置字符串、不旋转原图。混排分段/夹注不能靠全页x排序确定：保留column_id、unit_order与具体疑点，展示分列校样，不冒称顺序已确认。

照片配对使用photo_unit_ids；允许缺项、1:n和残迹。不能按相同y或数量相等直接zip。原图与打印体每行使用真实裁片；长条照片配长串文字不是“一字一图”。

## 检索、打印和验收

罕见、非标准、被纠错或有一点一画差异的形都要看字库实图。搜索没有命中不等于没有Unicode，检查作者明示字名、构形、变体和部件。Unicode编码、原形身份和释读是不同问题，不能用释读字覆盖原排。

需要可输入排印时依guwenzi-compose复用整字/结构化组字；已确认字保留正式Unicode，其他保留gwz身份与对应字体/映射。不要把PNG称作已完成的字体交付。

需要精校时可比较最终字体的实际渲染，包括普通Unicode和符号。没有比较、差异未解决或检测遗漏只记录为质量情况，不影响任务完成或交付，也不宣称全对。检查器给出具体编号与可选建议，不能为凑覆盖率重复调用或拒绝交付可用结果。

ExportCapture从台账生成字体正文、字体包与独立核对表，不手工重写位置表。CaptureAudit不认证学术释读，也不决定任务能否完成。最终给真实下载链接，质量情况与任务完成状态分开；不得为了等“全部验收通过”而拖延交付。

## 解释写作

解读分别写清页面所见、作者主张与Agent推断。HTML/PDF附相关原图、物理页和证据编号；稀有字保持文本层与字形层双层记录，不做Unicode规范化或繁简转换。文字型PDF优先保留原文，不一律重新OCR；普通论证阅读不必全页逐字验收。外部CLI按guwenzi-remote的独立参考执行，不能把网站未暴露的工具当作可用。
