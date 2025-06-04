<template>
  <div class="main-container">
    <!-- Search Bar -->
    <div class="header-bar">
      <input
        v-model="search"
        @input="onSearch"
        class="search-bar"
        type="text"
        placeholder="Search notes..."
        aria-label="Search notes" />
      <button class="add-btn" @click="openCreateDialog" title="Add Note" aria-label="Add Note">
        <span>＋</span>
      </button>
    </div>

    <!-- Notes list -->
    <div class="notes-grid" v-if="filteredNotes.length > 0">
      <div class="note-card" v-for="note in filteredNotes" :key="note.id">
        <div class="card-header">
          <span class="note-title">{{ note.title }}</span>
          <div class="card-actions">
            <button @click="openEditDialog(note)" aria-label="Edit Note" class="edit-btn">✏️</button>
            <button @click="deleteNote(note.id)" aria-label="Delete Note" class="delete-btn">🗑️</button>
          </div>
        </div>
        <div class="note-content">{{ note.contentSnippet }}</div>
        <div class="note-date" v-if="note.updatedAt">Edited: {{ formatDate(note.updatedAt) }}</div>
      </div>
    </div>
    <div v-else class="empty-state">
      <p>No notes found.</p>
    </div>

    <!-- Add/Edit Note Modal -->
    <div class="modal-backdrop" v-if="showDialog" @click.self="closeDialog">
      <div class="modal-content" role="dialog" aria-modal="true">
        <h2>{{ dialogMode === 'create' ? 'Add New Note' : 'Edit Note' }}</h2>
        <form @submit.prevent="submitDialog">
          <input
            class="note-input"
            v-model="dialogNote.title"
            placeholder="Title"
            maxlength="64"
            required
            aria-label="Note title" />
          <textarea
            class="note-textarea"
            v-model="dialogNote.content"
            placeholder="Write your note here..."
            rows="6"
            required
            aria-label="Note content"></textarea>
          <div class="dialog-actions">
            <button type="submit" class="accent-btn">
              {{ dialogMode === 'create' ? 'Add Note' : 'Save' }}
            </button>
            <button type="button" class="cancel-btn" @click="closeDialog">Cancel</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
// PUBLIC_INTERFACE
/**
 * QuickNoteMain.vue - Main Container for QuickNote Organizer
 * Handles the note CRUD and search functionality.
 */
import { ref, computed } from 'vue'

interface Note {
  id: number;
  title: string;
  content: string;
  createdAt: Date;
  updatedAt: Date;
}

const notes = ref<Note[]>([])
const search = ref('')
const showDialog = ref(false)
const dialogMode = ref<'create' | 'edit'>('create')
const dialogNote = ref<{ id?: number; title: string; content: string }>({ title: '', content: '' })

/**
 * Filter notes by the search string.
 */
const filteredNotes = computed(() => {
  if (!search.value) {
    // Show newest first
    return notes.value.map(n => ({ ...n, contentSnippet: snippet(n.content) })).sort((a, b) => (b.updatedAt?.getTime() || 0) - (a.updatedAt?.getTime() || 0))
  }
  const s = search.value.toLowerCase()
  return notes.value
    .filter(
      (n) =>
        n.title.toLowerCase().includes(s) ||
        n.content.toLowerCase().includes(s)
    )
    .map(n => ({ ...n, contentSnippet: snippet(n.content) }))
    .sort((a, b) => (b.updatedAt?.getTime() || 0) - (a.updatedAt?.getTime() || 0))
})

function snippet(content: string) {
  const trimmed = content.trim()
  if (trimmed.length <= 80) return trimmed
  return trimmed.slice(0, 80) + '...'
}

function onSearch() {
  // No debounce for simplicity, but could add for larger note sets.
  // filteredNotes is computed.
}

// PUBLIC_INTERFACE
function openCreateDialog() {
  dialogMode.value = 'create'
  dialogNote.value = { title: '', content: '' }
  showDialog.value = true
}

// PUBLIC_INTERFACE
function openEditDialog(note: Note) {
  dialogMode.value = 'edit'
  dialogNote.value = { id: note.id, title: note.title, content: note.content }
  showDialog.value = true
}

function closeDialog() {
  showDialog.value = false
}

function submitDialog() {
  const now = new Date()
  if (dialogMode.value === 'create') {
    // Add a new note
    notes.value.push({
      id: Date.now(),
      title: dialogNote.value.title.trim(),
      content: dialogNote.value.content.trim(),
      createdAt: now,
      updatedAt: now
    })
  } else if (dialogMode.value === 'edit' && dialogNote.value.id != null) {
    // Edit existing note
    const idx = notes.value.findIndex(n => n.id === dialogNote.value.id)
    if (idx !== -1) {
      notes.value[idx] = {
        ...notes.value[idx],
        title: dialogNote.value.title.trim(),
        content: dialogNote.value.content.trim(),
        updatedAt: now
      }
    }
  }
  showDialog.value = false
}

