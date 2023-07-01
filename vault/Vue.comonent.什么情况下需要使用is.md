---
id: hso45cbtrxmc5jmddfj8ijl
title: 什么情况下需要使用is
desc: ''
updated: 1686362683441
created: 1686362158676
---

### 解决问题

html5 规范对一些标签的使用做了约束, 比如在 `<tbody>` 标签中的子标签只可以是 `<tr>` 标签.

只有如果我们自定义了一个组件来表示表格中的一行, 比如自定义了子组件 `row`, 模版: `<tr><td>{{content}}<td></tr>`

`<tbody>` 中如果直接使用 `<row>` 标签部分浏览器可能会解析 DOM 出错, 这种情况可以使用 `is` 解决

```
# 这样的写法因为不符合 html5 规范, 解析 DOM 可能会出错
<tbody>
  <row></row>
  <row></row>
  <row></row>
</tbody>
```

```
# 这样的写法因为不符合 html5 规范, 解析 DOM 可能会出错
<tbody>
  <tr is="row"></tr>
  <tr is="row"></tr>
  <tr is="row"></tr>
</tbody>
```
