<script setup>
import { computed, onBeforeMount, ref, watch } from 'vue'

import StatusFilter from './components/StatusFilter.vue'
import TodoItem from './components/TodoItem.vue'

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

			<TransitionGroup
				tag="section"
				name="todolist"
				class="todoapp__main"
				v-if="todos.length > 0"
			>
				<TodoItem
					v-for="todo of visibleTodos"
					:key="todo.id"
					:todo="todo"
					@delete="todos.splice(todos.indexOf(todo), 1)"
					@update="Object.assign(todo, $event)"
				/>
			</TransitionGroup>

			<footer class="todoapp__footer" data-cy="Footer">
				<span class="todo-count" data-cy="TodosCounter">
					{{ activeTodos.length }} items left
				</span>

				<StatusFilter v-model="status" />

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
