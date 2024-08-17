---
id: 92kk0lno5j9satb5pnvm4if
title: Powershell 代理配置
desc: ''
updated: 1723900836735
created: 1723900831552
---

## 函数控制代理

```powershell
# notepad $PROFILE

function proxy_on {$Env:http_proxy="http://127.0.0.1:10809";$Env:https_proxy="http://127.0.0.1:10809"}
function proxy_off {$Env:http_proxy="";$Env:https_proxy=""}
```