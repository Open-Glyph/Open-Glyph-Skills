# Open Glyph Skills

这是 Open Glyph（开放古文字）的可复用 Agent Skills 集合，供能够读取 `SKILL.md` 的 Agent 使用。

## 0.4.0：结构化组字，不再手写笔画

先用现有 DINO 检索找候选，或用 `glyph_lookup` 查整字、变体和部件；再给 `glyph_compose` 一个选中的字形、IDS 或结构树。程序负责部件依赖、布局和底层 KAGE，Agent 负责选择与核对。

```sh
python -m pip install -U guwenzi-tools
guwenzi-tools glyph lookup 旣 --output work/variants
guwenzi-tools glyph compose --ids '⿱艹⿰日月' --output work/composed
```

检查 `tools --server` 是否包含 `glyph_lookup` 以及 `glyph_compose` 的 `glyph/ids/structure` 参数。旧 `kage_json` 输入已退役；旧服务需同步升级，不让 Agent 回退到逐笔编程。字库与模型在服务端，不向每个沙箱复制。字体生成成功仍需原图核对，DINO 分数不作验收依据。

## 三个 Skill

- [`guwenzi-remote`](guwenzi-remote/SKILL.md)：最基础、最重要的工具调用 Skill。说明如何使用 `guwenzi-tools` 登录、提交文档、调用 YOLO 版面分析、投影裁切、DINO 字形检索、整字复用与结构化组字、转写和导出。它强调每次读取结果后由 Agent 自己决定下一步，不要求固定流水线。
- [`guwenzi-read`](guwenzi-read/SKILL.md)：古文字论文、扫描资料、PDF、Word、PPT 的证据提取、阅读、比较与释读记录。
- [`guwenzi-compose`](guwenzi-compose/SKILL.md)：古文字组字、KAGE 据图重建、字体映射和古文字论文撰写排版。

推荐使用顺序：先加载 `guwenzi-remote` 了解工具契约；需要阅读资料时再加载 `guwenzi-read`；需要造字或撰写排版论文时再加载 `guwenzi-compose`。

## 安装 CLI

工具调用 Skill 依赖公开的 `guwenzi-tools` 客户端：

```sh
python -m pip install -U guwenzi-tools
guwenzi-tools login
```

登录凭据由用户在 Open Glyph 网站中授权。Skill 不包含服务地址、Token、AccessKey 或任何用户数据。

## 0.3.6 配套的快速阅读方式

先用 `guwenzi-tools tools --server` 检查远端支持的参数。新版 CLI 和支持新参数的服务可以：

- 用 `page image` 先取页图和原生文字 trace，不默认跑版面分析或切整页字槽；字体/path 等完整原生证据仍按需提取。
- 对已有清楚段落直接阅读；需要定位时才做 `regions`，需要分行或字槽时再显式请求。
- 用 `batch crop`、`batch glyph_search`、`batch glyph_compose` 集中处理多个疑字，查看带编号的候选/造字拼图；只为不清楚的项另取原图。
- 复用精确相同且已核对的字形或配方，最后统一整理文字、字体和排版，不逐字重做整个流程。
- 首次检索遇到冷启动时按任务 ID 继续等待，不自动重发 POST；检索候选、重建字形和学术释读仍分层记录。

拼图是缩放预览，不能拿其坐标裁原图，也不能把相似度当成识字概率。旧版客户端可通过通用 `call` 传递远端支持的新参数。

更新 CLI 不会自动覆盖 Agent 已安装的 Skill 文件。更新本仓库中的三个 Skill 并重新加载；只使用基础工具 Skill 时，也可运行：

```sh
guwenzi-tools agent install-skill --directory /your-agent/skills/guwenzi-remote
```

## 安全与证据边界

本仓库只包含 Markdown 指令，不包含模型权重、论文、字库图片、用户文件、服务器 IP、EAS 地址、API Token、AccessKey、私有路径或部署配置。

Skill 要求 Agent：

- 保留原始文件、原始图片、路径程序、哈希和页面坐标；
- 不做 NFC/NFKC、繁简转换或静默码位替换；
- 将来源编码、具体字形、永久 `gwz:` 身份和作者释读分开记录；
- 将 YOLO 类别、投影槽位、OCR 输出和 DINO 相似度视为证据或候选，而不是自动确认；
- 不把 `c1`、`asset_id`、`span_id`、Unicode、Seal、PUA 和 `gwz:` 互相混用；
- 对未确认的字形保留原图和不确定状态，不编造释读。

## 许可证

本仓库中的 Skill 文本使用 MIT License。外部 CLI、模型、字体、KAGE 引擎、GlyphWiki 数据、论文和字库资产仍分别受其自身许可证约束；本仓库不重新授权这些外部材料。
