<script setup lang="ts">
import { ref, onMounted } from 'vue';

const props = defineProps<{
  hasTodos: boolean;
  allCompleted: boolean;
  isLoading: boolean;
  title: string;
}>();

const emit = defineEmits<{
  (e: 'update:title', value: string): void;
  (e: 'add'): void;
  (e: 'toggleAll'): void;
}>();

const inputRef = ref<HTMLInputElement | null>(null);

onMounted(() => {
  inputRef.value?.focus();
});
</script>

<template>
  <header class="todoapp__header">
    <button
      v-if="hasTodos"
      type="button"
      class="todoapp__toggle-all"
      :class="{ active: allCompleted }"
      data-cy="ToggleAllButton"
      @click="emit('toggleAll')"
    />

    <form @submit.prevent="emit('add')">
      <input
        ref="inputRef"
        type="text"
        class="todoapp__new-todo"
        placeholder="What needs to be done?"
        data-cy="NewTodoField"
        :value="title"
        :disabled="isLoading"
        @input="e => emit('update:title', (e.target as HTMLInputElement).value)"
      />
    </form>
  </header>
</template>