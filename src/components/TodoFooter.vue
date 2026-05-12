<script setup lang="ts">
import { Filter } from '../types/Filter'

defineProps<{
  notCompletedCount: number
  hasCompletedTodos: boolean
  selectedFilter: Filter
}>()

const emit = defineEmits<{
  (e: 'filterChange', filter: Filter): void
  (e: 'clearCompleted'): void
}>()

const filterOptions = [
  {
    value: Filter.All,
    label: 'All',
    href: '#/',
    dataCy: 'FilterLinkAll',
  },
  {
    value: Filter.Active,
    label: 'Active',
    href: '#/active',
    dataCy: 'FilterLinkActive',
  },
  {
    value: Filter.Completed,
    label: 'Completed',
    href: '#/completed',
    dataCy: 'FilterLinkCompleted',
  },
]
</script>

<template>
  <footer
    class="todoapp__footer"
    data-cy="Footer"
  >
    <span
      class="todo-count"
      data-cy="TodosCounter"
    >
      {{ notCompletedCount }} items left
    </span>

    <nav
      class="filter"
      data-cy="Filter"
    >
      <a
        v-for="filter in filterOptions"
        :key="filter.value"
        :href="filter.href"
        :data-cy="filter.dataCy"
        :class="[
          'filter__link',
          { selected: selectedFilter === filter.value },
        ]"
        @click.prevent="emit('filterChange', filter.value)"
      >
        {{ filter.label }}
      </a>
    </nav>

    <button
      type="button"
      class="todoapp__clear-completed"
      data-cy="ClearCompletedButton"
      :disabled="!hasCompletedTodos"
      @click="emit('clearCompleted')"
    >
      Clear completed
    </button>
  </footer>
</template>