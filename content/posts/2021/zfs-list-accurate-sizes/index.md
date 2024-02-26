---
title: "ZFS list rough cheatsheet"
description: "Accurately list used space for specific queries, such as each snapshot or each dataset, using `zfs list`"
date: "2021-12-05"
slug: "2024/zfs-list-rough-cheatsheet"
categories:
  - "Storage"
  - "Servers"
  - "Homelab"
tags:
  - "ZFS"
---
### List & sort ascending only total used space by all snapshots of individual datasets:
```shell
zfs list -r -o name,usedsnap -s usedsnap
```

### List & sort ascending actual disk usage = disk usage from specific dataset without children dataset's storage
```shell
zfs list -r -o name,usedds -s usedds
```

### List all space related columns available in `zfs list`:
```shell
zfs list -ro space
```

`-s [column name]` sorts `zfs list` output in ascending order by selected column's data, `-S [column name]` sorts the same descending.