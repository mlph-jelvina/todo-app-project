<script setup lang="ts">
import TodoItem from "./TodoItem.vue";
import type { Todo } from "../types/todo";

defineProps<{
  todos: Todo[];
}>();

const emit = defineEmits<{
  (e: "toggle", id: string): void;
  (e: "delete", id: string): void;
  (e: "edit", payload: { id: string; text: string }): void;
}>();
</script>

<template>
  <ul v-if="todos.length" class="list">
    <TodoItem
      v-for="t in todos"
      :key="t.id"
      :todo="t"
      @toggle="emit('toggle', $event)"
      @delete="emit('delete', $event)"
      @edit="emit('edit', $event)"
    />
  </ul>

  <div v-else class="empty">
    <p class="empty__text">No tasks yet.</p>
  </div>
</template>

<style scoped>
.list {
  margin: 0;
  padding: 0;
  display: grid;
  gap: 10px;
}

.empty {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 48px 24px;
  border: 1px dashed var(--border);
  border-radius: 12px;
  background: color-mix(in oklab, var(--surface) 50%, transparent);
  text-align: center;
}

.empty__text {
  margin: 0;
  font-size: 16px;
  font-weight: 500;
  color: var(--text);
}
</style>

