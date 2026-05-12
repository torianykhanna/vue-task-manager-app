<script setup lang="ts">
import { watch } from "vue";
import { ErrorMessages } from "../../types/ErrorMessages";

const props = defineProps<{
  message: ErrorMessages;
}>();

const emit = defineEmits(["close"]);

watch(
  () => props.message,
  (newMsg) => {
    if (newMsg !== ErrorMessages.None) {
      setTimeout(() => emit("close"), 3000);
    }
  },
);
</script>

<template>
  <div
    class="notification is-danger is-light has-text-weight-normal"
    :class="{ hidden: message === ErrorMessages.None }"
  >
    <button class="delete" @click="emit('close')" />
    {{ message }}
  </div>
</template>
