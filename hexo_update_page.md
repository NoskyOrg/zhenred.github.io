---
title: hexo网页内容自动更新
categories:                
- 教程
- Hexo
date: 2018-11-5
---

hexo发布内容，需要进行`hexo g`，将之进行脚本处理
所以发布文章会很麻烦
我的方法是，通过文本编辑器将编辑好的`md文件`通过SFTP上传，脚本检测到有更新，自动执行命令

原理为：
脚本自动检测hexo目录下，有无增加的新文档，如果发现，就会进行`hexo g`
由于脚本只是通过目录下的文件数量进行处理，所以添加一个`.delete_lixyz.md`，
这是一个隐藏文件，hexo不会进行处理

使用方法：
在每次需要更新文档时，只要将`.delete_lixyz.md`同时上传，让脚本检测到文件数量的更改，自动更新内容。

脚本配合计划任务，每十秒检测一次

```bash
* * * * * bash /www/test.sh
* * * * * sleep 10; bash /www/test.sh
* * * * * sleep 20; bash /www/test.sh
* * * * * sleep 30; bash /www/test.sh
* * * * * sleep 40; bash /www/test.sh
* * * * * sleep 50; bash /www/test.sh
```

脚本内容

```bash
#!/bin/bash

countExists=$(ls -la /www/hexo/source/_posts | wc -l)
countTxt=$(cat /www/count_lixyz)

echo $countExists

if (( ${countExists} != ${countTxt} ))
then
    cd /www/hexo && hexo g
    echo ${countExists} > /www/count_lixyz
else
    echo a > /tmp/a
fi

rm -f /www/hexo/source/_posts/.delete_lixyz.md

```
