---
id: 6ge2ff3sl9vlfq96jic1uop
title: transition
desc: ''
updated: 1686401655558
created: 1686401286789
---

### Vue 内置的 Transition 组件通过在被包裹的组件或标签状态发生变化时, 利用钩子函数在适当的时刻添加和删除 css class 实现过渡动画效果

#### 添加/显示时

- v-enter[-from], 开始状态
- v-enter-active, 全过程
- v-enter-to, 结束状态

#### 删除/隐藏时

- v-leave[-from], 开始状态
- v-leave-active, 全过程
- v-leave-to, 结束状态

https://cn.vuejs.org/guide/built-ins/transition.html#transition-between-components

### 示例代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vue Transition</title>
    <script src="./vue.js"></script>
    <style>
        /* 显示 */
        /* .fade-enter {
            opacity: 0;
        }
        .fade-enter-active {
            transition: opacity 1s;
        }
        */

        /* 隐藏 */
        /*
        .fade-leave-to {
            opacity: 0;
        }
        .fade-leave-active {
            transition: opacity 1s;
        } */

        .fade-enter,
        .fade-leave-to {
            opacity: 0;
        }

        .fade-enter-active,
        .fade-leave-active {
            transition: opacity 1s;
        }

    </style>
</head>
<body>
    <div id="app">
        <transition name="fade">
            <h1 v-if="show">Hello world</h1>
        </transition>

        <button @click="toggleTitle">Toggle Title</button>
    </div>
    <script>
        var vm = new Vue({
            el: "#app",
            data: {
                show: true,
            },
            methods: {
                toggleTitle: function() {
                    this.show = !this.show
                }
            }
        })
    </script>
    
</body>
</html>
```