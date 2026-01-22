<script setup lang="ts">
import { computed, onMounted, ref, watch } from "vue";
import TodoInput from "./components/TodoInput.vue";
import TodoList from "./components/TodoList.vue";
import type { Todo, TodoFilter } from "./types/todo";

const STORAGE_KEY = "todos-v1";

const todos = ref<Todo[]>([]);
const filter = ref<TodoFilter>("all");

const newText = ref("");
const editingId = ref<string | null>(null);
const editingText = ref("");

const remainingCount = computed(
  () => todos.value.filter((t) => !t.completed).length,
);
const completedCount = computed(
  () => todos.value.filter((t) => t.completed).length,
);

const filteredTodos = computed(() => {
  const list = todos.value;
  if (filter.value === "active") return list.filter((t) => !t.completed);
  if (filter.value === "completed") return list.filter((t) => t.completed);
  return list;
});

function uid() {
  return `${Date.now().toString(36)}-${Math.random().toString(36).slice(2, 10)}`;
}

function addTodo(text: string) {
  const now = Date.now();
  todos.value.unshift({
    id: uid(),
    text,
    completed: false,
    createdAt: now,
    updatedAt: now,
  });
  newText.value = "";
}

function toggleTodo(id: string) {
  const t = todos.value.find((x) => x.id === id);
  if (!t) return;
  t.completed = !t.completed;
  t.updatedAt = Date.now();
}

function deleteTodo(id: string) {
  todos.value = todos.value.filter((t) => t.id !== id);
  if (editingId.value === id) cancelEdit();
}

function startEdit(todo: Todo) {
  editingId.value = todo.id;
  editingText.value = todo.text;
}

function cancelEdit() {
  editingId.value = null;
  editingText.value = "";
}

function saveEdit(payload: { id: string; text: string }) {
  const t = todos.value.find((x) => x.id === payload.id);
  if (!t) return;
  const trimmed = payload.text.trim();
  if (!trimmed) return;
  t.text = trimmed;
  t.updatedAt = Date.now();
  cancelEdit();
}

function clearCompleted() {
  todos.value = todos.value.filter((t) => !t.completed);
  cancelEdit();
}

function isTodoShape(v: unknown): v is Todo {
  if (!v || typeof v !== "object") return false;
  const o = v as Record<string, unknown>;
  return (
    typeof o.id === "string" &&
    typeof o.text === "string" &&
    typeof o.completed === "boolean" &&
    typeof o.createdAt === "number" &&
    typeof o.updatedAt === "number"
  );
}

function loadFromStorage(): Todo[] {
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    if (!raw) return [];
    const parsed: unknown = JSON.parse(raw);
    if (!Array.isArray(parsed)) return [];
    return parsed.filter(isTodoShape);
  } catch {
    return [];
  }
}

onMounted(() => {
  todos.value = loadFromStorage();
});

watch(
  todos,
  (v) => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(v));
  },
  { deep: true },
);
</script>

<template>
  <main class="app">
    <header class="app__header">
      <div>
        <h1 class="app__title">Todo</h1>
        <p class="app__subtitle">
          {{ remainingCount }} remaining · {{ completedCount }} completed
        </p>
      </div>
    </header>

    <section class="panel">
      <TodoInput
        v-model="newText"
        placeholder="Add a new task…"
        submit-label="Add"
        :auto-focus="true"
        @submit="addTodo"
      />

      <div class="toolbar">
        <div class="filters" role="tablist" aria-label="Filters">
          <button
            class="seg"
            type="button"
            :aria-pressed="filter === 'all'"
            :class="{ 'seg--active': filter === 'all' }"
            @click="filter = 'all'"
          >
            All ({{ todos.length }})
          </button>
          <button
            class="seg"
            type="button"
            :aria-pressed="filter === 'active'"
            :class="{ 'seg--active': filter === 'active' }"
            @click="filter = 'active'"
          >
            Active ({{ remainingCount }})
          </button>
          <button
            class="seg"
            type="button"
            :aria-pressed="filter === 'completed'"
            :class="{ 'seg--active': filter === 'completed' }"
            @click="filter = 'completed'"
          >
            Completed ({{ completedCount }})
          </button>
        </div>

        <button
          class="btn btn--ghost"
          type="button"
          :disabled="completedCount === 0"
          @click="clearCompleted"
        >
          Clear completed
        </button>
      </div>

      <TodoList
        :todos="filteredTodos"
        :editing-id="editingId"
        :editing-text="editingText"
        @toggle="toggleTodo"
        @delete="deleteTodo"
        @startEdit="startEdit"
        @update:editingText="editingText = $event"
        @saveEdit="saveEdit"
        @cancelEdit="cancelEdit"
      />
    </section>
  </main>
</template>

<style scoped>
.app {
  width: min(820px, 100%);
  margin: 0 auto;
  padding: 2.25rem 1rem 3rem;
}

.app__header {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  margin-bottom: 1rem;
}

.app__title {
  margin: 0;
  font-size: 2.1rem;
  letter-spacing: -0.02em;
}

.app__subtitle {
  margin: 0.25rem 0 0;
  color: var(--muted);
  font-size: 0.95rem;
}

.panel {
  background: var(--panel);
  border: 1px solid var(--border);
  border-radius: 18px;
  padding: 1rem;
  display: grid;
  gap: 0.9rem;
}

.toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 0.75rem;
  flex-wrap: wrap;
}

.filters {
  display: inline-flex;
  border: 1px solid var(--border);
  border-radius: 999px;
  background: color-mix(in oklab, var(--panel) 92%, transparent);
  padding: 0.25rem;
  gap: 0.25rem;
}

.seg {
  border: 0;
  background: transparent;
  color: var(--muted);
  padding: 0.45rem 0.75rem;
  border-radius: 999px;
  cursor: pointer;
}

.seg--active {
  background: color-mix(in oklab, var(--brand) 18%, transparent);
  color: var(--text);
}
</style>
