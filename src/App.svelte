<script>
	import { createClient } from '@supabase/supabase-js'
	import { onMount } from 'svelte'

	const { SUPABASE_URL, SUPABASE_KEY } = env
	const supabase = createClient(SUPABASE_URL, SUPABASE_KEY)

	let err = null, editError = null
	let todos = []

	onMount(async () => {
		const { error, data } = await supabase
			.from('todos')
			.select()

		if (error) err = error
		if (data) todos = data
	})

	let title = null

	async function addTodo() {
		if (!title) return err = 'Please fill in the fields'

		const { error, data } = await supabase
			.from('todos')
			.insert({ title })
			.select()
			.single()

		if (error) err = 'Could not add a todo'
		if (data) todos = [...todos, data]
	}

	let editedTodo = null

	function editTodo(id) {
		const todoIndex = todos.findIndex(v => v.id === id)
		editedTodo = todos[todoIndex].title
		todos[todoIndex].edit = true
	}

	function editCancel(id) {
		const todoIndex = todos.findIndex(v => v.id === id)
		todos[todoIndex].edit = false
	}

	async function editSave(id) {
		if (!editedTodo) return editError = 'Please fill in the fields'

		const todoIndex = todos.findIndex(v => v.id === id)
		todos[todoIndex].edit = false

		const { error, data } = await supabase
			.from('todos')
			.update({ title: editedTodo })
			.eq('id', id)
			.single()
			.select()

		if (error) editError = 'Could not edit a todo'
		if (data) todos[todoIndex] = data
	}

	async function deleteTodo(id) {
		const { error, data } = await supabase
			.from('todos')
			.delete()
			.eq('id', id)
			.select()
			.single()

		if (error) err = 'Could not delete a todo'
		if (data) todos = todos.filter(v => v.id !== data.id)
	}
</script>

<main>
	<div class="new-todo">
		<input class="title" bind:value={title} type="text" placeholder="New Todo">
		<button class="add" on:click={addTodo}>Add</button>
	</div>

	{#if err}
		<p>{err}</p>
	{/if}

	<div class="todos">
		{#each todos as todo}
			<div class="todo">
				{#if todo.edit}
					<div class="edit">
						<h4 style="margin: 0; margin-bottom: 5px;">Edit</h4>
						<input bind:value={editedTodo} class="edit-input "type="text" placeholder="Edit Todo">
						<button on:click={() => editSave(todo.id)}>Save</button>
						<button on:click={() => editCancel(todo.id)}>Cancel</button>

						{#if editError}
							<p>{editError}</p>
						{/if}
					</div>
				{:else}
					<h4>{todo.title}</h4>
					<button on:click={() => editTodo(todo.id)}>Edit</button>
					<button on:click={() => deleteTodo(todo.id)}>Delete</button>
				{/if}
			</div>
		{/each}
	</div>
</main>

<style>
	main {
		max-width: 600px;
		margin: auto;
	}

	.new-todo {
		display: flex;
		margin-bottom: 12px;
	}

	.edit-input {
		width: 100%;
	}

	.add {
		margin: 3px;
	}

	.title {
		margin: 3px;
		width: 100%;
	}

	.todo {
		border: 1px solid black;
		padding: 6px;
		margin: 8px 0;
	}
</style>
