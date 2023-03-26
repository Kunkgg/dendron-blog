---
id: yp8m25sacaz05k3z0gd2yrf
title: 如何把多个仓库合并为一个
desc: ''
updated: 1679794984893
created: 1679794756088
---

## 步骤

```
1. git remote add <frontend> <frontend-repo-url>
1. git fetch <frontend>
1. git checkout -b <frontend> <frontend/main>
1. git checkout master && git merge <frontend> --allow-unrelated-histories
```