<!-- eslint-disable @typescript-eslint/no-unused-vars -->
<script setup lang="ts">
import type { Todo } from '../types/Todo'
import { ref } from 'vue'

const props = defineProps<{
  visibleTodos: Todo[]
  status: string
}>()

const emit = defineEmits<{
  (e: 'add-todo', todo: Todo): void
  (e: 'toggle-items'): void
  (e: 'set-error', message: string): void
}>()

const title = ref('')

function addTodo() {
  if (!title.value) {
    // errorMessage.value = 'Title should not be empty'
    emit('set-error', 'Title should not be empty')
    return
  }

  emit('add-todo', {
    id: Date.now(),
    title: title.value,
    completed: false,
  })

  title.value = ''
}

</script>

<template>
  <header className="todoapp__header">
    <button
      v-if="props.visibleTodos.length > 0"
      type="button"
      class="todoapp__toggle-all"
      :class="{ active: false }"
      @click="emit('toggle-items')"
      ></button>

    <form @submit.prevent="addTodo">
      <input
        type="text"
        className="todoapp__new-todo"
        placeholder="What needs to be done?"
        v-model="title"
        />
        <!-- disabled="{isAdding}" -->
    </form>
  </header>
</template>