function deleteNote(id: number) {
  if (window.confirm('Delete this note?')) {
    notes.value = notes.value.filter((n) => n.id !== id)
  }
}

function formatDate(dt: Date | undefined) {
  if (!dt) return ''
  return dt.toLocaleString()
}
</script>

<style scoped>
.main-container {
  max-width: 760px;
  margin: 32px auto;
  padding: 24px;
  background: #fff;
  border-radius: 14px;
  box-shadow: 0 4px 24px rgba(25, 118, 210, 0.10);
  min-height: 70vh;
  font-family: 'Inter', Arial, sans-serif;
}

.header-bar {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 2.5rem;
}

.search-bar {
  flex: 1 1 0%;
  font-size: 1.1rem;
  padding: 0.65rem 1rem;
  border-radius: 2rem;
  border: 1.5px solid #cfd8dc;
  background: #f7fafc;
  outline: none;
  transition: border 0.2s;
}

.search-bar:focus {
  border-color: #1976d2;
}

.add-btn {
  background: #1976D2;
  color: #fff;
  border: none;
  font-size: 1.9rem;
  border-radius: 50%;
  width: 45px;
  height: 45px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2px 8px rgba(25, 118, 210, 0.13);
  transition: background 0.2s;
}

.add-btn:hover {
  background: #12509d;
}

.notes-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(270px, 1fr));
  gap: 1.45rem;
}

.note-card {
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 2px 14px rgba(25, 118, 210, 0.08);
  padding: 1.2rem 1.25rem 1.1rem 1.25rem;
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
  position: relative;
  transition: box-shadow 0.18s;
  border-left: 6px solid #1976D2;
}

.note-card:hover {
  box-shadow: 0 4px 24px rgba(25, 118, 210, 0.16);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 0.45rem;
  margin-bottom: 0.2rem;
}

.note-title {
  font-weight: 600;
  font-size: 1.1rem;
  color: #1976D2;
  max-width: 70%;
  word-break: break-word;
}

.card-actions {
  display: flex;
  gap: 0.25rem;
}

.edit-btn, .delete-btn {
  background: transparent;
  border: none;
  font-size: 1.2rem;
  cursor: pointer;
  transition: color 0.16s;
  padding: 2px 4px;
}

/* Accent #FFC107 for accent actions */
.edit-btn:hover {
  color: #FFC107;
}
.delete-btn:hover {
  color: #D32F2F;
}

.note-content {
  color: #333;
  font-size: 1rem;
  margin-bottom: 0.2rem;
  line-height: 1.45;
}

.note-date {
  color: #bababa;
  font-size: 0.88rem;
}

/* Modal styles */
.modal-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(25, 118, 210, 0.09);
  z-index: 50;
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  max-width: 380px;
  background: #fff;
  border-radius: 16px;
  box-shadow: 0 6px 44px rgba(25, 118, 210, 0.14);
  padding: 2.2rem 2rem 1.5rem 2rem;
  display: flex;
  flex-direction: column;
  gap: 1.1rem;
  min-width: 270px;
}

.modal-content h2 {
  color: #1976D2;
  text-align: center;
  margin: 0 0 0.65rem 0;
  font-size: 1.22rem;
  font-weight: 600;
}

.note-input, .note-textarea {
  width: 100%;
  padding: 0.65rem 0.7rem;
  margin-bottom: 0.75rem;
  border-radius: 8px;
  border: 1.5px solid #cfd8dc;
  font-size: 1rem;
  outline: none;
  resize: none;
  background: #f7fafc;
}

.note-input:focus, .note-textarea:focus {
  border-color: #1976D2;
}

.dialog-actions {
  display: flex;
  justify-content: flex-end;
  gap: 1rem;
  margin-top: 0.2rem;
}

.accent-btn {
  background: #FFC107;
  color: #232323;
  border: none;
  border-radius: 5px;
  padding: 0.55rem 1.2rem;
  font-size: 1.02rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.18s;
}

.accent-btn:hover {
  background: #ffd54f;
}

.cancel-btn {
  background: transparent;
  color: #1976d2;
  border: 1.5px solid #1976d2;
  border-radius: 5px;
  padding: 0.55rem 1.2rem;
  font-size: 1.02rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.15s, color 0.15s;
}

.cancel-btn:hover {
  background: #1976d2;
  color: #fff;
}

.empty-state {
  text-align: center;
  color: #b3b3b3;
  margin-top: 54px;
  font-size: 1.1rem;
}
</style>
