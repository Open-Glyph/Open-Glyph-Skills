---
name: guwenzi-read
description: Read and explain ancient-character papers and excavated-text material with separate text and glyph evidence. Use with guwenzi-remote for atomic tools; do not force a fixed OCR pipeline or use this skill for glyph composition and paper typesetting.
---

# 古文字论文阅读与证据整理

## 竖排文献：方向、分段与对齐

先看整页确认纸面是否正立，再确定每个区域的 writing_mode：vertical-rl 为右列先、列内自上而下，vertical-lr 为左列先；横排用 horizontal-tb。不要靠长宽比认定方向，也不要为读竖排默认旋转原图。layout_analyze 若支持 writing_mode 则显式传入；单列 split_line 用 direction=vertical-ttb，裁框仍是未旋转原图坐标。投影只提出候选，分离部件、点画、夹注仍由 Agent 合并检查；窄小候选不能直接丢弃。

先按页→段落/简号→照片段/正文列→字序组织。右列优先只在同一阅读区域内适用；标题、上下段、双行夹注、照片和释文要分组，不把全页按 x 排序。先查 live schema；有 reading_plan 时，用 regions 的 section_id、writing_mode、原像素 bbox、units 与 section_order 保存计划。多个段落须显式 section_order；整页取图仍用 detail=image，不为阅读强制全页切字。

照片与打印释文的字距不同，不按相同 y 值配对。reading_plan.alignments 的 source_ids/target_ids 给逻辑序列，anchors 明确匹配、gap、compound 或 annotation；支持空一侧和 1:n。缺字框、省略号、合文、重文、残迹各保留原序号；未对齐区间单列，绝不只因两边数量相等就逐项 zip 或吞掉缺字。几何和调用者锚点都不是古字身份确认。

字体验收不用 DINO。若 live schema 有 glyph_verify，批量输入原排 source_asset_id、最终 font_asset_id 和明确 codepoint（含 PUA）；查看实际字体差异图。needs_review 表示需精查，不是识字判决或任务执行失败；保留可打印校样并修订，不能标成已验收，也不因风格/扫描差异盲目重试。ExportReport.sections 可指定 writing_mode，文字内容始终按阅读顺序保存、不倒置字符串；PDF 为逻辑顺序横排，HTML 可竖排预览。


先加载 `guwenzi-remote` 了解工具契约。阅读过程由 Agent 判断下一步，不要把“上传—整页准备—批量切槽—逐槽下载”当作默认流水线。

## 两类文献

### 文字型 PDF

保留并优先阅读原生文字层、字体、PDF path、内嵌图片和坐标。普通文字不需要重新 OCR。遇到无编码 path、来源编码异常、图片字或文字层与页面不一致时，再把相关局部转成图像，调用 `crop`、`glyph_search`、`glyph_register` 或 `glyph_compose`。

原文、作者释读、Agent 修订和字形身份必须分开记录；不要用释读覆盖原文字层。

`detail=image` 是轻量取页和文字 trace，不代表已经提取全部字体或 path。需要这些原生资产时，再在同一 mode/DPI 下请求相应原生证据；不要只因先看了页图，就把有原始曲线的字改成位图描摹或重新造字。

### 图片型 PDF / 图片 / Office 页面

先检查 `tools --server`。新服务用 `page_prepare` 的 `detail=image`（CLI 0.3.6+：`page image`）只取页图和原生文字证据，再按需调用 `layout_analyze` 的 `regions` 模式。已有清楚的段落图或明确裁框时可直接阅读，不为看清一段而生成整页字槽。YOLO只负责提出文字、图、表、图注等区域；图表按顺序保存，不强迫识别。Agent查看放大的文字区域后：

1. 能读出的文字直接输出整块文字；
2. 不确定某个字才选择局部框并 `crop`；
3. 仍不确定才 `glyph_search`；
4. 检索未命中时，回到原句、括注、作者明说的字名和清楚构形，并查正式 Unicode/GlyphWiki；检索失败不等于无编码；
5. 身份未定时保留原图与 `glyph_register` 身份；需要排印时先 `glyph_lookup` 复用整字/部件，再 `glyph_compose` 传 glyph、ids 或 structure，不编写底层 KAGE 或逐笔坐标。DINO 选中的公开 gwz 编码可以进入 lookup，但只有有对应程序且快照相符的 GlyphWiki 项能直接组字；
6. 最后用 `transcribe` 保存完整文字区域及结构化字形引用。

