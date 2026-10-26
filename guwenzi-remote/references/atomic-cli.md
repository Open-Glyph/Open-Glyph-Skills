# 外部 guwenzi-tools CLI：证据、造字与排印

仅用于有终端和看图能力的外部 Agent；网站出现 TaskState / PreparePage 时使用托管工具，不安装 CLI 或整库。

## 连接与工具发现

```sh
guwenzi-tools login
guwenzi-tools whoami
guwenzi-tools tools --server
```

登录凭据不进入提示词或成果；使用返回的真实 schema、asset_id、document_id 和下载地址。远端负责检测、DINO 检索和 KAGE，不向每个客户端下载模型/字库。CLI 自身不认识图中文字。

## 指定页面与候选

```sh
guwenzi-tools document open paper.pdf
guwenzi-tools page image DOCUMENT_ID 32 --output work/page
guwenzi-tools glyph detect ASSET_ID --output work/candidates --preview-only
guwenzi-tools glyph search CROP_ASSET_ID --output work/search
guwenzi-tools glyph lookup 旣 --output work/variants
```

用用户指定的物理页；不要遍历别页寻找释文。page image 不机械跑全套版面/切字。源字节、原始分辨率与坐标空间保留；模型视图用灰度。先查看总览，再按候选编号分批看图并保存辨认，所有 contact_sheets 都可接续，不能只看第一张。

单字检测提出字框，不确认字义。来源特有、罕见或有争议的形用 glyph_search 查形、glyph_lookup 查真实字库预览；DINO 相似度不是身份概率。相同 Unicode 不等于同一种原排形体。可登记未识原图得到 gwz 身份，但原图本身不是排印字体。

有可信原生 PDF 字体/path 时优先复用，不为了调用次数重建。实际需要时才调用原子 crop、reading_plan、split_line，参数以实时 schema 为准。用户要求检测器独占坐标时，模型不得自拟 bbox、补框或用投影重裁。几何排序保留方向和分段；vertical-rl 为右列先、列内上到下，不倒置原字串。

## 按结构造字，不写笔画

```sh
guwenzi-tools glyph lookup u65e3-j --mode parts --output work/parts
guwenzi-tools glyph compose --glyph '实际返回的glyph_ref' --output work/reused
guwenzi-tools glyph compose --ids '⿱艹⿰日月' --output work/composed
```

glyph/ids/structure 三选一。部件和比例由 Agent 选择，程序处理依赖、几何布局和全部底层 KAGE；不编写 kage_json 或笔画坐标。原图可用 --hint-asset 绑定。只有确有依据的标准字符才传 --unicode-codepoint；否则字体局部 PUA 保留形体，不猜一个熟字替代。批量用现有 glyph_compose 的 items 或 batch helper，并保留各项成败。

print_font.status=ready 表示字体文件可用。下载 OTF/TTF、mapping 与可复制文本；PNG/SVG 是预览或中间轮廓，不替代最终字体正文。

## 整篇正文排印（0.5.1+）

```sh
guwenzi-tools typeset manuscript.json --output delivery
# 已有 WeasyPrint/Pango 时可增加 --pdf
```

manifest 示例（将引用换成工具实际返回的值）：

```json
{
  "title": "排印正文",
  "sections": [{"heading": "正文", "text_path": "body.txt"}],
  "fonts": [{"font_path": "glyphs.otf", "mapping_path": "glyph-map.json"}]
}
```

正文保留普通 Unicode，缺字写成 `⟦gwz:sha256:实际64位摘要⟧`。文件路径相对 manifest。程序按 gwz 身份把各字体汇成统一 cmap，导出可编辑 DOCX（内嵌 TTF）、HTML（内嵌 OTF）、TXT 和字体/映射 ZIP，避免把多个单字字体的 U+E000 错串在一起。TXT 离开配套字体不能保证缺字显示。

reading.html 是原图证据预览，不是排印正文。主文档用字体文字，原图对照另作附注。缺实际字体、损坏文件、cmap 冲突会明确报错，不降级成图片、空白或猜读；这与可选的字形质量检查不同。

## 核对不作为完成条件

glyph_verify / verify-font 是可选质量诊断。未执行、needs_review、blocked 或未达相似度均不阻止排印、交付和任务完成，不触发无限重试或为了“补全回执”重复调用。按用户需要精校时检查真实最终字体，保留来源、码位和差异；不要把新字体自动继承成已核对。

交付完成与字形正确分开记录。可直接交付可用结果并说明疑点，不能声称未经核对的形已得到学术确认。DINO 不作验收，0.95 不是正确率或发布阈值。

## 状态与安全

transcribe 只保存 Agent 给出的文字/结构引用，不是 OCR。gwz 身份、来源字体码位、PUA、Unicode 和学者释读分层保存；不做 NFC/NFKC、繁简转换或静默替字。batch 保留单项索引；preview-only 只控制下载视图，不删原始资产引用。

长任务可 --detach 后 status JOB_ID --wait 接续。已提交请求状态未知时按回执查，不盲目重发 POST；429 不自动重试。分页、批次是显示窗口，不另设全文字数/时间上限。输出用可读文件名与目录，保留原文和未解决事项，下载链接只使用工具返回值。

