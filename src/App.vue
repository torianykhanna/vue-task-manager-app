<script setup lang="ts">
import { ref, computed, onMounted, watch } from "vue";

import * as todoService from "./api/todos";
import { Todo } from "./types/Todo";
import { Filter } from "./types/Filter";
import { ErrorMessages } from "./types/ErrorMessages";
import { getFilteredTodos } from "./utils/getFilteredTodos";

import TodoList from "./components/TodoList/TodoList.vue";
import TodoItem from "./components/TodoItem/TodoItem.vue";
import TodoFooter from "./components/TodoFooter/TodoFooter.vue";
import TodoHeader from "./components/TodoHeader/TodoHeader.vue";
import ErrorMessage from "./components/ErrorMessage/ErrorMessage.vue";

const todos = ref<Todo[]>([]);
const errorMessage = ref<ErrorMessages>(ErrorMessages.None);
const newTitle = ref('');
const isLoading = ref(false);
const tempTodo = ref<Todo | null>(null);
const selectedFilter = ref<Filter>(Filter.All);
const updatingTodoIds = ref<number[]>([]);

const newTodoFieldRef = ref<HTMLInputElement | null>(null);

const hasTodos = computed(() => todos.value.length > 0);
const filteredTodos = computed(() => getFilteredTodos(todos.value, selectedFilter.value));
const notCompletedCount = computed(() => todos.value.filter(t => !t.completed).length);
const allCompleted = computed(() => hasTodos.value && todos.value.every(t => t.completed));
const hasCompletedTodos = computed(() => todos.value.some(t => t.completed));

const showError = (message: ErrorMessages) => {
  errorMessage.value = message;
};

const loadTodos = async () => {
  try {
    todos.value = await todoService.getTodos();
  } catch {
    showError(ErrorMessages.LoadTodos);
  }
};

onMounted(loadTodos);

const handleAddTodo = async () => {
  const trimmedTitle = newTitle.value.trim();
  if (!trimmedTitle) {
    showError(ErrorMessages.EmptyTitle);
    return;
  }

  isLoading.value = true;
  errorMessage.value = ErrorMessages.None;

  const newTempTodo: Todo = {
    id: 0,
    title: trimmedTitle,
    completed: false,
    userId: todoService.USER_ID,
  };

  tempTodo.value = newTempTodo;

  try {
    const newTodo = await todoService.addTodo(trimmedTitle);
    todos.value.push(newTodo);
    newTitle.value = '';
  } catch {
    showError(ErrorMessages.AddTodo);
  } finally {
    tempTodo.value = null;
    isLoading.value = false;
  }
};

const handleDeleteTodo = async (todoId: number) => {
  updatingTodoIds.value.push(todoId);
  try {
    await todoService.deleteTodo(todoId);
    todos.value = todos.value.filter(t => t.id !== todoId);
  } catch {
    showError(ErrorMessages.DeleteTodo);
  } finally {
    updatingTodoIds.value = updatingTodoIds.value.filter(id => id !== todoId);
  }
};

</script>

<template>
  <div class="todoapp">
    <h1 class="todoapp__title">todos</h1>

    <div class="todoapp__content">
      <TodoHeader
        v-model:title="newTitle"
        :is-loading="isLoading"
        :all-completed="allCompleted"
        :has-todos="todos.length > 0"
        @add="handleAddTodo"
      />

      <TodoList
        v-if="todos.length > 0 || tempTodo"
        :todos="filteredTodos"
        :updating-todo-id="updatingTodoIds"
        :temp-todo="tempTodo"
        @delete="handleDeleteTodo"
      />
      
      <TodoFooter
        v-if="todos.length > 0"
        v-model="selectedFilter"
        :count="notCompletedCount"
        :has-completed="hasCompletedTodos"
      />
    </div>

    <ErrorMessage 
      :message="errorMessage" 
      @close="errorMessage = ErrorMessages.None" 
    />
  </div>
</template>