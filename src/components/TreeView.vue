<template>
  <ul class="tree-view list-unstyled">
    <TreeItem
      v-for="item in items"
      :key="item.id"
      :item="item"
      @show-subfolder="handleEmit"
      @selected-folder="handleFolder"
    ></TreeItem>
  </ul>
</template>

<script setup>
import { defineProps } from 'vue';
import TreeItem from './TreeItem.vue';

const emit = defineEmits(['fetchSubfolder', 'showSubfolder', 'selectedFolder']);

const props = defineProps({
  items: {
    type: Array,
    required: true,
  },
});

const handleEmit = (subfolder) => {
  emit('showSubfolder', subfolder);
};

const handleFolder = (parent) => {
  emit('selectedFolder', parent);
};
</script>

<style scoped>
.tree-view {
  margin: 0;
  padding: 0;
}
</style>
