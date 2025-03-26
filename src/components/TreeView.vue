<template>
  <ul class="tree-view list-unstyled">
    <TreeItem
      v-for="item in items"
      :key="item.id"
      :item="item"
      @fetch-subfolder="handleEmit"
    ></TreeItem>
  </ul>
</template>

<script setup>
import { defineProps } from 'vue';
import TreeItem from './TreeItem.vue';

const emit = defineEmits(['fetchSubfolder']);

const props = defineProps({
  items: {
    type: Array,
    required: true,
  },
});

const handleEmit = (itemId) => {
  emit('fetchSubfolder', itemId);
};
</script>

<style scoped>
.tree-view {
  margin: 0;
  padding: 0;
}
</style>
