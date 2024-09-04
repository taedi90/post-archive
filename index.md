---
title: Index
draft: false
tags:
  - example-tag
---
```dataview
TABLE dateformat(date, "yyyy-MM-dd") as date
FROM "/"
SORT date desc
WHERE draft = false
```