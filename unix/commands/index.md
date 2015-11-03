---
layout: bootstrap
title: UNIX commands
type: page
nav: pages
---

# UNIX commands

## ファイル・ディレクトリ検索

### ディスク全体の使用量

```
$ sudo du -h --max-depth=1 / 2>/dev/null
```

### ディスク使用量の多いディレクトリとファイルを調べる

```
$ sudo du -S -a -m / | sort -nr | head -20
```

### ファイルをサイズ順に過去7日間で変更されたものから表示

```
$ sudo ls -S -all -k `sudo find / -size +100k -mtime -7 -print` | head -20
```

## Audit

### Auditログを日付を変換して表示

```
$ sudo ausearch -i | grep -v '^\-' | more
$ sudo ausearch -i --input /var/log/audit/audit.log | grep -v '^\-' | more
```

## 履歴

### 直前の履歴を削除

```
$ history -d $(history | head -n -1 | wc -l)
```
