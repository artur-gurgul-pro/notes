---
layout: default
title: Regex - notes
---

How to match “any character” in the regular expression

- `.` &rarr; any char except newline
- `\.` &rarr; the actual dot character
- `.?` = `.{0,1}` &rarr; match any char except newline zero or one times
- `.*` = `.{0,}` &rarr; match any char except newline zero or more times
- `.+` = `.{1,}` &rarr; match any char except newline one or more times