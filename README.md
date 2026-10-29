# 鲁迅传记知识图谱（Lu Xun Biography Knowledge Graph）

> English: [README.en.md](README.en.md)

本仓库发布基于四部鲁迅传记文本构建的鲁迅传记知识图谱，经实体—关系抽取（DeepSeek-v4-flash）、实体归一化与跨书去重后构建，可用于关系推理、多跳检索、社区发现等知识图谱任务。

图谱采用"实体—关系—实体"的有向三元组结构：实体划分为 9 种类型，关系划分为 12 种类型，每条关系记录其编号、名称和主宾类型约束。主人公鲁迅（`G_PERSON_017`）是图谱中的核心节点，与 1,327 条关系相连，占全图关系总数的 43.2%。原始传记文本受版权保护，不在发布范围内；本仓库仅发布图谱结构本身。

## 文件说明

| 文件 | 内容 | 规模 |
|---|---|---|
| `entities.csv` | 实体表：`entity_id, entity_name, entity_type, entity_desc` | 3,165 行 |
| `triples.csv` | 三元组表：`subject_id/name/type, relation_id/name/type, object_id/name/type` | 3,075 行 |
| `relations.csv` | 关系类型定义：`relation_id, name_en, name_zh, category_zh, domain_constraint` | 12 行 |

编码均为 UTF-8（含 BOM，可直接用 Excel 打开）。`entity_id` 前缀与 `entity_type` 一致（如 `G_PERSON_017`），可作为数据一致性自检。相对项目内部完整数据，本发布版删除了溯源列（书名、文本块编号与原文证据句），未修改任何图谱内容。

## 实体类型（9 种）

| 类型 | 含义 | 数量 |
|---|---|---|
| PERSON | 真实历史人物（鲁迅、许广平、周作人等） | 1,033 |
| WORK | 文学作品与学术作品（《呐喊》《朝花夕拾》等） | 824 |
| PLACE | 地理与建筑场所（绍兴、仙台、北平、上海等） | 409 |
| ORG | 机构、社团与刊物 | 394 |
| OTHER | 无法归入其他类型的实体 | 147 |
| CONCEPT | 抽象思想与观念 | 124 |
| CHARACTER | 文学作品中的虚构人物（阿Q、祥林嫂等） | 102 |
| EVENT | 历史事件与生平节点 | 68 |
| PERIOD | 时段与时代背景 | 64 |

## 关系类型（12 种，有向）

| ID | 名称 | 主宾约束 | 数量 |
|---|---|---|---|
| R001 | Creation 创作 | PERSON → WORK | 640 |
| R002 | Association 交往 | PERSON → PERSON | 456 |
| R003 | Residence 居住 | PERSON → PLACE | 404 |
| R004 | Occurrence 发生于 | EVENT → PLACE/PERIOD | 101 |
| R005 | Participation 参与 | PERSON → EVENT | 107 |
| R006 | Affiliation 属于 | ENTITY → ORG/PERIOD | 418 |
| R007 | Mention 提及 | WORK → ENTITY | 546 |
| R008 | Containment 包含 | WORK → CHARACTER | 115 |
| R009 | Influence 影响 | PERSON/CONCEPT → PERSON | 80 |
| R010 | Opposition 对立 | PERSON → PERSON | 85 |
| R011 | Mentorship 师生 | PERSON → PERSON | 32 |
| R012 | Kinship 亲属 | PERSON → PERSON | 91 |

## 数据规模

- 实体 3,165 个，三元组 3,075 条（归一化去重后），其中 2,210 个实体至少参与一条关系
- 主人公鲁迅（`G_PERSON_017`）关联三元组 1,327 条，占 43.2%
- 实体与关系的类型分布见上两表，合计分别为 3,165 与 3,075
- 上述统计均可由本仓库文件直接复算

## 示例三元组

| 主语 | 关系 | 宾语 | 关系编号 |
|---|---|---|---|
| 鲁迅 | 创作 | 狂人日记 | R001 |
| 鲁迅 | 交往 | 许寿裳 | R002 |
| 章太炎 | 师生 | 鲁迅 | R011 |
| 鲁迅 | 亲属 | 周作人 | R012 |
| 鲁迅 | 对立 | 陈西滢 | R010 |

## 使用说明

两张主表通过实体编号关联：`triples.csv` 的主语/宾语编号对应 `entities.csv` 的 `entity_id`。

```python
import pandas as pd

ent = pd.read_csv("entities.csv", encoding="utf-8-sig")
tri = pd.read_csv("triples.csv", encoding="utf-8-sig")
rel = pd.read_csv("relations.csv", encoding="utf-8-sig")

# 主语侧实体信息并入三元组
merged = tri.merge(ent, left_on="subject_id", right_on="entity_id", how="left")

# 以鲁迅为中心的一跳子图
hub_id = "G_PERSON_017"
neighbors = tri[(tri["subject_id"] == hub_id) | (tri["object_id"] == hub_id)]
```

`entity_desc` 为抽取阶段生成的简短标注（如"身份:…""关系:…"等），用于辅助理解实体，不作事实断言。
