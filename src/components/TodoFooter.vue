<!-- eslint-disable @typescript-eslint/no-unused-vars -->
<script setup lang="ts">
// import type { SortType } from '../types/Filter'
// import originalTodos from '../data/todo'
import type { Todo } from '../types/Todo'
import type { SortType } from '../types/Filter.ts'
import { computed } from 'vue'
const props = defineProps<{
  status: string
  activeTodos: Todo[]
  completedTodos: Todo[]
  todos: Todo[]
}>()

const emit = defineEmits<{
  (e: 'update:status', value: string): void
  (e: 'clear-completed'): void
}>()

const isCompleted = computed(() => props.todos.some((todo) => todo.completed))

console.log('isCompleted: ', isCompleted)
</script>

<template>
  <footer className="todoapp__footer">
    <span class="todo-count">{{ props.activeTodos.length }} items left</span>

    <nav className="filter">
      <!-- :class="{ selected: sortType === SortType?.All }" -->
      <!-- @click="onSortChange(SortType.All)" -->
      <a
        href="#/"
        class="filter__link"
        :class="{ selected: props.status === 'all' }"
        @click="emit('update:status', 'all')"
      >
        All
      </a>

      <a
        href="#/active"
        class="filter__link"
        :class="{ selected: props.status === 'active' }"
        @click="emit('update:status', 'active')"
      >
        <!-- @click="status = 'active'" -->
        Active
      </a>
      <a
        href="#/completed"
        class="filter__link"
        :class="{ selected: props.status === 'completed' }"
        @click="emit('update:status', 'completed')"
      >
        Completed
      </a>
    </nav>

    <button
      v-if="isCompleted"
      class="todoapp__clear-completed"
      :disabled="props.todos.length === props.activeTodos.length"
      @click="emit('clear-completed')"
    >
      Clear completed
    </button>
  </footer>
</template>
