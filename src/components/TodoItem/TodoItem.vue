<script setup lang="ts">
import { ref, watch, nextTick } from "vue";
import { Todo } from "../../types/Todo";

const props = defineProps<{
  todo: Todo;
  isLoading?: boolean;
}>();

const emit = defineEmits<{
  (e: "delete"): void;
  (e: "toggle"): void;
  (e: "update", title: string): void;
}>();

const isEditing = ref(false);
const editedTitle = ref(props.todo.title);
const editInput = ref<HTMLInputElement | null>(null);

watch(
  () => props.todo.title,
  (newTitle) => {
    editedTitle.value = newTitle;
    isEditing.value = false;
  },
);

const startEdit = async () => {
  isEditing.value = true;
  await nextTick();
  editInput.value?.focus();
};

const saveChanges = () => {
  const trimmed = editedTitle.value.trim();
  if (trimmed === props.todo.title) {
    isEditing.value = false;
    return;
  }
  if (!trimmed) {
    emit("delete");
    return;
  }
  emit("update", trimmed);
};

const cancelEdit = () => {
  editedTitle.value = props.todo.title;
  isEditing.value = false;
};
</script>

<template>
  <div class="todo" :class="{ completed: todo.completed }">
    <label class="todo__status-label">
      <input
        type="checkbox"
        class="todo__status"
        :checked="todo.completed"
        :disabled="isLoading"
        @change="emit('toggle')"
      />
    </label>

    <form v-if="isEditing" @submit.prevent="saveChanges">
      <input
        ref="editInput"
        type="text"
        class="todo__title-field"
        v-model="editedTitle"
        @blur="saveChanges"
        @keyup.esc="cancelEdit"
        :disabled="isLoading"
      />
    </form>

    <template v-else>
      <span class="todo__title" @dblclick="startEdit">
        {{ todo.title }}
      </span>
      <button
        class="todo__remove"
        :disabled="isLoading"
        @click="emit('delete')"
      >
        ×
      </button>
    </template>

    <div class="modal overlay" :class="{ 'is-active': isLoading }">
      <div class="modal-background has-background-white-ter" />
      <div class="loader" />
    </div>
  </div>
</template>
