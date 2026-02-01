---
name: anki-near-homophones
description: Uses anki-mcp-server to find near-homophones (derivatives, similar pronunciation, similar word forms) for English words from a vocabulary deck, adds diverse memory hints (词源/读音/拼写/语义/场景/联想等), presents matches for user confirmation, then adds confirmed cards with memory hints to a near-homophones deck. Use when the user asks to find 近形词、衍生词、读音相近、词型相近、记忆提示 from Anki English deck or to add such cards.
---

# Anki 近形词工作流

从 Anki 英语单词卡组获取单词，为每个词找出近形词（衍生词、读音相近、词型相近），并为每对搭配**多元化记忆提示**，**先展示匹配结果供用户确认**，确认后再向近形词卡组添加新卡片（含记忆提示字段）。

## 前置条件

- 已配置并启用 **anki-mcp-server**（通过 AnkiConnect 连接本地 Anki）
- Anki 中已有英语单词卡组；可指定或创建「近形词」目标卡组

## 工作流

### 阶段一：获取词源并匹配近形词

1. **确定词源**
   - 若用户指定了卡组名：用 MCP 的 `list_decks` 确认存在，再用 `search_notes` 按卡组查询笔记。
   - 若用户给了一组单词：直接使用该列表，无需从 Anki 拉取。
   - 从笔记中提取「单词」字段（通常是 Front 或用户卡组里的单词字段）。

2. **为每个单词找近形词**
   - 用你的语言知识为每个词列出：
     - **衍生词**：同词根/词缀（如 act → action, actor, react）
     - **读音相近**：易混音（如 affect/effect, principle/principal）
     - **词型相近**：拼写易混（如 desert/dessert, quite/quiet）
   - 每条关系标类型：`衍生` / `读音相近` / `词型相近`。

3. **为每对「原词–近形词」生成记忆提示**
   - 每条至少给 1 条记忆提示；可根据关系类型从不同角度选最贴切的一种或多种（见下方「记忆提示角度」）。
   - 提示要简短、可背、易联想，避免长段落。

4. **整理成待确认列表**
   - 按「原词」分组，每项包含：原词、近形词、关系类型、记忆提示、简短说明（可选）。

### 阶段二：展示结果并等待确认

5. **仅展示，不写入**
   - 用表格或清晰列表展示所有「原词 → 近形词（类型）+ 记忆提示」。
   - 示例格式：

   | 原词 | 近形词 | 关系类型 | 记忆提示 | 简要说明 |
   |------|--------|----------|----------|----------|
   | affect | effect | 读音相近 | affect 重音在前＝「施加」影响；effect 重音在后＝「造成」结果 | 易混：影响 vs 结果 |
   | desert | dessert | 词型相近 | dessert 多一个 s＝甜点(sweet) 多一份 | 沙漠 vs 甜点 |
   | act | action | 衍生 | -tion 名词：act 做 → action 行为/动作 | 动词→名词 |

6. **明确征求用户确认**
   - 说明：「以上为匹配结果。请确认要加入近形词卡组的项（可逐条说保留/删除）。确认后我将只对您同意的项在 Anki 中创建卡片。」
   - **在用户明确确认前，不要调用 `create_note` 或 `batch_create_notes`。**

### 阶段三：确认后再写入 Anki

7. **确认后执行**
   - 若用户同意全部或部分：仅对用户确认的「原词–近形词」对执行添加。
   - 用 MCP：`create_deck` 确保目标卡组存在（如「近形词」或用户指定名称）。
   - 用 `create_note` 或 `batch_create_notes` 在**近形词卡组**中创建卡片。
   - 卡片内容建议：
     - 正面：近形词（或「原词 + 近形词」）。
     - 背面：原词、关系类型、释义/例句、**记忆提示**（单独字段或并入备注，与用户笔记类型一致）。
   - 若笔记类型有「记忆提示」或「备注」等字段，务必把生成的记忆提示写入，便于复习时强化区分。

8. **反馈**
   - 简要列出已添加的卡片数量及卡组名；若有失败或跳过，一并说明。

## 使用 MCP 工具时的注意点

- **list_decks**：获取所有卡组，用于定位英语卡组和近形词卡组。
- **search_notes**：用 Anki 查询语法按 `deck:"卡组名"` 等条件查笔记；再根据笔记类型取单词字段。
- **get_note_info** / **get_note_type_info**：需要时用来确认字段名和笔记结构，保证创建时字段正确。
- **create_deck**：若不存在「近形词」卡组则先创建。
- **create_note** / **batch_create_notes**：仅在用户确认后，向近形词卡组添加新笔记；指定 `deckName` 为目标近形词卡组。

## 近形词类型说明

| 类型 | 含义 | 示例 |
|------|------|------|
| 衍生 | 同词根/词缀，派生关系 | decide → decision, decisive |
| 读音相近 | 发音易混 | their / there / they're |
| 词型相近 | 拼写易混 | adapt / adopt, accept / except |

## 记忆提示角度（多元化）

为每条「原词–近形词」选最贴切的 1～2 种角度写记忆提示，避免千篇一律：

| 角度 | 适用类型 | 示例 |
|------|----------|------|
| **词源/词根** | 衍生、词型 | "port = 携带 → import 进口, export 出口"；"cept = 拿 → accept 接受, except 排除" |
| **读音对比** | 读音相近 | "affect 重音在前 / effect 重音在后"；"principal 末尾像 pal 朋友＝校长" |
| **拼写对比** | 词型相近 | "dessert 多一个 s = 甜点多一份"；"quiet 中间是 i = 安静一点" |
| **语义一句区分** | 读音/词型 | "affect 影响（动词） / effect 效果（名词）"；"desert 沙漠 / dessert 甜点" |
| **场景/例句** | 任意 | "effect: the effect of the drug（药的效果）"；"affect: it didn't affect me（没影响我）" |
| **联想/谐音** | 任意 | "principal 校长：校长是你的 pal"；"stationary 静止的：a 一个，站着不动" |
| **词性/搭配** | 衍生、易混 | "decide + to do / decision + to do 或 that 从句"；"advice 名 / advise 动" |
| **字母/形态口诀** | 词型 | "accept：先 accept 才能 except 排除别的"；"quite 很 / quiet 静：e 在最后＝很安静" |

- **衍生词**：优先用词根/词缀、词性变化、搭配区别。
- **读音相近**：优先用重音位置、元音差异、谐音或一句区分。
- **词型相近**：优先用「多/少哪字母」、拼写口诀、语义一句区分。
- 每条提示控制在 1～2 句话，便于写进卡片背面或「记忆提示」字段。

## 原则

- **先展示、后写入**：匹配结果仅用于展示与确认，不默认写入。
- **用户确认是必须步骤**：未确认不调用任何写 Anki 的接口。
- **只添加用户同意的项**：用户可删减列表后再确认，只对最终确认列表写卡片。
