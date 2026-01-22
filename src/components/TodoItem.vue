<script setup lang="ts">
import TodoInput from "./TodoInput.vue";
import type { Todo } from "../types/todo";

const props = defineProps<{
  todo: Todo;
  isEditing: boolean;
  editingText: string;
}>();

const emit = defineEmits<{
  (e: "toggle", id: string): void;
  (e: "delete", id: string): void;
  (e: "startEdit", todo: Todo): void;
  (e: "update:editingText", value: string): void;
  (e: "saveEdit", payload: { id: string; text: string }): void;
  (e: "cancelEdit"): void;
}>();

function toggle() {
  emit("toggle", props.todo.id);
}

function del() {
  emit("delete", props.todo.id);
}

function startEdit() {
  emit("startEdit", props.todo);
}

function save(text: string) {
  emit("saveEdit", { id: props.todo.id, text });
}
</script>

<template>
  <li class="todo-item" :class="{ 'todo-item--done': todo.completed }">
    <div v-if="!isEditing" class="todo-item__row">
      <label class="todo-item__left">
        <input
          class="todo-item__checkbox"
          type="checkbox"
          :checked="todo.completed"
          @change="toggle"
        />
        <span class="todo-item__text">{{ todo.text }}</span>
      </label>

      <div class="todo-item__actions">
        <button class="btn btn--ghost" type="button" @click="startEdit">
          Edit
        </button>
        <button class="btn btn--danger" type="button" @click="del">
          Delete
        </button>
      </div>
    </div>

    <div v-else class="todo-item__edit">
      <TodoInput
        :model-value="editingText"
        submit-label="Save"
        :show-cancel="true"
        :auto-focus="true"
        placeholder="Edit task..."
        @update:model-value="emit('update:editingText', $event)"
        @submit="save"
        @cancel="emit('cancelEdit')"
      />
    </div>
  </li>
</template>

<style scoped>
.todo-item {
  border: 1px solid var(--border);
  background: color-mix(in oklab, var(--panel) 92%, transparent);
  border-radius: 14px;
  padding: 0.85rem;
}

.todo-item__row {
  display: flex;
  gap: 0.75rem;
  align-items: center;
  justify-content: space-between;
}

.todo-item__left {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  min-width: 0;
  cursor: pointer;
}

.todo-item__checkbox {
  width: 18px;
  height: 18px;
}

.todo-item__text {
  text-align: left;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.todo-item--done .todo-item__text {
  opacity: 0.7;
  text-decoration: line-through;
}

.todo-item__actions {
  display: flex;
  gap: 0.5rem;
  flex-shrink: 0;
}

.todo-item__edit {
  width: 100%;
}

@media (max-width: 520px) {
  .todo-item__row {
    flex-direction: column;
    align-items: stretch;
  }
  .todo-item__actions {
    justify-content: flex-end;
  }
}
</style>

