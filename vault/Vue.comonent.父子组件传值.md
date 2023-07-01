---
id: e19myf3atoptslqzmsszxw5
title: 父子组件传值
desc: ''
updated: 1686367164976
created: 1686364434658
---

### 单向数据流

子组件不能修改父子组件传递的数据, 需要先复制

### 父 -> 子, props

### 子 -> 父, this.$emit("event", ...)

代码示例

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>父子组件传值 Props $emit</title>
    <script src="./vue.js"></script>
    <style>
        .btn {
            width: 50px;
            height: 30px;
        }
    </style>
  </head>
  <body>
    <div id="app">
      <counter :count="0" @increment="handelIncrement"></counter>
      <counter :count="10" @increment="handelIncrement"></counter>
      <p>
        Total: {{ total }}
      </p>
    </div>
  </body>
  <script>
    var counter = {
      template: '<button @click="increment" class="btn">{{number}}</button>',
      props: ["count"],
      data: function () {
        return {
          number: this.count,
        };
      },
      methods: {
        increment: function () {
          let step = 1;
          this.number += step;
          this.$emit("increment", step);
        },
      },
    };

    var vm = new Vue({
      el: "#app",
      components: {
        counter: counter,
      },
      data: {
        total: 10,
      },
      methods: {
        handelIncrement: function (step) {
            this.total += step;
        },
      },
    });
  </script>
</html>
```

