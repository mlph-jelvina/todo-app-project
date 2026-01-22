<script setup lang="ts">
import { computed, nextTick, onMounted, ref, watch } from "vue";

const props = withDefaults(
  defineProps<{
    modelValue: string;
    placeholder?: string;
    submitLabel?: string;
    showCancel?: boolean;
    autoFocus?: boolean;
  }>(),
  {
    placeholder: "What do you need to do?",
    submitLabel: "Add",
    showCancel: false,
    autoFocus: false,
  },
);

const emit = defineEmits<{
  (e: "update:modelValue", value: string): void;
  (e: "submit", value: string): void;
  (e: "cancel"): void;
}>();

const value = computed({
  get: () => props.modelValue,
  set: (v: string) => emit("update:modelValue", v),
});

const inputEl = ref<HTMLInputElement | null>(null);

async function focusIfNeeded() {
  if (!props.autoFocus) return;
  await nextTick();
  inputEl.value?.focus();
}

onMounted(() => {
  void focusIfNeeded();
});

watch(
  () => props.autoFocus,
  () => {
    void focusIfNeeded();
  },
);

const isEmpty = computed(() => value.value.trim().length === 0);

function handleSubmit() {
  const trimmed = value.value.trim();
  if (!trimmed) return;
  emit("submit", trimmed);
}

function handleCancel() {
  emit("cancel");
}
</script>

<template>
  <form class="todo-input" @submit.prevent="handleSubmit">
    <input
      ref="inputEl"
      v-model="value"
      class="todo-input__field"
      type="text"
      :placeholder="placeholder"
      autocomplete="off"
    />

    <div class="todo-input__actions">
      <button class="btn btn--primary" type="submit" :disabled="isEmpty">
        {{ submitLabel }}
      </button>
      <button
        v-if="showCancel"
        class="btn btn--ghost"
        type="button"
        @click="handleCancel"
      >
        Cancel
      </button>
    </div>
  </form>
</template>

<style scoped>
.todo-input {
  display: flex;
  gap: 0.75rem;
  align-items: center;
}

.todo-input__field {
  flex: 1;
  min-width: 0;
  padding: 0.75rem 0.9rem;
  border-radius: 12px;
  border: 1px solid var(--border);
  background: var(--panel);
  color: var(--text);
  outline: none;
}

.todo-input__field:focus {
  border-color: color-mix(in oklab, var(--brand) 55%, var(--border));
  box-shadow: 0 0 0 4px color-mix(in oklab, var(--brand) 20%, transparent);
}

.todo-input__actions {
  display: flex;
  gap: 0.5rem;
}

@media (max-width: 520px) {
  .todo-input {
    flex-direction: column;
    align-items: stretch;
  }
  .todo-input__actions {
    justify-content: stretch;
  }
  .todo-input__actions :deep(.btn) {
    width: 100%;
  }
}
</style>