不要默认生成所有字符槽，也不要把槽位边界当成字形真值。

一段里有多个疑字时，先集中列出原图坐标，再用 `crop` 的 `items`、`glyph_search` 的 `asset_ids` 批量取得证据；参数以远端 schema 为准。一次查看带编号的候选拼图后，仅对仍不清楚的项取原图放大。原文、字体与映射表最后统一整理，避免每识一个字都重新排版整份文档。批量不能省略逐项证据或把相似分数变成确认结论；同 Unicode 也不意味着可以合并不同原形。

新实例第一次检索可能包含模型和索引缓存准备。已取得任务 ID 后，通过 `status` 继续等待，不因终端等待超时重新提交整批；缓存命中很快，也不能据此承诺所有新疑字都同样快。

## 定位、投影与最终裁框

- 最终单字 bbox 由 Agent 查看完整行、邻字、括注和部件后提出，再用 `crop` 按原图像素执行；裁切工具本身不识字。
- 投影适合已经纠偏、方向一致、行列和字距稳定、有连续白带的打印正文。优先用于分栏、分行和等宽字格的候选槽位。
- 脚注号、上下标、夹注、标点和比例字体只能把投影当建议；须合并孤立点与分离部件，并参考邻字间距。
- 拓片、摹本、楚简照片、粘连污损字形不让投影决定单字边界。推荐流程是：版面区域 → 纠偏 → 行列投影 → 字槽候选 → 部件合并 → 视觉复核 → 原像素裁切。

## 证据纪律

- `asset_id`是文件/图像字节身份；`span_id`是位置建议；候选`c1`是一次搜索内的临时编号；`gwz:`是具体字形资产身份。
- Unicode、Seal、PUA、字体 GID、字形身份和释读不是同一层信息。
- 不做 NFC/NFKC、繁简转换或静默码位替换。
- 原图和放大图都保留；放大图不增加位图信息。
- 无法确认就写“未解/待核”，不要用上下文强猜。
- YOLO类别、投影边界、OCR输出、DINO排序和模型选择都必须标记其证据状态。
- 但“保守”不等于忽略直接文字证据：原句明确写出“释为某字”且原排形相符时，可据两者确认；同时保留失败检索，说明相似度没有识别成功。

## 解读写作

每一条解读区分：

1. 页面实际可见内容；
2. 论文作者的释读和论证；
3. Agent 根据字形、文例、文献和候选比较作出的推断。

HTML/PDF解读应附原页图、相关原字形图、页码、block/asset 引用和不确定项，并明确 PDF 物理页与书内页码及跨页断句。不要只输出一串现代 Unicode 字符而丢掉原始字形。

用户要求可复制、可编辑或打印版时，规整打印体必须交付文字层，不能只给图片：已确认字符直接保留正式 Unicode；确需重建的形用结构化部件组字，不能确认标准码位时使用字体局部 PUA，并随附字体与 mapping。支持打印字体的新服务会返回 glyph_compose.print_font；检查状态并交付字体/映射，批量使用顶层合并包，不能把单字 PUA 码直接串联。原生 DOCX/PDF 中已经有编码的文字不得因渲染或检索过程退化为正文图片。能力不足或身份未定时明确列出未完成项，不声称已经实现可编辑排印。正文显示文字字形，原排图保留为可回看的证据。甲骨、金文、楚简原形仍保留原图，不以现代楷形或重建字体替代证据。

忠实照录原排时，正文中每个输出字形与符号都必须和对应原排比较，普通 Unicode 不能免检。比较最终字体/文档的实际渲染，而不只是 KAGE 预览或来源哈希；DINO 高分不能自动放行，尤其要查短点、弯笔、封口和连接差异。发现不符按 `guwenzi-compose` 的逐项验收流程修订；不能用“字体差异”掩盖不同构形，也不能把未通过的可打印候选称作完成。检查器的误拦与仍未解决项应如实报告；这不要求为普通阅读预先切完所有字槽。

`transcribe` 只是保存 Agent 已经完成的阅读结果，不是 OCR，也不会自行判断哪些字需要检索。
