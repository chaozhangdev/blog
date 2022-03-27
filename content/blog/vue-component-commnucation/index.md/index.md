---
title: Vue Component Communication
date: 2021-06-29
description: 5 common methods in VUE component communication.
---

### 1. props / $emit

The parent component passes data to the child component through `props`, and the child component can communicate to the parent component through `$emit`.

#### The parent component passes the value to the child component

The following is an example to illustrate how the parent component transmits data to the child component: How to get the data in the parent component section.vue in the child component article.vue articles: `['Introduction to Web Dev', 'Computer Network', 'CSS in 30s']`

```vue
// parent component
<template>
  <div class="section">
    <com-article :articles="articleList"></com-article>
  </div>
</template>

<script>
import comArticle from "./test/article.vue"
export default {
  name: "HelloWorld",
  components: { comArticle },
  data() {
    return {
      articleList: [
        "Introduction to Web Dev",
        "Computer Network",
        "CSS in 30s",
      ],
    }
  },
}
</script>
```

```vue
// child component
<template>
  <div>
    <span v-for="(item, index) in articles" :key="index">{{ item }}</span>
  </div>
</template>

<script>
export default {
  props: ["articles"],
}
</script>
```

> Summary: props can only be passed from the upper-level components to the next-level components (parent-child components), the so-called one-way data flow. Moreover, the prop is read-only and cannot be modified, and all modifications will be invalidated and warned.

#### Child component passes value to parent component

My own understanding of $emit is this: $emit binds a custom event, when this statement is executed, the parameter arg is passed to the parent component, and the parent component monitors and receives parameters through v-on. Through an example, how to transfer data from the child component to the parent component.
On the basis of the previous example, click on the item of the ariticle rendered on the page, and the subscript displayed in the array in the parent component.

```vue
// parent component
<template>
  <div class="section">
    <com-article
      :articles="articleList"
      @onEmitIndex="onEmitIndex"
    ></com-article>
    <p>{{ currentIndex }}</p>
  </div>
</template>

<script>
import comArticle from "./test/article.vue"
export default {
  name: "HelloWorld",
  components: { comArticle },
  data() {
    return {
      currentIndex: -1,
      articleList: [
        "Introduction to Web Dev",
        "Computer Network",
        "CSS in 30s",
      ],
    }
  },
  methods: {
    onEmitIndex(idx) {
      this.currentIndex = idx
    },
  },
}
</script>
```

```vue
// child component
<template>
  <div>
    <div
      v-for="(item, index) in articles"
      :key="index"
      @click="emitIndex(index)"
    >
      {{ item }}
    </div>
  </div>
</template>

<script>
export default {
  props: ["articles"],
  methods: {
    emitIndex(index) {
      this.$emit("onEmitIndex", index)
    },
  },
}
</script>
```

### 2. $children / $parent

```vue
// parent component
<template>
  <div class="hello_world">
    <div>{{ msg }}</div>
    <com-a></com-a>
    <button @click="changeA">change child's value</button>
  </div>
</template>

<script>
import ComA from "./test/comA.vue"
export default {
  name: "HelloWorld",
  components: { ComA },
  data() {
    return {
      msg: "Welcome",
    }
  },
  methods: {
    changeA() {
      // get child component A
      this.$children[0].messageA = "this is new value"
    },
  },
}
</script>
```

```vue
// child component
<template>
  <div class="com_a">
    <span>{{ messageA }}</span>
    <p>Parent component's value: {{ parentVal }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      messageA: "this is old",
    }
  },
  computed: {
    parentVal() {
      return this.$parent.msg
    },
  },
}
</script>
```

> Pay attention to the boundary conditions, such as taking `$parent` on `#app` to get an instance of `new Vue()`, taking `$parent` on this instance to get `undefined`, and taking `$children` on the bottom of the child component is an empty array. Also note that the values of `$parent` and `$children` are not the same, the value of `$children` is an array, and `$parent` is an object

Conclusion: The above two methods are used for communication between parent and child components, and it is more common to use props to communicate between parent and child components; neither of them can be used for communication between non-parent and child components.

### 3. eventBus

eventBus is also known as event bus. It can be used as a concept of communication bridge in vue, just like all components share the same event center, which can be registered to send events or receive events, so components can notify other components.

> EventBus also has inconveniences. When the project is large, it is easy to cause disasters that are difficult to maintain.

Few steps to use eventBus to achieve data communication between components:

#### 1. Initialization

Create an event bus and export it so that other modules can use or monitor it.

```vue
import Vue from 'vue'; export const EventBus = new Vue();
```

#### 2. Send Event

Suppose you have two components: additionNum and showNum, these two components can be sibling components or parent-child components; here we take sibling components as an example:

```vue
<template>
  <div>
    <show-num-com></show-num-com>
    <addition-num-com></addition-num-com>
  </div>
</template>

<script>
import showNumCom from "./showNum.vue"
import additionNumCom from "./additionNum.vue"
export default {
  components: { showNumCom, additionNumCom },
}
</script>
```

```vue
// addtionNum.vue : send event
<template>
  <div>
    <button @click="additionHandle">+ Add</button>
  </div>
</template>

<script>
import { EventBus } from "./event-bus.js"
export default {
  data() {
    return {
      num: 1,
    }
  },
  methods: {
    additionHandle() {
      EventBus.$emit("addition", {
        num: this.num++,
      })
    },
  },
}
</script>
```

#### 3. Receive Event

In this way, click the add button in the component addtionNum.vue, and use the passed num in showNum.vue to display the result of the summation.

```vue
// showNum.vue : receive event

<template>
  <div>Sum: {{ count }}</div>
</template>

<script>
import { EventBus } from "./event-bus.js"
export default {
  data() {
    return {
      count: 0,
    }
  },
  mounted() {
    EventBus.$on("addition", param => {
      this.count = this.count + param.num
    })
  },
}
</script>
```

#### 4. Remove Event Listener

```vue
import { eventBus } from 'event-bus.js'; EventBus.$off('addition', {});
```

### 4. localStorage / sessionStorage

It is relatively simple to use, but the disadvantage is that the data and status are messy and not easy to maintain. Get data through `window.localStorage.getItem(key)` and store data through `window.localStorage.setItem(key, value)`

> Note that using `JSON.parse()` / `JSON.stringify()` for data format conversion `localStorage` / `sessionStorage` can be combined with vuex to achieve persistent storage of data, while using vuex to solve the problem of data and state confusion.

### 5. VueX

Please see [this blog]('../../../../vuex-explanation/index.md') for detailed explination of VueX.
