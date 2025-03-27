<script setup lang="ts">
import { ref, onMounted } from 'vue';
import axios from 'axios';
import { Modal } from 'bootstrap';
import TreeView from './components/TreeView.vue';

// Types for TypeScript support
interface Folder {
  id: string;
  name: string;
  parentId?: string | null;
  description?: string;
  subfolders?: number;
}

interface TreeNode {
  id: string;
  subfolders?: TreeNode[];
  name?: string;
}

// State management
const rootFolders = ref<Folder[]>([]);
const selectedFolder = ref<Folder | null>(null);
const subfolders = ref<Folder[]>([]);
const isLoading = ref(false);
const error = ref<string | null>(null);
const API_URL = 'http://localhost:3000';

// TODO: delete
const beverageData = [
  {
    label: 'Beverages',
    children: [
      { label: 'Water' },
      { label: 'Coffee' },
      {
        label: 'Tea',
        children: [
          { label: 'Black Tea' },
          { label: 'White Tea' },
          {
            label: 'Green Tea',
            children: [
              { label: 'Sencha' },
              { label: 'Gyokuro' },
              { label: 'Matcha' },
              { label: 'Pi Lo Chun' },
            ],
          },
        ],
      },
    ],
  },
];

// Modal references
const folderModal = ref<HTMLElement | null>(null);
const deleteModal = ref<HTMLElement | null>(null);

// Form data for new folder
const newFolderName = ref('');
const newFolderDescription = ref('');

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

// Method to select a folder and load its subfolders
async function selectFolder(folder: Folder) {
  try {
    isLoading.value = true;
    selectedFolder.value = folder;
    const response = await axios.get(
      `${API_URL}/folders/${folder.id}/subfolders`
    );
    subfolders.value = response.data;
    selectFolderTree(selectedFolder);
  } catch (err) {
    error.value = 'Failed to load subfolders';
    console.error(err);
  } finally {
    isLoading.value = false;
  }
}

function findNodeById(root, targetId: string): TreeNode | null {
  // if (root.parentId === null && )
  console.log(root.value, targetId);
  for (const child of root.value) {
    console.log('child');
    console.log(child);
    const foundNode = findNodeById(child, targetId);
    if (foundNode) {
      return foundNode;
    }
  }
  // If current node matches, return it
  if (root.id === targetId) {
    return root;
  }

  // If node has children, recursively search
  if (root.subfolders) {
    for (const child of root.subfolders) {
      const foundNode = findNodeById(child, targetId);
      if (foundNode) {
        return foundNode;
      }
    }
  }

  // Node not found
  return null;
}

function getNode(rootFolders, targetId) {
  console.log('rootFolders');
  console.log(rootFolders);
  if (rootFolders.value) {
    for (const child of rootFolders.value) {
      if (child.id === targetId) {
        return child;
      }

      if (child.subfolders) {
        for (const subfolder of child.subfolders) {
          const foundNode = getNode(subfolder, targetId);
          if (foundNode) {
            return foundNode;
          }
        }
      }
    }
  }

  return null;
}

// Method to fetch subfolder
async function fetchSubfolder(id: string) {
  try {
    isLoading.value = true;
    console.log(id);
    const response = await axios.get(`${API_URL}/folders/${id}/tree`);
    console.log(response.data);
    const currentFolderIndex = rootFolders.value.findIndex(
      (folder) => folder.id === id
    );
    const node = getNode(rootFolders, id);
    console.log('node');
    console.log(node);
    if (node) {
      node.subfolders = response.data.subfolders;
    }
    // if (currentFolderIndex != -1) {
    //   rootFolders.value[currentFolderIndex].subfolders =
    //     response.data.subfolders;
    //   selectedFolder.value = rootFolders.value[currentFolderIndex];
    //   subfolders.value = selectedFolder.value.subfolders;
    // } else {
    //   const result = findNodeById(rootFolders, id);
    //   console.log('result');
    //   console.log(result);
    // }
  } catch (err) {
    error.value = 'Failed to load subfolders';
    console.error(err);
  } finally {
    isLoading.value = false;
  }
}

// Method to select a folder tree
async function selectFolderTree(folder: Folder) {
  try {
    isLoading.value = true;
    console.log(folder.value);
    // selectedFolder.value = folder;
    console.log('seletedfolder', selectedFolder.value);
    const response = await axios.get(
      `${API_URL}/folders/${folder.value.id}/tree`
    );
    console.log(response.data);
    const currentFolderIndex = rootFolders.value.findIndex(
      (folder) => folder.id === selectedFolder.value.id
    );
    rootFolders.value[currentFolderIndex].subfolders = response.data.subfolders;
    selectedFolder.value = response.data.subfolders;
  } catch (err) {
    error.value = 'Failed to load subfolders';
    console.error(err);
  } finally {
    isLoading.value = false;
  }
}

// Method to open new folder modal
function openNewFolderModal() {
  if (folderModal.value) {
    const modalInstance = new Modal(folderModal.value);
    modalInstance.show();
  }
}

