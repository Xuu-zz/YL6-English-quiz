---
name: yl6-variant-generator
description: 错一道题出 2–3 道同类变式，考点不变、外壳换掉。
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [education, yilin, grade6, variant, drill, json]
    related_skills: [yl6-question-generator, error-cluster-yl6, grammar-variant-factory]
---

# 变式题：错一道，练三道

错题之后最忌讳「再做一遍原题」——记住了答案，没学会规则。本技能把**考点钉死、外壳换掉**。

## When to Use / 什么时候用

- 「这道题再出 3 道类似的」
- 错题本里点「举一反三」
- 老师备课要「同一考点换个问法」检验是不是真会

## 入参

一条**原题 JSON**（`yl6.item/1` 的任一条目），最少要有 `id / unit / tag / type / stem / answer`。

```json
{"id":"6A-U2-C-01","unit":"Unit 2","tag":"past_form","type":"choice",
 "stem":"It ____ hard yesterday, so we stayed at home.",
 "options":{"A":"rain","B":"rains","C":"rained","D":"raining"},"answer":"C","answer_text":"rained"}
```

缺 `tag` 时先用 `eng-grammar-tag-extract-yl6` 打标签，再变式。**不要凭题干自己猜考点。**

## 换什么、不换什么

| 维度 | 可换成 | 注意 |
|---|---|---|
| 主语 | I / you / he / she / we / they / Mike / Su Hai and Su Yang | 换完必须重判主谓一致，答案可能跟着变 |
| 时间状语 | yesterday / last Sunday / two days ago / just now / this morning | 只能换成**同属过去**的时间；过去时的题不许换成 `now` |
| 场景 | at school / in the park / at home / on the farm / in the library / on the playground | 动宾搭配必须成立（`water the flowers` 可以，`water the desk` 不行） |
| 动词 | **同变化类型**：规则→规则，不规则→不规则 | 把 `rain` 换成 `go` 就是换了难度，不是同考点 |
| 句式（可选第三道） | 肯定→否定 / 陈述→一般疑问 / 划线提问 | 只在该 tag 支持时用 |

## 三种变式模板（按 2–3 道取）

1. **换主语**：`Mike ____ (visit) his uncle last Sunday.`
2. **换时间状语 + 场景**：`We ____ (visit) the museum two days ago.`
3. **换句式**：`He didn't ____ (visit) his uncle yesterday.` ／ `____ you ____ (visit) ...?`

## 输出

```json
{
  "schema": "yl6.variant/1",
  "source_id": "6A-U2-C-01",
  "tag": "past_form",
  "kept": ["考点：一般过去时过去式", "句式：主句陈述"],
  "changed": ["主语", "时间状语"],
  "variants": [
    {
      "id": "6A-U2-C-01-v1",
      "derived_from": "6A-U2-C-01",
      "unit": "Unit 2",
      "tag": "past_form",
      "type": "choice",
      "stem": "Mike ____ his uncle last Sunday.",
      "options": {"A": "visit", "B": "visits", "C": "visited", "D": "visiting"},
      "answer": "C",
      "answer_text": "visited",
      "accept": [],
      "explain": "last Sunday 是过去时间，visit 是规则动词，加 -ed。",
      "point": "一般过去时：规则动词 + -ed。",
      "trap": "主语变第三人称后手滑选 visits。",
      "level": "中"
    }
  ]
}
```

## 硬规则

1. **答案跟着改**：改了主语/时间/动词，`answer` 和 `answer_text` 必须重算，不能沿用原题。
2. **三道变式的正确答案不能都是同一个字母**，至少换两次位置。
3. **不提升难度**：变式与原题同一档；要加难就明说「第三道加难」，并把 `level` 标 `难`。
4. **变式必须能独立作答**，题干里不许出现「和上一题一样」「改为」这类依赖原题的词，除非是句型转换题。
5. **选项去重**：同形词（`his`）会让两个选项一样，出库前查一遍。
6. **do/doing 的题不许互换**：原题考 `finish doing`，变式还得是 doing，不能变成 `want to do`——那是换考点。
7. `be born` 的变式只能换主语和时间（`I was born in ... / They were born in ...`），不许变出 `did born`。

## 自检

- 每道变式的 `stem` 单独读一遍，句子成不成立？（六年级句子靠读，不靠语法树）
- 把 `answer` 遮住自己做一遍，做出来的和标注的一致吗？
- `variants` 长度 2–3，`source_id` 回填了吗？

## 踩过的坑

1. **换主语最容易翻车主谓一致**：`He don't` 在原题里看不出来，变式里就成了新错因。
2. **过去时变式误用 `now`**：句式换了，时间状语没跟着换，句子自相矛盾。
3. **只换人名等于没变**：`Mike → Liu Tao` 学生闭眼都会，属于无效变式；至少换两个维度。
4. **变式解析照抄原题解析**：解析要点出这次改动的那个线索（`last Sunday` / 主语第三人称）。
