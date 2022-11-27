<template>
  <div id="app">
    <!-- Item固定高度 -->
    <!-- <VirtualList :listData="data" :itemSize="100"/> -->

    <!-- Item不定高度 -->
    <VirtualList2 :listData="data" v-slot="slotProps">
      <Item :item="slotProps.item" />
    </VirtualList2>
  </div>
</template>
  
<script>
import VirtualList from "./components/item固定高度/VirtualList.vue";


import VirtualList2 from "./components/item不定高度/VirtualList.vue"
import Item from "./components/item不定高度/item.vue"
/* 固定高度 */
// let d = [];
// for (let i = 0; i < 100000; i++) {
//   d.push({ id: i, value: i });
// }
/* 不定高度 */
import faker from "faker";

let data = [];
for (let id = 0; id < 1000; id++) {
  data.push({
    id,
    value: faker.lorem.sentences() // 长文本
  });
}

export default {
  name: "App",
  data() {
    return {
      data: data
    };
  },
  components: {
    VirtualList,
    VirtualList2,
    Item
  }
};
</script>
  
<style>
html {
  height: 100%;
}

body {
  height: 100%;
  margin: 0;
}

#app {
  height: 100%;
}
</style>
  