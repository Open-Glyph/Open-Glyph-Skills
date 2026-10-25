---
name: guwenzi-compose
description: Compose or reconstruct ancient-character glyphs and write evidence-based palaeographic papers by reusing catalog glyphs or assembling IDS/components, with printable fonts and verifiable evidence. The agent never authors low-level stroke programs. Do not use this skill for ordinary reading alone.
---

# 古文字组字、重建与论文排版

## 线上直接图形复检（以 live schema 为准）

本流程不使用 DINO 做字体验收。glyph_compose 带 hint_asset_id 时检查 print_font.visual_acceptance；ready 只说明文件可下载。已有最终字体或批次合并字体，用 glyph_verify 的 items 批量传 source_asset_id、font_asset_id、codepoint。不要把单字字体的 PUA 地址套到合并包；以最终 cmap 为准。原图和字体必须属于当前任务。

glyph_verify 直接比较最终字体渲染并返回 difference_asset、render_asset、evidence_asset。pass 仅是所列图形容差内通过，不是字符身份认证；needs_review 则具体看差异，修正选字、部件或布局后重查，不把它当工具异常重发，也不把所有粗细差异都判作错字。保留候选的可打印文字及原图，禁止将未通过结果标为精确复原。没有此工具时说明服务尚未更新，不虚构成功。

竖排页面的字形保持正立；先按段落/简号和列方向确定 source occurrence，再做字体映射。同 Unicode 的异形不合并来源身份。missing_mark、omission_mark 和分段边界保留序号，符号也参与最终字体检查。原形、PUA 地址和文献释读分层保存。


本 Skill 只在用户要求组字、无编码字形重建、字体/字形表或古文字论文写作排版时使用。阅读文献和按需检索先使用 `guwenzi-remote` 与 `guwenzi-read`，不要预先批量处理整页或所有槽位。

## 字形层必须独立保存

始终区分：原始图片/path、来源字体/码位、`gwz:`具体资产、KAGE重建程序、派生字体、作者释读和 Agent 推断。任何派生 SVG、PNG、OTF 或现代字都不能替代原件。不要做 NFC/NFKC、繁简转换或静默码位替换。

## 优先复用，再按结构组字

AI 负责看图、选字/变体/部件、判断上下左右关系与复核；程序负责部件依赖、布局、全部笔画坐标和字体生成。不要编写 KAGE 数字串、自包含 table/parts、kage_json 或逐笔描画，也不要求沙箱安装 Node、KAGE 或整个字库。

1. 有原始字体/PDF path 时优先保留原始曲线。已确认字符保持正式 Unicode，不因造字改成释读字。
2. 可疑字先用已有 glyph_search 做 DINO 图像检索并查看候选；选定后 glyph_select 给出的公开 gwz 编码可交 glyph_lookup。能关联到 GlyphWiki 且来源快照相符的候选可以直接复用。图片字库与 KAGE 程序库不是一回事，非 GlyphWiki 图片不保证有可组字部件。
3. 已知字或部件用 glyph_lookup 查询，查看实际预览后选 glyph_ref。有合适整字/异形时直接 glyph_compose(glyph=...)，不要拆成笔画重造。
4. 无合适整字才传 IDS（如 ⿱艹⿰日月），或包含选定部件引用的结构树。由服务自动解决部件与依赖；比例需修正时用 weights，换形时换 glyph_ref。
5. 需要从现成字取部件时，用 glyph_lookup(mode="parts")。工具返回候选预览和可用的结构模板；模板中的 boxes 由程序提供，通常只需替换 children 中的 glyph。
6. 找不到部件时查更小的已有部件，保留具体缺口和原图，不偷偷换成熟字、不用空白冒充完成，也不退回让 AI 写整字底层笔画。拓本、手写、残损形不强行变成现代排印体。

## 结构化工具（先检查 live schema）

必须有 glyph_lookup，且 glyph_compose 支持 glyph/ids/structure；旧服务缺这些字段时说明需要更新，不虚构调用，不回退到旧配方教程。

~~~sh
guwenzi-tools glyph lookup 旣 --output work/variants
guwenzi-tools glyph lookup u65e3-j --mode parts --output work/parts
guwenzi-tools glyph compose --ids '⿱艹⿰日月' --output work/composed
~~~

网页沙箱直接调用已暴露的原子工具，不为这些步骤安装 CLI。DINO 与字库查询在服务端执行，不给每个沙箱复制模型或整库。

调用示意（引用必须替换为本次 lookup 的真实结果）：

~~~json
{"glyph":"<glyph_ref>","hint_asset_id":"<original_asset_id>","unicode_codepoint":"U+65E3"}
~~~

~~~json
{"structure":{"op":"⿰","children":[{"glyph":"<left_ref>"},{"glyph":"<right_ref>"}],"weights":[1,2]},"hint_asset_id":"<original_asset_id>"}
~~~

glyph、ids、structure 三选一。上下/左右/包围/交叠等为几何提案，不自动证明字符身份；不支持的扩展 IDS 应报告，不能删掉选择符凑成可用结构。批量用 items，每项仍带自己的原图和可选确认码位。先看合并校样，只修有差异的项，最后统一字体与全文排版。

