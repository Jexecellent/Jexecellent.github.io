---
title: 删除github仓库中的文件夹命令
---

记录删除github仓库中的文件夹命令，git的使用一直没有顺风顺水，命令的参数都不知道是做什么的，先记录下来以备查阅~

``` bash
$ git rm -h
```

用法：git rm [<选项>] [–] <文件>…

| 参数      |    意义 | 
| :-------- | --------:| 
| -n, –dry-run  | 演习 | 
| -q, –quiet     |   不列出删除的文件 | 
| –cached      |    只从索引区删除 | 
| -f， –force     |    忽略文件更新状态检查 | 
| -r     |    允许递归删除 | 
| –ignore-unmatch     |    即使没有匹配，也以零状态退出 | 

## 操作案例

``` bash
$ git rm -r --cached  "directory-name"  // 要删除的文件夹名字
$ git commit -m "remove new gitignore directory"
$ git push origin master
```
