<script setup>
import { computed, onBeforeMount, ref, watch } from 'vue'

import StatusFilter from "./components/StatusFilter.vue";
import TodoItem from './components/TodoItem.vue';

const todos = ref([])

onBeforeMount(() => {
	try {
		todos.value = JSON.parse(localStorage.getItem('todos'))
	} catch (error) {}

	if (!Array.isArray(todos.value)) {
		todos.value = []
	}
})

const title = ref('')
const errorMessage = ref('')
const status = ref('all')

function addTodo() {
	if (!title.value) {
		errorMessage.value = 'Title should not be empty'

		return
	}

	todos.value.push({
		id: Date.now(),
		title: title.value,
		completed: false,
	})

	title.value = ''
}

const activeTodos = computed(() => todos.value.filter(todo => !todo.completed))

const visibleTodos = computed(() => {
	if (status.value === 'active') {
		return activeTodos.value
	}

	if (status.value === 'completed') {
		return todos.value.filter(todo => todo.completed)
	}

	return todos.value
})

watch(
	todos,
	newTodos => {
		localStorage.setItem('todos', JSON.stringify(newTodos))
	},
	{ deep: true }
)
</script>

<template>
	<div class="todoapp">
		<h1 class="todoapp__title">todos</h1>

		<div class="todoapp__content">
			<header class="todoapp__header">
				<button
					type="button"
					class="todoapp__toggle-all"
					:class="{ active: activeTodos.length === 0 }"
				></button>

				<form @submit.prevent="addTodo">
					<input
						type="text"
						class="todoapp__new-todo"
						placeholder="What needs to be done?"
						v-model="title"
						@input="errorMessage = ''"
					/>
				</form>
			</header>

      <TodoItem 
        v-for="todo of visibleTodos"
        :key="todo.id"
        :todo="todo"
        @delete="todos.splice(todos.indexOf(todo), 1)"
        @update="todos[todos.indexOf(todo)] = $event"
      />

			<!-- <section class="todoapp__main" data-cy="TodoList">
				<div
					v-for="(todo, i) of visibleTodos"
					:key="todo.id"
					class="todo"
					:class="{ completed: todo.completed }"
				>
					<label class="todo__status-label">
						<input
							type="checkbox"
							class="todo__status"
							:checked="todo.completed"
							v-model="todo.completed"
						/>
					</label>

					<form v-if="false">
						<input
							type="text"
							class="todo__title-field"
							placeholder="Empty todo will be deleted"
						/>
					</form>

					<template v-else>
						<span class="todo__title">{{ todo.title }}</span>
						<button
							type="button"
							class="todo__remove"
							@click="todos.splice(i, 1)"
						>
							×
						</button>
					</template>

					<div class="modal overlay" :class="{ 'is-active': false }">
						<div class="modal-background has-background-white-ter"></div>
						<div class="loader"></div>
					</div>
				</div>
			</section> -->

			<footer class="todoapp__footer" data-cy="Footer">
				<span class="todo-count" data-cy="TodosCounter">
					{{ activeTodos.length }} items left
				</span>

				<StatusFilter v-model="status"/>

				<button
					type="button"
					class="todoapp__clear-completed"
					:disabled="todos.length === activeTodos.length"
					@click="todos = activeTodos"
				>
					Clear completed
				</button>
			</footer>
		</div>

		<div
			v-if="errorMessage"
			class="notification is-danger is-light has-text-weight-normal"
		>
			<button type="button" class="delete" @click="errorMessage = ''"></button>
			{{ errorMessage }}
		</div>
	</div>
</template>
