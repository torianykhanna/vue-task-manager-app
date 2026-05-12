<script setup lang="ts">
import type { Todo } from '../types/Todo'
import TodoItem from '../components/TodoItem.vue'

defineProps<{
  todos: Todo[]
  updatingTodoId: number[]
  tempTodo: Todo | null
}>()

const emit = defineEmits<{
  (e: 'delete', todoId: number): void
  (e: 'toggle', todo: Todo): void
  (e: 'update', todo: Todo, title: string): void
}>()
</script>

<template>
  <section
    v-if="todos.length > 0 || tempTodo"
    class="todoapp__main"
    data-cy="TodoList"
  >
    <TodoItem
      v-for="todo in todos"
      :key="todo.id"
      :todo="todo"
      :is-loading="updatingTodoId.includes(todo.id)"
      @delete="emit('delete', todo.id)"
      @toggle="emit('toggle', todo)"
      @update="(title: string) => emit('update', todo, title)"
    />

    <TodoItem
      v-if="tempTodo"
      :todo="tempTodo"
      :is-loading="true"
      @delete="emit('delete', tempTodo.id)"
      @toggle="() => {}"
      @update="() => {}"
    />
  </section>
</template>