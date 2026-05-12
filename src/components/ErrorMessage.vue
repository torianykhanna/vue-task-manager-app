<script setup lang="ts">
import { watch } from 'vue'
import { ErrorMessages } from '../types/ErrorMessages'

const props = defineProps<{
  message: ErrorMessages
}>()

const emit = defineEmits<{
  (e: 'close'): void
}>()

watch(
  () => props.message,
  (newMessage, _, onCleanup) => {
    if (newMessage === ErrorMessages.None) {
      return
    }

    const timerId = window.setTimeout(() => {
      emit('close')
    }, 3000)

    onCleanup(() => {
      window.clearTimeout(timerId)
    })
  }
)
</script>

<template>
  <div
    data-cy="ErrorNotification"
    :class="[
      'notification is-danger is-light has-text-weight-normal',
      { hidden: props.message === ErrorMessages.None },
    ]"
  >
    <button
      data-cy="HideErrorButton"
      type="button"
      class="delete"
      @click="emit('close')"
    />

    {{ props.message }}
  </div>
</template>
