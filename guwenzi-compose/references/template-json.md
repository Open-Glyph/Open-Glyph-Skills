# 图像模板、ASCII JSON 与保留的 IDS 路径

只在需要按图找底字、展开部件、增删饰笔时读。本地已有字与论文编译见 `openglyph-latex.md`，不用先联网。

## 找实际底字与节点

网站当前工具表有接口时用 `SearchTemplates(page_key,unit_ids)` →
`InspectTemplate(page_key,unit_id,program,node)`。CLI 0.6+ 对应：

```sh
guwenzi-tools glyph templates ASSET_ID --scope all --output work/templates
guwenzi-tools glyph inspect --program @program.json --hint-asset ASSET_ID --output work/nodes
```

scope 可为 all/glyph/component。候选有实际字图与固定来源的 program，不凭相似度选第一名。
不要把整字图冒充独立部件图；没有部件图时，先找底字再展开真实节点。
节点路径是该底字版本中的出现位置，不等于语义 IDS 部件，也不是通用笔画序号。

Inspect 的 diagnostics.proposals 可附 SOURCE/CURRENT/PROPOSAL 与局部差异图。
候选 program 已包含建议编辑，符合原图时选 proposal_id，不要再重复添加一次相同修改。
建议绑定这张原图，不能套给另一字；无合适候选才自行编辑 JSON。

```sh
guwenzi-tools glyph compose --proposal RETURNED_ID --hint-asset ASSET_ID --output work/font
# 或提交实际编辑过的程序，两条命令是替代关系：
guwenzi-tools glyph compose --program @edited.json --hint-asset ASSET_ID --output work/font
```

## 字段与编辑

单字程序使用 `schema="guwenzi.template-program.v1"`、base、edits，可选 modules/select。
base 与引用取自实际返回结果；下面展示的是字段，不是可直接提交的虚构部件：

```json
{
  "schema":"guwenzi.template-program.v1",
  "base":"实际候选base",
  "edits":[{"op":"replace","node":"已查看的节点路径","glyph":"实际部件引用"}]
}
```

- `replace`: node/glyph，换一个出现位置的部件。
- `insert`: parent/id/glyph/box，id 用 ASCII 小写标识符，新增路径为 a_ID。
- `drop`: node，删除已看清的多余记录或部件。
- `place`: node/box，调整部件框；`move`: node/delta，整数平移。
- `group`: parent/id/nodes，将同层相邻且原顺序的记录组成模块，不改变覆盖顺序。

原图绑定的服务可用 `box="auto"` 或 fit_nodes 做受约束的位置/大小拟合；这不决定该加哪一笔。
明确的 box 使用父部件 200 单位整数框 [left,top,right,bottom]，不是裁字像素。
离线构建使用服务已解析的整数框，不把未解析 auto 当作离线功能。

基本基元有 primitive:dot/bar/slash/loop。复杂新部件优先检索、从已有记录抽取，或组合模块；
不是任意起一个名字就能得到不存在的几何。禁止手写原始 KAGE 笔画串。

`select` 使用实际 parent/nodes，先选择再执行 edits，保留原父层字格；带作用域变换的层拒绝抽取。
`modules` 定义同级子程序，用 module:名称 引用；可以复用但不循环、不嵌套 modules 字典。

## IDS/structure 始终并行可用

整字用 glyph，简单上下/左右等层次可用 ids 或结构树。与 program/proposal_id 合计五种互斥输入。
用户指定 IDS 时遵从，不强行改写为模板 JSON；默认优先可编辑模板与 LaTeX 项目。

```sh
guwenzi-tools glyph compose --ids '⿱艹⿰日月' --output work/ids-font
```

结构树可用实际 ASCII glyph_ref 作叶子，适合用户无法键入某个部件时；不要假造该引用。
两条造字路径都要看实际输出，下载真实字体和映射；其 LaTeX 包可进入论文，PNG 不代替字体正文。
机器核对只提供诊断，形体与释读由人确认；需要确认时使用当前环境的提问工具，不能自问自答。
