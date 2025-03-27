<script setup lang="ts">
import { ref, onMounted } from 'vue';
import axios from 'axios';
import TreeView from './components/TreeView.vue';

// Types for TypeScript support
interface Folder {
  id: string;
  name: string;
  parentId?: string | null;
  description?: string;
  subfolders?: number;
}

// State management
const rootFolders = ref<Folder[]>([]);
const selectedFolder = ref<Folder | null>(null);
const subfolders = ref<Folder[]>([]);
const isLoading = ref(false);
const error = ref<string | null>(null);
const API_URL = 'http://localhost:3000';

// Fetch root folders on component mount
onMounted(async () => {
  try {
    isLoading.value = true;
    const response = await axios.get(`${API_URL}/folders`);
    rootFolders.value = response.data;
  } catch (err) {
    error.value = 'Failed to load root folders';
    console.error(err);
  } finally {
    isLoading.value = false;
  }
});

function showSubfolder(subfolderItem) {
  subfolders.value = subfolderItem;
}

function setSelectedFolder(folder) {
  selectedFolder.value = folder;
}
</script>

<template>
  <div class="container my-4 folder-explorer">
    <!-- Error Alert -->
    <div
      v-if="error"
      class="alert alert-danger alert-dismissible fade show"
      role="alert"
    >
      {{ error }}
      <button
        type="button"
        class="btn-close"
        data-bs-dismiss="alert"
        aria-label="Close"
      ></button>
    </div>

    <!-- Main Explorer Container -->
    <div class="row">
      <!-- Left Panel: Folder Structure -->
      <div class="col-md-4 border-end mb-4">
        <div class="d-flex justify-content-between align-items-center mb-3">
          <h4 class="mb-0">Folder Structure</h4>
        </div>

        <div>
          <tree-view
            :items="rootFolders"
            @show-subfolder="showSubfolder"
            @selected-folder="setSelectedFolder"
          ></tree-view>
        </div>
      </div>

      <!-- Right Panel: Subfolders -->
      <div class="col-md-8">
        <div v-if="selectedFolder" class="card">
          <div
            class="card-header d-flex justify-content-between align-items-center"
          >
            <h4 class="mb-0">Subfolders of {{ selectedFolder.name }}</h4>
          </div>

          <div v-if="subfolders.length" class="list-group list-group-flush">
            <div
              v-for="subfolder in subfolders"
              :key="subfolder.id"
              class="list-group-item d-flex justify-content-between align-items-center"
            >
              <span>
                <i class="bi bi-folder me-2"></i>
                {{ subfolder.name }}
              </span>
            </div>
          </div>

          <div v-else class="card-body text-center text-muted">
            No subfolders found
          </div>
        </div>

        <div v-else class="alert alert-info text-center">
          Select a folder to view its contents
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.folder-explorer {
  height: 100vh;
  overflow: hidden;
}
</style>
