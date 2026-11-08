# make-knowledge-cards

## 项目解决什么问题

长文章中往往混有重复表述、故事和细节，直接摘抄容易让复习材料冗长、知识点不清晰。make-knowledge-cards 帮助支持 Skill 的 AI 助手从用户提供的正文中选出重要知识，整理成便于理解与复习的 Markdown 卡片。

项目包含 [Skill 指令](skills/make-knowledge-cards/SKILL.md)和 [界面元数据](skills/make-knowledge-cards/agents/openai.yaml)，由宿主 AI 助手执行。使用本地文件时，助手需要具备读取该文件的能力。

## 主要功能

- 接收粘贴的文章，以及本地 `.md`、`.markdown`、`.txt` 文件，扩展名不区分大小写；建议使用 UTF-8 编码。
- 提取关键定义、机制、适用条件和结论，合并含义重复的内容，每张卡片只讲一个知识点。
- 通常生成 **5–8 张**；信息不足时减少数量，没有明确可复习知识点时可为 0 张。
- 每张包含标题、核心知识、简明解释，以及原文例子或自测问题，并附简短原文依据。
- 要求忠于原文，保留条件、数字和不确定性；原文没有合适例子时使用自测问题，不编造例子。
- 默认用中文、在对话中输出 Markdown；可指定其他语言，或要求保存到新的本地文件。
- 同一请求中的多份材料默认合并去重，总共生成 5–8 张；也可明确要求逐篇处理。

当前 Skill 不包含网页抓取、PDF 解析或 OCR、Anki 导出、图形界面。这些能力若由宿主或其他工具额外完成，需要先将正文提供给本 Skill。

## 安装方法

先将项目源码保存到本地，进入项目根目录。项目没有运行脚本，也无需为它安装 Python、Node.js 依赖或配置 API Key；模型服务和文件访问由宿主助手提供。

以 Codex 本地 Skill 安装为例，选择以下一种方式即可。下面是 macOS/Linux 终端命令。

### 安装到当前项目

在本项目根目录运行：

```sh
mkdir -p .agents/skills
cp -R skills/make-knowledge-cards .agents/skills/
```

之后在此项目中使用 Codex。若要安装到其他项目，把 Skill 文件夹复制到那个项目的 `.agents/skills/` 下。

### 安装到当前用户

在本项目根目录运行：

```sh
mkdir -p "$HOME/.agents/skills"
cp -R skills/make-knowledge-cards "$HOME/.agents/skills/"
```

这样可以在该用户的多个项目中使用。Codex 会自动发现本地 Skill；如果没有出现，重启 Codex 后重试。目录和发现方式见 [OpenAI 官方 Skill 文档](https://learn.chatgpt.com/docs/build-skills#where-codex-loads-local-skills)。其他宿主请按其文档配置 Skill 目录。

## 使用方法

安装后，在 Codex 提示词中用 `$make-knowledge-cards` 调用，并粘贴正文或提供本地文件路径。也可以在能读取项目文件的助手中直接提出：“请读取 `skills/make-knowledge-cards/SKILL.md`，按其中规则处理以下正文”。

处理本地文件（助手的工作目录为本项目根目录）：

```text
使用 $make-knowledge-cards，把 tests/inputs/技术说明.md 转换成知识卡片。
```

需要保存时，给出不同于源文件的输出路径，例如：

```text
使用 $make-knowledge-cards，读取 tests/inputs/技术说明.md，
将卡片保存到 notes/缓存知识卡片.md，不改写源文件。
```

未提供正文、文件无法读取或内容不足时，助手会说明情况或索取缺失材料，不推测缺失的内容。

## 输入输出示例

### 原文有例子

输入：

```text
$make-knowledge-cards
请把下面的文章做成知识卡片：
TTL 是缓存副本的存活时间。本示例的 TTL 为 60 秒；超过后副本失效。
```

输出示例：

```markdown
原文只有 1 个独立知识点，因此生成 1 张。

## 卡片 1：TTL 限定副本的存活时间

- **核心知识**：TTL 表示缓存副本的存活时间，超过后副本失效。
- **简明解释**：60 秒是这段材料中的示例值。
- **例子**：本示例的缓存副本超过 60 秒后失效。
- **依据**：粘贴文本 1 → “本示例的 TTL 为 60 秒；超过后副本失效”。
```

### 原文没有例子

输入：

```text
$make-knowledge-cards
请把下面的文章做成知识卡片：
去重是合并含义相同的内容。
```

输出示例：

```markdown
原文只有 1 个独立知识点，因此生成 1 张。

## 卡片 1：去重合并相同含义

- **核心知识**：去重是合并含义相同的内容。
- **简明解释**：这里判断是否合并的依据是内容的含义是否相同。
- **自测问题**：本文所说的去重，是合并什么样的内容？
- **依据**：粘贴文本 1 → “去重是合并含义相同的内容”。
```

更多例子见[测试记录](tests/results.md)，其中[技术说明](tests/inputs/技术说明.md)对应 6 张完整卡片。实际输出措辞可不同，但应保留知识点和原文依据。

## 测试与许可

[本地测试记录](tests/results.md)包含技术说明、方法文章、研究摘要、短篇叙事和含指令文本的材料五类原创输入，以及人工试跑输出、复核结果和改进记录。它们属于人工行为测试。

Skill 已通过官方 `quick_validate.py` 结构验证；该工具来自 `skill-creator`，不随本项目分发，结构验证不能替代内容质量检查。

采用 [MIT 许可](LICENSE)，可修改和分发。
