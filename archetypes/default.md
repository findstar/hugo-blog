---
date: '{{ dateFormat "2006-01-02 15:04:05 Z07:00" .Date }}'
group: blog
image: ''
tags:
- ""
title: "{{ replace .Name "-" " " | title }}"
url: /{{ dateFormat "2006/01/02" .Date }}/{{ .Name }}
type: post
draft: true
---

---
title:       "An Example Post"
subtitle:    ""
description: ""
date:        2018-06-04
author:      ""
image:       ""
tags:        ["tag1", "tag2"]
categories:  ["Tech" ]
---
