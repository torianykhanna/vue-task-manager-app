<script setup lang="ts">
defineProps<{
  hasTodos: boolean
  allCompleted: boolean
  isLoading: boolean
  newTitle: string
}>()

const emit = defineEmits<{
  (e: 'update:title', value: string): void
  (e: 'addTodo'): void
  (e: 'toggleAll'): void
}>()
</script>

<template>
  <header class="todoapp__header">
    <button
      v-if="hasTodos"
      data-cy="ToggleAllButton"
      type="button"
      :class="[
        'todoapp__toggle-all',
        { active: allCompleted },
      ]"
      @click="emit('toggleAll')"
    />

    <form @submit.prevent="emit('addTodo')">
      <input
        data-cy="NewTodoField"
        type="text"
        class="todoapp__new-todo"
        placeholder="What needs to be done?"
        :value="newTitle"
        @input="emit('update:title', ($event.target as HTMLInputElement).value)"
        autofocus
        :disabled="isLoading"
      />
    </form>
  </header>
</template>