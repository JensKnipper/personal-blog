---
draft: true
---

https://www.hashbangcode.com/article/creating-presentations-markdown-marp

how to generate slides from markdown with marp

https://github.com/marp-team/marp

markdown is perfect for writing, taking notes etc.
formatting is simple and does not distract you unlike other writing tools


list of themes

list of possible settings

Install marpit CLI
```bash
npm install -g @marp-team/marp-cli
```

To view the slides `VSCode` with `Marp for VS Code` Plugin is recommended.
https://github.com/marp-team/marp-vscode

Generate HTML
```bash
marp intro.md
```

Generate PDF - needs chromium installed!
```bash
marp intro.md --pdf --allow-local-files
```


```markdown
---
marp: true
title: presentation title
description: presentation description
theme: uncover
paginate: true
_paginate: false # do not paginate the current page
header: "**Your Company** presentation title"
footer: "footer"
---

# Title slide
Some description for the title

---
# Next slide
* listing
** more
** and more

---
```

