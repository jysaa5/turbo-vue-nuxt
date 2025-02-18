<template>
  <div class="grid-container">
    <div
      v-for="item in items"
      :key="item.id"
      :ref="(el) => setItemRef(el as HTMLElement, item)"
      class="grid-item"
      :style="{ backgroundImage: `url(${item.src})` }"
    >{{ item.id }}</div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch, watchEffect } from 'vue';
import {
  draggable,
  dropTargetForElements,
} from '@atlaskit/pragmatic-drag-and-drop/element/adapter';
import {combine} from '@atlaskit/pragmatic-drag-and-drop/combine'

type Item = {
  id: string;
  src: string;
};

const items = ref<Item[]>([
  { id: 'battery', src: './icons/battery.png' },
  { id: 'drill', src: './icons/drill.png' },
  { id: 'koala', src: './icons/koala.png' },
  { id: 'ui', src: './icons/ui.png' },
  { id: 'wallet', src: './icons/wallet.png' },
  { id: 'yeti', src: './icons/yeti.png' }
]);

const itemRefs = ref<Record<string, HTMLElement | null>>({});
const setItemRef = (el: HTMLElement | null, item: Item) => {
  if (el) {
    itemRefs.value[item.id] = el;
  }
};

  watchEffect( () => {
    Object.values(itemRefs.value).forEach((el) => {
      if (!el) return;

      combine(
        draggable({
          element: el,
          getInitialData: () => ({ type: 'grid-item', id: el.dataset.id }),
        }),
        dropTargetForElements({
          element: el,
          getData: () => ({ id: el.dataset.id }),
          onDrop({ source }) {
            const sourceId = source.data.id;
            const targetId = el.dataset.id;

            if (!sourceId || !targetId || sourceId === targetId) return;

            const oldIndex = items.value.findIndex((item) => item.id === sourceId);
            const newIndex = items.value.findIndex((item) => item.id === targetId);
            const movedItem = items.value.splice(oldIndex, 1)[0];
            items.value.splice(newIndex, 0, movedItem);
          },
        })
      );
    });
  });

onMounted(() => {
});
</script>

<style>
.grid-container {
  display: grid;
  grid-template-columns: repeat(3, 96px);
  gap: 10px;
  padding: 20px;
}
.grid-item {
  width: 96px;
  height: 96px;
  background-size: cover;
  background-position: center;
  border-radius: 8px;
  cursor: grab;
}
</style>
