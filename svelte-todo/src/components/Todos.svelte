<script lang="ts">
    import { tick } from 'svelte'

    import type { FiltersType, ITodo } from '$root/types/todo'
    import { useStorage } from '$root/stores/useStorage'

    import AddTodo from '$root/components/AddTodo.svelte'
    import Todo from '$root/components/Todo.svelte'
    import TodosLeft from './TodosLeft.svelte'
    import FilterTodos from './FilterTodos.svelte'
    import ClearTodos from './ClearTodos.svelte'

    // state
    let todos = useStorage<ITodo[]>('todos', [])

    let selectedFilter: FiltersType = 'all'
    let filtering = false

    $: todosAmount = $todos.length
    $: incompleteTodos = $todos.filter((todo) => !todo.completed).length
    $: filteredTodos = filterTodos($todos, selectedFilter)
    $: completedTodos = $todos.filter((todo) => todo.completed).length
    $: duration = filtering ? 0 : 250

    async function setFilter(newFilter: FiltersType): Promise<void> {
        filtering = true
        await tick()
        selectedFilter = newFilter
        await tick()
        filtering = false
    }

    function filterTodos(todos: ITodo[], filter: FiltersType): ITodo[] {
        switch (filter) {
        case 'all':
            return todos
        case 'active':
            return todos.filter((todo) => !todo.completed)
        case 'completed':
            return todos.filter((todo) => todo.completed)
        }
    }

    // methods
    function generateRandomId(): string {
        return Math.random().toString(16).slice(2)
    }

    function addTodo(todo: string): void {
        let newTodo: ITodo = {
            id: generateRandomId(),
            text: todo,
            completed: false,
        }
        $todos = [...$todos, newTodo]
    }

    function toggleCompleted(event: MouseEvent): void {
        let { checked } = event.target as HTMLInputElement

        $todos = $todos.map((todo) => ({
        ...todo,
        completed: checked,
        }))
    }

    function completeTodo(id: string): void {
        $todos = $todos.map((todo) => {
            if (todo.id === id) {
                todo.completed = !todo.completed
            }
            return todo
        })
    }

    function removeTodo(id: string): void {
        $todos = $todos.filter((todo) => todo.id !== id)
    }

    function editTodo(id: string, newTodo: string): void {
        let currentTodo = $todos.findIndex((todo) => todo.id === id)
        $todos[currentTodo].text = newTodo
    }

    function clearCompleted(): void {
        $todos = $todos.filter((todo) => todo.completed !== true)
    }
</script>

<main>
    <h1 class="title">todos</h1>

    <section class="todos">
        <AddTodo {addTodo} {toggleCompleted} {todosAmount} />

        {#if todosAmount}
        <ul class="todo-list">
            {#each filteredTodos as todo (todo.id)}
                <Todo {todo} {completeTodo} {removeTodo} {editTodo} {duration}/>
            {/each}
        </ul>

        <div class="actions">
            <TodosLeft {incompleteTodos} />
            <FilterTodos {selectedFilter} {setFilter} />
            <ClearTodos {clearCompleted} {completedTodos} />
        </div>
        {/if}
    </section>
</main>

<style>
  /* Todos */

  .title {
    font-size: var(--font-80);
    font-weight: inherit;
    text-align: center;
    color: var(--color-title);
  }

  .todos {
    --width: 500px;
    --todos-bg: hsl(0 0% 98%);
    --todos-text: hsl(220 20% 14%);

    width: var(--width);
    color: var(--todos-text);
    background-color: var(--todos-bg);
    border-radius: var(--radius-base);
    border: 1px solid var(--color-gray-90);
    box-shadow: 0 0 4px var(--shadow-1);
  }

  .todo-list {
    list-style: none;
  }

  .actions {
    position: relative;
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: var(--spacing-8) var(--spacing-16);
    font-size: 0.9rem;
    border-top: 1px solid var(--color-gray-90);
  }

  .actions:before {
    content: '';
    height: 40px;
    position: absolute;
    right: 0;
    bottom: 0;
    left: 0;
    box-shadow: 0 1px 1px hsla(0, 0%, 0%, 0.2), 0 8px 0 -3px hsl(0, 0%, 96%),
      0 9px 1px -3px hsla(0, 0%, 0%, 0.2), 0 16px 0 -6px hsl(0, 0%, 96%),
      0 17px 2px -6px hsla(0, 0%, 0%, 0.2);
    z-index: -1;
  }

  /* Todo */

  .todo {
    font-size: var(--font-24);
    font-weight: 400;
    border-bottom: 1px solid #ededed;
  }

  .todo:last-child {
    border-bottom: none;
  }

  .todo-check,
  .todo-text {
    display: block;
    padding: var(--spacing-16);
    color: var(--color-gray-28);
    transition: color 0.4s;
  }

  .todo-check {
    border-radius: 100%;
  }

  .completed {
    color: var(--color-gray-58);
    text-decoration: line-through;
  }

  .todo-item {
    position: relative;
    display: flex;
    align-items: center;
    padding: 0 var(--spacing-8);
  }

  .editing .todo-item {
    display: none;
  }

  .edit {
    width: 100%;
    padding: var(--spacing-8);
    font-size: var(--font-24);
    border: 1px solid #999;
    border-radius: var(--radius-base);
    box-shadow: inset 0 -1px 5px 0 var(--shadow-1);
  }

  .toggle {
    position: absolute;
    top: 26px;
    left: 13px;
    transform: scale(2);
    opacity: 0;
  }

  .toggle + label {
    background-image: url('data:image/svg+xml;utf8,%3Csvg%20xmlns%3D%22http%3A//www.w3.org/2000/svg%22%20width%3D%2240%22%20height%3D%2240%22%20viewBox%3D%22-10%20-18%20100%20135%22%3E%3Ccircle%20cx%3D%2250%22%20cy%3D%2250%22%20r%3D%2250%22%20fill%3D%22none%22%20stroke%3D%22%23949494%22%20stroke-width%3D%223%22/%3E%3C/svg%3E');
    background-repeat: no-repeat;
    background-position: 84% 50%;
  }

  .toggle:checked + label {
    background-image: url('data:image/svg+xml;utf8,%3Csvg%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%20width%3D%2240%22%20height%3D%2240%22%20viewBox%3D%22-10%20-18%20100%20135%22%3E%3Ccircle%20cx%3D%2250%22%20cy%3D%2250%22%20r%3D%2250%22%20fill%3D%22none%22%20stroke%3D%22%2359A193%22%20stroke-width%3D%223%22%2F%3E%3Cpath%20fill%3D%22%233EA390%22%20d%3D%22M72%2025L42%2071%2027%2056l-4%204%2020%2020%2034-52z%22%2F%3E%3C%2Fsvg%3E');
  }

  .remove {
    display: none;
    margin-left: auto;
    font-size: var(--font-32);
    color: var(--color-gray-58);
    transition: color 0.2s ease-out;
  }

  .remove:hover {
    color: var(--color-highlight);
  }

  .remove:after {
    content: '×';
  }

  .todo:hover .remove {
    display: block;
  }

  /* Filters */

  .filters {
    display: flex;
    gap: var(--spacing-4);
  }

  .filter {
    text-transform: capitalize;
    padding: var(--spacing-4) var(--spacing-8);
    border: 1px solid transparent;
    border-radius: var(--radius-base);
  }

  .filter:hover {
    border: 1px solid var(--color-highlight);
  }

  .selected {
    border-color: var(--color-highlight);
  }
</style>

