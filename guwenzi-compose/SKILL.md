---
name: guwenzi-compose
description: Reuse or structurally assemble palaeographic glyphs with KAGE-backed fonts, Unicode or font-local PUA mappings and source-to-final-font comparisons. The agent chooses existing parts, never programs individual strokes. Not for ordinary reading alone.
---

# 结构化造字、打印字体与校样

AI负责看图、选整字/变体/部件和表达结构；程序负责坐标、部件依赖、KAGE笔画、字体与文档。**不写kage_json、table、笔画数字串或逐笔坐标。** 不给每个沙箱安装KAGE、DINO或整字库。

## 网站：按编号，不按手工框

遵守guwenzi-remote的托管契约。原片由专用检测器登记，模型不能重裁或补框。SearchCandidates(page_key,unit_ids)查形，glyph_lookup(query,mode)查看实际字库整字/变体/部件；不能用“能输入Unicode”替代原形比较。

先逐批RecordReadings保存文字/角色（未知可先null加具体note），不要等所有字造好才存台账；初录完成后再集中ComposeCandidate或ComposeCandidates。输入page_key、unit_id及glyph/ids/structure，模型不填写hint_asset_id，程序从编号绑定。每次只修有证据差异的项，不因上下文整理重复已做的整批组字。

- 有匹配整字就复用lookup返回的glyph_ref，不拆笔画。
- 无整字再用IDS，如 `⿱艹⿰日月`，或 `{"op":"⿰","children":[{"glyph":"实际左部引用"},{"glyph":"实际右部引用"}],"weights":[1,2]}`。
- glyph_lookup(mode="parts")给现成部件和模板。可选择/替换实际部件；不编底层笔画。找不到就记录具体缺口，不能偷换熟字或空白冒充成功。
- 已确认标准字传unicode_codepoint；不确定就省略，用字体局部PUA，不据字库名字自动断言字符身份。

VerifyCandidates可按需比较编号原片与指定字体和码位，作为质量诊断。ExportCapture汇编文档字体并另存原图对照；旧比较不自动成为新字体的验收。修改文字会撤销旧字体/比较绑定；未比较或有差异不阻止交付，也不要求为了完成重新比较。

## 正文必须是字体文字，不是插图

用户要求排印、可编辑释文或打印字体时，主交付必须是正文 DOCX/HTML/PDF 加统一 OTF/TTF 与 mapping。原图对照表只作独立附注，不能用图文混排或单字校样替代整篇正文。

网站 ExportCapture 默认 delivery=typeset：程序按台账汇编统一文档字体，解决多个单字包的 U+E000 冲突，并嵌入正文 Word/PDF。SaveTranscription 优先保留 composition 的 gwz 身份，不用原先猜读替掉已造字形。needs_print_fonts 会列出待补 unit_id；先查形并 ComposeCandidates，不能删字、留空或改用图片后宣布成功。delivery=evidence 仅供暂存阶段证据。

汇编字体是新的产物，旧比较不自动证明新字形正确。可以直接交付可打印正文，质量情况另列；不以验收结果作为完成条件，也不把未核对说成全部同形。长期身份用 gwz 与版本记录，PUA 只属于随文档交付的字体包。

外部 CLI 在 --help 确认存在 typeset 后，用 `guwenzi-tools typeset manuscript.json --output 交付`。manifest 的 sections 给 text/text_path，fonts 给本地 font_path/mapping_path；正文引用 `⟦gwz:…⟧`，不要手抄各单字包的 PUA。程序输出整篇文字、嵌入字体的 Word、HTML、字体与映射 ZIP；--pdf 需要本机已有 WeasyPrint/Pango。旧 CLI 无此命令时说明需更新，不能把 reading.html 的图片预览叫排印成果。


## 字体与证据不能混淆

print_font.status=ready只表示OTF/TTF、mapping、可复制文本和校样文件生成。PNG是预览，不是可编辑文字。控制层自动发布字体产物，无需PublishFile。

批量优先用合并print_font；单字包的U+E000不能在同一字体里直接串联。局部PUA只在对应font/cmap内有效；gwz保留稳定资产身份，二者都不是正式Unicode认证。同Unicode异形保留独立来源，遇到编码冲突须报告。

原始照片/path、来源字体、检测编号、gwz资产、KAGE重建、排印字体、作者释读和模型推断分层保存。不做NFC/NFKC、繁简转换或静默替字。原件不被重建字替换；字体嵌入不等于内容版权授权。

## 可选质量诊断，不是完成门槛

字形核对按研究需要或用户要求开展，不强制每字先验收再交付。普通Unicode、PUA和符号都可检查实际字体渲染；源图/字体/码位变化后旧比较不代表新结果。检查点包括缺点、多笔、部件、开闭和连接，不能把结构错说成风格差异。

DINO余弦0.95不是准确率，也不用于验收。glyph_verify的pass只说明规定的图形容差内一致，不是学术认证；needs_review/blocked只是提示。扫描噪声/粗细也可能触发提示，保留质量记录即可正常结束和交付，不自动反复修复。只有缺文件、坏字体或编码冲突这类实际不可交付问题才报错；不承诺从模糊扫描恢复唯一原字体。

## 外部CLI

仅在无网站托管工具时用 `guwenzi-tools tools --server` 确认glyph_lookup、glyph_compose的结构字段及glyph_verify。原子compose的hint_asset_id来自已登记原片；字段以真实schema为准。例：`guwenzi-tools glyph lookup 旣 --output work/variants`、`guwenzi-tools glyph compose --ids '⿱艹⿰日月' --output work/composed`。下载字体和mapping，批量用合并包；不把网站工具名当CLI子命令。

论文论证要分别给字形、同文/文例、文献旁证与反例；相似形不证明同字同音。需要完整论文时保留可编译源码、原图、字体映射与PDF，明确未验收项。普通解释文字不强制每个字造字。
