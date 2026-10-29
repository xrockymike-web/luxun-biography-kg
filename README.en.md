# Lu Xun Biography Knowledge Graph

> 中文版：[README.md](README.md)

This repository releases a knowledge graph of Lu Xun biographies, constructed from four biographical texts through entity–relation extraction (DeepSeek-v4-flash), entity normalization, and cross-book deduplication. It can be used for relation reasoning, multi-hop retrieval, community detection, and other knowledge-graph tasks.

The graph follows a directed "entity–relation–entity" triple structure: entities are divided into 9 types and relations into 12 types, with each relation recording its identifier, name, and subject–object type constraint. Lu Xun (`G_PERSON_017`) is the central node of the graph, connected to 1,327 relations, 43.2% of all relations. The original biographical texts are copyrighted and are not redistributed; this repository releases the graph structure only.

## Files

| File | Content | Size |
|---|---|---|
| `entities.csv` | Entities: `entity_id, entity_name, entity_type, entity_desc` | 3,165 rows |
| `triples.csv` | Triples: `subject_id/name/type, relation_id/name/type, object_id/name/type` | 3,075 rows |
| `relations.csv` | Relation-type definitions: `relation_id, name_en, name_zh, category_zh, domain_constraint` | 12 rows |

All files are UTF-8 (with BOM; can be opened directly in Excel). The `entity_id` prefix matches `entity_type` (e.g. `G_PERSON_017`), which can serve as a consistency self-check. Compared with the internal full data, this release removes the provenance columns (book titles, text-chunk identifiers, and evidence sentences) without modifying any graph content.

## Entity types (9)

| Type | Meaning | Count |
|---|---|---|
| PERSON | Real historical figures (Lu Xun, Xu Guangping, Zhou Zuoren, etc.) | 1,033 |
| WORK | Literary and scholarly works (*Call to Arms*, *Dawn Blossoms Plucked at Dusk*, etc.) | 824 |
| PLACE | Geographical and architectural sites (Shaoxing, Sendai, Beiping, Shanghai, etc.) | 409 |
| ORG | Institutions, societies, and periodicals | 394 |
| OTHER | Entities that fit no other type | 147 |
| CONCEPT | Abstract ideas and concepts | 124 |
| CHARACTER | Fictional characters in literary works (Ah Q, Xianglin's Wife, etc.) | 102 |
| EVENT | Historical events and life milestones | 68 |
| PERIOD | Time spans and era backgrounds | 64 |

## Relation types (12, directed)

| ID | Name | Constraint | Count |
|---|---|---|---|
| R001 | Creation | PERSON → WORK | 640 |
| R002 | Association | PERSON → PERSON | 456 |
| R003 | Residence | PERSON → PLACE | 404 |
| R004 | Occurrence | EVENT → PLACE/PERIOD | 101 |
| R005 | Participation | PERSON → EVENT | 107 |
| R006 | Affiliation | ENTITY → ORG/PERIOD | 418 |
| R007 | Mention | WORK → ENTITY | 546 |
| R008 | Containment | WORK → CHARACTER | 115 |
| R009 | Influence | PERSON/CONCEPT → PERSON | 80 |
| R010 | Opposition | PERSON → PERSON | 85 |
| R011 | Mentorship | PERSON → PERSON | 32 |
| R012 | Kinship | PERSON → PERSON | 91 |

## Data scale

- 3,165 entities and 3,075 triples (after normalization and deduplication); 2,210 entities participate in at least one relation
- Lu Xun (`G_PERSON_017`) is incident to 1,327 triples, 43.2% of the graph
- The type distributions are given in the two tables above, summing to 3,165 and 3,075 respectively
- All statistics above can be recomputed directly from the files in this repository

## Example triples

| Subject | Relation | Object | Relation ID |
|---|---|---|---|
| Lu Xun | Creation | Diary of a Madman | R001 |
| Lu Xun | Association | Xu Shoushang | R002 |
| Zhang Taiyan | Mentorship | Lu Xun | R011 |
| Lu Xun | Kinship | Zhou Zuoren | R012 |
| Lu Xun | Opposition | Chen Xiying | R010 |

## Usage

The two main tables are linked by entity identifier: the subject/object IDs in `triples.csv` correspond to `entity_id` in `entities.csv`.

```python
import pandas as pd

ent = pd.read_csv("entities.csv", encoding="utf-8-sig")
tri = pd.read_csv("triples.csv", encoding="utf-8-sig")
rel = pd.read_csv("relations.csv", encoding="utf-8-sig")

# Attach subject-side entity information to triples
merged = tri.merge(ent, left_on="subject_id", right_on="entity_id", how="left")

# One-hop subgraph centered on Lu Xun
hub_id = "G_PERSON_017"
neighbors = tri[(tri["subject_id"] == hub_id) | (tri["object_id"] == hub_id)]
```

`entity_desc` contains short annotations produced during extraction (e.g. "身份:…", "关系:…"), intended to aid entity interpretation rather than to assert facts.
