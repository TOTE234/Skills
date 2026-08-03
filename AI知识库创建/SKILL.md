---
name: ai-kb-sync
version: 3.0.0
description: "将用户的AI学习笔记、每日学习记录、踩坑经验整理为结构化AI知识库，并自动上传到飞书知识库。当用户发来学习笔记、每日记录、踩坑经验、新技能记录、工具使用心得等内容，并要求整理成知识库或上传到飞书时使用。"
metadata:
  requires:
    bins: ["lark-cli"]
---

# AI 知识库创建与飞书同步

> **前置条件：** 先阅读 [`../lark-shared/SKILL.md`](../../lark-shared/SKILL.md) 确认认证和权限。撰写飞书文档内容前，还需阅读 [`../lark-doc/SKILL.md`](../../lark-doc/SKILL.md) 了解文档操作规范。

## 角色定位

你是一个帮助用户整理个人 AI 学习知识库的助手。职责是：阅读用户发来的原始笔记（通常为日记体格式）→ 拆解为原子化、结构化的 Markdown 笔记 → 写入本地知识库 → 自动上传到飞书知识库 → 更新合并版知识库文件。

## 目录结构

```
my-ai-kb/
├── SKILL.md                           ← 你正在看的文件，定义完整流程和规则
├── feishu-config.json                 ← 飞书知识空间配置
├── 知识库_合并版_用于AI引用.md         ← 所有笔记的合并版（AI 引用专用）
├── wiki/                              ← 结构化知识：整理后的笔记（核心产出）
│   ├── concepts/                      ← 概念术语库：名词定义、知识点解释
│   ├── sops/                          ← 工作流库：分步骤的操作流程
│   ├── cases/                         ← 案例经验库：真实踩坑与复盘记录
│   ├── faqs/                          ← 常见问题库：高频问题与标准答案
│   └── snippets/                      ← 代码片段库：命令、快捷键、配置
└── skill-ai-kb-sync/
    └── SKILL.md                       ← Skill 副本（供 lark-cli 加载）
```

## 知识库配置

| 配置项 | 值 |
|--------|-----|
| 飞书知识空间 ID | `7666756865782451492` |
| 空间名称 | AI学习知识库 |
| 本地知识库根目录 | `/workspace/my-ai-kb/` |
| 配置文件 | `/workspace/my-ai-kb/feishu-config.json` |
| 合并版文件 | `/workspace/my-ai-kb/知识库_合并版_用于AI引用.md` |

---

## 写作规则

### 笔记拆分规则

用户笔记通常包含以下板块，对应知识库的五个模块：

| 用户笔记板块 | 目标模块 | 目录 | 判断标准 |
|--------------|----------|------|----------|
| 新技能 / 新知识 | concepts | `wiki/concepts/` | 名词定义、概念解释、知识点 |
| 实践（做了什么） | sops | `wiki/sops/` | 分步骤的操作流程、方法论 |
| 踩坑 & 解决 | cases | `wiki/cases/` | 踩坑记录（背景→行动→结果→原因→解决→经验） |
| 高频问题（从踩坑提炼） | faqs | `wiki/faqs/` | 问题-答案对，追加到已有 FAQ 文件 |
| 提示词 / 代码片段 | snippets | `wiki/snippets/` | 命令、快捷键、代码片段、配置 |

### 文件命名

- 用中文短语命名，语义清晰：`误下载macOS版本.md`
- 同一主题多个文件时加前缀：`GitHub下载资料流程.md`、`GitHub下载注意事项.md`

### 笔记结构（每篇必须包含）

```markdown
> **模块**：concepts / sops / cases / faqs / snippets
> **主题**：前端 / 后端 / 工具 / AI模型 / 通用
> **最后更新**：YYYY-MM-DD
> **复核周期**：长期 / 季度复核 / 临时

# 标题

正文内容...
```

### 写作原则

1. **原子化**：一篇只讲一个知识点或一个流程，不要把多个不相关内容塞进同一文件
2. **结论先行**：第一句话给出核心定义或结论，再展开细节
3. **结构化**：用 `#` `##` `###` 标题层级，步骤用有序列表，对比用表格
4. **交叉链接**：相关主题用 `[[文件名]]` 建立双向链接
5. **显式标注前提**：有时效性或条件性的内容写明适用条件
6. **完整吸收**：用户笔记中的每一条可复用经验都必须进入知识库，不遗漏

### 交叉链接规则

- 只链接真正相关的笔记，避免链接泛滥
- 在正文中用自然语句引入链接，如"关于文件后缀的识别，参见 [[文件后缀与操作系统对应关系]]"
- 概念类笔记是链接的"枢纽"，SOP 和案例链接到相关概念

