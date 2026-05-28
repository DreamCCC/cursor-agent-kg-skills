# Cursor Agent 知识图谱抽取资料包

本目录用于存放将来可复制到 Cursor Agent 内部的 Skill 与参考文档，目标是让 agent 按关系类型自动调研、比对现有数据，并输出可审核的候选三元组。

## 目录结构

```text
cursor-agent-kg-skills/
├── README.md
├── knowledge-graph-relation-extraction/
│   └── SKILL.md
└── reference/
    ├── agent-runtime.md
    ├── database-api.md
    ├── output-schema.md
    ├── quality-gate.md
    ├── workflow-validation.md
    └── relation-types/
```

## 使用方式

1. 将 `knowledge-graph-relation-extraction/` 作为 Cursor Agent Skill 上传或复制到目标环境。
2. 将 `reference/` 目录一并提供给 agent，作为抽取口径和输出规范。
3. 每次任务传入一个 `relation_code`，例如 `team_home_arena` 或 `retired_number`。
4. agent 默认只输出候选结果，不直接发布数据。

## 一期关系类型

- `team_home_arena` 球队主场馆
- `arena_location` 场馆所在地
- `retired_number` 退役号码
- `win_honor` 获得荣誉
- `referee_officiate` 裁判员执裁
- `family_legacy` 父子 / 家庭传承
- `coach_team` 执教
- `team_predecessor` 球队前身
- `team_championship_season` 球队夺冠赛季
- `team_historical_location` 球队历史所在地

