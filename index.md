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


<%*
const dv = this.DataviewAPI
const escapePipe = s => new String(s).replace(/\|/, '\\|') // required for links in Markdown table
/*
  You may want to collect your utilities in a central Javascript module below
  the Obsidian vault, e.g. in a file lib/utils.js.
  The file can be included via
    const { escapePipe } = require(app.vault.adapter.basePath + "/lib/utils.js")
*/
%>

| Project | Tasks |
| ------- | ----- |
<%
// Note that dv.table() cannot be used as it creates HTML but we want Markdown.
dv.pages("!#template")
  .where(p => p.doctype == 'project') // a custom attribute, specified like "doctype:: project"
  .map(p => {
    let project = escapePipe(dv.fileLink(p.file.path))
    let tasklist = p.file.tasks.map(t => `${t.text} ${escapePipe(t.link)}`).join('<br>')
    return `|${project}|${tasklist}|`
  })
  .join("\n")
%>