### FAQ 模块特殊规则

- 不新建文件，追加到 `wiki/faqs/软件下载安装类FAQ.md`
- 更新 `最后更新` 日期
- 每条问答格式：`**Q：问题？**\n\nA：答案。\n依据：[[相关笔记]]`
- 问答之间用 `---` 分隔

### 合并版文件规则

`知识库_合并版_用于AI引用.md` 是所有笔记的合并版，用于直接上传给 AI 引用：

1. **头部元信息**：版本日期、笔记总数、覆盖主题（每次更新时修改）
2. **给 AI 的使用说明**：说明知识库结构和引用方式
3. **按模块组织**：concepts → sops → cases → faqs → snippets
4. **新增笔记追加在附录前**：以"模块N：新增笔记（日期）"为章节标题
5. **附录统计表**：更新各模块的笔记数量
6. **编号连续递增**：每篇笔记有唯一编号，新笔记接着上一个编号

---

## 完整流程（六步）

### 第一步：读取配置并验证飞书连接

```bash
# 读取本地配置
cat /workspace/my-ai-kb/feishu-config.json

# 验证飞书连接（列出知识空间确认权限正常）
LARKSUITE_CLI_NO_UPDATE_NOTIFIER=1 LARKSUITE_CLI_NO_SKILLS_NOTIFIER=1 \
lark-cli wiki +space-list --as user --json
```

**如果配置文件不存在**，先创建知识空间：

```bash
LARKSUITE_CLI_NO_UPDATE_NOTIFIER=1 LARKSUITE_CLI_NO_SKILLS_NOTIFIER=1 \
lark-cli wiki +space-create --name "AI学习知识库" \
  --description "个人AI学习笔记的结构化知识库" --as user
```

将返回的 `space_id` 写入 `feishu-config.json`：

```json
{
  "space_id": "<返回的space_id>",
  "space_name": "AI学习知识库",
  "created_at": "<创建日期>"
}
```

### 第二步：解析用户笔记并拆分为原子笔记

分析用户发来的笔记内容（通常为日记体格式，包含"新技能/新知识""实践""踩坑&解决""提示词/代码片段""可复用总结"等板块），按上方「笔记拆分规则」表格归入对应模块，按「写作原则」生成结构化笔记。

### 第三步：写入本地知识库

将拆解后的笔记写入对应模块目录，文件名用中文短语，语义清晰。

**关键点**：
- 文件路径：`/workspace/my-ai-kb/wiki/<模块>/<文件名>.md`
- 文件名示例：`HF-Mirror国内镜像站.md`、`用AI学习新技能的闭环流程.md`
- 同一主题多个文件时加前缀区分

**对于 FAQ 模块**：不新建文件，而是读取已有的 `wiki/faqs/软件下载安装类FAQ.md`，在末尾追加新问答，并更新 `最后更新` 日期。

### 第四步：上传新笔记到飞书知识库

对每个新建的 Markdown 文件，执行两步操作：

#### 4.1 创建飞书知识库节点

```bash
# 在飞书知识库中创建节点（可并行创建多个）
LARKSUITE_CLI_NO_UPDATE_NOTIFIER=1 LARKSUITE_CLI_NO_SKILLS_NOTIFIER=1 \
lark-cli wiki +node-create \
  --space-id "<SPACE_ID>" \
  --title "<笔记标题>" \
  --as user --json
```

从返回结果中提取 `node_token`（用于后续更新）和 `obj_token`（文档ID）。

#### 4.2 用 Markdown 内容填充文档

```bash
# 复制文件到 cwd 下（lark-cli --content @file 只接受相对路径）
cp "/workspace/my-ai-kb/wiki/<模块>/<文件名>.md" "/workspace/my-ai-kb/_content_temp.md"

# 用 Markdown 内容覆盖文档
LARKSUITE_CLI_NO_UPDATE_NOTIFIER=1 LARKSUITE_CLI_NO_SKILLS_NOTIFIER=1 \
lark-cli docs +update \
  --doc "<node_token>" \
  --command overwrite \
  --doc-format markdown \
  --content "@_content_temp.md" --json
```

**注意**：
- `--content @<path>` 只接受当前工作目录下的相对路径，传绝对路径会报 `unsafe file path`
- 执行命令时 cwd 必须是 `/workspace/my-ai-kb/`
- 上传完成后删除临时文件 `rm -f _content_temp.md`
- 多个笔记可以并行创建节点，内容填充也可并行执行

#### 4.3 更新已有 FAQ 节点

如果 FAQ 有新增问答：

