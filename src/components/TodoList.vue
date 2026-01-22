<script setup lang="ts">
import TodoItem from "./TodoItem.vue";
import type { Todo } from "../types/todo";

defineProps<{
  todos: Todo[];
  editingId: string | null;
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
</script>

<template>
  <ul v-if="todos.length > 0" class="todo-list">
    <TodoItem
      v-for="t in todos"
      :key="t.id"
      :todo="t"
      :is-editing="editingId === t.id"
      :editing-text="editingText"
      @toggle="emit('toggle', $event)"
      @delete="emit('delete', $event)"
      @startEdit="emit('startEdit', $event)"
      @update:editingText="emit('update:editingText', $event)"
      @saveEdit="emit('saveEdit', $event)"
      @cancelEdit="emit('cancelEdit')"
    />
  </ul>

  <div v-else class="todo-empty">
    No tasks yet. Add one above.
  </div>
</template>

<style scoped>
.todo-list {
  list-style: none;
  padding: 0;
  margin: 0;
  display: grid;
  gap: 0.75rem;
}

.todo-empty {
  padding: 1rem;
  border: 1px dashed var(--border);
  border-radius: 14px;
  color: var(--muted);
  text-align: center;
}
</style>

