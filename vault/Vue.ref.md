---
id: jo94tw950js1ofakzoqezna
title: ref
desc: ''
updated: 1686363643843
created: 1686363375249
---

### 通过 ref 获取 DOM 节点或子组件实例引用

```
<div>
  <h1 ref="hello1">Hello</h1>
  <h1 ref="hello2">World</h1>

  <CustomCounter ref="counter1"></CustomCounter>
  <CustomCounter ref="counter2"></CustomCounter>
</div>
```

可以通过 `this.$refs.[ref_name]｀ 获取