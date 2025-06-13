<!-- eslint-disable @typescript-eslint/no-unused-vars -->
<script setup lang="ts">
import TodoHeader from '../components/TodoHeader.vue'
import { computed, onBeforeMount, ref, watch } from 'vue'
import TodoList from '../components/TodoList.vue'
import TodoFooter from '../components/TodoFooter.vue'
import type { Todo } from '../types/Todo'

const todos = ref<Todo[]>([])

// 1. Завантаження даних перед монтуванням
onBeforeMount(() => {
  loadFromLocalStorage()
})

// 2. Функція для повторного використання
const loadFromLocalStorage = () => {
  try {
    const data = localStorage.getItem('todos')
    todos.value = data ? JSON.parse(data) : []
  } catch (e) {
    console.error('Помилка завантаження даних:', e)
    todos.value = []
  }
}

// 3. Автоматичне збереження при змінах
watch(
  todos,
  (newTodos) => {
    localStorage.setItem('todos', JSON.stringify(newTodos))
  },
  { deep: true },
)

const status = ref('all')
const errorMessage = ref('')

const setErrorMessage = (message: string) => {
  errorMessage.value = message
}

const clearErrorMessage = () => {
  errorMessage.value = ''
}

const activeTodos = computed(() => todos.value.filter((todo) => !todo.completed))
const completedTodos = computed(() => todos.value.filter((todo) => todo.completed))

const visibleTodos = computed(() => {
  if (status.value === 'active') return todos.value.filter((t) => !t.completed)
  if (status.value === 'completed') return todos.value.filter((t) => t.completed)
  return todos.value
})

const handleClearCompleted = () => {
  todos.value = todos.value.filter((todo) => !todo.completed)
}

const handleAddTodo = (newTodo: Todo) => {
  if (!newTodo.title) {
    errorMessage.value = 'Title should not be empty'
    return
  }

  todos.value.push(newTodo)
}

const handleUpdateTodo = ({ id, title }: { id: number; title: string }) => {
  if (!title) {
    errorMessage.value = 'Title should not be empty'
    return
  }

  const index = todos.value.findIndex((todo) => todo.id === id)
  if (index !== -1) {
    todos.value[index].title = title
  }
}

const handleDeleteTodo = (id: number) => {
  todos.value = todos.value.filter(todo => todo.id !== id)
}


const handleToggleTodo = (id: number) => {
  todos.value = todos.value.map((todo) =>
    todo.id === id ? { ...todo, completed: !todo.completed } : todo,
  )
}

const handleToggleClick = () => {
  todos.value = todos.value.map((todo) => ({
    ...todo,
    completed: !todo.completed,
  }))
}
</script>

<template>
  <div class="todoapp">
    <h1 class="todoapp__title">todos {{ visibleTodos.length }}</h1>

    <div class="todoapp__content">
      <TodoHeader
        :visibleTodos="visibleTodos"
        :status="status"
        @add-todo="handleAddTodo"
        @toggle-items="handleToggleClick"
        @set-error="setErrorMessage"
      />

      <TodoList
        :visibleTodos="visibleTodos"
        :status="status"
        @add-todo="handleAddTodo"
        @update-todo="handleUpdateTodo"
        @delete-todo="handleDeleteTodo"
        @toggle-todo="handleToggleTodo"
        @clear-error="clearErrorMessage"
      />
      <!-- set-error="setErrorMessage" -->

      <TodoFooter
        v-if="todos.length > 0"
        :status="status"
        @update:status="(newStatus) => (status = newStatus)"
        :activeTodos="activeTodos"
        :completedTodos="completedTodos"
        :todos="todos"
        @clear-completed="handleClearCompleted"
      />
    </div>

    <div v-if="errorMessage" class="notification is-danger is-light has-text-weight-normal">
      <button class="delete" @click="clearErrorMessage"></button>
      {{ errorMessage }}
    </div>
    <!-- <ErrorNotification errorMessage="{errorMessage}" errorMessageHandler="{setErrorMessage}" /> -->
  </div>
</template>
