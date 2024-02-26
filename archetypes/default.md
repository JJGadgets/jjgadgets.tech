---
title: "{{ replace .File.ContentBaseName "-" " " | title }}"
date: "{{ .Date }}"
lastmod: "{{ .Date }}"
slug: "{{ time.Now.Format "2006" }}/{{ .File.ContentBaseName }}"
# categories: []
---
