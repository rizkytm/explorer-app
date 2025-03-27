<template>
  <li>
    <div
      @click="toggleExpand"
      class="tree-item d-flex align-items-center"
      :class="{ expanded: expanded }"
    >
      <i
        v-if="hasChildren"
        class="bi me-2"
        :class="expanded ? 'bi-chevron-down' : 'bi-chevron-right'"
      ></i>
      <span>{{ item.name }}</span>
    </div>

    <ul
      v-if="hasChildren"
      class="list-unstyled ps-3"
      :class="{ 'd-none': !expanded }"
    >
      <TreeItem
        v-for="child in item.subfolders"
        :key="child.label"
        :item="child"
        @show-subfolder="handleShowSubfolder"
        @selected-folder="handleSelectedFolder"
        @click.stop="fetchSubfolder(child.id, child)"
      ></TreeItem>
    </ul>
  </li>
</template>

<script setup>
import axios from 'axios';
import { ref, computed, defineProps } from 'vue';

const emit = defineEmits(['fetchSubfolder', 'showSubfolder', 'selectedFolder']);

const props = defineProps({
  item: {
    type: Object,
    required: true,
  },
});

const expanded = ref(false);

const hasChildren = computed(() => {
  return (
    props.item?._count?.subfolders > 0 || props.item?.subfolders?.length > 0
  );
});

const toggleExpand = () => {
  if (hasChildren.value) {
    expanded.value = !expanded.value;
  }
  fetchSubfolder(props.item.id, props.item)
  emit('showSubfolder', props.item?.subfolders);
};

const handleShowSubfolder = (value) => {
  emit('showSubfolder', value);
}

const handleSelectedFolder = (value) => {
  emit('selectedFolder', value);
}

// Method to fetch subfolder
async function fetchSubfolder(id, parent = null) {
  try {
    const API_URL = 'http://localhost:3000';
    const response = await axios.get(`${API_URL}/folders/${id}/subfolders`);
    parent.subfolders = response.data;
    emit('showSubfolder', response.data);
    emit('selectedFolder', parent);
  } catch (err) {
    console.error(err);
  }
}
</script>

<style scoped>
.tree-item {
  cursor: pointer;
  user-select: none;
}

.tree-item:hover {
  background-color: #f8f9fa;
}

.bi {
  font-size: 0.8rem;
}
</style>
