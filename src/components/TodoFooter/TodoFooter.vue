<script setup lang="ts">
import { Filter } from "../../types/Filter";

defineProps<{
  count: number;
  hasCompleted: boolean;
  modelValue: Filter; // для v-model
}>();

const emit = defineEmits(["update:modelValue", "clear"]);

const filters = [
  { val: Filter.All, label: "All", href: "#/" },
  { val: Filter.Active, label: "Active", href: "#/active" },
  { val: Filter.Completed, label: "Completed", href: "#/completed" },
];
</script>

<template>
  <footer class="todoapp__footer">
    <span class="todo-count">{{ count }} items left</span>

    <nav class="filter">
      <a
        v-for="f in filters"
        :key="f.val"
        :href="f.href"
        class="filter__link"
        :class="{ selected: modelValue === f.val }"
        @click.prevent="emit('update:modelValue', f.val)"
      >
        {{ f.label }}
      </a>
    </nav>

    <button
      class="todoapp__clear-completed"
      :disabled="!hasCompleted"
      @click="emit('clear')"
    >
      Clear completed
    </button>
  </footer>
</template>
