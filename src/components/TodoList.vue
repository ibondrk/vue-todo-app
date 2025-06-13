<script setup lang="ts">
import type { Todo } from '../types/Todo'
import { ref, onMounted, onUnmounted } from 'vue'

const props = defineProps<{
  status: string
  visibleTodos: Todo[]
}>()

const emit = defineEmits<{
  (e: 'update:status', value: string): void
  (e: 'clear-completed'): void
  (e: 'add-todo', todo: Todo): void
  (e: 'update-todo', payload: { id: number; title: string }): void
  (e: 'delete-todo', id: number): void
  (e: 'toggle-todo', id: number): void
  (e: 'clear-error'): void
}>()

const title = ref('')
const isDoubleClick = ref(false)
const choosedTodoId = ref<number | null>(null)
const formRefs = ref<HTMLFormElement[]>([])

const setFormRef = (el: HTMLFormElement | null, index: number) => {
  if (el) {
    formRefs.value[index] = el
  }
}

const clickOutsideHandler = (event: MouseEvent) => {
  if (!isDoubleClick.value) return

  const clickedOutsideAllForms = formRefs.value.every(
    (formEl) => formEl && !formEl.contains(event.target as Node),
  )

  if (clickedOutsideAllForms) {
    handleRenameSubmit()
  }
}

function handleDoubleClick(id: number, currentTitle: string) {
  isDoubleClick.value = true
  choosedTodoId.value = id
  title.value = currentTitle
}

function handleRenameSubmit() {
  if (choosedTodoId.value !== null) {
    emit('update-todo', {
      id: choosedTodoId.value,
      title: title.value,
    })
  }
  isDoubleClick.value = false
  choosedTodoId.value = null
}

function deleteTodo(todoId: number) {
  emit('delete-todo', todoId)
}

function toggleTodo(todoId: number) {
  emit('toggle-todo', todoId)
}

const escHandler = (event: KeyboardEvent) => {
  if (event.key === 'Escape' && isDoubleClick.value) {
    event.stopPropagation()
    isDoubleClick.value = false
    choosedTodoId.value = null
  }
}

onMounted(() => {
  document.addEventListener('keydown', escHandler)
  document.addEventListener('click', clickOutsideHandler)
})

onUnmounted(() => {
  document.removeEventListener('keydown', escHandler)
  document.removeEventListener('click', clickOutsideHandler)
})
</script>

<template>
  <TransitionGroup tag="section" name="todolist" class="todoapp__main">
    <div
      v-for="(todo, i) of props.visibleTodos"
      :key="todo.id"
      class="todo"
      :class="{ completed: todo.completed }"
    >
      <label class="todo__status-label">
        <input
          type="checkbox"
          class="todo__status"
          :checked="todo.completed"
          @change="() => toggleTodo(todo.id)"
        />
      </label>

      <form
        v-if="isDoubleClick && choosedTodoId === todo.id"
        @submit.prevent="handleRenameSubmit"
        @blur="handleRenameSubmit"
        :ref="(el) => setFormRef(el as HTMLFormElement | null, i)"
      >
        <input
          type="text"
          class="todoapp__change-todo"
          v-model.trim="title"
          v-focus
          @input="emit('clear-error')"
        />
      </form>

      <template v-else>
        <span class="todo__title" @dblclick="handleDoubleClick(todo.id, todo.title)">
          {{ todo.title }}
        </span>

        <button type="button" class="todo__remove" @click="deleteTodo(todo.id)">×</button>
      </template>
    </div>
  </TransitionGroup>
</template>

<style scoped>
.todolist-enter-active,
.todolist-leave-active {
  max-height: 60px;
  transition: all 0.5s ease;
}
.todolist-enter-from,
.todolist-leave-to {
  opacity: 0;
  max-height: 0;
  transform: scaleY(0);
}
</style>