```bash
# 先更新日期
LARKSUITE_CLI_NO_UPDATE_NOTIFIER=1 LARKSUITE_CLI_NO_SKILLS_NOTIFIER=1 \
lark-cli docs +update \
  --doc "<FAQ_node_token>" \
  --command str_replace \
  --doc-format markdown \
  --pattern "> **最后更新**：旧日期" \
  --content "> **最后更新**：新日期" --json

# 再追加新问答（创建临时文件）
LARKSUITE_CLI_NO_UPDATE_NOTIFIER=1 LARKSUITE_CLI_NO_SKILLS_NOTIFIER=1 \
lark-cli docs +update \
  --doc "<FAQ_node_token>" \
  --command append \
  --doc-format markdown \
  --content "@_faq_append.md" --json
```

### 第五步：更新合并版知识库文件

将所有笔记合并写入 `/workspace/my-ai-kb/知识库_合并版_用于AI引用.md`，按上方「合并版文件规则」操作：

1. 更新头部元信息：版本日期、笔记总数、覆盖主题
2. 追加新笔记：在附录前插入新模块章节，编号递增
3. 更新附录统计表：各模块笔记数
4. 上传到飞书：用 `overwrite` 命令更新飞书上的合并版节点

```bash
# 上传合并版到飞书（使用已知节点的 node_token）
cp "/workspace/my-ai-kb/知识库_合并版_用于AI引用.md" "/workspace/my-ai-kb/_merged_temp.md"

LARKSUITE_CLI_NO_UPDATE_NOTIFIER=1 LARKSUITE_CLI_NO_SKILLS_NOTIFIER=1 \
lark-cli docs +update \
  --doc "<合并版_node_token>" \
  --command overwrite \
  --doc-format markdown \
  --content "@_merged_temp.md" --json

rm -f "/workspace/my-ai-kb/_merged_temp.md"
```

**合并版节点 token**：首次上传时创建，记录在知识库节点列表中。可通过 `wiki +node-list --space-id <SPACE_ID>` 查找标题为"知识库合并版（AI引用专用）"的节点获取 token。

### 第六步：清理临时文件并回报用户

```bash
# 清理所有临时文件
rm -f /workspace/my-ai-kb/_content_temp.md /workspace/my-ai-kb/_faq_append.md /workspace/my-ai-kb/_merged_temp.md
```

回报内容：
- 新增了哪些笔记（文件名 + 模块分类）
- FAQ 新增了哪些问答
- 飞书知识库同步状态
- 知识库现有笔记总数

---

## 权限

| 操作 | 所需 scope |
|------|-----------|
| 创建知识空间 | `wiki:space:write_only` |
| 创建知识库节点 | `wiki:node:create` |
| 创建文档 | `docx:document:create` |
| 编辑文档内容 | `docx:document:write_only` |
| 读取文档内容 | `docs:document.content:read` |
| 列出知识空间 | `wiki:space:read` |

## 环境变量

执行所有 lark-cli 命令时加上环境变量以避免更新提示干扰输出：

```bash
LARKSUITE_CLI_NO_UPDATE_NOTIFIER=1 LARKSUITE_CLI_NO_SKILLS_NOTIFIER=1 lark-cli ...
```

## 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| `unsafe file path` | `--content @file` 传了绝对路径 | 复制文件到 cwd 下，用相对路径 `@_temp.md` |
| `confirmation_required` (exit 10) | 高风险写操作未确认 | 向用户确认后追加 `--yes` |
| `permission_denied` | 缺少 scope | 用 `lark-cli auth login --domain wiki --domain docs` 授权 |
| FAQ `str_replace` 失败 | 日期格式或文本不匹配 | 先 `docs +fetch` 确认当前内容再替换 |

## 并行优化

- **节点创建可并行**：多个笔记的 `wiki +node-create` 可同时执行
- **内容填充可并行**：多个 `docs +update` 可同时执行（不同文档）
- **串行依赖**：FAQ 更新（先 str_replace 再 append）、合并版更新（先本地写入再上传）必须串行

## 关键注意事项

1. **完整吸收用户输入**：用户笔记中的每一条可复用经验都必须进入知识库，不遗漏
2. **交叉链接**：新笔记要与已有相关笔记建立 `[[链接]]`
3. **FAQ 追加不覆盖**：FAQ 模块是追加新问答到已有文件，不是创建新文件
4. **合并版同步**：每次新增笔记后必须更新合并版文件并同步到飞书
5. **临时文件清理**：所有 `_*.md` 临时文件用完后必须删除，不污染知识库目录
6. **日期格式**：统一用 `YYYY-MM-DD` 格式
7. **文件名规范**：中文短语命名，语义清晰，不含特殊字符
