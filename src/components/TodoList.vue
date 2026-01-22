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

  <p v-else class="empty">No tasks yet.</p>
</template>

<style scoped>
.list {
  margin: 0;
  padding: 0;
  display: grid;
  gap: 10px;
}

.empty {
  margin: 0;
  color: #6b7280;
}
</style>

