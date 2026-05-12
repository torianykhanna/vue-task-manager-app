<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'

import * as todoService from './api/todos'

import type { Todo } from './types/Todo'
import { Filter } from './types/Filter'
import { ErrorMessages } from './types/ErrorMessages'

import { getFilteredTodos } from './utils/getFilteredTodos'

import TodoList from './components/TodoList.vue'
import TodoFooter from './components/TodoFooter.vue'
import TodoHeader from './components/TodoHeader.vue'
import ErrorMessage from './components/ErrorMessage.vue'

const todos = ref<Todo[]>([])
const errorMessage = ref<ErrorMessages>(ErrorMessages.None)
const newTitle = ref('')
const isLoading = ref(false)
const tempTodo = ref<Todo | null>(null)

const selectedFilter = ref<Filter>(Filter.All)
const updatingTodoId = ref<number[]>([])

const showError = (message: ErrorMessages) => {
  errorMessage.value = message
}

onMounted(async () => {
  try {
    todos.value = await todoService.getTodos()
  } catch {
    errorMessage.value = ErrorMessages.LoadTodos
  }
})

const handleFilterChange = (newFilterState: Filter) => {
  selectedFilter.value = newFilterState
}

const handleAddTodo = async () => {
  const trimmedTitle = newTitle.value.trim()

  if (!trimmedTitle) {
    showError(ErrorMessages.EmptyTitle)

    return
  }

  errorMessage.value = ErrorMessages.None
  isLoading.value = true

  const newTempTodo: Todo = {
    id: 0,
    title: trimmedTitle,
    completed: false,
    userId: todoService.USER_ID,
  }

  tempTodo.value = newTempTodo

  try {
    const newTodo = await todoService.addTodo(trimmedTitle)

    todos.value.push(newTodo)

    newTitle.value = ''
    tempTodo.value = null
  } catch {
    showError(ErrorMessages.AddTodo)

    tempTodo.value = null
  } finally {
    isLoading.value = false
  }
}

const handleDeleteTodo = async (todoId: number) => {
  errorMessage.value = ErrorMessages.None

  updatingTodoId.value.push(todoId)

  try {
    await todoService.deleteTodo(todoId)

    todos.value = todos.value.filter(todo => todo.id !== todoId)
  } catch {
    showError(ErrorMessages.DeleteTodo)
  } finally {
    updatingTodoId.value = updatingTodoId.value.filter(
      id => id !== todoId,
    )
  }
}

const handleClearCompleted = async () => {
  errorMessage.value = ErrorMessages.None

  const completedTodos = todos.value.filter(todo => todo.completed)

  const completedIds = completedTodos.map(todo => todo.id)

  updatingTodoId.value.push(...completedIds)

  const requests = completedTodos.map(todo =>
    todoService.deleteTodo(todo.id),
  )

  const results = await Promise.allSettled(requests)

  const hasError = results.some(
    result => result.status === 'rejected',
  )

  todos.value = todos.value.filter(todo => {
    const completedIndex = completedTodos.findIndex(
      completedTodo => completedTodo.id === todo.id,
    )

    return !(
      completedIndex !== -1
      && results[completedIndex].status === 'fulfilled'
    )
  })

  updatingTodoId.value = updatingTodoId.value.filter(
    id => !completedIds.includes(id),
  )

  if (hasError) {
    showError(ErrorMessages.DeleteTodo)
  }
}

const handleToggleTodo = async (todo: Todo) => {
  errorMessage.value = ErrorMessages.None

  updatingTodoId.value.push(todo.id)

  try {
    const updatedTodo = await todoService.updateTodo(
      todo.id,
      {
        completed: !todo.completed,
      },
    )

    todos.value = todos.value.map(currentTodo =>
      currentTodo.id === todo.id
        ? updatedTodo
        : currentTodo,
    )
  } catch {
    showError(ErrorMessages.UpdateTodo)
  } finally {
    updatingTodoId.value = updatingTodoId.value.filter(
      id => id !== todo.id,
    )
  }
}

const handleUpdateTodo = async (
  todo: Todo,
  title: string,
) => {
  errorMessage.value = ErrorMessages.None

  updatingTodoId.value.push(todo.id)

  try {
    const updatedTodo = await todoService.updateTodo(
      todo.id,
      { title },
    )

    todos.value = todos.value.map(currentTodo =>
      currentTodo.id === todo.id
        ? updatedTodo
        : currentTodo,
    )
  } catch {
    showError(ErrorMessages.UpdateTodo)
  } finally {
    updatingTodoId.value = updatingTodoId.value.filter(
      id => id !== todo.id,
    )
  }
}

const hasTodos = computed(() => todos.value.length > 0)

const hasCompletedTodos = computed(() =>
  todos.value.some(todo => todo.completed),
)

const allCompleted = computed(() =>
  hasTodos.value
  && todos.value.every(todo => todo.completed),
)

const filteredTodos = computed(() =>
  getFilteredTodos(
    todos.value,
    selectedFilter.value,
  ),
)

const notCompletedCount = computed(() =>
  todos.value.filter(todo => !todo.completed).length,
)

const handleToggleAll = () => {
  errorMessage.value = ErrorMessages.None

  const shouldCompleteAll = !allCompleted.value

  const todosToUpdate = todos.value.filter(
    todo => todo.completed !== shouldCompleteAll,
  )

  todosToUpdate.forEach(handleToggleTodo)
}
</script>

<template>
  <div class="todoapp">
    <h1 class="todoapp__title">
      todos
    </h1>

    <div class="todoapp__content">
      <TodoHeader
        :has-todos="hasTodos"
        :all-completed="allCompleted"
        :is-loading="isLoading"
        :new-title="newTitle"
        @update:title="newTitle = $event"
        @add-todo="handleAddTodo"
        @toggle-all="handleToggleAll"
      />

      <TodoList
        :todos="filteredTodos"
        :updating-todo-id="updatingTodoId"
        :temp-todo="tempTodo"
        @delete="handleDeleteTodo"
        @toggle="handleToggleTodo"
        @update="handleUpdateTodo"
      />

      <TodoFooter
        v-if="hasTodos"
        :not-completed-count="notCompletedCount"
        :has-completed-todos="hasCompletedTodos"
        :selected-filter="selectedFilter"
        @filter-change="handleFilterChange"
        @clear-completed="handleClearCompleted"
      />
    </div>

    <ErrorMessage
      :message="errorMessage"
      @close="errorMessage = ErrorMessages.None"
    />
  </div>
</template>