gwk: 引用固定的是字库程序及完整依赖闭包，gwz: 指具体字形资产，二者不等同于 Unicode 或释读。部件版本变化需重新看图选取；服务保存内部 KAGE 配方作复现证据，但不要把它重新交给 AI 逐笔编辑。

## 可排印字体输出（先检查服务 schema）

支持 `unicode_codepoint` 的 `glyph_compose` 会同时生成 `print_font`：OTF、TTF、mapping、可复制文字、内嵌字体的 DOCX 校样和 HTML 校样。已确认的标准字在请求中明确传如 `"unicode_codepoint":"U+655A"`；不要从 root 名称自动推断身份。省略时仅分配字体局部 PUA，不等于确认该形无 Unicode，也不允许用猜读填写码位。旧服务无这些字段时说明能力缺口，不把 PNG/SVG 当作已完成的字体交付。

检查 `print_font.status`，下载并交付 `otf_asset` 或 `ttf_asset`、`mapping_asset`、`text_asset`，需要 Word 时附 `word_sample_asset`；`asset` 仍是视觉校样，不能代替这些字体产物。CLI 可用 `asset ASSET_ID --output glyphs.ttf`；网页环境用 FetchAsset、PublishFile。OTF保留三次曲线，TTF做有误差记录的曲线转换，不做位图描边。

多字优先使用批次顶层的合并 `print_font`，不要把各单字字体的 U+E000 直接拼成一段；合并包才统一分配不冲突码位。同一字体包映射可复用，跨字体包的 PUA 不是全局编码。同码位异形冲突、部分失败或未生成字体要明确报告，不静默改码。

要求可编辑/打印文字时，验收包含：字符可复制、cmap与mapping一致、Word/PDF使用了对应字体、原图仍可回看。已编码正文不得仅以图片交付。字体嵌入不等于内容版权授权，学术核对状态仍单独保留。

## 原排复现的逐项字形验收

本节用于逐字照录或忠实复现原书打印体，不用于新写说明文字。所有被照录的位置都要检查，普通 Unicode、PUA、缺字框及省略符号均不免检。相同 Unicode 不等于相同排形；弯笔、点画、封口、连接不同不能一概解释成“字体风格”。

验收对象必须是最终 OTF/TTF 或最终 HTML/PDF/DOCX 中实际使用的字体渲染，不能只比较 SVG 预览或检查哈希。原图、最终字体、实际渲染、码位与位置都应绑定；任何一项改变均重新计算，不能沿用旧的“已核对”标记。最终文档仍须检查字体回退与裁切。

DINOv2 余弦分数只用于筛查/排序，0.95 不是正确率，也不是放行凭证。高分仍要做局部缺笔/多笔、连接与开闭结构比较；低分也可能来自比例、粗细或扫描噪声，不能直接判错字。保留对齐规则、各项指标、差异图和门槛，禁止通过局部弹性变形、删除疑似小点、放宽门槛来掩盖差异。二值化可用于诊断，不把二值描边替代 KAGE 造字。

已安装的本地工具若在 `--help` 中提供 `verify-font`，可批量提交最终字体与逐位置原图；以其实际退出码和证据为准。没有该能力时不得声称自动验收通过，也不得假定远程已发布同名工具。`print_font.status=ready` 仅说明字体文件可用，不说明原形一致；检查独立的 `visual_acceptance`。原生曲线提取也需验证最终字体转换结果。

发现结构差异就修正选字、部件或结构布局，再渲染最终字体、重新比较，只重做受影响项。保留可打印候选、原图与稳定来源 gwz 标识，不用空白替代字形；但未通过或证据不足只能标为未完成校样，不能冒充验收结果。反复迭代没有改善时说明未解决差异，不盲目死循环或承诺从模糊扫描恢复唯一原字体。严格比较器也要用漏点、错部件及同形轻微模糊的正负对照检验，公布误放和误拦，不能仅凭“全被拦住”就宣称可靠。

## 论文论证

每个新释读至少分开讨论：字形依据、同文/文例依据、文献旁证和反例检查。正文必须能回到页码、原图、字形资产和出处。括注释读不能替代原形；相似字形不能自动证明同字同音。

## 字体编码

- 有正式 Unicode 的字继续映射正式码位；不要为了保存论文中的每一个排形而全部改成 PUA。
- 只有无正式码位、但确需输入的字形/部件落入字体局部 PUA。PUA 只是该字体 `cmap` 中的调用地址，不是官方字符身份。
- 同一字符的论文原排异体若需精确区分，可另保留字形实例和原图；语义文本层仍优先使用正式 Unicode。
- `build-font` 若默认把所有输入分配到 PUA，最终组装时要把已确认字符恢复到正式码位，并在 mapping 中只留下真正无编码项的 PUA。

## 排版交付

交付原始图/path、程序生成的内部 KAGE 证据、组件版本、逐字 SVG、可选 OTF、字体 SHA、Unicode/PUA 映射表、原图—字体并排校样、manifest、可编译源码和最终 PDF。PUA 只在对应字体和 mapping 中有效；没有原始轮廓时保留图片作为研究证据。实际打开或打印产物检查字形非空、码位可复制、字体已嵌入。编译成功和视觉相似都不等于学术确认。
