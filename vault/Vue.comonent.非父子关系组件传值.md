---
id: 4vkzbpzztrthcw2jja8p488
title: 非父子关系组件传值
desc: ''
updated: 1686377975933
created: 1686369229869
---


### 典型非父子组件关系:

- 兄弟关系
- 祖孙关系
- 其他非父子组件关系

### 非父子组件传值解决方案

- VueX
- Bus 总线

### Bus 总线方案代码示例

```html
<body>
    <div id="app">
        <child content="Jun"></child>
        <child content="Kung"></child>
    </div>

    <script>
        Vue.prototype.bus = new Vue();

        Vue.component("child", {
            template: "<h1 @click='handleClick'>{{myContent}}</h1>",
            props: ['content'],
            data: function() {
                return {
                    myContent: this.content
                }
            },
            methods: {
                handleClick: function () {
                    this.bus.$emit("change", this.myContent)
                }
            },
            mounted: function() {
                let that = this
                this.bus.$on("change", function (data) {
                    that.myContent = data
                })
            }
        })

        var vm = new Vue({
            el: "#app",
        })
    </script>
</body>
```
