<script setup lang="ts">
import { ref, watch } from "vue";
import type { Todo } from "../types/todo";

const props = defineProps<{
  todo: Todo;
}>();

const emit = defineEmits<{
  (e: "toggle", id: string): void;
  (e: "delete", id: string): void;
  (e: "edit", payload: { id: string; text: string }): void;
}>();

// Local state (Component Basics)
const isEditing = ref(false);
const editText = ref(props.todo.text);

watch(
  () => props.todo.text,
  (newText) => {
    if (!isEditing.value) editText.value = newText;
  },
);

function toggle() {
  emit("toggle", props.todo.id);
}

function del() {
  emit("delete", props.todo.id);
}

function startEdit() {
  isEditing.value = true;
  editText.value = props.todo.text;
}

function cancelEdit() {
  isEditing.value = false;
  editText.value = props.todo.text;
}

function saveEdit() {
  const trimmed = editText.value.trim();
  if (!trimmed) return;
  emit("edit", { id: props.todo.id, text: trimmed });
  isEditing.value = false;
}
</script>

<template>
  <li class="item" :class="{ done: todo.completed }">
    <div v-if="!isEditing" class="row">
      <label class="left">
        <input type="checkbox" :checked="todo.completed" @change="toggle" />
        <span
          class="text"
          :style="{ textDecoration: todo.completed ? 'line-through' : 'none' }"
        >
          {{ todo.text }}
        </span>
      </label>

      <div class="actions">
        <button class="btn btn--secondary" type="button" @click="startEdit">
          Edit
        </button>
        <button class="btn btn--danger" type="button" @click="del">
          Delete
        </button>
      </div>
    </div>

    <form v-else class="row" @submit.prevent="saveEdit">
      <input v-model="editText" class="input" />
      <button class="btn" type="submit">Save</button>
      <button class="btn btn--secondary" type="button" @click="cancelEdit">
        Cancel
      </button>
    </form>
  </li>
</template>

<style scoped>
.item {
  list-style: none;
  border: 1px solid var(--border);
  background: white;
  border-radius: 12px;
  padding: 12px;
}

.row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
}

.left {
  display: flex;
  align-items: center;
  gap: 10px;
  flex: 1;
  min-width: 0;
}

.text {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.actions {
  display: flex;
  gap: 8px;
}

.input {
  flex: 1;
  padding: 10px 12px;
  border: 1px solid var(--border);
  border-radius: 8px;
  outline: none;
}

.input:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 4px color-mix(in oklab, var(--primary) 18%, transparent);
}

.btn {
  padding: 8px 10px;
  border: 1px solid var(--primary);
  border-radius: 8px;
  background: var(--primary);
  color: white;
}

.btn:hover {
  background: var(--primary-2);
  border-color: var(--primary-2);
}

.btn--secondary {
  background: white;
  color: var(--text);
  border-color: var(--border);
}

.btn--secondary:hover {
  background: #f8fafc;
  border-color: #cbd5e1;
}

.btn--danger {
  border-color: var(--danger);
  background: var(--danger);
}

.btn--danger:hover {
  filter: brightness(0.95);
}

.done {
  opacity: 0.85;
}
</style>

