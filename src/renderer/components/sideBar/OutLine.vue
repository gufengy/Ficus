<template>
  <div class="flex justify-between text-gray-600 text-sm pl-2 relative">
    <ul class="space-y-1 w-full overflow-y-auto" v-if="items !== undefined && items.length !== 0">
      <OutLineItem
          :item="item"
          v-for="(item, index) in items"
          :key="index"
          @click="getIndex(item)"
      />
    </ul>
    <div v-if="items === undefined || items.length === 0">
      <div style="font-size: 12px; font-family: 'Noto Sans SC'" class="pl-1">
        大纲内容为空。
      </div>
    </div>
  </div>
</template>

<script>

import OutLineItem from '@/renderer/components/sideBar/OutLineItem'
import bus from 'vue3-eventbus'
export default {
  name: 'OutLine',
  components: { OutLineItem },
  props: {
    items: {
      type: Array,
      required: true
    }
  },
  setup () {
    let titles = [-1, -1, -1, -1, -1, -1]

    bus.on('addToTitles', (child) => {
      titles[child.level - 1] = child.id
    })

    function getIndex (item) {
      titles[item.level - 1] = item.id
      bus.emit('scrollToHeading', { info: titles })
      titles = [-1, -1, -1, -1, -1, -1]
    }

    return {
      getIndex
    }
  }
}
</script>

<style scoped>
::-webkit-scrollbar {
  width: 5px;
  border-radius: 8px;

}

::-webkit-scrollbar-thumb {
  border-radius: 8px;
  background-color: #b7b7b7;
}

::-webkit-scrollbar-thumb:hover {
  background-color: #777777;
}
</style>
