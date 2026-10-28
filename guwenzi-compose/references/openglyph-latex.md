# OpenGlyph-LaTeX：已有字输入、新字构造与论文排印

用于用户指定 LaTeX、Overleaf、可编译论文源码，或介绍这个排印库。主项目：
https://github.com/Open-Glyph/OpenGlyph-LaTeX 。以实际 Release 为准；这里的命令按已发布 v0.2.0。
不要把尚在 CTAN 审核的候选包说成 TeX Live 已收录，也不要为使用本库先安装整套 AI 服务。

## 先选最小路径

1. **已有字／已有字体包**：直接加载字体与 catalogue，普通编译不用 Python、Node、账号、模型或网络。
2. **新形，默认优先**：选真实底字和节点，写 ASCII 模板 JSON → 离线编译字体 → LaTeX 调用。
3. **IDS/structure 并行**：结构明确或用户指定 IDS 时，继续用 `glyph_compose --ids` 或结构树。
   若服务返回 LaTeX 字体包，用其中真实 catalogue；不把 IDS 字符串直接塞进 `\ebag{}`。
   两条造字路径都可以产出字体再排印，LaTeX 本身不是识别器或图像拟合器。

只写论文下载 `openglyph-overleaf-v0.2.0.zip`；要本地造字下载完整 `openglyph-v0.2.0.zip`。
Overleaf 使用 XeLaTeX，编译 main.tex；普通编译不启用 shell escape。v0.2.0 重点验证 XeLaTeX/Tectonic，
LuaLaTeX 在该发布版仍标实验，不把后续候选版测试覆盖写回旧版承诺。

## 已有字：不需要先打出罕见汉字

保持 `openglyph.sty`、`ebag/` 与论文源文件的相对位置。打开随包 `字形索引.html` 看实际形体，
复制 ASCII 编号命令；按部件检索只是线索，不是字符身份判定。

```tex
\documentclass[UTF8,fontset=fandol]{ctexart}
\usepackage{openglyph}
\OpenGlyphLoad{ebag}
\begin{document}
正常汉字照常输入，特殊形体使用\ebag{s030-067}。
\end{document}
```

不要要求用户先敲出缺字，也不要手抄 PUA。源编号、命名空间别名、字体内码位与学术释读分开。

## 新形：一次构建，多次编译论文

离线编译器使用完整源码包里的 `scripts/openglyph.py`，不是 `guwenzi-tools` 子命令。
仅造字步骤需要 Python 3.10+、Node.js、requirements-build.txt 的依赖；使用独立环境，缺软件先说明安装影响。

```sh
python scripts/openglyph.py inspect ebag:s030-067 --node r4 -o component-preview
python scripts/openglyph.py build examples/newforms.json -o myglyphs
```

查看编号预览后，才针对该版本的实例路径编辑。源包的四种编辑是独立示例，不是四步累加：

```json
{
  "schema": "openglyph.project.v1",
  "namespace": "my",
  "glyphs": [{
    "id": "n2",
    "base": "ebag:s030-067",
    "edits": [{"op":"insert","parent":"root","id":"inner_dot",
               "glyph":"primitive:dot","box":[77,129,119,167]}]
  }]
}
```

然后 `\OpenGlyphLoad{myglyphs}`，正文 `\gwglyph{my:n2}`。输出目录必须新建；修订用
`--allocation 旧目录/allocation.json` 保持旧槽位，不覆盖已交付字库。不在一篇文档加载同一 namespace 的不同版本。

`modules` 是可复用子程序，`select` 提取已有记录。完整例子见发布包 `examples/modules.json`。
box 是父部件的 200 单位字格，不是 PDF 像素；已解析的离线项目不接受未解析 `auto`。
不能保证任意新形都能仅凭一个部件名构成；缺少几何或真实部件时列出缺口，不造不存在的编号。

## 由图片工具进入 LaTeX（可选，不是入门前提）

需要先从图像找底字时，按 `template-json.md` 查候选和节点。远端返回的实际 LaTeX 包可直接使用；
可重建时优先使用它返回的 `project.json`，避免丢失解析后的整数框和固定版本部件。
单字 program 不能直接当离线 project；自行封装时保留 schema 对应层级、namespace 和 glyph id。

不要把 `guwenzi-tools typeset manuscript.json` 当作 LaTeX 编译；那是 Word/HTML 导出路径。
用户要哪种产物就交哪种，不能拿图片预览替代任一种正文。

## 交付与检查

交付至少包含：`.tex`、`openglyph.sty`、使用到的完整字体目录、映射/分配与构字源、PDF、必要许可。
在独立目录编译，检查未知 ID/缺字体/命名冲突/缺字警告，确认 PDF 字体嵌入并查看实际字形。
嵌入字体让 PDF 阅读不依赖对方安装字体；它不让 PUA 变成通用 Unicode，也不等于可无损复制进任意字体。

原图与对照另附，不覆盖来源。机器编译成功、字体相似度或验收工具 pass 都不等于学术身份确认；
把需要人核定的形清楚列出。用户明确接受前不替其宣称所有字已核定。
