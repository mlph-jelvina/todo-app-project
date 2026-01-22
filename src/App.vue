<script setup lang="ts">
import { computed, onMounted, ref, watch } from "vue";
import TodoInput from "./components/TodoInput.vue";
import TodoList from "./components/TodoList.vue";
import type { Todo, TodoFilter } from "./types/todo";

const todos = ref<Todo[]>([]);
const filter = ref<TodoFilter>("all");

const filteredTodos = computed(() => {
  if (filter.value === "active") return todos.value.filter((t) => !t.completed);
  if (filter.value === "completed") return todos.value.filter((t) => t.completed);
  return todos.value;
});

const remaining = computed(() => todos.value.filter((t) => !t.completed).length);
const completed = computed(() => todos.value.filter((t) => t.completed).length);

function makeId() {
  return `${Date.now()}-${Math.random().toString(16).slice(2)}`;
}

function addTodo(text: string) {
  todos.value.unshift({
    id: makeId(),
    text,
    completed: false,
  });
}

function toggleTodo(id: string) {
  const t = todos.value.find((x) => x.id === id);
  if (!t) return;
  t.completed = !t.completed;
}

function deleteTodo(id: string) {
  todos.value = todos.value.filter((t) => t.id !== id);
}

function editTodo(payload: { id: string; text: string }) {
  const t = todos.value.find((x) => x.id === payload.id);
  if (!t) return;
  t.text = payload.text;
}

function clearCompleted() {
  todos.value = todos.value.filter((t) => !t.completed);
}

onMounted(() => {
  const raw = localStorage.getItem("todos");
  if (!raw) return;
  try {
    const parsed = JSON.parse(raw) as Todo[];
    if (Array.isArray(parsed)) todos.value = parsed;
  } catch {

  }
});

watch(
  todos,
  () => {
    localStorage.setItem("todos", JSON.stringify(todos.value));
  },
  { deep: true },
);
</script>

<template>
  <div class="card">
    <h1>Todo App</h1>

    <TodoInput @add="addTodo" />

    <div class="toolbar">
      <div class="filters">
        <button :class="{ active: filter === 'all' }" @click="filter = 'all'">
          All ({{ todos.length }})
        </button>
        <button
          :class="{ active: filter === 'active' }"
          @click="filter = 'active'"
        >
          Active ({{ remaining }})
        </button>
        <button
          :class="{ active: filter === 'completed' }"
          @click="filter = 'completed'"
        >
          Completed ({{ completed }})
        </button>
      </div>

      <button :disabled="completed === 0" @click="clearCompleted">
        Clear completed
      </button>
    </div>

    <TodoList
      :todos="filteredTodos"
      @toggle="toggleTodo"
      @delete="deleteTodo"
      @edit="editTodo"
    />
  </div>
</template>

<style scoped>
.card {
  width: 100%;
  max-width: 720px;
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 20px;
  display: grid;
  gap: 14px;
  box-shadow: 0 18px 50px rgba(15, 23, 42, 0.12);
}

h1 {
  margin: 0;
  font-size: 24px;
  letter-spacing: -0.02em;
}

.toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  flex-wrap: wrap;
}

.filters {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.filters button,
.toolbar > button {
  padding: 8px 10px;
  border-radius: 999px;
  border: 1px solid var(--border);
  background: #f8fafc;
  color: var(--text);
}

.filters button.active {
  border-color: var(--primary);
  background: var(--primary);
  color: white;
}

.filters button:hover,
.toolbar > button:hover {
  border-color: #cbd5e1;
  background: #ffffff;
}

.filters button.active:hover {
  background: var(--primary-2);
  border-color: var(--primary-2);
}

.toolbar > button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}
</style>

