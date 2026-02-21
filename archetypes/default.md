---
date: '{{ .Date }}'
draft: false
slug: '{{ substr (md5 (printf "%s%d" .Name now.Unix)) 0 8 }}'
type: posts
title: '{{ replace .File.ContentBaseName "-" " " | title }}'
collections: ""
categories: []
---

<!--more-->