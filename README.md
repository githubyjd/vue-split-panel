# split panel vue3 ts 面板分割

# 使用环境 ：VUE3、TS

```
    1. vue: "^3.3.4"
    2. vite: "^4.4.6"
    3. npm : 10.2.0
    4. node : v21.1.0
```

# 使用教程：

## 1. npm 安装插件:

```
npm install -S @yjd/vue-split-panel
```

## 2. script 引入挂载:

```
<script setup lang="ts">
import SplitPanel from "@yjd/vue-split-panel/SplitPanel.vue";
</script>
```

## 3. template 具体使用方式:

```
<template>
  <div>
    <SplitPanel/>
    <div style="height: 5px;background-color:rgba(30,31,34,0.99)"></div>
    <SplitPanel mode="vertical">
      <template #start>
        <div v-for="i in 40" :key="i">学习如登山，不畏艰难，持之以恒，必能登峰造极。 {{ i }}</div>
      </template>
      <template #end>
        <div v-for="i in 40" :key="i">坚持是学习的最好伙伴，只要你不放弃，知识的大门总会为你敞开。 {{ i }}</div>
      </template>
    </SplitPanel>
  </div>
</template>

```

## 截至 2024-04-21 正在开发中 ，使用过程如有困难可以发邮箱联系我改进：66085206@qq.com