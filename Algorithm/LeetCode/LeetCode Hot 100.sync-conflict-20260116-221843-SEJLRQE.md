---
project: LeetCode
tags:
  - LeetCode
  - Algorithm
  - DataStructure
---


```base
filters:
  and:
    - "!leetcode.isEmpty()"
    - file.tags.containsAny("Hot100")
properties:
  note.leetcode:
    displayName: 题目
  file.name:
    displayName: 题目
  note.difficulties:
    displayName: 难度
  note.link:
    displayName: 链接
views:
  - type: table
    name: LeetCode Hot 100
    order:
      - file.name
      - difficulties
      - file.tags
      - link
    sort:
      - property: leetcode
        direction: ASC
    columnSize:
      file.name: 235
      note.difficulties: 84
      file.tags: 235

```

