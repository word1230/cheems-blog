+++
date = '{{ .Date }}'
draft = false
slug = {{ substr (md5 (printf "%s%d" .Name now.Unix)) 0 8 }}  # 随机生成8位字符串
type = posts
title = '{{ replace .File.ContentBaseName "-" " " | title }}'
collection = ""
categories = ""
+++
