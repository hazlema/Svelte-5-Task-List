<script>
	/**
	 * @typedef {('all'|'completed'|'progress')} FiltersType
	 * @typedef {{id?: number | undefined; task: string; done: boolean; }} TodoType
	 * @typedef {TodoType[]} TodosType
	 */

	/** @type number */
	let counter = $state(1);

	/** @type TodosType */
	let todos = $state([]);

	/** @type FiltersType */
	let filters = $state('all');

	/** @type TodosType */
	let filteredTodos = $derived(filterTodos());

	/**
	 *  @function addTodoWithId
	 *  @param { TodoType } item
	 *  @description adds an item to the todo list with an ID
	 *  @return {void}
	 *******************************************************************************************/
	function addTodoWithId(item) {
		item['id'] = counter;
		counter++;
		todos.push(item);
	}

	/**
	 * @function getId
	 * @description Returns the event id
	 * @param {Event | KeyboardEvent} event
	 * @return {number}
	 *******************************************************************************************/
	function getId(event) {
		if (!(event.target instanceof HTMLElement)) {
			throw new Error('Event target is not an HTMLElement');
		}
		const id = event.target.dataset.id;
		if (!id) {
			throw new Error('No id found in dataset');
		}
		return parseInt(id, 10);
	}

	/**
	 *  @function addTodo
	 *  @description When the user presses a key or clicks the ADD button
	 *  @param {KeyboardEvent} [event]
	 *  @return { void }
	 *******************************************************************************************/
	function addTodo(event) {
		if (event && event.key != 'Enter') return;
		// event?.preventDefault();

		/** @type {EventTarget | null | undefined} */
		let ele = event instanceof EventTarget ? event.target : document.getElementById('todo-input');

		if (ele instanceof HTMLInputElement) {
			if (ele.value.trim().length >= 1) {
				addTodoWithId({
					task: ele.value.trim(),
					done: false
				});
			}
			ele.value = '';
		} else {
			throw new Error('Target is not an HTMLInputElement');
		}
	}

	/**
	 *  @function updateTodo
	 *  @description Update the todo
	 *  @param {KeyboardEvent} [event]
	 *  @return { void }
	 *******************************************************************************************/
	function updateTodo(event) {
		if (event) {
			let id = getId(event);
			console.log(`Update - ID ${id}`);

			/** @type {EventTarget | null | undefined} */
			let ele = event.target;

			if (ele && ele instanceof HTMLInputElement) {
				let found = todos.find((obj) => obj.id == id);

				if (found) {
					found.task = ele.value.trim();
				}
			} else {
				throw new Error('Target is not an HTMLInputElement');
			}
		}
	}

	/**
	 * @function remove(event)
	 * @description Removes an ID from the todo array
	 * @param { MouseEvent } event
	 * @return { void }
	 ******************************************************************************************/
	function remove(event) {
		let id = getId(event);
		console.log(`Remove Task - ID ${id}`);

		todos = todos.filter((obj) => obj.id !== id);
	}

	/**
	 * @function toggleCheck(event)
	 * @description Removes toggles the done status
	 * @param { Event } event
	 * @return {void}
	 ******************************************************************************************/
	function toggleCheck(event) {
		let id = getId(event);
		console.log(`Toggle Check - ID ${id}`);

		let found = todos.find((obj) => obj.id == id);

		if (found) {
			found.done = !found.done;
		}
	}

	/**
	 * @function setFilter
	 * @description Sets the filter for the view
	 * @param { FiltersType } value
	 * @return {void}
	 ******************************************************************************************/
	function setFilter(value) {
		filters = value;
	}

	/**
	 * @function filterTodos ($derived)
	 * @description Returns a subset of todos
	 * @return TodosType
	 ******************************************************************************************/
	function filterTodos() {
		if (filters === 'completed') return todos.filter((obj) => obj.done);
		if (filters === 'progress') return todos.filter((obj) => !obj.done);

		return todos;
	}

	/**
	 * @function summery
	 * @description generate the summery with the filteredTodos count
	 * @return string
	 ******************************************************************************************/
	function summery() {
		if (filters === 'completed') return `Completed tasks: ${filteredTodos.length}`;
		if (filters === 'progress') return `Tasks in progress: ${filteredTodos.length}`;

		return `Total tasks: ${filteredTodos.length}`;
	}

	/**
	 * Load tasks from local if they exist
	 ******************************************************************************************/
	$effect(() => {
		const loadTasks = localStorage.getItem('tasks');

		if (loadTasks) {
			console.log('Loading localStorage saved tasks...');
			todos = JSON.parse(loadTasks);
		}
	});

	/**
	 * Save tasks to local
	 ******************************************************************************************/
	$effect(() => {
		if (todos.length > 0) {
			console.log('Updating localStorage...');
			localStorage.setItem('tasks', JSON.stringify(todos));
		}
	});

	$inspect(todos, counter, filters);
</script>

<main>
		<h1>Task List</h1>
		<form id="todo-form">
			<input
				id="todo-input"
				onkeydown={(event) => addTodo(event)}
				type="text"
				placeholder="Enter a new task"
			/>
			<button onclick={() => addTodo()} type="button">Add</button>
		</form>
		<div class="todo-grid">
			{#each filteredTodos as todo}
				<div class="todo-item">
					<input
						data-id={todo.id}
						onchange={toggleCheck}
						type="checkbox"
						class="todo-checkbox"
						checked={todo.done}
					/>
					<input
						data-id={todo.id}
						onkeydown={(event) => updateTodo(event)}
						class="todo-text"
						class:completed={todo.done}
						value={todo.task}
					/>
					<button data-id={todo.id} onclick={remove} class="delete-btn">Delete</button>
				</div>
			{/each}
		</div>
		<br />
		<button onclick={() => setFilter('all')}>All</button>
		<button onclick={() => setFilter('progress')}>In Progress</button>
		<button onclick={() => setFilter('completed')}>Completed</button>
		<div class="summery">{@html summery()}</div>
</main>

<style>
	:global(body) {
		font-family: Arial, sans-serif;
		max-width: 500px;
		margin: 0 auto;
		padding: 20px;
		background-color: #1a1a1a;
		color: #e0e0e0;
		border-radius: 10px;
	}
	h1 {
		text-align: center;
		color: #ffffff;
	}
	#todo-form {
		display: grid;
		grid-template-columns: 1fr auto;
		gap: 10px;
		margin-bottom: 20px;
	}
	#todo-input {
		padding: 10px;
		font-size: 16px;
		background-color: #333;
		border: 1px solid #555;
		color: #e0e0e0;
	}
	button {
		padding: 10px 20px;
		font-size: 16px;
		background-color: #4caf50;
		color: white;
		border: none;
		cursor: pointer;
	}
	.todo-grid {
		display: grid;
		gap: 10px;
	}
	.todo-item {
		display: grid;
		grid-template-columns: auto 1fr auto;
		align-items: center;
		gap: 10px;
		background-color: #2a2a2a;
		padding: 10px;
		border-radius: 5px;
	}
	.delete-btn {
		background-color: #f44336;
	}
	.todo-text {
		background-color: transparent;
		border: none;
		color: #e0e0e0;
		font-size: 16px;
		width: 100%;
	}
	.todo-text:focus {
		outline: none;
		border-bottom: 1px solid #4caf50;
	}
	.completed .todo-text {
		text-decoration: line-through;
		color: #888;
	}
	.completed {
		text-decoration: line-through;
		color: #888;
	}
	.completed:focus {
		text-decoration: none;
		color: #888;
	}
	.summery {
		margin-top: 15px;
		font-size: 20px;
	}
</style>
