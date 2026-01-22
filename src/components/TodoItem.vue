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
        <input
          class="checkbox"
          type="checkbox"
          :checked="todo.completed"
          @change="toggle"
        />
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
  background: var(--surface);
  border-radius: 12px;
  padding: 12px;
  transition: box-shadow 120ms ease, transform 120ms ease,
    border-color 120ms ease;
}

.item:hover {
  border-color: color-mix(in oklab, var(--primary) 20%, var(--border));
  box-shadow: 0 10px 24px rgba(15, 23, 42, 0.08);
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

.checkbox {
  appearance: none;
  width: 20px;
  height: 20px;
  border-radius: 6px;
  border: 1px solid var(--border);
  background: var(--card);
  display: inline-grid;
  place-content: center;
  flex: 0 0 auto;
  transition: background 120ms ease, border-color 120ms ease,
    box-shadow 120ms ease, transform 120ms ease;
}

.checkbox:hover {
  border-color: color-mix(in oklab, var(--primary) 35%, var(--border));
}

.checkbox:focus-visible {
  outline: none;
  box-shadow: 0 0 0 4px color-mix(in oklab, var(--primary) 18%, transparent);
}

.checkbox:checked {
  background: var(--primary);
  border-color: var(--primary);
}

.checkbox:checked::after {
  content: "";
  width: 10px;
  height: 6px;
  border-left: 2px solid #fff;
  border-bottom: 2px solid #fff;
  transform: rotate(-45deg);
  margin-top: -1px;
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
  background: var(--card);
  color: var(--text);
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
  background: transparent;
  color: var(--text);
  border-color: var(--border);
}

.btn--secondary:hover {
  background: color-mix(in oklab, var(--surface) 88%, var(--primary));
  border-color: color-mix(in oklab, var(--primary) 22%, var(--border));
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

@media (max-width: 520px) {
  .row {
    flex-direction: column;
    align-items: stretch;
  }
  .actions {
    justify-content: flex-end;
  }
}
</style>

