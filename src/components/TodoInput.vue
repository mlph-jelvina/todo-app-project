<script setup lang="ts">
import { computed, onMounted, ref } from "vue";

const emit = defineEmits<{
  (e: "add", text: string): void;
}>();

const text = ref("");
const inputRef = ref<HTMLInputElement | null>(null);

const isEmpty = computed(() => text.value.trim().length === 0);

onMounted(() => {
  inputRef.value?.focus();
});

function submit() {
  const trimmed = text.value.trim();
  if (!trimmed) return;
  emit("add", trimmed);
  text.value = "";
  inputRef.value?.focus();
}
</script>

<template>
  <form class="row" @submit.prevent="submit">
    <input
      ref="inputRef"
      v-model="text"
      class="input"
      placeholder="Add a new task..."
      autocomplete="off"
    />
    <button class="btn" type="submit" :disabled="isEmpty">Add</button>
  </form>
</template>

<style scoped>
.row {
  display: flex;
  gap: 8px;
}

.input {
  flex: 1;
  padding: 10px 12px;
  border: 1px solid var(--border);
  border-radius: 8px;
  background: var(--surface);
  color: var(--text);
  outline: none;
}

.input:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 4px color-mix(in oklab, var(--primary) 18%, transparent);
}

.btn {
  padding: 10px 12px;
  border: 1px solid var(--primary);
  border-radius: 8px;
  background: var(--primary);
  color: white;
}

.btn:hover {
  background: var(--primary-2);
  border-color: var(--primary-2);
}

.btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}
</style>

