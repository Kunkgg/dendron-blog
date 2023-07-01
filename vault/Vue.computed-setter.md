---
id: tyyzu0ax4wuhswnknswivua
title: computed-setter
desc: ''
updated: 1686379237232
created: 1686379009675
---

### 注意

1. 定义的 computed 不能直接通过赋值的方式改变, 如果需要更新 computed, 可以通过 setter 方法改变其依赖, 依赖改变后, computed 会重新计算更新

2. 如果试图在 computed setter 方法中对该 computed 赋值, 会陷入无线调用 setter 的死循环

### 示例代码

```html
<body>
    <div id="app">
        <h3>{{firstName}}</h3>
        <h3>{{lastName}}</h3>
        <h3>{{fullName}}</h3>
    </div>
    <script>
        var vm = new Vue({
            el: "#app",
            data: {
                firstName: "Jun",
                lastName: "Kung"
            },
            computed: {
                fullName: {
                    get: function () {
                        return this.firstName + " " + this.lastName
                    },
                    set: function (newValue) {
                        var names = newValue.split(" ")
                        this.firstName = names[0]
                        this.lastName = names[1]
                    }
                }
            }
        }
        )
    </script>
</body>
```