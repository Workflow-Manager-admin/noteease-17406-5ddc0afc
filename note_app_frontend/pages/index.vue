<template>
  <div class="container">
    <!-- Search Bar -->
    <header>
      <input
        v-model="searchQuery"
        placeholder="Search notes..."
        class="search-bar"
      />
    </header>

    <!-- Tag Filters -->
    <div class="tag-filters">
      <button
        :class="{ active: selectedTag === null }"
        @click="selectTag(null)"
      >
        All
      </button>
      <button
        v-for="tag in uniqueTags"
        :key="tag"
        :class="{ active: selectedTag === tag }"
        @click="selectTag(tag)"
      >
        {{ tag }}
      </button>
    </div>

    <!-- Notes Grid -->
    <div class="notes-grid">
      <div class="note-card" v-for="note in filteredNotes" :key="note.id">
        <h3>{{ note.title }}</h3>
        <p>{{ note.contentSnippet }}</p>
        <!-- Note Tags -->
        <div class="tags">
          <span class="tag" v-for="tag in note.tags" :key="tag">{{ tag }}</span>
        </div>
        <!-- Card Actions -->
        <div class="actions">
          <button @click="startEdit(note)" class="btn-edit">Edit</button>
          <button @click="deleteNote(note.id)" class="btn-delete">Delete</button>
        </div>
      </div>
    </div>

    <!-- Floating Action Button -->
    <button class="fab" @click="openCreate">+</button>

    <!-- Create/Edit Modal -->
    <div v-if="showModal" class="modal-overlay">
      <div class="modal">
        <h2>{{ isEditing ? 'Edit Note' : 'Create Note' }}</h2>
        <input v-model="form.title" placeholder="Title" />
        <textarea v-model="form.content" placeholder="Content"></textarea>
        <input
          v-model="form.tagsString"
          placeholder="Tags (comma separated)"
        />
        <div class="modal-actions">
          <button @click="saveNote" class="btn-save">Save</button>
          <button @click="closeModal" class="btn-cancel">Cancel</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useLocalStorage } from '@vueuse/core'

// Persist notes in localStorage
const notes = useLocalStorage('notes', [])

// Reactive states
const searchQuery = ref('')
const selectedTag = ref(null)
const showModal = ref(false)
const isEditing = ref(false)
const editId = ref(null)
const form = ref({
  title: '',
  content: '',
  tagsString: ''
})

// Compute unique tags from notes
const uniqueTags = computed(() => {
  const set = new Set()
  notes.value.forEach(n => n.tags.forEach(t => set.add(t)))
  return Array.from(set)
})

// Compute filtered & mapped notes
const filteredNotes = computed(() => {
  return notes.value
    .filter(n => {
      const text = searchQuery.value.toLowerCase()
      const matchesSearch =
        n.title.toLowerCase().includes(text) ||
        n.content.toLowerCase().includes(text)
      const matchesTag =
        selectedTag.value === null || n.tags.includes(selectedTag.value)
      return matchesSearch && matchesTag
    })
    .map(n => ({
      ...n,
      contentSnippet:
        n.content.length > 100
          ? n.content.slice(0, 100) + '...'
          : n.content
    }))
})

// Filter selection handler
function selectTag(tag) {
  selectedTag.value = tag
}

// Open modal for creating a new note
function openCreate() {
  isEditing.value = false
  form.value = { title: '', content: '', tagsString: '' }
  showModal.value = true
}

// Open modal to edit existing note
function startEdit(note) {
  isEditing.value = true
  editId.value = note.id
  form.value = {
    title: note.title,
    content: note.content,
    tagsString: note.tags.join(',')
  }
  showModal.value = true
}

// Close modal
function closeModal() {
  showModal.value = false
}

// Save or update note
function saveNote() {
  const tags = form.value.tagsString
    .split(',')
    .map(t => t.trim())
    .filter(t => t)
  if (isEditing.value) {
    // Update
    const idx = notes.value.findIndex(n => n.id === editId.value)
    if (idx !== -1) {
      notes.value[idx] = {
        ...notes.value[idx],
        title: form.value.title,
        content: form.value.content,
        tags
      }
    }
  } else {
    // Create
    const newNote = {
      id: Date.now(),
      title: form.value.title,
      content: form.value.content,
      tags
    }
    notes.value.unshift(newNote)
  }
  closeModal()
}

// Delete a note by id
function deleteNote(id) {
  notes.value = notes.value.filter(n => n.id !== id)
}
</script>

<style scoped>
.container {
  padding: 16px;
  background-color: var(--color-secondary);
  min-height: 100vh;
}

/* Search Bar */
.search-bar {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  margin-bottom: 16px;
}

/* Tag Filters */
.tag-filters {
  display: flex;
  gap: 8px;
  margin-bottom: 16px;
}
.tag-filters button {
  padding: 4px 8px;
  border: none;
  background: #f0f0f0;
  border-radius: 4px;
  cursor: pointer;
}
.tag-filters button.active {
  background: var(--color-accent);
  color: #fff;
}

/* Notes Grid */
.notes-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 16px;
}
.note-card {
  background: var(--color-secondary);
  border: 1px solid #ddd;
  padding: 12px;
  border-radius: 8px;
  position: relative;
}
.note-card h3 {
  margin: 0 0 8px;
}
.tags {
  margin: 8px 0;
}
.tag {
  display: inline-block;
  background: var(--color-accent);
  color: #fff;
  padding: 2px 6px;
  border-radius: 4px;
  margin-right: 4px;
}

/* Card Actions */
.actions {
  position: absolute;
  top: 8px;
  right: 8px;
  display: flex;
  gap: 4px;
}
.btn-edit,
.btn-delete {
  background: transparent;
  border: none;
  cursor: pointer;
  color: var(--color-primary);
}

/* Floating Action Button */
.fab {
  position: fixed;
  bottom: 24px;
  right: 24px;
  background: var(--color-primary);
  color: #fff;
  border: none;
  width: 56px;
  height: 56px;
  border-radius: 50%;
  font-size: 24px;
  cursor: pointer;
}

/* Modal Overlay */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}
.modal {
  background: var(--color-secondary);
  padding: 24px;
  border-radius: 8px;
  width: 90%;
  max-width: 400px;
}
.modal input,
.modal textarea {
  width: 100%;
  margin-bottom: 12px;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}
.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 8px;
}
.btn-save {
  background: var(--color-primary);
  color: #fff;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
}
.btn-cancel {
  background: #ccc;
  color: #333;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
}
</style>