// Method to create a new folder
async function createFolder() {
  try {
    const newFolder = {
      name: newFolderName.value,
      description: newFolderDescription.value,
      parentId: selectedFolder.value?.id || null,
    };

    const response = await axios.post(`${API_URL}/folders`, newFolder);

    // Update UI based on where folder is created
    if (selectedFolder.value) {
      subfolders.value.push(response.data);
    } else {
      rootFolders.value.push(response.data);
    }

    // Reset form and close modal
    newFolderName.value = '';
    newFolderDescription.value = '';

    if (folderModal.value) {
      const modalInstance = Modal.getInstance(folderModal.value);
      modalInstance?.hide();
    }
  } catch (err) {
    error.value = 'Failed to create folder';
    console.error(err);
  }
}

// Preparation for delete operation
const folderToDelete = ref<Folder | null>(null);

// Method to prepare delete confirmation
function prepareDeleteFolder(folder: Folder) {
  folderToDelete.value = folder;
  if (deleteModal.value) {
    const modalInstance = new Modal(deleteModal.value);
    modalInstance.show();
  }
}

// Method to delete a folder
async function deleteFolder() {
  if (!folderToDelete.value) return;

  try {
    await axios.delete(`${API_URL}/folders/${folderToDelete.value.id}`, {
      params: { force: 'true' },
    });

    // Remove from appropriate list
    if (selectedFolder.value) {
      subfolders.value = subfolders.value.filter(
        (f) => f.id !== folderToDelete.value?.id
      );
    } else {
      rootFolders.value = rootFolders.value.filter(
        (f) => f.id !== folderToDelete.value?.id
      );
    }

    // Close delete modal
    if (deleteModal.value) {
      const modalInstance = Modal.getInstance(deleteModal.value);
      modalInstance?.hide();
    }
  } catch (err) {
    error.value = 'Failed to delete folder';
    console.error(err);
  }
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
      <div class="col-md-4 border-end">
        <div class="d-flex justify-content-between align-items-center mb-3">
          <h4 class="mb-0">Folder Structure</h4>
          <button class="btn btn-primary btn-sm" @click="openNewFolderModal">
            <i class="bi bi-plus"></i> Root Folder
          </button>
        </div>

        <div>
          <!-- <h2>Beverage Tree</h2> -->
          <tree-view
            :items="rootFolders"
            @fetch-subfolder="fetchSubfolder"
          ></tree-view>
        </div>

        <div class="list-group" style="display: none">
          <button
            v-for="folder in rootFolders"
            :key="folder.id"
            class="list-group-item list-group-item-action d-flex justify-content-between align-items-center"
            :class="{ active: selectedFolder?.id === folder.id }"
            @click="selectFolder(folder)"
          >
            <span>
              <i class="bi bi-folder me-2"></i>
              {{ folder.name }}
              <div v-if="folder.subfolders">
                <div v-for="subfolder in folder.subfolders" :key="subfolder.id">
                  {{ subfolder.name }}
                </div>
              </div>
            </span>
            <div>
              <button
                class="btn btn-danger btn-sm me-1"
                @click.stop="prepareDeleteFolder(folder)"
              >
                <i class="bi bi-trash"></i>
              </button>
            </div>
          </button>
        </div>
      </div>

      <!-- Right Panel: Subfolders -->
      <div class="col-md-8">
        <div v-if="selectedFolder" class="card">
          <div
            class="card-header d-flex justify-content-between align-items-center"
          >
            <h4 class="mb-0">Subfolders of {{ selectedFolder.name }}</h4>
            <button class="btn btn-success btn-sm" @click="openNewFolderModal">
              <i class="bi bi-plus"></i> Subfolder
            </button>
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
              <div>
                <button
                  class="btn btn-danger btn-sm"
                  @click="prepareDeleteFolder(subfolder)"
                >
                  <i class="bi bi-trash"></i>
                </button>
              </div>
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

    <!-- New Folder Modal -->
    <div ref="folderModal" class="modal fade" tabindex="-1">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">
              {{ selectedFolder ? 'New Subfolder' : 'New Root Folder' }}
            </h5>
            <button
              type="button"
              class="btn-close"
              data-bs-dismiss="modal"
              aria-label="Close"
            ></button>
          </div>
          <div class="modal-body">
            <form @submit.prevent="createFolder">
              <div class="mb-3">
                <label class="form-label">Folder Name</label>
                <input
                  v-model="newFolderName"
                  type="text"
                  class="form-control"
                  required
                />
              </div>
              <div class="mb-3">
                <label class="form-label">Description (Optional)</label>
                <textarea
                  v-model="newFolderDescription"
                  class="form-control"
                ></textarea>
              </div>
              <button type="submit" class="btn btn-primary">
                Create Folder
              </button>
            </form>
          </div>
        </div>
      </div>
    </div>

    <!-- Delete Confirmation Modal -->
    <div ref="deleteModal" class="modal fade" tabindex="-1">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Confirm Deletion</h5>
            <button
              type="button"
              class="btn-close"
              data-bs-dismiss="modal"
              aria-label="Close"
            ></button>
          </div>
          <div class="modal-body">
            Are you sure you want to delete "{{ folderToDelete?.name }}"? This
            will permanently remove the folder and all its contents.
          </div>
          <div class="modal-footer">
            <button
              type="button"
              class="btn btn-secondary"
              data-bs-dismiss="modal"
            >
              Cancel
            </button>
            <button type="button" class="btn btn-danger" @click="deleteFolder">
              Delete
            </button>
          </div>
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
