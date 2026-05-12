<script setup lang="ts">
import { ref, watch } from 'vue'
import type { Todo } from '../types/Todo'

const props = withDefaults(
  defineProps<{
    todo: Todo
    isLoading?: boolean
  }>(),
  {
    isLoading: false,
  },
)

const emit = defineEmits<{
  (e: 'delete'): void
  (e: 'toggle'): void
  (e: 'update', title: string): void
}>()

const isEditing = ref(false)
const editedTitle = ref(props.todo.title)

watch(
  () => props.todo.title,
  (newTitle) => {
    editedTitle.value = newTitle
    isEditing.value = false
  },
)

const saveChanges = () => {
  const trimmedTitle = editedTitle.value.trim()

  if (trimmedTitle === props.todo.title) {
    isEditing.value = false
    return
  }

  if (!trimmedTitle) {
    emit('delete')
    return
  }

  emit('update', trimmedTitle)
}

const handleSubmit = () => {
  saveChanges()
}

const handleBlur = () => {
  saveChanges()
}

const handleKeyUp = (event: KeyboardEvent) => {
  if (event.key === 'Escape') {
    editedTitle.value = props.todo.title
    isEditing.value = false
  }
}
</script>

<template>
  <div
    data-cy="Todo"
    :class="[
      'todo',
      { completed: todo.completed },
    ]"
  >
    <label class="todo__status-label">
      <input
        data-cy="TodoStatus"
        type="checkbox"
        class="todo__status"
        :checked="todo.completed"
        :disabled="isLoading"
        @change="emit('toggle')"
      />
    </label>

    <form
      v-if="isEditing"
      @submit.prevent="handleSubmit"
    >
      <input
        data-cy="TodoTitleField"
        type="text"
        class="todo__title-field"
        :value="editedTitle"
        autofocus
        :disabled="isLoading"
        @input="editedTitle = ($event.target as HTMLInputElement).value"
        @blur="handleBlur"
        @keyup="handleKeyUp"
      />
    </form>

    <template v-else>
      <span
        data-cy="TodoTitle"
        class="todo__title"
        @dblclick="isEditing = true"
      >
        {{ todo.title }}
      </span>

      <button
        type="button"
        class="todo__remove"
        data-cy="TodoDelete"
        :disabled="isLoading"
        @click="emit('delete')"
      >
        ×
      </button>
    </template>

    <div
      data-cy="TodoLoader"
      :class="[
        'modal overlay',
        { 'is-active': isLoading },
      ]"
    >
      <div class="modal-background has-background-white-ter" />
      <div class="loader" />
    </div>
  </div>
</template>