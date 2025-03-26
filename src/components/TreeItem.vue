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
        @click="getSubfolder(child.id)"
      ></TreeItem>
    </ul>
  </li>
</template>

<script setup>
import { ref, computed, defineProps } from 'vue';

const emit = defineEmits(['fetchSubfolder']);

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
  //   getSubfolder();
  emit('fetchSubfolder', props.item.id);
  if (hasChildren.value) {
    expanded.value = !expanded.value;
  }
};

const getSubfolder = (childId) => {
  emit('fetchSubfolder', childId);
};
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
